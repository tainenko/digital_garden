---
title: 用 OpenSpec 漸進式遷移 Legacy Gin+GORM 架構
type: topic
tags: [Golang, Gin, CleanArchitecture, OpenSpec, 重構, 漸進式遷移, GORM]
created: 2026-05-04
updated: 2026-05-04
sources: [github-go-gin-clean-arch, openspec-superpowers-multi-source]
---

# 用 OpenSpec 漸進式遷移 Legacy Gin+GORM 架構

## 你面對的問題

典型的 Legacy Gin+GORM 服務長這樣：

```
handlers/
  user.go       ← Handler 直接 new(gorm.DB)，業務邏輯和 DB 查詢混在一起
  order.go      ← 複製貼上 user.go 的 DB 連線邏輯
models/
  user.go       ← GORM struct，有時夾雜 validation 邏輯
  order.go
main.go         ← router 定義、DB 連線、middleware 全部堆在這裡
```

痛點很明確：

| 症狀 | 根本原因 |
|------|---------|
| 無法寫 unit test（需要真實 DB） | Handler 直接依賴 *gorm.DB，沒有抽象層 |
| 加一個功能需要動 3–4 個地方 | 職責沒有分層，散落各處 |
| 不敢改 handler，怕動到 DB 邏輯 | 業務邏輯與框架細節耦合 |
| 新人讀不懂資料從哪裡來 | 沒有明確的資料流向 |

目標架構：[[Gin_Clean_Architecture]] 的 4 層分離（Domain → Repository → UseCase → Handler）。

---

## 遷移策略：漸進式，不是大爆炸

**不要** 一次把整個 codebase 重寫。正確做法：

```
舊程式碼繼續跑
  ↓
每次新增功能或修 bug 時，對那個 module 套用新架構
  ↓
舊 module 在自然的時機點遷移（不強制）
  ↓
6–12 週後，新架構已覆蓋核心 module
```

**關鍵原則**：新功能只用新架構寫；舊功能優先維持可動，等自然觸碰再遷移。

---

## Phase 0｜Explore — 讓 AI 摸清現況

在 Claude Code 中執行：

```
請分析這個 Gin+GORM 專案的現有結構。
我需要你回答：
1. 各個 handler 做了哪些事？有哪些直接呼叫 DB 的地方？
2. 現有的 models 有哪些？有沒有 model 上夾雜業務邏輯？
3. 相依關係地圖：哪些 handler 依賴哪些 model？
4. 最需要被遷移的前 3 個 module（依照：最常被改動、測試覆蓋率最低、耦合最重）
```

輸出目標：一份**現況報告**，讓你知道從哪裡開始。

---

## Phase 1｜Proposal — 定義遷移範圍

```markdown
# proposal.md（讓 AI 幫你生成）

## 問題
handlers/user.go 將 JWT 驗證、業務邏輯、DB 查詢混在同一個函式，
導致無法單元測試，且每次需求變更都要同時改 3 個地方。

## 目標
將 User module 遷移至 Clean Architecture 4 層結構，
使 UseCase 層可被 mock Repository 單元測試。

## 邊界（這次只做這些）
- 遷移範圍：User module（CRUD + 登入）
- 不包含：Order / Product module（下一個 sprint）
- 不包含：資料庫 schema 變更
- 不包含：API 合約變更（endpoint 和 response 格式維持不變）
```

**邊界宣告非常重要**：防止 AI 把整個 repo 都重構掉。

---

## Phase 2｜Design — 架構決策

讓 AI 生成 `design.md`，至少要回答這 5 個問題：

### Q1：interface 放哪裡？
```
選擇 A：pkg/repository/interface/ + pkg/usecase/interface/（依 thnkrn 範例）
選擇 B：每個 module 自己的 interface.go

建議：用選擇 A，interface 與 impl 分開資料夾，grep 時更清楚。
```

### Q2：GORM model 和 Domain struct 要分嗎？
```
選擇 A：同一個 struct，domain 上加 gorm tag（thnkrn 做法，pragmatic）
選擇 B：分離 domain.User 和 model.UserORM，repository 做轉換

建議：Legacy 遷移用選擇 A，先讓結構對，再評估是否要分離。
```

### Q3：Wire 要現在引入嗎？
```
Legacy 遷移初期建議：先用手寫 constructor chain（main.go）。
Wire 引入時機：當 dependency graph 超過 5 個 provider 開始變複雜時。
```

### Q4：舊 handler 和新 handler 如何共存？
```
路由前綴分離法：
  /api/v1/users  → 舊 handler（維持）
  /api/v2/users  → 新 handler（Clean Architecture）
  → 測試穩定後，切換 v1 指向 v2
```

### Q5：測試策略？
```
遷移後的驗證順序：
1. Repository 整合測試（需要 testcontainers PostgreSQL）
2. UseCase 單元測試（mock Repository interface）
3. Handler E2E 測試（httptest + mock UseCase）
```

---

## Phase 3｜Behavioral Specs — 每層的行為規格

這是 OpenSpec 最重要的輸出。用 Given/When/Then 描述每層的預期行為：

### Repository Specs

```
[UserRepository.FindByID]
Given: 資料庫中存在 id=1 的 user
When: FindByID(ctx, 1)
Then: 回傳 domain.User{ID:1, ...}, nil

Given: 資料庫中不存在 id=999 的 user
When: FindByID(ctx, 999)
Then: 回傳 domain.User{}, gorm.ErrRecordNotFound

[UserRepository.Save]
Given: user.ID == 0（新建）
When: Save(ctx, user)
Then: 回傳帶有 DB 分配 ID 的 user, nil

Given: user.ID == 5（更新）
When: Save(ctx, user)
Then: 更新現有紀錄，回傳更新後 user, nil
```

### UseCase Specs

```
[UserUseCase.Register]
Given: email 不存在於 DB
When: Register(ctx, {email: "a@b.com", password: "123456"})
Then: password 以 bcrypt hash 儲存，回傳新 user（不含 password hash）

Given: email 已存在於 DB
When: Register(ctx, {email: "existing@b.com", ...})
Then: 回傳 ErrEmailAlreadyExists（不是 DB error）

[UserUseCase.Login]
Given: email/password 正確
When: Login(ctx, email, password)
Then: 回傳 JWT token string, nil

Given: password 錯誤
When: Login(ctx, email, wrongPassword)
Then: 回傳 "", ErrInvalidCredentials
```

### Handler Specs

```
[POST /api/users/register]
Given: 合法的 JSON body
When: 呼叫此 endpoint
Then: status 201, body {"id": N, "email": "..."}（無 password 欄位）

Given: email 格式錯誤
When: 呼叫此 endpoint
Then: status 400, body {"error": "invalid email format"}

Given: email 已存在
When: 呼叫此 endpoint
Then: status 409, body {"error": "email already exists"}
```

---

## Phase 4｜Tasks — 漸進式實作清單

把 Behavioral Specs 拆成可執行的小任務：

```markdown
## Sprint 1：建立骨架（不動舊程式碼）

- [ ] 建立 pkg/domain/user.go（從舊 models/user.go 複製，去除業務邏輯）
- [ ] 建立 pkg/repository/interface/user.go（定義 interface）
- [ ] 建立 pkg/usecase/interface/user.go（定義 interface）
- [ ] 建立 pkg/repository/user.go（GORM 實作，通過 Repository 整合測試）
- [ ] 建立 pkg/usecase/user.go（通過 UseCase 單元測試，用 mock repo）
- [ ] 建立 pkg/api/handler/user_v2.go（新版 handler）
- [ ] 在 server.go 新增 /api/v2/users 路由指向新 handler

## Sprint 2：驗證與切換

- [ ] E2E 測試確認 /api/v2/users 行為與 /api/v1/users 完全一致
- [ ] 將 /api/v1/users 路由靜默指向新 handler（保留舊 URL）
- [ ] 監控 7 天，確認無 regression
- [ ] 刪除舊 handlers/user.go

## Sprint 3：重複以上流程於下一個 module
```

---

## Phase 5｜Apply — Claude Code 實際執行

### Prompt 模板：生成 Repository

```
請參考 pkg/domain/user.go 的現有 struct 定義，
以及 pkg/repository/interface/user.go 的 interface，
幫我實作 pkg/repository/user.go。

規則：
- constructor 回傳 interface，不是具體型別
- 所有方法接受 context.Context 作為第一個參數
- FindByID 找不到時回傳 gorm.ErrRecordNotFound，不要 wrap
- Save 自動判斷 create vs update（依 ID 是否為 0）
- 不要新增任何 behavioral specs 以外的方法
```

### Prompt 模板：生成 UseCase（含測試）

```
請幫我實作 pkg/usecase/user.go，同時生成對應的 usecase_test.go。

UseCase 依賴：interfaces.UserRepository（用 mock 測試）
測試工具：testify/mock

測試案例必須涵蓋：
- Register 成功路徑
- Register email 重複 → ErrEmailAlreadyExists
- Login 成功 → 回傳 JWT
- Login 密碼錯誤 → ErrInvalidCredentials

不要測試 repository 行為（那是 repository_test.go 的事）。
```

### Prompt 模板：遷移舊 Handler

```
以下是現有的 handlers/user.go（舊版），它直接操作 *gorm.DB：

[貼上舊程式碼]

請將它遷移成 pkg/api/handler/user_v2.go，遵守：
1. 只依賴 services.UserUseCase interface，不知道 GORM 存在
2. 用 copier 或手寫轉換將 domain struct 轉為 Response struct
3. HTTP status code 對應：
   - ErrEmailAlreadyExists → 409
   - ErrInvalidCredentials → 401
   - ErrRecordNotFound → 404
   - 其他 error → 500
4. endpoint 路徑和 response 格式與舊版完全相同
```

---

## Phase 6｜Verify — 驗證遷移正確性

### 自動化驗證清單

```bash
# 1. UseCase 單元測試（不需 DB）
go test ./pkg/usecase/... -v

# 2. Repository 整合測試（需要 DB）
go test ./pkg/repository/... -v -tags=integration

# 3. Handler E2E 測試
go test ./pkg/api/handler/... -v

# 4. 確認沒有 import cycle
go build ./...

# 5. 確認新 handler 沒有直接 import gorm
grep -r "gorm.io" pkg/api/handler/   # 應該沒有輸出
grep -r "gorm.io" pkg/usecase/       # 應該沒有輸出
```

### 行為一致性驗證

```bash
# 對舊/新 endpoint 跑相同的 API 測試
# 確認 response 完全一致
diff \
  <(curl -s http://localhost:3000/api/v1/users) \
  <(curl -s http://localhost:3000/api/v2/users)
```

---

## Phase 7｜Archive — 記錄決策

```markdown
# archive/2026-05-user-module-migration.md

## 決策紀錄

### 為什麼 domain struct 保留 GORM tag？
理由：避免維護兩套 struct + mapping 函式的成本。
代價：domain 知道 ORM 的存在（輕微違反 Clean Architecture 純粹性）。
觸發重新評估的條件：若未來需要支援多種 DB 或更換 ORM。

### 為什麼不用 Wire？
理由：目前只有 User 一個 module，手寫 constructor chain 更直接。
計畫：超過 5 個 provider 時引入 Wire。

### 為什麼用 /api/v2 而不是直接替換？
理由：業務不停機，可以 A/B 對比行為，有問題立即回滾。
```

---

## 常見陷阱

### 陷阱 1：讓 AI 一次重構整個 repo
```
❌ 「把整個 handlers/ 資料夾遷移到 Clean Architecture」
✓  「只遷移 handlers/user.go，其他不動」
```

### 陷阱 2：遺漏 interface 邊界
```
❌ UseCase 直接接收 *gorm.DB 參數
✓  UseCase 只接收 Repository interface，不知道 *gorm.DB 存在
```

### 陷阱 3：在 Domain struct 加業務邏輯
```go
❌
type User struct {
    Password string
    func (u *User) ValidatePassword(raw string) bool { ... }  // 不要在 domain 加方法
}

✓  驗證邏輯放在 UseCase，domain 只是資料容器
```

### 陷阱 4：Handler 回傳 domain struct 而非 DTO
```go
❌ c.JSON(200, user)         // 直接暴露 domain，含 PasswordHash
✓  c.JSON(200, toResponse(user))  // 透過 Response struct 過濾欄位
```

### 陷阱 5：忘記 context 傳遞
```go
❌ repo.FindAll()
✓  repo.FindAll(c.Request.Context())  // 支援 timeout / cancellation
```

---

## 速查：一個 Module 的完整 Prompt 序列

```
1. [Explore]
   「分析 handlers/order.go，列出所有 DB 操作、業務邏輯、和 HTTP 細節，
    告訴我遷移到 Clean Architecture 需要建立哪些檔案」

2. [Domain]
   「根據現有 models/order.go，建立 pkg/domain/order.go，
    移除所有非資料欄位的 method，只保留 struct + GORM tag」

3. [Repository interface]
   「根據 handlers/order.go 中的所有 DB 操作，
    定義 pkg/repository/interface/order.go 的 OrderRepository interface，
    方法簽名包含 context.Context，錯誤回傳 error」

4. [Repository impl]
   「實作 pkg/repository/order.go，通過以下測試案例：[貼上 specs]」

5. [UseCase]
   「實作 pkg/usecase/order.go + order_test.go，
    mock OrderRepository interface，覆蓋所有 behavioral specs」

6. [Handler]
   「將 handlers/order.go 遷移為 pkg/api/handler/order_v2.go，
    只依賴 OrderUseCase interface，response 格式與舊版相同」

7. [Verify]
   「確認 pkg/api/handler/order_v2.go 沒有 import gorm，
    usecase 沒有 import gin，domain 沒有 import 任何 pkg/*」
```

---

## 相關頁面

- [[Gin_Clean_Architecture]] — 目標架構：4 層結構、Wire DI、完整程式碼範例
- [[OpenSpec工作流]] — 8 階段 explore → archive 流程
- [[Spec驅動開發]] — 為什麼要先寫規格再讓 AI 實作
- [[Go Wire深度實戰]] — 遷移後期引入 Wire 的操作指南
- [[Go PostgreSQL測試策略]] — Repository 整合測試：testcontainers-go
- [[DDD領域驅動設計]] — Domain layer 的理論基礎
