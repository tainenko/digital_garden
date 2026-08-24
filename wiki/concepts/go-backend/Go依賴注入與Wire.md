---
title: Go 依賴注入與 Wire
type: concept
tags: [golang, dependency-injection, wire, fx, clean-architecture]
created: 2026-04-29
updated: 2026-04-29
---

# Go 依賴注入與 Wire

## 為什麼需要依賴注入

```go
// ❌ 沒有 DI：硬耦合
func NewOrderService() *OrderService {
    db := connectToDB("postgres://...")  // 在 service 內部建立依賴
    redis := connectToRedis("redis://...")
    return &OrderService{db: db, redis: redis}
}
// 問題：
// 1. 無法替換依賴（測試時無法用 mock DB）
// 2. 初始化順序、錯誤處理散落各處
// 3. 同一個 DB 連線可能被建立多次

// ✅ 有 DI：依賴從外部注入
func NewOrderService(db *sql.DB, redis *redis.Client) *OrderService {
    return &OrderService{db: db, redis: redis}
}
// 好處：
// 1. 可以注入 mock（測試友好）
// 2. 依賴的生命週期由外部管理（singleton vs 每次新建）
// 3. 單一職責：Service 只關心業務邏輯
```

## 手動 DI（小型專案）

```go
// cmd/main.go
func main() {
    // 按依賴順序手動建立
    cfg := config.Load()

    db, err := sql.Open("pgx", cfg.DatabaseURL)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    redisClient := redis.NewClient(&redis.Options{Addr: cfg.RedisAddr})
    defer redisClient.Close()

    // Repository 層
    orderRepo := repository.NewOrderRepository(db)
    userRepo := repository.NewUserRepository(db)

    // Service 層
    orderService := service.NewOrderService(orderRepo, userRepo, redisClient)
    userService := service.NewUserService(userRepo)

    // Handler 層
    orderHandler := handler.NewOrderHandler(orderService)
    userHandler := handler.NewUserHandler(userService)

    // Server
    r := gin.New()
    orderHandler.Register(r)
    userHandler.Register(r)
    r.Run(cfg.Port)
}
```

缺點：當依賴複雜時，手動排序很繁瑣，容易出錯。

## Google Wire（自動生成 DI 代碼）

Wire 是一個**代碼生成**工具，根據你的 Provider 函數，自動生成依賴注入代碼。

```bash
go install github.com/google/wire/cmd/wire@latest
```

### 定義 Providers

```go
// internal/repository/provider.go
var ProviderSet = wire.NewSet(
    NewOrderRepository,
    NewUserRepository,
)

func NewOrderRepository(db *sql.DB) *OrderRepository {
    return &OrderRepository{db: db}
}

// internal/service/provider.go
var ProviderSet = wire.NewSet(
    NewOrderService,
    NewUserService,
)

func NewOrderService(repo *OrderRepository, userRepo *UserRepository, cache *redis.Client) *OrderService {
    return &OrderService{repo: repo, userRepo: userRepo, cache: cache}
}

// internal/handler/provider.go
var ProviderSet = wire.NewSet(
    NewOrderHandler,
    NewUserHandler,
)

// cmd/order/wire.go（這個檔案是 Wire 的輸入）
//go:build wireinject

package main

import (
    "github.com/google/wire"
    "yourco/internal/handler"
    "yourco/internal/repository"
    "yourco/internal/service"
)

func InitApp(cfg *Config) (*App, func(), error) {
    wire.Build(
        // 提供基礎設施
        NewDatabase,
        NewRedis,
        // 組合各層的 Provider Set
        repository.ProviderSet,
        service.ProviderSet,
        handler.ProviderSet,
        // 最終組裝
        NewApp,
    )
    return nil, nil, nil // Wire 會填充這裡
}
```

```bash
# 執行 Wire 生成 wire_gen.go
wire ./cmd/order/
```

**Wire 生成的代碼（wire_gen.go）**：
```go
// 自動生成，不要手動修改
func InitApp(cfg *Config) (*App, func(), error) {
    db, cleanup, err := NewDatabase(cfg.DatabaseURL)
    if err != nil {
        return nil, nil, err
    }
    redisClient := NewRedis(cfg.RedisAddr)

    orderRepo := repository.NewOrderRepository(db)
    userRepo := repository.NewUserRepository(db)
    orderService := service.NewOrderService(orderRepo, userRepo, redisClient)
    userService := service.NewUserService(userRepo)
    orderHandler := handler.NewOrderHandler(orderService)
    userHandler := handler.NewUserHandler(userService)

    app := NewApp(orderHandler, userHandler)
    return app, func() { cleanup() }, nil
}
```

### Wire 處理資源清理

```go
// NewDatabase 返回 cleanup 函數
func NewDatabase(dsn string) (*sql.DB, func(), error) {
    db, err := sql.Open("pgx", dsn)
    if err != nil {
        return nil, nil, err
    }
    cleanup := func() {
        db.Close()
    }
    return db, cleanup, nil
}

// Wire 會把 cleanup 函數串聯起來
// InitApp 返回的 cleanup 會依序關閉所有資源
```

## uber-go/fx（生產環境的 DI 框架）

fx 是 Uber 開源的 DI 框架，支援 lifecycle hooks，適合大型服務：

```go
import "go.uber.org/fx"

func main() {
    app := fx.New(
        // 提供依賴
        fx.Provide(
            NewConfig,
            NewDatabase,
            NewRedis,
            repository.NewOrderRepository,
            service.NewOrderService,
            handler.NewOrderHandler,
            NewHTTPServer,
        ),

        // 啟動
        fx.Invoke(func(srv *http.Server, lc fx.Lifecycle) {
            lc.Append(fx.Hook{
                OnStart: func(ctx context.Context) error {
                    go srv.ListenAndServe()
                    return nil
                },
                OnStop: func(ctx context.Context) error {
                    return srv.Shutdown(ctx) // 優雅關機自動整合
                },
            })
        }),
    )

    app.Run() // 阻塞，處理 SIGTERM 並呼叫 OnStop
}
```

**fx 的優點**：
- Lifecycle hooks（自動處理啟動/關機順序）
- 自動解析依賴順序
- 好的錯誤訊息（告訴你哪個依賴缺失）

**缺點**：
- 執行期 DI（Wire 是編譯期，更快）
- 使用反射，panic 而非編譯錯誤

## 測試中的 DI

```go
// 使用 interface 讓測試可以注入 mock
type OrderRepository interface {
    FindByID(ctx context.Context, id string) (*Order, error)
    Create(ctx context.Context, order *Order) error
}

// 生產實作
type postgresOrderRepo struct{ db *sql.DB }
func (r *postgresOrderRepo) FindByID(...)

// 測試 mock
type mockOrderRepo struct{ mock.Mock }
func (m *mockOrderRepo) FindByID(ctx context.Context, id string) (*Order, error) {
    args := m.Called(ctx, id)
    return args.Get(0).(*Order), args.Error(1)
}

// 測試
func TestOrderService_GetOrder(t *testing.T) {
    mockRepo := new(mockOrderRepo)
    mockRepo.On("FindByID", mock.Anything, "order-123").
        Return(&Order{ID: "order-123", Status: "paid"}, nil)

    svc := NewOrderService(mockRepo) // 注入 mock
    order, err := svc.GetOrder(context.Background(), "order-123")

    assert.NoError(t, err)
    assert.Equal(t, "paid", order.Status)
    mockRepo.AssertExpectations(t)
}
```

## DI 選型建議

| 情境 | 推薦 |
|------|------|
| 小型服務（< 10 個依賴）| 手動 DI |
| 中型服務，追求編譯期安全 | Wire |
| 大型服務，需要 lifecycle 管理 | uber-go/fx |
| 使用 Kratos 框架 | Wire（Kratos 內建）|

## 相關頁面

- [[Go微服務框架比較]] — Kratos 整合 Wire
- [[Go優雅關機與健康檢查]] — fx 的 lifecycle 與優雅關機整合
- [[Go錯誤處理最佳實踐]] — DI 初始化時的錯誤處理
