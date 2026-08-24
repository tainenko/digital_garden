---
title: Go效能調優
type: concept
tags: [golang, performance, pprof, optimization, production, GC-tuning, benchmark]
created: 2026-04-21
updated: 2026-04-21
sources: [go-production-perf-20tips, golang-perf-advanced-codeforgeek, golang-advanced-interview-secondtalent]
---

# Go效能調優

「任何沒有 data 支持的優化都是工程罪。」——先用 pprof 量測，再決定優化方向。

---

## 第一步：pprof 量測

```go
import (
    "net/http"
    _ "net/http/pprof"
)

func main() {
    go http.ListenAndServe(":6060", nil)
    // 正常服務啟動...
}
```

| Profile 類型 | 用途 |
|-------------|------|
| CPU | 找最熱的函式（100 samples/s） |
| Heap | 記憶體分配與 retention |
| Goroutine | 找 blocked/leaked goroutines |
| Mutex | Lock contention 定位 |
| Block | Sync primitive 阻塞分析 |

```bash
# 採樣 30 秒 CPU profile
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30

# 查看 heap
go tool pprof http://localhost:6060/debug/pprof/heap

# 生成 flame graph（需 graphviz）
(pprof) web
```

---

## 記憶體優化

### Pre-allocate（最高 ROI）

```go
// ❌ 觸發多次 reallocation
s := make([]int, 0)
for i := 0; i < 10000; i++ {
    s = append(s, i)
}

// ✅ 單次分配
s := make([]int, 0, 10000)
m := make(map[string]int, 10000)
```

### Sub-slice 記憶體洩漏

```go
// ❌ 整個 10MB array 無法被 GC
func bad(data []byte) []byte {
    return data[:10]
}

// ✅ 複製後原始 array 可被回收
func good(data []byte) []byte {
    result := make([]byte, 10)
    copy(result, data[:10])
    return result
}
```

### strings.Builder

```go
// ❌ 每次 + 產生新 string（immutable）
var s string
for _, part := range parts { s += part }

// ✅ 單一 []byte buffer，最後一次轉 string
var b strings.Builder
b.Grow(estimatedSize) // 可選：預分配
for _, part := range parts { b.WriteString(part) }
result := b.String()
```

---

## GC 調優

```bash
# 量測 GC 影響
GODEBUG=gctrace=1 ./myapp

# 調整觸發閾值
GOGC=200  # heap 長到 2x 才 GC（預設 1x）
```

典型決策：記憶體充足時調高 GOGC 換取更低 CPU 消耗。

---

## Benchmark 建立基準

```go
func BenchmarkStringBuilder(b *testing.B) {
    b.ReportAllocs() // 顯示每次 op 的分配次數和 bytes
    for i := 0; i < b.N; i++ {
        var sb strings.Builder
        for _, s := range data { sb.WriteString(s) }
        _ = sb.String()
    }
}
```

```bash
go test -bench=. -benchmem -count=5 | tee before.txt
# 優化後
go test -bench=. -benchmem -count=5 | tee after.txt
benchstat before.txt after.txt
```

---

## Lock Contention 優化

三層解決方案（依競爭強度選擇）：

```go
// 層 1：讀多寫少 → RWMutex
var mu sync.RWMutex
mu.RLock(); defer mu.RUnlock()  // 並發讀

// 層 2：單一數值 → atomic（lock-free，ns 級）
var counter int64
atomic.AddInt64(&counter, 1)

// 層 3：高競爭複雜狀態 → sharding
type ShardedMap struct {
    shards [256]struct {
        sync.RWMutex
        m map[string]any
    }
}
func (s *ShardedMap) shard(key string) *struct{ ... } {
    return &s.shards[fnv32(key)%256]
}
```

---

## DB 連線池調優

```go
db.SetMaxOpenConns(25)          // 最大開啟連線數
db.SetMaxIdleConns(25)          // 最大閒置連線數
db.SetConnMaxLifetime(5*time.Minute)  // 連線最長存活時間
```

---

## Profile-Guided Optimization (PGO，Go 1.21+)

```bash
# 1. 從 production 收集 CPU profile
curl -o cpu.pprof "http://prod:6060/debug/pprof/profile?seconds=30"

# 2. 用 profile 重新編譯
go build -pgo=cpu.pprof ./...
```

效果：hot path inlining 優化，通常帶來 2-7% 效能提升。

---

## 容器環境必做

```go
import _ "go.uber.org/automaxprocs"
// 自動根據 cgroup CPU quota 設定 GOMAXPROCS，而非讀取 node 總 CPU 數
```

---

## 相關頁面
- [[Go執行期內部機制]] — GC 機制、escape analysis、GOGC 原理
- [[Go並發模式]] — Worker pool、buffered channel 設計
- [[golang-principal-interview|Golang Principal Engineer 面試完整指南]]
