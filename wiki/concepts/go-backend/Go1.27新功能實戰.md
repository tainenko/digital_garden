---
title: Go 1.27 新功能實戰
type: concept
tags: [golang, go1.27, generic-methods, json-v2, post-quantum, senior]
created: 2026-08-25
updated: 2026-08-25
---

# Go 1.27 新功能實戰

> 發佈日期：2026 年 8 月。三大主線：Generic Methods 解除四年限制、encoding/json/v2 正式畢業、Post-Quantum 密碼學進標準庫。

---

## 一、Generic Methods（最重要語言改動）

Go 1.18 加了泛型，但 method 不能有自己的 type parameter——這個限制讓泛型在許多場景只能用 package-level function 繞道。Go 1.27 正式移除。

### 問題回顧

```go
// Go 1.26 以前：同樣邏輯要寫多個 method
func (r *Rand) Int32N(n int32) int32 { ... }
func (r *Rand) Int64N(n int64) int64 { ... }
func (r *Rand) IntN(n int) int { ... }
```

### Go 1.27 解法

```go
type intType interface {
    ~int | ~int32 | ~int64
}

type Rand struct{ /* ... */ }

// Method 可以有自己的 type parameter 了
func (r *Rand) N[Int intType](n Int) Int {
    // 統一實作
    return Int(r.source.Int64()) % n
}

// 呼叫時型別通常可以推導
r := &Rand{}
val32 := r.N[int32](100)
val64 := r.N[int64](1000)
valInt := r.N(50) // 推導為 int
```

### 實戰場景：通用 Repository

```go
type Repository[T any] struct {
    db *sql.DB
}

// 1.27 前只能用 package function
// func FindBy[T any, F any](repo *Repository[T], field string, val F) ([]T, error)

// 1.27 後可以直接是 method
func (r *Repository[T]) FindBy[F comparable](field string, val F) ([]T, error) {
    query := fmt.Sprintf("SELECT * FROM table WHERE %s = ?", field)
    rows, err := r.db.Query(query, val)
    if err != nil {
        return nil, err
    }
    // ... scan rows
}

// 使用
repo := &Repository[User]{db: db}
users, err := repo.FindBy("email", "tony@example.com")
users, err = repo.FindBy("age", 30)
```

### 限制（重要）

```go
// ❌ interface 的 method 不能宣告 type parameter
type Store interface {
    Find[T any](id int) T  // 編譯錯誤
}

// ❌ generic method 不能實作 interface method
type Stringer interface {
    String() string
}
func (r *Rand) String[T any]() string { ... }  // 不算實作 Stringer
```

---

## 二、encoding/json/v2 正式標準庫

從 Go 1.25 的 `GOEXPERIMENT=jsonv2` 實驗，1.27 正式進標準庫，不需要任何 flag。

### 兩個新 package

```go
import "encoding/json/v2"      // 高階：Marshal / Unmarshal
import "encoding/json/jsontext" // 低階：Streaming Token 處理
```

### 主要差異與破壞性變更

```go
// v1 行為：忽略大小寫匹配（case-insensitive）
// v2 行為：嚴格大小寫匹配（更快、更明確）

type User struct {
    Name string `json:"name"`
}

data := `{"Name": "Tony"}` // 注意大寫 N

var u User
json.Unmarshal([]byte(data), &u) // v1：成功，u.Name = "Tony"
                                  // v2：失敗或零值，"name" != "Name"
```

```go
// v2 新 tag 選項
type Config struct {
    Secret string `json:",omitzero"`    // 零值才省略（v1 是 omitempty）
    Data   []byte `json:",format:base64"` // 內建 base64 編碼
    Raw    any    `json:",unknown"`      // 收集未知欄位
}
```

### 效能

| 操作 | v1 | v2 |
|------|----|----|
| Marshal | 基準 | 持平 |
| Unmarshal | 基準 | **明顯更快** |

### 遷移注意

v1 和 v2 行為不相容，升級前需測試：
- 大小寫匹配行為改變
- `omitempty` vs `omitzero` 語意不同
- 部分 struct tag 格式更嚴格

---

## 三、其他重要更新

### `uuid` package

```go
import "uuid"

id := uuid.New()           // 生成 UUID v4
fmt.Println(id.String())   // "550e8400-e29b-41d4-a716-446655440000"

parsed, err := uuid.Parse("550e8400-e29b-41d4-a716-446655440000")
```

不再需要第三方套件（`github.com/google/uuid`）處理基本 UUID。

### Goroutine Leak Profile

```go
// 新的 pprof profile 類型
// go tool pprof http://localhost:6060/debug/pprof/goroutineleak

// 偵測長時間未退出的 goroutine
// 配合 [[Go記憶體洩漏排查]] 使用
```

### Post-Quantum 密碼學

```go
import "crypto/mlkem"  // ML-KEM（CRYSTALS-Kyber），NIST 標準

// 主要用於 TLS 1.3 key exchange
// 對應量子電腦破解 RSA/ECDH 的威脅
// 一般業務代碼不需直接使用，TLS 層自動受益
```

### `go fix` Modernizer 擴展

```bash
# 自動將舊寫法升級
go fix ./...

# 例如：自動將 encoding/json 改為 encoding/json/v2（若相容）
# 自動套用 Generic Methods 重構（部分場景）
```

---

## 版本對照速查

| 版本 | 關鍵功能 |
|------|---------|
| [[Go1.22新功能實戰\|Go 1.22]] | for-range 迴圈變數每輪新建、range over integer |
| [[Go1.23新功能實戰\|Go 1.23]] | range over func、timer 修正、toolchain 管理 |
| [[Go1.24新功能實戰\|Go 1.24]] | generic type alias、weak pointer、testing/synctest 實驗 |
| [[Go1.25新功能實戰\|Go 1.25]] | testing/synctest 穩定、json/v2 實驗（GOEXPERIMENT） |
| **Go 1.27（本頁）** | **Generic Methods、json/v2 畢業、Post-Quantum、uuid** |

---

## 相關頁面

- [[Go泛型設計]] — 泛型基礎，generic methods 的前置知識
- [[Go記憶體洩漏排查]] — 配合新的 goroutine leak profile
- [[Go安全性實踐]] — Post-Quantum 密碼學背景
- [[Go1.25新功能實戰]] — 上一版
