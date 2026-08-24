---
title: Claude Code 多人團隊協作指南
type: concept
tags: [claude-code, team, collaboration, workflow, onboarding]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Claude Code 多人團隊協作指南

## 團隊使用 Claude Code 的挑戰

| 挑戰 | 問題描述 |
|------|---------|
| 行為不一致 | 不同成員得到不同品質的輸出 |
| CLAUDE.md 衝突 | 每人有自己的設定，覆蓋彼此 |
| 知識不共享 | 一人找到好的 Prompt 技巧，其他人不知道 |
| 成本失控 | 不知道哪些操作消耗大量 token |
| 安全邊界不清 | 不知道 Claude 被允許做什麼 |

解法：**統一設定 + 明確邊界 + 知識文件化**

---

## 分層設定架構

```
~/.claude/settings.json        ← 個人全域設定（不提交）
  ├── 個人偏好（主題、語言）
  └── 個人 API key

<repo>/.claude/settings.json   ← 專案共享設定（提交到 git）
  ├── 共享 MCP servers
  ├── 共享 permissions
  └── 共享 hooks

<repo>/.claude/settings.local.json  ← 個人覆蓋（加入 .gitignore）
  └── 個人工具路徑、本機測試設定
```

### 團隊共享 settings.json 範例

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "${DATABASE_URL}"]
    }
  },
  "permissions": {
    "allow": [
      "Bash(git:*)",
      "Bash(go test:*)",
      "Bash(pytest:*)",
      "Bash(uv run:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(git push --force:*)",
      "Bash(git push origin main:*)"
    ]
  }
}
```

---

## 團隊 CLAUDE.md 規範

### 分層結構

```
<repo>/CLAUDE.md                     ← 主規範（所有成員遵守）
<repo>/backend/CLAUDE.md             ← 後端子目錄規範
<repo>/frontend/CLAUDE.md            ← 前端子目錄規範
<repo>/docs/CLAUDE.md                ← 文件目錄規範
```

### 主 CLAUDE.md 必包含內容

```markdown
# 專案說明

## 技術棧
- 語言：Go 1.24 + Python 3.12
- 資料庫：PostgreSQL 16（sqlc 生成 query）
- 快取：Redis 7
- CI：GitHub Actions

## 禁止操作
- 不修改 `core/engine/` 目錄（需 PR + 2 人 review）
- 不直接 push main 分支
- 不硬編碼任何密鑰或密碼
- 不安裝未在 pyproject.toml/go.mod 中的套件

## 必須遵守
- 所有新功能必須有測試（Coverage > 80%）
- 跑 `make lint` 確認通過再 commit
- PR 描述必須包含測試說明

## 命名規範
- Go：標準 Go 規範（godoc 可見函數用大寫）
- Python：snake_case，類型用 PascalCase
- DB Table：snake_case（orders, order_items）
- API Endpoint：kebab-case（/user-profiles）

## 測試規範
- Go：`_test.go` 在同目錄
- Python：`tests/` 目錄，`test_` 前綴
- 整合測試：`integration/` 目錄，需要 DB
```

---

## Subagents 團隊用法

### 共享 Subagent 定義

```yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: 專案代碼審查 Agent，按照團隊規範進行 Review
model: claude-opus-4-7
tools:
  - Read
  - Bash
---

你是這個專案的資深工程師，正在進行 Code Review。

評審重點（依照 CLAUDE.md 規範）：
1. 安全性：是否有 SQL Injection、敏感資料洩露、硬編碼密鑰
2. 測試：是否有足夠的測試覆蓋，邊界條件是否處理
3. 效能：是否有 N+1 查詢、不必要的 DB 呼叫
4. 可讀性：命名是否清晰、函數是否過長（> 50 行）
5. 符合規範：是否遵守 CLAUDE.md 中的規定

輸出格式：
- 🔴 MUST FIX：必須修改（安全/功能問題）
- 🟡 SHOULD FIX：建議修改（品質問題）
- 💡 SUGGESTION：可考慮的改善
- ✅ GOOD：值得稱讚的設計
```

```yaml
# .claude/agents/doc-writer.md
---
name: doc-writer
description: 根據程式碼自動生成 API 文件和 README
model: claude-sonnet-4-6
tools:
  - Read
  - Write
---

你是技術文件撰寫專家...
```

### 啟動 Subagent

```bash
# 任何團隊成員都可以用相同的 code-reviewer
claude "review 這個 PR 的變更" --subagent code-reviewer

# 或在對話中：
/agent code-reviewer
```

---

## 團隊 Onboarding 流程

### 新成員入職清單

```markdown
## Claude Code 環境設定

### 第一步：安裝
- [ ] `npm install -g @anthropic-ai/claude-code`
- [ ] 取得 Anthropic API Key（向 DevOps 申請）
- [ ] `claude config set apiKey <key>`

### 第二步：專案設定
- [ ] clone 專案（`.claude/settings.json` 自動生效）
- [ ] 複製 `.claude/settings.local.example.json` 為 `.claude/settings.local.json`
- [ ] 設定本機環境變數（DATABASE_URL, GITHUB_TOKEN）

### 第三步：確認可用
- [ ] `claude "列出這個專案的主要模組"` — 確認能讀取專案
- [ ] `claude "跑一下測試"` — 確認 Bash 工具可用
- [ ] `claude "用 code-reviewer subagent review 最新的 commit"` — 確認 subagent 可用

### 第四步：閱讀
- [ ] 閱讀 `CLAUDE.md`（了解禁止操作）
- [ ] 閱讀 `.claude/agents/` 目錄（了解可用 Subagents）
- [ ] 閱讀 `docs/claude-tips.md`（團隊累積的 Prompt 最佳實踐）
```

---

## 知識管理：累積 Prompt 最佳實踐

### 建立團隊 Prompt 知識庫

```markdown
# docs/claude-tips.md

## 高效 Prompt 技巧

### Debug 神咒
「這個 bug 我查了 1 小時沒找到，幫我從這幾個方向系統性排查：
 1. 輸入資料是否符合預期（加 log 看）
 2. 中間件/攔截器是否改變了資料
 3. 資料庫查詢結果是否正確（直接執行 SQL 看）
 4. 是否有快取問題
 5. 並發問題（race condition）」

### Code Review 咒語
「假設你是一位經驗豐富的 Go 工程師，
 對以下代碼做 Code Review，特別注意：
 - goroutine 洩漏
 - context 正確傳遞
 - error 是否被正確處理（不被 swallow）」

### 架構討論咒語
「我要做 [功能]，需要在以下方案中選擇：
 - 方案 A: [描述]
 - 方案 B: [描述]
 請從可維護性、效能、測試難度三個維度比較，
 並給出推薦，以及你考慮了哪些因素。」
```

---

## 成本控制

### 估算 Token 消耗

```
一般對話：1K–5K tokens/次
代碼審查（小 PR）：10K–30K tokens
大型重構：50K–200K tokens
長對話（context 累積）：可能超過 200K tokens
```

### 降低成本策略

```bash
# 1. /clear 重設 context（不帶著不相關的歷史）
/clear

# 2. 使用更小的模型做簡單任務（在 CLAUDE.md 說明）
# 「程式碼格式化、簡單 lint 修復，使用 claude-haiku-4-5」

# 3. Subagent 限制模型
# 在 .claude/agents/formatter.md 設定 model: claude-haiku-4-5

# 4. 大型任務拆小
# 「先只處理 auth 模組，不要動其他地方」
```

---

## 常見團隊問題

**Q: 如何確保每人的 Claude 行為一致？**
A: 所有規範寫入 `.claude/settings.json`（共享）和 `CLAUDE.md`（提交到 git）；個人設定只放 `settings.local.json`（忽略）。

**Q: 如何防止 Claude 動到不該動的地方？**
A: `CLAUDE.md` 明確列出禁止操作；`settings.json` 的 `deny` 規則限制危險指令；關鍵目錄加 `CODEOWNERS` 強制 review。

**Q: 如何分享一個好用的 Prompt？**
A: 加入 `docs/claude-tips.md` 並提 PR，讓所有人受益。

---

## 相關頁面

- [[CLAUDE.md撰寫最佳實踐]] — CLAUDE.md 詳細撰寫規範
- [[Claude Code Hooks 深度設定]] — 自動化 CI 整合
- [[Claude Code Subagents 完整指南]] — Subagent 設計
- [[Claude Code與Git整合工作流]] — Git 操作規範
