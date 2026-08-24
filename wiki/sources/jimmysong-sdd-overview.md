---
title: 規範驅動開發（SDD）簡介：從氛圍編程到 SDD
type: source-summary
tags: [SDD, Jimmy Song, AI Agent, 系統框架, CNCF]
created: 2026-05-18
updated: 2026-05-18
sources: [jimmysong-sdd-overview]
---

# 規範驅動開發（SDD）簡介：從氛圍編程到 SDD

## Origin
- **作者**：Jimmy Song（CNCF Ambassador、《Cloud Native Patterns》作者）
- **日期**：2025-11-03
- **URL**：https://jimmysong.io/zh/book/ai-handbook/sdd/overview/

## Key Takeaways

- **核心定位**：「讓『氛圍』變成『結構』」——SDD 是把 Vibe Coding 的直覺轉化為可管理工程實踐的方法論。
- **五大核心能力**：結構化任務分解 / 智能上下文工程 / 標準化交付體系 / 測試驅動的自愈式開發（Self-Healing TDD）/ 品質驅動的持續優化。
- **五階段 SDD 工作流**（閉環設計）：
  1. **Specify**：人 + AI 將需求擴展為結構化規範
  2. **Plan**：AI 輸出實現方案與架構
  3. **Tasks**：拆成可測試的小任務
  4. **Implement & Test**：AI 實現並驗證，失敗時回饋至前一步
  5. **Deploy**：生成部署腳本與 CI/CD 配置
- **三層協議棧**（AI-native 架構視角）：
  - **MCP**（Model-Context Protocol）：AI 與工具交互
  - **A2A**（Agent-to-Agent）：多 agent 協作
  - **AG-UI**：用戶與 Agent 的實時可視交互
- **準確率評估標準**：成功率 ≥90%（直接編譯運行）、可部署率 ≥85%（自動化測試通過）、返工率 ≤10%——「當某類任務準確率穩定在 90%+，AI 就從實驗變為生產力」。
- **代表性工具全景**（8 個）：Kiro / Spec-kit / Tessl / Qoder / LCEL / OpenDevin / AgentScript / CodePlan。
- **工程師角色轉變**：從「程式碼撰寫者」→「規範制定者」；未來願景：「規範被當作程式碼對待，AI 成為團隊中的準工程師」。

## Entities Mentioned
- Jimmy Song — CNCF 社群知名作者

## Concepts Mentioned
- [[Spec驅動開發]] — 本文分析框架
- [[SDD成熟度三層次]] — 工具全景補充
- [[Context Engineering最佳實踐]] — 智能上下文工程

## Contradictions / Tensions
- 「準確率 ≥90% 才算生產力」的標準很具體，但缺乏來源數據支撐。
- Self-Healing TDD 的概念雖然吸引人，實際上需要足夠完整的測試覆蓋才能「自愈」，這本身是個大前提。
