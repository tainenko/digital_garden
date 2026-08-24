---
title: Claude Code 記憶體系統深度指南
type: concept
tags: [claude-code, memory, CLAUDE.md, auto-memory, 記憶體, 持久化]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Claude Code 記憶體系統深度指南

每次 Claude Code session 都從空白的 context window 開始。記憶體系統讓 Claude 跨 session 保留知識——有兩種互補機制：你寫的 **CLAUDE.md**，以及 Claude 自己寫的 **Auto Memory**。

---

## 兩種記憶體的對比

| | CLAUDE.md | Auto Memory |
|--|-----------|-------------|
| **誰寫** | 你 | Claude 自己 |
| **包含什麼** | 指令與規則 | 學習到的模式與洞察 |
| **範圍** | 專案 / 用戶 / 組織 | 每個 git 工作樹 |
| **每次 session 載入** | 全部 | MEMORY.md 前 200 行或 25KB |
| **適合放什麼** | 代碼規範、架構決策、禁止事項 | Build 命令、除錯洞察、你糾正 Claude 後它學到的偏好 |

---

## 第一層：CLAUDE.md 文件體系

### 五個範圍，由窄到廣的優先順序

更具體的位置優先覆蓋更廣泛的位置：

| 範圍 | 路徑 | 用途 | 共享給 |
|------|------|------|--------|
| **Managed Policy** | macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md` | IT/DevOps 管理的組織規範 | 組織所有用戶 |
| **專案指令** | `./CLAUDE.md` 或 `./.claude/CLAUDE.md` | 團隊共享的專案規範 | 透過 git 共享給團隊 |
| **用戶指令** | `~/.claude/CLAUDE.md` | 個人偏好（所有專案） | 只有你 |
| **本地指令** | `./CLAUDE.local.md`（加入 .gitignore）| 個人的專案特定設定 | 只有你 |
| **子目錄** | `./src/CLAUDE.md` 等 | 子目錄專屬指令 | 按需載入（Claude 讀取該目錄文件時才載入）|

### CLAUDE.md 的載入機制

Claude Code 從當前工作目錄往上走，逐層載入 `CLAUDE.md` 和 `CLAUDE.local.md`：

```
工作目錄：foo/bar/
↓ 載入順序（全部疊加，不覆蓋）：
foo/bar/CLAUDE.md
foo/bar/CLAUDE.local.md
foo/CLAUDE.md
foo/CLAUDE.local.md
```

子目錄（如 `foo/bar/src/CLAUDE.md`）**不在 session 開始時載入**，而是在 Claude 讀取那個目錄的文件時才按需載入。

### 引用外部文件（@import）

```markdown
# CLAUDE.md 可以引用其他文件
See @README.md for project overview.
@docs/api-conventions.md
@~/.claude/my-personal-rules.md   ← 跨 worktree 共用個人規則
```

引用的文件在 session 開始時完整載入，深度最多 5 層。

---

## 第二層：`.claude/rules/` 路徑範圍規則

對於大型專案，把所有規則放進一個 CLAUDE.md 會讓它過於龐大。用 `.claude/rules/` 把規則模組化：

```
.claude/
├── CLAUDE.md               ← 主規則（每次 session 載入）
└── rules/
    ├── code-style.md       ← 代碼風格規則
    ├── testing.md          ← 測試規範
    ├── api-design.md       ← API 設計規則
    └── frontend/
        └── react.md        ← React 元件規範
```

### 路徑範圍規則（Path-scoped Rules）

加上 YAML frontmatter 的 `paths` 欄位，規則只在 Claude 處理匹配文件時才載入：

```markdown
---
paths:
  - "src/api/**/*.ts"
  - "src/handlers/**/*.ts"
---

# API 開發規範

- 所有 API endpoint 必須包含輸入驗證
- 使用標準 ErrorResponse 格式
- 回傳值必須有 OpenAPI 文件註解
```

```markdown
---
paths:
  - "**/*.test.ts"
  - "**/*.spec.ts"
---

# 測試規範

- 每個測試文件最多 200 行
- describe block 命名：[Unit] 功能描述
- 禁止使用 any 類型
```

這樣只有在 Claude 正在處理 TypeScript API 文件時，API 規則才會進入 context，節省 token。

### 個人規則（User-level Rules）

`~/.claude/rules/` 裡的規則適用於你所有的專案：

```
~/.claude/rules/
├── preferences.md     ← 個人代碼風格
└── workflow.md        ← 個人工作流程習慣
```

---

## 第三層：Auto Memory（自動記憶）

Claude 自動維護的筆記本。當 Claude 學到有用的東西（你的更正、偏好、Build 命令），它會自己把它寫下來。

> 需要 Claude Code v2.1.59 或更高版本。

### 存儲位置

```
~/.claude/projects/<project>/memory/
├── MEMORY.md          ← 索引文件（每次 session 開始載入前 200 行）
├── debugging.md       ← 除錯模式的詳細筆記
├── api-conventions.md ← API 設計決策記錄
└── build-config.md    ← Build 配置和命令
```

`<project>` 路徑由 git 倉庫決定：同一個 repo 的所有 worktree 和子目錄共享一個 auto memory 目錄。

### MEMORY.md 是索引，不是全部

```
MEMORY.md（索引）：只載入前 200 行
├── 指向 debugging.md 的指標
├── 指向 api-conventions.md 的指標
└── 最重要的摘要（常用 Build 命令等）

debugging.md、api-conventions.md 等：
└── 按需讀取（Claude 需要時才讀）
```

### 讓 Claude 記住某件事

直接在對話中說：

```
# 讓 Claude 記住
「記住這個專案要用 pnpm，不要用 npm」
「記住 API 測試需要本地 Redis 實例」
「記住用戶 token 存在 X-Auth-Token header，不在 Bearer」

# 讓 Claude 寫進 CLAUDE.md 而不是 auto memory
「把這條規則加到 CLAUDE.md：所有新函數必須有 godoc 說明」
```

### 開啟/關閉 Auto Memory

```json
// .claude/settings.json（關閉 auto memory）
{
  "autoMemoryEnabled": false
}
```

```bash
# 環境變數方式關閉
CLAUDE_CODE_DISABLE_AUTO_MEMORY=1 claude
```

用 `/memory` 命令查看和管理所有記憶體文件：
- 列出當前 session 載入的所有 CLAUDE.md 和 rules 文件
- 開啟 auto memory 目錄
- 切換 auto memory 開關

---

## 實戰：為一個新專案建立記憶體體系

### 第一步：用 `/init` 自動生成基礎 CLAUDE.md

```bash
/init
```

Claude 掃描你的專案（目錄結構、Build 系統、測試框架），生成包含 Build 命令、測試指令、專案慣例的初始 CLAUDE.md。

設定 `CLAUDE_CODE_NEW_INIT=1` 啟用互動式多步驟流程，可以同時設定 CLAUDE.md、Skills 和 Hooks。

### 第二步：建立 rules/ 模組

```bash
mkdir -p .claude/rules
```

把不同主題的規則拆分到獨立文件：

```markdown
# .claude/rules/go-style.md
---
paths:
  - "**/*.go"
---

# Go 代碼規範

- 錯誤處理：用 fmt.Errorf("context: %w", err) 包裝
- 命名：exported functions 用 PascalCase，unexported 用 camelCase
- 測試：用 table-driven tests，test helper 函數用 t.Helper()
- 不得 panic，除非在 init() 中
```

### 第三步：建立 CLAUDE.local.md（私人設定，不提交）

```markdown
# CLAUDE.local.md
# 加入 .gitignore

# 本地開發環境特有設定
本地 API 端點：http://localhost:8080
測試用 Redis：redis://localhost:6379/15
我的測試帳號：test@example.com
```

### 第四步：讓 Auto Memory 開始學習

前幾週正常使用 Claude Code。每次你糾正 Claude 的錯誤（「不，這裡應該用 pnpm」「這個 API 路徑是舊的，現在是...」），Claude 會自動把學到的東西寫進 auto memory。

定期用 `/memory` 檢查 auto memory 是否正確，有誤的刪掉或修正。

---

## 有效 CLAUDE.md 的寫作原則

### 長度：每個文件控制在 200 行以內

超過 200 行的文件：
- 消耗更多 context token
- Claude 遵循度下降
- 解法：用 path-scoped rules 把規則拆分到多個文件

### 具體性：可驗證的指令

```
❌ 「格式化代碼」→ Claude 不知道什麼算「格式化好了」
✅ 「用 2 空格縮排，不用 Tab」

❌ 「測試你的修改」
✅ 「提交前執行 go test ./...，全部通過才能繼續」

❌ 「保持文件組織良好」
✅ 「API handler 放在 src/api/handlers/，model 放在 src/models/」
```

### 一致性：定期清理矛盾

多個 CLAUDE.md 文件可能出現互相矛盾的指令，Claude 可能隨機選一個。用 `/memory` 定期審查，移除過時或衝突的規則。

### 禁止語言比建議語言更有效

```
# CLAUDE.md
✅ 「絕對不要直接修改 production database」
✅ 「禁止在 handler 層直接訪問 repository，必須透過 service 層」
✅ 「永遠不要刪除 migration 文件」

比「建議使用...」「最好...」更可靠
```

---

## 常見問題排查

### Claude 沒有遵循 CLAUDE.md 規則

1. 用 `/memory` 確認該文件是否被載入
2. 檢查規則是否在正確的位置（見 [五個範圍](#五個範圍由窄到廣的優先順序)）
3. 讓指令更具體（「用 2 空格縮排」比「格式化代碼」更有效）
4. 檢查不同 CLAUDE.md 文件之間是否有矛盾指令

### `/compact` 之後指令消失了

`/compact` 之後，根目錄的 CLAUDE.md 會重新載入；但子目錄的 CLAUDE.md 不會自動重新注入，要等 Claude 再次讀取那個目錄的文件才會觸發。

解法：把重要指令放進專案根目錄的 CLAUDE.md，而不是子目錄。

### Auto Memory 存了錯誤的東西

```bash
# 方法一：在對話中說
「刪除你記住的關於 npm 的筆記，我們這個專案用 pnpm」

# 方法二：直接編輯文件
~/.claude/projects/<project>/memory/
```

Auto memory 文件都是純 Markdown，直接編輯或刪除即可。

---

## 記憶體體系總覽

```
組織層（Managed Policy CLAUDE.md）
    ↓ 覆蓋所有用戶
用戶層（~/.claude/CLAUDE.md + ~/.claude/rules/）
    ↓ 覆蓋所有專案
專案層（./CLAUDE.md + ./.claude/rules/）
    ↓ 覆蓋當前專案
本地層（./CLAUDE.local.md）
    ↓ 個人的專案私有設定
Auto Memory（~/.claude/projects/<project>/memory/）
    ↓ Claude 自己學到的東西
```

每一層都疊加，不覆蓋。更具體的層優先。

---

## 相關頁面

- [[CLAUDE.md撰寫最佳實踐]]（CLAUDE.md 的寫作技巧詳解）
- [[Claude Code 入門完整指南]]
- [[Claude Code Subagents 完整指南]]（Subagent 也有自己的 memory）
- [[Claude Code Hooks 深度設定]]（配合 hooks 在記憶體載入時觸發）
- [[100%自動化原則與AI杠桿]]（從 95% 到 100% 需要系統性反饋機制）
