
## [2026-06-22] expand | 數據庫面試補充：MySQL 複製/調優 + Redis 分散式鎖 + 實體頁

- 承上則 ingest，補抓原文未收錄章節並補實體頁
- Created（2 實體）: [[MySQL]]、[[Redis]]
- Created（3 概念）: [[MySQL主從複製]]、[[MySQL慢查詢與執行計畫]]（→ concepts/SQL/）；[[Redis分散式鎖]]（→ concepts/redis/）
- Updated: [[數據庫面試簡答、30道高頻面試題（赐我白日夢）]]（concepts mentioned +3、questions raised 結案）；[[Redis快取三大問題]]、[[Redis核心機制]]（+回鏈分散式鎖/實體頁）；wiki/index.md（+2 entities、+2 SQL 概念、+1 Redis 概念，頁數 456→461）
- Key additions: (1) 三種複製模式取捨——異步（快、可能丟資料）/半同步（等 ≥1 從庫 ack，最常用）/全同步（等所有從庫，慢）；GTID 免手動指定 binlog 檔名+offset；(2) 調優流程——long_query_time + mysqldumpslow 找慢查詢；EXPLAIN 看 type(const>ref>range>index>ALL)/key/rows/Extra(Using filesort 為優化點)；索引失效情境（函式運算/隱式轉換/前綴 %/OR）；(3) 分散式鎖——SET NX EX 原子加鎖 + Lua 校驗 value 解鎖；RedLock 多數決(N/2+1)；看門狗續期；Kleppmann 對 RedLock 在 GC/時鐘漂移下的批評。
