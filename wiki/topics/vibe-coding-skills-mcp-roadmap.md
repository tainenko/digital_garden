---
title: Vibe Coding + Skills + MCP 學習路線圖
type: topic
tags: [claude, vibe-coding, skills, mcp, 學習路線, roadmap]
created: 2026-04-30
updated: 2026-04-30
sources: [bnext-anthropic-academy-13-courses.md]
---

# Vibe Coding + Skills + MCP 學習路線圖

針對想要掌握 **Vibe Coding（AI 輔助開發）**、**Agent Skills（可複用技能）**、**MCP（工具整合協定）** 三個面向的學習者設計。

整條路線大約 **3–4 個月**，每週投入 5–8 小時。

---

## 路線全覽

```
Week 1–2    Week 3–4    Week 5–6    Week 7–10   Week 11–14
   │            │            │            │            │
[基礎]      [Vibe]       [Skills]     [MCP]      [整合實戰]
   │            │            │            │            │
概念建立    Claude Code  SKILL.md    MCP Server  個人專案
工具安裝    日常使用     團隊工作流   連接外部工具   完整系統
```

---

## Phase 1：概念與工具基礎（Week 1–2）

**目標**：搞清楚每個概念是什麼、安裝好所有工具。

### 觀念建立

先讀懂這三個概念：

| 概念 | 一句話定義 | 推薦頁面 |
|------|-----------|---------|
| Vibe Coding | 用自然語言驅動 AI 寫程式，工程師從執行者轉為決策者 | [[Vibe Coding基礎概念]] |
| Skills | 可複用的 Claude 技能定義，類似函數但以 `SKILL.md` 描述 | [[Superpowers技能框架]] |
| MCP | 讓 Claude 連接外部工具的標準協定（資料庫、API、檔案系統） | [[Claude MCP 伺服器整合指南]] |

### 環境安裝

```bash
# 1. 安裝 Claude Code CLI
npm install -g @anthropic-ai/claude-code

# 2. 設定 API Key
export ANTHROPIC_API_KEY="sk-ant-..."

# 3. 驗證安裝
claude --version
```

### 官方課程（這週完成）

✅ **[AI Fluency: Framework & Foundations](https://academy.anthropic.com)**（1.1 小時）
- 建立對 AI 能力/限制的正確認知
- 學習 4D 框架：Delegate / Describe / Discern / Diligent
- 開始用 Claude 做日常任務

---

## Phase 2：Vibe Coding 實戰（Week 3–4）

**目標**：能用 Claude Code 完成一個完整的小功能，包含測試。

### 官方課程（這週完成）

✅ **[Claude Code in Action](https://academy.anthropic.com)**（~1 小時，15 堂課）
- 檔案操作與指令執行
- CLAUDE.md 管理專案上下文
- Hooks 自動化（格式化、測試）
- GitHub 整合

### 實作練習

**練習一：建立你的第一個專案 CLAUDE.md**

```markdown
# My Project — Claude 指令

## 技術棧
（你自己的技術棧）

## 測試
每次修改後執行 `xxx test`

## 禁止事項
- 不修改 config/ 目錄
```

**練習二：TDD 驅動開發**

```
你：「先幫我寫一個函數的測試：輸入 URL，
     回傳該頁面的 title，測試要包含正常情況和失敗情況」

Claude：（寫測試）

你：「好，現在讓測試通過」

Claude：（寫實作）
```

**練習三：用 Hooks 建立自動品質門禁**

參考 [[Claude Code Hooks 深度設定]]，設定：
- PostToolUse 自動格式化
- PostToolUse 自動執行測試

### 生產策略（必讀）

讀完 [[生產環境Vibe Coding四大策略]]，特別記住：
1. **可驗證抽象層**：先寫測試，AI 讓測試通過
2. **葉節點策略**：讓 AI 處理末端功能，保護核心邏輯
3. **前置規劃**：大任務先討論計畫再執行
4. **充分上下文**：告訴 Claude 相關檔案在哪

---

## Phase 3：Agent Skills 工作流（Week 5–6）

**目標**：建立至少 3 個可複用的 Skill，讓 Claude 在你的專案中有「專業分工」。

### 官方課程（這週完成）

✅ **[Introduction to Agent Skills](https://academy.anthropic.com)**（堂數未知）
- SKILL.md 撰寫格式
- 漸進式揭露（保持 context 效率）
- `allowed-tools` 設定
- 技能共用與管理

### SKILL.md 基礎格式

```markdown
# skill-name

## 觸發條件
當用戶要求 [做某件事] 時使用這個技能

## 步驟
1. 先讀取 [某些檔案]
2. 分析 [某些事項]
3. 執行 [某些操作]

## 限制工具
allowed-tools: Read, Write, Bash(go test:*)

## 輸出格式
[描述期望的輸出格式]
```

### 建議建立的 3 個入門 Skills

| Skill 名稱 | 觸發場景 | 允許的工具 |
|-----------|---------|-----------|
| `code-review` | 請 Claude 審查程式碼 | Read, Bash(lint) |
| `write-test` | 請 Claude 為某函數寫測試 | Read, Write, Bash(test) |
| `refactor` | 請 Claude 重構某個模組 | Read, Write |

### 進階參考

參考 [[Superpowers技能框架]] 的設計理念：
- TDD RED → GREEN → REFACTOR 循環
- 多 Agent Code Review（生成 + 審查 分離）
- Git Worktree 隔離實驗性修改

---

## Phase 4：MCP 工具整合（Week 7–10）

**目標**：建立至少一個自訂 MCP Server，讓 Claude 能存取你的核心資料。

### 官方課程（分兩步完成）

✅ **[Introduction to MCP](https://academy.anthropic.com)**（~1 小時，16 堂課）
- MCP 架構原理
- 用 Python SDK 建 Server
- MCP Inspector 測試

✅ **[MCP: Advanced Topics](https://academy.anthropic.com)**（~1.1 小時，15 堂課）
- 日誌 + 進度通知
- 雙向通訊
- Roots 權限模型
- stdio vs HTTP 傳輸

### MCP 學習路徑

**Week 7**：使用現成 MCP Server
```json
// .claude/settings.json
{
  "mcpServers": {
    "sqlite": {
      "command": "npx",
      "args": ["@anthropic/mcp-server-sqlite", "--db-path", "./dev.db"]
    }
  }
}
```
目標：讓 Claude 能直接查你的 SQLite 資料庫。

**Week 8**：建立第一個自訂 MCP Server

選一個你日常需要的功能，用 Python SDK 建 MCP Server：

```python
from mcp.server import Server
from mcp.server.stdio import stdio_server

server = Server("my-first-mcp")

@server.tool()
async def get_data(query: str) -> str:
    """從自訂資料來源取得資料"""
    # 你的邏輯
    return result

async def main():
    async with stdio_server() as streams:
        await server.run(*streams)
```

**Week 9–10**：進階主題
- 加入日誌記錄（Logging）
- 設定 Roots 限制存取範圍
- 實作雙向通訊

參考：[[Claude MCP 伺服器整合指南]]

---

## Phase 5：整合實戰專案（Week 11–14）

**目標**：用學到的所有工具，完成一個對你有實際價值的完整專案。

### 建議專案類型

選一個符合你實際需求的：

| 專案類型 | 涉及的知識點 |
|---------|------------|
| **AI 代碼審查機器人** | Claude Code + Skills（code-review）+ GitHub MCP |
| **個人知識庫問答系統** | Claude API + RAG + 自訂 MCP（查詢知識庫） |
| **資料分析 Agent** | Claude API + Tool Use + PostgreSQL MCP |
| **股票分析工具** | Claude API + Tool Use + 自訂 MCP（台股資料）|

### 整合實戰清單

```
□ 建立完整 CLAUDE.md（禁止事項 + 技術棧 + 測試指令）
□ 設定 3+ 個 Hooks（格式化 + 測試 + 通知）
□ 建立 2–3 個 Skills（對應你的常見工作場景）
□ 連接至少一個 MCP Server（本地或雲端資料來源）
□ 全部有 unit test 覆蓋核心邏輯
□ 用 Vibe Coding 完成 80%+ 的實作
```

---

## 選修：Claude API 深度開發（選擇性）

如果你想把 Claude 整合進自己的產品（不只是 Claude Code 工作流），加修：

✅ **[Building with the Claude API](https://academy.anthropic.com)**（84 堂，8.1 小時）

涵蓋：
- Prompt Caching（節省 90% Token 成本）
- RAG 系統設計
- Agent 架構
- API 的 MCP 整合

參考：[[Claude API基礎與最佳實踐]]、[[Claude Agent 設計模式]]

---

## 時間規劃速查

| 週次 | 主題 | 官方課程 | 實作目標 |
|------|------|---------|---------|
| 1–2 | 概念 + 工具安裝 | AI Fluency（1.1h） | 環境搭好，會基本對話 |
| 3–4 | Vibe Coding | Claude Code in Action（1h） | 完成一個有測試的小功能 |
| 5–6 | Agent Skills | Agent Skills（?h） | 建立 3 個可複用 Skill |
| 7–8 | MCP 入門 | MCP Intro（1h） | 連接一個外部資料源 |
| 9–10 | MCP 進階 | MCP Advanced（1.1h） | 完成生產級 MCP Server |
| 11–14 | 整合實戰 | — | 完整個人專案 |

總官方課程時數：約 **5–6 小時**（不含 API 深度開發的 8.1 小時）

---

## 學習資源索引

### Wiki 內部頁面

| 主題 | 頁面 |
|------|------|
| Vibe Coding 概念 | [[Vibe Coding基礎概念]]、[[Vibe Coding風險與限制]] |
| Claude Code 操作 | [[Claude Code 入門完整指南]]、[[Claude Code Hooks 深度設定]] |
| CLAUDE.md 設定 | [[CLAUDE.md撰寫最佳實踐]] |
| Prompt 技巧 | [[Claude Prompt工程核心技巧]] |
| Skills 設計 | [[Superpowers技能框架]]、[[Spec驅動開發]] |
| MCP 整合 | [[Claude MCP 伺服器整合指南]] |
| API 開發 | [[Claude API基礎與最佳實踐]] |
| Agent 設計 | [[Claude Agent 設計模式]] |
| 生產最佳實踐 | [[生產環境Vibe Coding四大策略]] |
| 全部課程說明 | [[Anthropic Academy 13堂課程完整指南]] |

### 外部資源

| 資源 | 說明 |
|------|------|
| [Anthropic Academy](https://academy.anthropic.com) | 官方課程平台，全免費 |
| [MCP 官方文件](https://modelcontextprotocol.io) | MCP 協定規格與 SDK |
| [Claude Code 文件](https://docs.anthropic.com/claude-code) | Claude Code 官方文件 |
| [MCP Server 社群清單](https://github.com/modelcontextprotocol/servers) | 現成 MCP Server 集合 |
