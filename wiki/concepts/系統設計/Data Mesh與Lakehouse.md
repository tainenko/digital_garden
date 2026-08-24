---
title: Data Mesh 與 Lakehouse
type: concept
tags: [system-design, data-mesh, lakehouse, data-governance, data-product, 2026]
created: 2026-05-15
updated: 2026-05-15
sources: [dev-to-system-design-2026]
---

# Data Mesh 與 Lakehouse

2026 系統設計的第三支柱：去中心化資料治理 + 湖倉一體儲存，解決規模化後的資料工程瓶頸。

## Data Mesh：資料所有權去中心化

### 核心問題
傳統集中式資料工程：一個中央資料團隊負責所有管道 → 成為組織瓶頸，資料品質責任模糊，業務知識流失。

### Data Mesh 四大原則

| 原則 | 說明 |
|------|------|
| **Domain Ownership** | 各業務領域團隊擁有並維護自己的資料 |
| **Data as Product** | 資料視為產品：有 SLA、有版本、有文件、有消費者 |
| **Self-serve Platform** | 中央提供基礎設施平台，各域團隊自助使用 |
| **Federated Governance** | 去中心化治理：各域有自主權，同時遵循全域互通標準 |

### Data Contracts（資料契約）
- 資料生產者與消費者之間的正式 API 契約
- 包含：Schema 定義、SLA（freshness / availability）、欄位語意說明
- 違約自動告警，取代人工溝通與口頭承諾
- 工具：Soda、Great Expectations、dbt contracts

## Lakehouse：湖倉一體

### 問題背景
- **Data Lake**：儲存便宜，格式靈活，但查詢慢、無 ACID、難治理
- **Data Warehouse**：查詢快、強 Schema，但儲存貴、不支援非結構化資料

### Lakehouse = 兩者最佳化整合

```
原始資料 (Object Storage: S3/GCS)
     ↓  [Delta Lake / Apache Iceberg / Apache Hudi]
開放格式 + ACID + Time Travel + Schema Evolution
     ↓
統一查詢層 (Spark / Trino / Databricks SQL / BigQuery)
     ↓
BI / ML / AI 應用
```

### 關鍵技術選擇

| 技術 | 職責 | 代表工具 |
|------|------|---------|
| Table Format | ACID + 版本管理 | Delta Lake、Apache Iceberg |
| Storage | 物件儲存 | S3、GCS、Azure Blob |
| Compute | 查詢引擎 | Spark、Trino、DuckDB |
| Catalog | 資料發現 | Unity Catalog、AWS Glue |

## Data Mesh × Lakehouse 的協作關係

- Lakehouse 提供「集中式儲存基礎設施」，Data Mesh 決定「誰負責哪部分資料的品質與治理」
- 每個 Domain 的資料作為 Lakehouse 中的獨立 Table 集合，附帶 Data Contract
- 中央平台團隊維護 Lakehouse 的統一 Catalog 與查詢引擎

## 與 AI-Native 架構的連結

- Feature Store（AI 特徵儲存）建立在 Lakehouse 之上
- AI 訓練資料的版本管理與 Time Travel 直接使用 Lakehouse 的 ACID 能力
- Causal Tracing 的歷史分析需要 Lakehouse 的長期資料保留

## 相關頁面

- [[AI-Native架構]] — Lakehouse 作為 Feature Store 的底層
- [[FinOps與GreenOps]] — 資料層的儲存成本優化（冷熱分層）
- [[Observability 3.0 Causal Tracing]] — 日誌與指標的長期儲存需要 Lakehouse
- [[系統設計核心技術棧]] — 資料庫選型在 Lakehouse 後的演進
