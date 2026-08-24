---
title: Coinbase 面試全流程（2026）
type: concept
tags: [面試, Coinbase, LLD, 系統設計, SQL, 行為面試, CodeSignal]
created: 2026-05-04
updated: 2026-05-04
sources: [techprep-coinbase-interview-process-2026]
---

# Coinbase 面試全流程（2026）

## 整體結構

**3–5 個階段，歷時 6–8 週**；強調 practical real-world coding，弱化傳統演算法題。

| 階段 | 時長 | 重點 |
|------|------|------|
| Recruiter Screen | 30 分鐘 | 背景、crypto 興趣、文化適配；常問「Why Coinbase?」 |
| Online Assessment（OA） | — | CodeSignal：Progressive Coding + 快速推理測驗 + Workstyles Survey |
| Technical Phone Screen | 60 分鐘 | Coding + 輕量技術討論；晉升 Onsite 的門檻關 |
| Virtual Onsite Loop | 4 × 60–90 分鐘 | Tech Execution / Domain / System Design / Behavioral |
| Hiring Manager Round | 45 分鐘 | 所有權心態、過往影響力、團隊契合度 |

---

## 各輪準備重點

### 1. OA — Progressive Coding

OA coding 題**不是孤立演算法題，而是逐步加功能**：

- 典型結構：先建 key-value store → 加 TTL → 加 prefix search
- 直接對應 [[Coinbase_HA_總覽]] 中的 Version A/B 題目池
- DSA 重點：graph（shortest path）、hash map、heap（Top K）、concurrency

### 2. LLD — Tech Execution Round（最關鍵）

**Coinbase 最具決定性的一關**，候選人最常引用為勝敗關鍵。

- 題型：建立 order management system、trade matching engine、in-memory banking system
- **Code modularity 比演算法效率更重要**——可讀、可擴充、方法小而可測試
- 常見 mid-round 加碼：面試官在進行中新增需求，考察系統可演化性
- 準備策略：
  - 練習「系統演化」而非只求解題
  - 命名規範、方法拆分、預留擴充點
  - 參考：[[Banking_System]]、[[In_Memory_Database]]

### 3. System Design — 金融基礎架構

題目不是通用的 Twitter/URL Shortener，而是金融場景：

| 常見題型 | 核心考點 |
|---------|---------|
| Real-time price aggregator | Caching、Read scaling、一致性 vs 可用性 |
| Idempotent payment pipeline | Idempotency Key、重試安全、Outbox Pattern |
| High-throughput notification system | Fan-out、訊息佇列、冪等送達 |

重點：**consistency vs availability 在金融資料下的取捨論述**。

### 4. SQL & Database Design — Domain Round

後端候選人報告率最高的考點：

- **Multi-currency schema 設計**：幣種、匯率、帳戶、交易的 ER 設計
- **Ledger table 查詢最佳化**：indexing strategy、高寫入量場景
- **Pagination（2025 候選人明確報告）**：
  - Cursor-based pagination vs Offset pagination
  - 何時選用：transaction history API 為典型案例

```
Offset pagination 問題：
  - 大偏移量時 DB 必須掃過並丟棄前 N 筆 → 效能差
  - 插入/刪除期間頁面不穩定

Cursor-based pagination 優點：
  - 以 indexed column（如 id 或 created_at）為遊標 → O(log n)
  - 結果穩定，適合即時資料
```

### 5. Behavioral — 文化信條驗證

Coinbase 的行為面試**主動驗證**以下文化信條：

- **Act like an Owner** — 主動承擔、端到端責任
- **Efficient Execution** — 明確優先級、快速交付、減少摩擦

**2025/2026 新增注意事項**：面試官明確詢問候選人是否接受 **apolitical 職場文化**（專注公司 crypto 使命，不涉及廣泛政治/社會話題）。需準備直接且真誠的回應。

準備方式：用 STAR 結構組織具體案例，強調**可量化的影響力**。

---

## 準備優先序

```
Priority 1 — LLD（Tech Execution）
  → 最具決定性；直接練 Banking System / In-Memory DB 等 progressive 題

Priority 2 — OA（Progressive Coding）
  → 熟悉 progressive 結構；複習 graph/hash map/heap/TTL

Priority 3 — System Design
  → 聚焦金融場景；idempotency、partition tolerance、caching

Priority 4 — SQL
  → Cursor-based pagination、大 ledger 索引、multi-currency schema

Priority 5 — Behavioral
  → 5 個 STAR 故事覆蓋 Owner + Execution；備好 apolitical culture 回應
```

---

## 薪資參考（2026/03，Levels.fyi）

| 職級 | Median Total Comp | Base | Bonus | Equity/年 |
|------|------------------|------|-------|-----------|
| IC3（Entry-Level） | $204K | ~$160K | ~$30K | ~$14K |
| IC5（Senior） | $410K | ~$240K | ~$60K | ~$110K |
| IC8（Principal+） | $1.19M+ | ~$350K | ~$150K | ~$700K+ |
| EM/Staff | $564K–$987K | — | — | — |

> 注意：Equity 受 COIN 股價波動影響大，計算 TC 時保守估算 equity 比例。

---

## 相關頁面

- [[Coinbase_HA_總覽]] — OA CodeSignal 格式、兩種題目池、各關難度
- [[Banking_System]] — LLD / OA 練習題：完整 Go 解法
- [[In_Memory_Database]] — LLD / OA 練習題：完整 Go 解法
- [[系統設計面試模板]] — System Design 通用框架
