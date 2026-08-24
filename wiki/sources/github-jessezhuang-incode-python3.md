---
title: JesseZhuang/InCodeLearning-Python3
type: source-summary
tags: [Python, 演算法, OOP設計, CodeSignal, LeetCode, 面試, 股票, 學習專案]
created: 2026-05-12
updated: 2026-05-12
sources: [github-jessezhuang-incode-python3]
---

# JesseZhuang/InCodeLearning-Python3

## Origin

- **URL**: https://github.com/JesseZhuang/InCodeLearning-Python3
- **Author**: JesseZhuang
- **Description**: Python 3 in-code learning — 結合 Python 語言基礎、演算法實作、OOP 設計題
- **Language**: Python 100%
- **Commits**: 409，**Stars**: 12
- **Date Ingested**: 2026-05-12

## Key Takeaways

- `algorithm/ood/` 資料夾包含六個 OOP 設計題 Python 實作：banking_system、in_memory_db、cloud_storage、codesignal_demo（動態中位數）、excel_sum_formula（LeetCode 631）、parking_system
- **banking_system.py** 是迄今最完整版本：同時具備 `cancel_payment`（取消排程）與 `pay_v2`（24h cashback）、transfer 轉帳（含 24h 過期）、top/top_senders 排行、merge_accounts + get_balance 歷史查詢；整合了 Coinbase HA 與 HashiCorp 版本的所有功能
- **cloud_storage.py** 是此 repo 獨有的 OOP 設計題：四層遞進，Level 3 有多用戶容量限制與帳戶合併，Level 4 有 backup/restore 快照——與 [[In_Memory_Database]] 的 backup/restore 設計思路相近
- **in_memory_db.py** 驗證了 [[In_Memory_Database]]（Coinbase HA Version B）的已知設計，增加 `compare_and_set` / `compare_and_delete` 原子操作細節
- **codesignal_demo.py** 實作動態中位數（ADD/DELETE/GET_MEDIAN），使用 SortedList + defaultdict 頻率追蹤
- **excel_sum_formula.py** 解 LeetCode 631（Hard）：兩種方案——Excel（eager 依賴 DAG 傳播）vs Excel2（lazy DFS memoization）
- `py3/` 目錄涵蓋 Python 3 標準庫約 30 個主題分區（並發、函數式、OOP、加密、網路、GUI 等）
- `stock/` 模組提供 CSV 股票資料處理工具
- 採用 PEP 8 強制規範，有 `.pep8` 設定與 `pep8_recursive.py` 驗證腳本

## Entities Mentioned

- [[JesseZhuang]] — 作者，Python/演算法學習者，持續維護 409 commits

## Concepts Mentioned

- [[Cloud_Storage_OOP設計題]] — 此 repo 獨有：四層雲端儲存 OOP 設計
- [[Banking_System]] — Coinbase 版本；此 repo 提供最完整的合併版 Python 解法
- [[HashiCorp_Banking_System_OA]] — HashiCorp 版本；此 repo 整合了兩者特性
- [[In_Memory_Database]] — Coinbase HA Version B；此 repo 增加 compare_and_set/delete 細節

## Contradictions/Tensions

- banking_system.py 同時有 cancel_payment 與 pay_v2 cashback，暗示 CodeSignal 題目版本可能持續演進，或不同批次面試有不同 Level 3 規格
- cloud_storage Level 4 的 backup/restore 與 in_memory_db Level 4 的 backup/restore 高度相似——可能是同一家公司（CodeSignal 平台）的題目設計慣例

## Questions Raised

- banking_system.py 中 transfer 有 24h 過期機制，與 [[Banking_System]] wiki 中的排程付款不同——是否為不同版本的 Level 3？
- JesseZhuang 是否同時有 Go 學習 repo？`stock/` 模組是否與本 wiki 的投資主題有交集？
