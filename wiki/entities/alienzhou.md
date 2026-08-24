---
title: alienzhou
type: entity
tags: [agent-harness, vibe-coding, typescript, educator, open-source]
created: 2026-06-26
updated: 2026-06-26
sources: [alienzhou-zero2agent]
---

# alienzhou

## 基本資料

- **GitHub**：https://github.com/alienzhou
- **身份**：前端/全棧工程師、技術教育者
- **背景**：具備真實 Agent Harness 產品開發與上線經驗，將踩坑記錄轉化為公開課程

## 代表作

### [[alienzhou-zero2agent|Zero2Agent]]

從零實現產品級 Agent Harness 的公開教學課程（2026），核心特色：

- 完整透明的開發過程——需求討論、設計決策、AI 協作對話、復盤筆記全部公開
- 每個迭代對應 Git Tag，可任意切換跟練
- 全程採用 [[Vibe Coding基礎概念|VibeCoding]] + [[Spec驅動開發|SDD]] 協同開發
- 以 TypeScript pnpm monorepo 組織，底層呼叫 [[Anthropic]] SDK

## 核心觀點

- **工具設計哲學**：對人好用 = 對 AI 好用；工具設計前先回答四問：解決什麼問題/控制什麼/輸出契約/邊界兜底
- **選 ripgrep 不選 RAG**：三層原因——效果、成本、可控性（顯式優於黑盒）
- **ToolContext 模式**：隱式依賴（process.cwd()）會在第三個工具進來時暴露，應從一開始就顯式注入運行上下文

## 相關頁面

- [[ToolContext模式]]
- [[ReAct Pattern]]
- [[Spec驅動開發]]
- [[Vibe Coding基礎概念]]
