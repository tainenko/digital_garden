---
title: Coinbase CodeSignal HA 總覽
type: concept
tags: [CodeSignal, Coinbase, HA, 面試, OA, 刷題]
created: 2026-05-04
updated: 2026-05-04
sources: [coinbase-codesignal-ha-guide, lodely-coinbase-oa-2025]
---

# Coinbase CodeSignal HA 總覽

## 基本資訊

| 項目 | 內容 |
|------|------|
| 題數 | **1 道大題，分 4 關（Level 1–4）** |
| 總時限 | **90 分鐘**（需一次完成） |
| 計分 | 每關 250 分，滿分 1000 → 換算至 600 分制 |
| 通過門檻 | 約 500+/1000（換算 ≈ 300+/600） |
| 監控 | 需要照片驗證 + 螢幕錄影；禁止切換頁籤 |

---

## 題目池（兩種版本輪替）

| 版本 | 題目 | 核心技術 |
|------|------|---------|
| A | [[Banking_System]] | HashMap、排序、Binary Search、歷史快照 |
| B | [[In_Memory_Database]] | HashMap、前綴搜尋、TTL 計時、備份還原 |

不同批次的應試者收到不同版本，兩版都需熟悉。

---

## 各 Level 難度與重點

### Version A：Banking System

| Level | 功能 | 難度 | 關鍵點 |
|-------|------|------|--------|
| 1 | create_account / deposit / transfer | ⭐ 易 | HashMap 基本操作 |
| 2 | top_spenders | ⭐⭐ 中 | 排序（支出降序 + 名稱升序） |
| 3 | schedule_payment / cancel_payment | ⭐⭐⭐ 中高 | 延遲執行、時間觸發 |
| 4 | get_balance / merge_accounts | ⭐⭐⭐⭐ 難 | Binary Search 歷史記錄、帳戶合併 |

### Version B：In-Memory Database

| Level | 功能 | 難度 | 關鍵點 |
|-------|------|------|--------|
| 1 | set / get / delete | ⭐ 易 | 巢狀 HashMap |
| 2 | scan / scan_by_prefix | ⭐⭐ 中 | 排序輸出、前綴過濾 |
| 3 | TTL 支援（set_at_with_ttl 系列） | ⭐⭐⭐ 中高 | 過期時間計算 |
| 4 | backup / restore | ⭐⭐⭐⭐ 難 | Deep Copy、TTL 偏移計算 |

---

## 監考機制（2026）

| 項目 | 說明 |
|------|------|
| 攝影機 | 強制開啟，全程錄影 |
| 螢幕錄影 | 全程錄製 |
| 監控行為 | Alt-Tab 切換、大量複製貼上、打字速度異常 |
| 分數快取 | 分數存入系統，Hiring Manager 可查歷史 |
| 完成時窗 | 收到後 7 天內完成 |

---

## OA DSA 題型清單（候選人回報匯整）

### 經典演算法題

| 題型 | 分類 | LeetCode 對應 |
|------|------|--------------|
| Most Profitable Window in Price Series | Sliding Window | #121 |
| Balanced Parentheses with Wildcards | Stack/String | #678 |
| Modular Exponentiation with Large Exponent | Math/Number Theory | #50 |
| Graph Shortest Path with Edge Failures | Graph/Dijkstra | #743 |
| Merge K Sorted Arrays | Divide & Conquer/Heap | #23 |
| Detect Cycle in Directed Graph | Graph/DFS | #207 |
| Longest Substring with At Most K Distinct | Two Pointers/Sliding Window | #340 |
| Buy/Sell Stock with K Transactions | DP | #188 |
| Iterator Pattern & Skip Iterator | Design Pattern | — |
| Median of Two Sorted Arrays | Binary Search | #4 |
| Validate Bracket Sequence | Stack | #20 |
| Longest Palindromic Substring | DP | #5 |
| Power of Large Numbers | Math/Modular Arithmetic | #50 |
| Sherlock and Permutations | Combinatorics | HackerRank |

### Fintech 特定題型（2026 重點）

| 題型 | 說明 |
|------|------|
| In-Memory Banking System | HashMap 帳戶狀態 + Priority Queue；對應 [[Banking_System]] |
| Task Management System with TTL | Task 在 N 秒後過期；對應 [[In_Memory_Database]] |
| Transaction Ledger Reconciliation | 找兩份亂序 ledger 的差異；Hashing + Sorting |

---

## 應試策略

1. **通關優先**：Level 1–2 確保 AC，Level 3 部分通過也有分，Level 4 難度最高不必強求
2. **測試案例節省**：每關有 40 個 test case，建議先用少數確認邏輯，不要一開始全跑
3. **Boilerplate 熟記**：HashMap 操作、排序 lambda、Binary Search 模板提前背好
4. **Level 不可退**：答完 Level N 才能看 Level N+1，且之前的測試必須持續通過（不可破壞舊邏輯）

---

## 相關頁面

- [[Coinbase_面試全流程]] — 完整 5 階段流程（Recruiter→OA→Phone→Onsite→HM）、各輪準備重點
- [[Banking_System]] — Version A 完整題目 + Go 解法
- [[In_Memory_Database]] — Version B 完整題目 + Go 解法
