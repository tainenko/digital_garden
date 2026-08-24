---
title: Claude Code 內部運作機制
type: concept
tags: [claude-code, tool-use, agent, llm, 原理, coding-agent]
created: 2026-05-07
updated: 2026-05-07
sources: [youtube-codememayb-claude-code-deep-dive]
---

# Claude Code 內部運作機制

## Claude Code 不是 LLM，而是 Coding Agent

傳統 LLM：文字輸入 → 文字輸出，無法讀檔、執行指令。

**Claude Code = Coding Agent**：在 LLM 外包一層「Agent 框架」，透過 **Tool Use** 機制讓 LLM 能操作電腦。

---

## Tool Use 機制詳解

Coding Agent 的核心技巧：在使用者 prompt 後面**自動附加工具說明**。

```
使用者問：「man.py 這個檔案在寫什麼？」

↓ Agent 在 prompt 後面偷偷加：
「如果你需要讀取檔案，請回傳格式：read_file <檔名>
 如果你需要寫入檔案，請回傳格式：write_file <檔名> <內容>」

↓ LLM 分析後，判斷需要讀檔，回傳：
「read_file man.py」

↓ Agent 收到，實際執行讀取 man.py

↓ 把 man.py 內容傳回給 LLM

↓ LLM 整合內容，輸出給使用者
```

這個來回的循環可以**多輪執行**，直到 LLM 認為任務完成。

---

## 為什麼這很重要

理解 Tool Use 循環可以幫助你：

1. **知道 Claude 什麼時候在「思考」**：每次工具呼叫都需要一個 LLM 推理週期，複雜任務需要多輪
2. **理解 Context 消耗**：每輪工具呼叫都會把工具輸出（如檔案內容）塞進 context，消耗 token
3. **善用 `@` 引用**：主動告訴 Claude 要讀哪個檔案，比讓它自己搜尋更快、更省 token
4. **設計 CLAUDE.md 時**：用 `@<路徑>` 引用重要檔案路徑，告訴 Claude「需要時去這裡讀」

---

## MCP 的角色

MCP（Model Context Protocol）是 Tool Use 的標準化延伸：

- 基本 Tool Use：read_file、write_file 等內建工具
- MCP Server：第三方提供的工具包（Playwright 瀏覽器操作、GitHub PR 操作、天氣查詢等）

**類比**：MCP Host（Claude Code）= 電腦，MCP Server = 外接 USB 裝置，工具（Tools）= 裝置提供的功能。

---

## Agent 循環的限制

- **Context 上限**：200k token（用 `/context` 可查看使用量）
- **工具呼叫失敗**：網路問題、權限不足等都會中斷循環
- **幻覺風險**：LLM 可能呼叫不存在的工具或以錯誤格式回傳

---

## 相關頁面

- [[Claude Code 入門完整指南]] — 操作指令與快捷鍵
- [[Claude MCP 伺服器整合指南]] — MCP 安裝與管理
- [[Claude Code Subagents 完整指南]] — Agent 嵌套執行
- [[Claude Code工程化架構與全景]] — 七層架構全貌
