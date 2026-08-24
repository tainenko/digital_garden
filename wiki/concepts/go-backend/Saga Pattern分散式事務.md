---
title: Saga Pattern 分散式事務
type: concept
tags: [saga, distributed-transaction, microservices, golang, eventual-consistency]
created: 2026-04-29
updated: 2026-04-29
---

# Saga Pattern 分散式事務

## 微服務沒有 ACID 事務

單體服務可以用資料庫事務（BEGIN / COMMIT / ROLLBACK）保證原子性。微服務因為 database-per-service，跨服務操作沒有全域事務。

```
❌ 這在微服務不可能做到
BEGIN TRANSACTION
  訂單服務：建立訂單
  庫存服務：扣減庫存
  金流服務：扣款
  通知服務：發確認信
COMMIT
```

**Saga** 是把一個長事務拆成一系列本地事務，每一步失敗時執行補償事務（compensating transaction）。

## 兩種 Saga 實作方式

### 1. Choreography（編舞）— 事件驅動

各服務監聽事件，自行決定下一步：

```
訂單服務          庫存服務           金流服務          通知服務
   │                 │                  │                 │
   │ → OrderCreated  │                  │                 │
   │                 ├─ 扣庫存          │                 │
   │                 │ → StockReserved  │                 │
   │                 │                  ├─ 扣款           │
   │                 │                  │ → PaymentDone   │
   │                 │                  │                 ├─ 發信
   │
   ── 失敗路徑 ──
   │
   │                 │ → StockFailed    │
   │ ← OrderCancelled（補償）           │
```

**優點**：低耦合，沒有單點故障
**缺點**：流程難以追蹤，debug 困難，難以保證執行順序

### 2. Orchestration（編排）— 中央協調

Saga Orchestrator 集中控制流程：

```go
// Orchestrator 負責整個流程
func (o *OrderSagaOrchestrator) Execute(ctx context.Context, orderID string) error {
    saga := &OrderSaga{OrderID: orderID, State: SagaStateStarted}

    // Step 1: 扣庫存
    if err := o.inventoryClient.Reserve(ctx, orderID); err != nil {
        return o.compensate(ctx, saga, SagaStepInventory)
    }
    saga.CompletedSteps = append(saga.CompletedSteps, SagaStepInventory)

    // Step 2: 扣款
    if err := o.paymentClient.Charge(ctx, orderID); err != nil {
        return o.compensate(ctx, saga, SagaStepPayment)
    }
    saga.CompletedSteps = append(saga.CompletedSteps, SagaStepPayment)

    // Step 3: 發通知
    if err := o.notificationClient.Send(ctx, orderID); err != nil {
        // 通知失敗不需要補償（非關鍵），記錄 log 即可
        log.Warnf("notification failed for order %s: %v", orderID, err)
    }

    saga.State = SagaStateCompleted
    return o.sagaRepo.Save(ctx, saga)
}

// 補償事務：從失敗點往回撤
func (o *OrderSagaOrchestrator) compensate(ctx context.Context, saga *OrderSaga, failedAt SagaStep) error {
    log.Infof("compensating saga %s from step %v", saga.OrderID, failedAt)

    // 逆序執行補償
    for i := len(saga.CompletedSteps) - 1; i >= 0; i-- {
        step := saga.CompletedSteps[i]
        switch step {
        case SagaStepInventory:
            if err := o.inventoryClient.Release(ctx, saga.OrderID); err != nil {
                // 補償失敗：記錄到 dead letter queue，人工介入
                o.dlq.Publish(ctx, CompensationFailedEvent{SagaID: saga.OrderID, Step: step})
            }
        case SagaStepPayment:
            if err := o.paymentClient.Refund(ctx, saga.OrderID); err != nil {
                o.dlq.Publish(ctx, CompensationFailedEvent{SagaID: saga.OrderID, Step: step})
            }
        }
    }

    saga.State = SagaStateCompensated
    return o.sagaRepo.Save(ctx, saga)
}
```

**優點**：流程清晰，易於 debug 和監控
**缺點**：Orchestrator 是中央單點（需要 HA）

## Saga 狀態持久化（關鍵！）

Saga 必須把狀態存到資料庫，防止 orchestrator 崩潰後流程中斷：

```go
type OrderSaga struct {
    ID             string        `db:"id"`
    OrderID        string        `db:"order_id"`
    State          SagaState     `db:"state"`        // started/completed/compensating/compensated
    CompletedSteps []SagaStep    `db:"completed_steps"` // JSON 陣列
    FailedStep     *SagaStep     `db:"failed_step"`
    CreatedAt      time.Time     `db:"created_at"`
    UpdatedAt      time.Time     `db:"updated_at"`
}

// 每次狀態轉換都要持久化
func (o *OrderSagaOrchestrator) transitionState(ctx context.Context, saga *OrderSaga, newState SagaState) error {
    saga.State = newState
    saga.UpdatedAt = time.Now()
    return o.sagaRepo.Update(ctx, saga)
}
```

## Outbox Pattern — 保證訊息投遞

Saga 依賴事件，但「寫 DB + 發 Kafka」這兩個操作不是原子的：

```
問題：
1. 寫 DB 成功
2. 發 Kafka 失敗（網路問題）
→ DB 有記錄，但下游服務沒收到事件 ← 資料不一致！
```

**Outbox Pattern** 解法：

```go
// 在同一個 DB 事務中，把「要發的事件」寫進 outbox 表
func (r *OrderRepository) CreateOrderWithOutbox(ctx context.Context, tx *sql.Tx, order *Order) error {
    // 1. 寫訂單
    if _, err := tx.ExecContext(ctx, "INSERT INTO orders ...", order); err != nil {
        return err
    }

    // 2. 同一個 tx 寫 outbox（原子！）
    event := OrderCreatedEvent{OrderID: order.ID, UserID: order.UserID}
    payload, _ := json.Marshal(event)
    if _, err := tx.ExecContext(ctx,
        "INSERT INTO outbox (aggregate_id, event_type, payload, status) VALUES ($1, $2, $3, 'pending')",
        order.ID, "OrderCreated", payload,
    ); err != nil {
        return err
    }

    return nil // tx 外部 commit
}

// 獨立的 Outbox Relay Goroutine，定期掃描 outbox 表發佈到 Kafka
func (r *OutboxRelay) Run(ctx context.Context) {
    ticker := time.NewTicker(1 * time.Second)
    for {
        select {
        case <-ctx.Done():
            return
        case <-ticker.C:
            r.processOutbox(ctx)
        }
    }
}

func (r *OutboxRelay) processOutbox(ctx context.Context) {
    rows, _ := r.db.QueryContext(ctx, "SELECT id, event_type, payload FROM outbox WHERE status = 'pending' LIMIT 100")
    for rows.Next() {
        var id int64
        var eventType string
        var payload []byte
        rows.Scan(&id, &eventType, &payload)

        if err := r.kafka.Publish(ctx, eventType, payload); err != nil {
            log.Warnf("outbox relay failed for id=%d: %v", id, err)
            continue
        }

        // 發成功才更新狀態
        r.db.ExecContext(ctx, "UPDATE outbox SET status = 'sent' WHERE id = $1", id)
    }
}
```

## 冪等性（Idempotency）

因為補償和重試，同一個操作可能被執行多次，必須設計冪等：

```go
// 每個操作用唯一 idempotency key
func (s *InventoryService) Reserve(ctx context.Context, req *ReserveRequest) error {
    // 用 saga_id + step 作為冪等鍵
    key := fmt.Sprintf("saga-%s-step-inventory", req.SagaID)

    // 嘗試插入（若已存在則忽略）
    _, err := s.db.ExecContext(ctx,
        "INSERT INTO idempotency_keys (key, processed_at) VALUES ($1, NOW()) ON CONFLICT DO NOTHING",
        key,
    )
    if err != nil {
        return err
    }

    // 實際執行庫存扣減
    return s.deductStock(ctx, req)
}
```

## Choreography vs Orchestration 選型

| 面向 | Choreography | Orchestration |
|------|-------------|---------------|
| 適用場景 | 簡單流程（2–3 步）| 複雜流程（4+ 步，有條件分支）|
| 可觀測性 | 差（需要全局追蹤）| 好（orchestrator 有完整狀態）|
| 擴展性 | 好（新服務監聽事件即可）| 差（需要修改 orchestrator）|
| 耦合度 | 低（只知道事件）| 中（orchestrator 知道所有服務）|
| Debug | 困難 | 容易 |

## 相關頁面

- [[微服務架構設計原則]] — 為什麼需要 Saga
- [[事件驅動架構與Kafka]] — Choreography 的事件基礎設施
- [[冪等性設計]] — Saga 重試的必要條件
- [[CQRS與Event Sourcing]] — Saga 事件常與 Event Sourcing 搭配
