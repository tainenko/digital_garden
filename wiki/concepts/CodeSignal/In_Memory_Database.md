---
title: In-Memory Database（Coinbase CodeSignal HA - Version B）
type: concept
tags: [CodeSignal, Coinbase, HA, Golang, 面試, OA, HashMap, TTL, Snapshot]
created: 2026-05-04
updated: 2026-05-04
sources: [coinbase-codesignal-ha-guide]
---

# In-Memory Database

## 題目說明

實作一個鍵值資料庫，支援 nested key-field 結構、TTL 過期、備份與還原。

---

## Level 1 — 基本 CRUD

```
set(key, field, value) -> null
  設置 key 下的 field = value

get(key, field) -> string | ""
  取得值；key/field 不存在回傳 ""

delete(key, field) -> bool
  刪除 field；存在回傳 true，否則 false
```

## Level 2 — 掃描

```
scan(key) -> []string
  回傳 key 下所有 field 的 "field(value)"，按 field 字母升序排列
  key 不存在或無 field 回傳 []

scan_by_prefix(key, prefix) -> []string
  與 scan 相同，但只回傳 field 開頭為 prefix 的項目
```

## Level 3 — TTL 支援

所有操作加入 timestamp 參數；呼叫時自動過期 TTL 到期的 field。

```
set_at(timestamp, key, field, value) -> null
  同 set，但記錄操作時間

set_at_with_ttl(timestamp, key, field, value, ttl) -> null
  在 timestamp + ttl 時點讓 field 過期（之後 get/scan 不可見）

delete_at(timestamp, key, field) -> bool
get_at(timestamp, key, field) -> string | ""
scan_at(timestamp, key) -> []string
scan_by_prefix_at(timestamp, key, prefix) -> []string
```

## Level 4 — 備份與還原

```
backup(timestamp) -> int
  備份當前所有資料（Deep Copy）；回傳備份中存活的 field 總數

restore(timestamp, timestamp_to_restore) -> null
  還原到指定備份時間點的狀態
  TTL 重新計算：新到期時間 = timestamp + (原到期時間 - timestamp_to_restore)
  即：剩餘存活時間從 restore 時間點重新開始計算
```

---

## 解題思路

- **資料結構**：`map[string]map[string]*fieldValue`，fieldValue 含 value、expiresAt、hasExpiry
- **TTL 過期**：讀取時檢查 `hasExpiry && timestamp >= expiresAt`，惰性清理（lazy deletion）
- **scan 排序**：收集所有未過期 field，sort.Strings，再格式化輸出
- **backup**：Deep copy 整個 records map，連同所有 fieldValue 物件一起複製
- **restore**：還原 Deep copy，並對每個有 TTL 的 field 重新計算 expiresAt：
  `newExpiresAt = restoreTimestamp + (originalExpiresAt - backupTimestamp)`

### 複雜度

| 操作 | 時間複雜度 | 說明 |
|------|-----------|------|
| set / get / delete | O(1) | HashMap |
| scan / scan_by_prefix | O(F log F) | F = field 數 |
| backup | O(K × F) | Deep copy |
| restore | O(K × F) | Deep copy + TTL 修正 |

---

## Go 完整解法

```go
package main

import (
	"fmt"
	"sort"
	"strings"
)

// ── 資料結構 ──────────────────────────────────────────────

type fieldValue struct {
	value     string
	expiresAt int
	hasExpiry bool
}

type backupEntry struct {
	timestamp int
	records   map[string]map[string]*fieldValue
}

type InMemoryDB struct {
	records map[string]map[string]*fieldValue
	backups []backupEntry
}

func NewInMemoryDB() *InMemoryDB {
	return &InMemoryDB{
		records: make(map[string]map[string]*fieldValue),
	}
}

// ── 內部工具 ──────────────────────────────────────────────

func (db *InMemoryDB) isExpired(fv *fieldValue, ts int) bool {
	return fv.hasExpiry && ts >= fv.expiresAt
}

func (db *InMemoryDB) getField(ts int, key, field string) (*fieldValue, bool) {
	fields, ok := db.records[key]
	if !ok {
		return nil, false
	}
	fv, ok := fields[field]
	if !ok || db.isExpired(fv, ts) {
		return nil, false
	}
	return fv, true
}

func deepCopyRecords(src map[string]map[string]*fieldValue) map[string]map[string]*fieldValue {
	dst := make(map[string]map[string]*fieldValue, len(src))
	for key, fields := range src {
		dstFields := make(map[string]*fieldValue, len(fields))
		for field, fv := range fields {
			copied := *fv
			dstFields[field] = &copied
		}
		dst[key] = dstFields
	}
	return dst
}

// ── Level 1 ───────────────────────────────────────────────

func (db *InMemoryDB) Set(key, field, value string) {
	db.SetAt(0, key, field, value)
}

func (db *InMemoryDB) Get(key, field string) (string, bool) {
	return db.GetAt(0, key, field)
}

func (db *InMemoryDB) Delete(key, field string) bool {
	return db.DeleteAt(0, key, field)
}

// ── Level 2 ───────────────────────────────────────────────

func (db *InMemoryDB) Scan(key string) []string {
	return db.ScanAt(0, key)
}

func (db *InMemoryDB) ScanByPrefix(key, prefix string) []string {
	return db.ScanByPrefixAt(0, key, prefix)
}

// ── Level 3 ───────────────────────────────────────────────

func (db *InMemoryDB) SetAt(ts int, key, field, value string) {
	if _, ok := db.records[key]; !ok {
		db.records[key] = make(map[string]*fieldValue)
	}
	db.records[key][field] = &fieldValue{value: value}
}

func (db *InMemoryDB) SetAtWithTTL(ts int, key, field, value string, ttl int) {
	if _, ok := db.records[key]; !ok {
		db.records[key] = make(map[string]*fieldValue)
	}
	db.records[key][field] = &fieldValue{
		value:     value,
		expiresAt: ts + ttl,
		hasExpiry: true,
	}
}

func (db *InMemoryDB) GetAt(ts int, key, field string) (string, bool) {
	fv, ok := db.getField(ts, key, field)
	if !ok {
		return "", false
	}
	return fv.value, true
}

func (db *InMemoryDB) DeleteAt(ts int, key, field string) bool {
	_, ok := db.getField(ts, key, field)
	if !ok {
		return false
	}
	delete(db.records[key], field)
	return true
}

func (db *InMemoryDB) ScanAt(ts int, key string) []string {
	return db.ScanByPrefixAt(ts, key, "")
}

func (db *InMemoryDB) ScanByPrefixAt(ts int, key, prefix string) []string {
	fields, ok := db.records[key]
	if !ok {
		return []string{}
	}
	var names []string
	for f, fv := range fields {
		if db.isExpired(fv, ts) {
			continue
		}
		if strings.HasPrefix(f, prefix) {
			names = append(names, f)
		}
	}
	sort.Strings(names)
	result := make([]string, len(names))
	for i, f := range names {
		result[i] = fmt.Sprintf("%s(%s)", f, fields[f].value)
	}
	return result
}

// ── Level 4 ───────────────────────────────────────────────

func (db *InMemoryDB) Backup(ts int) int {
	snapshot := deepCopyRecords(db.records)
	db.backups = append(db.backups, backupEntry{timestamp: ts, records: snapshot})

	// 計算存活 field 數
	count := 0
	for _, fields := range snapshot {
		for _, fv := range fields {
			if !db.isExpired(fv, ts) {
				count++
			}
		}
	}
	return count
}

func (db *InMemoryDB) Restore(ts, tsToRestore int) {
	// 找最後一個 timestamp <= tsToRestore 的備份
	idx := -1
	for i, b := range db.backups {
		if b.timestamp <= tsToRestore {
			idx = i
		}
	}
	if idx == -1 {
		return
	}
	backupTS := db.backups[idx].timestamp
	restored := deepCopyRecords(db.backups[idx].records)

	// 重新計算 TTL：剩餘時間從 ts 開始重新計算
	for _, fields := range restored {
		for _, fv := range fields {
			if fv.hasExpiry {
				remaining := fv.expiresAt - backupTS
				fv.expiresAt = ts + remaining
			}
		}
	}
	db.records = restored
}
```

---

## 相關頁面

- [[Coinbase_HA_總覽]] — 題目版本、分數結構、應試策略
- [[Banking_System]] — Version A 題目 + Go 解法
