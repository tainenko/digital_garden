---
title: Claude Code 工程化實戰（黃佳，極客時間）
type: source-summary
tags: [claude-code, engineering, course, geekbang, subagents, hooks, mcp, skills, headless]
created: 2026-05-07
updated: 2026-05-07
sources: [github-huangjia2019-claude-code-engineering]
---

# Claude Code 工程化實戰

## Origin

- **標題**：Claude Code 工程化實戰《Claude Code 工程化实战》
- **作者**：[[黃佳 huangjia2019]]
- **平台**：[[極客時間 GeekBang]]（極客時間專欄）
- **上線日期**：2026-01-28
- **GitHub**：https://github.com/huangjia2019/claude-code-engineering
- **Stats**：577 stars、211 forks、萬人訂閱（上線一個月，位居極客時間總榜第一）
- **擷取日期**：2026-05-07

## Key Takeaways

- 目標：把 Claude Code 從「對話式 coding 工具」升級為「可設計、可複用、可治理的工程化系統」
- 課程語言：簡體中文；代碼語言：JavaScript 48.8%、Python 34.1%、Shell 14%、TypeScript 3.1%
- **記憶系統四層**：用戶級 → 項目級 → 本地級 → 規則目錄（`.claude/rules/*.md`）
- **Subagent 四大模式**：只讀型代碼審查、高噪聲任務隔離、並行探索、Agent Teams 多會話
- **Commands 核心**：Slash 命令存於 `.claude/commands/`，支援 `$ARGUMENTS`、`@` 引用、命名空間（`/git:commit`）
- **Hooks 四事件**：`PreToolUse`、`PostToolUse`、`Stop`、`SubAgentStop`；分安全型（攔截危險操作）與品質型（自動格式化/Lint/測試）
- **MCP 三傳輸層**：stdio / HTTP / SSE；五大生產 Server：Context7、GitHub、Notion、DB、Fetch；可用 TypeScript 自建
- **Headless 模式**：無需人工守候終端，可嵌入 GitHub Actions 做自動 PR Review
- **Rules 系統**：指令規則（`.claude/rules/*.md` 按路徑載入）+ 三級權限規則（deny → ask → allow）
- **Agent SDK**：`query()` 方法、`@tool` 裝飾器自定義工具、兩段式測試修復工作流
- **Plugins**：Skills + Commands + Hooks 打包成含 manifest 的可分發插件包

## 課程結構（23 講）

### 第一部分：基礎篇
| 講次 | 主題 |
|------|------|
| 第 1 講 | Claude Code 全景導覽：技術棧與可擴展框架 |
| 第 2 講 | CLAUDE.md 記憶系統：永久生效的項目規範 |

### 第二部分：子代理專題
| 講次 | 主題 |
|------|------|
| 第 3 講 | 子代理核心概念：隔離執行與權限管理 |
| 第 4 講 | 只讀型子代理（代碼審查員項目） |
| 第 5 講 | 高噪聲任務處理（測試運行器與日誌分析） |
| 第 6 講 | 並行探索與流水線編排（多視角探索 + Bug 修復）|
| 第 7 講 | Agent Teams 多會話協作架構 |

### 第三部分：Skills 技能系統
| 講次 | 主題 |
|------|------|
| 第 9 講 | SKILL.md 結構與觸發機制 |
| 第 10 講 | 任務型 Skills 實戰（團隊標準命令集）|
| 第 11 講 | 漸進式披露架構設計（財務分析案例）|
| 第 12 講 | Skills 高級模式與 SubAgent 配合 |
| 第 13 講 | Skills 架構設計模式 |
| 第 14 講 | Skills 出圈與行業開放標準演進 |

### 第四部分：擴展機制
| 講次 | 主題 |
|------|------|
| 第 15 講 | Hooks 事件驅動自動化（上）|
| 第 16 講 | Hooks 高級模式與工程實踐（下）|
| 第 17 講 | MCP 協議與外部工具連接 |

### 第五部分：生產化與工程化
| 講次 | 主題 |
|------|------|
| 第 18 講 | Tools 工具系統深度剖析（五種原子操作）|
| 第 19 講 | Headless 模式與 CI/CD 集成 |
| 第 20 講 | Rules 規則系統深度剖析 |
| 第 21 講 | Agent SDK 基礎 |
| 第 22 講 | Agent SDK 高級應用（自動化測試修復）|
| 第 23 講 | Plugins 插件打包與分發（團隊能力包）|

## Entities Mentioned

- [[黃佳 huangjia2019]] — 作者，課程設計者
- [[極客時間 GeekBang]] — 出版平台

## Concepts Mentioned

- [[Claude Code 記憶體系統深度指南]] — 第 2 講，四層記憶架構
- [[Claude Code Subagents 完整指南]] — 第 3–7 講，隔離執行與並行模式
- [[Claude Code Hooks 深度設定]] — 第 15–16 講，SubAgentStop 事件、safety/quality hooks
- [[Claude MCP 伺服器整合指南]] — 第 17 講，TypeScript 自建 MCP Server
- [[Claude Code工程化架構與全景]] — 整體框架概覽
- [[Claude Code Headless模式與CICD]] — 第 19 講，無人值守自動化
- [[Claude Code Rules規則系統]] — 第 20 講，指令規則 + 三級權限
- [[Claude Code Plugins插件系統]] — 第 23 講，打包與分發

## Contradictions/Tensions

- Skills 與 Tools 的邊界：課程明確區分（Skills = 工作流編排；Tools = 原子操作），但實際使用中常混淆
- Plugins 標準尚在演進（第 14 講提到「行業開放標準」），跨平台 Skills/Plugins 互通性仍是待解問題
- Headless 模式的安全風險：無人值守 + 廣泛權限 → 需搭配 Rules 的 deny 規則做防護

## Questions Raised

- Agent Teams 多會話協作架構的具體狀態管理機制？
- `SubAgentStop` hook 的觸發時機與父 Agent 的協調方式？
- TypeScript MCP Server 的 token cost 控制最佳實踐？
