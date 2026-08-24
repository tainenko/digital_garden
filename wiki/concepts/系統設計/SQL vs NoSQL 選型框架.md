---
title: SQL vs NoSQL 選型框架
type: concept
tags: [system-design, database, sql, nosql, decision-framework]
created: 2026-04-20
updated: 2026-04-20
sources: [2026-04-20_aritra-sen_beginners-guide-system-design.md]
---

# SQL vs NoSQL 選型框架

系統設計面試中最常見的決策點之一。選型不是偏好問題，而是由系統需求決定。

> 重點：不需要選定具體產品（不必說 MySQL vs PostgreSQL），但必須說清楚選 SQL 或 NoSQL 的理由。

---

## 三個決策維度

### 1. 資料結構：結構化 vs 非結構化

| 情況 | 建議 |
|------|------|
| 資料有明確 schema、實體間有關聯 | SQL（關聯式） |
| 資料結構多變、不規則、巢狀 | NoSQL（文件型如 MongoDB） |
| 鍵值對查詢為主 | NoSQL（KV store 如 Redis、DynamoDB） |
| 大量圖關係 | NoSQL（圖資料庫如 Neo4j） |

### 2. 一致性要求

| 需求 | 建議 |
|------|------|
| 強一致性（ACID 事務） | SQL（天生支援） |
| 最終一致性可接受 | NoSQL（多數支援 AP，犧牲 C） |

**典型案例**：
- 支付/金融系統 → SQL（ACID 是必須，不接受資料不一致）
- 社群媒體貼文計數 → NoSQL AP（少算幾個讚沒關係）

### 3. 讀寫模式

| 模式 | 建議 |
|------|------|
| 讀重、查詢複雜、多表 JOIN | SQL（查詢優化器強） |
| 寫重、高吞吐量、需水平擴展 | NoSQL（水平擴展更容易） |
| 需要全文搜尋 | Elasticsearch（特殊用途） |

---

## 常見面試情境判斷

| 系統 | 建議 | 理由 |
|------|------|------|
| 支付/銀行交易 | SQL | ACID 事務必須 |
| 用戶個人資料 | SQL 或 NoSQL | 結構固定→SQL；需快速讀取→NoSQL |
| 社群媒體 Feed | NoSQL | 寫重、非結構化、需水平擴展 |
| 即時聊天訊息 | NoSQL | 大量寫入、時序查詢 |
| 電商商品目錄 | NoSQL | 屬性多變、讀重 |
| 訂單管理 | SQL | 強一致性、多表關聯 |

---

## 延伸：資料庫設計其他考量

完成 SQL vs NoSQL 選型後，還需要考慮：

- **Replication**（複製）：備援、讀取擴展
- **Sharding**（分片）：水平切分資料，處理超大資料量
- **Indexing**（索引）：加速特定查詢
- **Object Storage**：圖片/影片/Blob 另外存（如 S3），不放在主資料庫

---

## 相關概念

- [[分散式系統基礎概念]] — CAP 定理是選型的理論基礎
- [[Back-of-the-Envelope 估算]] — 讀寫比例估算驅動選型
- [[系統設計面試模板]] — 選型發生在 Step 5（資料庫設計）

## 來源

- [[aritra-sen-beginners-guide-system-design|A Beginner's Guide to System Design]]
