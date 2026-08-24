---
title: Go pprof 實戰完整指南
type: concept
tags: [golang, pprof, profiling, performance, cpu, memory, senior]
created: 2026-04-29
updated: 2026-04-29
---

# Go pprof 實戰完整指南

> pprof 是 Go 內建的效能分析工具。原則：**先量測，後優化。** 沒有 data 支持的優化都是猜測。

## 六種 Profile 類型

| Profile | 分析對象 | 何時用 |
|---------|---------|-------|
| `cpu` | 哪些函數佔用最多 CPU 時間 | 服務響應慢、CPU 使用率高 |
| `heap` | 記憶體分配量與 retain 量 | 記憶體佔用高、GC 頻繁 |
| `goroutine` | 所有 goroutine 的當前 stack | goroutine 數量異常增長 |
| `allocs` | 所有記憶體分配（含已回收）| 找 GC 壓力來源 |
| `mutex` | mutex lock contention | CPU 高但 throughput 低 |
| `block` | goroutine 阻塞時間 | 找 channel/syscall 瓶頸 |
| `threadcreate` | OS thread 建立 | 找 CGo 或 OS 呼叫問題 |

## 開啟 pprof Endpoint（HTTP Server）

```go
import (
    "net/http"
    _ "net/http/pprof" // 只需 import，自動注冊 /debug/pprof/* 路由
)

func main() {
    // 在獨立的 port 上跑 pprof server（不要暴露到公網！）
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()

    // 開啟 mutex 和 block profiling（預設關閉）
    runtime.SetMutexProfileFraction(1)  // 1 = 100% 採樣
    runtime.SetBlockProfileRate(1)      // 1 = 所有 block 事件
    // 注意：生產環境採樣率設低，高採樣率有效能影響

    // 你的服務正常啟動
    startMyServer()
}
```

**已自動注冊的路由**：
```
GET /debug/pprof/          # 總覽頁
GET /debug/pprof/cmdline   # 命令列
GET /debug/pprof/profile   # CPU profile（預設 30s）
GET /debug/pprof/heap      # Heap profile
GET /debug/pprof/goroutine # Goroutine stacks
GET /debug/pprof/allocs    # Alloc profile
GET /debug/pprof/mutex     # Mutex profile
GET /debug/pprof/block     # Block profile
```

## CPU Profiling 實戰

### 採集

```bash
# 採集 30 秒 CPU profile，存成 cpu.prof
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30

# 或先下載，再分析（不阻塞互動）
curl -s "localhost:6060/debug/pprof/profile?seconds=30" -o cpu.prof
go tool pprof cpu.prof
```

### 解讀

```
# pprof 互動命令
(pprof) top10          # 列出最熱的 10 個函數
(pprof) top10 -cum     # 按累計時間排序（包含子函數）
(pprof) list funcname  # 顯示函數的每行時間
(pprof) web            # 在瀏覽器開啟火焰圖（需要 graphviz）
(pprof) svg            # 輸出 SVG 圖
```

```
top10 輸出解讀：
      flat  flat%   sum%        cum   cum%
     300ms 30.00% 30.00%      300ms 30.00%  runtime.mallocgc
     200ms 20.00% 50.00%      500ms 50.00%  encoding/json.Marshal
     100ms 10.00% 60.00%      800ms 80.00%  myapp.processRequest

flat  = 這個函數本身佔用的時間（不包含呼叫的子函數）
cum   = 累計時間（包含子函數）
flat 高 → 這個函數本身是熱點
cum 高但 flat 低 → 是呼叫鏈的上層，問題在它呼叫的子函數
```

### 火焰圖（Flame Graph）

```bash
# 用 go tool pprof 的 web 命令
go tool pprof -http=:8081 cpu.prof
# 在瀏覽器開 localhost:8081，點 Flame Graph
```

**火焰圖閱讀技巧**：
- 橫軸 = CPU 時間比例（越寬越熱）
- 縱軸 = 呼叫堆疊深度（頂部 = 葉節點，即真正執行的函數）
- **找最寬的葉節點**，那是真正的熱點
- 高聳的塔（大 cum，小 flat）= 呼叫鏈，問題在更底層

## Heap Profiling 實戰

```bash
# 下載 heap profile（inuse_space = 當前使用中的記憶體）
go tool pprof http://localhost:6060/debug/pprof/heap

# 查看分配量（alloc_space = 歷史所有分配，含已釋放）
go tool pprof -alloc_space http://localhost:6060/debug/pprof/heap
```

```
# Heap profile 命令
(pprof) top10              # 按 inuse_space 排序
(pprof) top10 -inuse_objects  # 按物件數量排序
(pprof) top10 -alloc_space    # 按分配總量排序

# 解讀範例
Showing nodes accounting for 45.23MB, 78.43% of 57.67MB total
      flat  flat%   sum%        cum   cum%
   25.00MB 43.35% 43.35%    25.00MB 43.35%  bytes.makeSlice
   10.00MB 17.34% 60.69%    35.00MB 60.69%  encoding/json.Unmarshal
    5.00MB  8.67% 69.36%     5.00MB  8.67%  myapp.parseResponse

→ json.Unmarshal 消耗最多記憶體，考慮用 json.Decoder streaming 或 gjson
```

### 比較兩份 Heap Profile（找記憶體增長）

```bash
# 採集 t1
curl -s localhost:6060/debug/pprof/heap -o heap1.prof

# 等一段時間後採集 t2
curl -s localhost:6060/debug/pprof/heap -o heap2.prof

# 比較差值（只顯示增長的部分）
go tool pprof -base heap1.prof heap2.prof
(pprof) top10  # 哪些函數造成記憶體增長
```

## Goroutine Profiling（找 goroutine 洩漏）

```bash
go tool pprof http://localhost:6060/debug/pprof/goroutine
```

```
(pprof) top10       # 哪些 goroutine 數量最多
(pprof) traces      # 列出所有 goroutine 的 stack trace
(pprof) list funcname  # 某個函數建立的 goroutine

# 快速檢查：直接看文字
curl localhost:6060/debug/pprof/goroutine?debug=1 | head -50

# 更詳細的 stack（debug=2）
curl localhost:6060/debug/pprof/goroutine?debug=2 > goroutine.txt
```

**洩漏的特徵**：goroutine 數量隨時間線性增長，且 stack trace 指向同一個函數。

## 在單元測試中 Profiling

```go
func BenchmarkProcessOrder(b *testing.B) {
    // 開啟 CPU profiling（-cpuprofile 旗標）
    // 直接在 test 裡也可以手動觸發
    f, _ := os.Create("cpu.prof")
    pprof.StartCPUProfile(f)
    defer pprof.StopCPUProfile()

    for i := 0; i < b.N; i++ {
        processOrder(testOrder)
    }
}
```

```bash
# 跑 benchmark 並自動生成 profile
go test -bench=BenchmarkProcessOrder -cpuprofile=cpu.prof -memprofile=mem.prof ./...

# 分析
go tool pprof cpu.prof
go tool pprof mem.prof
```

## 生產環境持續 Profiling

每次手動採集很麻煩，可以用持續 profiling（Continuous Profiling）工具：

```go
// Pyroscope（開源，支援 Go）
import "github.com/grafana/pyroscope-go"

func main() {
    pyroscope.Start(pyroscope.Config{
        ApplicationName: "order-service",
        ServerAddress:   "http://pyroscope:4040",
        ProfileTypes: []pyroscope.ProfileType{
            pyroscope.ProfileCPU,
            pyroscope.ProfileAllocObjects,
            pyroscope.ProfileAllocSpace,
            pyroscope.ProfileInuseObjects,
            pyroscope.ProfileInuseSpace,
        },
    })
    // 自動在後台採集並上傳到 Pyroscope Server
}
```

## 實戰診斷流程

### 情況 1：CPU 使用率異常高

```bash
# 1. 採集 CPU profile
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30

# 2. 看 top10 + flame graph
(pprof) top10 -cum
(pprof) web  # 開啟火焰圖

# 常見原因：
# - JSON/Protobuf 序列化（考慮換 sonic/jsoniter 或 gob）
# - regexp.MustCompile 在 hot path 裡（應該預先編譯）
# - 過多的 fmt.Sprintf（考慮 strconv.Itoa）
# - sync.Mutex 競爭（看 mutex profile）
# - GC 壓力（inuse memory 高 → 看 heap profile）
```

### 情況 2：記憶體持續增長（疑似洩漏）

```bash
# 1. 採集兩個時間點的 heap profile
curl -s localhost:6060/debug/pprof/heap -o t1.prof
sleep 60
curl -s localhost:6060/debug/pprof/heap -o t2.prof

# 2. 比較差值
go tool pprof -base t1.prof t2.prof
(pprof) top10

# 3. 同時看 goroutine 數量
curl localhost:6060/debug/pprof/goroutine?debug=1 | grep "^goroutine" | wc -l

# 常見原因：
# - goroutine 洩漏（channel 沒人讀，goroutine 永遠阻塞）
# - slice/map 持續 append 但不釋放
# - http.Response.Body 沒 Close
# - cache 沒有大小限制
```

### 情況 3：Throughput 低但 CPU 不高

```bash
# 懷疑 lock contention → 看 mutex profile
go tool pprof http://localhost:6060/debug/pprof/mutex

# 懷疑 I/O 等待 → 看 block profile
go tool pprof http://localhost:6060/debug/pprof/block

# 常見原因：
# - sync.Mutex 被大量並發爭用（考慮 shard 或換 sync.Map / atomic）
# - channel 滿了，sender 阻塞（增大 buffer 或增加 consumer）
# - 資料庫連線池耗盡（增大 MaxOpenConns）
```

## `runtime.ReadMemStats` 即時監控

```go
func reportMemStats() {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)

    log.Printf("Alloc=%vMB TotalAlloc=%vMB Sys=%vMB NumGC=%d GCPause=%vms",
        m.Alloc/1024/1024,
        m.TotalAlloc/1024/1024,
        m.Sys/1024/1024,
        m.NumGC,
        float64(m.PauseNs[(m.NumGC+255)%256])/1e6,
    )
}

// 重點指標：
// Alloc       = 當前 heap 使用量
// TotalAlloc  = 歷史累計分配量（只增不減，用來算分配速率）
// Sys         = 向 OS 申請的總記憶體
// NumGC       = GC 次數
// PauseNs     = 最近 256 次 GC 的 stop-the-world 時間（環形陣列）
// HeapInuse   = heap 中真正使用的 span 大小
// HeapReleased = 已歸還給 OS 的記憶體
```

## 相關頁面

- [[Go效能調優]] — 找到熱點後的優化技術
- [[Go記憶體洩漏排查]] — pprof 診斷後的修復方式
- [[Go執行期內部機制]] — GC 和 escape analysis 的底層機制
- [[Go測試基準與模糊測試]] — Benchmark + pprof 的結合
