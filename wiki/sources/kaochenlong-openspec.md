---
title: OpenSpec 讓 SDD 變簡單的三個指令（高見龍）
type: source-summary
tags: [OpenSpec, SDD, brownfield, 工具教學, 台灣]
created: 2026-05-26
updated: 2026-05-26
sources: [kaochenlong-openspec]
---

# OpenSpec 讓 SDD 變簡單的三個指令（高見龍）

## Origin
- **作者**：高見龍（[[高見龍]]）
- **日期**：2026-01-15
- **URL**：https://kaochenlong.com/openspec

## Key Takeaways

- **3 個核心指令**：`proposal → apply → archive` — 所有 OpenSpec 操作的使用者端面貌，8 階段工作流的入口點
- **Brownfield-first 不只是口號**：明確對比 Kiro / Spec Kit 的 greenfield 設計，OpenSpec 專門支援「1→n 持續開發」，不是只適合全新專案
- **目錄結構**（含 AGENTS.md）：
  ```
  openspec/
  ├── AGENTS.md       ← AI 工作流文件（告訴 AI 該怎麼用 OpenSpec）
  ├── project.md      ← 技術棧、慣例、架構約束
  ├── specs/          ← 當前系統規格的真相源
  └── changes/        ← 進行中的提案（含 archive 子目錄）
  ```
- **規格格式核心**：SHALL/MUST 關鍵字 + `#### Scenario` 標題 + WHEN/THEN/AND 語句
- **Delta 格式**：`## ADDED` / `## MODIFIED` / `## REMOVED` 三種 header
- **CLI 工具完整表**：`list`（查看活躍 changes 或既有 specs）、`validate [name]`（格式合規檢查）、`show [name]`（詳細提案資訊）、`view`（互動式儀表板）
- **何時不需要 Proposal**（繞過 SDD 的合法情境）：
  1. Bug fix（讓程式碼符合已有規格）
  2. 錯字與格式調整
  3. 非破壞性依賴更新
  4. 不改變規格行為的設定變更
  5. 為已記錄功能補充測試
- **多 AI 支援**：Claude、OpenAI、Cursor、GitHub Copilot、Gemini；不需要 API key

## Entities Mentioned
- [[高見龍]] — 作者
- [[OpenSpec]] — 本文主題框架
- [[Kiro]] — 對比工具（greenfield-first）
- [[GitHub-spec-kit]] — 對比工具（greenfield-first）
- [[Tessl]] — 同類工具（Spec-as-source 層）

## Concepts Mentioned
- [[OpenSpec工作流]] — 3 指令是 8 階段流程的使用者介面
- [[OpenSpec文件格式與驗證]] — AGENTS.md、project.md 結構；何時不需要 Proposal
- [[Spec驅動開發]] — 方法論背景
- [[SDD成熟度三層次]] — OpenSpec 屬 Spec-anchored 層

## Contradictions/Tensions
- 本文提到 `AGENTS.md` 和 `project.md` 作為專案初始化文件，而既有 wiki 頁面（基於 ForceInjection repo）提到 `config.yaml` 作為 Context Anchor。兩者可能是不同版本或不同命名，待確認是否為同一概念的演進。

## Questions Raised
- `AGENTS.md` 與 `config.yaml` 是否為同一概念的不同版本名稱？
- 「何時不需要 Proposal」清單是否完整？是否有更細的決策規則？
