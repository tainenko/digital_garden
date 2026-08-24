---
title: Claude Code 入門完整指南
type: concept
tags: [claude, claude-code, cli, 入門, 工具, slash-commands, shortcuts]
created: 2026-04-30
updated: 2026-05-07
sources: [youtube-codememayb-claude-code-deep-dive]
---

# Claude Code 入門完整指南

Claude Code 是 Anthropic 官方的命令列 AI 編程工具（CLI），讓你在終端機中直接與 Claude 協作開發，無需切換到瀏覽器。

---

## 安裝與啟動

```bash
# 安裝（需要 Node.js 18+）
npm install -g @anthropic-ai/claude-code

# 啟動（在專案目錄下執行）
claude

# 指定模型啟動
claude --model claude-opus-4-7
```

驗證 API Key：首次啟動會引導設定 `ANTHROPIC_API_KEY`，也可以事先設環境變數：

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

---

## 核心操作模式

### 1. 互動模式（預設）

直接在終端執行 `claude`，進入 REPL 對話。你可以：

- 詢問程式碼問題
- 請 Claude 閱讀、修改、新增檔案
- 執行終端指令並分析輸出

### 2. 單次指令模式（Non-interactive）

```bash
# 直接傳入 prompt，適合腳本整合
claude -p "幫我寫一個 Go 的 HTTP middleware 記錄請求時間"

# 讀取檔案內容後分析
claude -p "分析這段程式碼有哪些問題" < main.go
```

### 3. 管道模式

```bash
# 分析 git diff
git diff | claude -p "總結這次變更，寫 commit message"

# 分析測試失敗
go test ./... 2>&1 | claude -p "測試失敗了，幫我找原因"
```

---

## 特殊前綴指令

| 前綴 | 功能 |
|------|------|
| `!` | **Bash Mode**（輸入框變粉紅色）：直接執行終端機指令，無需退出 Claude Code |
| `@<路徑>` | **引用檔案**：將指定檔案加入本次 prompt 的 context |
| `#<內容>` | **快速寫入 CLAUDE.md**（2.0.70 以前版本）：直接附加內容到 CLAUDE.md |

## 斜線指令（Slash Commands）完整表

| 指令 | 功能 |
|------|------|
| `/help` | 顯示所有可用指令與快捷鍵 |
| `/init` | 掃描 codebase，自動生成 CLAUDE.md（架構、指令、模組） |
| `/clear` | 清空整個對話歷史 |
| `/compact` | 壓縮對話（摘要目前狀態後繼續，節省 context） |
| `/context` | 顯示 context 使用量（200k token 上限的細分：system prompt / 工具 / memory / 對話）|
| `/model` | 切換模型（pro 方案預設 sonnet；Max 方案預設 opus 4.5）|
| `/rewind` | 回到指定的歷史對話節點，可選擇只還原 code 或同時還原 conversation |
| `/mcp` | 管理 MCP server（查看已安裝清單、連線狀態、工具列表）|
| `/task` | 查看所有背景執行任務，按 `k` 可 kill 任務 |
| `/resume` | 恢復上次的對話（重新啟動 Claude Code 後使用）|
| `/memory` | 選擇層級後直接編輯 CLAUDE.md |
| `/exit` | 退出 Claude Code，回到一般終端 |
| `/doctor` | 檢查目前安裝版本與健康狀態 |

## 鍵盤快捷鍵

| 快捷鍵 | 功能 |
|--------|------|
| `Shift+Tab` | 循環切換許可權模式：Manual（每次確認）→ Accept Edits On → Plan Mode |
| 雙按 `Esc` | 快速呼叫 `/rewind`，選擇還原點 |
| `Ctrl+B` | 將當前執行程序移到背景（不阻塞對話繼續） |
| `Ctrl+C` × 2 | 退出 Claude Code |

## 啟動選項

```bash
claude          # 互動模式
claude -c       # 繼續上次對話（continue）
claude -p "..."  # 單次 prompt 模式（non-interactive）
```

---

## 檔案操作能力

Claude Code 可以直接讀寫專案中的任何檔案。常見用法：

```
# 在對話中請 Claude 讀取並修改
「幫我看 src/handlers/user.go，把 error handling 改成符合 Go 最佳實踐」

# 跨檔案重構
「把 utils/helpers.go 裡的 ParseDate 函數移到 pkg/dateutil 套件，並更新所有使用它的地方」

# 生成新檔案
「根據 schema.sql，生成對應的 Go struct 和 CRUD repository」
```

---

## 上下文管理技巧

### 使用 CLAUDE.md

在專案根目錄放 `CLAUDE.md`，Claude Code 每次啟動都會自動讀取：

```markdown
# 專案規範
- 語言：Go 1.22+
- 框架：Gin + Wire
- 測試：需要配套 unit test，使用 testify
- 不要修改 migrations/ 目錄
```

### 壓縮對話避免 context 爆掉

長對話中執行 `/compact` 可以讓 Claude 摘要目前的狀態後繼續，避免 context window 限制。

### 分拆任務給子 agent

大型任務可以要求 Claude 使用子 agent 並行處理：

```
「這個重構任務有三個獨立部分，請分別開三個子 agent 同時處理：
1. 更新 user module
2. 更新 payment module  
3. 更新 notification module」
```

---

## 權限控制

Claude Code 要求明確授權才會執行某些危險操作。常見設定：

```json
// .claude/settings.json
{
  "permissions": {
    "allow": [
      "Bash(go test:*)",
      "Bash(go build:*)",
      "Bash(git status)",
      "Bash(git diff:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(git push:*)"
    ]
  }
}
```

---

## 最佳實踐

1. **先說任務邊界**：「只修改 handlers 目錄，不要動 models」
2. **給範例**：「照著 `user_handler.go` 的風格寫 `order_handler.go`」
3. **分段確認**：大任務要求 Claude 先列計畫，確認後再執行
4. **善用 TDD**：先寫測試，讓 Claude 讓測試通過，而不是直接讓它「寫功能」
5. **版控保護**：重大修改前先 commit，萬一改壞了方便 revert

---

## 相關頁面

- [[Claude Code內部運作機制]] — Tool Use 原理
- [[CLAUDE.md撰寫最佳實踐]]
- [[Claude MCP 伺服器整合指南]]
- [[Claude Code Hooks 深度設定]]
- [[Claude Prompt工程核心技巧]]
- [[生產環境Vibe Coding四大策略]]
