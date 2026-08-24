---
title: OpenSpec + Superpowers：Spec 驅動開發框架完整指南
type: source-summary
tags: [openspec, superpowers, spec-driven-development, vibe-coding, claude-code]
created: 2026-04-28
updated: 2026-04-28
sources: [openspec-superpowers-multi-source]
---

# OpenSpec + Superpowers：Spec 驅動開發框架完整指南

## Origin

- **來源類型**：多篇網路文章合成
- **主要來源**：
  - https://www.heyuan110.com/posts/ai/2026-04-09-claude-code-openspec-superpowers/（Claude Code + OpenSpec + Superpowers 整合指南）
  - https://docs.bswen.com/blog/2026-03-27-openspec-vs-superpowers/（OpenSpec vs Superpowers 框架比較）
- **日期**：2026-03-27 / 2026-04-09

---

## Key Takeaways

- **OpenSpec** 是 Fission-AI 開源的 spec 驅動開發框架，核心價值是「決策可追溯」——解答「為什麼做這個改動？」
- **Superpowers** 是 obra 開源的技能框架，安裝在 Claude Code 中，核心價值是「程式碼品質執行」——解答「這段程式碼正確嗎？」
- 兩者解決不同的問題，可以疊加使用，但**不會自動串聯**，需手動在 CLAUDE.md 中整合
- OpenSpec 三個核心指令：`explore`（探索）→ `propose`（生成四份文件）→ `apply`（逐任務實作）→ `archive`（版本化決策記錄）
- Superpowers 強制執行 TDD（RED → GREEN → REFACTOR）、自動 code review、git worktree 隔離
- **三道牆**：Claude Code 解 Wall 1（自動化），Superpowers 解 Wall 2（工程紀律），只有 OpenSpec 的 Delta/Archive 機制能解 Wall 3（設計理由消失）
- 實際案例：Next.js + PostgreSQL 部落格系統，8 小時完成，87% 測試覆蓋率，上線第一週零 bug
- 對小型專案（< 2 小時）用 Claude Code 單獨即可；中型（4–16 小時團隊）才值得跑完整 triple stack

---

## Entities Mentioned

- [[OpenSpec]] — Fission-AI 開源的 spec 驅動開發框架
- [[Superpowers]] — obra 開源的 Claude Code 技能框架，強制 TDD
- [[Cursor]] — 支援 Superpowers 的主流 Vibe Coding IDE

---

## Concepts Mentioned

- [[Spec驅動開發]] — 用結構化規格文件驅動 AI 實作的方法論
- [[OpenSpec工作流]] — 8 階段：explore → propose → design → specs → tasks → apply → verify → archive
- [[Superpowers技能框架]] — TDD + code review + git worktree 的自動化執行框架
- [[Vibe Coding工具比較]] — OpenSpec/Superpowers 屬於「框架層」，在工具之上
- [[Vibe Coding風險與限制]] — 這兩個框架正是為了緩解 Vibe Coding 風險而生

---

## Contradictions / Tensions

- Superpowers 採用多 agent（controller + implementer + reviewer），token 消耗顯著高於 OpenSpec 的單 agent 模式——「更好的品質」vs「更高的成本」
- OpenSpec 強調決策文件，但若不執行 `/opsx:archive` 則舊版規格不會自動版本化，需要自律
- 有觀點認為同時用三個系統（Claude Code + OpenSpec + Superpowers）對小型團隊是 overkill

---

## Questions Raised

- Superpowers 的多 agent 架構在 token 成本上是否可接受？何時值得？
- OpenSpec 的 delta spec system 如何與 git branching model 整合？
- 這類 spec 框架對非技術 founder 的可及性如何？
