---
title: Go Wire 深度實戰
type: concept
tags: [golang, wire, dependency-injection, code-generation, provider, injector, senior]
created: 2026-04-30
updated: 2026-04-30
---

# Go Wire 深度實戰

> Wire 是 Google 開源的 Go **編譯時依賴注入**框架：透過靜態代碼分析和生成，在編譯前產生組裝代碼。無反射、無執行時容器、生成的代碼就是普通 Go 代碼。

## Wire vs 手動 DI vs uber-go/fx

```
手動 DI：main.go 手寫所有初始化，隨專案增長難以維護
Wire：靜態代碼生成，生成後就是普通 Go 代碼（可 debug、無執行時開銷）
fx：執行時容器，用反射注入，支援 Lifecycle hook，學習曲線較高

選擇建議：
- 中小型服務、簡單依賴圖 → Wire
- 大型服務、需要 Module 化組裝、複雜 lifecycle → fx
- 性能極敏感、不想引入代碼生成 → 手動 DI
```

---

## 基礎概念

### Provider（提供者）

Provider 是一個**普通 Go 函數**，接收依賴、返回組件。Wire 分析這些函數的輸入輸出來建立依賴圖。

```go
// wire.go（或 provider.go）

// Provider：接收依賴，返回組件
func NewDB(cfg *Config) (*sql.DB, func(), error) {
    db, err := sql.Open("pgx", cfg.DatabaseURL)
    if err != nil {
        return nil, nil, err
    }
    // 第二個返回值是 cleanup function（Wire 會在適當時機呼叫）
    cleanup := func() { db.Close() }
    return db, cleanup, nil
}

func NewUserRepository(db *sql.DB) *PostgresUserRepository {
    return &PostgresUserRepository{db: db}
}

func NewUserService(repo UserRepository) *UserService {
    return &UserService{repo: repo}
}

// Provider 返回介面（需要 wire.Bind 或 Interface Provider）
func NewUserService(repo UserRepository, mailer EmailSender) *UserService {
    return &UserService{repo: repo, mailer: mailer}
}
```

### Injector（注入器）

Injector 是一個帶有 `wire.Build(...)` 的**函數宣告**，Wire 根據它生成完整的初始化代碼。

```go
// wire_gen.go 會根據這個生成代碼
// +build wireinject

package main

import "github.com/google/wire"

// InitializeUserService 是 Injector：Wire 看到 wire.Build 就生成代碼
func InitializeUserService(cfg *Config) (*UserService, func(), error) {
    wire.Build(
        NewDB,
        NewUserRepository,
        NewSMTPEmailSender,
        NewUserService,
    )
    return nil, nil, nil // Wire 會替換這個 return
}
```

執行 `wire` 命令後生成 `wire_gen.go`：

```go
// wire_gen.go（自動生成，不要手動編輯）

func InitializeUserService(cfg *Config) (*UserService, func(), error) {
    db, cleanup, err := NewDB(cfg)
    if err != nil {
        return nil, nil, err
    }
    postgresUserRepository := NewUserRepository(db)
    smtpEmailSender := NewSMTPEmailSender(cfg)
    userService := NewUserService(postgresUserRepository, smtpEmailSender)
    return userService, func() {
        cleanup()
    }, nil
}
```

---

## 介面綁定（Interface Binding）

當 Provider 返回具體型別，但需要注入介面時，用 `wire.Bind`：

```go
// UserRepository 是介面，PostgresUserRepository 是具體實作

var RepositorySet = wire.NewSet(
    NewPostgresUserRepository,
    wire.Bind(new(UserRepository), new(*PostgresUserRepository)),
    // 告訴 Wire：需要 UserRepository 介面時，用 *PostgresUserRepository
)

// 或直接在 Injector 中
func InitializeApp(cfg *Config) (*App, func(), error) {
    wire.Build(
        NewDB,
        NewPostgresUserRepository,
        wire.Bind(new(UserRepository), new(*PostgresUserRepository)),
        NewPostgresOrderRepository,
        wire.Bind(new(OrderRepository), new(*PostgresOrderRepository)),
        NewUserService,
        NewOrderService,
        NewApp,
    )
    return nil, nil, nil
}
```

---

## ProviderSet（組織 Provider）

把相關 Provider 分組，方便組合：

```go
// infrastructure/wire.go
var InfrastructureSet = wire.NewSet(
    NewDB,
    NewRedisClient,
    NewKafkaEventBus,
    wire.Bind(new(EventBus), new(*KafkaEventBus)),
)

// repository/wire.go
var RepositorySet = wire.NewSet(
    NewPostgresUserRepository,
    wire.Bind(new(UserRepository), new(*PostgresUserRepository)),
    NewPostgresOrderRepository,
    wire.Bind(new(OrderRepository), new(*PostgresOrderRepository)),
)

// service/wire.go
var ServiceSet = wire.NewSet(
    NewUserService,
    NewOrderService,
    NewPricingService,
)

// handler/wire.go
var HandlerSet = wire.NewSet(
    NewUserHandler,
    NewOrderHandler,
)

// main wire.go（組合所有 Set）
func InitializeApp(cfg *Config) (*App, func(), error) {
    wire.Build(
        InfrastructureSet,
        RepositorySet,
        ServiceSet,
        HandlerSet,
        NewApp,
    )
    return nil, nil, nil
}
```

---

## Struct Provider（結構體注入）

直接用 struct 欄位作為 Provider，避免寫冗長的建構子：

```go
// 適合欄位多的 struct
type App struct {
    UserHandler  *UserHandler
    OrderHandler *OrderHandler
    Config       *Config
}

// 使用 wire.Struct 代替手寫建構子
var AppSet = wire.NewSet(
    wire.Struct(new(App), "*"), // "*" 表示所有欄位都由 Wire 注入
    // 或指定特定欄位：wire.Struct(new(App), "UserHandler", "OrderHandler")
)

// 等同於：
func NewApp(userHandler *UserHandler, orderHandler *OrderHandler, cfg *Config) *App {
    return &App{
        UserHandler:  userHandler,
        OrderHandler: orderHandler,
        Config:       cfg,
    }
}
```

---

## Value Provider（直接提供值）

提供不需要建構函數的值：

```go
func InitializeApp() (*App, func(), error) {
    wire.Build(
        // wire.Value：直接提供一個值
        wire.Value(context.Background()),

        // wire.InterfaceValue：提供介面值（適合 nil 或 mock）
        wire.InterfaceValue(new(Tracer), opentracing.GlobalTracer()),

        NewApp,
    )
    return nil, nil, nil
}
```

---

## Parameter Objects（參數物件）

當多個 Provider 需要同一個 Config 的不同欄位：

```go
// ❌ 問題：所有 Provider 都依賴整個 *Config，但只需要部分欄位
func NewRedisClient(cfg *Config) *redis.Client {
    return redis.NewClient(&redis.Options{Addr: cfg.RedisAddr})
}

// ✅ 用 wire.FieldsOf 提取特定欄位
type Config struct {
    DB    DatabaseConfig
    Redis RedisConfig
    SMTP  SMTPConfig
}

// 在 ProviderSet 中用 FieldsOf
var InfraSet = wire.NewSet(
    wire.FieldsOf(new(*Config), "DB", "Redis", "SMTP"),
    // 讓 Wire 知道可以從 *Config 中提取 DatabaseConfig、RedisConfig、SMTPConfig
    NewDB,       // 接收 DatabaseConfig
    NewRedis,    // 接收 RedisConfig
    NewSMTP,     // 接收 SMTPConfig
)

func NewDB(cfg DatabaseConfig) (*sql.DB, func(), error) { ... }
func NewRedis(cfg RedisConfig) (*redis.Client, func(), error) { ... }
```

---

## Cleanup Functions（清理函數）

Wire 處理多層的 cleanup，確保正確的反向清理順序：

```go
func NewDB(cfg *Config) (*sql.DB, func(), error) {
    db, err := sql.Open("pgx", cfg.DatabaseURL)
    if err != nil {
        return nil, nil, err
    }
    return db, func() { db.Close() }, nil
}

func NewRedisClient(cfg *Config) (*redis.Client, func(), error) {
    client := redis.NewClient(&redis.Options{Addr: cfg.RedisAddr})
    return client, func() { client.Close() }, nil
}

// Wire 生成的代碼：按依賴的反向順序呼叫 cleanup
// 類似 defer LIFO 順序
func InitializeApp(cfg *Config) (*App, func(), error) {
    db, cleanup1, err := NewDB(cfg)
    if err != nil { return nil, nil, err }

    redis, cleanup2, err := NewRedisClient(cfg)
    if err != nil {
        cleanup1() // 出錯時立即清理已建立的資源
        return nil, nil, err
    }

    app := NewApp(db, redis)
    return app, func() {
        cleanup2() // 後建立的先清理
        cleanup1()
    }, nil
}
```

---

## 完整 DDD + Wire 範例

```
order-service/
├── cmd/server/
│   └── main.go          ← wire.go 生成的 InitializeApp
├── internal/
│   ├── domain/order/
│   │   ├── entity.go
│   │   └── repository.go  ← 介面定義
│   ├── application/order/
│   │   └── service.go
│   ├── infrastructure/
│   │   ├── db.go
│   │   └── order_repo.go  ← 介面實作
│   └── interfaces/http/
│       └── handler.go
├── wire.go              ← Injector 宣告（帶 //go:build wireinject）
└── wire_gen.go          ← Wire 生成（不要手動編輯）
```

```go
// wire.go
//go:build wireinject
// +build wireinject

package main

import "github.com/google/wire"

func InitializeApp(cfg *config.Config) (*App, func(), error) {
    wire.Build(
        // Infrastructure
        infrastructure.NewDB,
        infrastructure.NewRedisClient,

        // Repositories（含介面綁定）
        infrastructure.NewPostgresOrderRepository,
        wire.Bind(new(domain.OrderRepository), new(*infrastructure.PostgresOrderRepository)),

        // Domain Services
        domain.NewPricingService,

        // Application Services
        application.NewOrderService,

        // HTTP Handlers
        http.NewOrderHandler,

        // App
        wire.Struct(new(App), "*"),
    )
    return nil, nil, nil
}
```

```go
// main.go
func main() {
    cfg := config.Load()

    app, cleanup, err := InitializeApp(cfg)
    if err != nil {
        log.Fatal(err)
    }
    defer cleanup()

    app.Run()
}
```

---

## 常用 Wire 指令

```bash
# 安裝 Wire CLI
go install github.com/google/wire/cmd/wire@latest

# 在當前目錄執行 Wire（生成 wire_gen.go）
wire

# 指定目錄
wire ./cmd/server/...

# 檢查依賴圖（不生成代碼，只檢查）
wire check ./...
```

---

## 常見錯誤與除錯

```go
// ❌ 錯誤 1：Provider 返回型別衝突（同型別有兩個 Provider）
var Set = wire.NewSet(
    NewPostgresDB,  // 返回 *sql.DB
    NewTestDB,      // 也返回 *sql.DB → Wire 不知道用哪個
)
// 修法：用不同型別區分
type WriteDB *sql.DB
type ReadDB  *sql.DB

// ❌ 錯誤 2：循環依賴
// A 依賴 B，B 依賴 A → Wire 無法生成代碼
// 修法：引入 interface 打破循環，或重新設計職責

// ❌ 錯誤 3：沒有加 build tag 導致 wire.go 和 wire_gen.go 都被編譯
// wire.go 開頭必須有：
//go:build wireinject

// ❌ 錯誤 4：忘記處理 cleanup 的 nil
// Wire 生成的 cleanup 在出錯時可能是 nil，不要直接 defer
app, cleanup, err := InitializeApp(cfg)
if err != nil {
    log.Fatal(err)
}
defer cleanup() // ✅ 只有 err == nil 才執行到這裡
```

---

## 相關頁面

- [[依賴注入與控制反轉]] — DI 原理與設計模式
- [[DDD領域驅動設計]] — Wire 組裝 DDD 各層的完整範例
- [[Go依賴注入與Wire]] — 基礎 Wire + uber-go/fx 比較
- [[Go介面設計模式]] — wire.Bind 的設計前提：以介面設計依賴
