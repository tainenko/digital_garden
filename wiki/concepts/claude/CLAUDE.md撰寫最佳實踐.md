---
title: CLAUDE.md 撰寫最佳實踐
type: concept
tags: [claude, claude-code, CLAUDE.md, 設定, 規範, context-management]
created: 2026-04-30
updated: 2026-05-07
sources: [youtube-codememayb-claude-code-deep-dive]
---

# CLAUDE.md 撰寫最佳實踐

`CLAUDE.md` 是 Claude Code 的專案級指令檔。每次啟動 Claude Code，它會自動讀取並遵守其中的規範。寫得好的 CLAUDE.md 能大幅減少你需要重複說明的內容，讓 Claude 在第一次嘗試就輸出符合規範的程式碼。

---

## 放置位置與載入時機

| 位置 | 作用範圍 | 載入時機 |
|------|---------|---------|
| `~/.claude/CLAUDE.md` | 全域，跨所有專案 | **每次都載入** |
| `<專案根>/CLAUDE.md` | 整個專案，check in Git，團隊共用 | **每次都載入** |
| `<專案根>/CLAUDE.local.md` | 僅限個人，**不** check in Git | **每次都載入** |
| `<子目錄>/CLAUDE.md` | 該子目錄範圍 | **只在 Claude 讀取該目錄檔案時**才載入 |

### 子目錄 CLAUDE.md 的關鍵差異

根目錄的 CLAUDE.md 每次對話都佔用 context token。**子目錄** CLAUDE.md 只在 Claude Code 需要讀取該目錄的檔案時才被載入——對大型 monorepo 來說，這是節省 context 的重要設計：

```
專案根/
├── CLAUDE.md          ← 每次都讀，放整體規範
├── CLAUDE.local.md    ← 個人設定，不 check in
├── api/
│   └── CLAUDE.md      ← 只在 Claude 讀 api/ 時載入
└── frontend/
    └── CLAUDE.md      ← 只在 Claude 讀 frontend/ 時載入
```

### 用 `/init` 自動生成初版

```bash
/init
```

Claude Code 會掃描整個 codebase，自動產生包含架構概覽、常用指令、重要模組的 CLAUDE.md 初版。

---

## 核心結構

```markdown
# 專案名稱 — LLM 指令

## 技術棧
- 語言版本：Go 1.24
- 框架：Gin v1.10、GORM v2
- 資料庫：PostgreSQL 16
- 測試：testify v1、gomock

## 目錄結構
\`\`\`
cmd/           ← 各服務入口
internal/      ← 核心業務邏輯（禁止外部 import）
pkg/           ← 可對外複用的套件
migrations/    ← 資料庫 migration（禁止 LLM 直接修改）
\`\`\`

## 程式碼規範
- 函數超過 50 行需拆分
- 禁用 global variable，使用依賴注入
- 所有 error 必須處理，不可 `_ = err`
- context.Context 必須是第一個參數

## 測試規範
- 新增功能必須附 unit test
- 測試用 table-driven 格式
- 不 mock database，用 testcontainers-go

## 禁止事項
- 不修改 migrations/ 目錄
- 不修改 go.mod、go.sum（除非明確要求）
- 不使用 `fmt.Println` 做 logging，一律用 slog

## 常用指令
- 執行測試：`go test ./...`
- 建置：`go build ./cmd/...`
- lint：`golangci-lint run`
```

---

## 七個讓 CLAUDE.md 更有效的技巧

### 1. 用「禁止」而非「建議」

❌ 效果較弱：
```markdown
最好用 slog 做 logging
```

✅ 效果更強：
```markdown
禁止使用 fmt.Println 做 logging，一律使用 slog 套件
```

### 2. 說明「為什麼」，不只說「什麼」

```markdown
## migrations/ 目錄
**禁止修改此目錄**。原因：migration 檔案一旦執行就不可逆，
所有 schema 變更必須通過 PR review 流程，由人工審查後才能合併。
```

提供原因後，Claude 在邊界情況下能更好地判斷意圖。

### 3. 提供範例，不只是規則

```markdown
## 錯誤處理
❌ 不要這樣：
```go
result, _ := db.Query(ctx, sql)
```

✅ 要這樣：
```go
result, err := db.Query(ctx, sql)
if err != nil {
    return fmt.Errorf("查詢失敗: %w", err)
}
```

### 4. 列出「先看哪些檔案」

```markdown
## 新 feature 開發前必讀
1. `internal/domain/` — 核心 domain model 定義
2. `internal/handler/user_handler.go` — handler 寫法範例
3. `internal/repository/user_repo.go` — repository 寫法範例

照這三個檔案的風格撰寫新程式碼。
```

### 5. 標示「高風險區域」

```markdown
## ⚠️ 高風險區域
以下區域修改須格外謹慎，強烈建議先討論再動手：
- `internal/auth/` — JWT 驗證邏輯，安全敏感
- `internal/billing/` — 金流處理，任何 bug 會影響收款
- `cmd/migrator/` — 資料庫 migration 執行器
```

### 6. 定義命名慣例

```markdown
## 命名慣例
- Handler 函式：`{動詞}{資源}Handler`，例如 `CreateUserHandler`
- Repository 介面：`{資源}Repository`，例如 `UserRepository`
- Service 介面：`{資源}Service`，例如 `UserService`
- 測試檔案：`{被測檔案}_test.go`
- 錯誤變數：`Err{說明}`，例如 `ErrUserNotFound`
```

### 7. 指定常用指令

```markdown
## 指令速查
```bash
# 測試
go test ./... -v -race

# 產生 mock
mockgen -source=internal/repository/user_repo.go -destination=mocks/user_repo_mock.go

# 執行 migration
goose -dir migrations postgres "$DATABASE_URL" up

# 啟動本地開發
docker-compose up -d && go run cmd/api/main.go
```

---

## 不同專案類型的 CLAUDE.md 模板

### Go 微服務

```markdown
# API Service — LLM 指令

## 技術棧
Go 1.24 / Gin / Wire / GORM / PostgreSQL / Redis / Kafka

## 架構
Clean Architecture：Handler → Service → Repository
禁止 Handler 直接存取 Repository（必須透過 Service）

## 測試策略
- Handler：用 httptest 做整合測試
- Service：純 unit test，mock repository
- Repository：用 testcontainers-go 跑真實 DB

## 禁止事項
- 不在 Service 層直接處理 HTTP 相關邏輯
- 不在 Handler 層寫業務邏輯
```

### Python FastAPI

```markdown
# FastAPI Service — LLM 指令

## 技術棧
Python 3.12 / FastAPI / SQLAlchemy 2.0 / Pydantic v2 / PostgreSQL

## 慣例
- 所有 schema 用 Pydantic v2 model
- 資料庫 session 用 dependency injection
- async/await 優先，只在 CPU 密集時用 sync

## 測試
- pytest + pytest-asyncio
- 用 httpx.AsyncClient 測試 endpoint
- 每個 test 有獨立 transaction，測試後 rollback
```

---

## 常見錯誤

| 錯誤 | 問題 |
|------|------|
| CLAUDE.md 過長（超過 500 行） | Claude 可能忽略後段內容 |
| 只寫「最好」「建議」 | 模糊指令效果差 |
| 沒有提供範例 | Claude 可能誤解規範的含義 |
| 規範互相矛盾 | Claude 會選擇其中一個，結果不可預期 |

---

## 團隊協作：CLAUDE.md check in 到 Git

[[Boris]]（Claude Code 創造者）的實戰做法：

```
CLAUDE.md 提交到 Git → 全團隊共同維護
```

工作流程：
1. **Claude 做錯某件事** → 立刻加入 CLAUDE.md，commit
2. **Code Review 發現問題** → 直接 tag Claude 讓它更新 CLAUDE.md
3. **同一件事不用說第二次** — 新成員、新 session 全部繼承

這讓 CLAUDE.md 成為**團隊集體記憶**，而非個人偏好設定。

---

## 相關頁面

- [[Claude Code 入門完整指南]]
- [[Claude Prompt工程核心技巧]]
- [[Claude Code Hooks 深度設定]]
- [[Claude Code多人團隊協作指南]]
- [[CoWork桌面工具指南]] — Boris 四大心法來源
- [[生產環境Vibe Coding四大策略]]
- [[Spec驅動開發]]
