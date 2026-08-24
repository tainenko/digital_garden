---
title: 附近地點搜尋（Yelp / Google Places）
type: topic
tags: [system-design, interview, geospatial, location, search]
created: 2026-04-20
updated: 2026-04-20
---

# 附近地點搜尋（Yelp / Google Places）

難度：中級｜核心技術：Geohash、四叉樹、空間索引、搜尋排序

---

## RESHADED 分析

### Requirements
**Functional:**
- 按目前位置搜尋附近的商家（半徑 500m / 2km / 5km）
- 按類別篩選（餐廳、加油站、ATM）
- 查看商家詳情、評論
- 用戶可新增評論和評分
- 商家可更新自身資訊（地址、營業時間）

**Non-functional:**
- 讀多寫少（100:1）
- 低搜尋延遲：＜200ms
- 每日活躍用戶：1億
- 商家總數：2億

### Estimates
```
商家資料：2億 × 1KB = 200 GB（可放入單台大型機器）
搜尋 QPS：1億 DAU × 5次/天 / 86400 ≈ 6,000 QPS
評論寫入：1億 × 0.1次/天 / 86400 ≈ 116 QPS
```

### High-Level Design

```
手機 App
    ↓ HTTPS
API Gateway（Load Balancer）
    ├─ 位置搜尋服務 (Location Search Service)
    │    ├─ Geohash Index (Redis)
    │    └─ 商家 DB (PostgreSQL + PostGIS)
    ├─ 商家詳情服務 (Business Service)
    │    └─ 商家詳情 DB + CDN 快取
    └─ 評論服務 (Review Service)
         └─ 評論 DB (PostgreSQL)
```

---

## 核心技術深探

### 1. Geohash 空間索引

Geohash 將二維座標編碼為一維字串，前綴相同的 hash 在地理上靠近：

```go
// Geohash 編碼（精度 6 = 約 1.2km × 0.6km）
const base32 = "0123456789bcdefghjkmnpqrstuvwxyz"

func Encode(lat, lng float64, precision int) string {
    minLat, maxLat := -90.0, 90.0
    minLng, maxLng := -180.0, 180.0

    var hash []byte
    var bits, hashLen int
    isLng := true  // 從 lng 開始交替

    for hashLen < precision {
        var mid float64
        var bit int

        if isLng {
            mid = (minLng + maxLng) / 2
            if lng >= mid {
                bit = 1
                minLng = mid
            } else {
                maxLng = mid
            }
        } else {
            mid = (minLat + maxLat) / 2
            if lat >= mid {
                bit = 1
                minLat = mid
            } else {
                maxLat = mid
            }
        }
        isLng = !isLng

        bits = (bits << 1) | bit
        if bits_count := (len(hash)*5 + 5); bits_count%5 == 0 {
            hash = append(hash, base32[bits])
            bits = 0
            hashLen++
        }
    }
    return string(hash)
}

// 取得 Geohash 的 8 個鄰居（解決跨邊界問題）
func Neighbors(hash string) []string {
    // 返回上下左右及四個對角的 geohash
    return geohash.Neighbors(hash)
}
```

### 2. 附近搜尋實作

```go
type LocationSearchService struct {
    redis    *redis.Client
    businessDB *sql.DB
}

func (s *LocationSearchService) SearchNearby(
    lat, lng float64, radiusKm float64, category string,
) ([]Business, error) {
    // 1. 根據搜尋半徑決定 Geohash 精度
    precision := radiusToPrecision(radiusKm)
    // radiusKm=0.5 → precision=7, radiusKm=2 → precision=6, radiusKm=5 → precision=5

    // 2. 取目標格子 + 8個鄰居
    centerHash := Encode(lat, lng, precision)
    searchHashes := append([]string{centerHash}, Neighbors(centerHash)...)

    // 3. 從 Redis 批次取得候選商家
    var candidates []Business
    for _, hash := range searchHashes {
        key := fmt.Sprintf("geohash:%s", hash)
        ids, err := s.redis.SMembers(context.Background(), key).Result()
        if err != nil {
            continue
        }
        for _, id := range ids {
            b, _ := s.businessDB.GetBusiness(id)
            candidates = append(candidates, b)
        }
    }

    // 4. 精確距離過濾 + 排序
    var results []Business
    for _, b := range candidates {
        dist := haversine(lat, lng, b.Lat, b.Lng)
        if dist <= radiusKm {
            b.Distance = dist
            results = append(results, b)
        }
    }

    // 5. 類別過濾
    if category != "" {
        results = filterByCategory(results, category)
    }

    // 6. 依距離排序（可加入評分權重）
    sort.Slice(results, func(i, j int) bool {
        return results[i].Distance < results[j].Distance
    })

    return results[:min(len(results), 20)], nil
}

func haversine(lat1, lng1, lat2, lng2 float64) float64 {
    const R = 6371.0  // 地球半徑（km）
    dLat := (lat2 - lat1) * math.Pi / 180
    dLng := (lng2 - lng1) * math.Pi / 180
    a := math.Sin(dLat/2)*math.Sin(dLat/2) +
        math.Cos(lat1*math.Pi/180)*math.Cos(lat2*math.Pi/180)*
            math.Sin(dLng/2)*math.Sin(dLng/2)
    c := 2 * math.Atan2(math.Sqrt(a), math.Sqrt(1-a))
    return R * c
}

func radiusToPrecision(radiusKm float64) int {
    switch {
    case radiusKm <= 0.5:  return 7  // ±78m
    case radiusKm <= 2.0:  return 6  // ±610m
    case radiusKm <= 20.0: return 5  // ±2.4km
    default:               return 4  // ±20km
    }
}
```

### 3. 商家資料模型

```sql
CREATE TABLE businesses (
    id          BIGINT PRIMARY KEY,
    name        VARCHAR(255),
    category    VARCHAR(64),
    lat         DECIMAL(9,6),
    lng         DECIMAL(9,6),
    geohash6    CHAR(6),   -- 用於空間索引
    geohash7    CHAR(7),
    address     TEXT,
    rating      DECIMAL(2,1),
    review_count INT,
    created_at  TIMESTAMP,
    updated_at  TIMESTAMP
);

CREATE INDEX idx_businesses_geohash6 ON businesses(geohash6);
CREATE INDEX idx_businesses_category ON businesses(category);

CREATE TABLE reviews (
    id          BIGINT PRIMARY KEY,
    business_id BIGINT,
    user_id     BIGINT,
    rating      TINYINT,    -- 1~5
    content     TEXT,
    created_at  TIMESTAMP,
    FOREIGN KEY (business_id) REFERENCES businesses(id)
);
```

### 4. Redis Geohash 索引維護

```go
// 新增商家時建立 Geohash 索引
func (s *LocationSearchService) IndexBusiness(b Business) error {
    pipe := s.redis.Pipeline()

    for precision := 4; precision <= 7; precision++ {
        hash := Encode(b.Lat, b.Lng, precision)
        key := fmt.Sprintf("geohash:%s", hash)
        pipe.SAdd(context.Background(), key, b.ID)
        pipe.Expire(context.Background(), key, 24*time.Hour)
    }

    _, err := pipe.Exec(context.Background())
    return err
}

// 商家移動時更新索引
func (s *LocationSearchService) UpdateBusinessLocation(
    businessID int64, oldLat, oldLng, newLat, newLng float64,
) error {
    pipe := s.redis.Pipeline()

    for precision := 4; precision <= 7; precision++ {
        oldHash := Encode(oldLat, oldLng, precision)
        newHash := Encode(newLat, newLng, precision)

        if oldHash != newHash {
            pipe.SRem(context.Background(),
                fmt.Sprintf("geohash:%s", oldHash), businessID)
            pipe.SAdd(context.Background(),
                fmt.Sprintf("geohash:%s", newHash), businessID)
        }
    }

    _, err := pipe.Exec(context.Background())
    return err
}
```

### 5. 搜尋排序邏輯

```go
type RankedBusiness struct {
    Business
    Score float64
}

func rankBusinesses(businesses []Business, userLat, userLng float64) []RankedBusiness {
    ranked := make([]RankedBusiness, 0, len(businesses))

    for _, b := range businesses {
        dist := haversine(userLat, userLng, b.Lat, b.Lng)

        // 綜合評分：距離(60%) + 評分(30%) + 熱門度(10%)
        distScore := 1.0 / (1.0 + dist)                    // 距離越近越高
        ratingScore := float64(b.Rating) / 5.0              // 正規化 1~5
        popularityScore := math.Log10(float64(b.ReviewCount+1)) / 5.0

        score := distScore*0.6 + ratingScore*0.3 + popularityScore*0.1

        ranked = append(ranked, RankedBusiness{
            Business: b,
            Score:    score,
        })
    }

    sort.Slice(ranked, func(i, j int) bool {
        return ranked[i].Score > ranked[j].Score
    })

    return ranked
}
```

---

## Geohash vs 四叉樹（Quadtree）比較

| 方式 | 優點 | 缺點 |
|------|------|------|
| **Geohash** | 實作簡單、字串前綴查詢、Redis 原生支援 | 邊界誤差需查鄰居補償 |
| **Quadtree** | 動態分割、密集區域自動細化 | 實作複雜、需維護樹狀結構 |
| **PostGIS** | 功能完整、SQL 整合 | 單機限制、擴展性差 |

> **推薦**：Yelp 規模用 Geohash + Redis；超大規模（Google Maps）用分散式四叉樹。

---

## 相關頁面

- [[sd-uber]] — 同為地理位置系統，Geohash 深度比較
- [[系統設計核心技術棧]] — Redis 空間命令
