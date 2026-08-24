---
title: Go 安全性實踐
type: concept
tags: [go, security, jwt, cors, rate-limiting, authentication]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Go 安全性實踐

## 安全威脅速覽

| 威脅 | 描述 | 本文對應章節 |
|------|------|------------|
| 認證繞過 | 未驗證 JWT / Session 偽造 | JWT 認證 |
| CORS 濫用 | 惡意網站讀取 API 回應 | CORS 設定 |
| DDoS / 爆破 | 短時間大量請求 | Rate Limiting |
| SQL Injection | 惡意 SQL 注入 | 參數化查詢 |
| 敏感資料洩露 | 錯誤訊息含內部資訊 | 錯誤處理 |

---

## JWT 認證

### 生成與驗證

```go
// go get github.com/golang-jwt/jwt/v5
import (
    "github.com/golang-jwt/jwt/v5"
    "time"
    "errors"
)

type Claims struct {
    UserID int64  `json:"user_id"`
    Role   string `json:"role"`
    jwt.RegisteredClaims
}

var ErrInvalidToken = errors.New("invalid token")

func GenerateToken(userID int64, role string, secret []byte) (string, error) {
    claims := Claims{
        UserID: userID,
        Role:   role,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(24 * time.Hour)),
            IssuedAt:  jwt.NewNumericDate(time.Now()),
            Issuer:    "my-service",
        },
    }
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(secret)
}

func ValidateToken(tokenStr string, secret []byte) (*Claims, error) {
    token, err := jwt.ParseWithClaims(tokenStr, &Claims{}, func(t *jwt.Token) (any, error) {
        // 重要：驗證 alg，防止 alg=none 攻擊
        if _, ok := t.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, ErrInvalidToken
        }
        return secret, nil
    })
    if err != nil || !token.Valid {
        return nil, ErrInvalidToken
    }
    claims, ok := token.Claims.(*Claims)
    if !ok {
        return nil, ErrInvalidToken
    }
    return claims, nil
}
```

### JWT Middleware（net/http）

```go
func JWTMiddleware(secret []byte, next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        auth := r.Header.Get("Authorization")
        if !strings.HasPrefix(auth, "Bearer ") {
            http.Error(w, "missing token", http.StatusUnauthorized)
            return
        }
        tokenStr := strings.TrimPrefix(auth, "Bearer ")
        claims, err := ValidateToken(tokenStr, secret)
        if err != nil {
            http.Error(w, "invalid token", http.StatusUnauthorized)
            return
        }
        // 將 claims 存入 context
        ctx := context.WithValue(r.Context(), claimsKey{}, claims)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

type claimsKey struct{}

func ClaimsFromCtx(ctx context.Context) (*Claims, bool) {
    c, ok := ctx.Value(claimsKey{}).(*Claims)
    return c, ok
}
```

### Refresh Token 機制

```go
// Access Token：短效（15 分鐘），儲存在記憶體
// Refresh Token：長效（30 天），儲存在 Redis + HttpOnly Cookie

func RefreshTokens(refreshToken string, rdb *redis.Client, secret []byte) (string, string, error) {
    // 1. 從 Redis 驗證 refresh token 是否存在且未被撤銷
    userID, err := rdb.Get(ctx, "refresh:"+refreshToken).Result()
    if err != nil {
        return "", "", ErrInvalidToken
    }

    // 2. 生成新的 access token
    id, _ := strconv.ParseInt(userID, 10, 64)
    newAccess, _ := GenerateToken(id, "user", secret)

    // 3. Rotate refresh token（舊的刪掉，發新的）
    newRefresh := generateSecureToken(32)
    pipe := rdb.Pipeline()
    pipe.Del(ctx, "refresh:"+refreshToken)
    pipe.Set(ctx, "refresh:"+newRefresh, userID, 30*24*time.Hour)
    pipe.Exec(ctx)

    return newAccess, newRefresh, nil
}

func generateSecureToken(n int) string {
    b := make([]byte, n)
    rand.Read(b)
    return base64.URLEncoding.EncodeToString(b)
}
```

---

## CORS 設定

```go
// go get github.com/rs/cors
import "github.com/rs/cors"

func NewCORSMiddleware(allowedOrigins []string) func(http.Handler) http.Handler {
    c := cors.New(cors.Options{
        AllowedOrigins:   allowedOrigins,   // ["https://myapp.com"]，不用 *
        AllowedMethods:   []string{"GET", "POST", "PUT", "DELETE", "OPTIONS"},
        AllowedHeaders:   []string{"Authorization", "Content-Type"},
        ExposedHeaders:   []string{"X-Request-ID"},
        AllowCredentials: true,   // 允許 Cookie（此時不能用 *）
        MaxAge:           86400,  // Preflight 快取 24 小時
    })
    return c.Handler
}

// 使用
handler := NewCORSMiddleware([]string{
    "https://app.example.com",
    "https://admin.example.com",
})
http.ListenAndServe(":8080", handler(mux))
```

**生產環境 CORS 要點**：
- 不要用 `AllowedOrigins: []string{"*"}` 同時設 `AllowCredentials: true`（瀏覽器會拒絕）
- 只允許你實際使用的 Method 和 Header
- 開發環境可用 `cors.AllowAll()`，生產環境要明確設定

---

## Rate Limiting

### 令牌桶（Token Bucket）— golang.org/x/time/rate

```go
import "golang.org/x/time/rate"

// 全域限流（整個 API）
globalLimiter := rate.NewLimiter(rate.Limit(1000), 2000) // 1000 req/s，burst 2000

// Per-IP 限流（更精細）
type IPRateLimiter struct {
    limiters sync.Map
    r        rate.Limit
    burst    int
}

func NewIPRateLimiter(r rate.Limit, burst int) *IPRateLimiter {
    return &IPRateLimiter{r: r, burst: burst}
}

func (l *IPRateLimiter) GetLimiter(ip string) *rate.Limiter {
    v, ok := l.limiters.Load(ip)
    if !ok {
        limiter := rate.NewLimiter(l.r, l.burst)
        l.limiters.Store(ip, limiter)
        return limiter
    }
    return v.(*rate.Limiter)
}

func RateLimitMiddleware(limiter *IPRateLimiter) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            ip := realIP(r)  // 考慮 X-Forwarded-For
            if !limiter.GetLimiter(ip).Allow() {
                w.Header().Set("Retry-After", "1")
                http.Error(w, "rate limit exceeded", http.StatusTooManyRequests)
                return
            }
            next.ServeHTTP(w, r)
        })
    }
}

func realIP(r *http.Request) string {
    if ip := r.Header.Get("X-Forwarded-For"); ip != "" {
        return strings.Split(ip, ",")[0]
    }
    host, _, _ := net.SplitHostPort(r.RemoteAddr)
    return host
}
```

### Redis 分散式限流（滑動窗口）

```go
// 適合多實例部署時（本地限流在多實例下不準確）
const rateLimitScript = `
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local window = tonumber(ARGV[2])
local now = tonumber(ARGV[3])

redis.call("ZREMRANGEBYSCORE", key, 0, now - window)
local count = redis.call("ZCARD", key)

if count < limit then
    redis.call("ZADD", key, now, now)
    redis.call("EXPIRE", key, window / 1000 + 1)
    return 1
end
return 0
`

func IsAllowed(rdb *redis.Client, key string, limit int, windowMs int64) (bool, error) {
    now := time.Now().UnixMilli()
    result, err := rdb.Eval(ctx, rateLimitScript,
        []string{"ratelimit:" + key},
        limit, windowMs, now,
    ).Int()
    return result == 1, err
}
```

---

## SQL Injection 防禦

```go
// ❌ 危險：字串拼接
query := "SELECT * FROM users WHERE name = '" + name + "'"

// ✅ 安全：參數化查詢
row := db.QueryRowContext(ctx, "SELECT * FROM users WHERE name = $1", name)

// ✅ 使用 sqlc（編譯期確保參數化）
// 所有 query 在 .sql 檔案中，由 sqlc 自動生成安全的 Go 代碼
user, err := queries.GetUserByName(ctx, name)
```

---

## 安全的錯誤處理

```go
// ❌ 洩露內部資訊
http.Error(w, err.Error(), http.StatusInternalServerError)
// 可能輸出：pq: relation "users" does not exist（暴露 DB schema）

// ✅ 對外隱藏細節，對內 log 完整資訊
type APIError struct {
    Code    string `json:"error"`
    Message string `json:"message"`
}

func handleError(w http.ResponseWriter, r *http.Request, err error, status int) {
    // 完整錯誤記錄在 log（含 request ID）
    slog.Error("request failed",
        "request_id", r.Header.Get("X-Request-ID"),
        "error", err,
        "path", r.URL.Path,
    )
    // 對外只回傳通用訊息
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(APIError{
        Code:    "INTERNAL_ERROR",
        Message: "An internal error occurred",
    })
}
```

---

## 密碼雜湊

```go
// go get golang.org/x/crypto
import "golang.org/x/crypto/bcrypt"

func HashPassword(password string) (string, error) {
    hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    return string(hash), err
}

func CheckPassword(password, hash string) bool {
    err := bcrypt.CompareHashAndPassword([]byte(hash), []byte(password))
    return err == nil
}
// 注意：bcrypt.DefaultCost = 10，登入耗時約 100ms，是刻意的（防爆破）
```

---

## 常見面試問題

**Q: JWT 的 alg=none 攻擊是什麼？**
A: 攻擊者將 JWT header 的 alg 改為 "none"，使 library 跳過簽名驗證。防禦：在解析時明確指定允許的 alg（見 ValidateToken 範例）。

**Q: CORS 是客戶端限制還是伺服器限制？**
A: CORS 是瀏覽器強制執行的機制（伺服器配合回傳 header），直接用 curl 呼叫不受 CORS 限制。因此 CORS 不是安全防禦，認證才是。

**Q: Rate Limiting 放在哪一層最好？**
A: 優先在 API Gateway / Nginx / CDN 層做（省去 App 的計算），App 層做第二道防線。分散式場景必須用共享存儲（Redis）。

**Q: bcrypt vs Argon2，選哪個？**
A: 新系統推薦 Argon2id（2015 Password Hashing Competition 冠軍，對 GPU 更友好）；已有 bcrypt 的系統無需立刻遷移，bcrypt 仍安全。

---

## 相關頁面

- [[Go錯誤處理最佳實踐]] — 錯誤分層與對外隱藏
- [[Go微服務配置管理]] — Secret 的安全存取
- [[冪等性設計]] — Rate Limiting 的補充（重複請求處理）
- [[Python FastAPI深度實戰]] — Python 版安全設定對照
