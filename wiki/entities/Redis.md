---
title: Redis
type: entity
tags: [快取, nosql, redis, 記憶體資料庫, 後端]
created: 2026-06-22
updated: 2026-06-22
sources: [zhuchangwu-database-interview-30-questions.md]
---

# Redis

開源的記憶體型 key-value 資料庫，最廣泛使用的分散式快取與資料結構伺服器（REmote DIctionary Server）。作者 Salvatore Sanfilippo（antirez）。

- **特性**：基於記憶體、命令執行單執行緒、epoll I/O 多路復用 → 見 [[Redis核心機制]]。
- **五大資料結構**：String / Hash / List / Set / Sorted Set，對應快取、物件儲存、佇列、去重、排行榜等場景。
- **持久化**：RDB 快照 + AOF 命令日誌。
- **典型用途**：快取層（緩解 MySQL 壓力）、分散式鎖、排行榜、計數器、限流、Session 儲存、訊息佇列。
- **常見坑**：快取雪崩 / 穿透 / 擊穿 → 見 [[Redis快取三大問題]]。

## 相關頁面
- [[Redis核心機制]]、[[Redis快取三大問題]]、[[Redis分散式鎖]]
- [[MySQL]] — Redis 常作為其前置快取
- [[系統設計核心技術棧]]
- 來源：[[數據庫面試簡答、30道高頻面試題（赐我白日夢）]]
</content>
