---
title: 通知系統（Notification System）
type: topic
tags: [system-design, interview, notification, push, sms, email]
created: 2026-04-20
updated: 2026-04-20
---

# 通知系統（Notification System）

難度：中級｜核心技術：Push / SMS / Email、訊息佇列、Fan-out、冪等性

---

## RESHADED 分析

### Requirements
**Functional:**
- 支援三種通知類型：Push（iOS/Android）、SMS、Email
- 使用者可設定通知偏好（關閉特定類型）
- 支援「排程通知」與「即時通知」
- 服務端可主動觸發（行銷推播）或由事件觸發（訂單成立）

**Non-functional:**
- 高吞吐量：每秒 100 萬次通知
- 軟即時性：Push 1秒內，SMS/Email 10秒內
- 可靠性：至少一次送達（at-least-once delivery）
- 使用者偏好需即時生效

### Estimates
```
DAU: 1億
每日通知量: 1億 × 10 = 10億次/天
峰值 QPS: 10億 / 86400 ≈ 12,000/秒
Push 儲存: 裝置 token × 100 bytes = 10億 × 100B = 100 GB
```

### High-Level Design

```
觸發來源 (服務/排程)
    ↓
通知服務 (Notification Service)
    ↓
使用者偏好查詢 → 過濾
    ↓
訊息佇列 (per channel)
  ├─ Push Queue → iOS Worker (APNs) / Android Worker (FCM)
  ├─ SMS Queue  → SMS Worker (Twilio/SNS)
  └─ Email Queue → Email Worker (SendGrid/SES)
    ↓
第三方服務發送
    ↓
送達狀態回調 → 儲存 Log
```

### API Design
```go
// 觸發通知
POST /v1/notifications
{
  "type": "push|sms|email",
  "user_ids": [123, 456],
  "template_id": "order_confirmed",
  "data": {"order_id": "abc123"}
}

// 更新使用者偏好
PUT /v1/users/{user_id}/notification-preferences
{
  "push_enabled": true,
  "sms_enabled": false,
  "email_enabled": true,
  "quiet_hours": {"start": "22:00", "end": "08:00"}
}
```

### Data Model

```sql
-- 裝置 Token 表
CREATE TABLE device_tokens (
    user_id     BIGINT,
    device_id   VARCHAR(64),
    token       VARCHAR(256),   -- APNs / FCM token
    platform    ENUM('ios', 'android'),
    updated_at  TIMESTAMP,
    PRIMARY KEY (user_id, device_id)
);

-- 通知記錄表（冪等性）
CREATE TABLE notification_logs (
    id              VARCHAR(64) PRIMARY KEY,  -- idempotency_key
    user_id         BIGINT,
    channel         ENUM('push', 'sms', 'email'),
    status          ENUM('pending', 'sent', 'failed', 'delivered'),
    payload         JSON,
    created_at      TIMESTAMP,
    sent_at         TIMESTAMP
);
```

---

## 核心技術深探

### 1. 三種通知渠道

| 渠道 | 第三方服務 | 限制 |
|------|----------|------|
| iOS Push | APNs（蘋果）| Token 需定期更新 |
| Android Push | FCM（Google）| 需處理 Token 失效 |
| SMS | Twilio、AWS SNS | 費用高，限制字數 |
| Email | SendGrid、AWS SES | 垃圾郵件過濾 |

### 2. 冪等性設計

重複觸發（網路重試）不能重複發送同一通知：

```go
type NotificationService struct {
    db    *sql.DB
    queue MessageQueue
}

func (s *NotificationService) Send(ctx context.Context, req SendRequest) error {
    idempotencyKey := fmt.Sprintf("%s-%d-%s",
        req.EventType, req.UserID, req.EventID)

    // 冪等性檢查
    existing, err := s.db.GetNotificationLog(ctx, idempotencyKey)
    if err == nil && existing.Status == "sent" {
        return nil  // 已送出，直接返回
    }

    // 記錄 pending 狀態
    if err := s.db.InsertLog(ctx, NotificationLog{
        ID:        idempotencyKey,
        UserID:    req.UserID,
        Channel:   req.Channel,
        Status:    "pending",
        Payload:   req.Payload,
        CreatedAt: time.Now(),
    }); err != nil {
        return err
    }

    // 發入 Queue
    return s.queue.Publish(ctx, req.Channel, idempotencyKey, req.Payload)
}
```

### 3. 使用者偏好過濾

```go
type UserPreference struct {
    UserID       int64
    PushEnabled  bool
    SMSEnabled   bool
    EmailEnabled bool
    QuietStart   int // 22 = 22:00
    QuietEnd     int // 8  = 08:00
}

func (s *NotificationService) filterByPreference(
    userID int64, channel string, userTZ *time.Location,
) (bool, error) {
    pref, err := s.prefCache.Get(userID)
    if err != nil {
        return false, err
    }

    // 渠道偏好
    switch channel {
    case "push":
        if !pref.PushEnabled { return false, nil }
    case "sms":
        if !pref.SMSEnabled { return false, nil }
    case "email":
        if !pref.EmailEnabled { return false, nil }
    }

    // 安靜時段（使用者本地時間）
    now := time.Now().In(userTZ).Hour()
    if pref.QuietStart > pref.QuietEnd {
        // 跨午夜：22:00 - 08:00
        if now >= pref.QuietStart || now < pref.QuietEnd {
            return false, nil
        }
    } else {
        if now >= pref.QuietStart && now < pref.QuietEnd {
            return false, nil
        }
    }

    return true, nil
}
```

### 4. Push Worker（APNs 範例）

```go
type PushWorker struct {
    apnsClient *apns2.Client
    fcmClient  *fcm.Client
    db         *sql.DB
}

func (w *PushWorker) ProcessMessage(msg QueueMessage) error {
    var payload PushPayload
    json.Unmarshal(msg.Body, &payload)

    tokens, err := w.db.GetDeviceTokens(payload.UserID)
    if err != nil {
        return err
    }

    var wg sync.WaitGroup
    for _, token := range tokens {
        wg.Add(1)
        go func(t DeviceToken) {
            defer wg.Done()

            var sendErr error
            switch t.Platform {
            case "ios":
                notification := &apns2.Notification{
                    DeviceToken: t.Token,
                    Topic:       "com.example.app",
                    Payload:     payload.ToAPNs(),
                }
                _, sendErr = w.apnsClient.Push(notification)

            case "android":
                msg := &fcm.Message{
                    Token:        t.Token,
                    Notification: payload.ToFCM(),
                }
                _, sendErr = w.fcmClient.Send(context.Background(), msg)
            }

            status := "sent"
            if sendErr != nil {
                status = "failed"
                // Token 失效時清除
                if isTokenExpired(sendErr) {
                    w.db.DeleteDeviceToken(t.UserID, t.DeviceID)
                }
            }

            w.db.UpdateNotificationStatus(msg.ID, status)
        }(token)
    }
    wg.Wait()
    return nil
}
```

### 5. 排程通知

```go
type ScheduledNotification struct {
    ID         string
    UserIDs    []int64
    TemplateID string
    Data       map[string]string
    SendAt     time.Time
}

// Cron Job 每分鐘掃描待發通知
func (s *Scheduler) Tick(ctx context.Context) {
    pending, err := s.db.GetPendingNotifications(ctx, time.Now())
    if err != nil {
        return
    }

    for _, n := range pending {
        s.notificationSvc.BulkSend(ctx, n)
        s.db.MarkScheduledSent(ctx, n.ID)
    }
}
```

---

## 擴展性設計

### 高吞吐量：Fan-out 架構

```
行銷推播（1億用戶）：
  ├─ 不直接查 DB，用 Segment（按地區/偏好分組）
  ├─ 批次分割：每批 1,000 人
  ├─ 並行 Worker 消費 Queue
  └─ 速率限制（避免第三方 API 超限）
```

### 失敗重試策略

```go
const (
    MaxRetries  = 3
    RetryDelay  = 5 * time.Minute  // 指數退避
)

func (w *Worker) processWithRetry(msg QueueMessage) {
    if msg.RetryCount >= MaxRetries {
        // 送入 DLQ（Dead Letter Queue）
        w.dlq.Publish(msg)
        w.db.UpdateStatus(msg.ID, "failed_permanent")
        return
    }

    if err := w.process(msg); err != nil {
        msg.RetryCount++
        delay := RetryDelay * time.Duration(1<<msg.RetryCount)
        w.queue.PublishDelayed(msg, delay)
    }
}
```

---

## Trade-offs

| 決策 | 選擇 | 理由 |
|------|------|------|
| 至少一次 vs 最多一次 | **至少一次** | 重複通知可接受，漏發不可接受 |
| 同步 vs 非同步 | **非同步（Queue）** | 解耦、容錯、可重試 |
| Push Token 儲存 | **獨立表** | 一個用戶多設備 |
| 使用者偏好快取 | **Redis TTL=5min** | 高頻讀取，偶爾不一致可接受 |

---

## 相關頁面

- [[sd-social-media-feed]] — Fan-out 相關概念
- [[sd-chat-system]] — 訊息佇列應用
- [[系統設計核心技術棧]] — Kafka / Redis 快速參考
