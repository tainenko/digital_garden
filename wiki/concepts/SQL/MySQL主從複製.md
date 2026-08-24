---
title: MySQL主從複製
type: concept
tags: [mysql, 複製, replication, binlog, gtid, 高可用, 面試]
created: 2026-06-22
updated: 2026-06-22
sources: [zhuchangwu-database-interview-30-questions.md]
---

# MySQL 主從複製（Replication）

主從複製用於讀寫分離、高可用與備援，核心是把主庫的 **binlog** 傳到從庫重放。

## binlog 複製機制
1. 主庫將寫操作記錄到 **binlog**。
2. 主庫建立專用複製帳號授權給從庫。
3. 從庫的 **I/O thread** 從指定 binlog 檔案與 position 拉取 binlog，寫入本地 **relay log**。
4. 從庫的 **SQL thread** 重放 relay log，套用變更。

## 三種複製模式（一致性 vs 效能取捨）
- **異步複製（Asynchronous）**：主庫寫 redo log、binlog 後**立即提交**，不等從庫確認。最快，但主庫當機時尚未同步的資料會遺失。
- **半同步複製（Semi-synchronous）**：主庫交易處於 prepare 狀態並寫完 binlog 後，**等待至少一個從庫回傳 ack** 才提交。一致性與效能折衷（最常用的高可用方案）。
- **全同步複製（Fully synchronous）**：主庫需等**所有**從庫完成複製才提交。一致性最強，但效能下降顯著。

## GTID 複製
**GTID（Global Transaction ID，全域交易識別碼）** 為每筆交易賦予全域唯一 ID，從庫依 GTID 自動追蹤同步進度，**免去手動指定 binlog 檔名與 offset**，使故障切換（failover）與重新指向主庫更簡單可靠。

## 相關頁面
- [[MySQL]]、[[MySQL事務與隔離級別]] — binlog 與交易提交時點相關
- [[MySQL慢查詢與執行計畫]]
- 來源：[[數據庫面試簡答、30道高頻面試題（赐我白日夢）]]
</content>
