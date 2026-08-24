---
title: Claude Code Subagents 完整指南
type: concept
tags: [claude-code, subagents, agent, AGENTS.md, 自動化, 任務委派]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Claude Code Subagents 完整指南

Subagents 是 Claude Code 的「任務委派」機制：當主對話需要做一件耗時、會佔用大量 context 的側邊任務（搜尋、探索、審查），可以把它交給一個獨立的 Claude 實例處理，只收回摘要結果。

> 「Subagent 做完那個任務後，只把結論帶回來，探索過程中產生的幾千行搜尋結果留在它自己的 context window 裡。」

---

## Subagents vs Agent Teams vs Skills

| | Subagents | Agent Teams | Skills |
|--|-----------|-------------|--------|
| **運作方式** | 在同一個 session 內委派任務 | 多個 Claude session 並行協作 | 可呼叫的工作流程指令 |
| **適合場景** | 單一側邊任務（探索、審查）| 大型並行工程任務 | 可重複的標準操作 |
| **Context 隔離** | 完全隔離 | 透過 git 協調 | 載入到主對話 |
| **Subagent 能再生 Subagent** | 否（禁止巢狀） | 可 | 否 |

---

## 內建 Subagents

Claude Code 內建三個常用 subagent，Claude 會自動判斷何時呼叫：

### Explore（探索）
- **模型**：Haiku（快速、低延遲）
- **工具**：只讀工具（禁止 Write、Edit）
- **用途**：搜尋代碼庫、定位文件、理解架構

Claude 呼叫 Explore 時會指定徹底程度：
- `quick`：單一目標查詢
- `medium`：適度探索
- `very thorough`：跨多個位置的全面分析

### Plan（規劃）
- **模型**：繼承主對話
- **工具**：只讀工具
- **用途**：在 Plan Mode 下收集 context 後再呈現計畫

### General-purpose（通用）
- **模型**：繼承主對話
- **工具**：全部工具
- **用途**：需要探索 + 修改、複雜推理、多步驟任務

---

## 自定義 Subagents

### 什麼時候值得建一個自定義 subagent

當你發現自己不斷用相同的指令召喚相同類型的 Claude 來做相同的任務，就值得把它定義成 subagent。

```
✅ 適合定義成 subagent：
- 每次 PR 完成都跑的 Code Review
- 每次修改 API 後都要跑的文件生成
- 每次處理資料前都要做的 SQL 驗證

❌ 不需要定義成 subagent：
- 偶爾才做的一次性任務
- 任務每次都不一樣
```

### 建立方式一：`/agents` 命令（推薦）

```
/agents
```

開啟 Subagent 管理介面後：
1. 切換到 **Library** 標籤
2. 選 **Create new agent**
3. 選擇範圍：**Personal**（用戶級）或 **Project**（專案級）
4. 選 **Generate with Claude** 後輸入描述，Claude 會自動生成設定

### 建立方式二：手動建立 Markdown 檔案

Subagent 是有 YAML frontmatter 的 Markdown 檔案，存放在特定目錄：

```
~/.claude/agents/        ← 個人層（所有專案可用）
.claude/agents/          ← 專案層（提交到 git，團隊共用）
```

#### Frontmatter 完整欄位

```yaml
---
name: code-reviewer
description: >
  Expert code reviewer. Proactively invoked after any code modification.
  Checks for security issues, performance problems, and style violations.
model: sonnet          # haiku | sonnet | opus（預設繼承主對話）
tools:                 # 省略 = 繼承主對話全部工具
  - Read
  - Grep
  - Glob
  - Bash
disallowedTools:       # 明確禁止的工具
  - Write
  - Edit
permissionMode: default  # default | acceptEdits | bypassPermissions | plan
maxTurns: 10           # 最大輪次（防止無限循環）
memory: user           # user（持久記憶）| none
skills:                # 可呼叫的 Skills
  - .claude/skills/coding-style.md
---

You are a senior code reviewer with 10 years of experience.

For each file changed, check:
1. Security: SQL injection, XSS, auth bypass
2. Performance: N+1 queries, unnecessary allocations
3. Style: follows the project's CLAUDE.md conventions

Return a structured report: severity (high/medium/low), location (file:line), description, suggestion.
```

#### 範例：Go 程式碼審查 subagent

```yaml
---
name: go-reviewer
description: >
  Go code reviewer. Use after editing any .go files.
  Checks for Go idioms, race conditions, error handling, and test coverage.
model: sonnet
tools:
  - Read
  - Grep
  - Glob
  - Bash
disallowedTools:
  - Write
  - Edit
maxTurns: 15
---

You are a senior Go engineer. After code changes, review the modified Go files for:

1. **Race conditions**: look for shared state without mutex or channel
2. **Error handling**: every error must be handled or explicitly ignored with `_`
3. **Go idioms**: prefer `fmt.Errorf("context: %w", err)` over bare returns
4. **Test coverage**: check if new functions have corresponding `_test.go` tests

Output: markdown table with columns [File, Line, Severity, Issue, Suggestion]
```

---

## 關鍵設計原則

### Description 是觸發機制

Claude 根據每個 subagent 的 `description` 決定是否委派任務。寫得好的 description 讓 Claude 知道「什麼時候該用這個 subagent」：

```yaml
# ❌ 太模糊
description: Helps with code

# ✅ 清楚說明觸發條件
description: >
  Go code quality reviewer. Proactively invoke after any modification
  to .go files to check for race conditions, error handling issues,
  and violations of Go idioms.
```

### 選對 Model

| 任務類型 | 建議 Model |
|---------|-----------|
| 只是搜尋、grep、讀文件 | Haiku（速度快、成本低） |
| 需要理解複雜邏輯 | Sonnet |
| 需要複雜推理或重構 | Opus |

### 工具最小化原則

只給 subagent 它真正需要的工具，理由：
- **安全**：審查類 subagent 不應該有 Write 權限
- **清晰**：工具限制本身就是 subagent 行為邊界的定義
- **成本**：工具清單越短，系統提示越小

```yaml
# 只讀審查 subagent
tools:
  - Read
  - Grep
  - Glob
  - Bash    # 僅用於 go vet、golangci-lint 等靜態分析

disallowedTools:
  - Write
  - Edit
```

---

## Explore-Plan-Execute 三段式模式

這是 2026 年最推薦的複雜任務 subagent 模式：

```
Phase 1: Explore（探索）
├── 只讀 subagent
├── 探索代碼庫，建立地圖
└── 輸出：「現有架構摘要」

Phase 2: Plan（規劃）
├── 只讀 subagent
├── 基於探索結果，規劃修改方案
└── 輸出：「修改計畫（檔案 + 函數 + 風險）」

Phase 3: Execute（執行）
├── 全工具 subagent
├── 依計畫實施修改
└── 輸出：「完成報告」
```

這個模式的關鍵是**每個階段有乾淨的交接**：前一個 subagent 的輸出作為下一個的輸入，而不是把所有探索過程的 context 都傳遞下去。

---

## Subagent 的持久記憶

開啟 `memory: user` 後，subagent 擁有自己的記憶目錄：

```
~/.claude/agent-memory/<agent-name>/
├── MEMORY.md       # 索引，每次 session 開始載入前 200 行
├── patterns.md     # 觀察到的代碼模式
└── issues.md       # 歷史發現的問題類型
```

適合需要「越用越聰明」的 subagent，例如：
- Code reviewer：記住這個專案常見的問題類型
- Test generator：記住這個專案的測試風格
- Documentation writer：記住這個專案的文件格式

---

## 用 CLI Flag 臨時定義

不想建檔案，快速測試時可以用 `--agents` flag：

```bash
claude --agents '{
  "quick-reviewer": {
    "description": "Quick code reviewer for this session",
    "prompt": "You are a code reviewer. Check for obvious bugs only.",
    "tools": ["Read", "Grep"],
    "model": "haiku"
  }
}'
```

這個 subagent 只存在於這個 session，不會寫入磁碟。

---

## 常見錯誤

| 問題 | 原因 | 解法 |
|------|------|------|
| Subagent 沒被自動呼叫 | Description 太模糊 | 加入觸發條件（「何時用」）和任務範圍 |
| Subagent 結果不完整 | MaxTurns 太低 | 調高 `maxTurns`（預設 10） |
| Subagent 修改了它不該碰的文件 | 工具沒有限制 | 加入 `disallowedTools: [Write, Edit]` |
| 想用 subagent 但巢狀呼叫 | Subagent 無法生成 subagent | 用 Agent Teams 代替 |

---

## 與 Skills 的搭配

Subagent 可以呼叫 Skills：

```yaml
---
name: feature-builder
description: Builds new features end-to-end
skills:
  - .claude/skills/create-handler.md
  - .claude/skills/create-test.md
  - .claude/skills/update-docs.md
---

When given a feature description, use the skills in order:
1. create-handler: build the API handler
2. create-test: write tests for it
3. update-docs: update the relevant docs
```

---

## 相關頁面

- [[Claude Code 入門完整指南]]
- [[Claude Agent 設計模式]]（API 層的 agent 設計）
- [[Claude Code Hooks 深度設定]]（配合 hooks 做自動觸發）
- [[Skills實戰：Threads自動爬文與發文]]（Skills 實戰範例）
- [[Claude MCP 伺服器整合指南]]（給 subagent 接 MCP 工具）
