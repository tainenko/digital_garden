---
title: 什麼是 EARS 與 BDD？Spec-Driven Development 規格驅動開發
type: source-summary
tags: [EARS, BDD, SDD, 需求語法, Gherkin]
created: 2026-05-18
updated: 2026-05-18
sources: [devtldrlss-ears-bdd-sdd]
---

# 什麼是 EARS 與 BDD？Spec-Driven Development 規格驅動開發

## Origin
- **平台**：Dev TLDRLSS
- **日期**：2026-02-13
- **URL**：https://dev.tldrlss.com/article/2026/02/whats-bdd-and-ears-sdd-scenario/

## Key Takeaways

- **核心主張**：「EARS 幫你立下嚴謹的規矩，BDD 幫你確認演出的結果。」兩者互補，分別解決「需求歧義」和「邏輯驗證」。
- **EARS（Easy Approach to Requirements Syntax）**：將模糊形容詞轉化為精準指令的五種句型：
  1. **Ubiquitous（通用型）**：「系統必須...」
  2. **Event-driven（事件驅動型）**：「當 [事件發生] 時，系統必須...」
  3. **Unwanted Behavior（異常處理型）**：「如果 [壞事發生]，系統必須...」
  4. **State-driven（狀態驅動型）**：「當處於 [某種狀態] 時，系統必須...」
  5. **Optional Feature（選配型）**：「若有 [特殊功能] 時，系統必須...」
- **BDD（Behavior-Driven Development）**：Gherkin 語法三部曲：
  - 🎬 **Given（背景）**：設定場景前提
  - ⚡ **When（動作）**：觸發的操作
  - ✅ **Then（結果）**：驗證的預期結果
- **EARS vs BDD 比較**：

  | 特性 | EARS | BDD |
  |------|------|-----|
  | 核心精神 | 精準定義 | 共識與驗證 |
  | 寫作形式 | 法律條文 | 電影劇本 |
  | 主要受眾 | PM、架構師 | 工程師、QA、AI |
  | 解決痛點 | 需求歧義 | 邏輯驗證 |

- **實戰範例**：電子錢包扣款 API 的完整 Prompt，同時包含 EARS 邏輯規則（事件驅動、異常處理、狀態驅動）和 BDD 驗收劇本（Jest 測試格式）。

## Concepts Mentioned
- [[EARS需求語法]] — 本文核心之一
- [[BDD行為驅動開發]] — 本文核心之一
- [[好規格寫作原則]] — EARS 和 BDD 是好規格的語法工具
- [[Spec驅動開發]] — SDD 的語法實現層

## Questions Raised
- EARS 的五種句型在複雜業務需求中是否需要組合使用？
- BDD 的 Gherkin 語法和 OpenSpec 的 Behavioral Specs 是同一層次的工具嗎？
