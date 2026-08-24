---
title: "SD題解：Rate Limiter（速率限制）"
type: topic
tags: [system-design, rate-limiter, token-bucket, sliding-window, golang]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：Rate Limiter（速率限制）

> **難度**: 概念題 ｜ **頻率**: 極高（幾乎必考概念）

---

## 題目

設計一個 Rate Limiter，限制每個用戶每秒最多 N 次 API 請求。

---

## 快速需求分析

**Functional:**
- 每個用戶有獨立的請求配額
- 超過限制回傳 429 Too Many Requests
- 支援多種限制粒度（per second / per minute / per IP）

**Non-functional (SPARCS):**
- 低延遲（<1ms 判斷，不能讓 Rate Limiter 本身成為瓶頸）
- 高可用（Rate Limiter 掛掉不能讓整個服務掛掉）
- 分散式環境下一致（多台 server 共享狀態）

---

## 四種演算法比較

| 演算法 | 原理 | 優點 | 缺點 |
|--------|------|------|------|
| **Fixed Window** | 固定時間窗口計數 | 實作最簡單 | 邊界突刺問題 |
| **Sliding Window Log** | 記錄每次請求時間戳 | 精確 | 記憶體消耗大 |
| **Sliding Window Counter** | 用前後窗口加權估算 | 省記憶體、近似精確 | 微小誤差 |
| **Token Bucket** | 令牌桶，持續補充令牌 | 允許短暫突發 | 實作稍複雜 |
| **Leaky Bucket** | 固定速率流出 | 輸出穩定 | 不允許突發 |

**面試首選**：Token Bucket（最常被問）或 Sliding Window Counter（Redis 原生支援）

---

## 架構設計

```
Client → API Gateway → Rate Limiter Middleware → Backend Service
                           ↓
                        Redis（共享計數器）
```

**為什麼用 Redis？**
- 原子操作（INCR / EXPIRE）
- 多台 API Server 共享同一個計數狀態
- 記憶體快取，延遲 <1ms

---

## Go 實現

### 1. Token Bucket（令牌桶）

```go
package ratelimiter

import (
    "sync"
    "time"
)

type TokenBucket struct {
    capacity     float64   // 桶的最大容量
    tokens       float64   // 當前令牌數
    refillRate   float64   // 每秒補充令牌數
    lastRefillAt time.Time
    mu           sync.Mutex
}

func NewTokenBucket(capacity, refillRate float64) *TokenBucket {
    return &TokenBucket{
        capacity:     capacity,
        tokens:       capacity, // 初始滿桶
        refillRate:   refillRate,
        lastRefillAt: time.Now(),
    }
}

func (tb *TokenBucket) Allow() bool {
    tb.mu.Lock()
    defer tb.mu.Unlock()

    tb.refill()

    if tb.tokens >= 1 {
        tb.tokens--
        return true
    }
    return false
}

func (tb *TokenBucket) refill() {
    now := time.Now()
    elapsed := now.Sub(tb.lastRefillAt).Seconds()
    tb.tokens = min(tb.capacity, tb.tokens+elapsed*tb.refillRate)
    tb.lastRefillAt = now
}

func min(a, b float64) float64 {
    if a < b {
        return a
    }
    return b
}
```

### 2. Sliding Window Counter（Redis 版，分散式）

```go
package ratelimiter

import (
    "context"
    "fmt"
    "time"

    "github.com/redis/go-redis/v9"
)

type SlidingWindowLimiter struct {
    client   *redis.Client
    limit    int64
    window   time.Duration
}

func NewSlidingWindowLimiter(client *redis.Client, limit int64, window time.Duration) *SlidingWindowLimiter {
    return &SlidingWindowLimiter{
        client: client,
        limit:  limit,
        window: window,
    }
}

// Allow 判斷 userID 是否允許請求
// 原理：用兩個固定窗口的計數，加權估算當前滑動窗口的請求數
func (s *SlidingWindowLimiter) Allow(ctx context.Context, userID string) (bool, error) {
    now := time.Now()
    windowSec := int64(s.window.Seconds())

    // 當前窗口和前一個窗口的 key
    currentWindow := now.Unix() / windowSec
    prevWindow := currentWindow - 1

    currentKey := fmt.Sprintf("rl:%s:%d", userID, currentWindow)
    prevKey := fmt.Sprintf("rl:%s:%d", userID, prevWindow)

    pipe := s.client.Pipeline()
    currentCount := pipe.Get(ctx, currentKey)
    prevCount := pipe.Get(ctx, prevKey)
    pipe.Exec(ctx)

    curr, _ := currentCount.Int64()
    prev, _ := prevCount.Int64()

    // 當前窗口已過去的比例
    elapsedRatio := float64(now.Unix()%windowSec) / float64(windowSec)

    // 加權估算：前窗口剩餘比例 * 前窗口計數 + 當前窗口計數
    estimatedCount := float64(prev)*(1-elapsedRatio) + float64(curr)

    if int64(estimatedCount) >= s.limit {
        return false, nil
    }

    // 增加計數並設置過期時間
    pipe2 := s.client.Pipeline()
    pipe2.Incr(ctx, currentKey)
    pipe2.Expire(ctx, currentKey, s.window*2)
    _, err := pipe2.Exec(ctx)

    return true, err
}
```

### 3. HTTP Middleware 封裝

```go
package middleware

import (
    "net/http"
)

func RateLimitMiddleware(limiter *SlidingWindowLimiter) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            userID := r.Header.Get("X-User-ID")
            if userID == "" {
                userID = r.RemoteAddr // fallback 到 IP
            }

            allowed, err := limiter.Allow(r.Context(), userID)
            if err != nil {
                // Rate Limiter 錯誤時 fail open（允許通過），避免拖垮服務
                next.ServeHTTP(w, r)
                return
            }

            if !allowed {
                w.Header().Set("Retry-After", "1")
                w.Header().Set("X-RateLimit-Limit", "100")
                http.Error(w, "Too Many Requests", http.StatusTooManyRequests)
                return
            }

            next.ServeHTTP(w, r)
        })
    }
}
```

---

## 分散式環境的進階問題

**Race Condition：** 多台 server 同時讀到 count=99 → 都 +1 → 實際計數 101

解法：用 Redis Lua Script 保證原子性：

```go
var luaScript = redis.NewScript(`
    local current = redis.call("INCR", KEYS[1])
    if current == 1 then
        redis.call("EXPIRE", KEYS[1], ARGV[1])
    end
    if current > tonumber(ARGV[2]) then
        redis.call("DECR", KEYS[1])
        return 0
    end
    return 1
`)

func (s *SlidingWindowLimiter) AllowAtomic(ctx context.Context, userID string) (bool, error) {
    key := fmt.Sprintf("rl:%s", userID)
    result, err := luaScript.Run(ctx, s.client, []string{key},
        int64(s.window.Seconds()), s.limit).Int64()
    return result == 1, err
}
```

---

## Trade-offs 辯護（面試必說）

| 決策 | 選擇 | 理由 |
|------|------|------|
| 演算法 | Sliding Window Counter | 記憶體 O(1)，比 Log 省，比 Fixed Window 精確 |
| 儲存 | Redis | 分散式共享狀態，原子操作，<1ms |
| 錯誤處理 | Fail Open | Rate Limiter 故障不能讓業務中斷 |
| 粒度 | Per User ID | 比 Per IP 更精確（NAT 後多人共用 IP）|

---

## 相關概念

- [[系統設計核心技術棧]] — Redis 原子操作
- [[分散式系統基礎概念]] — 分散式一致性問題
- [[系統設計面試模板]] — 完整面試流程
