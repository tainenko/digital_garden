---
title: Spec-Driven Development：The Waterfall Strikes Back
type: source-summary
tags: [SDD, 瀑布模型, 敏捷, 批評, AI開發]
created: 2026-05-18
updated: 2026-05-18
sources: [marmelab-sdd-waterfall]
---

# Spec-Driven Development：The Waterfall Strikes Back

## Origin
- **作者**：François Zaninotto（Marmelab 創辦人兼 CEO）
- **日期**：2025-11-12
- **URL**：https://marmelab.com/blog/2025/11/12/spec-driven-development-waterfall-strikes-back.html
- **工具提及**：Spec-Kit（GitHub）、Kiro（AWS）、Tessl、BMAD Method

## Key Takeaways

- **核心主張**：SDD 是瀑布式開發的 AI 版——用詳細規劃「保護」coding agent，和瀑布模型用大量文件讓開發者「直接翻譯規格為程式碼」的邏輯完全一樣。
- **七大問題**：
  1. **背景盲點**：agent 靠文字搜尋發現 context，常遺漏需要同步更新的既有功能
  2. **Markdown 過載**：開發者花大量時間閱讀冗長 Markdown，在其中找隱藏的錯誤
  3. **系統官僚化**：三步設計流程本身過重；規格充斥重複與捏造的邊界案例
  4. **虛假敏捷**：誤用「用戶故事」等敏捷術語，但本質是大設計前置（Big Design Up Front）
  5. **雙重 Code Review**：規格本身包含程式碼，需要審查兩次
  6. **虛假安全感**：agent 實際上常不遵循規格
  7. **收益遞減**：在既有大型 codebase 中幾乎無法運作
- **《沒有銀彈》引用**：軟體開發本質上是不確定性過程，詳細規劃無法消除這個不確定性。
- **替代方案：自然語言開發（Natural Language Development）**，靈感來自精實創業：
  1. 辨識產品中下一個最高風險假設
  2. 設計最簡單的實驗來測試
  3. 開發該實驗；失敗則回到步驟 2
- **案例**：作者用 Claude Code 在 10 小時內完成 3D 雕刻工具，「未寫任何規格」，純粹逐步迭代加功能。
- **名言**：「coding agent 就像內燃機的發明。SDD 把它們限制在火車上，當我們應該造汽車、飛機和一切其他東西。」

## Concepts Mentioned
- [[Spec驅動開發]] — 本文批評對象
- [[SDD適用邊界]] — 七大問題是邊界判斷的反面依據
- [[Vibe Coding基礎概念]] — 自然語言開發是 Vibe Coding 的學術化表述
