---
title: CQRS 與 Event Sourcing
type: concept
tags: [cqrs, event-sourcing, microservices, golang, architecture]
created: 2026-04-29
updated: 2026-04-29
---

# CQRS 與 Event Sourcing

## CQRS（Command Query Responsibility Segregation）

### 核心思想

**把「寫」和「讀」分成兩個獨立的模型：**

```
傳統 CRUD：
用戶 → CRUD 服務 → 同一個資料庫（讀寫都打同一個表）

CQRS：
用戶 → Command（寫）→ Write DB（正規化，如 PostgreSQL）
                           ↓ 事件同步
用戶 → Query（讀）  → Read DB（非正規化，如 Elasticsearch / Redis）
```

**為什麼要拆？**
- 讀和寫的需求完全不同：讀需要快速、可擴展；寫需要一致性保證
- 讀的資料結構往往是「已聚合」的（把多個表的資料預先合併），直接儲存方便前端用
- 讀的壓力通常遠大於寫（80/20 法則），需要獨立擴展

### Go 實作：Command 端

```go
// Command：描述「做什麼」的意圖
type CreateOrderCommand struct {
    CommandID string    // 冪等鍵
    UserID    string
    Items     []OrderItemInput
    Address   Address
}

// Command Handler：執行業務邏輯，產生 Domain Events
type OrderCommandHandler struct {
    repo      OrderWriteRepository
    eventBus  EventBus
}

func (h *OrderCommandHandler) Handle(ctx context.Context, cmd CreateOrderCommand) (*Order, error) {
    // 1. 業務規則驗證
    if err := validateOrder(cmd); err != nil {
        return nil, err
    }

    // 2. 建立 Domain Object
    order := Order{
        ID:     generateID(),
        UserID: cmd.UserID,
        Status: OrderStatusPending,
    }
    for _, item := range cmd.Items {
        order.Items = append(order.Items, OrderItem{
            ProductID:      item.ProductID,
            Quantity:       item.Quantity,
            UnitPriceCents: item.UnitPriceCents,
        })
    }

    // 3. 寫入 Write DB（正規化）
    if err := h.repo.Save(ctx, &order); err != nil {
        return nil, err
    }

    // 4. 發布事件，供 Read Model 同步
    h.eventBus.Publish(ctx, OrderCreatedEvent{
        OrderID:    order.ID,
        UserID:     order.UserID,
        Items:      order.Items,
        TotalCents: order.TotalCents(),
        CreatedAt:  time.Now(),
    })

    return &order, nil
}
```

### Go 實作：Query 端

```go
// Query：描述「要看什麼」
type GetOrdersByUserQuery struct {
    UserID string
    Page   int
    Limit  int
}

// Read Model：非正規化，預先計算好前端需要的格式
type OrderListView struct {
    OrderID      string    `json:"order_id"`
    Status       string    `json:"status"`
    TotalCents   int64     `json:"total_cents"`
    ItemCount    int       `json:"item_count"`
    FirstItem    string    `json:"first_item_name"` // 預先 join 好的
    CreatedAt    time.Time `json:"created_at"`
}

// Query Handler：直接從 Read DB 讀，不需要業務邏輯
type OrderQueryHandler struct {
    readDB *sqlx.DB // 可以是 replica 或另一個 DB
}

func (h *OrderQueryHandler) GetByUser(ctx context.Context, q GetOrdersByUserQuery) ([]OrderListView, error) {
    // 直接查 read_orders 這個非正規化的 view table，速度極快
    var views []OrderListView
    err := h.readDB.SelectContext(ctx, &views, `
        SELECT order_id, status, total_cents, item_count, first_item_name, created_at
        FROM read_orders
        WHERE user_id = $1
        ORDER BY created_at DESC
        LIMIT $2 OFFSET $3
    `, q.UserID, q.Limit, (q.Page-1)*q.Limit)
    return views, err
}

// Event Handler：收到 Write 端的事件，更新 Read Model
type OrderReadModelUpdater struct {
    db *sqlx.DB
}

func (u *OrderReadModelUpdater) OnOrderCreated(ctx context.Context, event OrderCreatedEvent) error {
    // 把 Write 端的資料轉成 Read 端需要的扁平結構
    _, err := u.db.ExecContext(ctx, `
        INSERT INTO read_orders (order_id, user_id, status, total_cents, item_count, first_item_name, created_at)
        VALUES ($1, $2, $3, $4, $5, $6, $7)
        ON CONFLICT (order_id) DO UPDATE SET status = EXCLUDED.status
    `,
        event.OrderID,
        event.UserID,
        "pending",
        event.TotalCents,
        len(event.Items),
        event.Items[0].Name, // 第一項商品名稱
        time.Now(),
    )
    return err
}
```

---

## Event Sourcing

### 核心思想

**不儲存「當前狀態」，而是儲存「所有導致此狀態的事件序列」：**

```
傳統方式（儲存狀態）：
orders 表：{ id: "123", status: "shipped", total: 5000 }
→ 無法知道「為什麼」狀態是這樣

Event Sourcing（儲存事件）：
events 表：
  { order_id: "123", type: "OrderCreated",  data: {...}, at: "10:00" }
  { order_id: "123", type: "PaymentDone",   data: {...}, at: "10:05" }
  { order_id: "123", type: "OrderShipped",  data: {...}, at: "10:30" }
→ 完整的歷史記錄，任何時間點的狀態都可以重建
```

### Go 實作

```go
// Event Store：只允許 append，不修改不刪除
type DomainEvent struct {
    ID          string          `db:"id"`
    AggregateID string          `db:"aggregate_id"`
    Type        string          `db:"type"`
    Payload     json.RawMessage `db:"payload"`
    Version     int             `db:"version"`  // 樂觀鎖
    OccurredAt  time.Time       `db:"occurred_at"`
}

type EventStore struct {
    db *sqlx.DB
}

func (s *EventStore) Append(ctx context.Context, aggregateID string, events []DomainEvent, expectedVersion int) error {
    tx, _ := s.db.BeginTxx(ctx, nil)
    defer tx.Rollback()

    // 樂觀鎖：確認目前版本符合預期
    var currentVersion int
    tx.QueryRowContext(ctx,
        "SELECT COALESCE(MAX(version), 0) FROM events WHERE aggregate_id = $1",
        aggregateID,
    ).Scan(&currentVersion)

    if currentVersion != expectedVersion {
        return ErrConcurrentModification
    }

    for i, event := range events {
        event.Version = expectedVersion + i + 1
        if _, err := tx.ExecContext(ctx,
            "INSERT INTO events (id, aggregate_id, type, payload, version, occurred_at) VALUES ($1,$2,$3,$4,$5,$6)",
            event.ID, aggregateID, event.Type, event.Payload, event.Version, event.OccurredAt,
        ); err != nil {
            return err
        }
    }

    return tx.Commit()
}

// 從事件重建 Aggregate
func (s *EventStore) Load(ctx context.Context, aggregateID string) (*Order, error) {
    var events []DomainEvent
    s.db.SelectContext(ctx, &events,
        "SELECT * FROM events WHERE aggregate_id = $1 ORDER BY version ASC",
        aggregateID,
    )

    order := &Order{}
    for _, event := range events {
        order.Apply(event) // 每個事件改變 Aggregate 狀態
    }
    return order, nil
}

// Aggregate：Apply 函數決定每個事件怎麼改變狀態
func (o *Order) Apply(event DomainEvent) {
    switch event.Type {
    case "OrderCreated":
        var payload OrderCreatedPayload
        json.Unmarshal(event.Payload, &payload)
        o.ID = payload.OrderID
        o.UserID = payload.UserID
        o.Items = payload.Items
        o.Status = OrderStatusPending
        o.Version = event.Version

    case "PaymentConfirmed":
        o.Status = OrderStatusPaid
        o.Version = event.Version

    case "OrderShipped":
        var payload OrderShippedPayload
        json.Unmarshal(event.Payload, &payload)
        o.Status = OrderStatusShipped
        o.TrackingNumber = payload.TrackingNumber
        o.Version = event.Version
    }
}
```

### Snapshot（快照）優化

事件太多時，重建 Aggregate 很慢：

```go
// 每 100 個事件建一個快照
func (s *EventStore) LoadWithSnapshot(ctx context.Context, aggregateID string) (*Order, error) {
    // 1. 先嘗試讀最新快照
    snapshot, err := s.snapshotStore.Load(ctx, aggregateID)
    if err != nil {
        // 沒有快照，從頭重建
        return s.Load(ctx, aggregateID)
    }

    // 2. 只讀快照之後的事件
    var events []DomainEvent
    s.db.SelectContext(ctx, &events,
        "SELECT * FROM events WHERE aggregate_id = $1 AND version > $2 ORDER BY version ASC",
        aggregateID, snapshot.Version,
    )

    order := snapshot.Order
    for _, event := range events {
        order.Apply(event)
    }
    return &order, nil
}
```

## CQRS + Event Sourcing 組合

兩者最常一起使用：

```
Command → Aggregate（驗證業務規則）
             ↓ 產生 Domain Events
          Event Store（append-only 儲存）
             ↓ 事件投影
   ┌─────────────────────────────┐
   │ Read Model Projections      │
   ├──────────────┬──────────────┤
   │ PostgreSQL   │ Elasticsearch│
   │ (訂單查詢)   │ (搜尋)       │
   └──────────────┴──────────────┘
```

## 何時適合用

**適合**：
- 需要完整審計日誌（金融、電商、醫療）
- 業務需要「時間旅行」（回到某時間點的狀態）
- 讀寫壓力極不平衡
- 多種 Read Model 需求（同一資料給不同系統用）

**不適合**：
- 簡單的 CRUD 應用（過度設計）
- 資料量小、查詢簡單
- 團隊對 Event Sourcing 不熟悉（學習曲線高）

## 相關頁面

- [[Saga Pattern分散式事務]] — 跨服務事務常與 Event Sourcing 搭配
- [[事件驅動架構與Kafka]] — Event Store 的事件常發布到 Kafka
- [[微服務架構設計原則]] — CQRS 是微服務的進階設計模式
