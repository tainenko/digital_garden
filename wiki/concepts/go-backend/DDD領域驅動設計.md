---
title: DDD 領域驅動設計
type: concept
tags: [ddd, domain-driven-design, bounded-context, aggregate, value-object, domain-event, golang, senior]
created: 2026-04-30
updated: 2026-04-30
---

# DDD 領域驅動設計（Domain-Driven Design）

DDD 是一套處理**複雜業務邏輯**的設計方法論，核心思想：讓**程式碼結構反映業務模型**，讓開發者和業務人員說同一種語言。

---

## 戰略設計（Strategic Design）

### Ubiquitous Language（通用語言）

整個團隊（開發 + 業務）使用同一套術語，**代碼中的命名直接反映業務詞彙**。

```go
// ❌ 技術語言（和業務脫節）
type Record struct {
    ID     int
    Status int // 1=active, 2=suspended, 3=closed
}

func (r *Record) UpdateStatus(s int) error { ... }

// ✅ 通用語言（代碼即文件）
type Account struct {
    ID     AccountID
    Status AccountStatus // Active, Suspended, Closed
}

func (a *Account) Suspend(reason string) error { ... }
func (a *Account) Reactivate() error           { ... }
func (a *Account) Close() error                { ... }
```

### Bounded Context（限界上下文）

一個大系統拆成多個**邊界清晰的子領域**，每個 Bounded Context 有自己的 Ubiquitous Language 和 Domain Model。**同一個詞在不同 Context 有不同含義**。

```
電商系統的 Bounded Context：

┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   訂單上下文     │    │   商品上下文     │    │   庫存上下文     │
│                 │    │                 │    │                 │
│ Order           │    │ Product         │    │ SKU             │
│ OrderItem       │    │ Category        │    │ StockLevel      │
│ ShippingAddress │    │ Pricing         │    │ Warehouse       │
│                 │    │                 │    │                 │
│ "Product" =     │    │ "Product" =     │    │ "Product" =     │
│ 快照（含當時價格）│    │ 完整商品資訊    │    │ 庫存追蹤單位    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                      │                      │
         └──────────────────────┼──────────────────────┘
                           Context Map
                    （定義 Context 間的關係）
```

**Context Map 關係類型**：
- **Shared Kernel**：共用核心模型（需要協調）
- **Customer/Supplier**：下游消費上游的輸出
- **Anti-Corruption Layer（ACL）**：翻譯層，防止外部模型污染本地模型
- **Published Language**：透過標準格式（如 event schema）溝通

---

## 戰術設計（Tactical Design）

### Entity（實體）

有**唯一身份（ID）**的物件，身份在整個生命週期不變，即使屬性改變。

```go
// Entity 的特徵：用 ID 判斷相等，不是用屬性
type Order struct {
    id         OrderID         // 身份
    customerID CustomerID
    status     OrderStatus
    items       []OrderItem
    createdAt  time.Time
}

func NewOrder(customerID CustomerID, items []OrderItem) (*Order, error) {
    if len(items) == 0 {
        return nil, ErrOrderMustHaveItems
    }
    return &Order{
        id:         NewOrderID(),
        customerID: customerID,
        status:     OrderStatusPending,
        items:      items,
        createdAt:  time.Now(),
    }, nil
}

// Entity 相等性：比較 ID，不比較屬性
func (o *Order) Equals(other *Order) bool {
    return o.id == other.id
}

// 行為封裝在 Entity 內（不是 getter/setter）
func (o *Order) Cancel(reason string) error {
    if o.status != OrderStatusPending {
        return ErrCannotCancelNonPendingOrder
    }
    o.status = OrderStatusCancelled
    o.addEvent(OrderCancelledEvent{OrderID: o.id, Reason: reason})
    return nil
}
```

### Value Object（值物件）

**無身份**，只有值；相等性由**所有屬性**決定；**不可變**（修改即建立新物件）。

```go
// Value Object：不可變，用屬性判斷相等
type Money struct {
    amount   int64  // 以分為單位，避免浮點問題
    currency string // "TWD", "USD"
}

func NewMoney(amount int64, currency string) (Money, error) {
    if amount < 0 {
        return Money{}, ErrNegativeAmount
    }
    if currency == "" {
        return Money{}, ErrEmptyCurrency
    }
    return Money{amount: amount, currency: currency}, nil
}

// 不可變：運算返回新物件
func (m Money) Add(other Money) (Money, error) {
    if m.currency != other.currency {
        return Money{}, ErrCurrencyMismatch
    }
    return Money{amount: m.amount + other.amount, currency: m.currency}, nil
}

func (m Money) Equals(other Money) bool {
    return m.amount == other.amount && m.currency == other.currency
}

// 其他 Value Object 範例
type Address struct {
    Street  string
    City    string
    Country string
    ZipCode string
}

type Email struct {
    value string
}

func NewEmail(s string) (Email, error) {
    // 驗證邏輯放在這裡
    if !isValidEmail(s) {
        return Email{}, ErrInvalidEmail
    }
    return Email{value: strings.ToLower(s)}, nil
}
```

### Aggregate（聚合）

一組相關 Entity 和 Value Object 的**一致性邊界**，由 **Aggregate Root** 統一控制存取。

```
關鍵規則：
1. 只能透過 Aggregate Root 存取內部物件
2. 外部只持有 Aggregate Root 的 ID 引用
3. 一個事務只修改一個 Aggregate（跨 Aggregate 用最終一致性）
4. Aggregate 應該小（避免鎖定範圍過大）
```

```go
// Order 是 Aggregate Root，OrderItem 只能透過 Order 存取
type Order struct {
    id     OrderID
    items  []OrderItem  // 不對外暴露（private）
    status OrderStatus
    events []DomainEvent
}

// ✅ 透過 Aggregate Root 的方法修改內部
func (o *Order) AddItem(productID ProductID, qty int, price Money) error {
    if o.status != OrderStatusPending {
        return ErrCannotModifyNonPendingOrder
    }
    // 業務規則：最多 20 個品項
    if len(o.items) >= 20 {
        return ErrOrderItemLimitExceeded
    }
    o.items = append(o.items, OrderItem{
        productID: productID,
        quantity:  qty,
        unitPrice: price,
    })
    return nil
}

func (o *Order) TotalPrice() Money {
    total, _ := NewMoney(0, "TWD")
    for _, item := range o.items {
        itemTotal, _ := item.unitPrice.Multiply(item.quantity)
        total, _ = total.Add(itemTotal)
    }
    return total
}

// ❌ 不應該直接暴露內部
func (o *Order) Items() []OrderItem { return o.items } // 避免！外部修改繞過業務規則
// ✅ 返回副本或 read-only view
func (o *Order) ItemCount() int { return len(o.items) }
```

### Domain Event（領域事件）

記錄**領域中發生了什麼**，用於解耦 Bounded Context、觸發副作用。

```go
// Domain Event 是不可變的事實記錄
type OrderPlacedEvent struct {
    OrderID    OrderID
    CustomerID CustomerID
    TotalPrice Money
    OccurredAt time.Time
}

func (e OrderPlacedEvent) EventType() string { return "order.placed" }
func (e OrderPlacedEvent) OccurredOn() time.Time { return e.OccurredAt }

// Aggregate 收集 Domain Events（而不是直接發布）
type Order struct {
    // ...
    events []DomainEvent
}

func (o *Order) Place() error {
    if err := o.validate(); err != nil {
        return err
    }
    o.status = OrderStatusPlaced
    // 記錄事件，由 Repository 在 Save 後發布
    o.events = append(o.events, OrderPlacedEvent{
        OrderID:    o.id,
        CustomerID: o.customerID,
        TotalPrice: o.TotalPrice(),
        OccurredAt: time.Now(),
    })
    return nil
}

func (o *Order) PopEvents() []DomainEvent {
    events := o.events
    o.events = nil
    return events
}
```

### Repository（倉儲）

**抽象化資料存取**，讓 Domain Layer 不依賴具體的資料庫技術。

```go
// Domain Layer 定義介面（依賴反轉）
type OrderRepository interface {
    FindByID(ctx context.Context, id OrderID) (*Order, error)
    Save(ctx context.Context, order *Order) error
    FindByCustomerID(ctx context.Context, customerID CustomerID, filter OrderFilter) ([]*Order, error)
    Delete(ctx context.Context, id OrderID) error
}

// Infrastructure Layer 實作（可以是 PostgreSQL、MongoDB、記憶體等）
type PostgresOrderRepository struct {
    db        *sql.DB
    publisher EventPublisher
}

func (r *PostgresOrderRepository) Save(ctx context.Context, order *Order) error {
    // 持久化 Aggregate
    tx, err := r.db.BeginTx(ctx, nil)
    if err != nil {
        return err
    }
    defer tx.Rollback()

    if err := r.upsertOrder(tx, order); err != nil {
        return err
    }
    if err := r.upsertOrderItems(tx, order); err != nil {
        return err
    }

    if err := tx.Commit(); err != nil {
        return err
    }

    // Commit 後發布 Domain Events
    for _, event := range order.PopEvents() {
        r.publisher.Publish(ctx, event)
    }
    return nil
}
```

### Domain Service（領域服務）

**跨 Aggregate 的業務邏輯**，不屬於任何單一 Entity 或 Value Object。

```go
// 領域服務：定價策略涉及 Order + Customer + Product，不屬於任何一個
type PricingService struct {
    discountRepo DiscountRepository
}

func (s *PricingService) CalculateOrderPrice(
    ctx context.Context,
    order *Order,
    customer *Customer,
) (Money, error) {
    basePrice := order.TotalPrice()

    // 跨 Aggregate 的業務規則
    discounts, err := s.discountRepo.FindApplicable(ctx, customer.Tier(), order.Items())
    if err != nil {
        return Money{}, err
    }

    return applyDiscounts(basePrice, discounts)
}
```

### Application Service（應用服務）

**協調 Domain Objects** 完成用例（Use Case），不包含業務邏輯。負責：事務管理、授權檢查、DTO 轉換。

```go
// Application Service：協調，不含業務邏輯
type OrderApplicationService struct {
    orderRepo   OrderRepository
    productRepo ProductRepository
    pricing     PricingService
}

func (s *OrderApplicationService) PlaceOrder(ctx context.Context, cmd PlaceOrderCommand) (OrderID, error) {
    // 1. 載入 Domain Objects
    customer, err := s.customerRepo.FindByID(ctx, cmd.CustomerID)
    if err != nil {
        return "", err
    }

    // 2. 構建 Aggregate（Domain 邏輯在 Entity/Service 中）
    items, err := s.buildOrderItems(ctx, cmd.Items)
    if err != nil {
        return "", err
    }

    order, err := NewOrder(cmd.CustomerID, items)
    if err != nil {
        return "", err
    }

    // 3. 呼叫 Domain Service
    finalPrice, err := s.pricing.CalculateOrderPrice(ctx, order, customer)
    if err != nil {
        return "", err
    }
    _ = finalPrice

    // 4. 執行業務操作
    if err := order.Place(); err != nil {
        return "", err
    }

    // 5. 持久化（Repository 負責發布 Domain Events）
    if err := s.orderRepo.Save(ctx, order); err != nil {
        return "", err
    }

    return order.ID(), nil
}
```

---

## DDD 分層架構

```
┌──────────────────────────────────────┐
│         Presentation Layer           │  HTTP Handler、gRPC Handler
│     (Controller / gRPC Handler)      │  只做輸入驗證、格式轉換
└───────────────────┬──────────────────┘
                    │ DTO（Command/Query）
┌───────────────────▼──────────────────┐
│         Application Layer            │  Application Service
│      (Use Case / Application         │  協調 Domain Objects
│           Service)                   │  管理事務邊界
└───────────────────┬──────────────────┘
                    │ Domain Objects
┌───────────────────▼──────────────────┐
│           Domain Layer               │  Entity, Value Object
│  (Entity, Value Object, Domain       │  Aggregate, Domain Event
│   Event, Domain Service,             │  Repository 介面（定義）
│   Repository interface)              │  ← 純業務邏輯，無框架依賴
└───────────────────┬──────────────────┘
                    │ 實作介面
┌───────────────────▼──────────────────┐
│       Infrastructure Layer           │  Repository 實作
│  (Repository impl, Event Bus,        │  DB、Cache、外部 API
│   External API, ORM)                 │  Event Publisher
└──────────────────────────────────────┘

依賴方向：上層依賴下層，但 Domain 層不依賴 Infrastructure
（透過依賴反轉：Domain 定義介面，Infrastructure 實作）
```

---

## Go 專案目錄結構

```
order-service/
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── domain/
│   │   └── order/
│   │       ├── order.go           # Entity（Aggregate Root）
│   │       ├── order_item.go      # Entity（內部）
│   │       ├── money.go           # Value Object
│   │       ├── events.go          # Domain Events
│   │       ├── repository.go      # Repository 介面
│   │       └── service.go         # Domain Service
│   ├── application/
│   │   └── order/
│   │       ├── service.go         # Application Service
│   │       └── commands.go        # Command DTOs
│   ├── infrastructure/
│   │   └── persistence/
│   │       └── postgres_order_repo.go  # Repository 實作
│   └── interfaces/
│       └── http/
│           └── order_handler.go   # HTTP Handler
└── wire.go                        # Wire 依賴注入
```

---

## 常見面試問題

**Q：Entity 和 Value Object 怎麼區分？**
A：問「如果兩個物件所有屬性都一樣，它們是同一個嗎？」是 → Value Object；否（需要追蹤同一性）→ Entity。例如：兩張同面額的鈔票（Value Object），兩個相同姓名的用戶（Entity，有各自的 ID）。

**Q：Aggregate 多大才合適？**
A：越小越好。原則：同一個事務內需要保持一致的最小集合。常見錯誤是把整個「訂單+客戶+商品」放成一個 Aggregate，正確做法是分成三個 Aggregate，跨 Aggregate 用 Domain Events 做最終一致性。

**Q：Repository 應該放在哪一層？**
A：**介面定義在 Domain 層**（`OrderRepository interface`），**實作在 Infrastructure 層**（`PostgresOrderRepository struct`）。這是依賴反轉原則的體現——Domain 層不依賴具體技術。

## 相關頁面

- [[依賴注入與控制反轉]] — DDD 分層架構如何用 DI 組裝
- [[Go Wire深度實戰]] — 用 Wire 組裝 DDD 各層的依賴
- [[微服務架構設計原則]] — Bounded Context 與微服務邊界的對應
- [[Go介面設計模式]] — Repository 模式的 Go 實作細節
- [[DDD AI Agent技能流水線]] — 將 DDD 知識封裝為 9 個 AI Agent 技能，從模糊需求到工程規格的閉環
- [[DDD回溯觸發機制]] — 品質門禁設計：invariant < 60% 等觸發條件自動退回上游重跑
