---
title: Order Management（Crypto Order Management System）
type: concept
tags: [CodeSignal, Coinbase, OA, Golang, HashMap, StateMachine, Sharding]
created: 2026-05-04
updated: 2026-05-04
sources: []
---

# Order Management（Crypto Order Management System）

## 題目說明

**難度**：Medium-Hard｜**類型**：Coding OA / Onsite

實作 in-memory 加密貨幣訂單管理系統，所有操作平均 O(1)，支援最多 200,000 條指令。

---

## 訂單結構

| 欄位 | 說明 |
|------|------|
| `orderId` | 唯一字串，最長 64 字元 |
| `symbol` | 如 `"BTC-USD"` |
| `side` | `"BUY"` 或 `"SELL"` |
| `qty` | 正整數 |
| `state` | `LIVE`（初始）→ `PAUSED`→ `CANCELLED` |

---

## Level 1 — 基本狀態機

**指令集**：

| 指令 | 說明 | 合法前置狀態 |
|------|------|-------------|
| `CREATE orderId symbol side qty` | 建立訂單，state = LIVE | 不存在 |
| `PAUSE orderId` | LIVE → PAUSED | LIVE |
| `RESUME orderId` | PAUSED → LIVE | PAUSED |
| `CANCEL orderId` | 任意 → CANCELLED | LIVE / PAUSED |
| `GET orderId` | 輸出 `<orderId> <state>` | 任意 |
| `COUNT state` | 輸出該 state 的訂單數 | — |

**ERROR 條件**：
- CREATE 重複 `orderId`
- 未知 `orderId`
- 非法狀態轉換（如 PAUSE 一個 PAUSED 的訂單）
- `side` 非 BUY/SELL
- `qty` 非正整數
- COUNT 的 state 非法

```go
type Order struct {
    ID, Symbol, Side string
    Qty              int
    State            string // "LIVE" | "PAUSED" | "CANCELLED"
}

type OrderSystem struct {
    orders     map[string]*Order
    stateCounts map[string]int
}

func NewOrderSystem() *OrderSystem {
    return &OrderSystem{
        orders:      make(map[string]*Order),
        stateCounts: map[string]int{"LIVE": 0, "PAUSED": 0, "CANCELLED": 0},
    }
}

func (s *OrderSystem) Create(id, symbol, side string, qty int) string {
    if _, exists := s.orders[id]; exists {
        return "ERROR"
    }
    if side != "BUY" && side != "SELL" || qty <= 0 {
        return "ERROR"
    }
    s.orders[id] = &Order{ID: id, Symbol: symbol, Side: side, Qty: qty, State: "LIVE"}
    s.stateCounts["LIVE"]++
    return ""
}

func (s *OrderSystem) Pause(id string) string {
    o, ok := s.orders[id]
    if !ok || o.State != "LIVE" {
        return "ERROR"
    }
    s.stateCounts["LIVE"]--
    s.stateCounts["PAUSED"]++
    o.State = "PAUSED"
    return ""
}

func (s *OrderSystem) Resume(id string) string {
    o, ok := s.orders[id]
    if !ok || o.State != "PAUSED" {
        return "ERROR"
    }
    s.stateCounts["PAUSED"]--
    s.stateCounts["LIVE"]++
    o.State = "LIVE"
    return ""
}

func (s *OrderSystem) Cancel(id string) string {
    o, ok := s.orders[id]
    if !ok || o.State == "CANCELLED" {
        return "ERROR"
    }
    s.stateCounts[o.State]--
    s.stateCounts["CANCELLED"]++
    o.State = "CANCELLED"
    return ""
}

func (s *OrderSystem) Get(id string) string {
    o, ok := s.orders[id]
    if !ok {
        return "ERROR"
    }
    return fmt.Sprintf("%s %s", o.ID, o.State)
}

func (s *OrderSystem) Count(state string) string {
    if _, ok := s.stateCounts[state]; !ok {
        return "ERROR"
    }
    return fmt.Sprintf("%d", s.stateCounts[state])
}
```

---

## Level 2 — 使用者索引 + 批次取消

**新增**：每個訂單關聯 `userID`，支援 `cancel_all_by_user`。

```go
type OrderSystemV2 struct {
    OrderSystem
    userOrders map[string]map[string]bool // userID → set of orderIDs
}

func (s *OrderSystemV2) Create(id, symbol, side, userID string, qty int) string {
    result := s.OrderSystem.Create(id, symbol, side, qty)
    if result == "ERROR" {
        return result
    }
    if s.userOrders[userID] == nil {
        s.userOrders[userID] = make(map[string]bool)
    }
    s.userOrders[userID][id] = true
    s.orders[id].UserID = userID
    return ""
}

func (s *OrderSystemV2) CancelAllByUser(userID string) int {
    ids := s.userOrders[userID]
    cancelled := 0
    for id := range ids {
        if s.Cancel(id) != "ERROR" {
            cancelled++
        }
    }
    delete(s.userOrders, userID)
    return cancelled
}
```

**⚠️ 重點**：單筆 `Cancel` 也必須同步清理 `userOrders` 索引，防止 memory leak。

---

## Level 3 — 按 user 分 shard

**設計**：N 個獨立的 stream，按 `hash(userID) % N` 分配。同一 user 的所有訂單進同一 stream。

```go
type Stream struct {
    orders     map[string]*Order
    userOrders map[string]map[string]bool
    stateCounts map[string]int
}

type ShardedSystem struct {
    streams       []*Stream
    orderToStream map[string]int // orderId → stream index（全域查找）
    numStreams     int
}

func NewShardedSystem(n int) *ShardedSystem {
    streams := make([]*Stream, n)
    for i := range streams {
        streams[i] = &Stream{
            orders:      make(map[string]*Order),
            userOrders:  make(map[string]map[string]bool),
            stateCounts: map[string]int{"LIVE": 0, "PAUSED": 0, "CANCELLED": 0},
        }
    }
    return &ShardedSystem{
        streams:       streams,
        orderToStream: make(map[string]int),
        numStreams:     n,
    }
}

func (s *ShardedSystem) streamFor(userID string) *Stream {
    h := fnv.New32a()
    h.Write([]byte(userID))
    return s.streams[h.Sum32()%uint32(s.numStreams)]
}
```

**優點**：
- 同一 user 的訂單集中在一個 stream，cancel_all_by_user 不需跨 stream
- 各 stream 可平行處理，適合高吞吐量場景

---

## 狀態轉移圖

```
        CREATE
          ↓
        LIVE ←──── RESUME
          │               ↑
        PAUSE          PAUSED
          │               │
          └──── CANCEL ───┘
                  ↓
              CANCELLED（終態）
```

---

## 複雜度

| 操作 | 時間 |
|------|------|
| CREATE / PAUSE / RESUME / CANCEL | O(1) |
| GET | O(1) |
| COUNT | O(1)（維護 running counter）|
| cancel_all_by_user | O(K)，K = 該 user 的訂單數 |

---

## 相關頁面

- [[Coinbase_OA題目總表]] — 所有 Coinbase OA 題目索引
- [[Banking_System]] — 同類型 in-memory 系統題（HA 格式）
