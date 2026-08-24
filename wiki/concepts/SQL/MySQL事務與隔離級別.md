---
title: MySQL事務與隔離級別
type: concept
tags: [mysql, 事務, 隔離級別, mvcc, acid, 面試]
created: 2026-06-22
updated: 2026-06-22
sources: [zhuchangwu-database-interview-30-questions.md]
---

# MySQL 事務與隔離級別

## ACID 四特性
- **Atomicity 原子性**：交易內操作全成功或全回滾（由 undo log 實現）。
- **Consistency 一致性**：交易前後資料庫保持合法狀態（最終目的）。
- **Isolation 隔離性**：併發交易彼此不互相干擾（由鎖 + MVCC 實現）。
- **Durability 持久性**：提交後永久生效（由 redo log 實現）。

## 三種讀異常
- **髒讀（Dirty Read）**：讀到另一交易**尚未提交**的修改。
- **不可重複讀（Non-repeatable Read）**：同一交易內兩次讀**同一列**結果不同（被別人 UPDATE/DELETE 後提交）。
- **幻讀（Phantom Read）**：同一交易內兩次**範圍查詢**返回的**列數**不同（被別人 INSERT 後提交）。

## 四種隔離級別
| 級別 | 髒讀 | 不可重複讀 | 幻讀 | InnoDB 預設 |
|------|------|-----------|------|------------|
| Read Uncommitted | ✗ 允許 | ✗ | ✗ | |
| Read Committed | ✓ 防止 | ✗ | ✗ | |
| **Repeatable Read** | ✓ | ✓ 防止 | ✓*（見下）| **✓ 預設** |
| Serializable | ✓ | ✓ | ✓ 防止 | |

> 級別越高隔離性越強、併發效能越低。

## InnoDB 的關鍵點：RR 級別就解決幻讀
標準 SQL 中 Repeatable Read 仍允許幻讀，需 Serializable 才能避免。但 **InnoDB 在 RR 級別透過 MVCC + Gap Lock（間隙鎖）已解決幻讀**：
- **MVCC（多版本併發控制）**：透過 undo log 版本鏈 + Read View，快照讀（一般 `SELECT`）讀到一致性快照，不加鎖即避免大部分幻讀。
- **Gap Lock / Next-Key Lock**：當前讀（`SELECT ... FOR UPDATE`、`UPDATE`、`DELETE`）鎖住索引區間，阻止其他交易在間隙 INSERT，封堵幻讀。

## 相關頁面
- [[MySQL索引與B+樹]] — 行鎖依附於索引；無索引會退化為表鎖
- [[資料庫正規化]]
- [[SQL核心概念]]
- 來源：[[數據庫面試簡答、30道高頻面試題（赐我白日夢）]]
</content>
