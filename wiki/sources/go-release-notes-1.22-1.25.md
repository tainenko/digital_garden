---
title: Go 1.22–1.25 Release Notes 實戰整理
type: source-summary
tags: [golang, go1.22, go1.23, go1.24, go1.25, release-notes]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Go 1.22–1.25 Release Notes 實戰整理

## Origin

- **類型**：合成知識（Release Notes + 工程實戰整理）
- **官方來源**：
  - https://go.dev/doc/go1.22
  - https://go.dev/doc/go1.23
  - https://go.dev/doc/go1.24
  - https://go.dev/doc/go1.25（預計 2025 年 8 月）
- **涵蓋期間**：2024 年 2 月 – 2025 年 8 月
- **整理日期**：2026-04-30

## Key Takeaways

- **Go 1.22（2024.02）迴圈變數修復** 是影響最廣的 breaking change：每次迭代的變數現在有獨立記憶體，原本用 `i := i` 的 goroutine 閉包 workaround 可以全面移除；同時加入 range-over-integers 語法糖
- **Go 1.22 net/http 路由增強** 使大多數應用不再需要 `gorilla/mux`：支援 `GET /users/{id}` 方法+路徑模式、`r.PathValue("id")` 取參數、`{path...}` 萬用符和 `{$}` 精確匹配
- **Go 1.23（2024.08）range-over-func** 是自泛型以來最大的語言變化：`for v := range someFunc` 中 someFunc 是 `func(yield func(V) bool)` 類型，配合 `iter.Seq[V]` / `iter.Seq2[K,V]` 讓 Go 有了一等公民的迭代器協議
- **Go 1.23 iter.Pull** 把推送式迭代器轉為拉取式，使 Zip、Merge 等雙迭代器操作成為可能；`slices.All`、`maps.Keys`、`maps.Values` 等標準庫函數回傳迭代器
- **Go 1.23 unique 套件** 提供值的 interning（規範化）：`unique.Make(v)` 對相同內容只存一份，Handle 比較是 O(1) 指標比較；`time.Timer`/`time.Ticker` 的 Reset 語意修正，消除多年來的 drain-before-reset 陷阱
- **Go 1.24（2025.02）泛型型別別名穩定** 補齊了 1.18 遺漏的功能：`type Set[K comparable] = map[K]struct{}` 現在合法；`tool` 指令統一管理開發工具版本，取代 `tools.go` hack
- **Go 1.24 Swiss Table Map** 讓查詢快 30–40%、插入快 20–30%，應用程式無需改代碼自動受益；`os.Root` 提供目錄沙盒防止路徑遍歷攻擊；`weak.Pointer[T]` 實現 GC-friendly 弱引用 cache
- **Go 1.24 testing/synctest**（實驗性）解決了時間敏感的並發測試問題：使用虛擬時鐘，`time.Sleep` 在測試中推進虛擬時間而非真實等待
- **Go 1.25（2025.08）** 持續鞏固：`testing/synctest` 持續完善、`t.Context()` 提供測試生命週期 context、`iter.Concat`/`slices.Chunk` 等標準庫擴充、後量子 TLS 升級到 ML-KEM 正式標準
- **升級優先度**：1.22 迴圈變數修復和 net/http 路由應立即採用；1.23 iterator 在設計 collection API 時採用；1.24 tool 指令立即遷移；synctest 在需要測試 timer/timeout 邏輯時引入

## Concepts Mentioned

- [[Go1.22新功能實戰]] — 完整實戰代碼：迴圈變數、range-int、net/http 路由、math/rand/v2
- [[Go1.23新功能實戰]] — range-over-func 深度指南、iter 套件、unique、Timer 修正
- [[Go1.24新功能實戰]] — 泛型別名、weak、os.Root、synctest、Swiss Table、tool 指令
- [[Go1.25新功能實戰]] — synctest 穩定化、t.Context()、iter.Concat、後量子 TLS

## Contradictions / Tensions

- **迴圈變數修復的 break change**：極少數依賴舊語意（在函數結束後讀迴圈變數、或 intentionally 共用變數）的代碼會悄悄改變行為；`go vet` 可以在升級前偵測
- **range-over-func 的性能**：inlining 後幾乎無 overhead，但調試時 stack trace 中會出現 yield 函數，初期可能令人困惑
- **os.Root 的 symlink 行為**：symlink 指向 root 外部時行為因 OS 不同而略有差異，需測試
- **weak.Pointer 的 GC 時機**：物件的回收時機依賴 GC 節奏，不應依賴「何時被回收」的確定性

## Questions Raised

- range-over-func 的 coroutine 語意是否會在未來版本進一步擴展（例如 generator/yield 關鍵字）？
- `testing/synctest` 正式穩定後，是否會與 `-race` 偵測整合？
- Go 的錯誤處理提案（`?` 運算子、check/handle）何時會有定論？
