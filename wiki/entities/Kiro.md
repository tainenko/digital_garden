---
title: Kiro
type: entity
tags: [Amazon, SDD, EARS, IDE, AI開發工具]
created: 2026-05-26
updated: 2026-05-26
sources: [kaochenlong-sdd]
---

# Kiro

Amazon 推出的 AI 編碼 IDE，定位在 [[SDD成熟度三層次]] 的 **Spec-anchored** 層次，以 EARS 格式為需求語言核心。

## 工作流程

```
Requirements → Design → Tasks
```

三個階段對應三份文件：`requirements.md`、`design.md`、`tasks.md`，規格納入版本控制並隨專案演進更新。

## 核心特色

| 特色 | 說明 |
|------|------|
| **EARS 格式** | 使用 `WHEN [條件] THEN [系統行為]` 結構強制明確需求 |
| **Property-based testing** | 根據規格自動生成基於屬性的測試案例 |
| **Hooks 功能** | 事件觸發的自動化規格關聯機制 |

## 定位

與 [[GitHub-spec-kit]]、[[OpenSpec]] 同屬 SDD 工具生態，但由 Amazon 官方維護，更偏向企業導向的標準化流程。相比 OpenSpec 的輕量 Brownfield-first 風格，Kiro 在工作流程上更為結構化。

## 相關頁面

- [[SDD成熟度三層次]] — Kiro 屬 Spec-anchored 層
- [[EARS需求語法]] — Kiro 的需求語法基礎
- [[Spec驅動開發]] — SDD 方法論總覽
- [[GitHub-spec-kit]] — 同類 SDD 工具（Spec-first 層）
- [[OpenSpec]] — 同類 SDD 工具（Spec-anchored 層）
