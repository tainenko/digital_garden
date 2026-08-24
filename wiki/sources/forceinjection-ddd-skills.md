---
title: ForceInjection/domain-driven-design-skills
type: source-summary
tags: [DDD, AI Agent, 技能, OpenSpec, 流水線]
created: 2026-05-25
updated: 2026-05-25
sources: [forceinjection-ddd-skills]
---

# ForceInjection/domain-driven-design-skills

## Origin

- **Title**: domain-driven-design-skills — AI Agent 導向的 DDD 完整建模技能系統
- **Author/Org**: [[ForceInjection]]（同作者亦有 OpenSpec-practise repo）
- **URL**: https://github.com/ForceInjection/domain-driven-design-skills
- **Date**: 2026-05（活躍維護中）
- **Stars**: 9（早期 repo，技術深度高）

## Key Takeaways

1. **9 技能 × 5 階段完整流水線**：覆蓋從「模糊需求 → 領域發現 → 戰略建模 → 戰術建模 → 模型驗證 → OpenSpec 工程規格」的完整閉環，解決現有 DDD 工具「各做各的」碎片化問題。

2. **SKILL.md 契約標準**：每個技能強制包含七節結構——觸發條件、輸入要求、執行步驟（5–7步）、輸出產物、驗證清單、回溯觸發條件、範例。使結構化輸出可機器解析。

3. **回溯觸發機制（Backtrack Triggers）**：8 條品質門禁條件，當下游階段發現問題時可觸發退回上游技能重跑——例如 `ddd-model-review` 發現 invariant 表達率 < 60%，強制退回 `ddd-aggregates`。

4. **非線性入口**：不需從頭開始。5 種場景入口（新專案 / 現有系統 / 局部深化 / 品質稽核 / 規格生成）各自有建議起始技能。

5. **ddd-openspec-bridge**：第 5 階段把 DDD 戰術產物（aggregate catalog、event catalog、repository interface）直接轉換為 OpenSpec 的 proposal/design/spec/tasks 結構，實現「DDD 建模 → AI 工程實作」無縫銜接。

6. **盲跑驗證（Blind-Run Protocol）**：用 Cargo Shipping 標準 DDD 樣本案例盲測，加權分數 **85.8%（B+ 合格）**，3/3 回溯觸發測試通過。

7. **防止無限迴圈設計**：同一回溯路徑最多執行 3 次；第 3 次觸發時升級為「需人工架構決策」，避免 AI Agent 陷入死迴圈。

8. **外部生態系引用**：以 Git submodule 納入 10 個代表性開源 DDD 技能（wondelai、robust-skills、antigravity-awesome-skills 等）作為比較基準與可選強化。

9. **語言邊界 ≠ 模型邊界**（P0 改進點）：現有技能需加強「語言邊界」與「模型邊界」的區分——這是最常見的 DDD 建模錯誤之一。

10. **覆蓋範圍邊界明確**：明確不涵蓋具體程式碼生成、測試策略、架構合規驗證——這讓技能保持聚焦且易維護。

## Entities Mentioned

- [[ForceInjection]] — 作者 GitHub 組織；同時維護 [[OpenSpec]]（openspec-practise repo）
- [[OpenSpec]] — 第 5 階段 ddd-openspec-bridge 的目標格式

## Concepts Mentioned

- [[DDD AI Agent技能流水線]] — 9技能×5階段核心架構
- [[DDD回溯觸發機制]] — 品質門禁與非線性反饋設計
- [[DDD領域驅動設計]] — 理論基礎（Eric Evans 戰略/戰術設計）
- [[OpenSpec工作流]] — ddd-openspec-bridge 的輸出目標
- [[BDD行為驅動開發]] — Scenario 格式（Given/When/Then）即為 BDD
- [[Spec驅動開發]] — 本技能系統的上層方法論

## Contradictions / Tensions

- 論文稱「任何規模團隊可用」，但技能執行品質高度依賴 AI 對領域的理解深度。中小團隊若缺乏 DDD 基礎，可能產生形式合格但語義空洞的文件。
- 與 [[SDD適用邊界]] 的批評有對話關係：本系統透過分階段技能 + 回溯機制試圖解決「規格維護稅」問題，但驗證僅用一個案例，泛化性待觀察。

## Questions Raised

- `ddd-openspec-bridge` 產出的 tasks 是否能直接餵給 `ddd-aggregates` 下游的 Claude Code 實作？端到端自動化程度如何？
- 10 個外部 submodule 技能是否有比較分析文件？forceinjection 自研 vs 社群哪方更強？
- Cargo Shipping 是相對成熟的 DDD 教學案例，下一個驗證案例應選用更模糊、邊界不清的真實業務問題。
