---
title: HashiCorp Banking System OA（OOP設計題）
type: concept
tags: [HashiCorp, OA, OOP, Banking, Go, Python, 面試, LLD]
created: 2026-05-12
updated: 2026-05-12
sources: [github-karuppiah7890-interview-questions]
---

# HashiCorp Banking System OA（OOP 設計題）

## 題目背景

來源：HashiCorp 工程師預評測（pre-assessment）
重點：OOP 設計——structures、classes、methods 等
解題語言：Go（thread-safe mutex 版本）、Python（sortedcontainers 版本）

> 與 [[Banking_System]]（Coinbase HA 版本）為高度重疊的同類題型，但細節規格不同。

---

## 四層遞進結構

### Level 1 — 帳戶基本操作

```
create_account(account_id) -> bool
deposit(account_id, amount) -> int
withdraw(account_id, amount) -> int | error
transfer(from_id, to_id, amount) -> int | error
```

### Level 2 — 消費排行榜

```
TOP_ACTIVITY  — 按交易總量（存 + 提 + 轉）降序排列
TOP_SPENDERS  — 按轉出總額降序排列，同額按名稱升序
```

### Level 3 — 排程轉帳 + Cashback

- **Scheduled Transfer**：指定未來時間點執行轉帳，逾 24 小時自動失效
- **Cashback**：每次提款金額的 **2%** 在 24 小時後自動退回帳戶
- 每次操作前先處理所有到期的排程事件（同 Coinbase 版本）

### Level 4 — 帳戶合併 + 歷史餘額

```
merge_accounts(id1, id2) -> bool
  id2 合併入 id1；繼承餘額、消費紀錄、交易歷史、排程事件

get_balance(account_id, time_at) -> int | error
  查詢帳戶在 time_at 時點的歷史餘額（Binary Search on history log）
```

---

## 實作重點

### Go 版本（Thread-Safe）

- `sync.Mutex` 保護所有帳戶與排程資料
- `map[string]*Account` 主資料結構
- `[]ScheduledPayment` 按到期時間排序，每次操作開頭先 flush 到期項目
- 自訂 sort interface 實作 `TOP_SPENDERS` 排序

### Python 版本

- `SortedList` / `SortedDict`（from `sortedcontainers`）處理時序查詢
- `Account` class 含 balance、history log、transaction count
- `Bank` class 統一管理帳戶與排程
- 處理 cashback 時用 binary search 找時間點

---

## 與 Coinbase HA 版本的差異

| 面向 | HashiCorp 版本 | Coinbase HA 版本 |
|------|---------------|-----------------|
| Level 2 排行 | TOP_ACTIVITY + TOP_SPENDERS | top_spenders only |
| Level 3 功能 | Scheduled transfer（24h 到期）+ Cashback 2% | schedule_payment + cancel_payment |
| Cashback | 有（提款 2%，24h 後退回） | 無 |
| Cancel 付款 | 無 cancel API | 有 cancel_payment |
| 語言 | Go + Python | Go |

---

## 相關頁面

- [[Banking_System]] — Coinbase HA 版本，結構相似但 API 有差異
- [[Coinbase_HA_總覽]] — Coinbase HA 整體面試格式
- [[Low Level Design OOD設計題型]] — OOP/LLD 設計題通用框架
- [[karuppiah7890]] — 此題解法的來源作者
