---
title: Go 同步原語與記憶體模型
type: concept
tags: [golang, sync, atomic, memory-model, mutex, channel, senior]
created: 2026-04-29
updated: 2026-04-29
---

# Go 同步原語與記憶體模型

## Go 記憶體模型（Memory Model）

Go 記憶體模型定義：**在什麼條件下，goroutine A 對變數的寫入，對 goroutine B 的讀取可見。**

### Happens-Before（先行發生）

```
如果事件 A happens-before 事件 B：
A 的所有記憶體寫入，對 B 執行時都是可見的。
```

**建立 happens-before 的方式**：

| 操作 | 說明 |
|------|------|
| channel send | `ch <- v` happens-before 對應的 `<-ch` |
| channel close | `close(ch)` happens-before 接收到 zero value |
| sync.Mutex Lock/Unlock | 第 n 次 Unlock happens-before 第 n+1 次 Lock |
| sync.WaitGroup Done | `wg.Done()` happens-before `wg.Wait()` 返回 |
| sync.Once | `once.Do(f)` 中的 f 返回 happens-before 任何 `once.Do` 返回 |
| atomic 操作 | 原子寫 happens-before 之後的原子讀（同一個變數）|

```go
// ❌ 數據競態：沒有 happens-before 保證
var x int
go func() { x = 1 }() // goroutine A 寫
fmt.Println(x)         // goroutine B 讀 — 不保證能看到 x=1

// ✅ 透過 channel 建立 happens-before
ch := make(chan struct{})
go func() {
    x = 1
    ch <- struct{}{} // x=1 的寫入 happens-before 這個 send
}()
<-ch         // 這個 receive happens-after send
fmt.Println(x) // 保證看到 x=1
```

### 數據競態偵測

```bash
# 加 -race 旗標，runtime 會偵測數據競態
go test -race ./...
go run -race main.go

# 輸出範例：
# WARNING: DATA RACE
# Write at 0x00c0000b6008 by goroutine 7:
#   main.(*Counter).Inc()
# Previous read at 0x00c0000b6008 by goroutine 6:
#   main.(*Counter).Value()
```

---

## sync.Mutex / sync.RWMutex

```go
// Mutex：排他鎖
type Counter struct {
    mu    sync.Mutex
    count int
}

func (c *Counter) Inc() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func (c *Counter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}

// RWMutex：讀多寫少時的優化
// RLock 允許多個 goroutine 同時持有，Lock 排他
type Cache struct {
    mu   sync.RWMutex
    data map[string][]byte
}

func (c *Cache) Get(key string) ([]byte, bool) {
    c.mu.RLock()         // 多個 reader 可以同時讀
    defer c.mu.RUnlock()
    v, ok := c.data[key]
    return v, ok
}

func (c *Cache) Set(key string, value []byte) {
    c.mu.Lock()         // writer 獨佔
    defer c.mu.Unlock()
    c.data[key] = value
}

// ⚠️ 注意：Mutex 是不可重入的
// 如果同一個 goroutine 兩次 Lock() → 死鎖！
func (c *Counter) DoubleInc() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.Inc() // ← 這裡會嘗試再次 Lock → 死鎖！
    // 修法：把業務邏輯拆成帶鎖和不帶鎖兩個版本
}
```

---

## sync/atomic（無鎖操作）

```go
import "sync/atomic"

// 整數計數器（最常見）
var count atomic.Int64

count.Add(1)
v := count.Load()
count.Store(100)
old := count.Swap(200) // 寫入並返回舊值
ok := count.CompareAndSwap(200, 300) // CAS：若當前值=200，設成300

// atomic.Value：任意型別的原子讀寫
// 用途：原子替換 config、cache 等大型物件
var config atomic.Value

func updateConfig(newCfg *Config) {
    config.Store(newCfg) // 原子替換整個 config
}

func getConfig() *Config {
    return config.Load().(*Config) // 取得當前 config（原子讀）
}

// atomic.Bool（Go 1.19+）
var isReady atomic.Bool
isReady.Store(true)
if isReady.Load() { ... }

// atomic.Pointer（Go 1.19+，型別安全）
var latest atomic.Pointer[Config]
latest.Store(newCfg)
cfg := latest.Load() // 型別安全，不需要斷言
```

### Mutex vs Atomic 選擇

```go
// 規則：
// 1. 單一整數/布林的讀寫 → atomic（最快）
// 2. 多個欄位需要一起更新（原子性）→ Mutex
// 3. 大型物件（struct/slice/map）整體替換 → atomic.Value
// 4. 讀多寫少的複雜結構 → sync.RWMutex

// 效能對比（大約）：
// atomic.Add  ≈ 10ns
// sync.Mutex  ≈ 20-40ns（無競爭）
// sync.Mutex  ≈ 100-1000ns（高競爭）
```

---

## sync.Map

適合**讀多寫少**且 key 集合相對穩定的情況：

```go
var m sync.Map

// Store
m.Store("key", "value")

// Load
if v, ok := m.Load("key"); ok {
    fmt.Println(v.(string))
}

// LoadOrStore（原子：不存在才設置）
actual, loaded := m.LoadOrStore("key", "default")
// loaded = true 表示已存在，actual 是舊值
// loaded = false 表示剛被設置，actual 是 "default"

// LoadAndDelete
v, loaded := m.LoadAndDelete("key")

// Range（遍歷，但不保證一致性快照）
m.Range(func(key, value any) bool {
    fmt.Println(key, value)
    return true // 返回 false 停止遍歷
})
```

**sync.Map vs map+RWMutex**：
- sync.Map：讀操作大多無鎖（dirty cache），適合讀遠多於寫，key 不常變
- map+RWMutex：寫入頻繁或 key 集合頻繁變化時更好
- sync.Map 的 Range 不是原子快照，遍歷過程中可能看到不一致狀態

---

## sync.Once

```go
// 保證 f 只被執行一次，即使有多個 goroutine 同時呼叫
var once sync.Once
var instance *DB

func GetDB() *DB {
    once.Do(func() {
        instance = newDB() // 只執行一次
    })
    return instance
}

// ⚠️ 注意：如果 f 發生 panic，once 仍然標記為「已執行」
// 之後的呼叫不會再執行 f，也不會 panic
// → 如果初始化可能失敗，不要用 sync.Once，改用 sync.OnceValue（Go 1.21+）

// sync.OnceValue（Go 1.21+）：帶返回值的 Once
var getDB = sync.OnceValue(func() *DB {
    db, err := sql.Open("pgx", dsn)
    if err != nil {
        panic(err) // 初始化失敗時 panic
    }
    return db
})

// 使用
db := getDB() // 第一次呼叫執行初始化，之後直接返回結果
```

---

## sync.WaitGroup

```go
var wg sync.WaitGroup

for i := 0; i < 10; i++ {
    wg.Add(1)
    go func(n int) {
        defer wg.Done()
        process(n)
    }(i)
}
wg.Wait() // 等所有 goroutine 完成

// ⚠️ 常見錯誤 1：在 goroutine 內部 Add
for i := 0; i < 10; i++ {
    go func(n int) {
        wg.Add(1) // ❌ race：可能在 Wait 返回後才 Add
        defer wg.Done()
    }(i)
}

// ⚠️ 常見錯誤 2：WaitGroup 被複製（應該用指標傳遞）
func process(wg sync.WaitGroup) { // ❌ 複製了 WaitGroup
    defer wg.Done()
}
func process(wg *sync.WaitGroup) { // ✅ 指標
    defer wg.Done()
}
```

---

## sync.Cond（條件變數）

適合「等待某個條件成立」的場景：

```go
type Queue struct {
    mu    sync.Mutex
    cond  *sync.Cond
    items []Item
}

func NewQueue() *Queue {
    q := &Queue{}
    q.cond = sync.NewCond(&q.mu)
    return q
}

// 生產者
func (q *Queue) Enqueue(item Item) {
    q.mu.Lock()
    q.items = append(q.items, item)
    q.mu.Unlock()
    q.cond.Signal() // 通知一個等待的消費者
    // q.cond.Broadcast() // 通知所有等待的消費者
}

// 消費者
func (q *Queue) Dequeue() Item {
    q.mu.Lock()
    defer q.mu.Unlock()

    for len(q.items) == 0 { // 注意：用 for 不用 if（避免虛假喚醒）
        q.cond.Wait() // 釋放鎖並等待 Signal
    }

    item := q.items[0]
    q.items = q.items[1:]
    return item
}

// 實務上 channel 通常比 sync.Cond 更慣用
// sync.Cond 主要用於：需要廣播（Broadcast）給多個 waiter 的場景
```

---

## singleflight（防快取擊穿）

```go
import "golang.org/x/sync/singleflight"

var sfGroup singleflight.Group

func getUser(ctx context.Context, id string) (*User, error) {
    // 即使 1000 個並發請求同時查同一個 user_id
    // 只有第一個真正執行，其他等待並共用結果
    result, err, shared := sfGroup.Do("user:"+id, func() (interface{}, error) {
        return db.QueryUser(ctx, id)
    })
    if err != nil {
        return nil, err
    }
    if shared {
        log.Printf("user %s result was shared", id)
    }
    return result.(*User), nil
}
```

---

## errgroup（並發任務的 WaitGroup + 錯誤處理）

```go
import "golang.org/x/sync/errgroup"

func fetchAll(ctx context.Context, ids []string) ([]*Item, error) {
    g, ctx := errgroup.WithContext(ctx) // 任一 goroutine 返回 error，ctx 被取消
    items := make([]*Item, len(ids))

    for i, id := range ids {
        i, id := i, id
        g.Go(func() error {
            item, err := fetchOne(ctx, id)
            if err != nil {
                return err // 觸發取消
            }
            items[i] = item
            return nil
        })
    }

    if err := g.Wait(); err != nil {
        return nil, err // 返回第一個錯誤
    }
    return items, nil
}

// 限制並發數量
g.SetLimit(10) // 最多 10 個 goroutine 同時跑
```

## 相關頁面

- [[Go並發模式]] — channel、Worker Pool、Fan-in/Fan-out
- [[Go執行期內部機制]] — GMP 排程器、goroutine 的底層實作
- [[分散式鎖]] — 跨進程的鎖（Redis SETNX）
- [[Go記憶體洩漏排查]] — 並發程式的洩漏場景
