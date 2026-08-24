---
title: MySQL慢查詢與執行計畫
type: concept
tags: [mysql, 慢查詢, explain, 執行計畫, 效能調優, 面試]
created: 2026-06-22
updated: 2026-06-22
sources: [zhuchangwu-database-interview-30-questions.md]
---

# MySQL 慢查詢與執行計畫（EXPLAIN）

定位並優化慢 SQL 是 MySQL 調優的核心流程。

## 找出慢查詢
- 查看慢查詢閾值：
  ```sql
  SHOW GLOBAL VARIABLES LIKE '%long_query_time%';
  ```
- 開啟慢查詢日誌後，用 `mysqldumpslow` 取出最慢的查詢：
  ```bash
  mysqldumpslow -s a1 -n 10 mysql.slow_log   # 取最慢 10 筆
  ```

## EXPLAIN 執行計畫分析
在 SQL 前加 `EXPLAIN` 觀察優化器選擇，重點欄位：
- **type**：存取策略，效率由優到劣：`const` > `ref` > `range` > `index` > `ALL`（`ALL` = 全表掃描，需警惕）。
- **key**：實際使用的索引（為 NULL 表示未走索引）。
- **rows**：預估掃描列數，越少越好。
- **Extra**：附加資訊，`Using index`（覆蓋索引，好）、`Using filesort` / `Using temporary`（需排序/暫存表，常為優化點）。

## 索引優化原則
- 適合建索引的欄位：出現在 **WHERE 子句**或 **JOIN 連接條件**中的欄位。
- 善用**覆蓋索引**免回表、遵守**最左前綴原則**（見 [[MySQL索引與B+樹]]）。
- **勿過度索引**：每個索引都增加寫入（INSERT/UPDATE/DELETE）時的維護成本與儲存。
- 避免索引失效：欄位上做運算/函式、隱式型別轉換、`LIKE '%x'` 前綴萬用字元、`OR` 連接非索引欄。

## 相關頁面
- [[MySQL索引與B+樹]] — 索引結構與最左前綴
- [[MySQL]]、[[SQL核心概念]]
- 來源：[[數據庫面試簡答、30道高頻面試題（赐我白日夢）]]
</content>
