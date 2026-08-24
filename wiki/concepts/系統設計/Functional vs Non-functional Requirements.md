---
title: Functional vs Non-functional Requirements
type: concept
tags: [system-design, interview, requirements, scoping]
created: 2026-04-20
updated: 2026-04-20
sources: [2026-04-20_aritra-sen_beginners-guide-system-design.md]
---

# Functional vs Non-functional Requirements

系統設計面試的第一步，也是最常被新手跳過的步驟。

> 先定義「做什麼」和「如何表現」，才能設計出有方向的系統。

---

## Functional Requirements（功能需求）

**定義**：系統要提供哪些功能？

目標：把「Design Instagram」這樣的模糊題縮窄成可在面試內完成的範圍。

**操作方式**：
1. 列出所有可能功能
2. 和面試官確認優先順序
3. 聚焦在面試官最在意的1–3個核心功能

**範例（Design Instagram）**：
- 用戶上傳照片 ✅（核心）
- 用戶瀏覽 Feed ✅
- 按讚/留言 （次要）
- Reels、Stories （通常不在範圍內）
- 搜尋功能 （視情況）

---

## Non-functional Requirements（非功能需求）

**定義**：系統應該如何表現？不是「做什麼」，而是「做得多好」。

| 需求類型 | 範例問題 |
|---------|---------|
| 規模 | 預期用戶數？DAU 是多少？ |
| 讀/寫比 | 讀重還是寫重？ |
| 一致性 | 強一致還是最終一致可接受？ |
| 可用性 | 需要幾個 9（99.9% vs 99.99%）？ |
| 延遲 | P99 延遲要在多少 ms 內？ |
| 安全 | 需要認證嗎？有隱私合規要求嗎？ |
| 儲存 | 資料需要保留多久？ |

---

## 為什麼這步很重要

非功能需求直接決定架構選型：

```
強一致性需求 → SQL（ACID）
最終一致性可接受 → NoSQL（AP 系統）

讀重 → Cache + Read Replica
寫重 → Queue + Write-optimized DB

高可用 → Load Balancer + Failover
低延遲 → CDN + Cache
```

沒有釐清需求，後面的設計就沒有根基可辯護。

---

## 相關概念

- [[系統設計面試模板]] — 這是 Step 1
- [[Back-of-the-Envelope 估算]] — 從非功能需求出發進行量化
- [[SQL vs NoSQL 選型框架]] — 一致性需求決定資料庫選型

## 來源

- [[aritra-sen-beginners-guide-system-design|A Beginner's Guide to System Design]]
