---
title: ForceInjection
type: entity
tags: [GitHub, DDD, OpenSpec, AI Agent, 開源]
created: 2026-05-25
updated: 2026-05-25
sources: [github-forceinjection-openspec-practise, forceinjection-ddd-skills]
---

# ForceInjection

## 基本資訊

- **類型**：GitHub 組織（開源工具作者）
- **GitHub**: https://github.com/ForceInjection
- **核心方向**：AI Agent 輔助軟體工程——spec 驅動開發 + DDD 建模

## 主要作品

| Repo | 主題 | Stars | Wiki 頁面 |
|------|------|-------|-----------|
| [OpenSpec-practise](https://github.com/ForceInjection/OpenSpec-practise) | OpenSpec 實踐庫：Context Anchor / Contract Guardian / Collaboration Middleware 三大角色框架 | 317⭐ | [[OpenSpec三大角色]]、[[OpenSpec文件格式與驗證]] |
| [domain-driven-design-skills](https://github.com/ForceInjection/domain-driven-design-skills) | 9技能×5階段 DDD AI Agent 完整建模流水線，含回溯機制與 OpenSpec 橋接 | 9⭐ | [[DDD AI Agent技能流水線]]、[[DDD回溯觸發機制]] |

## 技術主張

- **閉環優先**：不做孤立工具，每個 repo 都以「完整可驗證的工作流閉環」為設計目標
- **非線性思維**：流程不是瀑布，允許品質觸發回溯（DDD Skills 的 backtrack 機制）
- **語言無關**：OpenSpec 與 DDD Skills 均設計為語言無關（Node.js/Python/Go 均可使用）
- **可量化驗證**：DDD Skills 以 85.8% 加權分數 + 盲跑協議建立可重現的品質基準

## 與生態系的關係

- [[OpenSpec]] — ForceInjection 是 OpenSpec 最重要的實踐者之一（非規格原作者，但提供最完整的實踐指南）
- [[Spec驅動開發]] — 其兩個 repo 均服務於 SDD 方法論
- [[DDD領域驅動設計]] — domain-driven-design-skills 是 DDD 知識到 AI 實作工作流的橋梁

## 相關頁面

- [[OpenSpec工作流]] — OpenSpec 8 階段工作流
- [[OpenSpec三大角色]] — Context Anchor / Contract Guardian / Collaboration Middleware
- [[DDD AI Agent技能流水線]] — 9技能DDD建模流水線
- [[DDD回溯觸發機制]] — 品質門禁反饋設計
