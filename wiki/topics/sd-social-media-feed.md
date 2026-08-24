---
title: "SD題解：社群媒體 Feed（Instagram / Twitter）"
type: topic
tags: [system-design, social-media, feed, fanout, golang, medium]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：社群媒體 Feed（Instagram / Twitter）

> **難度**: 中級 ｜ **頻率**: 極高（必練）｜ **代表**: Instagram, Twitter, Facebook

---

## RESHADED 快速分析

**E - Estimation**（Twitter 規模）
- DAU：2億
- 每天發文：5千萬
- 每用戶每天看 200 條 → 讀取 QPS：2億 × 200 / 86400 ≈ **463,000 QPS**（讀重！）
- 寫入 QPS：5千萬 / 86400 ≈ 580/s
- 讀寫比約 800:1

**結論**：極度讀重，Cache 是核心。

---

## 核心挑戰：Fanout 策略

### Fanout on Write（Push 模式）

發文時，把這篇文章推送到所有 follower 的 Feed 列表。

```
User A 發文 → Fanout 服務 → 推送到 B、C、D 各自的 Feed Cache
```

**優點**：讀取 Feed 超快（直接從 Redis List 取）
**缺點**：大 V（1000萬粉絲）發一篇文 → 1000萬次寫入

### Fanout on Read（Pull 模式）

用戶讀 Feed 時，才即時聚合所有 following 的最新文章。

```
User B 讀 Feed → 查詢 B 所有 following 的最新文章 → 排序 → 回傳
```

**優點**：寫入簡單，大 V 不影響寫入
**缺點**：讀取慢，following 100人 → 100次 DB 查詢

### 混合策略（生產環境採用）

```go
const (
    CelebThreshold = 1_000_000 // 粉絲 > 100萬視為大 V
)

func (s *FeedService) PublishPost(authorID string, post *Post) error {
    followerCount, _ := s.getFollowerCount(authorID)

    if followerCount < CelebThreshold {
        // 普通用戶：Fanout on Write
        return s.fanoutWrite(authorID, post)
    }
    // 大 V：只存文章，讀取時 Pull
    return s.storeCelebPost(authorID, post)
}
```

---

## Go 實現

### 資料模型

```go
type Post struct {
    ID        string    `json:"id"`
    AuthorID  string    `json:"author_id"`
    Content   string    `json:"content"`
    MediaURL  string    `json:"media_url,omitempty"`
    CreatedAt time.Time `json:"created_at"`
    Likes     int64     `json:"likes"`
}

// Feed 快取：Redis Sorted Set
// Key: feed:{userID}
// Member: postID
// Score: 發文時間戳（用於時序排序）
```

### Fanout on Write 服務

```go
type FeedService struct {
    postDB      PostRepository
    userDB      UserRepository
    cache       *redis.Client
    fanoutQueue MessageQueue // Kafka
}

// fanoutWrite 把文章推送到所有 follower 的 Feed
func (s *FeedService) fanoutWrite(authorID string, post *Post) error {
    // 1. 存文章到 DB
    if err := s.postDB.Save(post); err != nil {
        return err
    }

    // 2. 非同步 Fanout（透過 Kafka，避免阻塞發文請求）
    return s.fanoutQueue.Publish("fanout", FanoutMessage{
        PostID:   post.ID,
        AuthorID: authorID,
        Score:    float64(post.CreatedAt.UnixMilli()),
    })
}

// Fanout Worker（消費 Kafka 訊息）
func (s *FeedService) ProcessFanout(msg FanoutMessage) error {
    // 取得所有 follower
    followers, err := s.userDB.GetFollowers(msg.AuthorID)
    if err != nil {
        return err
    }

    // 批次寫入 Redis（Pipeline 降低 RTT）
    pipe := s.cache.Pipeline()
    for _, followerID := range followers {
        feedKey := "feed:" + followerID
        pipe.ZAdd(context.Background(), feedKey, redis.Z{
            Score:  msg.Score,
            Member: msg.PostID,
        })
        // 每個 Feed 只保留最新 1000 條
        pipe.ZRemRangeByRank(context.Background(), feedKey, 0, -1001)
        pipe.Expire(context.Background(), feedKey, 7*24*time.Hour)
    }
    _, err = pipe.Exec(context.Background())
    return err
}
```

### 讀取 Feed（混合策略）

```go
func (s *FeedService) GetFeed(ctx context.Context, userID string, page, pageSize int) ([]*Post, error) {
    offset := int64(page * pageSize)
    limit := int64(pageSize)

    // 1. 從 Redis 取用戶的 Push Feed（普通用戶的文章）
    feedKey := "feed:" + userID
    postIDs, err := s.cache.ZRevRange(ctx, feedKey, offset, offset+limit-1).Result()

    // 2. 取用戶 following 的大 V 文章（Pull）
    celebPostIDs, _ := s.getCelebPosts(ctx, userID, pageSize)

    // 3. 合併並排序
    allIDs := mergeAndSort(postIDs, celebPostIDs)[:pageSize]

    // 4. 批次從 DB / Cache 取文章內容
    return s.batchGetPosts(ctx, allIDs)
}

func (s *FeedService) getCelebPosts(ctx context.Context, userID string, limit int) ([]string, error) {
    // 取出用戶 following 的大 V 列表
    celebs, _ := s.userDB.GetFollowingCelebs(userID)

    // 並行取每個大 V 的最新文章
    var mu sync.Mutex
    var allPostIDs []string
    var wg sync.WaitGroup

    for _, celebID := range celebs {
        wg.Add(1)
        go func(cID string) {
            defer wg.Done()
            ids, _ := s.postDB.GetRecentPostIDs(ctx, cID, limit)
            mu.Lock()
            allPostIDs = append(allPostIDs, ids...)
            mu.Unlock()
        }(celebID)
    }
    wg.Wait()
    return allPostIDs, nil
}
```

### 按讚計數（Counter 去重）

```go
// 按讚：Redis SADD 去重 + 非同步同步到 DB
func (s *FeedService) LikePost(ctx context.Context, userID, postID string) error {
    likeKey := "likes:" + postID

    // SADD 回傳 1 = 新增成功，0 = 已按過
    added, err := s.cache.SAdd(ctx, likeKey, userID).Result()
    if err != nil || added == 0 {
        return err // 已按過，忽略
    }

    // 更新 Like 計數
    s.cache.Incr(ctx, "likes_count:"+postID)

    // 非同步寫 DB
    go s.postDB.IncrementLikes(postID)
    return nil
}
```

---

## 架構圖

```
POST /post ──→ Post Service ──→ Kafka ──→ Fanout Workers
                   ↓                           ↓
               Posts DB               Feed Redis（Sorted Set）

GET /feed ───→ Feed Service
                   ├─ Redis（普通用戶 Push Feed）
                   └─ Posts DB（大 V Pull Feed）
                          ↓
                     合併排序 → 回傳
```

---

## Trade-offs 辯護

| 決策 | 選擇 | 理由 |
|------|------|------|
| Fanout | 混合策略 | 純 Write 大 V 成本太高；純 Read 太慢 |
| Feed 存儲 | Redis Sorted Set（score = 時間戳）| 天然支援時序排序，O(log n) 插入 |
| 非同步 Fanout | Kafka | 發文不應等 Fanout 完成，解耦 |
| Feed 大小限制 | 保留最新 1000 條 | 用戶不會翻到很舊的 Feed |

---

## 相關題解

- [[sd-distributed-cache|SD題解：分散式快取]] — Redis Feed 快取
- [[sd-chat-system|SD題解：即時聊天]] — 同樣使用 Kafka 解耦
