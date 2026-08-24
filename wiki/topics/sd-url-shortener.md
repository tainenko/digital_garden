---
title: "SD題解：URL Shortener（短網址服務）"
type: topic
tags: [system-design, url-shortener, base62, golang, easy]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：URL Shortener（短網址服務）

> **難度**: 入門 ｜ **頻率**: 極高（必練基礎題）｜ **代表**: TinyURL, bit.ly

---

## RESHADED 快速分析

**R - Requirements**
- 功能：輸入長網址 → 輸出短碼；短碼重定向到長網址
- 非功能：讀重（重定向:縮短 = 100:1）、高可用、低延遲（<10ms 重定向）

**E - Estimation**（假設中型規模）
- DAU：1億
- 每天縮短：1億/10 = 1,000萬筆
- 每天重定向：1,000萬 × 100 = 10億次
- 寫入 QPS：1,000萬 / 86,400 ≈ 116/s
- 讀取 QPS：116 × 100 = 11,600/s（峰值 3倍 ≈ 35,000/s）
- 5年儲存：1,000萬/天 × 365 × 5 × 500B ≈ **9TB**

**S - Storage**：NoSQL KV（DynamoDB），short_code → long_url

**H - High-level**：Client → CDN/LB → API Server → Redis Cache → DB

**A - APIs**
- `POST /shorten` body: `{url: "https://..."}`  → `{short_url: "https://short.ly/abc123"}`
- `GET /{code}` → `302 Redirect` to long URL

**D - Detailed**：短碼生成策略（見下方）

**E - Evaluation**：Hash 衝突問題、Hot Key 問題

**D - Distinctive**：短碼生成是最獨特元件

---

## 核心：短碼生成策略比較

| 策略 | 原理 | 優點 | 缺點 |
|------|------|------|------|
| MD5/SHA256 Hash | Hash(url) 取前 7 位 | 相同 URL → 相同短碼 | 衝突需重試 |
| 自增 ID + Base62 | 全域自增 ID 轉 Base62 | 無衝突，簡單 | 需要全域唯一 ID 服務 |
| UUID | 隨機 UUID 取前 7 位 | 分散式友好 | 可能衝突 |
| Snowflake ID + Base62 | Twitter Snowflake | 分散式、有序、無衝突 | 架構複雜度略高 |

**面試首選**：Base62 + 全域 ID（最乾淨，無衝突）

---

## Go 實現

### Base62 編碼

```go
package urlshortener

import (
    "strings"
)

const charset = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
const base = int64(len(charset)) // 62

// Encode 將數字 ID 轉為 Base62 短碼
func Encode(id int64) string {
    if id == 0 {
        return string(charset[0])
    }
    var sb strings.Builder
    for id > 0 {
        sb.WriteByte(charset[id%base])
        id /= base
    }
    // 反轉
    result := []byte(sb.String())
    for i, j := 0, len(result)-1; i < j; i, j = i+1, j-1 {
        result[i], result[j] = result[j], result[i]
    }
    return string(result)
}

// Decode 將 Base62 短碼轉回數字 ID
func Decode(code string) int64 {
    var id int64
    for _, c := range code {
        id = id*base + int64(strings.IndexRune(charset, c))
    }
    return id
}

// 62^7 = 3,521億 → 可支援數十億條短網址
// 7個字元已足夠，例如 "gB8k2mZ"
```

### URL Shortener 服務

```go
package urlshortener

import (
    "context"
    "crypto/md5"
    "errors"
    "fmt"
    "sync/atomic"
    "time"

    "github.com/redis/go-redis/v9"
)

type URLRecord struct {
    ShortCode  string    `json:"short_code"`
    LongURL    string    `json:"long_url"`
    CreatedAt  time.Time `json:"created_at"`
    ExpiresAt  *time.Time `json:"expires_at,omitempty"`
    ClickCount int64     `json:"click_count"`
}

type URLShortenerService struct {
    db      Database    // 介面，可換 DynamoDB / PostgreSQL
    cache   *redis.Client
    counter int64       // 模擬全域自增 ID（生產環境用 DB sequence 或 Snowflake）
    baseURL string
}

func NewService(db Database, cache *redis.Client, baseURL string) *URLShortenerService {
    return &URLShortenerService{
        db:      db,
        cache:   cache,
        baseURL: baseURL,
    }
}

// Shorten 縮短 URL
func (s *URLShortenerService) Shorten(ctx context.Context, longURL string) (string, error) {
    // 1. 驗證 URL
    if !isValidURL(longURL) {
        return "", errors.New("invalid URL")
    }

    // 2. 檢查是否已經縮短過（相同 URL 回傳相同短碼）
    existingCode, err := s.findExistingCode(ctx, longURL)
    if err == nil && existingCode != "" {
        return s.baseURL + "/" + existingCode, nil
    }

    // 3. 生成唯一 ID → Base62 短碼
    id := atomic.AddInt64(&s.counter, 1) // 生產環境用 DB sequence
    shortCode := Encode(id)

    // 4. 存入 DB
    record := &URLRecord{
        ShortCode: shortCode,
        LongURL:   longURL,
        CreatedAt: time.Now(),
    }
    if err := s.db.Save(ctx, record); err != nil {
        return "", fmt.Errorf("failed to save: %w", err)
    }

    // 5. 存入 Cache（預熱）
    s.cache.Set(ctx, "url:"+shortCode, longURL, 24*time.Hour)

    return s.baseURL + "/" + shortCode, nil
}

// Resolve 短碼還原長網址
func (s *URLShortenerService) Resolve(ctx context.Context, shortCode string) (string, error) {
    // 1. 先查 Cache（Cache-Aside 模式）
    cached, err := s.cache.Get(ctx, "url:"+shortCode).Result()
    if err == nil {
        // Cache Hit → 非同步更新點擊計數
        go s.incrementClickCount(shortCode)
        return cached, nil
    }

    // 2. Cache Miss → 查 DB
    record, err := s.db.FindByCode(ctx, shortCode)
    if err != nil {
        return "", errors.New("short URL not found")
    }

    // 3. 檢查是否過期
    if record.ExpiresAt != nil && time.Now().After(*record.ExpiresAt) {
        return "", errors.New("short URL expired")
    }

    // 4. 回填 Cache
    s.cache.Set(ctx, "url:"+shortCode, record.LongURL, 24*time.Hour)
    go s.incrementClickCount(shortCode)

    return record.LongURL, nil
}

// findExistingCode 用 MD5(url) 查找是否已存在（防止同 URL 產生多個短碼）
func (s *URLShortenerService) findExistingCode(ctx context.Context, longURL string) (string, error) {
    urlHash := fmt.Sprintf("%x", md5.Sum([]byte(longURL)))
    return s.cache.Get(ctx, "urlhash:"+urlHash).Result()
}

func (s *URLShortenerService) incrementClickCount(shortCode string) {
    ctx := context.Background()
    s.db.IncrementClicks(ctx, shortCode)
}
```

### HTTP Handler

```go
package handler

import (
    "encoding/json"
    "net/http"
)

type Handler struct {
    service *URLShortenerService
}

func (h *Handler) Shorten(w http.ResponseWriter, r *http.Request) {
    var req struct {
        URL       string `json:"url"`
        ExpiresIn int    `json:"expires_in"` // 秒，可選
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, "Bad Request", http.StatusBadRequest)
        return
    }

    shortURL, err := h.service.Shorten(r.Context(), req.URL)
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]string{"short_url": shortURL})
}

func (h *Handler) Redirect(w http.ResponseWriter, r *http.Request) {
    // 從 URL path 取短碼，如 /abc123
    shortCode := r.PathValue("code")

    longURL, err := h.service.Resolve(r.Context(), shortCode)
    if err != nil {
        http.Error(w, "Not Found", http.StatusNotFound)
        return
    }

    // 302（臨時重定向）而非 301（永久），以便統計點擊
    http.Redirect(w, r, longURL, http.StatusFound)
}
```

---

## 延伸問題

### Q: 為什麼用 302 不用 301？

- 301（永久）：瀏覽器快取，後續請求不經過服務器 → 無法統計點擊
- 302（臨時）：每次都經過服務器 → 可統計點擊數、可更新目標 URL

### Q: 如何防止惡意短碼（釣魚網站）？

加入 URL 黑名單檢查：
```go
func (s *URLShortenerService) checkBlacklist(url string) bool {
    // 1. 查本地黑名單（Redis Set）
    // 2. 整合 Google Safe Browsing API
    return s.cache.SIsMember(context.Background(), "blacklist", extractDomain(url)).Val()
}
```

### Q: Custom Alias 怎麼做？

```go
// 允許用戶指定短碼，如 short.ly/my-brand
func (s *URLShortenerService) ShortenWithAlias(ctx context.Context, longURL, alias string) (string, error) {
    // 1. 驗證 alias 格式（字母數字，不超過 32 字元）
    // 2. 檢查是否已被佔用
    if _, err := s.db.FindByCode(ctx, alias); err == nil {
        return "", errors.New("alias already taken")
    }
    // 3. 直接用 alias 作為短碼存入 DB
    ...
}
```

---

## 相關題解

- [[sd-consistent-hashing|SD題解：一致性雜湊]] — 分散式快取的核心
- [[sd-rate-limiter|SD題解：Rate Limiter]] — 防止濫用縮短服務
- [[sd-distributed-cache|SD題解：分散式快取]] — LRU Cache 實現
