---
title: The Limits of Spec-Driven Development
type: source-summary
tags: [SDD, 限制, Context Engineering, AI開發]
created: 2026-05-18
updated: 2026-05-18
sources: [isoform-limits-sdd]
---

# The Limits of Spec-Driven Development

## Origin
- **平台**：Isoform Blog
- **日期**：2025-11-25
- **URL**：https://isoform.ai/blog/the-limits-of-spec-driven-development

## Key Takeaways

- **核心主張**：「現實的變化速度快於規格的更新速度」——SDD 複製了瀑布式開發的根本問題。
- **四大失敗模式**：
  1. **維護負擔**：需求演化時，規格、程式碼、文件三者難以同步；以帳單系統新增歐盟 VAT 為例，改程式碼比改規格更快。
  2. **缺失的 Context**：規格只描述「做什麼」，無法捕捉「為什麼這樣決策」——邊界條件、效能問題、用戶行為只在部署後才浮現。
  3. **虛假的完整感**：詳細規格造成「已涵蓋一切」的錯覺，減少迭代、扼殺創意解法，使開發變得僵化。
  4. **錯誤的抽象層次**：現有 SDD 工具聚焦實作細節（欄位、schema、signature），而非意圖與約束，產出「結構正確但方向偏差」的程式碼。
- **替代方案：Context Engineering**：以動態、意圖驅動的方式取代靜態規格——持續更新 context、在程式碼中嵌入決策理由、讓規格隨實作演化。

## Concepts Mentioned
- [[Spec驅動開發]] — 本文批評對象
- [[SDD適用邊界]] — 失敗模式的系統整理
