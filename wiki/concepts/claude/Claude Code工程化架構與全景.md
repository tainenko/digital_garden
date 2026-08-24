---
title: Claude Code 工程化架構與全景
type: concept
tags: [claude-code, engineering, architecture, overview]
created: 2026-05-07
updated: 2026-05-07
sources: [github-huangjia2019-claude-code-engineering]
---

# Claude Code 工程化架構與全景

## 核心理念

把 Claude Code 從「對話式 coding 工具」升級為「可設計、可複用、可治理的工程化系統」。（[[黃佳 huangjia2019]]，[[極客時間 GeekBang]] 課程核心命題）

## 七層可擴展框架

```
┌─────────────────────────────────┐
│  Plugins（插件打包與分發）       │  ← 第五部分：生產化
│  Agent SDK（程式化控制）         │
│  Headless（CI/CD 無人值守）      │
│  Rules（指令規則 + 權限規則）    │
├─────────────────────────────────┤
│  MCP（外部工具連接）             │  ← 第四部分：擴展機制
│  Hooks（事件驅動自動化）         │
│  Commands（Slash 命令模板）      │
├─────────────────────────────────┤
│  Skills（工作流技能系統）        │  ← 第三部分：技能
│  Sub-Agents（隔離執行）          │  ← 第二部分：子代理
│  Memory / CLAUDE.md（記憶系統） │  ← 第一部分：基礎
└─────────────────────────────────┘
```

## 各層職責

| 層 | 核心問題 | 詳細頁面 |
|----|---------|---------|
| **Memory** | Claude 記得什麼？怎麼記？ | [[Claude Code 記憶體系統深度指南]] |
| **Sub-Agents** | 如何隔離執行、並行分工？ | [[Claude Code Subagents 完整指南]] |
| **Skills** | 如何把工作流一次學會永久套用？ | [[Claude MCP 伺服器整合指南]] |
| **Commands** | 如何消除重複的 Slash 指令輸入？ | [[Claude Code Hooks 深度設定]] |
| **Hooks** | 如何在工具呼叫前後自動介入？ | [[Claude Code Hooks 深度設定]] |
| **MCP** | 如何連接外部工具（DB、GitHub…）？ | [[Claude MCP 伺服器整合指南]] |
| **Rules** | 什麼能做、什麼不能做？ | [[Claude Code Rules規則系統]] |
| **Headless** | 如何讓 Claude 無人值守在 CI 中跑？ | [[Claude Code Headless模式與CICD]] |
| **Agent SDK** | 如何用程式碼控制 Claude？ | [[Claude Code Subagents 完整指南]] |
| **Plugins** | 如何把能力打包分發給團隊？ | [[Claude Code Plugins插件系統]] |

## 工具原子操作（五種）

Claude Code 的所有內建工具最終對應五種原子操作：

1. **Read**（讀）— 讀取檔案、搜尋代碼
2. **Write**（寫）— 修改、建立檔案
3. **Search**（搜）— Grep、Glob、語意搜尋
4. **Execute**（執行）— Bash 命令、腳本
5. **Interact**（互動）— 呼叫 MCP、外部 API

複雜能力 = 這五種原子操作的組合湧現（emergent capabilities）。

## 設計哲學

- **「沒有 Hooks 是信任，有 Hooks 是被驗證的信任」** — 治理層的核心思路
- **「指令規則告訴 Claude 該怎麼做，權限規則告訴 Claude 能做什麼」** — Rules 雙軌制
- **「寫出有用的 Skills 是工藝，把它打包分發才是工程」** — Plugins 存在意義
- **「寫一個 Agent SDK 就是在為 Claude 寫作業系統」** — 程式化控制的定位

## 相關頁面

- [[Claude Code 入門完整指南]]
- [[Claude Code Subagents 完整指南]]
- [[Claude Code Hooks 深度設定]]
- [[Claude MCP 伺服器整合指南]]
- [[Claude Code Rules規則系統]]
- [[Claude Code Headless模式與CICD]]
- [[Claude Code Plugins插件系統]]
- [[黃佳 huangjia2019]]
