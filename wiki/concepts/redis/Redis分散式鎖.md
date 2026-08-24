---
title: Redis分散式鎖
type: concept
tags: [redis, 分散式鎖, setnx, redlock, 併發, 面試, 系統設計]
created: 2026-06-22
updated: 2026-06-22
sources: [zhuchangwu-database-interview-30-questions.md]
---

# Redis 分散式鎖

在多節點/多實例環境下保證同一資源同一時間只被一個程序操作，是 Redis 在快取之外的高頻用途。

## SETNX 基礎實作
- **SETNX（SET if Not eXists）**：只在 key 不存在時才設值，回傳 1 表示搶鎖成功。
- 必須**同時設過期時間**避免持鎖者當機造成死鎖；用單一原子命令：
  ```
  SET lock_key <unique_value> NX EX 30
  ```
- 解鎖時須**校驗 value 是唯一識別碼**（防止誤刪他人的鎖），用 Lua 腳本保證「比對 + 刪除」原子性。

## RedLock（官方多節點演算法）
針對單點 Redis 主從切換可能造成兩個 client 同時持鎖的問題，Redis 官方提出 **RedLock**：
- 向**多個獨立 Redis 節點**依序申請鎖。
- 取得**多數節點（N/2 + 1）**且總耗時小於鎖有效期，才算加鎖成功。
- 特性：互斥、靠過期避免死鎖、少數節點故障仍可運作（容錯）。

## 注意事項
- 鎖過期時間需大於業務執行時間，或用**看門狗（watchdog）自動續期**（如 Redisson 實作）。
- RedLock 有爭議：Martin Kleppmann 指出在 GC 暫停、時鐘漂移下仍可能失效，強一致場景建議用 ZooKeeper/etcd。

## 相關頁面
- [[Redis核心機制]]、[[Redis快取三大問題]]（擊穿解法之一即互斥鎖）
- [[Redis]]
- [[分散式任務排程系統]] — 也依賴分散式鎖（ZooKeeper）
- 來源：[[數據庫面試簡答、30道高頻面試題（赐我白日夢）]]
</content>
