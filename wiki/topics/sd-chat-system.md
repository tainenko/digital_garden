---
title: "SD題解：即時聊天系統（WhatsApp / Messenger）"
type: topic
tags: [system-design, chat, websocket, messaging, golang, medium]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：即時聊天系統（WhatsApp / Messenger）

> **難度**: 中級 ｜ **頻率**: 極高（必練）｜ **代表**: WhatsApp, Slack, Discord

---

## RESHADED 快速分析

**E - Estimation**（WhatsApp 規模）
- DAU：20億
- 每天訊息：1000億條
- 每秒訊息：1000億 / 86400 ≈ **116萬 QPS**（極高寫入）
- 每條訊息大小：~100B（文字），~1MB（圖片另存 S3）

**核心難點**：
1. 訊息必達（At-least-once delivery）
2. 訊息順序保證
3. 在線狀態的實時性
4. 群組訊息的擴散

---

## 架構設計

```
Client A ──WebSocket──→ Chat Server A ──→ Kafka ──→ Chat Server B ──WebSocket──→ Client B
                            ↓                              ↓
                       Message DB                   Push Notification
                      （Cassandra）                  （離線用戶）
```

**為什麼用 WebSocket？**
- 雙向通訊（服務器可主動推送）
- 比 Long Polling 延遲更低
- 持久連線，省去重複 handshake 開銷

---

## Go 實現

### WebSocket 連線管理

```go
package chat

import (
    "sync"

    "github.com/gorilla/websocket"
)

// Hub 管理所有連線（每台 Chat Server 一個 Hub）
type Hub struct {
    // userID → WebSocket 連線
    connections map[string]*websocket.Conn
    mu          sync.RWMutex
}

func NewHub() *Hub {
    return &Hub{connections: make(map[string]*websocket.Conn)}
}

func (h *Hub) Register(userID string, conn *websocket.Conn) {
    h.mu.Lock()
    defer h.mu.Unlock()
    h.connections[userID] = conn
}

func (h *Hub) Unregister(userID string) {
    h.mu.Lock()
    defer h.mu.Unlock()
    delete(h.connections, userID)
}

// Send 傳送訊息給指定用戶（如果在本台 Server）
func (h *Hub) Send(userID string, msg *Message) bool {
    h.mu.RLock()
    conn, ok := h.connections[userID]
    h.mu.RUnlock()

    if !ok {
        return false // 用戶不在本台 Server
    }
    conn.WriteJSON(msg)
    return true
}
```

### 訊息路由（跨伺服器）

問題：Client A 連在 Server 1，Client B 連在 Server 2，怎麼互傳訊息？

```go
type ChatService struct {
    hub             *Hub
    messageQueue    MessageQueue   // Kafka
    messageDB       MessageStore   // Cassandra
    presenceService PresenceService
    pushService     PushNotifier
}

type Message struct {
    ID         string    `json:"id"`
    SenderID   string    `json:"sender_id"`
    ReceiverID string    `json:"receiver_id"` // 1對1
    GroupID    string    `json:"group_id"`    // 群組（擇一）
    Content    string    `json:"content"`
    Type       string    `json:"type"` // text/image/file
    SentAt     int64     `json:"sent_at"` // Unix milliseconds
    Status     string    `json:"status"`  // sent/delivered/read
}

// SendMessage 傳送訊息的主流程
func (s *ChatService) SendMessage(msg *Message) error {
    // 1. 產生唯一訊息 ID（Snowflake，保證時序）
    msg.ID = generateSnowflakeID()
    msg.SentAt = time.Now().UnixMilli()
    msg.Status = "sent"

    // 2. 持久化到 DB（先存，再傳）
    if err := s.messageDB.Save(msg); err != nil {
        return err
    }

    // 3. 透過 Kafka 廣播（讓所有 Chat Server 都有機會送達）
    return s.messageQueue.Publish("messages", msg)
}

// DeliverMessage Kafka Consumer（所有 Chat Server 都訂閱）
func (s *ChatService) DeliverMessage(msg *Message) {
    if msg.GroupID != "" {
        s.deliverGroupMessage(msg)
        return
    }

    // 嘗試在本台 Server 送達
    delivered := s.hub.Send(msg.ReceiverID, msg)
    if delivered {
        s.updateStatus(msg.ID, "delivered")
        return
    }

    // 用戶不在本台 Server：送 Push Notification
    online := s.presenceService.IsOnline(msg.ReceiverID)
    if online {
        // 用戶在線但在別台 Server：透過 Redis Pub/Sub 路由
        s.routeToOtherServer(msg)
    } else {
        // 用戶離線：推播通知
        s.pushService.Notify(msg.ReceiverID, msg)
    }
}
```

### 在線狀態（Presence Service）

```go
type PresenceService struct {
    cache *redis.Client
}

const onlineTTL = 30 * time.Second // 30秒沒心跳則視為離線

// Heartbeat 客戶端每 10 秒發一次心跳
func (p *PresenceService) Heartbeat(userID string) {
    p.cache.Set(context.Background(),
        "online:"+userID, "1", onlineTTL)
}

func (p *PresenceService) IsOnline(userID string) bool {
    val, _ := p.cache.Get(context.Background(), "online:"+userID).Result()
    return val == "1"
}

// SetOffline 用戶斷線時呼叫
func (p *PresenceService) SetOffline(userID string) {
    p.cache.Del(context.Background(), "online:"+userID)
    // 廣播給好友（可選：讓好友的 UI 更新在線狀態）
    p.cache.Publish(context.Background(), "presence", userID+":offline")
}
```

### 訊息儲存（Cassandra Schema）

```
// Cassandra：按 (conversation_id, sent_at DESC) 儲存
// 天然支援「取最新 N 條訊息」查詢

CREATE TABLE messages (
    conversation_id TEXT,
    message_id      BIGINT,   -- Snowflake ID（時序有序）
    sender_id       TEXT,
    content         TEXT,
    type            TEXT,
    status          TEXT,
    sent_at         TIMESTAMP,
    PRIMARY KEY (conversation_id, message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);
```

為什麼用 Cassandra？
- 寫入極快（LSM tree）
- 天然支援按時間排序查詢
- 水平擴展能力強
- 讀少（用戶通常只看最近訊息）

### 群組訊息

```go
func (s *ChatService) deliverGroupMessage(msg *Message) {
    // 取群組所有成員
    members, _ := s.getGroupMembers(msg.GroupID)

    // 對每個成員做 Fanout（類似社群 Feed）
    for _, memberID := range members {
        if memberID == msg.SenderID {
            continue // 不送給自己
        }
        memberMsg := *msg
        memberMsg.ReceiverID = memberID

        // 非同步投遞
        go s.DeliverMessage(&memberMsg)
    }
}
```

---

## 訊息狀態機

```
發送中 → sent（存入 DB）
sent → delivered（接收方的 App 收到）
delivered → read（接收方讀取）
```

```go
// 已讀回執（Read Receipt）
func (s *ChatService) MarkAsRead(userID, conversationID string, upToMessageID string) {
    // 1. 更新 DB
    s.messageDB.MarkRead(conversationID, upToMessageID)
    // 2. 通知發送方（透過 WebSocket）
    senderID := s.getSenderID(upToMessageID)
    s.hub.Send(senderID, &Message{
        Type:    "read_receipt",
        Content: upToMessageID,
    })
}
```

---

## Trade-offs

| 決策 | 選擇 | 理由 |
|------|------|------|
| 傳輸協定 | WebSocket | 雙向即時，比輪詢延遲低 |
| 訊息 DB | Cassandra | 寫重、時序查詢、水平擴展 |
| 跨 Server 路由 | Kafka | 解耦，可靠傳遞 |
| 訊息 ID | Snowflake | 保證時序且全域唯一 |
| 在線狀態 | Redis TTL | 輕量，自動過期 |

---

## 相關題解

- [[sd-social-media-feed|SD題解：社群媒體 Feed]] — Fanout 模式（群組訊息相似）
- [[sd-rate-limiter|SD題解：Rate Limiter]] — 防止訊息轟炸
