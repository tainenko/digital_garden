---
title: "SD題解：Consistent Hashing（一致性雜湊）"
type: topic
tags: [system-design, consistent-hashing, distributed-cache, golang]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：Consistent Hashing（一致性雜湊）

> **難度**: 概念題 ｜ **頻率**: 極高（分散式快取/資料庫必考）

---

## 問題背景

普通 Hash 分片：`node = hash(key) % N`

**問題**：當節點數 N 改變（加機器/宕機），幾乎所有 key 都要重新映射 → 快取全部失效（Cache Stampede）

**Consistent Hashing 解法**：節點增減時，只有 `1/N` 的 key 需要重新映射

---

## 核心原理

1. 把 Hash 空間想像成一個圓環（0 ~ 2³² - 1）
2. 節點和 key 都映射到環上
3. 每個 key 順時針找到的第一個節點就是它的負責節點
4. 加入/移除節點時，只影響相鄰節點的 key

**虛擬節點（Virtual Nodes）**：每個物理節點在環上放多個虛擬點，讓負載更均衡

---

## Go 實現

```go
package consistenthash

import (
    "crypto/md5"
    "fmt"
    "sort"
    "sync"
)

type Hash func(data []byte) uint32

type ConsistentHash struct {
    hash         Hash
    replicas     int            // 每個節點的虛擬節點數
    ring         []int          // 排序後的虛擬節點 hash 值
    nodeMap      map[int]string // hash → 節點名稱
    mu           sync.RWMutex
}

func New(replicas int, fn Hash) *ConsistentHash {
    ch := &ConsistentHash{
        replicas: replicas,
        hash:     fn,
        nodeMap:  make(map[int]string),
    }
    if ch.hash == nil {
        // 預設用 MD5 取前4位元組
        ch.hash = func(data []byte) uint32 {
            h := md5.Sum(data)
            return uint32(h[0])<<24 | uint32(h[1])<<16 | uint32(h[2])<<8 | uint32(h[3])
        }
    }
    return ch
}

// AddNode 加入節點（含虛擬節點）
func (ch *ConsistentHash) AddNode(nodes ...string) {
    ch.mu.Lock()
    defer ch.mu.Unlock()

    for _, node := range nodes {
        for i := 0; i < ch.replicas; i++ {
            // 虛擬節點 key = "node-0", "node-1", ...
            virtualKey := fmt.Sprintf("%s-%d", node, i)
            h := int(ch.hash([]byte(virtualKey)))
            ch.ring = append(ch.ring, h)
            ch.nodeMap[h] = node
        }
    }
    sort.Ints(ch.ring)
}

// RemoveNode 移除節點
func (ch *ConsistentHash) RemoveNode(node string) {
    ch.mu.Lock()
    defer ch.mu.Unlock()

    for i := 0; i < ch.replicas; i++ {
        virtualKey := fmt.Sprintf("%s-%d", node, i)
        h := int(ch.hash([]byte(virtualKey)))
        delete(ch.nodeMap, h)

        // 從 ring 中移除
        idx := sort.SearchInts(ch.ring, h)
        if idx < len(ch.ring) && ch.ring[idx] == h {
            ch.ring = append(ch.ring[:idx], ch.ring[idx+1:]...)
        }
    }
}

// GetNode 找到 key 應該存在哪個節點
func (ch *ConsistentHash) GetNode(key string) string {
    ch.mu.RLock()
    defer ch.mu.RUnlock()

    if len(ch.ring) == 0 {
        return ""
    }

    h := int(ch.hash([]byte(key)))

    // 在環上找第一個 >= h 的位置（順時針）
    idx := sort.SearchInts(ch.ring, h)

    // 如果超過環尾，繞回到第一個節點
    if idx == len(ch.ring) {
        idx = 0
    }

    return ch.nodeMap[ch.ring[idx]]
}

// GetNodes 取得 key 的 N 個副本節點（用於複製）
func (ch *ConsistentHash) GetNodes(key string, n int) []string {
    ch.mu.RLock()
    defer ch.mu.RUnlock()

    if len(ch.ring) == 0 || n <= 0 {
        return nil
    }

    h := int(ch.hash([]byte(key)))
    idx := sort.SearchInts(ch.ring, h)

    seen := make(map[string]bool)
    var nodes []string

    for len(nodes) < n && len(nodes) < len(ch.nodeMap)/ch.replicas {
        nodeIdx := idx % len(ch.ring)
        node := ch.nodeMap[ch.ring[nodeIdx]]
        if !seen[node] {
            seen[node] = true
            nodes = append(nodes, node)
        }
        idx++
    }

    return nodes
}
```

### 使用範例

```go
func main() {
    ch := New(150, nil) // 每個節點 150 個虛擬節點

    // 初始 3 個節點
    ch.AddNode("node-A", "node-B", "node-C")

    // 查詢 key 的負責節點
    keys := []string{"user:1001", "user:1002", "product:42", "order:9999"}
    before := make(map[string]string)
    for _, k := range keys {
        before[k] = ch.GetNode(k)
        fmt.Printf("Before: %s → %s\n", k, before[k])
    }

    // 加入新節點
    ch.AddNode("node-D")
    fmt.Println("\n加入 node-D 後：")

    remapped := 0
    for _, k := range keys {
        after := ch.GetNode(k)
        changed := ""
        if before[k] != after {
            changed = " ← 重新映射"
            remapped++
        }
        fmt.Printf("After:  %s → %s%s\n", k, after, changed)
    }
    fmt.Printf("\n重新映射比例: %d/%d (理論值 ~25%%)\n", remapped, len(keys))
}
```

---

## 面試常問延伸

### Q: 為什麼需要虛擬節點？

沒有虛擬節點時，節點在環上的分布不均，導致部分節點承受過多負載。虛擬節點讓每個物理節點均勻散布在環上。

```
虛擬節點數 = 150 時，標準差 < 5%
虛擬節點數 = 10 時，標準差 > 20%
```

### Q: Consistent Hashing 用在哪裡？

- **分散式快取**（Memcached、Redis Cluster）：決定 key 存哪台
- **負載均衡**（Nginx）：讓同一個 user 固定打到同一台 server（Session Affinity）
- **資料庫 Sharding**：決定資料存哪個分片

### Q: Consistent Hashing vs Range Sharding？

| | Consistent Hashing | Range Sharding |
|--|--|--|
| 節點增減影響 | 只影響鄰近節點（1/N） | 可能影響所有節點 |
| 範圍查詢 | 不支援（hash 後無序） | 支援（按 key 排序）|
| 負載均衡 | 虛擬節點可均衡 | 需要手動調整 |
| 適用 | KV 快取、Hash 查詢 | 時序資料、範圍查詢 |

---

## 相關概念

- [[分散式系統基礎概念]] — 一致性雜湊的理論背景
- [[sd-distributed-cache|SD題解：分散式快取]] — 一致性雜湊的主要應用場景
- [[系統設計核心技術棧]] — Redis Cluster 的實作
