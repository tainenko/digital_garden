---
title: Go defer 與 panic 深度解析
type: concept
tags: [golang, defer, panic, recover, senior]
created: 2026-04-29
updated: 2026-04-29
---

# Go defer 與 panic 深度解析

## defer 的執行語意

### 基本規則

```go
func example() {
    defer fmt.Println("1") // 後 defer，先執行（LIFO）
    defer fmt.Println("2")
    defer fmt.Println("3")
    fmt.Println("normal")
}
// 輸出順序：normal → 3 → 2 → 1
```

**三個規則**：
1. 函數返回時（不論正常還是 panic）執行
2. LIFO 順序（後 defer 先執行）
3. defer 的**參數**在 defer 語句執行時就被求值（不是等到函數返回時）

### 參數立即求值

```go
func printValue(x int) {
    defer fmt.Println(x) // x 的值在這行就被捕捉了
    x = 100
}
printValue(1) // 輸出：1（不是 100）

// 對比：閉包捕捉的是引用
func printValueClosure(x int) {
    defer func() {
        fmt.Println(x) // x 是引用，會看到最新值
    }()
    x = 100
}
printValueClosure(1) // 輸出：100
```

---

## defer + 命名返回值（Named Return）

這是 Go 面試最常考的 defer 題：

```go
// ❌ 意外行為：defer 修改了返回值
func triple(x int) (result int) { // 命名返回值 result
    defer func() {
        result *= 3 // defer 可以修改命名返回值！
    }()
    return x // 這裡設定 result = x，然後執行 defer（result *= 3），最後返回
}
fmt.Println(triple(5)) // 輸出 15，不是 5！

// 執行順序：
// 1. return x → 相當於 result = x; return
// 2. defer 執行 → result *= 3
// 3. 實際返回 result（已被修改）
```

### 利用命名返回值處理錯誤

```go
// ✅ 常見的好用模式：defer 統一處理錯誤
func queryUser(id string) (user *User, err error) {
    tx, err := db.Begin()
    if err != nil {
        return nil, err
    }

    defer func() {
        if err != nil {
            tx.Rollback() // 如果有錯誤，回滾
        } else {
            err = tx.Commit() // 成功，提交（並把 commit 的 error 透過命名返回值傳出去）
        }
    }()

    user, err = tx.QueryRow("SELECT * FROM users WHERE id = $1", id).Scan(...)
    return // 不需要明確指定返回值，defer 裡已經設置好了
}
```

---

## defer 的效能考量

```go
// 在 hot path 上，defer 有額外開銷（雖然 Go 1.14+ 已大幅優化）
// Go 1.14 之前：defer ≈ 300ns
// Go 1.14+ 開放式 defer（open-coded defer）：幾乎無開銷

// 只有在以下情況 defer 仍有 overhead：
// - defer 在迴圈中（無法用 open-coded）
// - 有 recover() 時
// - 介面呼叫的 defer

// ❌ 迴圈內的 defer（每次迭代都 defer，函數結束才執行）
func processFiles(files []string) error {
    for _, f := range files {
        file, err := os.Open(f)
        if err != nil {
            return err
        }
        defer file.Close() // 所有 file 都等到函數結束才關！
        process(file)
    }
    return nil
}

// ✅ 用閉包或獨立函數，確保每次迭代後都關閉
func processFiles(files []string) error {
    for _, f := range files {
        if err := processFile(f); err != nil {
            return err
        }
    }
    return nil
}

func processFile(path string) error {
    file, err := os.Open(path)
    if err != nil {
        return err
    }
    defer file.Close() // 這個函數結束就關閉，正確
    return process(file)
}
```

---

## panic 與 recover

### panic 的語意

```go
// panic 會：
// 1. 立即停止當前函數的執行
// 2. 開始展開（unwind）call stack
// 3. 執行沿途所有 defer（可以在 defer 中 recover）
// 4. 若沒有 recover，程式崩潰並印出 stack trace

func riskyOperation() {
    defer fmt.Println("defer runs even during panic")
    panic("something went wrong")
    fmt.Println("this never runs")
}
```

### recover：在 defer 中攔截 panic

```go
func safeOperation() (err error) {
    defer func() {
        if r := recover(); r != nil {
            // r 是 panic 的值（可以是任何型別）
            err = fmt.Errorf("recovered from panic: %v", r)
            // 可選：印出 stack trace
            debug.PrintStack()
        }
    }()

    riskyOperation()
    return nil
}

// recover 只在 defer 函數中有效
// 且只能攔截同一個 goroutine 的 panic
```

### HTTP Server 的 Recovery Middleware

```go
func recoveryMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        defer func() {
            if rec := recover(); rec != nil {
                // 記錄完整 stack trace
                buf := make([]byte, 4096)
                n := runtime.Stack(buf, false)
                log.Errorf("panic recovered: %v\n%s", rec, buf[:n])

                // 返回 500，不讓 server 崩潰
                http.Error(w, "Internal Server Error", http.StatusInternalServerError)
            }
        }()
        next.ServeHTTP(w, r)
    })
}
```

---

## panic vs error：何時用哪個

```go
// 使用 panic 的場景（少數情況）：
// 1. 程式設計錯誤（不應該發生的情況）
func mustPositive(n int) int {
    if n <= 0 {
        panic(fmt.Sprintf("n must be positive, got %d", n))
    }
    return n
}

// 2. 初始化失敗（啟動時 fail-fast）
var db = mustConnect("postgres://...")

func mustConnect(dsn string) *sql.DB {
    db, err := sql.Open("pgx", dsn)
    if err != nil {
        panic("failed to connect to database: " + err.Error())
    }
    return db
}

// 3. 跨越 package 邊界的錯誤傳遞（少數情況）
// 在 package 內部用 panic，在 exported function 的 defer 裡 recover 轉成 error
// json.Marshal 就是這樣實作的

// ❌ 不應該用 panic 的場景：
// - 任何預期中的錯誤（輸入驗證、資源不存在、網路錯誤）
// - 用 error 返回值處理這些情況
```

---

## 常見 defer 陷阱

### 陷阱 1：defer 在 if 失敗後才 defer

```go
// ❌ 如果 Open 失敗，defer Close 仍然會被執行
func readFile(path string) error {
    f, err := os.Open(path)
    defer f.Close() // 如果 err != nil，f 是 nil！呼叫 nil.Close() 會 panic
    if err != nil {
        return err
    }
    // ...
}

// ✅ 在確認 Open 成功後才 defer
func readFile(path string) error {
    f, err := os.Open(path)
    if err != nil {
        return err
    }
    defer f.Close() // 只有 Open 成功，才 defer Close
    // ...
}
```

### 陷阱 2：defer 的閉包沒有捕捉正確的變數

```go
// ❌ 迴圈變數逃逸問題（Go 1.22 之前）
for i := 0; i < 3; i++ {
    defer func() {
        fmt.Println(i) // 所有 defer 共用同一個 i，最後都是 3
    }()
}

// ✅ Go 1.22 之前的修法：捕捉當前值
for i := 0; i < 3; i++ {
    i := i // 建立新的變數
    defer func() {
        fmt.Println(i) // 輸出 2, 1, 0
    }()
}

// Go 1.22+ 已修復迴圈變數問題，每次迭代有獨立的 i
```

### 陷阱 3：defer recover 必須在同一個 goroutine

```go
// ❌ 無法 recover 另一個 goroutine 的 panic
func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("recovered") // 這裡 recover 不到！
        }
    }()

    go func() {
        panic("goroutine panic") // 這個 panic 在另一個 goroutine，main 的 recover 抓不到
    }()

    time.Sleep(time.Second)
}

// ✅ 每個 goroutine 需要自己的 recover
go func() {
    defer func() {
        if r := recover(); r != nil {
            log.Printf("goroutine panic recovered: %v", r)
        }
    }()
    panic("goroutine panic") // 自己的 recover 可以抓到
}()
```

---

## defer 的好用模式

### 資源清理（最常見）

```go
func processDB() error {
    conn, err := db.Acquire(ctx)
    if err != nil { return err }
    defer conn.Release()

    tx, err := conn.Begin(ctx)
    if err != nil { return err }
    defer tx.Rollback() // 如果 Commit 先執行，Rollback 是空操作（已 committed 的無法 rollback）

    // ... 業務邏輯 ...

    return tx.Commit()
}
```

### 計時（Tracing）

```go
func (s *Service) CreateOrder(ctx context.Context) error {
    start := time.Now()
    defer func() {
        duration := time.Since(start)
        metrics.RecordDuration("create_order", duration)
        if duration > 500*time.Millisecond {
            log.Warnf("CreateOrder took %v", duration)
        }
    }()
    // ... 業務邏輯 ...
}
```

### 解鎖

```go
func (c *Cache) Set(key string, value []byte) {
    c.mu.Lock()
    defer c.mu.Unlock() // 即使函數 panic，鎖也會被釋放
    c.data[key] = value
}
```

## 相關頁面

- [[Go錯誤處理最佳實踐]] — error vs panic 的選擇
- [[Go並發模式]] — goroutine 中的 panic 處理
- [[Go效能調優]] — defer 的效能開銷與 open-coded defer
- [[Go面試陷阱題彙整]] — defer×recover×命名返回值的代碼輸出陷阱題
