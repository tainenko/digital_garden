---
title: Go 面試陷阱題彙整
type: concept
tags: [golang, interview, code-puzzle, gotcha, 八股文]
created: 2026-06-29
updated: 2026-06-29
sources: [topgoer-go-interview-106days]
---

# Go 面試陷阱題彙整

「代碼輸出陷阱題」（給一段 code 問輸出/能否編譯）是 Go 面試的固定題型，考的是對**編譯器行為與語言細節**的精確掌握，而非演算法。本頁收斂高頻 pattern；系統化原理見對應深度頁。

> ⚠️ **版本前提**：許多經典陷阱在新版 Go 已改變行為（尤其 Go 1.22 的 for-range 迴圈變數語意）。做舊題庫（如 [[topgoer-go-interview-106days|topgoer 106 天]]）務必先確認 Go 版本。

## 1. for-range 迴圈變數捕獲

```go
slice := []int{0, 1, 2, 3}
m := make(map[int]*int)
for key, val := range slice {
    m[key] = &val          // 取的都是同一個 val 的位址
}
// Go <1.22：全部印出 3（最後一次迭代的值）
// Go ≥1.22：每次迭代新建 val，印出 0,1,2,3
```

- **坑**：`range` 在舊版每輪重用同一個 `val` 變數，取址會全部指向最後值。
- **修正**：迴圈內 `v := val; m[key] = &v`，或直接 `m[key] = &slice[key]`。
- **Go 1.22 變更**：迴圈變數改為每次迭代新建，此經典陷阱在新版**不再成立**。相關語意見 [[Go同步原語與記憶體模型]] 對共享變數的討論。

## 2. 型別系統與可賦值性

```go
a := 5      // int
b := 8.1    // float64
_ = a + b   // 編譯錯誤：不同型別不能相加
```

```go
type T int
func F(t T) {}
var q int
F(q)        // 編譯錯誤：int 與具名型別 T 不可互賦
```

- **規則**：底層型別相同仍不可互賦，**除非至少一方非具名型別（named type）**。故 `type T []int; var q []int; F(q)` 可通過（`[]int` 是未具名複合型別）。
- Go spec 原文：`x's type V and T have identical underlying types and at least one of V or T is not a named type`。

## 3. 三索引切片 `a[low:high:max]`

```go
a := [5]int{1, 2, 3, 4, 5}
t := a[3:4:4]   // len=1, cap=1；t[0]==4
```

- 第三個索引 `max` 控制 **容量**（cap = max−low），用來限制後續 `append` 是否會覆寫原陣列。

## 4. defer / recover / 命名返回值互動

```go
func f(n int) (r int) {
    defer func() { r += n; recover() }()
    var g func()
    defer g()           // g 尚未賦值 → 呼叫時 panic
    g = func() { r += 2 }
    return n + 1
}
// f(3) == 7
```

- **三步拆解**：① `return n+1` 先把具名返回值 `r` 設為 4；② 執行第二個 defer `g()`，此時 `g` 為 nil → panic；③ 第一個 defer 執行 `r += n`（4+3=7）並 `recover()` 吞掉 panic，正常返回 **7**。
- 系統化版本（LIFO、迴圈 defer 陷阱、recovery middleware）見 [[Go defer與panic]]。

## 5. 嵌入結構 + 指標方法的 nil 解引用

```go
type T struct{}
func (*T) foo() {}
func (T) bar()  {}
type S struct{ *T }   // 嵌入 *T，零值為 nil

s := S{}
s.foo()   // OK：foo 的 receiver 是 *T，s.T(nil) 直接當 receiver
s.bar()   // panic：展開為 (*s.T).bar()，解引用 nil 指標
```

- **坑**：嵌入指標型別的方法集合包含值 receiver 方法時，呼叫值 receiver 方法需先**解引用**該指標；若指標為 nil 即 panic。
- `fmt.Printf("%#v", s)` → `main.S{T:(*main.T)(nil)}`，可看出嵌入指標為 nil。
- receiver 值/指標語意見 [[Go介面設計模式]]。

## 與其他頁面的分工

| 維度 | 頁面 |
|------|------|
| 陷阱題自測（本頁） | Go 面試陷阱題彙整 |
| defer/panic 系統化 | [[Go defer與panic]] |
| 排程/GC/逃逸分析 | [[Go執行期內部機制]] |
| 並發/記憶體模型 | [[Go同步原語與記憶體模型]] |
| 實作題（OA/白板） | [[Coinbase面試題與Go解答]]、[[HashiCorp_Banking_System_OA]] |

## 相關頁面

- [[topgoer-go-interview-106days]]
- [[Go defer與panic]]
- [[Go執行期內部機制]]
- [[大廠技術面試的底層邏輯]]
