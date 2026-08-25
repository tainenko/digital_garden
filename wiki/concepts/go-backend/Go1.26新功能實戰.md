---
title: Go 1.26 新功能實戰
type: concept
tags: [golang, go1.26, green-tea-gc, generics, new-builtin, hpke, senior]
created: 2026-08-25
updated: 2026-08-25
---

# Go 1.26 新功能實戰

> 發佈日期：2026 年 2 月。三大主線：Green Tea GC 正式預設啟用、兩項語言語法擴充（`new` 帶初始值、泛型自參照）、`crypto/hpke` 後量子加密新套件。

---

## 一、Green Tea GC 正式預設

Go 1.25 以 `GOEXPERIMENT=greenteagc` 引入實驗性 Green Tea GC，Go 1.26 正式設為預設，無需任何設定。

### 原理

傳統 GC 以「逐指針掃描」為主，記憶體存取分散、CPU cache 命中率低。Green Tea GC 改為以**連續記憶體頁（page）** 為單位批次掃描小物件，提升 locality 與 CPU 可擴展性。

### 效能預期

| 場景 | GC overhead 降低幅度 |
|------|---------------------|
| 大量小物件分配（如 JSON 解析、RPC） | 10–40% |
| 記憶體壓力高的服務 | 顯著 |
| 計算密集、低分配服務 | 改善有限 |

### 驗證方式

```bash
# 若需關閉回舊版 GC（用於對比測試）
GOGC=off GOEXPERIMENT=nongreenteagc go run main.go

# 用 pprof 觀察 GC 暫停時間
go tool pprof -alloc_objects http://localhost:6060/debug/pprof/heap
```

> 配合 [[Go pprof實戰完整指南]] 做 before/after 對比。

---

## 二、語言改動 1：`new` 帶初始值

`new` 原本只接受型別，Go 1.26 允許傳入**含初始值的表達式**。

```go
// Go 1.25 以前：需要兩行
x := int64(300)
ptr := &x

// Go 1.26：一行搞定
ptr := new(int64(300))
fmt.Println(*ptr) // 300

// 適用於任何型別
type Config struct {
    Timeout int
    Debug   bool
}
cfg := new(Config{Timeout: 30, Debug: true})
```

### 使用時機

- 回傳指標的工廠函式簡化
- 初始化 optional 欄位（`*string`、`*int` 等）

```go
// 常見 optional field 初始化
type Options struct {
    MaxRetries *int
    Timeout    *time.Duration
}

func DefaultOptions() *Options {
    return &Options{
        MaxRetries: new(int(3)),                   // 清晰
        Timeout:    new(time.Duration(30 * time.Second)),
    }
}
```

---

## 三、語言改動 2：泛型自參照（Self-Referential Generics）

型別約束現在可以參照自身，解決之前需要繁瑣 workaround 的遞迴泛型場景。

### 問題回顧

```go
// Go 1.25 以前：無法直接表達「A 的 Add 方法回傳相同型別」
// 只能用 interface{} 或 any 喪失型別安全
```

### Go 1.26 解法

```go
// 自參照約束：A 必須是 Adder[A]
type Adder[A Adder[A]] interface {
    Add(A) A
}

// 實作
type Vector2D struct{ X, Y float64 }

func (v Vector2D) Add(other Vector2D) Vector2D {
    return Vector2D{v.X + other.X, v.Y + other.Y}
}

// Vector2D 自動滿足 Adder[Vector2D] 約束
func Sum[A Adder[A]](items []A) A {
    var result A
    for _, item := range items {
        result = result.Add(item)
    }
    return result
}

vecs := []Vector2D{{1, 2}, {3, 4}, {5, 6}}
total := Sum(vecs) // Vector2D{9, 12}
```

### 實戰場景：Comparable Tree Node

```go
type Ordered[T Ordered[T]] interface {
    Less(T) bool
}

type TreeNode[T Ordered[T]] struct {
    Value T
    Left  *TreeNode[T]
    Right *TreeNode[T]
}

func (n *TreeNode[T]) Insert(val T) *TreeNode[T] {
    if n == nil {
        return &TreeNode[T]{Value: val}
    }
    if val.Less(n.Value) {
        n.Left = n.Left.Insert(val)
    } else {
        n.Right = n.Right.Insert(val)
    }
    return n
}
```

---

## 四、`crypto/hpke`：Hybrid Public Key Encryption

新增 `crypto/hpke` package，實作 RFC 9180 HPKE 標準，支援後量子混合 KEM。

```go
import "crypto/hpke"

// 建立 HPKE suite（KEM + KDF + AEAD 三件組）
suite := hpke.NewSuite(hpke.KEM_X25519_HKDF_SHA256,
                       hpke.KDF_HKDF_SHA256,
                       hpke.AEAD_AES128GCM)

// 接收方：生成 keypair
pubKey, privKey, _ := suite.GenerateKeyPair()

// 傳送方：加密
sender, _ := suite.NewSender(pubKey, nil)
enc, sealer, _ := sender.Setup(rand.Reader)
ciphertext, _ := sealer.Seal([]byte("secret message"), nil)

// 接收方：解密
receiver, _ := suite.NewReceiver(privKey, nil)
opener, _ := receiver.Setup(enc)
plaintext, _ := opener.Open(ciphertext, nil)
```

### 與 Go 1.25 後量子 TLS 的關係

- Go 1.25：TLS 1.3 自動支援 ML-KEM（後量子 key exchange）
- Go 1.26：`crypto/hpke` 提供應用層後量子加密 API（不限於 TLS）

---

## 五、`go fix` 全面翻新

`go fix` 從只支援少數固定修正，擴展為**可插拔 modernizer 框架**，包含數十個 fixer。

```bash
# 列出所有可用 fixer
go fix -list ./...

# 套用特定 fixer
go fix -fix=stdversion ./...   # 更新 //go:build 標記
go fix -fix=httpmux ./...      # 升級 net/http ServeMux 路由語法

# 套用所有 modernizer
go fix ./...
```

### 常用 Fixer 一覽

| Fixer | 說明 |
|-------|------|
| `stdversion` | 更新 `//go:build` 版本標記 |
| `httpmux` | 升級 `http.ServeMux` 路由到新語法 |
| `loopvar` | 處理 Go 1.22 迴圈變數語意的遺留寫法 |
| `slog` | 遷移舊 log 呼叫到 `log/slog` |

---

## 六、cgo 效能優化

cgo 基礎呼叫 overhead 降低約 **30%**，對有大量 C 互操作的服務有感。

---

## 版本對照速查

| 版本 | 關鍵功能 |
|------|---------|
| [[Go1.24新功能實戰\|Go 1.24]] | 泛型型別別名、weak.Pointer、Swiss Table Map |
| [[Go1.25新功能實戰\|Go 1.25]] | testing/synctest 穩定、Green Tea GC 實驗 |
| **Go 1.26（本頁）** | **Green Tea GC 預設、new 帶初始值、泛型自參照、crypto/hpke** |
| [[Go1.27新功能實戰\|Go 1.27]] | Generic Methods、encoding/json/v2 畢業、uuid |

---

## 相關頁面

- [[Go泛型設計]] — 泛型基礎，自參照約束的前置知識
- [[Go pprof實戰完整指南]] — 驗證 Green Tea GC 效果
- [[Go安全性實踐]] — HPKE 後量子加密背景
- [[Go執行期內部機制]] — GC 運作原理
- [[Go1.27新功能實戰]] — 下一版
