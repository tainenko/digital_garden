---
title: Go 記憶體洩漏排查與預防
type: concept
tags: [golang, memory-leak, goroutine-leak, pprof, goleak, senior]
created: 2026-04-29
updated: 2026-04-29
---

# Go 記憶體洩漏排查與預防

## Go 的記憶體洩漏不同於 C/C++

C/C++ 是忘記 free 導致洩漏。Go 有 GC，但仍然可能「邏輯洩漏」：

**物件仍被根物件（root）直接或間接引用，GC 不會回收。**

常見的洩漏根源：
1. **Goroutine 洩漏**（最常見）— goroutine 永遠阻塞，其持有的變數無法回收
2. **無界快取/全域 map** — 持續 append/insert，沒有驅逐機制
3. **未關閉的資源** — http.Response.Body、DB rows、File
4. **time.Ticker/time.Timer** — 忘記 Stop()
5. **Finalizer 誤用** — runtime.SetFinalizer 的迴圈引用

---

## 一、Goroutine 洩漏（最常見）

### 典型場景

```go
// ❌ 場景 1：channel 沒有人讀，goroutine 永遠阻塞
func processRequests(requests []Request) {
    results := make(chan Result) // 無緩衝
    for _, req := range requests {
        go func(r Request) {
            results <- process(r) // 如果沒有人讀 results，這行永遠阻塞
        }(req)
    }
    // 函數提前返回，results 被丟棄，goroutine 永遠阻塞 ← 洩漏
}

// ✅ 修復：確保有人讀 channel，或用 select + context 退出
func processRequests(ctx context.Context, requests []Request) []Result {
    results := make(chan Result, len(requests)) // 有緩衝，不會阻塞
    for _, req := range requests {
        req := req
        go func() {
            select {
            case <-ctx.Done():
                return // 父取消，退出
            case results <- process(req):
            }
        }()
    }
    var out []Result
    for range requests {
        out = append(out, <-results)
    }
    return out
}
```

```go
// ❌ 場景 2：for-select 沒有退出條件
func backgroundWorker(data <-chan Item) {
    go func() {
        for {
            select {
            case item := <-data:
                process(item)
            // 沒有 case <-ctx.Done()，這個 goroutine 永遠跑
            }
        }
    }()
}

// ✅ 修復：加入 done channel 或 context
func backgroundWorker(ctx context.Context, data <-chan Item) {
    go func() {
        for {
            select {
            case <-ctx.Done():
                return
            case item, ok := <-data:
                if !ok {
                    return // channel 關閉，退出
                }
                process(item)
            }
        }
    }()
}
```

```go
// ❌ 場景 3：http handler 裡啟動 goroutine，但沒有 context 退出
func handler(w http.ResponseWriter, r *http.Request) {
    go func() {
        // r.Context() 在請求結束後已取消，但 goroutine 仍在跑
        time.Sleep(10 * time.Second)
        log.Println("done")
    }()
    w.Write([]byte("ok"))
}

// ✅ 修復：把 request context 傳入 goroutine
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    go func() {
        select {
        case <-ctx.Done():
            return
        case <-time.After(10 * time.Second):
            log.Println("done")
        }
    }()
    w.Write([]byte("ok"))
}
```

### 偵測 Goroutine 洩漏

```bash
# 方法 1：pprof endpoint 觀察 goroutine 數量
watch -n 5 "curl -s localhost:6060/debug/pprof/goroutine?debug=1 | grep '^goroutine' | wc -l"
# 如果數量持續增長 → 洩漏

# 方法 2：dump 所有 goroutine stack
curl localhost:6060/debug/pprof/goroutine?debug=2 > goroutine_dump.txt
# 找重複出現最多次的 stack → 那是洩漏的 goroutine
```

```go
// 方法 3：用 goleak 在測試中檢測
import "go.uber.org/goleak"

func TestMyFunction(t *testing.T) {
    defer goleak.VerifyNone(t) // 測試結束後，檢查是否有 goroutine 洩漏

    myFunction() // 如果這個函數洩漏了 goroutine，測試會失敗
}

// 或在 TestMain 中統一
func TestMain(m *testing.M) {
    goleak.VerifyTestMain(m)
}
```

---

## 二、無界快取/全域 Map

```go
// ❌ 快取沒有大小限制，持續增長
var cache = make(map[string][]byte)
var mu sync.Mutex

func getFromCache(key string) []byte {
    mu.Lock()
    defer mu.Unlock()
    if v, ok := cache[key]; ok {
        return v
    }
    v := fetchFromDB(key)
    cache[key] = v // 只進不出，記憶體持續增長
    return v
}

// ✅ 修復：使用有大小限制的 LRU cache
import lru "github.com/hashicorp/golang-lru/v2"

var cache, _ = lru.New[string, []byte](10000) // 最多 10000 個 entry，超出自動逐出

func getFromCache(key string) []byte {
    if v, ok := cache.Get(key); ok {
        return v
    }
    v := fetchFromDB(key)
    cache.Add(key, v)
    return v
}
```

---

## 三、未關閉的資源

```go
// ❌ http.Response.Body 沒有 Close（非常常見！）
func fetch(url string) ([]byte, error) {
    resp, err := http.Get(url)
    if err != nil {
        return nil, err
    }
    // 沒有 defer resp.Body.Close()
    // → HTTP 連線無法放回連線池，連線洩漏
    return io.ReadAll(resp.Body)
}

// ✅ 修復
func fetch(url string) ([]byte, error) {
    resp, err := http.Get(url)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close() // 必須

    return io.ReadAll(resp.Body)
}
```

```go
// ❌ DB Rows 沒有 Close
func queryUsers(db *sql.DB) {
    rows, _ := db.Query("SELECT id, name FROM users")
    // 沒有 defer rows.Close()
    // → 資料庫連線沒有釋放
    for rows.Next() {
        // ...
    }
}

// ✅ 修復
func queryUsers(db *sql.DB) error {
    rows, err := db.Query("SELECT id, name FROM users")
    if err != nil {
        return err
    }
    defer rows.Close() // 必須

    for rows.Next() {
        // ...
    }
    return rows.Err()
}
```

```go
// ❌ time.Ticker 沒有 Stop（goroutine 洩漏）
func pollEvery5s() {
    ticker := time.NewTicker(5 * time.Second)
    go func() {
        for range ticker.C {
            doWork()
        }
        // 沒有人呼叫 ticker.Stop()，內部 goroutine 永遠跑
    }()
}

// ✅ 修復
func pollEvery5s(ctx context.Context) {
    ticker := time.NewTicker(5 * time.Second)
    go func() {
        defer ticker.Stop()
        for {
            select {
            case <-ctx.Done():
                return
            case <-ticker.C:
                doWork()
            }
        }
    }()
}
```

---

## 四、Slice 持有大 Array 的引用

```go
// ❌ 子 slice 持有整個底層 array，大 array 無法被 GC
func getFirstFive(data []byte) []byte {
    return data[:5] // 這個 slice 仍然持有整個 data 的底層 array！
}

func processLargeFile(filename string) []byte {
    data, _ := os.ReadFile(filename) // 讀取 100MB 的檔案
    header := getFirstFive(data)     // 只需要前 5 bytes
    return header
    // data 被 GC 回收，但底層 array（100MB）仍被 header 引用！
}

// ✅ 修復：複製需要的部分
func getFirstFive(data []byte) []byte {
    result := make([]byte, 5)
    copy(result, data[:5]) // 建立新的底層 array，只有 5 bytes
    return result
}
```

---

## 五、String 轉換後的引用

```go
// 背景知識：string 和 []byte 的轉換通常涉及記憶體複製
// 但某些情況下 Go 優化成共用記憶體，導致意外 retain

// ❌ map key 從大 byte slice 轉換而來，retain 了底層 array
func processChunk(chunk []byte) {
    key := string(chunk[:10]) // 如果這裡被優化成不複製...
    globalMap[key] = true     // key 持有 chunk 的底層 array！
}

// ✅ 確保 string 是獨立的（strings.Clone Go 1.20+）
func processChunk(chunk []byte) {
    key := strings.Clone(string(chunk[:10])) // 強制複製，確保獨立
    globalMap[key] = true
}
```

---

## 系統性排查流程

```
Step 1：確認洩漏
  → 觀察 /debug/pprof/heap（inuse_space 是否持續增長）
  → 觀察 goroutine 數量是否持續增長

Step 2：定位
  → heap 洩漏：比較兩份 heap profile（-base）
  → goroutine 洩漏：dump goroutine stacks，找重複最多的

Step 3：修復（優先順序）
  → goroutine 洩漏：加 context 退出，關閉 channel
  → 資源洩漏：補 defer Close()
  → 快取洩漏：加 TTL 或大小限制
  → slice 洩漏：copy 而非 sub-slice

Step 4：驗證
  → goleak 加入測試
  → 壓測後觀察記憶體是否穩定
```

## 常用指令速查

```bash
# 監控 goroutine 數量（每 5 秒刷新）
watch -n 5 "curl -s localhost:6060/debug/pprof/goroutine?debug=1 | head -5"

# 比較記憶體增長
curl -s localhost:6060/debug/pprof/heap -o t1.prof && sleep 60 && \
curl -s localhost:6060/debug/pprof/heap -o t2.prof && \
go tool pprof -base t1.prof -inuse_space t2.prof

# 找開啟的 fd（file descriptor 洩漏）
ls -la /proc/$(pgrep myservice)/fd | wc -l
```

## 相關頁面

- [[Go pprof實戰完整指南]] — profile 的採集和解讀
- [[Go Context深度解析]] — context 取消防止 goroutine 洩漏
- [[Go並發模式]] — 正確的 goroutine 生命週期管理
- [[Go效能調優]] — sync.Pool 減少 allocation 壓力
