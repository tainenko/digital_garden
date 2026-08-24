---
title: Go並發模式
type: concept
tags: [golang, concurrency, goroutine, channel, mutex, patterns, interview]
created: 2026-04-21
updated: 2026-04-21
sources: [golang-advanced-interview-secondtalent, golang-perf-advanced-codeforgeek]
---

# Go並發模式

Go 並發的核心哲學是「share memory by communicating」，但選擇 channel 還是 mutex 需要具體場景判斷。

---

## Channel vs Mutex 決策框架

| 場景 | 推薦 | 理由 |
|------|------|------|
| 跨 goroutine 所有權轉移 | Channel | 語意清晰，防止共享 |
| 簡單共享狀態（計數器）| `sync/atomic` | lock-free，ns 級操作 |
| 複雜多變量共享狀態 | `sync.Mutex` | 原子性保護多個欄位 |
| 讀多寫少 | `sync.RWMutex` | 允許並發讀 |
| CPU-intensive 並行計算 | Mutex/Atomic | Channel send 觸發 context switch |

```go
// ❌ 不必要的 channel（保護簡單計數器）
ch := make(chan int, 1)
ch <- count + 1

// ✅ atomic 更好
atomic.AddInt64(&count, 1)
```

---

## 六大並發模式

### 1. Worker Pool（控制並發上限）

```go
func worker(id int, jobs <-chan Job, results chan<- Result) {
    for j := range jobs {
        results <- process(j)
    }
}

jobs := make(chan Job, 100)
results := make(chan Result, 100)

for i := 0; i < workerCount; i++ {
    go worker(i, jobs, results)
}
```

用途：HTTP 請求處理、批次任務、防止無限制 goroutine 建立。

### 2. Fan-out / Fan-in

```go
// Fan-out: 一個 input → 多個 worker
func fanOut(in <-chan Work, n int) []<-chan Result {
    channels := make([]<-chan Result, n)
    for i := range channels {
        channels[i] = process(in)
    }
    return channels
}

// Fan-in: 多個 input → 一個 output
func fanIn(channels ...<-chan Result) <-chan Result {
    merged := make(chan Result)
    var wg sync.WaitGroup
    for _, ch := range channels {
        wg.Add(1)
        go func(c <-chan Result) {
            defer wg.Done()
            for v := range c { merged <- v }
        }(ch)
    }
    go func() { wg.Wait(); close(merged) }()
    return merged
}
```

### 3. Pipeline

```go
func gen(nums ...int) <-chan int { ... }
func sq(in <-chan int) <-chan int { ... }

// 組合
c := sq(sq(gen(2, 3, 4)))
```

### 4. Rate Limiting

```go
// 單機 Token Bucket
import "golang.org/x/time/rate"

limiter := rate.NewLimiter(rate.Every(time.Second/10), 1) // 10 req/s

if !limiter.Allow() {
    return errors.New("rate limited")
}

// 分散式：Redis SETNX + EXPIRE atomic ops
```

### 5. Context 取消樹

```go
ctx, cancel := context.WithTimeout(parentCtx, 5*time.Second)
defer cancel()

select {
case result := <-doWork(ctx):
    return result
case <-ctx.Done():
    return ctx.Err()
}
```

關鍵：cancel parent 自動 cascade 到所有 child context。

### 6. Graceful Shutdown

```go
quit := make(chan os.Signal, 1)
signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
<-quit

ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()
server.Shutdown(ctx) // 停止接受新請求，等現有請求完成
```

---

## Select 語意

```go
select {
case v := <-ch1:    // 收到 ch1
case ch2 <- x:      // 成功送出到 ch2
case <-time.After(1*time.Second): // timeout
default:            // non-blocking，立即回傳
}
```

多個 case 同時 ready 時：**隨機選擇**（防止飢餓）。

---

## 常見 Goroutine Leak

1. Channel 沒有關閉，receiver goroutine 永遠阻塞
2. Context 沒有傳入，goroutine 無法被取消
3. Panic 沒有 recover，整個程式崩潰

偵測：`pprof` goroutine profile 查看 blocked goroutines 數量趨勢。

---

## 相關頁面
- [[Go執行期內部機制]] — GMP scheduler、channel 的實際成本
- [[Go效能調優]] — lock contention profiling、worker pool 調優
- [[golang-principal-interview|Golang Principal Engineer 面試完整指南]]
