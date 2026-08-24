---
title: Go 1.23 新功能實戰
type: concept
tags: [golang, go1.23, range-over-func, iterator, iter, unique, timer, senior]
created: 2026-04-30
updated: 2026-04-30
---

# Go 1.23 新功能實戰

> 發佈日期：2024 年 8 月。核心亮點是 **range-over-func**——Go 終於有了一等公民的迭代器協議，改變了集合、流處理的寫法。

## 一、Range over Functions（迭代器）

### 基本概念

Go 1.23 允許 `for range` 直接迭代**函數**，只要該函數符合特定的簽名（yield pattern）：

```go
// 迭代器的三種有效簽名：
type (
    // 無值迭代（只迭代次數）
    iter.Seq0 = func(yield func() bool)

    // 單值迭代（最常用）
    iter.Seq[V any] = func(yield func(V) bool)

    // 雙值迭代（key-value）
    iter.Seq2[K, V any] = func(yield func(K, V) bool)
)

// yield 返回 false 代表 break，返回 true 代表繼續
```

### 實作自己的迭代器

```go
// 範例：實作一個 Filter 迭代器
func Filter[V any](seq iter.Seq[V], predicate func(V) bool) iter.Seq[V] {
    return func(yield func(V) bool) {
        for v := range seq {
            if predicate(v) {
                if !yield(v) { // yield 返回 false → break
                    return
                }
            }
        }
    }
}

// 範例：從 slice 建立迭代器
func SliceIter[V any](s []V) iter.Seq[V] {
    return func(yield func(V) bool) {
        for _, v := range s {
            if !yield(v) {
                return
            }
        }
    }
}

// 使用
nums := []int{1, 2, 3, 4, 5, 6}
evens := Filter(SliceIter(nums), func(n int) bool { return n%2 == 0 })

for n := range evens {
    fmt.Println(n) // 2, 4, 6
}
```

### 標準庫的迭代器支援

Go 1.23 同步更新了 `slices`、`maps` 套件，新增回傳迭代器的函數：

```go
import (
    "slices"
    "maps"
)

// slices 套件新函數
nums := []int{3, 1, 4, 1, 5, 9}

for i, v := range slices.All(nums) {        // Seq2[int, int]：索引+值
    fmt.Printf("nums[%d]=%d\n", i, v)
}

for v := range slices.Values(nums) {         // Seq[int]：只有值
    fmt.Println(v)
}

for i, v := range slices.Backward(nums) {    // Seq2[int, int]：逆序
    fmt.Printf("nums[%d]=%d\n", i, v)
}

// 收集迭代器結果回 slice
doubled := slices.Collect(func(yield func(int) bool) {
    for _, v := range nums {
        if !yield(v * 2) { return }
    }
})

// maps 套件新函數
m := map[string]int{"a": 1, "b": 2, "c": 3}

for k := range maps.Keys(m) {               // Seq[string]
    fmt.Println(k)
}

for v := range maps.Values(m) {             // Seq[int]
    fmt.Println(v)
}

for k, v := range maps.All(m) {             // Seq2[string, int]
    fmt.Printf("%s=%d\n", k, v)
}
```

### 雙值迭代器（Seq2）

```go
// 自訂 Seq2：線性方程組解
func LinearPairs(max int) iter.Seq2[int, int] {
    return func(yield func(int, int) bool) {
        for x := 0; x < max; x++ {
            y := x * 2 + 1
            if !yield(x, y) {
                return
            }
        }
    }
}

for x, y := range LinearPairs(5) {
    fmt.Printf("(%d, %d)\n", x, y)
}
// (0, 1), (1, 3), (2, 5), (3, 7), (4, 9)
```

### 鏈式操作（函數式風格）

```go
// Take：取前 N 個
func Take[V any](seq iter.Seq[V], n int) iter.Seq[V] {
    return func(yield func(V) bool) {
        count := 0
        for v := range seq {
            if count >= n || !yield(v) {
                return
            }
            count++
        }
    }
}

// Map：轉換值
func Map[In, Out any](seq iter.Seq[In], fn func(In) Out) iter.Seq[Out] {
    return func(yield func(Out) bool) {
        for v := range seq {
            if !yield(fn(v)) {
                return
            }
        }
    }
}

// 鏈式使用（惰性求值！每個元素只讀一次）
result := slices.Collect(
    Take(
        Map(
            Filter(SliceIter(largeSlice), isEven),
            square,
        ),
        10,
    ),
)
```

---

## 二、iter.Pull：拉取式迭代器

有時需要在兩個迭代器之間交替，或需要「暫停」迭代——`iter.Pull` 把推送式迭代器轉成拉取式：

```go
import "iter"

// iter.Pull 返回 next 和 stop 函數
next, stop := iter.Pull(slices.Values(nums))
defer stop() // 必須在使用完後 stop，釋放資源

v1, ok1 := next() // 取第一個
v2, ok2 := next() // 取第二個
// ...
```

**實用場景：Zip 兩個迭代器**

```go
func Zip[A, B any](seqA iter.Seq[A], seqB iter.Seq[B]) iter.Seq2[A, B] {
    return func(yield func(A, B) bool) {
        nextA, stopA := iter.Pull(seqA)
        nextB, stopB := iter.Pull(seqB)
        defer stopA()
        defer stopB()

        for {
            a, okA := nextA()
            b, okB := nextB()
            if !okA || !okB || !yield(a, b) {
                return
            }
        }
    }
}

for a, b := range Zip(slices.Values(as), slices.Values(bs)) {
    fmt.Printf("(%v, %v)\n", a, b)
}
```

---

## 三、unique 套件（值的 Interning）

`unique` 套件提供**值的規範化（interning/canonicalization）**：相同內容的值只存一份，節省記憶體，且比較用指標比較（O(1)）而非值比較。

```go
import "unique"

// Make：建立或取得規範化的 Handle
type Language struct {
    Code string
    Name string
}

h1 := unique.Make(Language{"zh", "Chinese"})
h2 := unique.Make(Language{"zh", "Chinese"})
h3 := unique.Make(Language{"en", "English"})

// Handle 的比較是指標比較（O(1)），即使結構體很大
fmt.Println(h1 == h2) // true（相同底層值，同一個 Handle）
fmt.Println(h1 == h3) // false

// 取得原始值
lang := h1.Value() // Language{Code: "zh", Name: "Chinese"}
```

**實用場景：減少重複字串的記憶體佔用**

```go
// 場景：大量 HTTP 請求中的 Content-Type header 值
type ContentType = unique.Handle[string]

func parseRequest(r *http.Request) ContentType {
    return unique.Make(r.Header.Get("Content-Type"))
    // "application/json" 在整個程式中只存一份
}

// 可以用 map key（因為 Handle 可比較）
cache := make(map[unique.Handle[string]][]byte)
key := unique.Make("application/json")
cache[key] = someData
```

---

## 四、Timer / Ticker 行為修正

Go 1.23 修正了 `time.Timer` 和 `time.Ticker` 的 Reset 語意——這是多年來的痛點。

```go
// ❌ Go 1.22 之前：Reset 之前必須先 drain channel，非常容易出錯
timer := time.NewTimer(5 * time.Second)
// 停止 timer 的正確（但繁瑣）方式：
if !timer.Stop() {
    select {
    case <-timer.C:
    default:
    }
}
timer.Reset(3 * time.Second) // 現在才能安全 Reset

// ✅ Go 1.23+：Stop 後直接 Reset，不需要 drain
timer := time.NewTimer(5 * time.Second)
timer.Stop()          // 停止
timer.Reset(3 * time.Second) // 可以直接 Reset，channel 已自動清空

// Ticker 同樣簡化
ticker := time.NewTicker(1 * time.Second)
ticker.Reset(500 * time.Millisecond) // 直接 Reset，無需先 Stop
```

**背後原因**：Go 1.23 確保 Stop 後 channel 不會再收到值，消除了 race condition 的根源。

---

## 五、`slices.Sorted` 等新工具

```go
// slices.Sorted：從迭代器收集並排序
sorted := slices.Sorted(maps.Values(m)) // 取 map 的所有 value 並排序

// slices.SortedFunc：自訂比較
type Person struct{ Name string; Age int }
people := []Person{{"Bob", 30}, {"Alice", 25}, {"Charlie", 35}}
byAge := slices.SortedFunc(slices.Values(people), func(a, b Person) int {
    return cmp.Compare(a.Age, b.Age)
})

// slices.Repeat：重複元素
repeated := slices.Repeat([]int{1, 2, 3}, 3)
// [1 2 3 1 2 3 1 2 3]
```

---

## 升級檢查清單

```bash
# 更新到 go 1.23
go mod edit -go=1.23

# 修改 Timer/Ticker 相關代碼（可能需要移除舊的 drain 邏輯）
grep -r "timer.Stop\|timer.Reset\|ticker.Reset" .

# 測試迭代器相關（如有自定義 collection type 可考慮加迭代器支援）
go test -race ./...
```

---

## 相關頁面

- [[Go1.22新功能實戰]] — 迴圈變數修復、net/http 路由、range-over-int
- [[Go1.24新功能實戰]] — 泛型型別別名、weak 套件、os.Root
- [[Go泛型設計]] — iter.Seq 用到的泛型機制
- [[Go並發模式]] — Timer 在並發場景中的使用
