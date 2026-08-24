---
title: "SD題解：CDN 設計（內容分發網路）"
type: topic
tags: [system-design, cdn, cache, edge, golang, easy]
created: 2026-04-29
updated: 2026-04-29
---

# SD題解：CDN 設計（內容分發網路）

> **難度**: 入門 ｜ **頻率**: 高（常作為子系統出現在影片串流/社群 Feed 題）｜ **代表**: Cloudflare, Akamai, AWS CloudFront

---

## RESHADED 快速分析

**R - Requirements**
- 功能：把靜態資源（圖片、JS、CSS、影片片段）就近快取在邊緣節點，讓用戶從最近的 PoP（Point of Presence）取得內容
- 非功能：低延遲（<50ms）、高可用（99.99%）、全球覆蓋、自動 Failover 到 Origin

**E - Estimation**（假設中型 CDN）
- 節點數：全球 200 個 PoP
- 每天請求量：1,000 億次
- 讀取 QPS：1,000億 / 86,400 ≈ **115萬/s**（峰值 3x ≈ 350萬/s）
- 快取命中率：90%（即只有 10% 流量回源）
- Origin QPS：115萬 × 10% ≈ **11.5萬/s**（大幅降低 Origin 壓力）
- 邊緣節點儲存：每節點 10TB（熱資料），全網 = 200 × 10TB = 2PB

**S - Storage**：邊緣節點本地 SSD（L1 Cache）+ 區域快取集群（L2 Cache）+ Origin（S3 / Object Storage）

**H - High-level**

```
用戶
 │
 ▼
DNS（Anycast / GeoDNS 解析到最近 PoP）
 │
 ▼
Edge PoP Node
 ├─ L1 Cache（命中 → 直接回傳）
 └─ L2 Regional Cache（命中 → 回傳並回填 L1）
     └─ Cache Miss → Origin Pull → 存 L1/L2 → 回傳
```

**A - APIs**（對 Origin 的內部協議，非用戶直接呼叫）
- `GET /resource/{key}` — 邊緣節點向 Origin 或上層快取拉取內容
- `PURGE /resource/{key}` — 管理員主動刪除快取（CDN Invalidation API）
- `GET /health` — 節點健康檢查

**D - Detailed**：Pull vs Push 模式、快取失效策略、路由選擇（見下方）

**E - Evaluation**：快取命中率（Cache Hit Ratio）、P99 延遲、回源率、節點故障自動切換

**D - Distinctive**：Anycast BGP 路由 + 分層快取（L1/L2/Origin）是 CDN 最獨特的技術組合

---

## 核心決策：Pull vs Push CDN

| 面向 | Pull CDN | Push CDN |
|------|----------|----------|
| **運作方式** | 第一次請求時從 Origin 拉取，後續快取 | 提前主動推送到所有 PoP |
| **適合內容** | 靜態資源（圖片、JS、CSS） | 大型固定內容（軟體安裝包、影片） |
| **優點** | 零管理、節點只存熱資料 | 無回源延遲、命中率 100% |
| **缺點** | 第一次請求有回源延遲（Cold Start） | 管理複雜、浪費冷門 PoP 儲存空間 |
| **業界範例** | Cloudflare（預設 Pull）| AWS CloudFront（支援兩種）|

**面試首選**：Pull CDN（較常考，配合 TTL 和主動 Purge 解決一致性問題）

---

## 核心決策：路由到最近 PoP

### 方法比較

| 方法 | 原理 | 精度 | 延遲 |
|------|------|------|------|
| GeoDNS | DNS 依 IP 地理位置回傳最近 PoP IP | 中（IP 庫不完整） | DNS TTL（分鐘級） |
| Anycast BGP | 所有 PoP 宣告相同 IP，BGP 路由最短路徑 | 高（網路拓樸決定） | 毫秒級自動切換 |
| HTTP 302 Redirect | 用戶先打中央伺服器，再重定向到最近 PoP | 高 | 多一個 RTT |

**業界實踐**：Cloudflare 用 Anycast；AWS CloudFront 用 GeoDNS。

---

## 核心決策：快取失效策略

快取失效是 CDN 最難的問題（Phil Karlton 名言：計算機科學只有兩件難事）。

| 策略 | 機制 | 適用場景 |
|------|------|---------|
| TTL 過期 | 設定 Cache-Control: max-age=3600 | 靜態資源（圖片、字型） |
| 版本化 URL | `/static/app.v2.js`，永久快取 | JS/CSS 部署更新 |
| 主動 Purge | 呼叫 CDN API 刪除指定 key 的快取 | 即時性要求高（新聞、商品頁） |
| Stale-While-Revalidate | 回傳舊快取，背景更新 | 可容忍短暫過期的場景 |

---

## Go 實現

### 邊緣節點快取核心（L1 LRU）

```go
package cdn

import (
    "container/list"
    "sync"
    "time"
)

type CacheEntry struct {
    key        string
    value      []byte
    contentType string
    expiresAt  time.Time
    size       int64
}

type EdgeCache struct {
    mu       sync.RWMutex
    capacity int64           // 最大容量（bytes）
    used     int64
    items    map[string]*list.Element
    lru      *list.List
}

func NewEdgeCache(capacityBytes int64) *EdgeCache {
    return &EdgeCache{
        capacity: capacityBytes,
        items:    make(map[string]*list.Element),
        lru:      list.New(),
    }
}

func (c *EdgeCache) Get(key string) (*CacheEntry, bool) {
    c.mu.Lock()
    defer c.mu.Unlock()

    el, ok := c.items[key]
    if !ok {
        return nil, false
    }

    entry := el.Value.(*CacheEntry)
    // TTL 檢查
    if time.Now().After(entry.expiresAt) {
        c.removeLocked(key, el)
        return nil, false
    }

    // LRU：移到隊首
    c.lru.MoveToFront(el)
    return entry, true
}

func (c *EdgeCache) Set(key string, value []byte, contentType string, ttl time.Duration) {
    c.mu.Lock()
    defer c.mu.Unlock()

    size := int64(len(value))

    // 已存在則更新
    if el, ok := c.items[key]; ok {
        old := el.Value.(*CacheEntry)
        c.used -= old.size
        c.lru.MoveToFront(el)
        old.value = value
        old.expiresAt = time.Now().Add(ttl)
        old.size = size
        c.used += size
        return
    }

    // 容量不足時逐出 LRU 尾端
    for c.used+size > c.capacity && c.lru.Len() > 0 {
        tail := c.lru.Back()
        if tail == nil {
            break
        }
        old := tail.Value.(*CacheEntry)
        c.removeLocked(old.key, tail)
    }

    entry := &CacheEntry{
        key:         key,
        value:       value,
        contentType: contentType,
        expiresAt:   time.Now().Add(ttl),
        size:        size,
    }
    el := c.lru.PushFront(entry)
    c.items[key] = el
    c.used += size
}

func (c *EdgeCache) Purge(key string) bool {
    c.mu.Lock()
    defer c.mu.Unlock()
    el, ok := c.items[key]
    if !ok {
        return false
    }
    c.removeLocked(key, el)
    return true
}

func (c *EdgeCache) removeLocked(key string, el *list.Element) {
    entry := el.Value.(*CacheEntry)
    c.used -= entry.size
    c.lru.Remove(el)
    delete(c.items, key)
}
```

### 邊緣節點 HTTP 服務（Pull 模式）

```go
package cdn

import (
    "context"
    "fmt"
    "io"
    "net/http"
    "time"
)

type EdgeNode struct {
    cache      *EdgeCache
    originURL  string
    l2URL      string // 上層區域快取
    httpClient *http.Client
}

func NewEdgeNode(originURL, l2URL string, cacheSizeBytes int64) *EdgeNode {
    return &EdgeNode{
        cache:     NewEdgeCache(cacheSizeBytes),
        originURL: originURL,
        l2URL:     l2URL,
        httpClient: &http.Client{Timeout: 10 * time.Second},
    }
}

func (n *EdgeNode) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    key := r.URL.Path

    // 1. L1 Cache Hit
    if entry, ok := n.cache.Get(key); ok {
        w.Header().Set("X-Cache", "HIT-L1")
        w.Header().Set("Content-Type", entry.contentType)
        w.Write(entry.value)
        return
    }

    // 2. L2 區域快取
    if body, ct, ttl, ok := n.fetchFromL2(r.Context(), key); ok {
        n.cache.Set(key, body, ct, ttl)
        w.Header().Set("X-Cache", "HIT-L2")
        w.Header().Set("Content-Type", ct)
        w.Write(body)
        return
    }

    // 3. 回源（Origin Pull）
    body, ct, ttl, err := n.fetchFromOrigin(r.Context(), key)
    if err != nil {
        http.Error(w, "Origin unavailable", http.StatusBadGateway)
        return
    }

    n.cache.Set(key, body, ct, ttl)
    w.Header().Set("X-Cache", "MISS")
    w.Header().Set("Content-Type", ct)
    w.Write(body)
}

func (n *EdgeNode) fetchFromOrigin(ctx context.Context, path string) ([]byte, string, time.Duration, error) {
    resp, err := n.httpClient.Get(n.originURL + path)
    if err != nil {
        return nil, "", 0, err
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusOK {
        return nil, "", 0, fmt.Errorf("origin returned %d", resp.StatusCode)
    }

    body, err := io.ReadAll(resp.Body)
    if err != nil {
        return nil, "", 0, err
    }

    ct := resp.Header.Get("Content-Type")
    ttl := parseCacheControl(resp.Header.Get("Cache-Control"))
    return body, ct, ttl, nil
}

func (n *EdgeNode) fetchFromL2(ctx context.Context, path string) ([]byte, string, time.Duration, bool) {
    if n.l2URL == "" {
        return nil, "", 0, false
    }
    resp, err := n.httpClient.Get(n.l2URL + path)
    if err != nil || resp.StatusCode != http.StatusOK {
        return nil, "", 0, false
    }
    defer resp.Body.Close()
    body, _ := io.ReadAll(resp.Body)
    ct := resp.Header.Get("Content-Type")
    ttl := parseCacheControl(resp.Header.Get("Cache-Control"))
    return body, ct, ttl, true
}

// parseCacheControl 解析 "max-age=3600" → time.Duration
func parseCacheControl(header string) time.Duration {
    var maxAge int
    if _, err := fmt.Sscanf(header, "max-age=%d", &maxAge); err == nil && maxAge > 0 {
        return time.Duration(maxAge) * time.Second
    }
    return 24 * time.Hour // 預設快取 24 小時
}
```

### 主動 Purge API（快取失效）

```go
package cdn

import (
    "encoding/json"
    "net/http"
)

type PurgeRequest struct {
    Keys []string `json:"keys"` // 支援 Wildcard，如 "/images/product/*"
}

// PurgeHandler 讓 Origin 在內容更新後主動通知 CDN 節點清除快取
func (n *EdgeNode) PurgeHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method Not Allowed", http.StatusMethodNotAllowed)
        return
    }

    // 生產環境需驗證管理員 Token
    if r.Header.Get("X-CDN-Admin-Token") != adminToken {
        http.Error(w, "Unauthorized", http.StatusUnauthorized)
        return
    }

    var req PurgeRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, "Bad Request", http.StatusBadRequest)
        return
    }

    purged := 0
    for _, key := range req.Keys {
        if n.cache.Purge(key) {
            purged++
        }
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]int{"purged": purged})
}

const adminToken = "secret-admin-token" // 生產環境從環境變數取
```

---

## 延伸問題

### Q: 如何處理快取雪崩（Cache Stampede）？

當某個熱點資源 TTL 同時過期，大量請求同時打到 Origin：

```go
// 解法：Singleflight — 相同 key 的並發請求只允許一個回源
import "golang.org/x/sync/singleflight"

var group singleflight.Group

func (n *EdgeNode) fetchWithSingleflight(ctx context.Context, key string) ([]byte, error) {
    result, err, _ := group.Do(key, func() (interface{}, error) {
        body, ct, ttl, err := n.fetchFromOrigin(ctx, key)
        if err != nil {
            return nil, err
        }
        n.cache.Set(key, body, ct, ttl)
        return body, nil
    })
    if err != nil {
        return nil, err
    }
    return result.([]byte), nil
}
```

### Q: 如何處理動態內容（個人化頁面）？

- **不快取**：帶 `Cache-Control: no-store`，CDN 直接穿透回源
- **Vary Header**：`Vary: Accept-Language` 讓 CDN 依語言分別快取
- **Edge Side Includes（ESI）**：頁面拆分靜態骨架（快取）+ 動態填充（不快取）
- **面試要點**：CDN 適合靜態資源，動態個人化不適合快取

### Q: 節點故障如何處理（HA）？

```
用戶 → PoP A（故障）→ Anycast 自動路由到 PoP B → 正常服務
                         （BGP 撤回故障節點路由，秒級切換）
```

健康檢查：每個節點向 L2/Origin 定期發 ping；Load Balancer 層做 Active Health Check。

### Q: HTTPS/TLS 在 CDN 的處理？

- CDN 在邊緣節點終止 TLS（TLS Termination），用戶到 PoP 是 HTTPS
- PoP 到 Origin 的內部鏈路也可用 HTTPS（依安全需求）
- 好處：邊緣節點距離用戶近，TLS 握手 RTT 大幅降低

---

## 相關題解

- [[sd-video-streaming|SD題解：影片串流]] — CDN 在影片中的核心角色（HLS 分片分發）
- [[sd-url-shortener|SD題解：URL Shortener]] — 架構圖中也用到 CDN 作為靜態資源加速
- [[sd-distributed-cache|SD題解：分散式快取]] — LRU 原理與 CDN L1 快取共用
- [[sd-web-crawler|SD題解：分散式網路爬蟲]] — 爬蟲如何繞過 CDN 抓到 Origin 真實內容
