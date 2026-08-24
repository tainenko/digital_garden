---
title: Banking System（Coinbase CodeSignal HA - Version A）
type: concept
tags: [CodeSignal, Coinbase, HA, Golang, 面試, OA, HashMap, BinarySearch]
created: 2026-05-04
updated: 2026-05-04
sources: [coinbase-codesignal-ha-guide]
---

# Banking System

## 題目說明

實作一個銀行系統，timestamp 嚴格遞增，**後來的操作必須先處理所有到期的排程付款**。

---

## Level 1 — 帳戶基本操作

```
create_account(timestamp, account_id) -> bool
  建立帳戶；已存在回傳 false

deposit(timestamp, account_id, amount) -> int | ""
  存款；帳戶不存在回傳 ""；否則回傳新餘額

transfer(timestamp, from_id, to_id, amount) -> int | ""
  轉帳；帳戶不存在/余額不足/相同帳戶回傳 ""；否則回傳 from 的新餘額
```

## Level 2 — 消費排行

```
top_spenders(timestamp, n) -> []string
  回傳消費總額最高的 n 位，格式 "id(amount)"
  同消費額按帳戶名稱升序排列
```

## Level 3 — 排程付款

```
schedule_payment(timestamp, account_id, amount, delay) -> payment_id | ""
  在 timestamp+delay 時從帳戶扣除 amount；帳戶不存在回傳 ""
  回傳唯一 payment_id（如 "payment1"）

cancel_payment(timestamp, account_id, payment_id) -> bool
  取消排程付款；payment_id 不存在/已執行/帳戶不符回傳 false
```

## Level 4 — 歷史餘額 + 帳戶合併

```
get_balance(timestamp, account_id, time_at) -> int | ""
  查詢帳戶在 time_at 時點的歷史餘額
  time_at 必須 <= timestamp；帳戶不存在或尚未建立回傳 ""

merge_accounts(timestamp, account_id_1, account_id_2) -> bool
  將 account_id_2 合併進 account_id_1
  合併後 acc2 消失；acc1 繼承 acc2 的餘額、消費紀錄、歷史、排程付款
  兩帳戶相同或任一不存在回傳 false
```

---

## 解題思路

- **Account struct**：保存 balance、spending（累計轉出）、history（時間戳→餘額的有序切片）
- **排程付款**：用 slice 維護未執行的付款，每次操作開頭先處理所有 dueAt <= timestamp 的付款，按 dueAt 升序、同時間按建立順序處理
- **get_balance**：在 history 上做 binary search，找最後一個 timestamp <= time_at 的紀錄
- **merge_accounts**：Sorted merge 兩個 history 切片；把 acc2 的排程付款轉移給 acc1

### 複雜度

| 操作 | 時間複雜度 | 說明 |
|------|-----------|------|
| create / deposit / transfer | O(P log P) | P = 到期付款數 |
| top_spenders | O(N log N) | N = 帳戶數 |
| get_balance | O(log H) | H = 歷史紀錄數 |
| merge_accounts | O(H1+H2) | Sorted merge |

---

## Go 完整解法

```go
package main

import (
	"fmt"
	"sort"
)

// ── 資料結構 ──────────────────────────────────────────────

type historyEntry struct {
	ts      int
	balance int
}

type account struct {
	id       string
	balance  int
	spending int
	history  []historyEntry // 嚴格遞增 ts
}

type payment struct {
	id        string
	accountID string
	amount    int
	dueAt     int
	cancelled bool
}

type BankingSystem struct {
	accounts map[string]*account
	payments map[string]*payment
	pending  []*payment // 未執行的排程，按 dueAt 升序
	counter  int
}

func NewBankingSystem() *BankingSystem {
	return &BankingSystem{
		accounts: make(map[string]*account),
		payments: make(map[string]*payment),
	}
}

// ── 內部工具 ──────────────────────────────────────────────

// recordHistory 在帳戶歷史末尾追加紀錄（ts 嚴格遞增保證）
func (b *BankingSystem) recordHistory(a *account, ts int) {
	// 若同一 ts 已有紀錄則更新最後一筆（同 ts 多次操作取最終值）
	if len(a.history) > 0 && a.history[len(a.history)-1].ts == ts {
		a.history[len(a.history)-1].balance = a.balance
	} else {
		a.history = append(a.history, historyEntry{ts, a.balance})
	}
}

// processPayments 處理所有 dueAt <= ts 的排程付款
func (b *BankingSystem) processPayments(ts int) {
	remaining := b.pending[:0]
	// pending 已按 dueAt 排序，找到第一個 dueAt > ts 的分界
	for _, p := range b.pending {
		if p.cancelled {
			continue
		}
		if p.dueAt > ts {
			remaining = append(remaining, p)
			continue
		}
		// 執行付款
		a, ok := b.accounts[p.accountID]
		if ok && a.balance >= p.amount {
			a.balance -= p.amount
			a.spending += p.amount
			b.recordHistory(a, p.dueAt)
		}
		// 餘額不足時靜默跳過（按題意）
	}
	b.pending = remaining
}

// balanceAt 在 history 中 binary search 找 timeAt 時的餘額
func balanceAt(history []historyEntry, timeAt int) (int, bool) {
	lo, hi, idx := 0, len(history)-1, -1
	for lo <= hi {
		mid := (lo + hi) / 2
		if history[mid].ts <= timeAt {
			idx = mid
			lo = mid + 1
		} else {
			hi = mid - 1
		}
	}
	if idx == -1 {
		return 0, false
	}
	return history[idx].balance, true
}

// ── Level 1 ───────────────────────────────────────────────

func (b *BankingSystem) CreateAccount(ts int, id string) bool {
	b.processPayments(ts)
	if _, exists := b.accounts[id]; exists {
		return false
	}
	a := &account{id: id, history: []historyEntry{{ts, 0}}}
	b.accounts[id] = a
	return true
}

func (b *BankingSystem) Deposit(ts int, id string, amount int) (int, bool) {
	b.processPayments(ts)
	a, ok := b.accounts[id]
	if !ok {
		return 0, false
	}
	a.balance += amount
	b.recordHistory(a, ts)
	return a.balance, true
}

func (b *BankingSystem) Transfer(ts int, fromID, toID string, amount int) (int, bool) {
	b.processPayments(ts)
	from, fromOK := b.accounts[fromID]
	to, toOK := b.accounts[toID]
	if !fromOK || !toOK || fromID == toID || from.balance < amount {
		return 0, false
	}
	from.balance -= amount
	from.spending += amount
	to.balance += amount
	b.recordHistory(from, ts)
	b.recordHistory(to, ts)
	return from.balance, true
}

// ── Level 2 ───────────────────────────────────────────────

func (b *BankingSystem) TopSpenders(ts int, n int) []string {
	b.processPayments(ts)
	type entry struct {
		id       string
		spending int
	}
	list := make([]entry, 0, len(b.accounts))
	for id, a := range b.accounts {
		list = append(list, entry{id, a.spending})
	}
	sort.Slice(list, func(i, j int) bool {
		if list[i].spending != list[j].spending {
			return list[i].spending > list[j].spending
		}
		return list[i].id < list[j].id
	})
	if n > len(list) {
		n = len(list)
	}
	result := make([]string, n)
	for i := 0; i < n; i++ {
		result[i] = fmt.Sprintf("%s(%d)", list[i].id, list[i].spending)
	}
	return result
}

// ── Level 3 ───────────────────────────────────────────────

func (b *BankingSystem) SchedulePayment(ts int, id string, amount, delay int) (string, bool) {
	b.processPayments(ts)
	if _, ok := b.accounts[id]; !ok {
		return "", false
	}
	b.counter++
	pid := fmt.Sprintf("payment%d", b.counter)
	p := &payment{
		id:        pid,
		accountID: id,
		amount:    amount,
		dueAt:     ts + delay,
	}
	b.payments[pid] = p
	// 插入排序位置維持 pending 按 dueAt 有序
	idx := sort.Search(len(b.pending), func(i int) bool {
		return b.pending[i].dueAt > p.dueAt
	})
	b.pending = append(b.pending, nil)
	copy(b.pending[idx+1:], b.pending[idx:])
	b.pending[idx] = p
	return pid, true
}

func (b *BankingSystem) CancelPayment(ts int, id, pid string) bool {
	b.processPayments(ts)
	p, ok := b.payments[pid]
	if !ok || p.accountID != id || p.cancelled || p.dueAt <= ts {
		return false
	}
	p.cancelled = true
	return true
}

// ── Level 4 ───────────────────────────────────────────────

func (b *BankingSystem) GetBalance(ts int, id string, timeAt int) (int, bool) {
	b.processPayments(ts)
	a, ok := b.accounts[id]
	if !ok {
		return 0, false
	}
	return balanceAt(a.history, timeAt)
}

func (b *BankingSystem) MergeAccounts(ts int, id1, id2 string) bool {
	b.processPayments(ts)
	a1, ok1 := b.accounts[id1]
	a2, ok2 := b.accounts[id2]
	if !ok1 || !ok2 || id1 == id2 {
		return false
	}
	// 合併餘額與消費
	a1.balance += a2.balance
	a1.spending += a2.spending
	// Sorted merge history
	merged := make([]historyEntry, 0, len(a1.history)+len(a2.history))
	i, j := 0, 0
	for i < len(a1.history) && j < len(a2.history) {
		if a1.history[i].ts <= a2.history[j].ts {
			merged = append(merged, a1.history[i])
			i++
		} else {
			merged = append(merged, a2.history[j])
			j++
		}
	}
	merged = append(merged, a1.history[i:]...)
	merged = append(merged, a2.history[j:]...)
	a1.history = merged
	// 轉移 pending 付款
	for _, p := range b.pending {
		if p.accountID == id2 {
			p.accountID = id1
		}
	}
	b.recordHistory(a1, ts)
	delete(b.accounts, id2)
	return true
}
```

---

## 相關頁面

- [[Coinbase_HA_總覽]] — 題目版本、分數結構、應試策略
- [[In_Memory_Database]] — Version B 題目 + Go 解法
- [[HashiCorp_Banking_System_OA]] — HashiCorp 版本：Level 2 多 TOP_ACTIVITY、Level 3 有 Cashback 機制，無 cancel_payment
- [[JesseZhuang]] — Python 最完整合併版：同時有 cancel_payment + pay_v2 cashback + transfer（24h 過期）+ top/top_senders
