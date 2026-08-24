# Golang/Backend Principal Engineer 面試準備（5篇文章合成）

- **收錄日期**: 2026-04-21
- **語言**: 英文為主

## 來源列表

1. https://www.secondtalent.com/interview-guide/golang/ — 23 Advanced Golang Backend Interview Questions
2. https://www.onsites.fyi/blog/article/google-L6-software-engineer-interview-questions — Google L6 Staff Engineer Interview Guide
3. https://codeforgeek.com/senior-golang-interview-questions/ — Senior Golang Interview: Advanced Concurrency & Performance
4. https://newsletter.eng-leadership.com/p/how-to-nail-big-tech-behavioral-interviews — How to Nail Big Tech Behavioral Interviews as Senior Engineer
5. https://leapcell.io/blog/go-production-performance-tips — Go in Production: 20 Must-Know Performance Tuning Tips
6. https://www.linkedin.com/pulse/prepping-partnersenior-staffprincipal-ic-interviews-brett-flegg — Prepping for Partner/Senior Staff/Principal IC Interviews

---

## Principal Engineer vs Senior Engineer 核心差異

| 維度 | Senior (L5) | Principal/Staff (L6+) |
|------|------------|----------------------|
| 所有權 | Feature/Subsystem | 整個 Platform |
| 影響範圍 | Team | 多 Team/整個 Org |
| 設計焦點 | 局部架構 | 組織級技術方向 |
| 領導力 | Team Delivery | 長期方向設定 |
| Coding 面試 | 核心關卡 | 重要但非主要決定因素 |
| 系統設計 | 重要 | 佔 hiring decision 60%+ |
| Behavioral | 加分 | 決定 leveling |

## Go Runtime 核心面試知識點

### GMP 排程器
- G (Goroutine)：輕量執行單元，初始 stack 僅 2-8KB
- M (Machine)：OS thread，執行實際代碼
- P (Processor)：執行上下文，數量等於 GOMAXPROCS
- blocking system call 時：scheduler 將 M 從 P 解除，spin 新 M 繼續執行其他 G
- Go 1.14+ 異步搶佔（asynchronous preemption）：用 OS signal 中斷 tight loop

### GC 機制
- 三色標記清除（tri-color mark-and-sweep），並發執行
- GOGC 預設 100（heap 翻倍時觸發）；調高可減少 GC 頻率但增加記憶體用量
- STW (stop-the-world) pause < 1ms
- 不壓縮 heap（no compaction）→ 需手動用 sync.Pool 重用物件

### Escape Analysis
- `go build -gcflags="-m"` 可查看哪些變數逃逸到 heap
- Stack 分配：函式回傳後立即回收，零 GC 開銷
- 常見逃逸原因：return pointer、closure 捕獲、interface{} 轉換

## Production 效能調優 20 Tips 摘要

1. 先量測（pprof）再優化
2. 用 benchmark 建立基準
3. slice/map 預分配容量
4. sync.Pool 重用短命物件
5. strings.Builder 取代 + 拼接
6. sub-slicing 後 copy() 防記憶體洩漏
7. 大 struct 傳 pointer，小 struct 傳 value
8. 容器環境用 uber-go/automaxprocs
9. buffered channel 解耦 producer/consumer
10. sync.WaitGroup 協調並發任務
11. 減少 lock contention（RWMutex、atomic、sharding）
12. Worker Pool 控制並發上限
13. map[key]struct{} 實作 Set（零記憶體）
14. hot loop 提取常數計算
15. 避免 interface dispatch 在 critical path
16. binary 發布加 -ldflags="-s -w"
17. 用 escape analysis 了解分配行為
18. 最小化 cgo 呼叫
19. PGO（Profile-Guided Optimization，Go 1.21+）
20. 保持 Go 版本最新
