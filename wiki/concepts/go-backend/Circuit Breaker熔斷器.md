---
title: Circuit Breaker 熔斷器
type: concept
tags: [circuit-breaker, resilience, microservices, golang, fault-tolerance]
created: 2026-04-29
updated: 2026-04-29
---

# Circuit Breaker 熔斷器

## 解決什麼問題

微服務 A 呼叫服務 B，如果 B 掛掉或很慢，A 會不斷等待，最終耗盡所有連線池和 goroutine，導致 A 也掛掉——**故障級聯（cascading failure）**。

Circuit Breaker 在感知到 B 異常後，立即「斷路」：後續對 B 的呼叫直接返回錯誤，不再真正發出請求。等 B 恢復後再「閉路」重新允許呼叫。

## 三個狀態

```
                失敗率超過閾值
Closed（閉路）──────────────→ Open（斷路）
   ↑                              │
   │ 探測成功                      │ 等待 timeout
   │                              ↓
   └──────────────────── Half-Open（半開）
                             │
                             │ 探測失敗
                             ↓
                          Open（重新斷路）
```

- **Closed**：正常狀態，所有請求通過，統計失敗率
- **Open**：斷路狀態，所有請求直接 fast-fail，不觸達下游
- **Half-Open**：嘗試性放行少量請求，測試下游是否恢復

## Go 實現（從零開始）

```go
package circuitbreaker

import (
    "errors"
    "sync"
    "time"
)

type State int

const (
    StateClosed   State = iota // 正常
    StateOpen                  // 斷路
    StateHalfOpen             // 半開
)

var ErrCircuitOpen = errors.New("circuit breaker is open")

type CircuitBreaker struct {
    mu sync.Mutex

    // 設定
    maxFailures  int           // 達到此失敗次數後斷路
    timeout      time.Duration // 斷路後多久嘗試半開
    maxHalfOpen  int           // 半開狀態最多允許幾個請求

    // 狀態
    state        State
    failures     int
    successes    int
    halfOpenCount int
    lastFailure  time.Time
}

func New(maxFailures int, timeout time.Duration) *CircuitBreaker {
    return &CircuitBreaker{
        maxFailures: maxFailures,
        timeout:     timeout,
        maxHalfOpen: 3,
        state:       StateClosed,
    }
}

func (cb *CircuitBreaker) Execute(fn func() error) error {
    cb.mu.Lock()
    state := cb.currentState()

    switch state {
    case StateOpen:
        cb.mu.Unlock()
        return ErrCircuitOpen

    case StateHalfOpen:
        if cb.halfOpenCount >= cb.maxHalfOpen {
            cb.mu.Unlock()
            return ErrCircuitOpen
        }
        cb.halfOpenCount++
        cb.mu.Unlock()

    case StateClosed:
        cb.mu.Unlock()
    }

    // 執行實際操作
    err := fn()

    cb.mu.Lock()
    defer cb.mu.Unlock()

    if err != nil {
        cb.onFailure()
        return err
    }
    cb.onSuccess()
    return nil
}

func (cb *CircuitBreaker) currentState() State {
    if cb.state == StateOpen {
        if time.Since(cb.lastFailure) > cb.timeout {
            cb.state = StateHalfOpen
            cb.halfOpenCount = 0
        }
    }
    return cb.state
}

func (cb *CircuitBreaker) onFailure() {
    cb.failures++
    cb.lastFailure = time.Now()

    if cb.state == StateHalfOpen || cb.failures >= cb.maxFailures {
        cb.state = StateOpen
        cb.failures = 0
        cb.halfOpenCount = 0
    }
}

func (cb *CircuitBreaker) onSuccess() {
    cb.failures = 0
    if cb.state == StateHalfOpen {
        cb.successes++
        if cb.successes >= cb.maxHalfOpen {
            cb.state = StateClosed
            cb.successes = 0
        }
    }
}
```

## 使用 sony/gobreaker（生產推薦）

```go
import "github.com/sony/gobreaker"

var cb *gobreaker.CircuitBreaker

func init() {
    cb = gobreaker.NewCircuitBreaker(gobreaker.Settings{
        Name:        "payment-service",
        MaxRequests: 5,    // Half-Open 狀態最多放行 5 個請求
        Interval:    10 * time.Second, // Closed 狀態計數視窗
        Timeout:     30 * time.Second, // Open → Half-Open 等待時間

        ReadyToTrip: func(counts gobreaker.Counts) bool {
            // 失敗率 > 60%，且至少有 5 次請求才斷路
            return counts.Requests >= 5 &&
                float64(counts.TotalFailures)/float64(counts.Requests) >= 0.6
        },

        OnStateChange: func(name string, from, to gobreaker.State) {
            log.Printf("circuit breaker %s: %s → %s", name, from, to)
            // 可以發 metric 或 alert
        },
    })
}

func CallPaymentService(req PaymentRequest) (*PaymentResponse, error) {
    result, err := cb.Execute(func() (interface{}, error) {
        // 實際的 HTTP 或 gRPC 呼叫
        return paymentClient.Charge(req)
    })
    if err != nil {
        if errors.Is(err, gobreaker.ErrOpenState) {
            // 斷路狀態：快速失敗，可以走 fallback
            return fallbackPayment(req), nil
        }
        return nil, err
    }
    return result.(*PaymentResponse), nil
}
```

## 搭配 Fallback（降級策略）

```go
func GetProductRecommendations(userID string) []Product {
    result, err := cb.Execute(func() (interface{}, error) {
        return recommendationService.Get(userID)
    })

    if err != nil {
        // Fallback：推薦服務掛掉時，回傳熱門商品（靜態快取）
        log.Warnf("recommendation service unavailable, using fallback: %v", err)
        return getPopularProducts() // 從 Redis 快取取
    }

    return result.([]Product)
}
```

## 進階：滑動視窗（Sliding Window）

gobreaker 用計數視窗（Interval 內重置）。更精準的是時間滑動視窗：

```go
// 用環形陣列實作最近 N 秒的失敗率統計
type SlidingWindow struct {
    buckets  []int64
    size     int
    interval time.Duration
    cursor   int
    mu       sync.Mutex
}

func NewSlidingWindow(size int, interval time.Duration) *SlidingWindow {
    return &SlidingWindow{
        buckets:  make([]int64, size),
        size:     size,
        interval: interval,
    }
}

func (sw *SlidingWindow) Increment() {
    sw.mu.Lock()
    defer sw.mu.Unlock()
    sw.buckets[sw.cursor]++
}

func (sw *SlidingWindow) Total() int64 {
    sw.mu.Lock()
    defer sw.mu.Unlock()
    var total int64
    for _, v := range sw.buckets {
        total += v
    }
    return total
}
```

## 配置建議

| 場景 | MaxRequests | Timeout | ReadyToTrip |
|------|------------|---------|-------------|
| 資料庫呼叫 | 3 | 60s | 失敗率 > 50% |
| 外部 API | 5 | 30s | 失敗率 > 60% |
| 內部 gRPC | 10 | 15s | 失敗率 > 40% |
| 非關鍵服務 | 3 | 10s | 失敗率 > 80% |

## 相關頁面

- [[微服務架構設計原則]] — 故障隔離設計
- [[服務發現與負載均衡]] — 與熔斷器配合使用
- [[OpenTelemetry分散式追蹤]] — 監控熔斷器狀態
- [[Go優雅關機與健康檢查]] — 服務下線的優雅處理
