---
title: Agent Harness 實作
type: concept
tags: [agent-harness, react-loop, tool-use, typescript, architecture]
created: 2026-06-26
updated: 2026-06-29
sources: [alienzhou-zero2agent, github-codejunkie99-agentic-harness]
---

# Agent Harness 實作

## 定義

**Agent Harness** 是「宿主層」——把模型、循環（如 [[ReAct Pattern|ReAct]]）、工具調用與運行上下文組合起來，讓 Agent 能穩定執行任務的框架骨架。它不是 Agent 本身，而是讓 Agent 能跑起來的基礎設施。

## 三層核心組件

### 1. Loop（循環引擎）

實現 ReAct 的 Thought → Action → Observation 閉環：

1. 把當前對話歷史 + 工具清單送給 LLM
2. 收到 `tool_use` block → 執行對應工具，把結果作為 `tool_result` 加回歷史
3. 收到純文字（Final Answer）→ 結束循環
4. 重複 1–3

### 2. Tools（工具注冊表）

每個工具具備：
- **JSON Schema 定義**：供 LLM 理解如何呼叫
- **執行函數**：`execute(input, ctx: ToolContext) → string`
- **錯誤處理**：工具失敗時返回 `tool_result` 而非拋出，讓 LLM 自行決定後續步驟

Zero2Agent 已實作的工具：

| 工具 | 功能 | 底層 |
|------|------|------|
| `read_file` | 讀取檔案內容 | Node.js fs |
| `list_directory` | 列出目錄結構 | Node.js fs |
| `grep_search` | 內容搜索（正則） | ripgrep `--json` |
| `find_files` | 文件名模式搜索 | ripgrep `--files` |

### 3. LLM Client（模型抽象）

統一封裝模型調用，支援流式輸出（`messages.stream()`）和事件回調（`LoopEventHandlers`）。[[Anthropic]] SDK 為主，設計上可替換為其他提供商。

## 架構分層（Zero2Agent 實作）

```
CLI (tui)           ← 用戶交互、輸入輸出、工具調用摘要展示
    ↓
Agent Core          ← ReAct 循環、ToolContext 注入、歷史管理
    ↓
Tools               ← grep_search、find_files、read_file 等
    ↓
LLM API             ← Anthropic SDK（流式）
```

## 關鍵設計決策

- **選 ripgrep 不選 RAG**：效果（精確匹配）+ 成本（零向量化）+ 可控性（結果可解釋）
- **[[ToolContext模式]]**：顯式注入 `cwd` 等運行環境，避免隱式 `process.cwd()`
- **相對路徑輸出**：所有工具輸出相對路徑，省 token + 工具鏈銜接一致
- **流式輸出 + 事件回調**：`onToolStart/onToolEnd` 讓 TUI 即時顯示進度

## 與其他概念的對比

| 概念 | 定位 |
|------|------|
| [[AI Agent核心架構 Model+Context+Tools]] | 理論架構層次 |
| Agent Harness 實作 | 工程落地層次 |
| Claude Code / Codex | 成熟產品，Zero2Agent 以此為對標 |
| LangGraph / LangChain | 高層框架，Zero2Agent 是從頭實作底層 |
| [[github-codejunkie99-agentic-harness\|agentic-harness]] | **Rust 版**宿主層：單一 binary 跑遍 laptop/CI/沙盒/邊緣，編譯期型別安全；與 Zero2Agent（TypeScript）形成語言對照 |

## 語言路線對比：TypeScript vs Rust

同樣是「從零實作 Agent Harness」，兩條開源路線給出相反的工程取捨：

| 維度 | [[alienzhou-zero2agent\|Zero2Agent]]（[[alienzhou]]） | [[github-codejunkie99-agentic-harness\|agentic-harness]]（[[codejunkie99]]） |
|------|------|------|
| 語言 | TypeScript（pnpm monorepo） | Rust（Cargo） |
| 賣點 | 生態成熟、迭代快、教學透明 | 單一可攜 binary、編譯期型別安全、無 JS |
| 部署目標 | Node 環境 | native / CI / 遠端 Linux 沙盒 / Cloudflare Workers |
| 取向 | 教學課程（設計決策全公開） | 成品框架（SDK + CLI + HTTP server） |
| MCP | 規劃中 | `McpServerOptions` 內建掛載 |

## 學習資源

- [[alienzhou-zero2agent|Zero2Agent]] — 最完整的從零實作教學課程，含設計文件、VibeCoding 實錄
- [[claude-code-book|Claude Code 深度剖析]] — Agent Harness 內部機制的深度解析（繁體中文 PDF）
- [[Claude Code內部運作機制]] — Tool Use 循環與 MCP 整合

## 相關頁面

- [[ReAct Pattern]]
- [[ToolContext模式]]
- [[Vibe Coding基礎概念]]
- [[Spec驅動開發]]
