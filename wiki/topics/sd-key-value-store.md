---
title: 分散式 Key-Value Store（DynamoDB / Redis Cluster）
type: topic
tags: [system-design, interview, distributed-systems, storage, consistency]
created: 2026-04-20
updated: 2026-04-20
---

# 分散式 Key-Value Store（DynamoDB / Redis Cluster）

難度：進階｜核心技術：Consistent Hashing、複製、CAP 定理、向量時鐘、LSM Tree

---

## RESHADED 分析

### Requirements
**Functional:**
- `Put(key, value)` — 儲存鍵值對
- `Get(key)` → value — 讀取
- `Delete(key)` — 刪除
- Key/Value 大小：key ≤ 10 bytes，value ≤ 100 KB
- 支援 TTL（過期自動刪除）

**Non-functional:**
- 可用性優先（AP 系統）
- 最終一致性（Eventual Consistency）
- 可橫向擴展到數千個節點
- 低延遲：P99 讀寫 < 10ms

### Estimates
```
總儲存量: 1 PB（分散到 1,000 個節點 = 每節點 1 TB）
讀 QPS: 1,000,000/秒
寫 QPS: 100,000/秒
```

---

## 核心技術深探

### 1. 資料分片：一致性 Hash

參見 [[sd-consistent-hashing]]，核心是虛擬節點確保均勻分布：

```go
type KVRouter struct {
    ring      *ConsistentHashRing
    nodes     map[string]*KVNode
    mu        sync.RWMutex
}

func (r *KVRouter) GetNodes(key string, count int) []*KVNode {
    r.mu.RLock()
    defer r.mu.RUnlock()

    // 取 N 個連續的節點（用於複製）
    nodeIDs := r.ring.GetN(key, count)
    nodes := make([]*KVNode, 0, len(nodeIDs))
    for _, id := range nodeIDs {
        if n, ok := r.nodes[id]; ok {
            nodes = append(nodes, n)
        }
    }
    return nodes
}
```

### 2. 複製策略（N/W/R 模型，仿 Dynamo）

```go
const (
    N = 3  // 複製到 3 個節點
    W = 2  // 寫入需要 2 個節點確認（Quorum Write）
    R = 2  // 讀取需要 2 個節點回應（Quorum Read）
    // W + R > N 保證強一致性；W=1 R=1 保證高可用
)

type KVClient struct {
    router  *KVRouter
}

func (c *KVClient) Put(ctx context.Context, key string, value []byte, ttl time.Duration) error {
    nodes := c.router.GetNodes(key, N)

    // 向量時鐘版本號
    vc := NewVectorClock()
    vc.Increment(localNodeID)

    successCount := 0
    var mu sync.Mutex
    var wg sync.WaitGroup

    for _, node := range nodes {
        wg.Add(1)
        go func(n *KVNode) {
            defer wg.Done()
            err := n.Put(ctx, key, value, vc, ttl)
            if err == nil {
                mu.Lock()
                successCount++
                mu.Unlock()
            }
        }(node)
    }
    wg.Wait()

    if successCount < W {
        return ErrWriteQuorumNotMet
    }
    return nil
}

func (c *KVClient) Get(ctx context.Context, key string) ([]byte, error) {
    nodes := c.router.GetNodes(key, N)

    type result struct {
        value []byte
        vc    VectorClock
        err   error
    }
    results := make(chan result, N)

    for _, node := range nodes {
        go func(n *KVNode) {
            v, vc, err := n.Get(ctx, key)
            results <- result{v, vc, err}
        }(node)
    }

    var responses []result
    for i := 0; i < R; i++ {
        r := <-results
        if r.err == nil {
            responses = append(responses, r)
        }
    }

    if len(responses) < R {
        return nil, ErrReadQuorumNotMet
    }

    // 用向量時鐘選最新版本（衝突時用 Last-Write-Wins）
    latest := responses[0]
    for _, r := range responses[1:] {
        if r.vc.IsNewerThan(latest.vc) {
            latest = r
        }
    }

    return latest.value, nil
}
```

### 3. 向量時鐘（衝突偵測）

```go
type VectorClock map[string]int64  // nodeID → version

func (vc VectorClock) Increment(nodeID string) {
    vc[nodeID]++
}

func (vc VectorClock) IsNewerThan(other VectorClock) bool {
    hasGreater := false
    for node, ver := range vc {
        otherVer := other[node]
        if ver < otherVer {
            return false  // vc 中有比 other 小的，不是更新
        }
        if ver > otherVer {
            hasGreater = true
        }
    }
    return hasGreater
}

func (vc VectorClock) IsConflictWith(other VectorClock) bool {
    // 沒有任一方完全 ≥ 另一方
    return !vc.IsNewerThan(other) && !other.IsNewerThan(vc)
}
```

### 4. 儲存引擎：LSM Tree（Log-Structured Merge Tree）

RocksDB / LevelDB 等 KV 儲存引擎的核心原理：

```go
// WAL（Write-Ahead Log）+ MemTable + SSTable
type LSMTree struct {
    wal      *WAL
    memTable *MemTable  // 有序的記憶體結構（紅黑樹/Skip List）
    ssTables []*SSTable // 磁碟上的不可變有序文件（L0 → L1 → L2...）
    mu       sync.RWMutex
}

func (t *LSMTree) Put(key, value []byte) error {
    // 1. 先寫 WAL（crash recovery 用）
    if err := t.wal.Append(key, value); err != nil {
        return err
    }

    // 2. 寫入 MemTable
    t.mu.Lock()
    t.memTable.Put(key, value)
    t.mu.Unlock()

    // 3. MemTable 超過 64MB，轉為 SSTable（immutable flush）
    if t.memTable.Size() > 64*1024*1024 {
        go t.flushMemTable()
    }

    return nil
}

func (t *LSMTree) Get(key []byte) ([]byte, bool) {
    // 讀取順序：MemTable → L0 SSTable → L1 → L2...（越新的越先）
    if v, ok := t.memTable.Get(key); ok {
        return v, true
    }

    for _, sst := range t.ssTables {
        if sst.MightContain(key) {  // Bloom Filter 快速過濾
            if v, ok := sst.Get(key); ok {
                return v, true
            }
        }
    }

    return nil, false
}

// 後台 Compaction：合併多個 SSTable，刪除舊版本/已刪除的 key
func (t *LSMTree) compact() {
    // Leveled Compaction：L0 → L1 時合併重疊的 key range
    // 減少讀放大（read amplification）
}
```

**LSM Tree vs B-Tree 比較**：

| 特性 | LSM Tree | B-Tree |
|------|---------|--------|
| 寫入性能 | 極高（順序寫） | 中等（隨機寫） |
| 讀取性能 | 較慢（多層查找） | 快（O(log n)） |
| 空間效率 | 低（壓縮前有多份） | 高 |
| 適用 | 寫多讀少（Cassandra、RocksDB）| 讀多寫少（InnoDB、PostgreSQL）|

### 5. Gossip 協議（節點發現與故障檢測）

```go
type GossipNode struct {
    NodeID    string
    Address   string
    Heartbeat int64
    Status    string  // alive, suspected, dead
}

type GossipService struct {
    localNode *GossipNode
    peers     map[string]*GossipNode
    mu        sync.RWMutex
}

// 每秒隨機選 3 個節點交換狀態
func (s *GossipService) Gossip() {
    for range time.Tick(time.Second) {
        targets := s.randomPeers(3)

        for _, target := range targets {
            go func(t *GossipNode) {
                myState := s.getLocalState()
                theirState, err := sendGossip(t.Address, myState)
                if err != nil {
                    t.Status = "suspected"
                    return
                }

                s.mu.Lock()
                s.mergeState(theirState)
                s.mu.Unlock()
            }(target)
        }

        // 更新本地心跳
        s.mu.Lock()
        s.localNode.Heartbeat++
        s.mu.Unlock()
    }
}
```

---

## Trade-offs

| 決策 | 選擇 | 理由 |
|------|------|------|
| CAP | **AP（可用性 + 分區容忍）** | KV Store 通常選最終一致性 |
| 複製 | **N=3, W=2, R=2** | 平衡讀寫，一個節點故障不影響服務 |
| 儲存引擎 | **LSM Tree** | 高寫入吞吐量 |
| 故障檢測 | **Gossip** | 去中心化，無單點故障 |
| 衝突解決 | **向量時鐘 + LWW** | 精確衝突偵測 + 簡單解決策略 |

---

## 相關頁面

- [[sd-consistent-hashing]] — 一致性 Hash 詳細實作
- [[sd-distributed-cache]] — 快取系統設計
- [[分散式系統基礎概念]] — CAP 定理、一致性模型
