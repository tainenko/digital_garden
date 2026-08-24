---
title: Go 1.22 新功能實戰
type: concept
tags: [golang, go1.22, loop-variable, range-integer, net/http, math/rand, senior]
created: 2026-04-30
updated: 2026-04-30
---

# Go 1.22 新功能實戰

> 發佈日期：2024 年 2 月。最重要的版本之一——修復了困擾 Go 開發者十餘年的迴圈變數語意問題。

## 一、迴圈變數語意修復（Loop Variable Semantics）

### 問題根源

Go 1.22 之前，`for` 迴圈中的迭代變數在每次迭代之間**共用同一個記憶體位址**，導致閉包捕捉到的永遠是最後一個值。

```go
// ❌ Go 1.21 之前：所有 goroutine 共用同一個 i
for i := 0; i < 3; i++ {
    go func() {
        fmt.Println(i) // 都印出 3（最終值）
    }()
}

// ✅ Go 1.21 之前的修法：手動建立新變數
for i := 0; i < 3; i++ {
    i := i // shadow 出新的 i
    go func() {
        fmt.Println(i) // 正確：0, 1, 2
    }()
}
```

### Go 1.22 的修法

從 Go 1.22 起，`for` 迴圈的每次迭代都會建立**新的迭代變數**（不同記憶體位址）。舊的 shadow 寫法變成多餘。

```go
// ✅ Go 1.22+：直接正確，不再需要 i := i
for i := 0; i < 3; i++ {
    go func() {
        fmt.Println(i) // 正確：0, 1, 2（每個 goroutine 有自己的 i）
    }()
}

// range 也一樣
for _, v := range items {
    go func() {
        process(v) // Go 1.22+：v 是獨立的，不會全部指向最後一個元素
    }()
}
```

**升級注意事項**：如果舊代碼依賴迴圈變數共用（罕見但存在），Go 1.22 可能改變行為。用 `go vet` 可以偵測。

---

## 二、Range over Integers

Go 1.22 允許直接對整數使用 `range`：

```go
// ✅ 新語法：range 整數
for i := range 5 {
    fmt.Println(i) // 0, 1, 2, 3, 4
}

// 等同於
for i := 0; i < 5; i++ { ... }

// 實用場景：建立固定長度的 slice
workers := make([]Worker, 0, n)
for range n {
    workers = append(workers, newWorker())
}

// 只需要迭代次數，不需要索引時
for range 3 {
    retry()
}
```

---

## 三、net/http 路由增強

Go 1.22 大幅強化了 `net/http` 的內建路由器，過去需要用 `gorilla/mux` 的場景現在原生支援。

### Method Pattern

```go
mux := http.NewServeMux()

// ✅ 新：方法 + 路徑模式
mux.HandleFunc("GET /users", listUsers)
mux.HandleFunc("POST /users", createUser)
mux.HandleFunc("GET /users/{id}", getUser)
mux.HandleFunc("PUT /users/{id}", updateUser)
mux.HandleFunc("DELETE /users/{id}", deleteUser)

// 舊：只能用路徑，無法限制方法
// mux.HandleFunc("/users", handleUsers) // 需要在 handler 內部 switch r.Method
```

### Path Parameters（路徑參數）

```go
// 在 handler 中取得路徑參數
func getUser(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id") // 取得 {id} 的值
    // id = "123"（從 GET /users/123）
}

// Wildcard：{name...} 匹配剩餘所有路徑段
mux.HandleFunc("GET /files/{path...}", serveFile)

func serveFile(w http.ResponseWriter, r *http.Request) {
    path := r.PathValue("path") // "docs/2024/report.pdf"
}

// 精確匹配（加 {$} 防止路徑前綴匹配）
mux.HandleFunc("GET /{$}", handleRoot) // 只匹配 "/"，不匹配 "/anything"
```

### Host Pattern

```go
// 支援 Host-based routing
mux.HandleFunc("api.example.com/users", handleAPIUsers)
mux.HandleFunc("example.com/users", handleWebUsers)
```

**原生 mux vs gorilla/mux**：對於大多數應用，Go 1.22 的內建路由已夠用。複雜的需求（正則匹配、中介軟體鏈）仍可考慮 `chi` 或 `gorilla/mux`。

---

## 四、math/rand/v2

全新的隨機數套件，修正了 v1 的 API 設計問題。

```go
import "math/rand/v2"

// v2 的改變：
// 1. 不再需要 rand.Seed()（每個程式自動隨機化）
// 2. 移除全域鎖（並發安全不需要手動同步）
// 3. 更好的分佈函數命名

// ✅ 新 API
n := rand.IntN(100)       // [0, 100) 的隨機整數（N 後綴代表 n 為上界）
f := rand.Float64()        // [0.0, 1.0)
b := rand.N[int64](1000)  // 泛型版本

// Shuffle（v1 也有，但 v2 API 更清楚）
items := []string{"a", "b", "c", "d"}
rand.Shuffle(len(items), func(i, j int) {
    items[i], items[j] = items[j], items[i]
})

// 固定種子（測試用）
r := rand.New(rand.NewPCG(seed1, seed2))
n = r.IntN(100)

// v1 寫法（仍可用，但不推薦新代碼）
// rand.Intn(100)  ← 注意是 Intn 不是 IntN
```

**v2 vs v1 主要差異**：
| 面向 | v1 | v2 |
|------|----|----|
| 全域隨機化 | 需 `rand.Seed(time.Now().UnixNano())` | 自動 |
| 上界函數 | `rand.Intn(n)` | `rand.IntN(n)` |
| 演算法 | 線性同餘（較慢） | PCG / ChaCha8（更快更好） |
| 泛型支援 | 無 | `rand.N[T](n)` |

---

## 五、database/sql 改善

```go
// Null 型別新增泛型版本（Go 1.22+）
// 過去：sql.NullString, sql.NullInt64, sql.NullBool...
// 現在：sql.Null[T]

type Product struct {
    ID          int
    Description sql.Null[string]  // 可為 NULL 的字串
    Weight      sql.Null[float64] // 可為 NULL 的浮點數
    ExpiredAt   sql.Null[time.Time]
}

var p Product
row.Scan(&p.ID, &p.Description, &p.Weight, &p.ExpiredAt)

if p.Description.Valid {
    fmt.Println(p.Description.V) // 用 .V 而非 .String
}
```

---

## 相關頁面

- [[Go1.23新功能實戰]] — range-over-func 迭代器、iter 套件
- [[Go1.24新功能實戰]] — 泛型型別別名、weak 套件
- [[Go並發模式]] — goroutine 與 range 的互動
- [[Go介面設計模式]] — net/http 新路由的中介軟體設計
