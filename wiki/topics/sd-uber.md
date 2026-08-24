---
title: "SD題解：叫車服務（Uber / Lyft）"
type: topic
tags: [system-design, uber, geohash, location, matching, golang, hard]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：叫車服務（Uber / Lyft）

> **難度**: 進階 ｜ **頻率**: 高 ｜ **代表**: Uber, Lyft, LINE TAXI

---

## RESHADED 快速分析

**R - Requirements**
- 乘客叫車、系統配對最近司機、即時追蹤位置
- 非功能：低延遲配對（<1s）、即時位置更新（每 4 秒）、高可用

**E - Estimation**
- DAU：1億乘客 + 1000萬司機
- 同時在線司機：100萬
- 位置更新：100萬司機 × 每 4 秒一次 = **250,000 寫入/秒**
- 叫車 QPS：每天 2000萬趟 / 86400 ≈ 230/s（配對需求）
- 地理查詢 QPS：配對時需查詢附近司機 ≈ 每秒數千次

---

## 核心挑戰：地理位置索引

如何高效找出「5公里內的所有司機」？

### Geohash（主流方案）

把地球劃分為遞迴格子，每個格子對應一個字串。

```
精度等級：
geohash 長度 1 → 5,000 km × 5,000 km
geohash 長度 4 → 39.1 km × 19.5 km
geohash 長度 6 → 1.2 km × 0.6 km  ← 叫車用這個
geohash 長度 8 → 38 m × 19 m
```

**關鍵性質**：相同前綴的 geohash → 地理上相鄰

---

## Go 實現

### Geohash 編碼

```go
package geo

import "math"

const base32 = "0123456789bcdefghjkmnpqrstuvwxyz"

// Encode 將 (lat, lng) 編碼為 geohash 字串
func Encode(lat, lng float64, precision int) string {
    var geohash []byte
    var minLat, maxLat = -90.0, 90.0
    var minLng, maxLng = -180.0, 180.0
    var isLng = true // 交替編碼經/緯度
    var bit, idx = 0, 0

    for len(geohash) < precision {
        if isLng {
            mid := (minLng + maxLng) / 2
            if lng > mid {
                idx = idx*2 + 1
                minLng = mid
            } else {
                idx = idx * 2
                maxLng = mid
            }
        } else {
            mid := (minLat + maxLat) / 2
            if lat > mid {
                idx = idx*2 + 1
                minLat = mid
            } else {
                idx = idx * 2
                maxLat = mid
            }
        }
        isLng = !isLng
        bit++
        if bit == 5 {
            geohash = append(geohash, base32[idx])
            bit, idx = 0, 0
        }
    }
    return string(geohash)
}

// Neighbors 取得相鄰的 8 個格子（防止邊界問題）
func Neighbors(hash string) []string {
    // 返回上下左右 + 四個對角共 8 個 geohash
    // 實際實作需計算偏移量，此處示意
    return []string{
        hash[:len(hash)-1] + "neighbor1",
        // ... 實際 8 個相鄰 geohash
    }
}

// Distance 計算兩點距離（Haversine 公式）
func Distance(lat1, lng1, lat2, lng2 float64) float64 {
    const earthRadius = 6371.0 // km

    dLat := toRad(lat2 - lat1)
    dLng := toRad(lng2 - lng1)
    a := math.Sin(dLat/2)*math.Sin(dLat/2) +
        math.Cos(toRad(lat1))*math.Cos(toRad(lat2))*
            math.Sin(dLng/2)*math.Sin(dLng/2)
    c := 2 * math.Atan2(math.Sqrt(a), math.Sqrt(1-a))
    return earthRadius * c
}

func toRad(deg float64) float64 {
    return deg * math.Pi / 180
}
```

### 司機位置服務

```go
package location

type DriverLocation struct {
    DriverID  string
    Lat, Lng  float64
    Geohash   string
    UpdatedAt time.Time
    Status    string // available / on_trip / offline
}

type LocationService struct {
    cache *redis.Client
    db    LocationDB
}

// UpdateLocation 司機每 4 秒更新位置
func (s *LocationService) UpdateLocation(ctx context.Context, driverID string, lat, lng float64) error {
    geohash := geo.Encode(lat, lng, 6) // 精度 6，約 1.2km 精度

    // 1. 更新司機的當前位置
    locKey := "driver:loc:" + driverID
    s.cache.HSet(ctx, locKey,
        "lat", lat,
        "lng", lng,
        "geohash", geohash,
        "updated_at", time.Now().Unix(),
    )
    s.cache.Expire(ctx, locKey, 30*time.Second) // 30秒無更新視為離線

    // 2. 更新地理索引（Geohash → 司機集合）
    // 先從舊 geohash 移除
    oldGeohash, _ := s.cache.HGet(ctx, locKey, "geohash").Result()
    if oldGeohash != "" && oldGeohash != geohash {
        s.cache.SRem(ctx, "geo:"+oldGeohash, driverID)
    }
    // 加入新 geohash
    s.cache.SAdd(ctx, "geo:"+geohash, driverID)
    s.cache.Expire(ctx, "geo:"+geohash, 60*time.Second)

    // 3. 廣播給正在追蹤此司機的乘客（透過 WebSocket）
    s.broadcastToPassenger(ctx, driverID, lat, lng)

    return nil
}

// FindNearbyDrivers 找指定位置附近的可用司機
func (s *LocationService) FindNearbyDrivers(ctx context.Context, lat, lng float64, radiusKm float64) ([]*DriverLocation, error) {
    geohash := geo.Encode(lat, lng, 6)

    // 查詢當前格子 + 8 個相鄰格子（防止邊界問題）
    searchArea := append(geo.Neighbors(geohash), geohash)

    var driverIDs []string
    for _, gh := range searchArea {
        ids, _ := s.cache.SMembers(ctx, "geo:"+gh).Result()
        driverIDs = append(driverIDs, ids...)
    }

    // 批次取得位置資訊並精確計算距離
    var nearby []*DriverLocation
    for _, driverID := range driverIDs {
        loc, err := s.getDriverLocation(ctx, driverID)
        if err != nil || loc.Status != "available" {
            continue
        }
        dist := geo.Distance(lat, lng, loc.Lat, loc.Lng)
        if dist <= radiusKm {
            nearby = append(nearby, loc)
        }
    }

    // 按距離排序
    sort.Slice(nearby, func(i, j int) bool {
        di := geo.Distance(lat, lng, nearby[i].Lat, nearby[i].Lng)
        dj := geo.Distance(lat, lng, nearby[j].Lat, nearby[j].Lng)
        return di < dj
    })

    return nearby, nil
}
```

### 配對服務

```go
type MatchingService struct {
    locationSvc *LocationService
    tripDB      TripDB
    notifier    WebSocketHub
}

type TripRequest struct {
    PassengerID  string
    PickupLat    float64
    PickupLng    float64
    DropoffLat   float64
    DropoffLng   float64
}

// MatchDriver 配對司機（最近可用司機）
func (m *MatchingService) MatchDriver(ctx context.Context, req TripRequest) (*Trip, error) {
    const maxRetries = 3
    const searchRadiusKm = 5.0

    for attempt := 0; attempt < maxRetries; attempt++ {
        // 1. 找附近司機
        drivers, err := m.locationSvc.FindNearbyDrivers(
            ctx, req.PickupLat, req.PickupLng,
            searchRadiusKm*float64(attempt+1)) // 每次重試擴大範圍
        if err != nil || len(drivers) == 0 {
            continue
        }

        // 2. 嘗試配對最近的司機
        for _, driver := range drivers {
            trip, err := m.offerTrip(ctx, driver.DriverID, req)
            if err == nil {
                return trip, nil
            }
            // 司機拒絕 / 已接單 → 試下一個
        }
    }
    return nil, errors.New("no available drivers nearby")
}

// offerTrip 向司機發送配對請求（司機有 10 秒接受）
func (m *MatchingService) offerTrip(ctx context.Context, driverID string, req TripRequest) (*Trip, error) {
    offer := &TripOffer{
        RequestID:   generateID(),
        PassengerID: req.PassengerID,
        PickupLat:   req.PickupLat,
        PickupLng:   req.PickupLng,
        ExpiresAt:   time.Now().Add(10 * time.Second),
    }

    // 透過 WebSocket 推送給司機 App
    m.notifier.Send(driverID, offer)

    // 等待司機回應（最多 10 秒）
    responseCh := m.waitForDriverResponse(driverID, offer.RequestID)
    select {
    case accepted := <-responseCh:
        if !accepted {
            return nil, errors.New("driver declined")
        }
        // 建立行程
        trip := &Trip{
            ID:          generateID(),
            DriverID:    driverID,
            PassengerID: req.PassengerID,
            Status:      "matched",
        }
        m.tripDB.Create(ctx, trip)
        return trip, nil
    case <-time.After(10 * time.Second):
        return nil, errors.New("driver did not respond")
    }
}
```

---

## 架構圖

```
司機 App ──WebSocket──→ Location Service ──→ Redis（Geohash 索引）
                                                     ↑
乘客叫車 ──────────────→ Matching Service ──查附近司機
                                ↓
                        WebSocket 推送給司機
                                ↓
                        司機接受 → Trip Service
                                ↓
乘客 App ←──WebSocket── Trip Service（即時位置更新）
```

---

## Trade-offs

| 決策 | 選擇 | 理由 |
|------|------|------|
| 位置索引 | Geohash | 實作簡單、Redis 原生支援；Quadtree 更精確但複雜 |
| 即時通訊 | WebSocket | 雙向即時，位置更新必須低延遲 |
| 配對策略 | 最近司機優先 | 簡單有效；進階版可加入 ETA 估算 |
| 位置更新頻率 | 4 秒 | 平衡即時性與電池/流量消耗 |

---

## 相關題解

- [[sd-chat-system|SD題解：即時聊天]] — WebSocket 連線管理
- [[sd-consistent-hashing|SD題解：一致性雜湊]] — Location Server 的水平擴展
