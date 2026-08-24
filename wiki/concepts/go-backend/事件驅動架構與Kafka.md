---
title: 事件驅動架構與 Kafka
type: concept
tags: [event-driven, kafka, messaging, microservices, golang]
created: 2026-04-29
updated: 2026-04-29
---

# 事件驅動架構與 Kafka

## 事件驅動架構的核心概念

### 訊息類型

| 類型 | 描述 | 範例 |
|------|------|------|
| **Event**（事件） | 「已發生的事」，不可變，有歷史意義 | OrderCreated, PaymentFailed |
| **Command**（命令） | 「請執行某件事」，有意圖，可以被拒絕 | CreateOrder, CancelShipment |
| **Query**（查詢） | 「告訴我某件事」，無副作用 | GetOrder, ListProducts |

**設計原則**：
- Event 用過去式命名：`OrderCreated`，不用 `CreateOrder`
- Event 是事實，不包含「接下來該做什麼」的指示
- Event 應該包含足夠的資訊讓消費者不用再查詢

### Event Schema 設計

```go
// 好的 Event 設計：自包含（self-contained）
type OrderCreatedEvent struct {
    EventID   string    `json:"event_id"`   // 唯一 ID，用於冪等去重
    EventType string    `json:"event_type"` // "order.created"
    Version   int       `json:"version"`    // schema 版本，向前相容用
    OccurredAt time.Time `json:"occurred_at"`

    // Payload：足夠的業務資料，不用再打 API 查詢
    OrderID    string      `json:"order_id"`
    UserID     string      `json:"user_id"`
    UserEmail  string      `json:"user_email"`  // 冗餘但方便消費者
    Items      []EventItem `json:"items"`
    TotalCents int64       `json:"total_cents"`
}

// 壞的設計：只有 ID，消費者需要再查
type OrderCreatedEventBad struct {
    OrderID string `json:"order_id"` // 消費者要再打 /api/orders/{id} 才能知道內容
}
```

## Kafka 核心概念

```
Producer → Topic（分成多個 Partition）→ Consumer Group
                                          ├─ Consumer 1（處理 Partition 0,1）
                                          └─ Consumer 2（處理 Partition 2,3）
```

- **Topic**：訊息的邏輯分類（如 `orders`、`payments`）
- **Partition**：Topic 的物理分片，實現並行消費
- **Consumer Group**：一組消費者共同消費一個 Topic，每個 Partition 只被一個消費者處理
- **Offset**：消費者的消費位置，可以重播

**Partition Key 選擇**：相同 key 的訊息進同一個 Partition，保證有序性：
```go
// 同一個 orderID 的所有事件保證有序
msg.Key = []byte(orderID)
```

## Go + Kafka 實作（confluent-kafka-go）

### Producer

```go
package kafka

import (
    "encoding/json"
    "fmt"

    "github.com/confluentinc/confluent-kafka-go/v2/kafka"
)

type Producer struct {
    producer *kafka.Producer
}

func NewProducer(brokers string) (*Producer, error) {
    p, err := kafka.NewProducer(&kafka.ConfigMap{
        "bootstrap.servers":            brokers,
        "acks":                         "all",    // 等所有 replica 確認
        "enable.idempotence":           true,     // 精確一次語意
        "max.in.flight.requests.per.connection": 5,
        "retries":                      2147483647,
    })
    if err != nil {
        return nil, err
    }

    // 非同步處理 delivery report
    go func() {
        for e := range p.Events() {
            switch ev := e.(type) {
            case *kafka.Message:
                if ev.TopicPartition.Error != nil {
                    fmt.Printf("delivery failed: %v\n", ev.TopicPartition.Error)
                }
            }
        }
    }()

    return &Producer{producer: p}, nil
}

func (p *Producer) Publish(topic string, key string, event interface{}) error {
    payload, err := json.Marshal(event)
    if err != nil {
        return err
    }

    return p.producer.Produce(&kafka.Message{
        TopicPartition: kafka.TopicPartition{Topic: &topic, Partition: kafka.PartitionAny},
        Key:            []byte(key),
        Value:          payload,
        Headers: []kafka.Header{
            {Key: "event-type", Value: []byte(fmt.Sprintf("%T", event))},
            {Key: "content-type", Value: []byte("application/json")},
        },
    }, nil)
}

func (p *Producer) Flush(timeoutMs int) {
    p.producer.Flush(timeoutMs)
}
```

### Consumer（帶冪等處理）

```go
type Consumer struct {
    consumer *kafka.Consumer
    handlers map[string]EventHandler
    db       *sql.DB // 用於冪等去重
}

type EventHandler func(ctx context.Context, payload []byte) error

func (c *Consumer) RegisterHandler(eventType string, handler EventHandler) {
    c.handlers[eventType] = handler
}

func (c *Consumer) Start(ctx context.Context, topics []string) error {
    c.consumer.SubscribeTopics(topics, nil)

    for {
        select {
        case <-ctx.Done():
            return nil
        default:
            msg, err := c.consumer.ReadMessage(100 * time.Millisecond) // 100ms timeout
            if err != nil {
                if err.(kafka.Error).Code() == kafka.ErrTimedOut {
                    continue
                }
                return err
            }

            if err := c.processMessage(ctx, msg); err != nil {
                // 處理失敗：記錄 log，不 commit offset，讓訊息重新投遞
                log.Errorf("failed to process message: %v", err)
                continue
            }

            // 手動 commit（At-least-once 語意）
            c.consumer.CommitMessage(msg)
        }
    }
}

func (c *Consumer) processMessage(ctx context.Context, msg *kafka.Message) error {
    // 1. 取得 event type
    var eventType string
    for _, h := range msg.Headers {
        if h.Key == "event-type" {
            eventType = string(h.Value)
        }
    }

    // 2. 冪等去重：用 event_id 確保不重複處理
    var envelope struct {
        EventID string `json:"event_id"`
    }
    json.Unmarshal(msg.Value, &envelope)

    processed, _ := c.isAlreadyProcessed(ctx, envelope.EventID)
    if processed {
        log.Infof("duplicate event %s, skipping", envelope.EventID)
        return nil
    }

    // 3. 呼叫對應的 handler
    handler, ok := c.handlers[eventType]
    if !ok {
        log.Warnf("no handler for event type: %s", eventType)
        return nil
    }

    if err := handler(ctx, msg.Value); err != nil {
        return err
    }

    // 4. 記錄已處理
    return c.markProcessed(ctx, envelope.EventID)
}

func (c *Consumer) isAlreadyProcessed(ctx context.Context, eventID string) (bool, error) {
    var count int
    err := c.db.QueryRowContext(ctx,
        "SELECT COUNT(1) FROM processed_events WHERE event_id = $1", eventID,
    ).Scan(&count)
    return count > 0, err
}

func (c *Consumer) markProcessed(ctx context.Context, eventID string) error {
    _, err := c.db.ExecContext(ctx,
        "INSERT INTO processed_events (event_id, processed_at) VALUES ($1, NOW()) ON CONFLICT DO NOTHING",
        eventID,
    )
    return err
}
```

## Dead Letter Queue（DLQ）

消費失敗超過 N 次的訊息，發送到 DLQ 供人工審查：

```go
func (c *Consumer) processWithRetry(ctx context.Context, msg *kafka.Message, maxRetries int) error {
    var lastErr error
    for attempt := 0; attempt <= maxRetries; attempt++ {
        if attempt > 0 {
            time.Sleep(time.Duration(attempt*attempt) * time.Second) // exponential backoff
        }
        lastErr = c.processMessage(ctx, msg)
        if lastErr == nil {
            return nil
        }
        log.Warnf("attempt %d/%d failed: %v", attempt+1, maxRetries, lastErr)
    }

    // 超過重試次數，送到 DLQ
    dlqTopic := "dlq." + *msg.TopicPartition.Topic
    c.producer.Publish(dlqTopic, string(msg.Key), map[string]interface{}{
        "original_message": string(msg.Value),
        "error":            lastErr.Error(),
        "failed_at":        time.Now(),
    })
    return nil // 不返回錯誤，避免無限重試
}
```

## Kafka Streams 常見模式

### Fan-out（一對多廣播）

```
OrderCreated → 庫存服務（扣庫存）
             → 通知服務（發確認信）
             → 分析服務（記錄事件）
             → 搜尋服務（更新索引）
```

每個服務有自己的 Consumer Group，獨立消費同一個 Topic。

### Event Aggregation（聚合計算）

```go
// 用 Redis 累計每個商品的 24 小時銷量
func (h *SalesAggregator) OnOrderCreated(ctx context.Context, payload []byte) error {
    var event OrderCreatedEvent
    json.Unmarshal(payload, &event)

    pipe := h.redis.Pipeline()
    for _, item := range event.Items {
        key := fmt.Sprintf("sales:product:%s:%s", item.ProductID, time.Now().Format("2006-01-02"))
        pipe.IncrBy(ctx, key, int64(item.Quantity))
        pipe.Expire(ctx, key, 25*time.Hour) // 保留 25 小時（稍微多於一天）
    }
    _, err := pipe.Exec(ctx)
    return err
}
```

## 相關頁面

- [[Saga Pattern分散式事務]] — Kafka 作為 Saga Choreography 的事件匯流排
- [[CQRS與Event Sourcing]] — Event Store 的事件發布到 Kafka
- [[冪等性設計]] — Consumer 端必須做冪等處理
- [[微服務架構設計原則]] — 事件驅動是微服務解耦的關鍵手段
