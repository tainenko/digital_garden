---
title: "SD題解：音樂串流（Spotify）"
type: topic
tags: [system-design, music-streaming, offline, drm, golang, medium]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：音樂串流（Spotify）

> **難度**: 中級 ｜ **頻率**: 中等 ｜ **代表**: Spotify, Apple Music, KKBOX

---

## 與影片串流的核心差異

| | 影片串流 | 音樂串流 |
|--|--|--|
| 檔案大小 | GB 級 | MB 級（3分鐘歌曲 ~3–10MB） |
| 轉碼 | 多解析度 HLS | 多碼率 MP3/AAC（128k/256k/320k）|
| 離線 | 少見 | 核心功能（下載後離線播放）|
| 版權 | YouTube 版權管理 | 唱片公司授權、DRM 加密 |
| 歌詞 | 少 | 同步歌詞（LRC 格式）|
| 推薦 | 影片推薦 | 每日推薦歌單是核心產品 |

---

## RESHADED 快速分析

**R - Requirements**
- 搜尋/瀏覽音樂、播放、建立歌單
- 離線下載（Premium 功能）
- 歌詞同步顯示
- 推薦算法（每日推薦、Discovery Weekly）

**E - Estimation**
- MAU：6億（Spotify 實際數字）
- 歌曲總數：1億首
- 每首歌 ~5MB（256kbps AAC）→ 儲存 500TB
- DAU：2億，每人每天聽 30 首 → 播放 QPS：2億 × 30 / 86400 ≈ 70,000

---

## Go 實現

### 1. 音樂元數據與搜尋

```go
package music

type Track struct {
    ID          string   `json:"id"`
    Title       string   `json:"title"`
    Artist      []string `json:"artist"`
    Album       string   `json:"album"`
    Duration    int      `json:"duration"` // seconds
    Genres      []string `json:"genres"`
    ReleaseDate string   `json:"release_date"`
    Popularity  int      `json:"popularity"` // 0-100
    AudioURLs   map[string]string `json:"audio_urls"` // quality → CDN URL
    LyricsID    string   `json:"lyrics_id,omitempty"`
}

type SearchService struct {
    es    *elasticsearch.Client // 全文搜尋
    cache *redis.Client
}

// Search 搜尋音樂（Elasticsearch）
func (s *SearchService) Search(ctx context.Context, query string, offset, limit int) ([]*Track, int, error) {
    cacheKey := fmt.Sprintf("search:%s:%d:%d", query, offset, limit)

    // 熱門搜尋詞快取
    if cached, err := s.cache.Get(ctx, cacheKey).Bytes(); err == nil {
        var results []*Track
        json.Unmarshal(cached, &results)
        return results, len(results), nil
    }

    res, err := s.es.Search(
        s.es.Search.WithIndex("tracks"),
        s.es.Search.WithBody(strings.NewReader(fmt.Sprintf(`{
            "query": {
                "multi_match": {
                    "query": %q,
                    "fields": ["title^3", "artist^2", "album"],
                    "fuzziness": "AUTO"
                }
            },
            "sort": [{"popularity": "desc"}],
            "from": %d,
            "size": %d
        }`, query, offset, limit))),
    )
    // ... 解析結果
    return nil, 0, err
}
```

### 2. 音訊串流（帶 Range 支援）

音樂播放器需要支援快轉（HTTP Range Request）：

```go
func streamAudioHandler(w http.ResponseWriter, r *http.Request) {
    trackID := r.PathValue("id")
    quality := r.URL.Query().Get("q") // 128/256/320
    if quality == "" {
        quality = "256"
    }

    // 取得 CDN 簽名 URL（防止直接分享）
    audioURL, err := getSignedAudioURL(trackID, quality, 1*time.Hour)
    if err != nil {
        http.Error(w, "Not Found", http.StatusNotFound)
        return
    }

    // 直接重定向到 CDN（讓 CDN 處理 Range Request）
    // CDN 支援 HTTP 206 Partial Content，讓播放器快轉
    http.Redirect(w, r, audioURL, http.StatusTemporaryRedirect)
}

// getSignedAudioURL 生成有時效的簽名 URL（防止盜鏈）
func getSignedAudioURL(trackID, quality string, ttl time.Duration) (string, error) {
    expires := time.Now().Add(ttl).Unix()
    path := fmt.Sprintf("/audio/%s/%skbps.aac", trackID, quality)
    // HMAC 簽名
    mac := hmac.New(sha256.New, []byte(cdnSecret))
    mac.Write([]byte(fmt.Sprintf("%s:%d", path, expires)))
    sig := base64.URLEncoding.EncodeToString(mac.Sum(nil))
    return fmt.Sprintf("https://cdn.music.com%s?exp=%d&sig=%s", path, expires, sig), nil
}
```

### 3. 離線下載（DRM 加密）

Premium 用戶可以下載歌曲離線聆聽，但需防止盜版：

```go
type OfflineService struct {
    storage ObjectStorage
    db      OfflineDB
    crypto  DRMService
}

// DownloadTrack 下載加密音訊到本地
func (s *OfflineService) DownloadTrack(ctx context.Context, userID, trackID string) (*DownloadInfo, error) {
    // 1. 驗證用戶是否為 Premium
    if !s.isPremium(ctx, userID) {
        return nil, errors.New("premium subscription required")
    }

    // 2. 為此用戶生成唯一的加密 Key（每個用戶的 Key 不同）
    encKey, err := s.crypto.GenerateUserKey(userID, trackID)
    if err != nil {
        return nil, err
    }

    // 3. 取得加密後的音訊下載 URL
    // 音訊本身已用 AES-128 加密存在 S3（Common Encryption）
    downloadURL, _ := s.storage.GetSignedURL(
        fmt.Sprintf("drm/%s/encrypted.aac", trackID), 24*time.Hour)

    // 4. 儲存下載記錄（過期時自動刪除授權）
    s.db.RecordDownload(ctx, userID, trackID, time.Now().Add(30*24*time.Hour))

    return &DownloadInfo{
        DownloadURL:    downloadURL,
        EncryptionKey:  encKey,
        ExpiresAt:      time.Now().Add(30 * 24 * time.Hour),
    }, nil
}

// ValidateOfflineAccess 播放離線歌曲前驗證授權
func (s *OfflineService) ValidateOfflineAccess(ctx context.Context, userID, trackID string) bool {
    record, err := s.db.GetDownload(ctx, userID, trackID)
    if err != nil {
        return false
    }
    // 檢查是否過期 & 用戶仍為 Premium
    return time.Now().Before(record.ExpiresAt) && s.isPremium(ctx, userID)
}
```

### 4. 歌詞同步

```go
type LyricLine struct {
    TimeMs  int    `json:"time_ms"` // 幾毫秒時顯示
    Text    string `json:"text"`
}

type LyricsService struct {
    db    LyricsDB
    cache *redis.Client
}

func (l *LyricsService) GetSyncedLyrics(ctx context.Context, trackID string) ([]*LyricLine, error) {
    cacheKey := "lyrics:" + trackID
    if cached, err := l.cache.Get(ctx, cacheKey).Bytes(); err == nil {
        var lines []*LyricLine
        json.Unmarshal(cached, &lines)
        return lines, nil
    }

    lines, err := l.db.GetLyrics(ctx, trackID)
    if err != nil {
        return nil, err
    }

    // 快取 24 小時（歌詞不常變）
    data, _ := json.Marshal(lines)
    l.cache.Set(ctx, cacheKey, data, 24*time.Hour)
    return lines, nil
}
```

### 5. 推薦系統（簡化版 Collaborative Filtering）

```go
type RecommendService struct {
    model RecommendModel
    cache *redis.Client
}

// GetDailyMix 每日推薦歌單
func (r *RecommendService) GetDailyMix(ctx context.Context, userID string) ([]*Track, error) {
    cacheKey := fmt.Sprintf("daily_mix:%s:%s", userID, today())

    if cached, err := r.cache.LRange(ctx, cacheKey, 0, 29).Result(); err == nil && len(cached) > 0 {
        return r.fetchTracks(ctx, cached)
    }

    // 1. 取用戶最近 30 天的聆聽歷史
    history, _ := r.getListeningHistory(ctx, userID, 30)

    // 2. 基於協同過濾找相似用戶 → 取他們喜歡但此用戶未聽過的歌
    // （實際 Spotify 使用更複雜的深度學習模型）
    candidates, _ := r.model.GetCandidates(userID, history, 100)

    // 3. 重新排序（多樣性 + 個人化）
    recommended := r.rerank(candidates, history, 30)

    // 4. 快取到明天
    trackIDs := extractIDs(recommended)
    tomorrow := time.Until(nextMidnight())
    r.cache.RPush(ctx, cacheKey, trackIDs)
    r.cache.Expire(ctx, cacheKey, tomorrow)

    return recommended, nil
}
```

---

## 架構圖

```
[播放流程]
App → API Gateway → Track Service（元數據）
                 ↓
           CDN（音訊檔，全球分發）
           帶 Range Request 支援

[下載流程]
App → Offline Service（驗證 Premium）→ S3（DRM 加密音訊）
       ↓
   本地儲存（加密，App 持有解密 Key）

[推薦流程]
每日定時任務 → 協同過濾模型 → Redis（預計算結果）
App → Recommend Service → Redis Cache → 回傳歌單
```

---

## 相關題解

- [[sd-video-streaming|SD題解：影片串流]] — 相似的 CDN + 轉碼架構
- [[sd-distributed-cache|SD題解：分散式快取]] — Redis 快取策略
