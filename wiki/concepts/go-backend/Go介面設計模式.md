---
title: Go 介面設計模式
type: concept
tags: [golang, interfaces, design-patterns, clean-architecture]
created: 2026-04-29
updated: 2026-04-29
---

# Go 介面設計模式

## Go 介面的獨特性

Go 的介面是**隱式實作（implicit implementation）**：型別不需要宣告「我實作了這個介面」，只要有對應的方法就自動滿足。

```go
// 定義介面
type Stringer interface {
    String() string
}

// 任何有 String() 方法的型別都自動實作了 Stringer
type User struct{ Name string }
func (u User) String() string { return u.Name }

// 不需要 class User implements Stringer（Java 風格）
var s Stringer = User{Name: "Alice"} // 自動滿足
```

## 介面的黃金原則

### 小介面（Interface Segregation）

```go
// ❌ 大介面：使用者被迫實作不需要的方法
type DataStore interface {
    Find(id string) (*Record, error)
    FindAll() ([]*Record, error)
    Create(r *Record) error
    Update(r *Record) error
    Delete(id string) error
    Search(query string) ([]*Record, error)
    Count() (int, error)
}

// ✅ 小介面：按使用場景切
type Finder interface {
    Find(id string) (*Record, error)
}

type Creator interface {
    Create(r *Record) error
}

type Updater interface {
    Update(r *Record) error
}

// 需要多個能力時組合
type ReadWriter interface {
    Finder
    Creator
    Updater
}
```

**Go 諺語**：「The bigger the interface, the weaker the abstraction.」

### 在使用者端定義介面（不在實作端）

```go
// ❌ 錯誤：在 package database 定義介面，使用者依賴整個 package
// database/repository.go
type Repository interface {
    FindUser(id string) (*User, error)
    // ...很多方法
}

// ✅ 正確：在使用者端（service）定義只需要的介面
// service/order.go
type userFinder interface { // 小寫：package 私有
    FindUser(ctx context.Context, id string) (*User, error)
}

type OrderService struct {
    users userFinder // 依賴的是介面，不是具體實作
}
```

## 常見設計模式

### Repository Pattern

```go
// 定義介面（在 domain/service 層）
type OrderRepository interface {
    FindByID(ctx context.Context, id string) (*Order, error)
    FindByUserID(ctx context.Context, userID string, page, limit int) ([]*Order, int, error)
    Create(ctx context.Context, order *Order) error
    UpdateStatus(ctx context.Context, id string, status OrderStatus) error
}

// 生產實作（在 infrastructure 層）
type postgresOrderRepository struct {
    db *pgxpool.Pool
}

func NewPostgresOrderRepository(db *pgxpool.Pool) OrderRepository {
    return &postgresOrderRepository{db: db}
}

func (r *postgresOrderRepository) FindByID(ctx context.Context, id string) (*Order, error) {
    // 實際 SQL 查詢
}

// 測試實作（in-memory）
type inMemoryOrderRepository struct {
    mu     sync.RWMutex
    orders map[string]*Order
}

func NewInMemoryOrderRepository() OrderRepository {
    return &inMemoryOrderRepository{
        orders: make(map[string]*Order),
    }
}

func (r *inMemoryOrderRepository) FindByID(ctx context.Context, id string) (*Order, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()
    order, ok := r.orders[id]
    if !ok {
        return nil, ErrNotFound
    }
    return order, nil
}
```

### Middleware / Decorator Pattern

```go
// 核心介面
type OrderService interface {
    CreateOrder(ctx context.Context, req *CreateOrderRequest) (*Order, error)
    GetOrder(ctx context.Context, id string) (*Order, error)
}

// 裝飾器：Logging
type loggingOrderService struct {
    next   OrderService
    logger *zap.Logger
}

func WithLogging(svc OrderService, logger *zap.Logger) OrderService {
    return &loggingOrderService{next: svc, logger: logger}
}

func (s *loggingOrderService) CreateOrder(ctx context.Context, req *CreateOrderRequest) (*Order, error) {
    start := time.Now()
    order, err := s.next.CreateOrder(ctx, req)
    s.logger.Info("CreateOrder",
        zap.String("user_id", req.UserID),
        zap.Duration("duration", time.Since(start)),
        zap.Error(err),
    )
    return order, err
}

// 裝飾器：Metrics
type metricsOrderService struct {
    next    OrderService
    counter metric.Int64Counter
}

func WithMetrics(svc OrderService, counter metric.Int64Counter) OrderService {
    return &metricsOrderService{next: svc, counter: counter}
}

// 組合多個裝飾器（洋蔥圈結構）
func NewOrderServiceWithMiddleware(svc OrderService, logger *zap.Logger, counter metric.Int64Counter) OrderService {
    svc = WithMetrics(svc, counter)
    svc = WithLogging(svc, logger)  // 最外層
    return svc
}
```

### Functional Options Pattern

```go
// 適合有很多可選參數的結構
type HTTPClient struct {
    timeout     time.Duration
    maxRetries  int
    baseURL     string
    userAgent   string
    interceptors []func(*http.Request) *http.Request
}

type Option func(*HTTPClient)

func WithTimeout(d time.Duration) Option {
    return func(c *HTTPClient) { c.timeout = d }
}

func WithMaxRetries(n int) Option {
    return func(c *HTTPClient) { c.maxRetries = n }
}

func WithUserAgent(ua string) Option {
    return func(c *HTTPClient) { c.userAgent = ua }
}

func NewHTTPClient(baseURL string, opts ...Option) *HTTPClient {
    c := &HTTPClient{
        timeout:    30 * time.Second,
        maxRetries: 3,
        baseURL:    baseURL,
        userAgent:  "my-service/1.0",
    }
    for _, opt := range opts {
        opt(c)
    }
    return c
}

// 使用
client := NewHTTPClient("https://api.example.com",
    WithTimeout(5*time.Second),
    WithMaxRetries(5),
    WithUserAgent("order-service/2.0"),
)
```

### Strategy Pattern

```go
// 不同的付款方式
type PaymentStrategy interface {
    Charge(ctx context.Context, amount int64, metadata map[string]string) (string, error)
    Refund(ctx context.Context, transactionID string, amount int64) error
}

type StripeStrategy struct{ client *stripe.Client }
type GreenWorldStrategy struct{ client *greenworld.Client }
type CryptoStrategy struct{ wallet string }

// 實作各個 Strategy
func (s *StripeStrategy) Charge(ctx context.Context, amount int64, metadata map[string]string) (string, error) {
    // Stripe 扣款邏輯
}

// 使用 Strategy
type PaymentService struct {
    strategies map[string]PaymentStrategy
}

func (s *PaymentService) ProcessPayment(ctx context.Context, method string, amount int64) (string, error) {
    strategy, ok := s.strategies[method]
    if !ok {
        return "", fmt.Errorf("unsupported payment method: %s", method)
    }
    return strategy.Charge(ctx, amount, nil)
}
```

## 介面的型別斷言

```go
// 型別斷言：在 runtime 確認介面的具體型別
func processError(err error) {
    // 方式一：單一型別斷言（panic if wrong type）
    valErr := err.(*ValidationError) // 不推薦，可能 panic

    // 方式二：安全斷言（推薦）
    if valErr, ok := err.(*ValidationError); ok {
        // 處理 ValidationError
        _ = valErr
    }

    // 方式三：errors.As（可穿透 error wrapping，推薦）
    var valErr *ValidationError
    if errors.As(err, &valErr) {
        // 即使 err 是 fmt.Errorf("...: %w", valErr) 也能找到
    }

    // 方式四：type switch
    switch e := err.(type) {
    case *ValidationError:
        handleValidation(e)
    case *NetworkError:
        handleNetwork(e)
    default:
        handleUnknown(err)
    }
}
```

## 空介面與 any

```go
// any = interface{}，接受任何型別
// 謹慎使用：失去型別安全
func processAny(val any) {
    // 必須斷言才能使用
    switch v := val.(type) {
    case string:
        fmt.Println("string:", v)
    case int:
        fmt.Println("int:", v)
    case []byte:
        fmt.Println("bytes:", v)
    }
}

// Go 1.18+ 泛型：比 any 更安全的替代方案
func Map[T, R any](slice []T, fn func(T) R) []R {
    result := make([]R, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// 使用
names := Map(users, func(u User) string { return u.Name })
```

## 相關頁面

- [[Go依賴注入與Wire]] — 介面是 DI 的基礎
- [[Go並發模式]] — channel 是 Go 的另一種多型機制
- [[Go錯誤處理最佳實踐]] — error 本身是介面
- [[Go微服務框架比較]] — go-kit 大量使用介面和 middleware 模式
