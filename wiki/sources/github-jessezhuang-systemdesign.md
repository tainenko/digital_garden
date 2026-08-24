---
title: JesseZhuang/SystemDesign
type: source-summary
tags: [系統設計, 分散式系統, DDIA, 面試, AWS, 學習資源]
created: 2026-05-13
updated: 2026-05-13
sources: [github-jessezhuang-systemdesign]
---

# JesseZhuang/SystemDesign

## Origin

- **URL**: https://github.com/JesseZhuang/SystemDesign
- **Author**: [[JesseZhuang]]（同 InCodeLearning-Python3 作者）
- **Description**: 系統設計面試綜合學習指南
- **License**: GPL-3.0
- **Date Ingested**: 2026-05-13

## Key Takeaways

- 三大區塊：`basics/`（24 個分散式系統基礎主題）、`designs/`（22 個完整系統設計案例）、`DDIA/`（《Designing Data-Intensive Applications》12 章筆記）
- `basics/` 涵蓋：一致性雜湊、CAP 定理、分片、索引、快取、負載均衡、訊息佇列、代理、WebSocket、Paxos 共識算法
- `designs/` 22 個案例，其中未見於本 wiki 的新主題：**時序資料庫**（TimescaleDB + Pinterest Goku）、**分散式任務排程**（Cassandra + Message Broker + ZooKeeper）、**廣告點擊系統**（已有）、**Pastebin**、**Pinterest**、**航空訂票系統**
- `DDIA/` 全 12 章對應《Designing Data-Intensive Applications》（Martin Kleppmann 著），ch01–ch12
- 推薦資源：Gaurav Sen 系列影片、David Malan CS75 可擴展性講座、DDIA 書籍
- 面試建議：事先研讀目標公司工程 blog，因面試題通常對應其實際架構挑戰

## Entities Mentioned

- [[JesseZhuang]] — 作者
- [[Gaurav Sen (GKCS)]] — 推薦學習資源

## Concepts Mentioned

- [[時序資料庫設計]] — TimescaleDB + Pinterest Goku（此 repo 新增）
- [[分散式任務排程系統]] — Cassandra + Message Broker + ZooKeeper（此 repo 新增）
- [[分散式系統基礎概念]] — basics/ 資料夾覆蓋內容

## Contradictions/Tensions

- DDIA 為 2017 年著作，部分內容（Hadoop、Spark 生態）已有演進，需對照 2024–2026 最新實踐
- `designs/` 案例無日期標記，難以確認是否為最新設計

## Questions Raised

- JesseZhuang 的 DDIA 筆記深度如何？是否有摘要或原創分析？
- `aws/` 和 `google/` 資料夾內容是否值得單獨 ingest？
