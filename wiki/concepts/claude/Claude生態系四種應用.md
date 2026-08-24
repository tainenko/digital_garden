---
title: Claude 生態系四種應用
type: concept
tags: [claude, ecosystem, cowork, claude-code, overview]
created: 2026-05-07
updated: 2026-05-07
sources: [aiposthub-claude-tutorial-2026]
---

# Claude 生態系四種應用

[[Anthropic]] 圍繞 Claude 打造了四種不同入口，對應不同使用場景與技術門檻。

## 四種應用

### 1. Claude.ai（網頁 / 手機）
- **定位**：一般使用者的主要入口
- **特點**：對話介面、Projects、Skills、Connectors（Gmail、Chrome、Adobe 等）
- **門檻**：無需技術背景
- **功能範圍**：文字生成、分析、工具整合、長期記憶（Projects）

### 2. Claude Cowork（桌面應用）
- **定位**：非技術用戶的自動化工具
- **特點**：直接操作本機檔案、輸出 Excel/Word/PPT，GUI 介面
- **門檻**：安裝即用，不需寫程式
- **功能範圍**：桌面代理、跨 App 操作、Dispatch 遠端指揮
- 詳見 [[CoWork桌面工具指南]]、[[Claude Dispatch遠端控制]]

### 3. Claude Code（終端機 CLI）
- **定位**：開發者的 AI 編程助理
- **特點**：從命令列操控整個代碼庫，支援 Computer Use（直接操控 Mac）
- **門檻**：需要開發者背景
- **功能範圍**：全端開發、Subagents、MCP 整合、CLAUDE.md 客製化
- 詳見 [[Claude Code 入門完整指南]]、[[Claude Code Subagents 完整指南]]

### 4. Claude Code Channels（2026 新）
- **定位**：企業 / 團隊協作環境
- **特點**：多人共用 Claude Code，整合到 Slack 等協作工具
- **門檻**：企業方案
- **功能範圍**：團隊 AI 工作流、非同步任務執行

## 選擇指南

| 你的狀況 | 推薦入口 |
|---------|---------|
| 日常文字工作、工具整合 | Claude.ai |
| 想讓 AI 直接操作電腦、輸出文件 | Cowork |
| 工程師、想寫程式 / Vibe Coding | Claude Code |
| 企業團隊協作 | Claude Code Channels |

## 三層使用架構

不管用哪個入口，Claude 都有相同的三層結構：

1. **Prompts**：單次對話指令，下次就忘
2. **Projects**：長期記憶空間，記住背景與規則
3. **Skills**：永久套用的工作流，一次設定自動執行

詳見 [[Claude API基礎與最佳實踐]]、[[Claude Prompt工程核心技巧]]

## 相關頁面

- [[CoWork桌面工具指南]]
- [[Claude Dispatch遠端控制]]
- [[Claude Code 入門完整指南]]
- [[Anthropic Managed Agents]]
- [[Claude for Chrome完整教學]]
