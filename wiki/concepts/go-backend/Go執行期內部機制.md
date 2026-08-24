---
title: Go執行期內部機制
type: concept
tags: [golang, runtime, scheduler, GMP, GC, escape-analysis, memory]
created: 2026-04-21
updated: 2026-04-21
sources: [golang-advanced-interview-secondtalent, golang-perf-advanced-codeforgeek, go-production-perf-20tips]
---

# Go執行期內部機制

Principal-level Go 面試必考核心：排程器、GC、記憶體模型的深度理解。

---

## GMP 排程模型

| 元素 | 全名 | 說明 |
|------|------|------|
| **G** | Goroutine | 輕量執行單元，初始 stack 2-8KB，可長到 GB |
| **M** | Machine | OS thread，真正執行代碼 |
| **P** | Processor | 執行上下文，數量 = `GOMAXPROCS`（預設 = CPU 核數）|

### 關鍵行為
- **Blocking system call**：scheduler 將 M 從 P 解除（hand-off），spin 新 M 讓 P 繼續執行其他 G
- **異步搶佔（Go 1.14+）**：用 OS signal (`SIGURG`) 中斷 tight loop，防止 goroutine 飢餓
- **Work stealing**：P 的 local run queue 空了，會從其他 P 的 queue 偷 G

```
P0: [G1, G2, G3] → M0 執行 G1
P1: []            ← 從 P0 steal G3
```

### 調優
```bash
# 容器環境必用（避免讀錯 CPU 數）
go get go.uber.org/automaxprocs
```

---

## 垃圾回收（GC）

### 算法
- **三色標記清除**（tri-color mark-and-sweep）
- **並發執行**：GC 與程式同時跑，STW pause < 1ms
- **不壓縮 heap**（no compaction）——避免大停頓，但會有記憶體碎片

### GOGC 調優
```bash
# 預設 100：heap 翻倍時觸發 GC
GOGC=100  # conservative
GOGC=300  # 允許 heap 長到 3x 才 GC，降低頻率 + 增加記憶體用量
GOGC=off  # 完全關閉（測試用）
```

實際案例：API gateway 100K QPS，GOGC 100→300 使 GC 頻率降 30%，記憶體增加 20%。

### sync.Pool 降低 GC 壓力
```go
var bufPool = sync.Pool{
    New: func() any { return new(bytes.Buffer) },
}

func handle(data []byte) {
    buf := bufPool.Get().(*bytes.Buffer)
    defer func() {
        buf.Reset()
        bufPool.Put(buf)
    }()
    buf.Write(data)
}
```

注意：Pool 中的物件可能隨時被 GC 清空，只適合無狀態、可重建的物件。

---

## Escape Analysis（逃逸分析）

編譯期決定變數分配在 stack（快）還是 heap（GC 負擔）。

```bash
go build -gcflags="-m" ./...
# 輸出: ./main.go:10:9: &x escapes to heap
```

### 常見逃逸原因
1. Return pointer（最常見）
2. 賦值給 interface{}
3. Closure 捕獲變數
4. 傳給 goroutine（heap 生命週期不確定）

---

## Struct Memory Layout

現代 CPU 以 **64-byte cache line** 為單位讀取記憶體：

```go
// 低效：有 padding，浪費 7 bytes
type Bad struct {
    a bool   // 1 byte + 7 bytes padding
    b int64  // 8 bytes
}

// 高效：無 padding
type Good struct {
    b int64  // 8 bytes
    a bool   // 1 byte
}
```

False sharing 問題：多個 goroutine 寫不同 counter 但在同一 cache line，解法是 padding 讓每個 counter 獨佔一個 cache line。

---

## 相關頁面
- [[Go並發模式]] — 基於 runtime 機制的並發設計
- [[Go效能調優]] — 實戰調優工具與技巧
- [[golang-principal-interview|Golang Principal Engineer 面試完整指南]]
