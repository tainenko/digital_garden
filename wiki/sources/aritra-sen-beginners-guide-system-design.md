---
title: "A Beginner's Guide to System Design"
type: source-summary
tags: [system-design, interview, backend, distributed-systems, career]
created: 2026-04-20
updated: 2026-04-20
sources: [2026-04-20_aritra-sen_beginners-guide-system-design.md]
---

# A Beginner's Guide to System Design

**作者**: [[Aritra Sen]]（前 Google L4 → Meta Reality Labs）
**原文**: https://medium.com/@sentalkssane/a-beginners-guide-to-system-design-76d64689788b
**發布**: 2024-09-18 ｜ **收錄**: 2026-04-20

---

## 核心主張

> 系統設計面試最大的問題不是資源不夠，而是**沒有結構**。用一套固定模板應對面試，加上紮實基礎與大量練習，就能在 45–60 分鐘內產出可辯護的架構設計。

---

## Key Takeaways

1. **工作經驗幫助有限**：系統設計面試考的是可擴展分散式後端，和日常工作脫節，必須另外準備。
2. **五步準備法**：基礎 → 模板 → 練題 → Mock → 公司客製化。
3. **模板是時間管理工具**：有結構才不會在45分鐘內走偏，見[[系統設計面試模板]]。
4. **Queue 是神器**：非同步處理是解決高頻請求的萬能藥，要理解「何時用、為何用」。
5. **估算先於設計**：Back-of-the-envelope 估算直接影響資料庫選型、是否需要 Cache/CDN/Shard。
6. **SQL vs NoSQL 不是偏好**：由一致性需求、讀寫比例、資料結構決定，見[[SQL vs NoSQL 選型框架]]。
7. **設計可以辯護**：面試官在乎的是你能不能 reason about 你的選擇，不只是答案對不對。
8. **練題要跨資源**：同一題看多個解法，理解不同 trade-off。
9. **公司題目有規律**：LeetCode Discuss 按公司標籤搜尋，高重複率。
10. **Whiteboard 要練**：用滑鼠在 Excalidraw 畫架構比想像中難。

---

## 提到的實體

- [[Aritra Sen]] — 作者，前 Google、現 Meta Reality Labs
- [[Gaurav Sen (GKCS)]] — YouTube 教學資源，系統設計基礎概念
- [[Alex Xu]] — System Design Interview Vol.I & II 書籍作者

## 提到的概念

- [[系統設計面試模板]] — 6步結構：需求 → 估算 → API → 高層設計 → DB設計 → 詳細選型
- [[Functional vs Non-functional Requirements]] — 面試第一步：釐清要做什麼、要如何表現
- [[Back-of-the-Envelope 估算]] — 量化需求，驅動架構選型
- [[SQL vs NoSQL 選型框架]] — 一致性、讀寫比、資料結構三維決策
- [[分散式系統基礎概念]] — Load Balancing、CAP 定理、Caching、Queue 等

---

## 矛盾/張力

- 建議「避免太早深入細節」，但面試官通常希望看到深度——時間拿捏是關鍵技能，文章未提供具體時間分配建議。
- 強調「practice the obvious」（練習重複題型），但實際面試有時會出現變形題，純背題型風險存在。

## 待追問題

- CAP 定理的三種組合（CA/CP/AP）在常見系統設計題中各有哪些代表案例？
- Message Queue vs Pub-Sub 的選型邏輯是什麼？
- 什麼情況下需要 Sharding？Consistent Hashing 如何解決 Sharding 的再均衡問題？
