---
title: SDD 規格驅動開發（高見龍）
type: source-summary
tags: [SDD, EARS, 成熟度, 工具比較, 台灣]
created: 2026-05-18
updated: 2026-05-18
sources: [kaochenlong-sdd]
---

# SDD 規格驅動開發（高見龍）

## Origin
- **作者**：高見龍（台灣知名工程師教育者）
- **日期**：2026-01-12
- **URL**：https://kaochenlong.com/sdd-spec-driven-development

## Key Takeaways

- **核心定義**：「在開始之前，先定義什麼叫完成」——規格是 AI 與人類的共同語言。
- **SDD 三階段**：
  1. **Requirements**（`requirements.md`）：使用者故事 + 驗收標準
  2. **Design**（`design.md`）：架構圖、資料模型、錯誤處理策略——在編碼前發現衝突的需求
  3. **Tasks**：追蹤的小任務，每個任務追溯至原始需求編號
- **SDD 成熟度三層次**（最重要的分類框架）：
  1. **Spec-first**：寫規格後生成程式碼，規格事後可丟棄
  2. **Spec-anchored**：規格納入版控持續更新，是變更的基準點（OpenSpec、Kiro 屬此層）
  3. **Spec-as-source**：程式碼完全由規格自動生成，不直接編輯程式碼（Tessl 屬此層）
- **EARS 格式（WHEN/THEN 結構）**：強制明確化需求、易於轉化為測試、減少 AI 猜測空間。
- **主流工具比較**：

  | 工具 | 特色 | 成熟度層次 |
  |------|------|-----------|
  | **Kiro**（Amazon） | EARS 格式、屬性測試、Hooks 功能 | Spec-anchored |
  | **Spec Kit**（GitHub） | 高度可客製化、Constitution 概念 | Spec-first |
  | **Tessl** | 追求 Spec-as-source、10,000+ OSS usage specs | Spec-as-source |
  | **OpenSpec** | 輕量、Brownfield-first、specs/changes 分離 | Spec-anchored |

- **SDD 的限制**（誠實版）：
  - 簡單小工具可能不值得投入規格時間
  - 工程師角色更接近 PM，減少親手寫程式的時間
  - 需學習 EARS 格式等新技能
  - 工具生態仍在快速演化，投資回報存疑
- **適用場景**：✅ 0→1 新產品、探索性開發、漸進式改進現有系統；❌ 快速原型、POC、單人簡單工具。

## Concepts Mentioned
- [[SDD成熟度三層次]] — 本文核心貢獻
- [[EARS需求語法]] — SDD 的語法工具
- [[Spec驅動開發]] — 整體框架
- [[SDD適用邊界]] — 限制與適用場景
