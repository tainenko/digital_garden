---
title: Go 泛型設計（Generics）
type: concept
tags: [golang, generics, type-parameters, constraints, senior]
created: 2026-04-29
updated: 2026-04-29
---

# Go 泛型設計（Generics）

Go 1.18（2022）引入泛型，解決「重複的型別邏輯」問題，但代價是複雜度。

## 基本語法

```go
// 型別參數放在函數名稱後的方括號
func Map[T, R any](slice []T, fn func(T) R) []R {
    result := make([]R, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// 使用：型別推斷（type inference），不需要手動指定型別
nums := []int{1, 2, 3, 4}
doubled := Map(nums, func(n int) int { return n * 2 })
strs := Map(nums, func(n int) string { return strconv.Itoa(n) })

// 需要時才手動指定型別
result := Map[int, string](nums, strconv.Itoa)
```

## Constraint（約束）

約束限制型別參數必須滿足的條件：

```go
// any = interface{}（最寬鬆，可以是任何型別）
func Print[T any](v T) { fmt.Println(v) }

// comparable：可以用 == != 比較的型別（基本型別、pointer、struct）
func Contains[T comparable](slice []T, target T) bool {
    for _, v := range slice {
        if v == target {
            return true
        }
    }
    return false
}

// 自定義約束：型別集合（Type Set）
type Number interface {
    int | int8 | int16 | int32 | int64 |
    uint | uint8 | uint16 | uint32 | uint64 |
    float32 | float64
}

func Sum[T Number](nums []T) T {
    var total T
    for _, n := range nums {
        total += n
    }
    return total
}

// ~ 波浪號：包含底層型別（underlying type）是該型別的自定義型別
type Celsius float64
type Fahrenheit float64

type Ordered interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64 |
    ~string
}

func Min[T Ordered](a, b T) T {
    if a < b {
        return a
    }
    return b
}

// Celsius 的底層型別是 float64，~float64 匹配
var c1 Celsius = 100
var c2 Celsius = 200
result := Min(c1, c2) // ✅ 有效
```

## 標準庫的 Constraints Package

```go
import "golang.org/x/exp/constraints"

// constraints.Ordered：所有可排序型別（整數、浮點、字串）
// constraints.Integer：所有整數型別
// constraints.Float：所有浮點型別
// constraints.Signed：所有有號整數
// constraints.Unsigned：所有無號整數

// Go 1.21 標準庫也加入了 cmp 和 slices package
import (
    "cmp"
    "slices"
)

// 排序任何可比較的 slice
nums := []int{3, 1, 4, 1, 5, 9, 2, 6}
slices.Sort(nums)

// 找最大值
max := slices.Max(nums)

// Binary search
idx, found := slices.BinarySearch(nums, 5)
```

## 泛型型別（Generic Types）

```go
// 泛型 Stack
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}

func (s *Stack[T]) Len() int { return len(s.items) }

// 使用
intStack := Stack[int]{}
intStack.Push(1)
intStack.Push(2)
v, _ := intStack.Pop() // v = 2

strStack := Stack[string]{}
strStack.Push("hello")
```

## 泛型 + 介面約束

```go
// 約束型別必須實作介面
type Stringer interface {
    String() string
}

func PrintAll[T Stringer](items []T) {
    for _, item := range items {
        fmt.Println(item.String())
    }
}

// 更複雜的約束：同時要求介面方法 + 可比較
type Identifiable interface {
    comparable
    ID() string
}

func Deduplicate[T Identifiable](items []T) []T {
    seen := make(map[string]bool)
    var result []T
    for _, item := range items {
        id := item.ID()
        if !seen[id] {
            seen[id] = true
            result = append(result, item)
        }
    }
    return result
}
```

## 實用泛型工具函數庫

```go
package generics

// Filter：過濾
func Filter[T any](slice []T, predicate func(T) bool) []T {
    var result []T
    for _, v := range slice {
        if predicate(v) {
            result = append(result, v)
        }
    }
    return result
}

// Reduce：聚合
func Reduce[T, R any](slice []T, initial R, fn func(R, T) R) R {
    result := initial
    for _, v := range slice {
        result = fn(result, v)
    }
    return result
}

// Keys：取 map 的所有 key
func Keys[K comparable, V any](m map[K]V) []K {
    keys := make([]K, 0, len(m))
    for k := range m {
        keys = append(keys, k)
    }
    return keys
}

// Values：取 map 的所有 value
func Values[K comparable, V any](m map[K]V) []V {
    values := make([]V, 0, len(m))
    for _, v := range m {
        values = append(values, v)
    }
    return values
}

// GroupBy：按 key 分組
func GroupBy[T any, K comparable](slice []T, keyFn func(T) K) map[K][]T {
    result := make(map[K][]T)
    for _, item := range slice {
        key := keyFn(item)
        result[key] = append(result[key], item)
    }
    return result
}

// Must：包裝只有 error 會失敗的函數（用於初始化）
func Must[T any](val T, err error) T {
    if err != nil {
        panic(err)
    }
    return val
}

// Ptr：取任何值的指標（解決 &literal 不合法的問題）
func Ptr[T any](v T) *T { return &v }
```

## 泛型的限制與陷阱

### 1. 不能在方法上新增型別參數

```go
type MyType struct{}

// ❌ 不允許：Go 不支援 parametric method
func (m MyType) DoSomething[T any](v T) {}

// ✅ 用函數替代
func DoSomething[T any](m MyType, v T) {}

// ✅ 或在型別層級加型別參數
type MyType[T any] struct{ value T }
func (m MyType[T]) DoSomething() T { return m.value }
```

### 2. 型別推斷的限制

```go
// 有時需要手動指定型別
func Zero[T any]() T {
    var zero T
    return zero
}

// ❌ 無法推斷（沒有參數可以推斷）
v := Zero() // compile error

// ✅ 必須手動指定
v := Zero[int]()
```

### 3. 泛型 interface 不能被當作型別斷言目標

```go
type Number interface { int | float64 }

// ❌ 不能用含 union 的 interface 做型別斷言
var x any = 42
if v, ok := x.(Number); ok { // compile error
}

// ✅ 只能做具體型別斷言
if v, ok := x.(int); ok { ... }
```

### 4. 效能考量

```go
// 泛型透過 GCShape 實作：相同底層記憶體形狀（shape）的型別共用一份代碼
// int / int32 / *T 等 pointer 型別 → 共用同一份機器碼
// float64 / struct 等不同 size → 各自編譯一份

// 實際效能通常和手寫特化代碼相近
// 但有些情況 interface dispatch 會帶來 overhead：
type Numeric interface { ~int | ~float64 }
func SumSlice[T Numeric](s []T) T { ... }
// vs
func SumIntSlice(s []int) int { ... }
// 兩者效能基本相同（同 shape，同一份機器碼）
```

## 何時用泛型，何時不用

### ✅ 適合用泛型

- **容器型別**：Stack, Queue, Set, Cache — 邏輯和型別無關
- **工具函數**：Map, Filter, Reduce, Contains — 對任何型別都一樣的操作
- **算法**：排序、搜尋、歸約 — 邏輯只依賴比較操作
- **消除 `interface{}` 強制轉型**：把 `func process(v interface{})` 改成型別安全版本

### ❌ 不適合用泛型

- **業務邏輯**：訂單處理邏輯和型別高度相關，泛型只增加複雜度
- **只有一兩個型別**：直接寫兩個具體函數更清楚
- **需要不同行為**：不同型別的處理邏輯不同，應該用介面
- **過度抽象**：「以防萬一」的泛型 — 等真的需要再加

```go
// 判斷準則：
// 「如果把型別參數換成 any，邏輯是否完全相同？」
// → 是：適合泛型
// → 否：用介面或具體型別
```

## 相關頁面

- [[Go介面設計模式]] — 泛型 vs 介面的選擇
- [[Go效能調優]] — 泛型的 GCShape 與效能影響
- [[Go執行期內部機制]] — 泛型的底層實作原理
