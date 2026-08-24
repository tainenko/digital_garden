---
title: "SD題解：訂票系統（Ticketmaster）"
type: topic
tags: [system-design, ticketing, concurrency, optimistic-locking, golang, medium]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：訂票系統（Ticketmaster）

> **難度**: 中級 ｜ **頻率**: 高 ｜ **代表**: Ticketmaster, 演唱會搶票, 12306

---

## RESHADED 快速分析

**R - Requirements**
- 功能：瀏覽活動、選位、預訂、付款
- 非功能：高並發下防止超賣（最核心！）、強一致性、低延遲

**E - Estimation**（大型演唱會開賣）
- 座位：10萬個
- 開賣瞬間 QPS：**100萬+**（明星效應）
- 但實際可成交：10萬次/開賣時段

挑戰：瞬間 100 萬請求搶 10 萬張票，需要正確拒絕 90 萬個請求。

---

## 核心問題：Race Condition

如果兩個用戶同時訂同一個座位：

```
User A 讀: seat_123 = available
User B 讀: seat_123 = available
User A 寫: seat_123 = reserved (by A)
User B 寫: seat_123 = reserved (by B)  ← 超賣！
```

---

## 解法比較

| 方案 | 原理 | 適用 |
|------|------|------|
| 悲觀鎖（DB FOR UPDATE）| 讀取時鎖定，其他人等待 | 衝突率高 |
| 樂觀鎖（Version 欄位）| 提交時比較版本，失敗重試 | 衝突率低 |
| Redis 原子操作（SETNX）| Redis 層原子佔位，再非同步寫 DB | 高並發首選 |
| 排隊 + Queue | 請求進 Queue，依序處理 | 超高並發（搶票系統）|

**實際大型訂票系統**：排隊 + Redis 原子佔位 + DB 最終一致

---

## Go 實現

### 樂觀鎖（Optimistic Locking）

```go
package ticketing

import (
    "context"
    "errors"
    "time"
)

type Seat struct {
    ID        string
    EventID   string
    SeatNo    string
    Status    string // available / reserved / sold
    ReservedBy string
    Version   int64  // 樂觀鎖版本號
    UpdatedAt time.Time
}

var (
    ErrSeatNotAvailable = errors.New("seat not available")
    ErrVersionConflict  = errors.New("concurrent update conflict, please retry")
)

type TicketDB interface {
    GetSeat(ctx context.Context, seatID string) (*Seat, error)
    UpdateSeatWithVersion(ctx context.Context, seat *Seat, expectedVersion int64) (bool, error)
}

// ReserveWithOptimisticLock 樂觀鎖搶票
func ReserveWithOptimisticLock(ctx context.Context, db TicketDB, seatID, userID string) error {
    maxRetries := 3
    for attempt := 0; attempt < maxRetries; attempt++ {
        // 1. 讀取座位資訊（帶版本號）
        seat, err := db.GetSeat(ctx, seatID)
        if err != nil {
            return err
        }

        // 2. 檢查是否可預訂
        if seat.Status != "available" {
            return ErrSeatNotAvailable
        }

        // 3. 嘗試更新（帶版本檢查）
        updated := &Seat{
            ID:         seat.ID,
            EventID:    seat.EventID,
            SeatNo:     seat.SeatNo,
            Status:     "reserved",
            ReservedBy: userID,
            Version:    seat.Version + 1,
            UpdatedAt:  time.Now(),
        }

        // SQL: UPDATE seats SET status='reserved', version=version+1
        //      WHERE id=? AND version=?（版本不符則不更新）
        success, err := db.UpdateSeatWithVersion(ctx, updated, seat.Version)
        if err != nil {
            return err
        }
        if success {
            return nil // 搶票成功
        }
        // 版本衝突，重試
    }
    return ErrVersionConflict
}
```

### Redis 原子佔位（高並發首選）

```go
type TicketService struct {
    cache *redis.Client
    db    TicketDB
    queue MessageQueue // Kafka
}

// ReserveWithRedis Redis SETNX 原子佔位
func (s *TicketService) ReserveWithRedis(ctx context.Context, seatID, userID string) error {
    lockKey := "seat_lock:" + seatID
    reserveKey := "seat:" + seatID

    // 1. 分散式鎖（防止同一張票被處理兩次）
    locked, err := s.cache.SetNX(ctx, lockKey, userID, 10*time.Second).Result()
    if err != nil {
        return err
    }
    if !locked {
        return ErrSeatNotAvailable // 已被他人佔位
    }
    defer s.cache.Del(ctx, lockKey) // 確保鎖被釋放

    // 2. 檢查 Redis 中的即時狀態
    val, _ := s.cache.Get(ctx, reserveKey).Result()
    if val != "" {
        return ErrSeatNotAvailable
    }

    // 3. 設定 15 分鐘暫保留（用戶付款時間）
    s.cache.Set(ctx, reserveKey, userID, 15*time.Minute)

    // 4. 非同步寫入 DB（最終一致）
    go s.queue.Publish("seat-reservation", ReservationEvent{
        SeatID: seatID,
        UserID: userID,
        TTL:    15 * time.Minute,
    })

    return nil
}

// ConfirmPayment 付款成功，確認訂票
func (s *TicketService) ConfirmPayment(ctx context.Context, seatID, userID string) error {
    reserveKey := "seat:" + seatID

    // 確認座位仍然被此用戶佔用
    owner, err := s.cache.Get(ctx, reserveKey).Result()
    if err != nil || owner != userID {
        return errors.New("reservation expired or invalid")
    }

    // 更新為永久 sold 狀態
    s.cache.Set(ctx, reserveKey, "sold:"+userID, 0) // 不過期
    s.db.ConfirmSeat(ctx, seatID, userID)
    return nil
}
```

### 排隊系統（Waiting Room）

```go
// WaitingRoom 高峰期讓用戶排隊，依序放入
type WaitingRoom struct {
    cache *redis.Client
}

// EnterQueue 進入等待隊列，回傳排隊位置
func (w *WaitingRoom) EnterQueue(ctx context.Context, userID, eventID string) (int64, error) {
    queueKey := "queue:" + eventID
    // ZADD score=時間戳（先到先服務）
    score := float64(time.Now().UnixMilli())
    w.cache.ZAdd(ctx, queueKey, redis.Z{Score: score, Member: userID})
    // 取得排隊位置
    rank, err := w.cache.ZRank(ctx, queueKey, userID).Result()
    return rank + 1, err
}

// GetQueuePosition 查詢當前排隊位置
func (w *WaitingRoom) GetQueuePosition(ctx context.Context, userID, eventID string) int64 {
    rank, _ := w.cache.ZRank(ctx, context.Background(), "queue:"+eventID, userID).Result()
    return rank + 1
}

// PopFromQueue 每秒放 N 個用戶進入搶票
func (w *WaitingRoom) PopFromQueue(ctx context.Context, eventID string, batchSize int64) []string {
    queueKey := "queue:" + eventID
    // 取排在最前面的 N 個用戶
    members, _ := w.cache.ZRange(ctx, queueKey, 0, batchSize-1).Result()
    // 從隊列移除
    w.cache.ZRemRangeByRank(ctx, queueKey, 0, batchSize-1)
    return members
}
```

---

## 架構圖

```
用戶請求 → Rate Limiter → Waiting Room（排隊）
                               ↓（每秒放 N 人）
                          搶票 API Server
                               ↓
                     Redis（SETNX 原子佔位）
                               ↓
                          Kafka（非同步）
                               ↓
                          DB（最終確認）
```

---

## Trade-offs

| 決策 | 選擇 | 理由 |
|------|------|------|
| 並發控制 | Redis SETNX | 原子操作，比 DB 樂觀鎖快100倍 |
| 暫保留時間 | 15 分鐘 | 太短用戶付款來不及；太長佔位不付款浪費 |
| 超高峰 | Waiting Room 排隊 | 保護後端不被壓垮，用戶體驗更好 |
| DB 一致性 | 最終一致（Kafka 非同步）| 搶票判斷在 Redis，DB 只是持久化 |

---

## 相關題解

- [[sd-rate-limiter|SD題解：Rate Limiter]] — 防止刷票機器人
- [[sd-distributed-cache|SD題解：分散式快取]] — Redis 核心用法
