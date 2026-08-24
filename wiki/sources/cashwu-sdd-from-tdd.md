---
title: SDD — 從 TDD 到規格驅動開發：AI 時代的延伸
type: source-summary
tags: [SDD, TDD, AI開發, 測試, 規格]
created: 2026-05-18
updated: 2026-05-18
sources: [cashwu-sdd-from-tdd]
---

# SDD — 從 TDD 到規格驅動開發：AI 時代的延伸

## Origin
- **作者**：Cash Wu（台灣工程師）
- **日期**：2026-02-15
- **URL**：https://blog.cashwu.com/blog/2026/sdd-from-tdd-to-spec-driven-development

## Key Takeaways

- **SDD 的起點是自然語言 prompt 的根本限制**：要求 AI「實作使用者註冊功能」會導致過度實作——AI 不清楚業務邊界，只能猜測並傾向「做多不做少」。
- **SDD 是 TDD 在 AI 時代的延伸**：不是競爭關係，而是上下游補充——「規格定義 WHAT，測試驗證做對了嗎，AI 負責 HOW」。
- **SDD 三個實踐層次**：
  1. **基本層**：結構化 Markdown 規格（功能範圍、API 設計、邊界條件）
  2. **進階層**：OpenSpec 等工具化框架（標準化格式與流程管理）
  3. **整合層**：結合 TDD，形成「規格 → 測試 → AI 實作 → Review」完整閉環
- **Given-When-Then 的 AI 時代價值**：
  - **結構化**：固定框架便於 AI 解析，不需猜測意圖
  - **具體**：實例化需求勝過抽象描述
  - **可驗證**：每個 Then 都可編程驗證，與測試天然整合
- **SDD 不是銀彈**：它減少溝通摩擦，但無法取代思考。品質把關、設計決策、邊界條件發現仍需人類判斷。

## Concepts Mentioned
- [[Spec驅動開發]] — 本文定位 SDD 的方法論框架
- [[BDD行為驅動開發]] — Given-When-Then 的來源
- [[SDD適用邊界]] — 「不是銀彈」的明確聲明

## Questions Raised
- 在「規格 → 測試 → AI 實作 → Review」的閉環中，測試應該在規格完成後、AI 開始前就寫好嗎？
