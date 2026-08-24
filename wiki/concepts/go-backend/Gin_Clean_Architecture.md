---
title: Gin Clean Architecture（Go）
type: concept
tags: [Golang, Gin, CleanArchitecture, Wire, GORM, DI, 架構]
created: 2026-05-04
updated: 2026-05-04
sources: [github-go-gin-clean-arch]
---

# Gin Clean Architecture（Go）

## 核心原則

Clean Architecture（Uncle Bob）的核心是**依賴規則**：

> 依賴方向只能由外向內。內層不知道外層的存在。

```
┌──────────────────────────────────────┐
│  Handler（Delivery / API）            │ ← 最外層：HTTP 細節
│  ┌────────────────────────────────┐  │
│  │  UseCase（Business Logic）      │  │ ← 業務規則
│  │  ┌──────────────────────────┐  │  │
│  │  │  Repository（Data Access）│  │  │ ← 資料存取抽象
│  │  │  ┌────────────────────┐  │  │  │
│  │  │  │  Domain（Entities） │  │  │  │ ← 最內層：純資料結構
│  │  │  └────────────────────┘  │  │  │
│  │  └──────────────────────────┘  │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
```

---

## 目錄結構

```
go-gin-clean-arch/
├── cmd/
│   └── api/
│       ├── docs/           ← Swagger 自動生成（make swag）
│       └── main.go         ← 入口：LoadConfig → InitializeAPI → Start
├── pkg/
│   ├── domain/
│   │   └── user.go         ← Domain Entity（純 struct，含 GORM tag）
│   ├── repository/
│   │   ├── interface/
│   │   │   └── user.go     ← UserRepository interface（上層依賴此抽象）
│   │   └── user.go         ← GORM 實作
│   ├── usecase/
│   │   ├── interface/
│   │   │   └── user.go     ← UserUseCase interface
│   │   └── user.go         ← 業務邏輯實作（依賴 UserRepository interface）
│   ├── api/
│   │   ├── handler/
│   │   │   └── user.go     ← Gin Handler（依賴 UserUseCase interface）
│   │   ├── middleware/
│   │   │   └── auth.go     ← JWT Bearer 驗證 middleware
│   │   └── server.go       ← Gin engine + 路由設定
│   ├── config/
│   │   └── config.go       ← Viper 讀取 .env
│   ├── db/
│   │   └── connection.go   ← GORM 連線 + AutoMigrate
│   └── di/
│       ├── wire.go         ← Wire injector 宣告
│       └── wire_gen.go     ← make wire 自動生成
└── makefile
```

---

## 各層職責與程式碼

### 1. Domain Layer — `pkg/domain/`

**只有 struct，無邏輯、無依賴。**

```go
type Users struct {
    ID      uint   `json:"id" gorm:"unique;not null"`
    Name    string `json:"name"`
    Surname string `json:"surname"`
}
```

> ⚠️ 此 repo 在 domain struct 上直接加 GORM tag，是為了方便 AutoMigrate。嚴格 Clean Architecture 應分離 domain entity 與 ORM model。

---

### 2. Repository Layer — `pkg/repository/`

**兩個檔案：interface（抽象）+ 實作（GORM）**

**interface（讓 UseCase 依賴的抽象）：**
```go
// pkg/repository/interface/user.go
type UserRepository interface {
    FindAll(ctx context.Context) ([]domain.Users, error)
    FindByID(ctx context.Context, id uint) (domain.Users, error)
    Save(ctx context.Context, user domain.Users) (domain.Users, error)
    Delete(ctx context.Context, user domain.Users) error
}
```

**實作（constructor 回傳 interface，隱藏具體型別）：**
```go
// pkg/repository/user.go
type userDatabase struct{ DB *gorm.DB }

func NewUserRepository(DB *gorm.DB) interfaces.UserRepository {
    return &userDatabase{DB}   // ← 回傳 interface，不是 *userDatabase
}
```

---

### 3. UseCase Layer — `pkg/usecase/`

**業務邏輯。依賴 Repository interface，不知道 GORM 存在。**

```go
type userUseCase struct {
    userRepo interfaces.UserRepository   // ← 只知道 interface
}

func NewUserUseCase(repo interfaces.UserRepository) services.UserUseCase {
    return &userUseCase{userRepo: repo}
}
```

---

### 4. Handler Layer — `pkg/api/handler/`

**HTTP 細節。依賴 UseCase interface。使用 `copier` 做 DTO 轉換。**

```go
type Response struct {
    ID      uint   `copier:"must"`
    Name    string `copier:"must"`
    Surname string `copier:"must"`
}

func (cr *UserHandler) FindAll(c *gin.Context) {
    users, err := cr.userUseCase.FindAll(c.Request.Context())
    if err != nil {
        c.AbortWithStatus(http.StatusNotFound)
        return
    }
    response := []Response{}
    copier.Copy(&response, &users)   // ← domain → DTO
    c.JSON(http.StatusOK, response)
}
```

---

### 5. Server & Routing — `pkg/api/server.go`

```go
func NewServerHTTP(userHandler *handler.UserHandler) *ServerHTTP {
    engine := gin.New()
    engine.Use(gin.Logger())
    engine.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerfiles.Handler))
    engine.POST("/login", middleware.LoginHandler)

    api := engine.Group("/api", middleware.AuthorizationMiddleware)
    api.GET("users", userHandler.FindAll)
    api.GET("users/:id", userHandler.FindByID)
    api.POST("users", userHandler.Save)
    api.DELETE("users/:id", userHandler.Delete)

    return &ServerHTTP{engine: engine}
}
```

---

### 6. Dependency Injection — `pkg/di/wire.go`

Wire 宣告 provider chain，`make wire` 自動生成 `wire_gen.go`。

```go
//go:build wireinject

func InitializeAPI(cfg config.Config) (*http.ServerHTTP, error) {
    wire.Build(
        db.ConnectDatabase,          // Config → *gorm.DB
        repository.NewUserRepository, // *gorm.DB → UserRepository
        usecase.NewUserUseCase,       // UserRepository → UserUseCase
        handler.NewUserHandler,       // UserUseCase → *UserHandler
        http.NewServerHTTP,           // *UserHandler → *ServerHTTP
    )
    return &http.ServerHTTP{}, nil
}
```

---

## 新增一個 Domain 的完整流程

以新增 `Product` 為例，需依序建立 **7 個檔案**：

```
Step 1: pkg/domain/product.go            ← 定義 struct
Step 2: pkg/repository/interface/product.go  ← 定義 Repository interface
Step 3: pkg/repository/product.go        ← GORM 實作
Step 4: pkg/usecase/interface/product.go ← 定義 UseCase interface
Step 5: pkg/usecase/product.go           ← 業務邏輯實作
Step 6: pkg/api/handler/product.go       ← Gin Handler + Swagger 註解
Step 7: pkg/api/server.go（更新）        ← 新增路由 + 注入 ProductHandler
Step 8: pkg/di/wire.go（更新）           ← 加入新 provider
       → make wire                       ← 重新生成 wire_gen.go
```

---

## 使用 OpenSpec / AI 工具設計架構

Clean Architecture 的層級結構很適合用 [[OpenSpec工作流]] 進行規格驅動開發：

### 1. Explore（探索現有結構）
```
讓 AI 讀取現有 domain/repository/usecase/handler 各層，
確認命名規範與 constructor 回傳 interface 的模式。
```

### 2. Proposal（提出新功能）
```
提出需求：「新增 Order 模組，支援 CRUD + 按 userID 查詢 + 狀態篩選」
讓 AI 列出要建立的 7 個檔案與各自職責。
```

### 3. Behavioral Specs（行為規格）
```go
// UseCase specs
FindByUserID(ctx, userID) → []Order, error
// - userID 不存在時回傳空 slice，非錯誤
// - 按 createdAt DESC 排序

UpdateStatus(ctx, id, status) → Order, error
// - 狀態只允許: pending → paid → shipped → delivered
// - 非法狀態轉換回傳 ErrInvalidTransition
```

### 4. Tasks（實作清單）
```
[ ] pkg/domain/order.go
[ ] pkg/repository/interface/order.go（5 個方法）
[ ] pkg/repository/order.go（GORM 實作）
[ ] pkg/usecase/interface/order.go
[ ] pkg/usecase/order.go（含狀態機驗證）
[ ] pkg/api/handler/order.go（含 Swagger godoc）
[ ] 更新 server.go + wire.go
[ ] make wire
```

### Claude Code 實際 Prompt 範例

```
分析 pkg/domain/user.go, pkg/repository/interface/user.go,
pkg/usecase/interface/user.go, pkg/api/handler/user.go 的結構，
然後用完全相同的模式為 Order domain 生成對應的 7 個檔案。
Order 包含 fields: ID, UserID, Amount, Status, CreatedAt
UseCase 方法: FindAll, FindByID, FindByUserID, Save, UpdateStatus, Delete
```

---

## 架構優缺點

| 優點 | 缺點 |
|------|------|
| 層級清晰，職責單一 | 單一 Domain 需建 7 個檔案（boilerplate 多）|
| UseCase 可 100% 單元測試（mock Repository） | Wire 學習曲線（需理解 provider graph） |
| 可輕易換掉 DB（只改 Repository 實作） | 小型專案過度設計 |
| Swagger 自動生成 | GORM tag 在 domain struct 破壞純粹性 |

---

## 相關頁面

- [[Go Wire深度實戰]] — Wire provider/injector 詳細說明
- [[DDD領域驅動設計]] — Clean Architecture 的理論基礎
- [[Go介面設計模式]] — Repository / UseCase interface 模式
- [[OpenSpec工作流]] — 用 AI 工具設計此類架構的 8 步流程
- [[gRPC設計與實戰]] — 若需取代 Gin HTTP 層
