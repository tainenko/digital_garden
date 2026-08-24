---
title: "Zero2Agent：從零實現產品級 Agent Harness"
type: source-summary
tags: [agent-harness, vibe-coding, typescript, react-loop, tool-use, sdd]
created: 2026-06-26
updated: 2026-06-26
sources: [alienzhou-zero2agent]
---

# Zero2Agent：從零實現產品級 Agent Harness

## Origin

- **標題**：Zero2Agent — A hands-on course on building an AI Agent Harness from scratch
- **作者**：alienzhou（GitHub handle）
- **URL**：https://github.com/alienzhou/zero2agent
- **語言**：TypeScript（pnpm monorepo），文件以中文為主
- **授權**：MIT

## Key Takeaways

- **定位**：這是一個教學課程 repo，不是現成工具。目標是帶讀者從第一行程式碼開始，實現類似 Claude Code / Codex 的 **Agent Harness**（循環調度 + 工具調用 + 上下文管理）
- **差異化**：市面學習資源要嘛只講概念，要嘛只有最終代碼。Zero2Agent 完整公開「為什麼這樣做」的設計決策——含需求討論記錄（`.discuss/`）、VibeCoding 對話實錄（`.vibecoding/`）、復盤筆記（`retros/`）
- **[[ReAct Pattern]] 實作**：E01-S001 跑通最基礎的 ReAct 循環（Thought→Action→Observation），以 Anthropic SDK Tool Use 為底層。這是整個課程的心跳，後續所有迭代都建在這上面
- **工具設計四問框架**：E01-S002 展示如何設計 Agent 工具——「解決什麼問題→控制什麼/自動化什麼→輸出契約→邊界兜底」。結論：對人好用 = 對 AI 好用
- **ToolContext 模式**：E01-S003 發現第三個工具（`find_files`）進來時，前兩個工具的隱式假設（`process.cwd()`）被暴露。引入 `ToolContext` 作為工具運行環境的顯式注入，成為後續所有工具的擴展點
- **ripgrep 整合**：`grep_search` 使用 `@vscode/ripgrep`（`--json` 模式解析）；`find_files` 使用 `--files` 模式。選 ripgrep 而非 RAG 的三層原因：效果、成本、可控性
- **流式輸出**：從 `client.messages.create()` 升級為 `client.messages.stream()`；引入 `LoopEventHandlers`（onText/onToolStart/onToolEnd/onToolError）實現 TUI 即時反饋
- **Spec-Driven 實踐**：每個 Story 先寫設計文件（`details/`下五篇固定格式：overview→technical-design→task-list→verification-checklist→backlog），再用 AI 實作。是 [[Spec驅動開發]] 的真實案例
- **全程 VibeCoding**：`.vibecoding/` 按 Epic/Story 組織，每階段（需求討論→設計→Spec→實作）都有對話記錄和 `learnings.md`，是目前少見的 Agentic Engineering 公開記錄

## Repo 結構

| 目錄 | 用途 |
|------|------|
| `packages/core/` | Agent Harness 核心（Loop、Tools、LLM Client） |
| `packages/tui/` | CLI 界面，TUI 展示工具調用 |
| `packages/shared/` | 共享工具、類型、Logger |
| `specs/` | 課程入口 + Story 技術文件 |
| `retros/` | 迭代復盤筆記 |
| `.vibecoding/` | AI 協作對話實錄 |
| `.discuss/` | 需求討論記錄 |
| `CHANGELOG.md` | 迭代日誌，按 E0x-S0xx 兩層版本結構 |

## 當前進度（截至 2026-06-26）

| 迭代 | 標題 | Git Tag | 狀態 |
|------|------|---------|------|
| E01-S001 | ReAct 基礎版 | `E01-S001-react-basic` | Done |
| E01-S002 | 內容搜索（grep_search） | `E01-S002-grep-search` | Done |
| E01-S003 | 文件搜索（find_files） | `E01-S003-file-search` | Done |
| E01-S004+ | TBD | — | Planned |

Epic 路線圖：
- **Epic 1**（進行中）：能看/能查——只讀工具最小閉環
- **Epic 2**（規劃中）：能動/能改/能執行
- **Epic 3**：基礎能力與產品化
- **Epic 4**：健壯性與上下文管理
- **Epic 5**：擴展能力（AGENTS、Skills、MCP、Hooks）

## Entities Mentioned

- [[alienzhou]] — 作者，前端/全棧工程師背景，基於真實 Agent Harness 產品開發經驗整理此課程
- [[Anthropic]] — 底層模型 API（Anthropic SDK + Claude）
- [[Cursor]] — 類比工具，課程定位對標

## Concepts Mentioned

- [[ReAct Pattern]] — 課程核心運行模式
- [[Spec驅動開發]] — 每個 Story 的開發方式
- [[Vibe Coding基礎概念]] — 課程全程採用 VibeCoding 協作
- [[ToolContext模式]] — E01-S003 引入的工具運行環境抽象
- [[Agent Harness實作]] — 本課程的核心產出概念

## Contradictions/Tensions

- 作者明確說明此非生產工具，而是「教具」——這與 Zero2Agent 名字給人的「成品」感有落差，需要讀者調整預期
- ripgrep 路徑依賴（macOS/Linux；Windows 需額外處理）

## Questions Raised

- Epic 5 的 Skills/MCP/Hooks 實作時間表？
- 課程是否計劃涵蓋多 Agent 協作（Orchestrator 模式）？
- VibeCoding 記錄中使用的是哪個版本的 Claude？
