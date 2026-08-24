---
title: Senior Golang Interview Questions: Advanced Concurrency and Performance
type: source-summary
tags: [golang, performance, concurrency, GC, memory, scheduler, interview]
created: 2026-04-21
updated: 2026-04-21
sources: [2026-04-21_multi-source_golang-principal-interview.md]
---

# Senior Golang Interview Questions: Advanced Concurrency and Performance

## Origin
- **標題**: Senior Golang Interview Questions: Advanced Concurrency and Performance
- **作者**: CodeForGeek
- **URL**: https://codeforgeek.com/senior-golang-interview-questions/

## Key Takeaways
1. Go 排程器異步搶佔（Go 1.14+）：用 OS signal 中斷 tight loop，防止 goroutine 飢餓
2. Blocking system call 時，scheduler 將 M 從 P 解除，spin 新 M 保持 CPU 利用率
3. Channel 隱藏成本：unbuffered channel 每次 send 都觸發 context switch；用於 CPU-intensive 計算反而更慢
4. `sync/atomic` 的操作直接編譯為 CPU 硬體指令（lock-free），比 mutex 快幾個量級（nanoseconds vs microseconds）
5. Struct 欄位排列影響 cache line（64 bytes）：大到小排列消除 padding，熱點 atomic counter 需跨 cache line 避免 false sharing
6. GC 不壓縮 heap：需用 sync.Pool 重用 buffer 降低分配壓力；`GOGC` 預設 100
7. Escape analysis 工具：`go build -gcflags="-m"`；return pointer 是最常見的逃逸原因
8. init() 三大問題：隱藏依賴、執行順序依賴 import graph、無法回傳 error（只能 panic）；應用明確的 constructor

## Junior vs Senior Go 思維對比

| 主題 | Junior 做法 | Senior 做法 |
|------|-----------|-----------|
| Channels | 總是用 channel | 依場景選 channel/mutex/atomic |
| GC | 讓它自動管 | 調 GOGC、最小化分配 |
| Context | 傳資料用 | Deadline/cancellation 樹狀結構 |
| Pointers | 傳 reference | 最小化 heap escape |
| init() | 方便的初始化 | 明確依賴注入 |

## Concepts mentioned
- [[Go執行期內部機制]] — 排程器、GC、struct padding、escape analysis
- [[Go並發模式]] — channels 成本、atomic vs mutex 決策
- [[Go效能調優]] — sync.Pool、GOGC、pprof profiling

## Contradictions/tensions
- Go 語言哲學「share memory by communicating」（channel），但效能極限場景下 atomic 才是正解

## Questions raised
- 如何在大型 monorepo 中系統性地消除不必要的 heap escape？
