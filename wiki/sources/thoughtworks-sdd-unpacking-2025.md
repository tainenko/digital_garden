---
title: Spec-driven development：Unpacking one of 2025's key new AI-assisted engineering practices
type: source-summary
tags: [SDD, Thoughtworks, 敏捷, AI開發, Context Engineering]
created: 2026-05-18
updated: 2026-05-18
sources: [thoughtworks-sdd-unpacking-2025]
---

# Spec-driven development：Unpacking one of 2025's key new AI-assisted engineering practices

## Origin
- **作者**：Liu Shangqi（Thoughtworks APAC 技術總監）
- **日期**：2025-12-04
- **URL**：https://www.thoughtworks.com/en-us/insights/blog/agile-engineering-practices/spec-driven-development-unpacking-2025-new-engineering-practices

## Key Takeaways

- **SDD 定義**：「以精心撰寫的軟體需求規範作為 prompt，借助 AI coding agent 生成可執行程式碼」的開發範式——核心是將規劃與實作分離。
- **業界兩種視角**：
  - **激進派**：規格是唯一真相源，程式碼只是副產品
  - **保守派**：規格驅動程式碼生成，但可執行程式碼仍是需要維護的主要 artifact
- **SDD ≠ 瀑布**（Thoughtworks 的官方立場）：瀑布的問題在於反饋週期太長、設計與實作脫節；SDD 透過「短而有效的 AI 輔助迭代」引入工程紀律，與瀑布本質不同。
- **好規格的標準**：
  - 使用領域導向語言（Domain-oriented language），而非技術實作細節
  - 使用 Given/When/Then 結構增強清晰度
  - 簡潔而全面，覆蓋關鍵路徑
  - 結合自然語言與「半結構化輸入」以減少幻覺
- **核心挑戰與風險**：
  - 非確定性程式碼生成讓維護與升級複雜化
  - 「規格漂移（Spec drift）與幻覺本質上難以避免」
  - 缺乏系統性的規格品質評估方法
  - 需要確定性 CI/CD 實踐來確保品質
- **與 Context Engineering 的整合**：SDD 透過 AGENTS.md 檔案和 MCP 伺服器組織資訊、降低 token 消耗，同時提升程式碼生成準確度。

## Entities Mentioned
- Thoughtworks — 全球技術諮詢公司

## Concepts Mentioned
- [[Spec驅動開發]] — 本文分析對象
- [[SDD適用邊界]] — 挑戰與風險補充
- [[Context Engineering最佳實踐]] — SDD 的輔助機制

## Contradictions / Tensions
- Thoughtworks 說「SDD ≠ 瀑布」，但 Marmelab CEO 說「SDD = AI 版瀑布」——兩家機構在相同問題上有正面衝突，值得並陳。
- 「規格漂移難以避免」的承認，與 spec-kit 宣傳的高效自動化之間存在落差。
