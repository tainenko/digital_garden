---
title: AI Agent 核心架構：Model + Context + Tools
type: concept
tags: [ai-agent, architecture, context-engineering, tools, mcp, rag, multi-agent, 框架]
created: 2026-05-07
updated: 2026-05-07
sources: [s09g-ai-agent-bootcamp]
---

# AI Agent 核心架構：Model + Context + Tools

## 核心公式

```
Agent = Model + Context + Tools
```

由 [[Bojie Li (s09g)]]（Pine AI Chief Scientist）在「AI Agent 實戰營」中提出的統一框架。

---

## 三個組成部分

### Model（模型）— 決策大腦

- 提供理解與推理能力
- LLM 透過先驗知識比傳統 RL（Q-learning）高出 **250–400×** 的樣本效率
- 不同任務選擇不同模型（推理型 vs 快速型）

### Context（上下文）— 作業系統

Context 是 Agent 的「作業系統」，包含：

```
Context = 系統指令 + 對話歷史 + 推理鏈 + 工具呼叫記錄 + 用戶記憶 + 知識庫片段
```

**Context 的關鍵挑戰**：
- KV Cache 效率：不良設計讓延遲/成本暴增（→ [[Context Engineering最佳實踐]]）
- 長期記憶：跨 session 的用戶偏好與行為記錄
- 壓縮策略：在有限 context window 內保留最有用的資訊

### Tools（工具）— 雙手

工具讓 Agent 從文字輸出擴展到真實行動。三大類：

| 類別 | 範例工具 |
|------|---------|
| **感知工具** | 網路搜尋、多模態理解、檔案讀取、財經數據（Yahoo Finance）、天氣（Open-Meteo）|
| **執行工具** | 代碼解釋器、虛擬終端、檔案操作、Shell 管理 |
| **協作工具** | Browser Use 自動化、Email/Telegram/Slack 通知、人工確認（Human-in-the-Loop）|

---

## Agent 的七個層次（從簡到複雜）

1. **Simple LLM Call** — 一問一答
2. **Tool Use** — 單輪工具呼叫
3. **ReAct Loop** — 推理→行動→觀察迴圈
4. **RAG Agent** — 知識庫輔助
5. **Coding Agent** — 程式生成與執行
6. **Multi-Agent** — Agent 間協作
7. **Self-Evolving Agent** — 從經驗學習更新策略

---

## 與 MCP 的關係

MCP（Model Context Protocol）是工具層的標準化協定：
- MCP Server 提供封裝好的工具集
- Agent 透過 MCP 連接外部服務（GitHub、資料庫、瀏覽器等）
- 詳見 [[Claude MCP 伺服器整合指南]]

---

## Agent 評測

Agent 能力可用以下基準衡量（→ [[AI Agent評測基準]]）：
- **SWE-bench**：解決真實 GitHub issue
- **GAIA**：通用 AI 助理任務
- **Terminal-Bench**：終端機自動化
- **OSWorld**：完整 OS 操作

---

## 相關頁面

- [[Context Engineering最佳實踐]] — Context 設計深度指南
- [[AI Agent評測基準]] — 如何量化 Agent 能力
- [[Claude Code內部運作機制]] — Tool Use 底層機制
- [[Claude MCP 伺服器整合指南]] — 工具標準化協定
- [[RAG檢索增強生成實戰]] — 知識庫整合
- [[LangGraph Agent工作流設計]] — 工作流框架
- [[Bojie Li (s09g)]] — 框架提出者
