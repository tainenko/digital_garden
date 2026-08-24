---
title: ForceInjection/OpenSpec-practise
type: source-summary
tags: [openspec, sdd, spec-driven-development, ddd, ai-coding, node, python, ecommerce]
created: 2026-05-15
updated: 2026-05-15
sources: [github-forceinjection-openspec-practise]
---

# ForceInjection/OpenSpec-practise

## Origin

- **Repo:** https://github.com/ForceInjection/OpenSpec-practise
- **Stars:** 317 | **Forks:** 56 | **License:** Apache-2.0
- **Latest Release:** OpenSpec 1.3.0 (2026-04-13)
- **Community:** "AI Force Injection" 社群 AI 輔助程式設計討論
- **Date ingested:** 2026-05-15

## Key Takeaways

- **核心定位**：OpenSpec 的完整實踐學習庫，以電商 MVP 為例，示範從規格到雙語言（Node.js + Python）落地的完整流程
- **文件格式強制規範**：`proposal.md` 需有 `## Why / ## What Changes / ## Capabilities`；`spec.md` 需使用 Delta Headers（`## ADDED/MODIFIED/REMOVED Requirements`）+ Gherkin 場景；`openspec validate` 強制檢查
- **三大角色框架**：Context Anchor（config.yaml 為持久 AI 記憶）/ Contract Guardian（規格約束 AI 輸出變異性）/ Collaboration Middleware（人類意圖 ↔ AI 執行的雙向橋樑）
- **四階段開發週期**：Intent Alignment → Spec-Driven Implementation → Automated Verification Loop → Production Evolution
- **DDD × OpenSpec 映射**：Bounded Context → specs/ 子目錄；Domain Command → Requirement；Aggregate Behavior → Scenario；Application Service → Technical Design
- **語言無關性證明**：相同 spec 文件驅動 Node.js（Express）與 Python（FastAPI + Pydantic）兩套實作，HTTP 契約與測試行為完全一致
- **`/opsx:propose` 一鍵生成**：單一指令產出 proposal.md + design.md + specs/ + tasks.md 四份文件，支援 Claude Code、Cursor、GitHub Copilot 等 20+ AI 工具

## Entities Mentioned

- [[ForceInjection]] — 作者，"AI Force Injection" 社群
- [[OpenSpec]] — 核心框架，Fission-AI 開源，npm: `@fission-ai/openspec`

## Concepts Mentioned

- [[OpenSpec文件格式與驗證]] — 各文件格式強制規則 + validate 指令
- [[OpenSpec三大角色]] — Context Anchor / Contract Guardian / Collaboration Middleware
- [[OpenSpec工作流]] — 8 階段流程（更新：加入 DDD 映射、/opsx:propose 細節）
- [[Spec驅動開發]] — SDD 方法論背景
- [[DDD領域驅動設計]] — Bounded Context / Aggregate / Domain Service 映射關係

## Contradictions/Tensions

- 現有 wiki [[OpenSpec工作流]] 將流程分為 8 階段（explore → … → archive），但本 repo 的 User Manual 將 Proposal 階段描述為「一步生成四份文件」，簡化了原本各自獨立的 design/specs/tasks 階段——意味著 `/opsx:propose` 在實踐中已合併多個階段
- 文件格式的「強制性」：validate 會拒絕不合規格的文件，但 design.md 與 tasks.md 無嚴格格式要求——格式彈性與規格嚴謹性的邊界需要工程判斷

## Questions Raised

- OpenSpec 1.3.0 vs 舊版的差異？`config.yaml` 取代哪些舊文件？
- Brownfield 遷移的實際複雜度？現有大型 codebase 的 spec 補寫優先順序如何決定？
- `/opsx:propose` 生成品質的一致性？何時需要人工大幅修改？
