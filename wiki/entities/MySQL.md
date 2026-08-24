---
title: MySQL
type: entity
tags: [資料庫, rdbms, mysql, innodb, 後端]
created: 2026-06-22
updated: 2026-06-22
sources: [zhuchangwu-database-interview-30-questions.md]
---

# MySQL

最主流的開源關聯式資料庫（RDBMS），後端系統的預設持久層之一。原由 MySQL AB 開發，現屬 Oracle。

- **預設儲存引擎**：InnoDB（支援交易、行鎖、外鍵、崩潰自動恢復、聚簇索引）；另有 MyISAM（表鎖、不支援交易、支援全文索引）。
- **索引結構**：B+ 樹 → 見 [[MySQL索引與B+樹]]。
- **交易**：ACID，預設隔離級別 Repeatable Read，MVCC + Gap Lock → 見 [[MySQL事務與隔離級別]]。
- **高可用**：主從複製（async / semi-sync / sync）+ GTID → 見 [[MySQL主從複製]]。
- **調優**：慢查詢日誌 + `EXPLAIN` 執行計畫 → 見 [[MySQL慢查詢與執行計畫]]。

## 相關頁面
- [[MySQL索引與B+樹]]、[[MySQL事務與隔離級別]]、[[資料庫正規化]]、[[MySQL主從複製]]、[[MySQL慢查詢與執行計畫]]
- [[Redis]] — 常作為 MySQL 前的快取層
- [[SQL核心概念]]、[[SQL vs NoSQL 選型框架]]
- 來源：[[數據庫面試簡答、30道高頻面試題（赐我白日夢）]]
</content>
