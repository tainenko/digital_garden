---
title: 【深入淺出 Claude Code】運作原理、MCP、Agent Skills、Hooks、Subagents、Plugins 一次搞懂
type: source-summary
tags: [claude-code, mcp, hooks, subagents, plugins, tool-use, context-management, slash-commands]
created: 2026-05-07
updated: 2026-05-07
sources: [youtube-codememayb-claude-code-deep-dive]
---

# 【深入淺出 Claude Code】運作原理、MCP、Agent Skills、Hooks、Subagents、Plugins 一次搞懂

## Origin

- **標題**：【深入淺出 Claude Code】運作原理、MCP、Agent Skills、Hooks、Subagents、Plugins 一次搞懂！ #2026最新版 #ai #claude
- **作者/頻道**：Code Me Maybe 碼上學
- **來源**：YouTube（https://www.youtube.com/watch?v=PmlEWW8WMf0）
- **Clipping**：`/Users/tonyk/Downloads/【深入淺出 Claude Code】...webm`（逐字稿：同目錄 .txt）
- **擷取日期**：2026-05-07
- **備註**：逐字稿為 TurboScribe 轉錄，30 分鐘後截斷（免費版限制）；涵蓋運作原理、Context 管理、MCP；Hooks/Subagents/Plugins 段落未收錄

## Key Takeaways

- **Claude Code = Coding Agent**：不是單純 LLM（只能文字進文字出），而是透過 **Tool Use** 機制讓 LLM 可以呼叫工具（讀檔、寫檔、執行指令）
- **Tool Use 核心機制**：Coding Agent 在使用者 prompt 後面偷偷加一段說明（「需要讀檔請回傳 `read_file 檔名`」），LLM 回傳工具呼叫格式 → Agent 執行 → 把結果再傳回 LLM → 最終輸出
- **CLAUDE.md 四個層級**：
  1. 全域：`~/.claude/CLAUDE.md`（跨所有專案）
  2. 專案根目錄：`CLAUDE.md`（check in git，團隊共用）
  3. 專案本地：`CLAUDE.local.md`（不 check in，個人設定）
  4. 子目錄：`<subdir>/CLAUDE.md`（只在 Claude 讀取該目錄檔案時才載入，節省 context）
- **子目錄 CLAUDE.md 的差異**：根目錄 CLAUDE.md 每次都夾帶；子目錄 CLAUDE.md 只在 Claude 需要讀該目錄時才讀入——是節省 context 的關鍵設計
- **`/init` 指令**：掃描整個 codebase，自動生成 CLAUDE.md（架構概覽、常用指令、重要模組）
- **`/context` 指令**：顯示 200k token 使用量細分（system prompt、工具、memory file、對話等）
- **`@` 引用檔案**：在 prompt 中用 `@<路徑>` 精準引入特定檔案，比讓 Claude 自行搜尋更高效
- **Shift+Tab 三段模式切換**：Manual（每次確認）→ Accept Edits On → Plan Mode（先計畫後執行），循環切換
- **`claude -c`**：啟動時帶 `-c` 直接繼續上次對話（continue），等同 `/resume`
- **Ctrl+B**：在 Claude Code 終端機中讓當前執行的程序移到背景（不阻塞對話）
- **MCP scopes**：`claude mcp add --scope local`（預設，存 `~/.claude.json`）/ `--scope project`（存 `.mcp.json`，可 check in）/ `--scope user`（使用者層級）
- **`/task` 指令**：列出所有背景任務，按 `k` 可 kill 任務

## 重要指令速查表（影片示範）

| 指令/快捷鍵 | 功能 |
|------------|------|
| `!` 前綴 | Bash mode（粉紅色，直接輸入終端機指令）|
| `@` 前綴 | 引用檔案加入 context |
| `#` 前綴 | 快速新增內容到 CLAUDE.md（2.0.70 以前版本）|
| `/init` | 掃描 codebase 生成 CLAUDE.md |
| `/compact` | 壓縮對話（節省 context）|
| `/context` | 顯示 token 使用量細分 |
| `/clear` | 清空對話 |
| `/rewind` 或雙按 Esc | 回到指定對話節點（可選 restore code / conversation）|
| `/model` | 切換模型（opus/sonnet 等）|
| `/mcp` | 管理 MCP server（查看工具清單、連線狀態）|
| `/task` | 查看背景任務（按 k 可 kill）|
| `/resume` | 恢復上次對話 |
| `/exit` | 退出 Claude Code（回到一般終端）|
| Shift+Tab | 切換許可權模式（manual/accept edits/plan）|
| Ctrl+B | 把當前程序移背景執行 |
| `claude -c` | 啟動時直接繼續上次對話 |

## Concepts Mentioned

- [[Claude Code內部運作機制]] — Tool Use 機制詳解
- [[CLAUDE.md撰寫最佳實踐]] — 四個層級與子目錄行為
- [[Claude MCP 伺服器整合指南]] — MCP 安裝、scope、工具清單
- [[Claude Code 入門完整指南]] — 斜線指令、快捷鍵、操作模式
- [[Claude Code工程化架構與全景]] — 整體架構
- [[Code Me Maybe 碼上學]] — 頻道實體

## Contradictions/Tensions

- `/model` 顯示：免費/pro 方案 default 是 sonnet；Max 方案 default 是 opus 4.5（各方案 default 不同）
- 2.0.70 版本後移除了 `#` 快捷鍵新增 CLAUDE.md，改為只能用指令方式請 Claude 幫忙更新

## Questions Raised

- Hooks、Subagents、Plugins 的完整說明（逐字稿被截斷，未涵蓋）
- `claude mcp add` 的完整參數列表
- 各訂閱方案的 default 模型完整對照表
