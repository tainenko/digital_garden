---
title: Coinbase's Interview Process (2026)（TechPrep）
type: source-summary
tags: [面試, Coinbase, CodeSignal, LLD, 系統設計, SQL, 行為面試]
created: 2026-05-04
updated: 2026-05-04
sources: []
---

# Coinbase's Interview Process (2026)（TechPrep）

## Origin

- **Title**: Coinbase's Interview Process (2026)
- **Source**: https://www.techprep.app/blog/coinbase-interview-process
- **Publisher**: TechPrep
- **Date**: 2026

---

## Key Takeaways

1. **5 階段流程，6–8 週**：Recruiter Screen → OA（CodeSignal）→ Technical Phone Screen → Virtual Onsite Loop（4 rounds）→ Hiring Manager Round
2. **OA 是 progressive coding**：從基本題逐步加功能（如先建 key-value store，再加 TTL、prefix search），而非孤立演算法題
3. **LLD（Tech Execution）是最具決定性的關卡**：強調 code modularity 勝於演算法效率；要求能應付 mid-round 加需求，系統要可演化
4. **System Design 聚焦金融基礎架構**：real-time price aggregator、idempotent payment pipeline、high-throughput notification system；常考 consistency vs availability trade-off
5. **SQL Domain Round**：multi-currency schema 設計、大 ledger table 查詢、2025 候選人明確報告被問 cursor-based vs offset pagination
6. **行為面試綁定文化信條**：「Act like an Owner」、「Efficient Execution」；面試官會主動驗證；2025/2026 面試官明確詢問候選人對 apolitical 職場的接受度
7. **DSA 不是主角**：以 graph（shortest path）、hash map、concurrency 為主；出現在 OA 階段

---

## Entities Mentioned

- [[Coinbase]] — 受訪公司

---

## Concepts Mentioned

- [[Coinbase_面試全流程]] — 完整 5 階段流程、各輪準備重點
- [[Coinbase_HA_總覽]] — OA（CodeSignal）格式詳解
- [[Banking_System]] — OA 中的 progressive coding 範例題
- [[In_Memory_Database]] — OA 中的 progressive coding 範例題

---

## Contradictions / Tensions

- 文中強調「DSA 不是主角」，但 OA 仍含 DSA 題（尤其 graph、heap）；不可完全忽略

---

## Questions Raised

- Hiring Manager Round 的具體問題範例？
- Domain round 是否所有職能都有 SQL 題，還是僅後端？
- Virtual Onsite 的 4 rounds 順序是否固定？
