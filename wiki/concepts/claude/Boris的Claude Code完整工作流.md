---
title: Boris 的 Claude Code 完整工作流
type: concept
tags: [boris, claude-code, workflow, setup, parallel, worktree, opus, iterm2]
created: 2026-05-06
updated: 2026-05-06
sources: [boris-claude-code-setup-youtube, boris-cowork-startup-ideas-podcast]
---

# Boris 的 Claude Code 完整工作流

[[Boris]]（Claude Code 創造者）公開分享的個人工作流。核心主張：**開箱即用才是最強——不需要深度客製化，Claude Code 預設就能高效運作。**

---

## 一、環境設定

### 終端機：iTerm2 + 系統通知

Boris 偏好 **iTerm2**（非 Ghostty，但 Ghostty 是 Claude Code 團隊多數人的選擇）。  
關鍵設定：**iTerm2 系統通知**——當 Claude 需要輸入時自動彈出通知，不需盯著螢幕。

```
Tab 1: feature work
Tab 2: testing
Tab 3: code review
Tab 4: debugging
Tab 5: documentation
```

### 並行策略：5 個 Git Worktree

Boris 同時跑 **5 個 Claude Code 實例**，每個在獨立 git worktree：

```bash
# 建立 worktree（Claude Code 團隊偏好的方式）
git worktree add ../project-a feature/user-auth
git worktree add ../project-b fix/payment-bug

# 或使用 Claude Code 內建
claude --worktree feature-a
```

- 各 worktree 獨立，修改不互相衝突
- 用 shell alias（`za`、`zb`、`zc`）一鍵切換
- Boris 也跑 **5–10 個 claude.ai/code** 的 web sessions
- 早上在 iOS app 啟動任務，回到桌面繼續

> **管理者心態**：Boris 不執行程式碼，他管理任務。「照顧」這 5 個 Claude，在需要決策時介入。

---

## 二、模型選擇

### Opus 優先（現在：4.7，當時：4.5）

Boris 的選擇邏輯：

```
Opus 每 token 更貴
↓
但 Opus 更聰明、更少引導、工具使用更準確
↓
任務完成需要的 token 總量更少
↓
整體成本反而比 Sonnet 更低
```

> 「我用過最好的 coding 模型。就算比 Sonnet 慢，因為不需要太多引導，最終幾乎總是更快。」

### Opus 4.7 Effort 等級

| Effort | 適用場景 | Boris 預設 |
|--------|---------|-----------|
| `max` | 最難的 debug / 架構決策 | 非預設 |
| `xhigh` | 4.7 的新預設 | Boris 的 4.7 預設 |
| `high` | 一般工作 | Boris 在 4.5 時期的預設 |
| `medium` | 簡單查詢 | — |

---

## 三、工作流程

### 每個 Session 的標準流程

```
1. Plan Mode（Shift+Tab ×2）
2. ↕ 來回討論，直到計畫確認
3. 切換 auto-accept 模式
4. Claude 自動執行
5. 驗證（瀏覽器 / 測試 / bash）
6. /go → /simplify → PR
```

### 計畫模式（Plan Mode）

Boris 稱之為「最被低估的功能」：

- `Shift+Tab` 按兩次進入
- 在計畫模式下 Claude 只討論不執行
- 確認計畫後切換 auto-accept，Claude 通常能「一次到位」

> 「好的計畫非常重要，能避免後期大量問題。計畫確定了，程式碼就確定了。」

### 驗證優先原則

**驗證是 #1 優先級**，品質提升 **2–3 倍**：

| 場景 | 驗證方式 |
|------|---------|
| 前端 UI | Claude Chrome 擴充套件（直接看瀏覽器） |
| 後端 API | 啟動 server，跑 end-to-end test |
| CLI 工具 | Bash 自動化測試 |
| 桌面 App | Computer use |

---

## 四、Slash Commands

存放在 `.claude/commands/`，**check in 到 Git，全團隊共用**。

### Boris 常用 Commands

| Command | 功能 |
|---------|------|
| `/commit-push-pr` | 一鍵 commit → push → 建立 PR |
| `/simplify` | 平行 agent 審查，找重用/品質/效率問題 |
| `/go` | 測試 → /simplify → PR 的完整流程 |
| `/batch` | 互動規劃後，平行執行到多個 worktree |
| `/loop` | 排程定期任務（本機執行） |
| `/btw` | 旁鏈對話——不中斷主任務，快速問一個問題 |
| `/rewind` | 丟棄失敗嘗試，回到更早狀態 |
| `/compact` | LLM 摘要壓縮 context |

### `/go` 的實際用法

```
提示詞：「Claude 幫我做 blah blah /go」

→ Claude 自動執行：
  1. end-to-end 測試（bash / 瀏覽器 / computer use）
  2. /simplify（程式碼品質審查）
  3. 建立 PR
```

### `/loop` Boris 的定期任務

| 任務 | 頻率 |
|------|------|
| `/babysit`（PR 狀態監控）| 每 5 分鐘 |
| `/slack-feedback`（Slack 反饋整理）| 每 30 分鐘 |
| `/post-merge-sweeper` | merge 後 |
| `/pr-pruner`（清理過期 PR）| 每 1 小時 |

---

## 五、Hooks 設定

### PostToolUse：自動格式化

```json
"PostToolUse": [{
  "matcher": "Write|Edit",
  "hooks": [{
    "type": "command",
    "command": "bun run format || true"
  }]
}]
```

每次 Claude 寫入或修改檔案後自動跑 formatter，**零人工干預**。

---

## 六、MCP 整合

Boris 的 `.mcp.json` 常駐整合：

| MCP | 用途 |
|-----|------|
| Slack | 搜尋訊息、發布、連結 bug thread |
| BigQuery | 自主查詢，Boris 6 個月沒親手寫 SQL |
| Sentry | 抓取錯誤日誌 |

---

## 七、CLAUDE.md 維護

### 團隊實踐

```
全團隊一份 CLAUDE.md → check in Git → 每週多次更新
```

**觸發型更新**：
```
Claude 做錯 X → 立刻加進 CLAUDE.md：「不要做 X」
Code Review 發現問題 → 留言「@.claude add to CLAUDE.md」
→ Claude 自動更新 → 全團隊未來 session 自動繼承
```

### CLAUDE.md 範例內容
- Always use `bun`, not `npm`
- TypeCheck、test、lint 的執行方式
- PR pre-check 流程
- Never use TypeScript enums; prefer literal unions
- 特定工具的限制與反模式

這是[[複利工程思維]]的核心實踐場景。

---

## 八、Context 管理

### 基本原則

| 情境 | 策略 |
|------|------|
| 新任務 | `/clear` → 手寫 brief，精確控制 context |
| 相關延伸任務 | `/compact <hint>` → LLM 摘要，保持動能 |
| 失敗嘗試 | `/rewind`（雙 Esc）→ 丟棄，不要「修正」——修正會污染 context |

### 設定提前壓縮

```bash
export CLAUDE_CODE_AUTO_COMPACT_WINDOW=400000
```

Context 超過 30–40 萬 token 後品質下降，提前壓縮避免「context rot」。

---

## 九、提示詞技巧

### 給清晰的任務簡報

三個必備元素：
1. **Goal**：成功是什麼樣子
2. **Constraints**：不能動什麼、依賴什麼 API
3. **Acceptance criteria**：如何驗證完成

**範例**：
```
Goal: 為 /api/login 加上 rate limiting
Constraints: 不修改 DB schema、auth flow 不變、使用已設定的 Redis
Acceptance: 每 IP 每分鐘 5 次請求，超過返回 429；現有測試通過；新增 rate limit 測試案例
```

### 推動更好的結果

```
「Grill me on these changes and don't make a PR until I pass」
→ 讓 Claude 當 reviewer，你通過後才建 PR

「Knowing everything you know now, scrap this and implement the elegant solution」
→ 放棄第一個方案，要求 Boris 認可的優雅解法
```

---

## 十、設定簡約哲學

Boris 強調：**Claude Code 不需要大量客製化就能高效運作。**

他個人設定的清單：
- ✅ iTerm2 系統通知
- ✅ 5 個 worktree 並行
- ✅ CLAUDE.md（team-maintained）
- ✅ PostToolUse 自動格式化 hook
- ✅ `/commit-push-pr`、`/go` 等少數 slash commands
- ✅ Slack / BigQuery / Sentry MCP
- ❌ 沒有複雜的自定義 agent
- ❌ 沒有大量的 settings.json 調整

> 「我的設定可能出乎意料地樸素。Claude Code 開箱即用效果很好，我個人不太客製化。」

---

## 相關頁面

- [[複利工程思維]] — 本工作流的核心哲學
- [[Boris]] — 人物簡介與四大心法
- [[CoWork桌面工具指南]] — GUI 版本的工作方式
- [[CLAUDE.md撰寫最佳實踐]] — CLAUDE.md 深度指南
- [[Claude Code Hooks 深度設定]] — Hooks 完整說明
- [[Claude MCP 伺服器整合指南]] — MCP 整合教學
- [[Claude Code多人團隊協作指南]] — 團隊共用設定
- [[生產環境Vibe Coding四大策略]] — Erik Schluntz 的互補策略
