---
title: 廣告點擊統計系統（Ad Click Aggregator）
type: topic
tags: [system-design, interview, analytics, streaming, aggregation]
created: 2026-04-20
updated: 2026-04-20
---

# 廣告點擊統計系統（Ad Click Aggregator）

難度：進階｜核心技術：串流處理、時間窗口聚合、Lambda Architecture、冪等性

---

## RESHADED 分析

### Requirements
**Functional:**
- 記錄每一次廣告點擊（click event）
- 即時查詢：過去 M 分鐘內特定廣告的點擊數
- 聚合查詢：按廣告ID / 地區 / 裝置類型分組統計
- 支援「Top N 廣告」查詢（過去 1 小時點擊最多的廣告）
- 資料保留：原始事件 90 天，聚合結果 1 年

**Non-functional:**
- 超高寫入吞吐量：100 萬次點擊/秒
- 查詢延遲：＜1 秒
- 統計準確性：允許 0.1% 誤差（近似算法）
- 冪等性：網路重試不重複計算

### Estimates
```
點擊量: 100萬次/秒 = 864億次/天
原始事件儲存（90天）：
  每次事件 100 bytes → 864億 × 100B × 90 = 777 TB
聚合後儲存（每分鐘聚合）：
  廣告ID(1億) × 每分鐘 × 1440 分 × 365 天 ≈ 可接受
```

### High-Level Design

```
用戶點擊 → Click API (無狀態)
    ↓ 批次寫入
Kafka (raw_clicks topic)
    ├─ 串流處理層 (Flink / Spark Streaming)
    │    ├─ 1分鐘 Tumbling Window 聚合
    │    ├─ 冪等性去重
    │    └─ 寫入 OLAP DB (ClickHouse / Druid)
    │
    └─ 批次處理層 (Hadoop / Spark Batch)
         ├─ 每日重新計算（修正延遲資料）
         └─ 寫入 Data Warehouse

查詢層：
  即時查詢 → OLAP DB
  歷史報表 → Data Warehouse
```

### API Design
```go
// 記錄點擊
POST /v1/clicks
{
  "ad_id":    "ad_123",
  "user_id":  "usr_456",
  "timestamp": 1704067200,
  "source_ip": "1.2.3.4",
  "region":   "TW",
  "device":   "mobile"
}
// 回應: 202 Accepted（非同步處理）

// 查詢點擊數
GET /v1/stats/clicks?ad_id=ad_123&window=60&granularity=minute
// 回應:
{
  "ad_id": "ad_123",
  "total": 12345,
  "time_series": [
    {"timestamp": 1704067200, "count": 200},
    {"timestamp": 1704067260, "count": 185}
  ]
}

// Top N 查詢
GET /v1/stats/top-ads?limit=10&window=3600
```

---

## 核心技術深探

### 1. 點擊事件寫入（高吞吐量）

```go
type ClickEvent struct {
    AdID      string    `json:"ad_id"`
    UserID    string    `json:"user_id"`
    Timestamp int64     `json:"timestamp"`
    Region    string    `json:"region"`
    Device    string    `json:"device"`
    ClickID   string    `json:"click_id"`  // 冪等性 Key
}

type ClickHandler struct {
    producer kafka.Producer
    // 本地緩衝區，批次寫入 Kafka
    buffer   []ClickEvent
    mu       sync.Mutex
    ticker   *time.Ticker
}

func (h *ClickHandler) RecordClick(event ClickEvent) {
    // 生成冪等性 key（防止重複點擊計算）
    event.ClickID = generateClickID(event.UserID, event.AdID, event.Timestamp)

    h.mu.Lock()
    h.buffer = append(h.buffer, event)
    h.mu.Unlock()
}

// 每 100ms 批次發送
func (h *ClickHandler) flush() {
    h.mu.Lock()
    batch := h.buffer
    h.buffer = nil
    h.mu.Unlock()

    if len(batch) == 0 {
        return
    }

    msgs := make([]kafka.Message, 0, len(batch))
    for _, e := range batch {
        b, _ := json.Marshal(e)
        msgs = append(msgs, kafka.Message{
            Key:   []byte(e.AdID),  // 同一廣告的事件發到同一 partition
            Value: b,
        })
    }
    h.producer.WriteMessages(context.Background(), msgs...)
}
```

### 2. 串流聚合（Flink 邏輯的 Go 模擬）

```go
// 時間窗口聚合器（1分鐘 Tumbling Window）
type WindowAggregator struct {
    windows map[string]*WindowBucket  // key: "ad_id:window_start"
    mu      sync.RWMutex
    output  chan AggregationResult
}

type WindowBucket struct {
    AdID        string
    WindowStart int64
    Count       int64
    Regions     map[string]int64
    seen        *BloomFilter  // 冪等性去重
}

func (a *WindowAggregator) Process(event ClickEvent) {
    windowStart := event.Timestamp - (event.Timestamp % 60)  // 對齊到分鐘
    key := fmt.Sprintf("%s:%d", event.AdID, windowStart)

    a.mu.Lock()
    defer a.mu.Unlock()

    bucket, ok := a.windows[key]
    if !ok {
        bucket = &WindowBucket{
            AdID:        event.AdID,
            WindowStart: windowStart,
            Regions:     make(map[string]int64),
            seen:        NewBloomFilter(100000, 0.01),
        }
        a.windows[key] = bucket
    }

    // 冪等性去重
    if bucket.seen.Test([]byte(event.ClickID)) {
        return
    }
    bucket.seen.Add([]byte(event.ClickID))

    bucket.Count++
    bucket.Regions[event.Region]++
}

// 每分鐘輸出已完成的窗口
func (a *WindowAggregator) tick() {
    now := time.Now().Unix()
    cutoff := now - 120  // 延遲 2 分鐘確保遲到事件

    a.mu.Lock()
    defer a.mu.Unlock()

    for key, bucket := range a.windows {
        if bucket.WindowStart < cutoff {
            a.output <- AggregationResult{
                AdID:        bucket.AdID,
                WindowStart: bucket.WindowStart,
                Count:       bucket.Count,
                Regions:     bucket.Regions,
            }
            delete(a.windows, key)
        }
    }
}
```

### 3. Top N 廣告（Count-Min Sketch + Heap）

精確 Top N 在大量廣告中代價很高，使用近似算法：

```go
// Count-Min Sketch：用於近似頻率統計
type CountMinSketch struct {
    table  [][]int64
    hashes []func([]byte) uint32
    width  uint32
    depth  int
}

func NewCountMinSketch(width uint32, depth int) *CountMinSketch {
    table := make([][]int64, depth)
    hashes := make([]func([]byte) uint32, depth)
    for i := 0; i < depth; i++ {
        table[i] = make([]int64, width)
        seed := uint32(i * 2654435761)
        hashes[i] = func(data []byte) uint32 {
            h := seed
            for _, b := range data {
                h ^= uint32(b)
                h *= 16777619
            }
            return h
        }
    }
    return &CountMinSketch{table: table, hashes: hashes, width: width, depth: depth}
}

func (s *CountMinSketch) Increment(key string) {
    data := []byte(key)
    for i, h := range s.hashes {
        col := h(data) % s.width
        s.table[i][col]++
    }
}

func (s *CountMinSketch) Estimate(key string) int64 {
    data := []byte(key)
    min := int64(math.MaxInt64)
    for i, h := range s.hashes {
        col := h(data) % s.width
        if s.table[i][col] < min {
            min = s.table[i][col]
        }
    }
    return min
}

// Top N 維護（小頂堆）
type TopNTracker struct {
    sketch *CountMinSketch
    heap   *minHeap  // 大小為 N 的最小堆
    mu     sync.RWMutex
}

func (t *TopNTracker) Update(adID string) {
    t.mu.Lock()
    defer t.mu.Unlock()

    t.sketch.Increment(adID)
    count := t.sketch.Estimate(adID)

    if t.heap.Len() < t.heap.capacity || count > t.heap.Min() {
        t.heap.PushOrUpdate(adID, count)
    }
}

func (t *TopNTracker) GetTopN() []AdCount {
    t.mu.RLock()
    defer t.mu.RUnlock()
    return t.heap.Sorted()
}
```

### 4. Lambda Architecture（批次修正）

串流處理可能有誤差（遲到事件、機器故障），用批次處理重算修正：

```
每日凌晨 2:00：
  Spark Job 讀取 Kafka（保留 7 天原始事件）
  → 重新計算昨日所有聚合結果
  → 覆蓋寫入 OLAP DB
  → 確保歷史資料準確性
```

---

## Trade-offs

| 決策 | 選擇 | 理由 |
|------|------|------|
| 即時性 vs 準確性 | **先即時，每日批次修正** | Lambda Architecture |
| 精確計數 vs 近似 | **Count-Min Sketch** | Top N 允許 0.1% 誤差，節省大量記憶體 |
| 寫入模式 | **Kafka 緩衝批次** | 應對 100萬/秒 峰值 |
| 冪等性 | **ClickID + Bloom Filter** | 快速去重，誤判率可控 |
| 儲存 | **ClickHouse** | 列存儲，聚合查詢性能佳 |

---

## 相關頁面

- [[sd-url-shortener]] — 高吞吐量寫入類似問題
- [[系統設計核心技術棧]] — Kafka 使用指南
- [[sd-typeahead]] — Bloom Filter 相關應用
