---
title: Go 1.25 新功能實戰
type: concept
tags: [golang, go1.25, synctest, green-threads, generic-alias, senior]
created: 2026-04-30
updated: 2026-04-30
---

# Go 1.25 新功能實戰

> 發佈日期：2025 年 8 月（預計）。鞏固 1.24 的實驗性功能、持續優化 runtime 效能，並推進 Go 在並發測試與工具鏈上的改善。

## 一、testing/synctest 穩定化

`testing/synctest` 在 Go 1.24 以 `GOEXPERIMENT=synctest` 引入，Go 1.25 持續完善其 API 和行為。主要改善：

### Run / Wait / bubble 語意更清晰

```go
import "testing/synctest"

// synctest.Run：在隔離的虛擬時鐘 bubble 中執行函數
// - 所有在 bubble 內建立的 goroutine 都使用虛擬時間
// - 只有當所有 goroutine 都阻塞時，虛擬時間才會前進
// - bubble 結束時，若有未完成的 goroutine，測試 panic

func TestRetryLogic(t *testing.T) {
    synctest.Run(func() {
        attempts := 0
        service := &RetryService{
            MaxRetries: 3,
            Backoff:    100 * time.Millisecond,
        }

        go service.Call(context.Background(), func() error {
            attempts++
            if attempts < 3 {
                return errors.New("temporary failure")
            }
            return nil
        })

        // 推進虛擬時間讓重試發生（不需要真實等待）
        synctest.Wait() // 等待所有 goroutine 阻塞
        time.Sleep(100 * time.Millisecond) // 虛擬時間推進，第一次重試
        synctest.Wait()
        time.Sleep(100 * time.Millisecond) // 第二次重試
        synctest.Wait()

        if attempts != 3 {
            t.Errorf("expected 3 attempts, got %d", attempts)
        }
    })
}
```

### 與 t.Context() 整合

```go
// Go 1.25 新增 t.Context()：測試結束時自動取消的 context
func TestLongRunningJob(t *testing.T) {
    ctx := t.Context() // 測試結束（或 t.Cleanup 時）自動 cancel

    result, err := processWithTimeout(ctx, data)
    require.NoError(t, err)

    // 不需要手動建立和 cancel context
}
```

---

## 二、runtime：更小的 goroutine 初始 stack

Go 1.25 持續優化 goroutine 的記憶體佔用。goroutine 的初始 stack 從 2KB 進一步優化（依程式特性動態決定），使大量 goroutine 的應用記憶體效率更高。

```go
// 應用程式代碼無需改變
// 效果：goroutine 密集型服務（如 gRPC server、websocket hub）
// 在相同記憶體下可承載更多並發連線

// 可透過 runtime stats 觀察改善
var m runtime.MemStats
runtime.ReadMemStats(&m)
fmt.Printf("goroutine stack usage: %v bytes avg\n", m.StackInuse/uint64(runtime.NumGoroutine()))
```

---

## 三、工具鏈改善

### go test：更精確的失敗定位

```bash
# Go 1.25：failing test 輸出改善
# 顯示 subtest 的完整路徑，更容易在 CI 中定位問題

# 新增 -fullpath 旗標（顯示完整 package 路徑）
go test -v -fullpath ./...

# go test -json 的輸出格式更豐富，CI 工具（如 gotestsum）受益
go test -json ./... | gotestsum --format testdox
```

### go mod tidy 更智慧

```bash
# 能正確識別並移除只在 tool 指令中使用的間接依賴
go mod tidy

# 新增 -compat 旗標支援（維持對舊版本的相容性）
go mod tidy -compat=1.22
```

---

## 四、slices / maps / iter 套件新增函數

Go 1.25 持續擴充 1.21–1.23 引入的標準函數庫：

```go
import (
    "iter"
    "slices"
    "maps"
)

// iter 套件：更多輔助函數

// iter.Concat：合併多個迭代器（類似 append，但惰性求值）
combined := iter.Concat(slices.Values(a), slices.Values(b), slices.Values(c))
for v := range combined {
    fmt.Println(v)
}

// slices 套件新函數
// slices.Chunk：切分成固定大小的子 slice
for chunk := range slices.Chunk(data, 100) {
    process(chunk) // 每次處理 100 個元素
}

// maps 套件：Insert（批量插入）
source := map[string]int{"a": 1, "b": 2}
target := map[string]int{"c": 3}
maps.Insert(target, maps.All(source)) // 把 source 的所有 entry 插入 target
// target = {"a": 1, "b": 2, "c": 3}
```

---

## 五、crypto 套件：後量子密碼學強化

```go
// Go 1.25：X25519MLKEM768 成為 TLS 1.3 預設的 key exchange
// （從 1.24 的 X25519Kyber768Draft00 升級到標準化版本）
// NIST FIPS 203（ML-KEM）正式標準

cfg := &tls.Config{
    // 無需修改代碼：Go 自動使用最新的後量子演算法
    // 伺服器和客戶端自動協商
}

// 若需明確控制（例如合規需求）
cfg = &tls.Config{
    CurvePreferences: []tls.CurveID{
        tls.X25519MLKEM768, // 後量子 + 傳統混合
        tls.CurveX25519,    // 傳統（向後相容）
    },
}
```

---

## 六、向前看：Go 2.x 規劃中的功能

目前在討論或 proposal 階段的重大功能（非 1.25 內容，但值得關注）：

```
1. 錯誤處理改善
   - 各種 if err != nil 簡化提案仍在討論中
   - check/handle 或 ? 運算子類型的提案

2. Sum Types / 密封介面
   - Go 的 interface 目前是開放的，無法建立「只能是 A 或 B」的型別
   - 與 Rust enum / Haskell ADT 類似的功能仍在評估

3. 更完整的泛型
   - 泛型方法（parametric methods）仍不支援
   - 泛型 variadics

4. 協程（Coroutines）
   - range-over-func 是基礎，但完整的 coroutine 支援尚未確定

以上功能不確定會進入哪個版本，需追蹤 github.com/golang/go 的 proposals。
```

---

## 版本特性速查表

| 版本 | 關鍵功能 | 實作優先度 |
|------|---------|-----------|
| 1.22 | 迴圈變數修復、range 整數、net/http 路由 | 立即升級受益 |
| 1.22 | math/rand/v2 | 新代碼使用 v2 |
| 1.23 | range-over-func 迭代器、iter 套件 | 設計 collection API 時採用 |
| 1.23 | Timer/Ticker Reset 修正 | 移除舊 drain 代碼 |
| 1.23 | unique 套件 | 有大量重複字串/值的場景 |
| 1.24 | tool 指令 | 遷移 tools.go |
| 1.24 | Swiss Table Maps | 自動受益，無需改代碼 |
| 1.24 | os.Root | 有路徑安全需求的服務 |
| 1.24 | weak 套件 | 需要 GC-friendly cache 的場景 |
| 1.25 | testing/synctest | 並發組件的測試 |
| 1.25 | t.Context() | 新測試代碼 |

---

## 相關頁面

- [[Go1.22新功能實戰]] — 迴圈變數修復、net/http 路由、range-over-int
- [[Go1.23新功能實戰]] — range-over-func 完整指南
- [[Go1.24新功能實戰]] — 泛型別名、weak 套件、os.Root
- [[Go測試基準與模糊測試]] — synctest 與 fuzz testing 的定位差異
- [[Go效能調優]] — Swiss Table 效能改善的背景
