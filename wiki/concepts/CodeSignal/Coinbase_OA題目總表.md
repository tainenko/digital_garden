---
title: Coinbase OA 題目總表
type: concept
tags: [CodeSignal, Coinbase, OA, 面試, 刷題, LLD]
created: 2026-05-04
updated: 2026-05-04
sources: [lodely-coinbase-oa-2025, techprep-coinbase-interview-process-2026]
---

# Coinbase OA 題目總表

來源：interviewdb.io、prachub.com、hack2hire.com、GitHub 候選人分享

---

## OA 格式快速回顧

| 項目 | 內容 |
|------|------|
| 平台 | CodeSignal（2026 起，取代 HackerRank/Codility） |
| 時限 | 90 分鐘 |
| 題數 | 2–3 題（或 1 道 4-Level progressive 大題） |
| 監考 | 攝影機 + 螢幕錄影；偵測 alt-tab / 大量複製貼上 |
| 通過門檻 | ~500+/800 |
| 核心主題 | 金融交易系統、HashMap、事件流、圖、DP |

---

## 題目索引

| # | 題目 | 類型 | 難度 | 資料完整度 | 詳細頁面 |
|---|------|------|------|-----------|---------|
| 1 | [[Max_Fee]] | Coding OA | Hard | ★★★★★ | [[Max_Fee]] |
| 2 | [[NFT_Generator]] | Coding OA | Hard | ★★★★★ | [[NFT_Generator]] |
| 3 | [[Order_Management]] | Coding OA | Med-Hard | ★★★★★ | [[Order_Management]] |
| 4 | Banking System | Coding OA | Med-Hard | ★★★★★ | [[Banking_System]] |
| 5 | In-Memory Database | Coding OA | Med-Hard | ★★★★★ | [[In_Memory_Database]] |
| 6 | Garbled Logs | Coding Onsite | Medium | ★★★☆☆ | 見下方 |
| 7 | Exchange | Coding Onsite | Medium | ★★★☆☆ | 見下方 |
| 8 | Price Stream | Coding OA | Medium | ★★★☆☆ | 見下方 |
| 9 | Transaction Filter | Coding Screening | Medium | ★★☆☆☆ | 見下方 |
| 10 | Order Stream | Coding Onsite | Hard | ★★☆☆☆ | 見下方（Order Management 延伸）|
| 11 | Tabemono（食べ物） | Coding Onsite | Easy | ★☆☆☆☆ | 見下方 |
| 12 | Drone | Coding Onsite | Unknown | ★☆☆☆☆ | 見下方 |
| 13 | Blockchain Indexer | Coding Onsite | Unknown | ★☆☆☆☆ | 見下方 |
| 14 | Frontend OA | Frontend OA | Medium | ★★★☆☆ | 見下方 |

---

## 各題摘要

### 1. Max Fee（Maximize Block Fee）
→ 完整頁面：[[Max_Fee]]

Block 容量 100，選 transaction 子集最大化 fee。
- Level 1：Greedy（fee/size density 排序）
- Level 2：加入 parentId 依賴鏈（child 需要 parent 才能入選）

---

### 2. NFT Generator（Generate NFT Metadata and Ensure Uniqueness）
→ 完整頁面：[[NFT_Generator]]

從多個 trait 類別的 Cartesian product 中，用 seed 確定性地生成 n 個唯一 NFT。
- Level 1：枚舉組合
- Level 2：處理 category 內重複值
- Level 3：weighted random + 避免 retry
- 進階：n > total combinations → 回傳 `ERROR`

---

### 3. Order Management（Crypto Order Management System）
→ 完整頁面：[[Order_Management]]

in-memory order state machine，O(1) 操作。
- Level 1：CREATE/PAUSE/RESUME/CANCEL/GET/COUNT
- Level 2：user_id 索引 + cancel_all_by_user
- Level 3：按 hash(user_id) % N 分 shard

---

### 4 & 5. Banking System / In-Memory Database
→ [[Banking_System]] / [[In_Memory_Database]]

CodeSignal HA 4-Level progressive 題（詳見各頁面）

---

### 6. Garbled Logs（Thread Logs Processing）

**問題**：日誌行格式 `<timestamp> thread=<id> msg=<message>`，多執行緒日誌交錯輸入，需整理還原每個執行緒的時間序日誌。

**核心操作**：
- Parse 每行：提取 (timestamp, threadId, message)
- 按 threadId 分組
- 每組內按 timestamp 升序排列
- 回傳 `Map<threadId, List<LogEntry>>`

**關鍵考點**：字串解析、HashMap + sorting、處理相同 timestamp 的邊界條件。

**複雜度**：O(N log N)，N = 日誌行數。

---

### 7. Exchange（Order Book / Currency Exchange）

兩種版本流傳，面試時可能拿到任一種：

**Version A：Currency Exchange（Graph）**
- Input：幣種對與匯率，如 `USD→BTC = 0.000025`
- Find：從 A 到 B 的最高轉換率（任意路徑）
- 解法：Directed weighted graph + DFS，乘積最大化（log space + 最短路徑）

**Version B：Order Book Matching**
- Implement order book：buy 以最低賣價成交，sell 以最高買價成交
- FIFO 價格時間優先
- Level 1：靜態清單配對
- Level 2：即時事件流、部分成交

**關鍵考點**：Graph traversal（Version A），Priority Queue / Heap（Version B）

---

### 8. Price Stream（Most Profitable Window in Price Series）

**問題**：給定價格序列，找最大買賣利潤。

**Level 1**：`max(prices[j] - prices[i])` where j > i — O(N)，維護 running minimum。

**Level 2（推測）**：允許多次交易，或找特定視窗大小 W 的最大差值。

**Level 3（推測）**：Streaming 版本，即時處理新價格，維護 running answer。

**LeetCode 對應**：#121（基礎）、#122（多次交易）、#239（Sliding Window Maximum）

---

### 9. Transaction Filter（Generic Search / Iterator with Filter）

**問題**：設計 API 對 transaction list 做多條件篩選。

**方向 A：Search API**
- Transactions 有 `id`, `amount`, `currency`, `timestamp`, `status` 等欄位
- 支援 `=`, `>`, `<` 運算子，多條件 AND

**方向 B：Iterator Pattern**
- 包裹 iterator，lazily 套用 predicate
- 類似 LeetCode "Skip Iterator"

**Level 推測**：
- Level 1：單條件 `=`
- Level 2：多條件 AND
- Level 3：OR + 混合運算子
- Level 4：Streaming / Lazy evaluation / Pagination

---

### 10. Order Stream

Order Management 的延伸版，加入 streaming 處理與聚合統計。

**推測內容**：
- 持續接收 order events（place/cancel/fill）
- 統計特定時間視窗內的 volume、VWAP（Volume Weighted Average Price）
- Level 3/4：可能含亂序 event 處理、滑動時間窗口

→ 核心資料結構同 [[Order_Management]]，加上時間序聚合邏輯。

---

### 11. Tabemono（食べ物 / Food Delivery System）

**內部題名**：Tabemono（日文「食物」）。Hack2hire 標示為 Easy Onsite。

**最近似公開題**：LeetCode #1418 "Display Table of Food Orders in a Restaurant"。

**推測問題**：
- 給定訂單清單（顧客、桌號、餐點）
- 輸出各桌各餐點的數量表格
- 可能延伸：外送路由、菜單管理、訂單狀態追蹤

---

### 12. Drone

公開資料極少，interviewdb.io / hack2hire 均確認存在但內容付費鎖定。

**常見 "Drone" 面試題模式**：
- 格狀地圖中，drone 需要送達所有目標位置
- 最小化總路徑距離 / drone 數量
- 解法方向：BFS、MST、TSP 變形

---

### 13. Blockchain Indexer

公開資料極少，interviewdb.io 確認存在（CodingOnsite）。

**推測 Level 結構**（依 Coinbase 工程部落格的實際 indexer 需求）：
- Level 1：建立 address → transactions 的索引；查詢某地址的所有交易
- Level 2：查詢某地址在指定 block height 的餘額
- Level 3：支援 block reorganization（reorg）——blocks 可被撤回並替換，索引要能更新
- Level 4：區間查詢（block X 到 Y 的交易）、聚合統計（地址總轉入/轉出量）

**核心考點**：HashMap、時間序歷史快照（類似 [[Banking_System]] 的 get_balance），可能加 trie。

---

### 14. Frontend OA（Task Management System）

**平台**：CodeSignal，90 分鐘，**需要 React**。

**Level 1**：從 `.json` 檔讀取並顯示 task backlog 清單。
**Level 2**：透過表單新增 task。
**Level 3**：從 API 取得 task ID 清單，再逐一 fetch task 詳情（兩層 API call）。
**Level 4**：支援 task 優先級的新增與移除。

**類似題**：GreatFrontend "Job Board" 實作題。

---

## 核心演算法速查

| 演算法 | 對應題目 |
|--------|---------|
| HashMap + 狀態機 | Order Management、Banking System |
| Greedy（fee density 排序） | Max Fee Level 1 |
| Tree DP / DAG 依賴 | Max Fee Level 2 |
| Mixed-Radix Decoding | NFT Generator |
| Binary Search on History | Banking System Level 4 |
| Sorted Merge | Banking System MergeAccounts |
| Deep Copy + TTL 偏移 | In-Memory Database Level 4 |
| Sliding Window | Price Stream、Garbled Logs |
| Graph DFS / Dijkstra | Exchange Version A |
| Priority Queue | Exchange Version B（Order Book） |

---

## 相關頁面

- [[Coinbase_HA_總覽]] — 4-Level progressive 大題格式（Banking System / In-Memory Database）
- [[Coinbase_面試全流程]] — 完整 5 階段面試結構
- [[Max_Fee]] — 完整題目 + Go 解法
- [[NFT_Generator]] — 完整題目 + Go 解法
- [[Order_Management]] — 完整題目 + Go 解法
