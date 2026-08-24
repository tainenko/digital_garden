---
title: RESHADED面試框架
type: concept
tags: [system-design, interview, framework, reshaded]
created: 2026-04-20
updated: 2026-04-20
sources: [2026-04-20_multi-source_system-design-interview-synthesis.md]
---

# RESHADED 面試框架

SystemDesignHandbook.com 提出的系統設計面試框架，用縮寫幫助記憶完整流程的八個面向。

> 優點：字母縮寫有助於不遺漏步驟，尤其適合容易緊張的面試情境。

## 八個字母

| 字母 | 代表 | 說明 |
|------|------|------|
| **R** | Requirements | 需求釐清（功能＋非功能）|
| **E** | Estimation | 規模估算（DAU / 讀寫 / 儲存）|
| **S** | Storage | 資料儲存策略（SQL/NoSQL/Object Storage）|
| **H** | High-level design | 高層架構圖 |
| **A** | APIs | API 設計與介面定義 |
| **D** | Detailed design | 深度元件設計 |
| **E** | Evaluation | 評估與 trade-off 辯護 |
| **D** | Distinctive component | 識別並深入系統最獨特/最難的元件 |

## 與其他框架的差異

RESHADED 相比 [[系統設計面試模板]]（Aritra Sen 的 6 步）的新增之處：

- **S（Storage）** 提前獨立處理：儲存策略在高層設計之前就決定，而不是放在 DB 設計步驟
- **D（Distinctive component）** 最後步驟：明確要求找出「最難/最獨特」的元件做深入，避免深挖瑣碎細節

## SPARCS：非功能需求記憶法

配套的非功能需求縮寫，在 **R（Requirements）** 步驟使用：

| 字母 | 代表 |
|------|------|
| **S** | Scalability（可擴展性）|
| **P** | Performance（效能）|
| **A** | Availability（可用性）|
| **R** | Reliability（可靠性）|
| **C** | Consistency（一致性）|
| **S** | Security（安全性）|

見[[Functional vs Non-functional Requirements]]。

## SLIC FAST：常用建構積木記憶法

面試中常用到的技術元件縮寫：

| 字母 | 代表 |
|------|------|
| **S** | Search（搜尋引擎，如 Elasticsearch）|
| **L** | Load balancer |
| **I** | Interaction with CDN |
| **C** | Cache（Redis 等）|
| **F** | Front-end servers |
| **A** | Analytics |
| **S** | Storage（DB / Object Storage）|
| **T** | Task queues（Kafka / RabbitMQ）|

## 實際應用範例（TinyURL）

RESHADED 框架套用到 URL Shortener：

- **R**: 縮短 URL、追蹤點擊、高可用、低延遲
- **E**: 76 次縮短/秒、7,600 次重定向/秒、需 ~1,600 台伺服器
- **S**: 短網址→長網址 KV 存儲，NoSQL（DynamoDB / Cassandra）
- **H**: Client → LB → App Server → Cache → DB
- **A**: `POST /shorten {url}` → `{short_code}`；`GET /{code}` → 302 redirect
- **D**: 短碼生成（Hash vs 計數器）、Cache 策略、DB Sharding
- **E**: Hash 衝突問題、Hot key 問題、跨區一致性 trade-off
- **D**: 短碼生成服務是最獨特的元件——深挖一致性雜湊與衝突處理

## 相關概念

- [[系統設計面試框架比較]] — 與其他框架的對照
- [[面試時間分配]] — RESHADED 各步驟的時間建議
- [[Functional vs Non-functional Requirements]] — R 步驟的詳細說明
- [[Back-of-the-Envelope 估算]] — E 步驟的詳細說明
- [[系統設計核心技術棧]] — SLIC FAST 各元件的詳細說明

## 來源

- [[system-design-interview-synthesis|系統設計面試資源綜合整理]]
