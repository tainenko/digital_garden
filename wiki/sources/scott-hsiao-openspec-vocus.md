---
title: OpenSpec：AI 時代的輕量文件管理（Scott Hsiao／方格子）
type: source-summary
tags: [OpenSpec, SDD, BMAD, Vibe Coding, 文件管理, SOHO]
created: 2026-05-26
updated: 2026-05-26
sources: [scott-hsiao-openspec-vocus]
---

# OpenSpec：AI 時代的輕量文件管理（Scott Hsiao／方格子）

## Origin
- **作者**：Scott Hsiao
- **平台**：Vocus 方格子（AI 讓你更強大專欄）
- **日期**：2026-03-10（更新 2026-03-11）
- **URL**：https://vocus.cc/article/69afabccfd89780001c4960b

## Key Takeaways

- **核心問題**：Vibe Coding 快速生成程式碼，但產生文件缺口——開發者與 AI 對需求的理解不一致，導致無窮修改循環
- **製造業類比**：SDD 的 proposal → 執行 → 歸檔，對應製造業的 BOM（物料清單）→ ECR（變更請求）→ ECO（變更單）→ ECN（變更通知）；SDD 是軟體開發的輕量版「工程變更管理」
- **SDD 五個開發階段**：需求與規格定義 → 架構設計 → 實作（程式碼）→ 驗證（測試）→ 部署與維運
- **三工具定位對照**（最核心貢獻）：

  | 工具 | 適用規模 | 特色 |
  |------|---------|------|
  | [[BMAD]] | 大型組織 | 詳細流程、角色分工明確 |
  | [[GitHub-spec-kit]] | 0→1 新專案 | 奠定基礎的方法 |
  | [[OpenSpec]] | 成長期 1→100 專案 | 簡單、快速、易上手 |

- **OpenSpec v1.0 五步工作流**：`explore → new → ff → apply → archive`（與高見龍文章一致，印證 v1.0 工作流）
- **目標用戶**：SOHO 工作者、中小型專案，不需要複雜流程，重視「能動就好」的效率

## Entities Mentioned
- [[OpenSpec]] — 本文主推工具
- [[BMAD]] — 大型組織 SDD 工具（github.com/bmad-code-org/BMAD-METHOD）
- [[GitHub-spec-kit]] — 0→1 新專案工具

## Concepts Mentioned
- [[Spec驅動開發]] — 方法論總覽（+三工具定位、+製造業類比）
- [[OpenSpec工作流]] — v1.0 五步流程再度確認
- [[Vibe Coding基礎概念]] — 本文問題背景

## Questions Raised
- BMAD 的詳細工作流為何？「角色分工」具體指哪些角色？
- 「1→100 成長期」的具體定義是什麼？（人數？複雜度？）
