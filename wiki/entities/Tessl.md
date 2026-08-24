---
title: Tessl
type: entity
tags: [SDD, Spec-as-source, AI開發工具, 代碼生成]
created: 2026-05-26
updated: 2026-05-26
sources: [kaochenlong-sdd]
---

# Tessl

追求 **Spec-as-source** 最高層次的 SDD 框架，目標是讓程式碼完全由規格自動生成，開發者不直接編輯程式碼。

## 核心概念

Tessl 的三大元素：

- **Plans** — 高層開發計畫
- **Specs** — 詳細行為規格（唯一真相源）
- **Tests** — 由規格衍生的驗證層

程式碼是 Specs 的副產品，任何改動都發生在規格層面，生成的程式碼不可直接修改。

## 差異化資源

提供超過 **10,000 個開源函式庫的 usage specs**，降低從零撰寫規格的門檻。

## 成熟度與風險

- 目前仍在**實驗階段**，尚未達到生產成熟度
- 對規格品質要求極高——規格有錯，生成的程式碼全錯
- 工具生態仍在快速演化，長期投資回報存疑

## 在 SDD 三層次中的位置

| 層次 | 代表工具 |
|------|---------|
| Spec-first | [[GitHub-spec-kit]] |
| Spec-anchored | [[OpenSpec]]、[[Kiro]] |
| **Spec-as-source** | **Tessl** |

## 相關頁面

- [[SDD成熟度三層次]] — Tessl 屬最高層次 Spec-as-source
- [[Spec驅動開發]] — SDD 方法論總覽
- [[SDD適用邊界]] — Spec-as-source 的局限與門檻
- [[Kiro]] — 同類工具（Spec-anchored 層）
- [[OpenSpec]] — 同類工具（Spec-anchored 層）
