---
title: "SD題解：分散式網路爬蟲（Web Crawler）"
type: topic
tags: [system-design, web-crawler, bloom-filter, distributed, golang, hard]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：分散式網路爬蟲（Web Crawler）

> **難度**: 進階 ｜ **頻率**: 中等 ｜ **代表**: Google Search Index, Common Crawl

---

## RESHADED 快速分析

**R - Requirements**
- 從種子 URL 出發，遞迴爬取網頁
- 提取文字內容存入索引、提取超連結繼續爬取
- 遵守 robots.txt、避免重複爬取
- 非功能：大規模（數十億網頁）、可擴展

**E - Estimation**
- 目標：10億個網頁，在 1 個月內爬完
- 平均網頁大小：500KB（含 HTML/圖片）
- 每秒需爬：10億 / (30天 × 86400秒) ≈ **385 頁/秒**
- 每個網頁含 ~50個連結 → URL 佇列吞吐量：385 × 50 = 19,250 URLs/秒
- 儲存：10億 × 500KB = **500TB**

---

## 核心挑戰

1. **URL 去重**：10 億 URL，不能每次都查 DB
2. **爬取優先度**：重要頁面（PageRank 高）先爬
3. **禮貌性爬取**：同一個網站不能爬太快（robots.txt, Crawl-delay）
4. **分散式協調**：多台爬蟲如何分工不衝突

---

## Go 實現

### 1. Bloom Filter（URL 去重）

Bloom Filter 可用少量記憶體判斷「這個 URL 是否已爬過」：
- False Positive：可能誤判「已爬過」（跳過未爬的 URL，可接受）
- False Negative：不會誤判「未爬過」（不會重複爬）

```go
package crawler

import (
    "crypto/md5"
    "crypto/sha256"
    "math"
    "sync"
)

type BloomFilter struct {
    bits    []bool
    size    uint
    hashFns int
    mu      sync.RWMutex
}

// NewBloomFilter 建立 Bloom Filter
// n = 預期元素數量, fp = 誤判率（如 0.01 = 1%）
func NewBloomFilter(n int, fp float64) *BloomFilter {
    // 最佳 bit 數公式：m = -n*ln(fp) / (ln(2))^2
    size := uint(-float64(n)*math.Log(fp) / math.Pow(math.Log(2), 2))
    // 最佳 hash 函數數：k = (m/n) * ln(2)
    hashFns := int(float64(size) / float64(n) * math.Log(2))
    if hashFns < 2 {
        hashFns = 2
    }

    return &BloomFilter{
        bits:    make([]bool, size),
        size:    size,
        hashFns: hashFns,
    }
}

// Add 加入 URL
func (bf *BloomFilter) Add(url string) {
    bf.mu.Lock()
    defer bf.mu.Unlock()
    for _, pos := range bf.positions(url) {
        bf.bits[pos] = true
    }
}

// Contains 檢查 URL 是否可能已存在
func (bf *BloomFilter) Contains(url string) bool {
    bf.mu.RLock()
    defer bf.mu.RUnlock()
    for _, pos := range bf.positions(url) {
        if !bf.bits[pos] {
            return false // 確定不存在
        }
    }
    return true // 可能存在（有 fp% 誤判率）
}

func (bf *BloomFilter) positions(url string) []uint {
    positions := make([]uint, bf.hashFns)
    md5Hash := md5.Sum([]byte(url))
    sha256Hash := sha256.Sum256([]byte(url))

    for i := 0; i < bf.hashFns; i++ {
        // 用兩個 hash 函數線性組合模擬多個 hash
        h1 := uint(md5Hash[0])<<8 | uint(md5Hash[1])
        h2 := uint(sha256Hash[0])<<8 | uint(sha256Hash[1])
        positions[i] = (h1 + uint(i)*h2) % bf.size
    }
    return positions
}

// 記憶體估算：10億 URL, fp=1% → 約 1.2 GB
// 10億 × (-ln(0.01)) / (ln(2))^2 / 8 bytes ≈ 1.2 GB
```

### 2. 優先佇列（URL Frontier）

```go
type URLFrontier struct {
    // 多個優先級佇列（0=最高，9=最低）
    queues [10]*PriorityQueue
    mu     sync.Mutex
}

type URLItem struct {
    URL      string
    Priority int     // 0-9（PageRank/重要性）
    Domain   string
    Depth    int
}

func (f *URLFrontier) Enqueue(item *URLItem) {
    f.mu.Lock()
    defer f.mu.Unlock()
    f.queues[item.Priority].Push(item)
}

// Dequeue 取出下一個要爬的 URL
// 考慮：禮貌性爬取（同一 domain 間隔至少 1 秒）
func (f *URLFrontier) Dequeue(politeness *PolitenessTracker) *URLItem {
    f.mu.Lock()
    defer f.mu.Unlock()

    for priority := 0; priority < 10; priority++ {
        q := f.queues[priority]
        for q.Len() > 0 {
            item := q.Peek().(*URLItem)
            if politeness.CanCrawl(item.Domain) {
                politeness.Record(item.Domain)
                return q.Pop().(*URLItem)
            }
        }
    }
    return nil
}
```

### 3. 禮貌性爬取（Politeness）

```go
type PolitenessTracker struct {
    lastCrawled map[string]time.Time // domain → 最後爬取時間
    crawlDelay  map[string]time.Duration // 從 robots.txt 讀取
    mu          sync.RWMutex
}

func (p *PolitenessTracker) CanCrawl(domain string) bool {
    p.mu.RLock()
    defer p.mu.RUnlock()

    lastTime, ok := p.lastCrawled[domain]
    if !ok {
        return true
    }
    delay := p.getCrawlDelay(domain)
    return time.Since(lastTime) >= delay
}

func (p *PolitenessTracker) getCrawlDelay(domain string) time.Duration {
    if delay, ok := p.crawlDelay[domain]; ok {
        return delay
    }
    return 1 * time.Second // 預設 1 秒
}

// ParseRobotsTxt 解析 robots.txt
func (p *PolitenessTracker) ParseRobotsTxt(domain, content string) {
    // 解析 Crawl-delay 和 Disallow 規則
    lines := strings.Split(content, "\n")
    for _, line := range lines {
        if strings.HasPrefix(line, "Crawl-delay:") {
            seconds, _ := strconv.Atoi(strings.TrimSpace(strings.TrimPrefix(line, "Crawl-delay:")))
            p.mu.Lock()
            p.crawlDelay[domain] = time.Duration(seconds) * time.Second
            p.mu.Unlock()
        }
    }
}
```

### 4. 爬蟲主體

```go
type Crawler struct {
    frontier   *URLFrontier
    bloom      *BloomFilter
    politeness *PolitenessTracker
    client     *http.Client
    parser     HTMLParser
    storage    ContentStorage
    workers    int
}

func (c *Crawler) Start(ctx context.Context) {
    sem := make(chan struct{}, c.workers) // 控制並發數

    for {
        select {
        case <-ctx.Done():
            return
        default:
        }

        item := c.frontier.Dequeue(c.politeness)
        if item == nil {
            time.Sleep(100 * time.Millisecond)
            continue
        }

        sem <- struct{}{} // 取得 semaphore
        go func(urlItem *URLItem) {
            defer func() { <-sem }() // 釋放
            c.crawlPage(ctx, urlItem)
        }(item)
    }
}

func (c *Crawler) crawlPage(ctx context.Context, item *URLItem) {
    // 1. 下載網頁
    resp, err := c.client.Get(item.URL)
    if err != nil || resp.StatusCode != 200 {
        return
    }
    defer resp.Body.Close()
    body, _ := io.ReadAll(resp.Body)

    // 2. 儲存內容
    c.storage.Save(item.URL, body)

    // 3. 提取連結
    links := c.parser.ExtractLinks(item.URL, body)

    // 4. 新連結加入佇列（Bloom Filter 去重）
    for _, link := range links {
        if !c.bloom.Contains(link) {
            c.bloom.Add(link)
            priority := c.estimatePriority(link, item.Depth)
            c.frontier.Enqueue(&URLItem{
                URL:      link,
                Priority: priority,
                Domain:   extractDomain(link),
                Depth:    item.Depth + 1,
            })
        }
    }
}

func (c *Crawler) estimatePriority(url string, depth int) int {
    // 深度越深，優先度越低
    if depth > 5 {
        return 9
    }
    // 知名網站優先
    if isHighValueDomain(url) {
        return 0
    }
    return min(depth, 5)
}
```

### 5. 分散式協調（多台爬蟲分工）

```go
// 用一致性雜湊將 domain 分配給特定爬蟲節點
// 確保同一個 domain 的 URL 由同一台爬蟲處理（禮貌性爬取更容易管理）

type DistributedCoordinator struct {
    ring    *ConsistentHash // 見 sd-consistent-hashing
    nodeID  string
}

func (d *DistributedCoordinator) ShouldCrawl(url string) bool {
    domain := extractDomain(url)
    assignedNode := d.ring.GetNode(domain)
    return assignedNode == d.nodeID
}
```

---

## 架構圖

```
種子 URL → URL Frontier（優先佇列）
                ↓
         ┌──────┼──────┐
      Worker1  Worker2  Worker3   ← 多個爬蟲 Worker
         ↓       ↓       ↓
      Bloom Filter 去重（共享，Redis 或 BitMap）
         ↓
    HTTP 下載（遵守 robots.txt）
         ↓
    HTML 解析 → 儲存內容 → 提取新 URL
                               ↓
                        加入 URL Frontier（循環）
```

---

## Trade-offs

| 決策 | 選擇 | 理由 |
|------|------|------|
| URL 去重 | Bloom Filter | 記憶體 O(1)，1.2GB 處理 10億 URL；DB 查詢太慢 |
| URL 排序 | 多級優先佇列 | 重要頁面先爬，最大化爬取價值 |
| 禮貌性 | Crawl-delay + domain 限速 | 避免 IP 被封、對目標網站友好 |
| 分工 | 一致性雜湊（domain → worker）| 同 domain 由同台 worker 處理，利於限速管理 |

---

## 相關題解

- [[sd-consistent-hashing|SD題解：一致性雜湊]] — 分散式 Worker 的分工機制
- [[sd-distributed-cache|SD題解：分散式快取]] — Bloom Filter 的原理類似
