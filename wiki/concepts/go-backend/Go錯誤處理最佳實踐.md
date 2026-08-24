---
title: Go 錯誤處理最佳實踐
type: concept
tags: [golang, error-handling, best-practices, microservices]
created: 2026-04-29
updated: 2026-04-29
---

# Go 錯誤處理最佳實踐

## Go 錯誤處理的核心哲學

Go 把錯誤當普通值（不是異常），這讓錯誤路徑和正常路徑同樣清晰可見，但也需要明確的策略來避免混亂。

## 錯誤的三種類型

### 1. Sentinel Error（哨兵錯誤）

預定義的錯誤值，用 `errors.Is` 比較：

```go
// 定義
var (
    ErrNotFound   = errors.New("not found")
    ErrUnauthorized = errors.New("unauthorized")
    ErrDuplicate  = errors.New("duplicate")
)

// 使用
func (r *Repo) FindByID(ctx context.Context, id string) (*Order, error) {
    if !found {
        return nil, ErrNotFound
    }
    return order, nil
}

// 比對
err := repo.FindByID(ctx, id)
if errors.Is(err, ErrNotFound) {
    // 處理不存在
}
```

**適合**：預期的業務錯誤，呼叫者需要判斷錯誤類型

### 2. Custom Error Type（自定義錯誤類型）

需要攜帶額外資訊時：

```go
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error on field %s: %s", e.Field, e.Message)
}

// 使用 errors.As 取得具體類型
var valErr *ValidationError
if errors.As(err, &valErr) {
    // 可以訪問 valErr.Field, valErr.Message
    respondWithValidationError(w, valErr)
}
```

### 3. 包裝錯誤（Error Wrapping）

保留原始錯誤，加入上下文：

```go
func (s *OrderService) CreateOrder(ctx context.Context, req *CreateOrderRequest) (*Order, error) {
    user, err := s.userRepo.FindByID(ctx, req.UserID)
    if err != nil {
        // %w 包裝：保留 errors.Is / errors.As 的能力
        return nil, fmt.Errorf("get user for order: %w", err)
    }

    order, err := s.orderRepo.Create(ctx, user, req)
    if err != nil {
        return nil, fmt.Errorf("create order in db: %w", err)
    }

    return order, nil
}

// 錯誤鏈：
// "create order: get user for order: not found"
// 可以用 errors.Is(err, ErrNotFound) 解包到底層
```

## 錯誤處理模式

### 只在你能處理的地方處理

```go
// ❌ 錯誤：每層都 log，導致重複 log
func (r *Repo) FindByID(id string) (*Order, error) {
    row := r.db.QueryRow(...)
    if err != nil {
        log.Errorf("db error: %v", err) // ← 這層不應該 log
        return nil, err
    }
}
func (s *Service) GetOrder(id string) (*Order, error) {
    order, err := r.FindByID(id)
    if err != nil {
        log.Errorf("service error: %v", err) // ← 重複 log
        return nil, err
    }
}

// ✅ 正確：只在最外層（Handler）log
func (r *Repo) FindByID(id string) (*Order, error) {
    if err != nil {
        return nil, fmt.Errorf("FindByID %s: %w", id, err) // 只包裝，不 log
    }
}
func (h *Handler) GetOrder(w http.ResponseWriter, r *http.Request) {
    order, err := h.service.GetOrder(id)
    if err != nil {
        log.Errorf("GetOrder handler: %v", err) // 最外層才 log
        http.Error(w, "internal error", 500)
    }
}
```

### 分層的 Error Mapping

各層用各層的錯誤類型，在邊界轉換：

```go
// Repository 層：DB 錯誤
func (r *OrderRepo) FindByID(ctx context.Context, id string) (*Order, error) {
    var order Order
    err := r.db.GetContext(ctx, &order, "SELECT * FROM orders WHERE id = $1", id)
    if err == sql.ErrNoRows {
        return nil, ErrNotFound // 轉換成業務層錯誤
    }
    if err != nil {
        return nil, fmt.Errorf("db query: %w", err)
    }
    return &order, nil
}

// Service 層：業務錯誤
func (s *OrderService) GetOrder(ctx context.Context, userID, orderID string) (*Order, error) {
    order, err := s.repo.FindByID(ctx, orderID)
    if errors.Is(err, ErrNotFound) {
        return nil, ErrNotFound // 向上透傳
    }
    if err != nil {
        return nil, fmt.Errorf("get order: %w", err)
    }

    // 業務規則：只能看自己的訂單
    if order.UserID != userID {
        return nil, ErrForbidden
    }

    return order, nil
}

// Handler 層：HTTP 錯誤
func (h *Handler) GetOrder(w http.ResponseWriter, r *http.Request) {
    order, err := h.service.GetOrder(r.Context(), userID, orderID)
    if err != nil {
        switch {
        case errors.Is(err, ErrNotFound):
            respondJSON(w, 404, map[string]string{"error": "order not found"})
        case errors.Is(err, ErrForbidden):
            respondJSON(w, 403, map[string]string{"error": "access denied"})
        default:
            log.Errorf("unexpected error: %v", err)
            respondJSON(w, 500, map[string]string{"error": "internal error"})
        }
        return
    }
    respondJSON(w, 200, order)
}
```

## 結構化錯誤（適合 API 回應）

```go
type AppError struct {
    Code    string `json:"code"`    // 機器可讀，如 "ORDER_NOT_FOUND"
    Message string `json:"message"` // 人可讀
    Status  int    `json:"-"`       // HTTP status code
    Err     error  `json:"-"`       // 原始錯誤（不暴露給 client）
}

func (e *AppError) Error() string { return e.Message }
func (e *AppError) Unwrap() error  { return e.Err }

// 預定義常見錯誤
var (
    ErrOrderNotFound = &AppError{Code: "ORDER_NOT_FOUND", Message: "Order not found", Status: 404}
    ErrForbidden     = &AppError{Code: "FORBIDDEN", Message: "Access denied", Status: 403}
)

func NewInternalError(err error) *AppError {
    return &AppError{
        Code:    "INTERNAL_ERROR",
        Message: "An internal error occurred",
        Status:  500,
        Err:     err, // 只記錄，不回傳給 client
    }
}

// Handler 統一處理
func handleError(w http.ResponseWriter, err error) {
    var appErr *AppError
    if errors.As(err, &appErr) {
        if appErr.Err != nil {
            log.Errorf("internal error: %v", appErr.Err)
        }
        respondJSON(w, appErr.Status, appErr)
        return
    }

    // 未知錯誤
    log.Errorf("unexpected error: %v", err)
    respondJSON(w, 500, map[string]string{"code": "INTERNAL_ERROR", "message": "Internal error"})
}
```

## 重試邏輯

```go
func withRetry(ctx context.Context, maxAttempts int, fn func() error) error {
    var lastErr error
    for attempt := 0; attempt < maxAttempts; attempt++ {
        if attempt > 0 {
            // Exponential backoff with jitter
            backoff := time.Duration(math.Pow(2, float64(attempt))) * 100 * time.Millisecond
            jitter := time.Duration(rand.Intn(100)) * time.Millisecond
            select {
            case <-ctx.Done():
                return ctx.Err()
            case <-time.After(backoff + jitter):
            }
        }

        lastErr = fn()
        if lastErr == nil {
            return nil
        }

        // 只重試可重試的錯誤
        if !isRetriable(lastErr) {
            return lastErr
        }

        log.Warnf("attempt %d failed: %v", attempt+1, lastErr)
    }
    return fmt.Errorf("after %d attempts: %w", maxAttempts, lastErr)
}

func isRetriable(err error) bool {
    // gRPC 的可重試錯誤
    if st, ok := status.FromError(err); ok {
        switch st.Code() {
        case codes.Unavailable, codes.DeadlineExceeded, codes.ResourceExhausted:
            return true
        }
    }
    // Context 取消不重試
    if errors.Is(err, context.Canceled) || errors.Is(err, context.DeadlineExceeded) {
        return false
    }
    return false
}
```

## Panic vs Error

```go
// Panic：用於程式設計錯誤（永遠不應該發生）
func mustGetConfig(key string) string {
    val := os.Getenv(key)
    if val == "" {
        panic(fmt.Sprintf("required env var %s is not set", key)) // 啟動時 fail-fast
    }
    return val
}

// Recovery：在頂層 middleware 捕捉 panic，不讓 server 崩潰
func recoveryMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        defer func() {
            if r := recover(); r != nil {
                log.Errorf("panic recovered: %v\n%s", r, debug.Stack())
                http.Error(w, "internal server error", 500)
            }
        }()
        next.ServeHTTP(w, r)
    })
}
```

## 常見錯誤（Anti-patterns）

```go
// ❌ 忽略錯誤
result, _ := doSomething()

// ❌ 只用 err != nil，不分類型
if err != nil {
    return err // 丟失上下文
}

// ❌ 用 string 比較錯誤
if err.Error() == "not found" { // 脆弱，錯誤訊息改了就壞了

// ❌ 每個 if err != nil 都 log
log.Error(err) // 重複 log

// ✅ 正確做法
if err != nil {
    return fmt.Errorf("operation context: %w", err) // 包裝上下文，讓最外層 log
}
```

## 相關頁面

- [[Go並發模式]] — context 取消的錯誤處理
- [[冪等性設計]] — 重試邏輯與冪等的配合
- [[gRPC設計與實戰]] — gRPC status code 的錯誤映射
- [[Go微服務框架比較]] — 不同框架的錯誤處理風格
