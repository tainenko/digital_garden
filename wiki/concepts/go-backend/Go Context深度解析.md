---
title: Go Context 深度解析
type: concept
tags: [golang, context, cancellation, timeout, tracing, senior]
created: 2026-04-29
updated: 2026-04-29
---

# Go Context 深度解析

## Context 是什麼

`context.Context` 是 Go 跨 API 邊界傳遞的標準載體，攜帶三類資訊：

| 功能 | 說明 | 函數 |
|------|------|------|
| **取消信號** | 告訴下游「現在可以停了」| `WithCancel` |
| **截止時間** | 到點自動取消 | `WithDeadline` / `WithTimeout` |
| **請求範圍值** | Trace ID、用戶 ID 等跨層資料 | `WithValue` |

## Context 樹（核心機制）

```
context.Background()
    └─ WithCancel → cancelCtx（呼叫 cancel() 取消）
        └─ WithTimeout(5s) → timerCtx（5 秒後或父取消時取消）
            └─ WithValue("user_id", "u123")
                └─ 傳入 HTTP Handler / DB Query / gRPC Call
```

**關鍵特性**：父取消 → 所有子孫自動取消；子取消 → 不影響父。

## 四個建構函數

```go
// 1. Background：根 context，永不取消
// 用於 main、init、全域初始化
ctx := context.Background()

// 2. TODO：佔位，表示「之後會加 context，現在先用這個」
// 比用 Background 更清楚表達意圖
ctx := context.TODO()

// 3. WithCancel：手動取消
ctx, cancel := context.WithCancel(parent)
defer cancel() // 必須呼叫，否則 goroutine 洩漏！

// 4. WithTimeout / WithDeadline：時間到自動取消
ctx, cancel := context.WithTimeout(parent, 5*time.Second)
defer cancel() // 即使 timeout 前完成，也要呼叫 cancel 釋放資源

ctx, cancel := context.WithDeadline(parent, time.Now().Add(5*time.Second))
defer cancel()

// Go 1.21+：WithoutCancel（建立不繼承取消的子 context）
// 用於需要在 parent 取消後繼續執行的清理操作
cleanupCtx := context.WithoutCancel(ctx)
go cleanup(cleanupCtx)
```

## 取消信號的傳播

```go
func fetchData(ctx context.Context, url string) ([]byte, error) {
    // 建立 HTTP request，綁定 context
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        // ctx 被取消時，err 會是 context.Canceled 或 context.DeadlineExceeded
        if ctx.Err() != nil {
            return nil, fmt.Errorf("request cancelled: %w", ctx.Err())
        }
        return nil, err
    }
    defer resp.Body.Close()
    return io.ReadAll(resp.Body)
}

// 呼叫端：5 秒超時
func handler(w http.ResponseWriter, r *http.Request) {
    ctx, cancel := context.WithTimeout(r.Context(), 5*time.Second)
    defer cancel()

    data, err := fetchData(ctx, "https://api.example.com/data")
    if errors.Is(err, context.DeadlineExceeded) {
        http.Error(w, "upstream timeout", http.StatusGatewayTimeout)
        return
    }
    // ...
}
```

## 在 goroutine 中正確使用 Context

```go
// ❌ 錯誤：goroutine 啟動後 ctx 可能已取消
func processAsync(ctx context.Context, items []Item) {
    for _, item := range items {
        go func(item Item) {
            // 沒有檢查 ctx，即使請求已取消仍繼續執行
            process(item)
        }(item)
    }
}

// ✅ 正確：goroutine 內部也要 select ctx
func processAsync(ctx context.Context, items []Item) {
    for _, item := range items {
        item := item
        go func() {
            select {
            case <-ctx.Done():
                return // 父已取消，退出
            default:
            }
            process(ctx, item) // 把 ctx 傳下去
        }()
    }
}

// ✅ 長迴圈：每次迭代都要檢查
func longRunningJob(ctx context.Context, ids []string) error {
    for _, id := range ids {
        // 每次迭代檢查一次
        if err := ctx.Err(); err != nil {
            return fmt.Errorf("job interrupted after %d items: %w", processed, err)
        }
        if err := processOne(ctx, id); err != nil {
            return err
        }
        processed++
    }
    return nil
}
```

## WithValue：傳遞請求範圍資料

```go
// ✅ 正確：用自定義型別作為 key，避免與其他 package 衝突
type contextKey string

const (
    keyUserID   contextKey = "user_id"
    keyTraceID  contextKey = "trace_id"
    keyLogger   contextKey = "logger"
)

// 包裝成類型安全的 helper
func WithUserID(ctx context.Context, userID string) context.Context {
    return context.WithValue(ctx, keyUserID, userID)
}

func UserIDFromContext(ctx context.Context) (string, bool) {
    id, ok := ctx.Value(keyUserID).(string)
    return id, ok
}

// 在 middleware 設定
func authMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        userID := validateToken(r.Header.Get("Authorization"))
        ctx := WithUserID(r.Context(), userID)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

// 在 handler 取得
func createOrder(w http.ResponseWriter, r *http.Request) {
    userID, ok := UserIDFromContext(r.Context())
    if !ok {
        http.Error(w, "unauthorized", 401)
        return
    }
    // ...
}
```

## ❌ Context 的 Anti-patterns

### 1. 把 context 存進 struct

```go
// ❌ 錯誤：context 應該是參數，不是 struct 欄位
type Server struct {
    ctx context.Context // 不應該這樣
}

// ✅ 正確：每次呼叫傳入當次的 ctx
func (s *Server) HandleRequest(ctx context.Context, req *Request) error {
    return s.db.Query(ctx, req.Query)
}
```

### 2. 用 context.Background() 繞過取消

```go
// ❌ 建立新的 Background context，切斷了取消鏈
func saveToCache(ctx context.Context, key string, value []byte) {
    // 如果請求被取消，cache 寫入仍然繼續（可能是預期行為，也可能是 bug）
    go redis.Set(context.Background(), key, value, time.Hour)
}

// ✅ 需要在取消後繼續執行，用 WithoutCancel（Go 1.21+）
func saveToCache(ctx context.Context, key string, value []byte) {
    cacheCtx := context.WithoutCancel(ctx) // 保留 value，但不繼承取消
    go redis.Set(cacheCtx, key, value, time.Hour)
}
```

### 3. context value 存放業務邏輯必需的參數

```go
// ❌ 把 DB 連線或業務參數塞進 context
ctx = context.WithValue(ctx, "db", db) // 難以追蹤、型別不安全

// ✅ context value 只放：trace ID、request ID、logger、auth info
// 業務必需的參數應該作為明確的函數參數
func CreateOrder(ctx context.Context, db *DB, req CreateOrderRequest) (*Order, error)
```

### 4. 忘記呼叫 cancel()

```go
// ❌ cancel 沒有被呼叫，即使函數返回，內部 timer goroutine 還在跑
func fetch(parent context.Context) {
    ctx, _ := context.WithTimeout(parent, 5*time.Second) // cancel 被拋棄
    http.Get(ctx, "https://...")
}

// ✅ 一定要 defer cancel()
func fetch(parent context.Context) {
    ctx, cancel := context.WithTimeout(parent, 5*time.Second)
    defer cancel()
    http.Get(ctx, "https://...")
}
```

## WithTimeout vs WithDeadline

```go
// WithTimeout：相對時間（從現在起 N 秒後）
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)

// WithDeadline：絕對時間（到某個時刻）
deadline := time.Now().Add(5 * time.Second)
ctx, cancel := context.WithDeadline(ctx, deadline)

// 兩者等價。WithTimeout 內部就是呼叫 WithDeadline

// 取得截止時間
if deadline, ok := ctx.Deadline(); ok {
    remaining := time.Until(deadline)
    if remaining < 100*time.Millisecond {
        return nil, errors.New("insufficient time remaining")
    }
}
```

## 實際應用：Middleware 鏈

```go
// 完整的 HTTP middleware 鏈示範
func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/orders", createOrderHandler)

    handler := chain(mux,
        requestIDMiddleware,  // 注入 request_id
        timeoutMiddleware(30*time.Second), // 請求超時
        authMiddleware,       // 驗證 token，注入 user_id
        tracingMiddleware,    // 注入 trace_id（OTel）
    )
    http.ListenAndServe(":8080", handler)
}

func timeoutMiddleware(timeout time.Duration) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            ctx, cancel := context.WithTimeout(r.Context(), timeout)
            defer cancel()

            done := make(chan struct{})
            go func() {
                next.ServeHTTP(w, r.WithContext(ctx))
                close(done)
            }()

            select {
            case <-done:
            case <-ctx.Done():
                w.WriteHeader(http.StatusGatewayTimeout)
            }
        })
    }
}
```

## context.Cause（Go 1.20+）

```go
// 傳遞更詳細的取消原因
ctx, cancel := context.WithCancelCause(parent)

// 在某個地方取消並附上原因
cancel(errors.New("user rate limited"))

// 任何地方都可以取得原因
if cause := context.Cause(ctx); cause != nil {
    log.Errorf("cancelled because: %v", cause)
}
```

## 相關頁面

- [[Go並發模式]] — goroutine 與 context 取消的配合
- [[Go優雅關機與健康檢查]] — context 在關機流程中的應用
- [[OpenTelemetry分散式追蹤]] — trace ID 通過 context 傳播
- [[Go錯誤處理最佳實踐]] — context.DeadlineExceeded / context.Canceled 的錯誤處理
