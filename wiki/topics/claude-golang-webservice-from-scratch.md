---
title: 用 Claude 從零開發 Golang Web Service 完整教程
type: topic
tags: [Golang, Gin, GORM, CleanArchitecture, ClaudeCode, Vibe Coding, 教程]
created: 2026-05-06
updated: 2026-05-06
sources: [github-go-gin-clean-arch, openspec-superpowers-multi-source, schluntz-vibe-coding-production-4rules]
---

# 用 Claude 從零開發 Golang Web Service 完整教程

> 本教程示範如何用 Claude Code 作為開發夥伴，從架構設計到實際 API 交付，
> 以 **Go + Gin + GORM + PostgreSQL** 為技術棧，完成一個具備 Clean Architecture 的 Web Service。

---

## 技術棧

| 層級 | 選用 |
|------|------|
| 語言 | Go 1.22+ |
| HTTP 框架 | Gin |
| ORM | GORM + PostgreSQL |
| 依賴注入 | Wire（中後期引入）|
| 設定管理 | Viper + `.env` |
| 認證 | JWT |
| 測試 | testify + testcontainers |

---

## 全局流程概覽

```
Step 1  本機環境確認
Step 2  讓 Claude 設計架構（對話式）
Step 3  建立 CLAUDE.md 護欄
Step 4  Claude 一鍵 scaffold 骨架
Step 5  逐層實作 Domain → Repository → UseCase → Handler
Step 6  用 Spec 讓 Claude 開發具體 API（POST /users 實例）
Step 7  整合測試 + CI 設定
```

---

## Step 1｜環境確認

```bash
go version      # >= 1.22
psql --version  # PostgreSQL 用戶端（本地開發用 Docker 亦可）
which air       # 熱重載（go install github.com/air-verse/air@latest）

# 建立專案目錄
mkdir myapp && cd myapp
git init
go mod init github.com/yourname/myapp
```

---

## Step 2｜讓 Claude 設計架構

在 Claude Code 中用對話式方式確定架構。這是最重要的一步：**在寫任何程式碼之前，先讓 Claude 理解你的意圖並提出設計**。

### 2-A：需求描述

```
我要從零開始建立一個 Go Web Service，技術棧如下：
- Go 1.22 + Gin + GORM + PostgreSQL
- 需要支援 JWT 認證
- 初期功能：使用者管理（CRUD）+ 文章管理（CRUD）

請為我設計：
1. 專案目錄結構（要能支援未來擴充）
2. 依賴關係圖（哪層可以引用哪層）
3. 核心 interface 設計（Repository / UseCase）
4. main.go 的啟動順序

要求：Clean Architecture，UseCase 可以被 mock 單元測試。
```

### 2-B：Claude 的輸出（預期樣貌）

Claude 應該輸出類似這樣的目錄結構：

```
myapp/
├── cmd/
│   └── api/
│       └── main.go              ← 入口點，只做 wire + 啟動
├── pkg/
│   ├── config/
│   │   └── config.go            ← Viper 設定
│   ├── domain/
│   │   ├── user.go              ← User struct（純資料）
│   │   └── post.go
│   ├── repository/
│   │   ├── interface/
│   │   │   ├── user.go          ← UserRepository interface
│   │   │   └── post.go
│   │   ├── user.go              ← GORM 實作
│   │   └── post.go
│   ├── usecase/
│   │   ├── interface/
│   │   │   ├── user.go          ← UserUseCase interface
│   │   │   └── post.go
│   │   ├── user.go              ← 業務邏輯實作
│   │   └── post.go
│   ├── api/
│   │   ├── handler/
│   │   │   ├── user.go          ← HTTP handler
│   │   │   └── post.go
│   │   ├── middleware/
│   │   │   └── auth.go          ← JWT middleware
│   │   └── route/
│   │       └── route.go         ← 路由定義
│   └── di/
│       ├── wire.go              ← Wire provider 宣告
│       └── wire_gen.go          ← go generate 產生
├── .env.example
├── CLAUDE.md                    ← AI 工作規則（Step 3 建立）
├── Makefile
└── docker-compose.yml
```

### 2-C：確認依賴方向

追問 Claude 確認邊界：

```
請用箭頭圖說明依賴方向，特別是：
1. Handler 可以 import UseCase interface 嗎？
2. UseCase 可以 import GORM 嗎？
3. Domain 可以 import 任何東西嗎？
```

正確答案：

```
Handler → UseCase interface → Repository interface → GORM（只在 repository impl 層）
Domain  → 不 import 任何 pkg/* 內部套件
```

---

## Step 3｜建立 CLAUDE.md 護欄

在讓 Claude 寫任何程式碼之前，先建立 `CLAUDE.md`。這個檔案告訴 Claude 在這個專案裡應該遵守什麼規則。

```bash
# 讓 Claude 幫你生成 CLAUDE.md
```

**提示詞**：

```
請為這個 Go Gin 專案生成一份 CLAUDE.md，內容要包含：
1. 層級規則：哪一層可以引用哪一層（用禁止句式說明）
2. 命名規則：struct/interface/method 的命名慣例
3. 錯誤處理規則：各層如何處理 error（不是 panic）
4. 測試規則：每個 UseCase 必須有 mock test，Repository 必須有整合測試
5. 高風險區域標記：domain/ 改動影響最大，修改前必須確認 interface 相容性

用禁止句式（"絕對不可以"、"禁止"）來表達規則，讓規則更明確。
```

**CLAUDE.md 範本**（讓 Claude 生成後確認內容）：

```markdown
# CLAUDE.md — myapp 專案 AI 工作規則

## 架構層級

4 層結構（由外到內）：Handler → UseCase → Repository → Domain

### 禁止事項（違反者立即中止）

- **禁止** Handler 直接 import `gorm.io` 或 `pkg/repository/`
- **禁止** UseCase import `gin` 或任何 HTTP 相關套件
- **禁止** Domain import `pkg/` 底下的任何內部套件
- **禁止** 在 Repository 實作業務邏輯（條件判斷、計算）
- **禁止** Handler 直接回傳 domain struct（必須透過 Response DTO）

## 命名規則

- Repository interface：`UserRepository`（動詞無 -er 複數）
- UseCase interface：`UserUseCase`
- Handler：`UserHandler`（持有 UseCase interface，不是 struct）
- 每個 method 第一個參數必須是 `ctx context.Context`

## 錯誤處理

- Domain error 定義在 `pkg/domain/errors.go`（如 `ErrUserNotFound`）
- Repository 回傳原始 DB error 或 Domain error，不做 HTTP status 判斷
- UseCase 將 Repository error 轉換為 Domain error
- Handler 將 Domain error 對應為 HTTP status code（統一在 handler 層做）

## 測試規則

- UseCase 必須有 unit test（用 testify/mock，不需要 DB）
- Repository 必須有整合測試（用 testcontainers-go）
- Handler 可以用 httptest 做 E2E 測試（mock UseCase）

## ⚠️ 高風險區域

- `pkg/domain/` — 修改任何 struct 欄位前，確認 Repository interface 相容性
- `pkg/di/wire.go` — 增加 provider 後需要重跑 `make wire`
```

---

## Step 4｜Claude 一鍵 Scaffold 骨架

有了 CLAUDE.md 護欄，現在讓 Claude 生成完整骨架。

**提示詞**：

```
請依照 CLAUDE.md 的規則和我們討論的目錄結構，
幫我生成以下骨架檔案（只要 struct 定義、interface 定義、空的 method 簽名，不需要實作）：

1. pkg/domain/user.go
2. pkg/domain/errors.go
3. pkg/repository/interface/user.go
4. pkg/usecase/interface/user.go
5. pkg/api/handler/user.go（空的 handler，只有 struct + constructor）
6. pkg/config/config.go（Viper 載入 .env）
7. cmd/api/main.go（只有啟動骨架，先不用 Wire）
8. docker-compose.yml（PostgreSQL）
9. Makefile（含 run/test/wire 指令）
10. .env.example

每個檔案先只生成骨架，讓我確認後再實作細節。
```

**為什麼先要骨架、不要實作**：
- 先確認目錄結構和 interface 設計正確
- 避免 Claude 在你還沒確認前就做出一堆你不要的決定
- 骨架錯誤比實作錯誤容易發現和修正

---

## Step 5｜逐層實作

骨架確認後，**按照 Domain → Repository → UseCase → Handler 的順序**，讓 Claude 逐層實作。

### 5-A：實作 Domain

```
請實作 pkg/domain/user.go。

User 需要的欄位：
- ID (uint, primary key, auto increment)
- Name (string, 必填, max 100)
- Email (string, 必填, unique)
- Password (string, 儲存 bcrypt hash)
- CreatedAt / UpdatedAt (time.Time, GORM autoCreateTime)

另外在 pkg/domain/errors.go 定義：
- ErrUserNotFound
- ErrEmailAlreadyExists
- ErrInvalidCredentials

GORM tag 直接加在 domain struct 上（pragmatic 做法）。
```

### 5-B：實作 Repository

```
請實作 pkg/repository/user.go，
實作 pkg/repository/interface/user.go 中定義的 UserRepository interface。

規則：
- constructor 回傳 interface，簽名：func NewUserRepository(db *gorm.DB) interfaces.UserRepository
- FindByID 找不到時回傳 domain.ErrUserNotFound
- FindByEmail 找不到時回傳 domain.ErrUserNotFound
- Create 遇到 unique constraint 時回傳 domain.ErrEmailAlreadyExists
- 所有方法接受 ctx context.Context

同時生成 pkg/repository/user_test.go（整合測試，用 testcontainers-go）：
測試案例：
- Create 成功
- Create email 重複 → ErrEmailAlreadyExists
- FindByID 存在 → 回傳 user
- FindByID 不存在 → ErrUserNotFound
```

### 5-C：實作 UseCase

```
請實作 pkg/usecase/user.go，
實作 pkg/usecase/interface/user.go 中的 UserUseCase interface。

依賴：interfaces.UserRepository（透過 constructor 注入）

方法：
- Register(ctx, name, email, rawPassword) (domain.User, error)
  → 用 bcrypt hash password，呼叫 repo.Create
  → email 重複時回傳 domain.ErrEmailAlreadyExists
- GetByID(ctx, id) (domain.User, error)
- Login(ctx, email, rawPassword) (tokenString string, error)
  → 驗證 email/password，成功回傳 JWT token
  → 失敗回傳 domain.ErrInvalidCredentials

同時生成 pkg/usecase/user_test.go（unit test，用 testify/mock）：
mock UserRepository，不需要 DB。
測試案例對應上面每個 method 的成功 + 失敗路徑。
```

---

## Step 6｜用 Spec 讓 Claude 開發具體 API

這是核心環節：**如何用規格讓 Claude 開發一隻完整 API**。

以 `POST /api/users` 為例，完整示範整個 Spec → 實作流程。

---

### 6-A：撰寫 API Spec

先讓 Claude 幫你生成 spec，或自己手寫。

**提示詞**（讓 Claude 生成 spec）：

```
請為 POST /api/users（建立新用戶）生成一份詳細的 API Spec，格式如下：

1. Endpoint 基本資訊
2. Request schema（JSON body 欄位、型別、驗證規則）
3. Response schema（成功/各種失敗情境）
4. Handler → UseCase → Repository 各層的 Behavioral Spec（Given/When/Then）
5. 需要新增的測試案例清單
```

**生成的 Spec 範例**：

```markdown
# POST /api/users — 建立新用戶

## Endpoint

POST /api/users
Content-Type: application/json
Authorization: 不需要（公開端點）

## Request Body

{
  "name":     string, 必填, 1–100 字元
  "email":    string, 必填, valid email format
  "password": string, 必填, 8–72 字元
}

## Response

### 成功 201
{
  "id":         uint
  "name":       string
  "email":      string
  "created_at": string (RFC3339)
}
注意：response 不含 password 欄位

### 失敗情境

| 情況 | Status | Body |
|------|--------|------|
| 缺少必填欄位 | 400 | {"error": "name is required"} |
| email 格式錯誤 | 400 | {"error": "invalid email format"} |
| password 太短 | 400 | {"error": "password must be at least 8 characters"} |
| email 已被使用 | 409 | {"error": "email already exists"} |
| 伺服器錯誤 | 500 | {"error": "internal server error"} |

## Behavioral Specs

### Handler 層

[POST /api/users - 成功路徑]
Given: 合法的 JSON body {name, email, password}
When: POST /api/users
Then: 呼叫 userUseCase.Register，status 201，body 含 id/name/email/created_at，不含 password

[POST /api/users - 驗證失敗]
Given: body 缺少 email
When: POST /api/users
Then: 不呼叫 UseCase，status 400，body {"error": "email is required"}

[POST /api/users - email 衝突]
Given: UseCase.Register 回傳 domain.ErrEmailAlreadyExists
When: POST /api/users
Then: status 409，body {"error": "email already exists"}

### UseCase 層（在 user_test.go 中）

[UserUseCase.Register - 成功]
Given: repo.FindByEmail 回傳 ErrUserNotFound（email 不存在）
When: Register(ctx, "Alice", "alice@example.com", "password123")
Then: repo.Create 被呼叫一次，password 欄位為 bcrypt hash，回傳帶 ID 的 user

[UserUseCase.Register - email 重複]
Given: repo.Create 回傳 domain.ErrEmailAlreadyExists
When: Register(ctx, ...)
Then: 回傳 domain.ErrEmailAlreadyExists，repo.Create 被呼叫一次

### Repository 層（在 user_repository_test.go 中）

[UserRepository.Create - 成功]
Given: DB 中無相同 email
When: Create(ctx, user)
Then: DB 中新增一筆紀錄，回傳含 auto-assign ID 的 user

[UserRepository.Create - unique violation]
Given: DB 中已有相同 email
When: Create(ctx, user)
Then: 回傳 domain.ErrEmailAlreadyExists

## 新增檔案清單

- pkg/api/handler/user.go → 新增 CreateUser method
- pkg/api/handler/user_test.go → 新增 TestCreateUser
- （UseCase / Repository 已在 Step 5 實作，只新增測試案例）
```

---

### 6-B：把 Spec 交給 Claude 實作

**提示詞**：

```
請根據以下 Spec，實作 POST /api/users 這隻 API：

[貼上上面的完整 Spec]

需要你生成：

1. pkg/api/handler/user.go 中的 CreateUser handler method
   - 用 binding:"required,email" 做 Gin validation
   - 定義 CreateUserRequest 和 CreateUserResponse struct
   - 錯誤對應：domain.ErrEmailAlreadyExists → 409，其他 error → 500

2. pkg/api/handler/user_test.go
   - mock UserUseCase interface（用 testify/mock）
   - 覆蓋 Spec 中所有 Handler 層的 Given/When/Then

3. 更新 pkg/api/route/route.go
   - 新增 POST /api/users 路由

遵守 CLAUDE.md 的規則：Handler 不 import gorm，Response struct 不含 password。
```

---

### 6-C：Claude 的輸出（預期樣貌）

Claude 應該輸出類似：

```go
// pkg/api/handler/user.go

type CreateUserRequest struct {
    Name     string `json:"name"     binding:"required,min=1,max=100"`
    Email    string `json:"email"    binding:"required,email"`
    Password string `json:"password" binding:"required,min=8,max=72"`
}

type CreateUserResponse struct {
    ID        uint      `json:"id"`
    Name      string    `json:"name"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
}

func (h *UserHandler) CreateUser(c *gin.Context) {
    var req CreateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    user, err := h.userUseCase.Register(c.Request.Context(), req.Name, req.Email, req.Password)
    if err != nil {
        switch {
        case errors.Is(err, domain.ErrEmailAlreadyExists):
            c.JSON(http.StatusConflict, gin.H{"error": "email already exists"})
        default:
            c.JSON(http.StatusInternalServerError, gin.H{"error": "internal server error"})
        }
        return
    }

    c.JSON(http.StatusCreated, CreateUserResponse{
        ID:        user.ID,
        Name:      user.Name,
        Email:     user.Email,
        CreatedAt: user.CreatedAt,
    })
}
```

```go
// pkg/api/handler/user_test.go

func TestCreateUser_Success(t *testing.T) {
    mockUC := new(mocks.UserUseCase)
    handler := NewUserHandler(mockUC)

    mockUC.On("Register", mock.Anything, "Alice", "alice@example.com", "password123").
        Return(domain.User{ID: 1, Name: "Alice", Email: "alice@example.com"}, nil)

    body := `{"name":"Alice","email":"alice@example.com","password":"password123"}`
    req := httptest.NewRequest(http.MethodPost, "/api/users", strings.NewReader(body))
    req.Header.Set("Content-Type", "application/json")
    w := httptest.NewRecorder()

    router := gin.New()
    router.POST("/api/users", handler.CreateUser)
    router.ServeHTTP(w, req)

    assert.Equal(t, http.StatusCreated, w.Code)
    var resp map[string]interface{}
    json.Unmarshal(w.Body.Bytes(), &resp)
    assert.Equal(t, float64(1), resp["id"])
    assert.Nil(t, resp["password"])  // 確認 password 不在 response 中
}

func TestCreateUser_EmailConflict(t *testing.T) {
    mockUC := new(mocks.UserUseCase)
    handler := NewUserHandler(mockUC)

    mockUC.On("Register", mock.Anything, mock.Anything, mock.Anything, mock.Anything).
        Return(domain.User{}, domain.ErrEmailAlreadyExists)

    body := `{"name":"Alice","email":"alice@example.com","password":"password123"}`
    req := httptest.NewRequest(http.MethodPost, "/api/users", strings.NewReader(body))
    req.Header.Set("Content-Type", "application/json")
    w := httptest.NewRecorder()

    router := gin.New()
    router.POST("/api/users", handler.CreateUser)
    router.ServeHTTP(w, req)

    assert.Equal(t, http.StatusConflict, w.Code)
}
```

---

### 6-D：驗證 Claude 的輸出

Claude 完成後，**你自己跑以下指令驗證**，不要只看 Claude 說沒問題：

```bash
# 確認編譯通過
go build ./...

# 確認 Handler 沒有 import gorm（架構邊界）
grep -r "gorm.io" pkg/api/handler/   # 應無輸出

# 跑 Handler 單元測試
go test ./pkg/api/handler/... -v -run TestCreateUser

# 跑 UseCase 單元測試
go test ./pkg/usecase/... -v

# 跑 Repository 整合測試（需要 Docker）
go test ./pkg/repository/... -v -tags=integration
```

---

## Step 7｜用同樣模式擴充更多 API

有了 `POST /api/users` 的完整流程，其他 API 只是重複同樣的 Spec → Apply 循環。

**快速擴充模式**（每隻 API 的標準提示詞）：

```
請依照以下規格實作 [METHOD] [PATH]：

## Spec

Endpoint: [METHOD] [PATH]
Auth: [需要 / 不需要 JWT]

Request:
[欄位清單]

Response:
[成功 / 失敗情境]

Behavioral Specs:
[Given/When/Then]

## 要求

1. 在 handler/[resource].go 新增 [MethodName] method
2. 在 handler/[resource]_test.go 新增對應測試
3. 更新 route/route.go
4. 遵守 CLAUDE.md 所有規則
5. 不要改動任何其他已存在的 method
```

**一個 Sprint 可以這樣組織**：

```
Sprint 1：User 基礎
- POST   /api/users          建立用戶
- POST   /api/users/login    登入，取得 JWT

Sprint 2：User 管理（需要 JWT）
- GET    /api/users/:id      取得單一用戶
- PATCH  /api/users/:id      更新用戶資料
- DELETE /api/users/:id      刪除用戶

Sprint 3：Post 功能
- POST   /api/posts          建立文章
- GET    /api/posts          列表（支援分頁）
- GET    /api/posts/:id      取得單篇
```

---

## 常見問題 & 提示詞修正

### Claude 把業務邏輯寫進 Repository

```
你在 repository/user.go 中加入了業務邏輯。
請移除 [具體說明是哪段]，改為：
- Repository 只做 DB 操作，回傳 DB 結果
- 業務邏輯（[說明是哪個判斷]）移到 usecase/user.go
```

### Claude 在 Handler 直接 import gorm

```
handler/user.go 違反了 CLAUDE.md 的規則，直接 import gorm.io。
請修正：Handler 只能依賴 usecase interface，
把 DB 相關的錯誤處理移到 repository 層，domain error 定義在 pkg/domain/errors.go。
```

### Claude 生成的 Response 暴露了 password 欄位

```
CreateUserResponse 不應該有 password 欄位。
請確認 Response struct 只有 id, name, email, created_at。
另外加一個測試確認 response body 的 JSON 中沒有 "password" key。
```

### Claude 生成了 Spec 以外的 method

```
你新增了 [method名] 這個 method，不在這次的 Spec 範圍內。
請刪除它，只實作 Spec 中明確列出的功能。
```

---

## 進階：引入 Wire 做自動 DI

當 provider 超過 5 個時，引入 Wire：

```
目前 cmd/api/main.go 有以下手寫的 constructor chain：
[貼上現有 main.go 的 DI 部分]

請幫我：
1. 建立 pkg/di/wire.go，把所有 provider 轉成 Wire 的 ProviderSet
2. 建立 pkg/di/injector.go，定義 InitializeApp() *gin.Engine
3. 更新 Makefile 加入 make wire 指令（go generate ./pkg/di/）
4. 更新 cmd/api/main.go 改用 InitializeApp()

不要改動 pkg/ 底下的任何業務程式碼，只動 DI 相關檔案。
```

---

## 完整指令備忘錄

```bash
# 啟動開發環境
docker-compose up -d     # 啟動 PostgreSQL
make run                 # 啟動 API server（air 熱重載）

# 測試
make test                # 所有 unit test
make test-integration    # Repository 整合測試（需 Docker）

# 程式碼生成
make wire                # 重新生成 Wire DI
make swag                # 重新生成 Swagger 文件（若有加 godoc）

# 確認架構邊界（定期執行）
grep -r "gorm.io" pkg/api/handler/ pkg/usecase/   # 應無輸出
grep -r "gin.Context" pkg/usecase/ pkg/repository/ # 應無輸出
go build ./...                                      # 無 import cycle
```

---

## 相關頁面

- [[Gin_Clean_Architecture]] — 目標架構 4 層結構詳解
- [[gin-legacy-to-clean-arch-openspec]] — 舊有專案遷移到此架構的指南
- [[Spec驅動開發]] — 為什麼先寫規格再讓 AI 實作
- [[OpenSpec工作流]] — 8 階段完整 SDD 流程
- [[Go Wire深度實戰]] — Wire provider/injector 詳細教學
- [[Go PostgreSQL測試策略]] — testcontainers-go 整合測試
- [[Go安全性實踐]] — JWT 生成/驗證、bcrypt、Rate Limiting
- [[Claude Code 入門完整指南]] — Claude Code 基本操作
- [[CLAUDE.md撰寫最佳實踐]] — 如何寫出有效的護欄規則
