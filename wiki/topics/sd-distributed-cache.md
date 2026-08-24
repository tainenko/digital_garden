---
title: "SD題解：分散式快取（LRU Cache）"
type: topic
tags: [system-design, cache, lru, distributed-cache, golang, easy]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：分散式快取（Distributed Cache / LRU Cache）

> **難度**: 入門 ｜ **頻率**: 極高（幾乎每題都用到）｜ **代表**: Redis, Memcached

---

## 兩個考法

1. **設計 LRU Cache 資料結構**（Algorithm/OOD 考法）
2. **設計分散式快取系統**（System Design 考法）

---

## Part 1：LRU Cache 資料結構（Go 實現）

**LRU（Least Recently Used）**：容量滿時，淘汰最久未使用的資料

**核心結構**：HashMap + 雙向鏈結串列
- HashMap：O(1) 查找
- 雙向鏈表：O(1) 移到頭部（最近使用）、O(1) 刪除尾部（最久未用）

```go
package cache

import "container/list"

type LRUCache struct {
    capacity int
    list     *list.List               // 雙向鏈表，頭=最近使用，尾=最久未用
    items    map[string]*list.Element // key → 鏈表節點
}

type entry struct {
    key   string
    value interface{}
}

func NewLRUCache(capacity int) *LRUCache {
    return &LRUCache{
        capacity: capacity,
        list:     list.New(),
        items:    make(map[string]*list.Element),
    }
}

// Get O(1)
func (c *LRUCache) Get(key string) (interface{}, bool) {
    if elem, ok := c.items[key]; ok {
        c.list.MoveToFront(elem) // 移到頭部（最近使用）
        return elem.Value.(*entry).value, true
    }
    return nil, false
}

// Put O(1)
func (c *LRUCache) Put(key string, value interface{}) {
    if elem, ok := c.items[key]; ok {
        // 更新已存在的 key
        c.list.MoveToFront(elem)
        elem.Value.(*entry).value = value
        return
    }

    // 新增 key
    if c.list.Len() >= c.capacity {
        // 容量滿了，淘汰最久未用（鏈表尾部）
        c.evict()
    }

    elem := c.list.PushFront(&entry{key, value})
    c.items[key] = elem
}

func (c *LRUCache) evict() {
    tail := c.list.Back()
    if tail == nil {
        return
    }
    c.list.Remove(tail)
    delete(c.items, tail.Value.(*entry).key)
}

// Delete O(1)
func (c *LRUCache) Delete(key string) {
    if elem, ok := c.items[key]; ok {
        c.list.Remove(elem)
        delete(c.items, key)
    }
}

func (c *LRUCache) Len() int {
    return c.list.Len()
}
```

### 線程安全版本（帶鎖）

```go
type ThreadSafeLRU struct {
    cache *LRUCache
    mu    sync.RWMutex
}

func (t *ThreadSafeLRU) Get(key string) (interface{}, bool) {
    t.mu.Lock() // 注意：MoveToFront 需要 write lock
    defer t.mu.Unlock()
    return t.cache.Get(key)
}

func (t *ThreadSafeLRU) Put(key string, value interface{}) {
    t.mu.Lock()
    defer t.mu.Unlock()
    t.cache.Put(key, value)
}
```

---

## Part 2：分散式快取系統設計

### 架構

```
Client App
    ↓
Cache Client Library（一致性雜湊決定打哪台）
    ↓         ↓         ↓
Cache Node1  Node2     Node3
（各自是獨立的 LRU Cache）
```

### Cache Client（含一致性雜湊）

```go
package dcache

import (
    "context"
    "fmt"
    "time"
)

type CacheClient struct {
    ring    *ConsistentHash // 見 sd-consistent-hashing
    clients map[string]*NodeClient
}

type NodeClient struct {
    addr string
}

// Get 從正確節點取資料
func (c *CacheClient) Get(ctx context.Context, key string) ([]byte, error) {
    node := c.ring.GetNode(key)
    client := c.clients[node]
    return client.get(ctx, key)
}

// Set 存到正確節點
func (c *CacheClient) Set(ctx context.Context, key string, value []byte, ttl time.Duration) error {
    node := c.ring.GetNode(key)
    client := c.clients[node]
    return client.set(ctx, key, value, ttl)
}

// GetWithFallback Cache-Aside 模式：先查快取，miss 則查 DB 並回填
func (c *CacheClient) GetWithFallback(ctx context.Context, key string, fetch func() ([]byte, error)) ([]byte, error) {
    // 1. 查快取
    if val, err := c.Get(ctx, key); err == nil {
        return val, nil
    }

    // 2. Cache Miss：查 DB
    val, err := fetch()
    if err != nil {
        return nil, fmt.Errorf("fetch failed: %w", err)
    }

    // 3. 回填快取（非同步，不讓回填影響主鏈路延遲）
    go func() {
        c.Set(context.Background(), key, val, 1*time.Hour)
    }()

    return val, nil
}
```

---

## 快取策略比較

```go
// Cache-Aside（最常見）
func cacheAside(key string) (Data, error) {
    if val, ok := cache.Get(key); ok {
        return val, nil         // Cache Hit
    }
    val, _ := db.Get(key)      // Cache Miss → 查 DB
    cache.Set(key, val, ttl)   // 回填
    return val, nil
}

// Write-Through（寫 DB 同時更新 Cache）
func writeThrough(key string, val Data) error {
    db.Set(key, val)            // 先寫 DB
    cache.Set(key, val, ttl)   // 再更新 Cache
    return nil
}

// Write-Behind / Write-Back（先寫 Cache，非同步寫 DB）
func writeBehind(key string, val Data) error {
    cache.Set(key, val, ttl)        // 立即寫 Cache
    queue.Publish("db-write", val)  // 非同步寫 DB（可能丟資料）
    return nil
}

// Write-Around（只寫 DB，不更新 Cache，讓 Cache 自然過期）
func writeAround(key string, val Data) error {
    db.Set(key, val)
    cache.Delete(key) // 主動刪除 Cache 讓它失效
    return nil
}
```

---

## 常見問題與解法

### Cache Stampede（快取擊穿）
**問題**：熱門 key 過期瞬間，大量請求同時打到 DB

```go
// 解法：Mutex Lock（只讓一個請求查 DB，其他等待）
func getWithMutex(key string) Data {
    if val, ok := cache.Get(key); ok {
        return val
    }
    mu := getKeyMutex(key) // 每個 key 一把鎖
    mu.Lock()
    defer mu.Unlock()

    // Double-check（等到鎖後再次確認）
    if val, ok := cache.Get(key); ok {
        return val
    }
    val, _ := db.Get(key)
    cache.Set(key, val, ttl)
    return val
}
```

### Cache Avalanche（快取雪崩）
**問題**：大量 key 同時過期，DB 被瞬間壓垮

```go
// 解法：TTL 加隨機抖動
func setWithJitter(key string, val Data, baseTTL time.Duration) {
    jitter := time.Duration(rand.Int63n(int64(baseTTL / 10))) // ±10% 隨機
    cache.Set(key, val, baseTTL+jitter)
}
```

### Cache Penetration（快取穿透）
**問題**：查詢不存在的 key，每次都打到 DB

```go
// 解法 1：空值也快取（短 TTL）
if val == nil {
    cache.Set(key, "NULL", 30*time.Second) // 快取空值
}

// 解法 2：Bloom Filter 前置過濾
if !bloomFilter.Contains(key) {
    return nil, ErrNotFound // 確定不存在，直接返回
}
```

---

## 相關題解

- [[sd-consistent-hashing|SD題解：一致性雜湊]] — 分散式快取的核心路由機制
- [[sd-url-shortener|SD題解：URL Shortener]] — Cache 的典型應用
- [[系統設計核心技術棧]] — Redis 詳細說明
