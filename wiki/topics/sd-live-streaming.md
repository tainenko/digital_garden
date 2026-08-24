---
title: 直播系統（Live Streaming / Twitch）
type: topic
tags: [system-design, interview, streaming, rtmp, webrtc, cdn]
created: 2026-04-20
updated: 2026-04-20
---

# 直播系統（Live Streaming / Twitch）

難度：進階｜核心技術：RTMP 推流、HLS/DASH、CDN 分發、邊緣節點、即時互動

---

## RESHADED 分析

### Requirements
**Functional:**
- 主播可開始/結束直播
- 觀眾可以觀看直播（低延遲 < 5秒）
- 直播聊天室（即時彈幕）
- 直播錄影（直播結束後可回看）
- 支援 1080p/720p/480p 多碼率

**Non-functional:**
- 同時在線主播：10萬人
- 同時觀眾：1億人
- 推流延遲（主播→觀眾）：＜5秒
- 可用性：99.99%

### Estimates
```
10萬主播同時推流，每路 6 Mbps（1080p）：
  上行頻寬: 10萬 × 6 Mbps = 600 Gbps

1億觀眾觀看，每人 4 Mbps：
  下行頻寬: 1億 × 4 Mbps = 400 Tbps（需 CDN 分發）

每小時錄影：1080p = 6 Mbps × 3600 = 2.7 GB/小時
10萬主播每日儲存：10萬 × 24h × 2.7 GB = 6.48 PB/天
```

### High-Level Design

```
主播端 (OBS / 手機)
    ↓ RTMP/SRT 推流
就近接入點 (Edge Ingestion Nodes)
    ↓ 轉碼
轉碼叢集 (Transcoding Cluster)
    ↓ HLS 分片
源站 Origin Servers
    ↓ 拉取
CDN 邊緣節點 (全球 PoPs)
    ↓ HTTP/HLS
觀眾端 (HTML5 Player)

即時互動（彈幕）：
主播/觀眾 → WebSocket Gateway → 聊天服務 → Redis Pub/Sub → 廣播
```

---

## 核心技術深探

### 1. 推流接入：RTMP → HLS 轉換

```go
// RTMP 推流接入處理器
type RTMPHandler struct {
    transcoder  TranscoderService
    storage     StorageService
    streamState *StreamStateManager
}

type StreamInfo struct {
    StreamKey  string
    UserID     int64
    Resolution []string  // ["1080p", "720p", "480p"]
    Status     string    // live, ended
    StartedAt  time.Time
}

func (h *RTMPHandler) OnPublish(streamKey string, conn RTMPConn) error {
    // 1. 驗證推流金鑰
    userID, err := h.streamState.ValidateStreamKey(streamKey)
    if err != nil {
        conn.Close()
        return err
    }

    // 2. 建立直播 Session
    stream := &StreamInfo{
        StreamKey:  streamKey,
        UserID:     userID,
        Resolution: []string{"1080p", "720p", "480p"},
        Status:     "live",
        StartedAt:  time.Now(),
    }
    h.streamState.SetLive(stream)

    // 3. 啟動轉碼（非同步）
    go h.transcoder.StartTranscoding(conn, stream)

    return nil
}
```

### 2. HLS 轉碼與分片

```go
type TranscoderService struct {
    storage StorageService
    cdn     CDNService
}

// 每個直播建立多個碼率輸出
func (t *TranscoderService) StartTranscoding(
    input RTMPConn, stream *StreamInfo,
) {
    profiles := []TranscodeProfile{
        {Resolution: "1080p", Bitrate: 6000, Height: 1080},
        {Resolution: "720p",  Bitrate: 3000, Height: 720},
        {Resolution: "480p",  Bitrate: 1500, Height: 480},
    }

    // 每 2 秒產生一個 .ts 分片
    segmentDuration := 2 * time.Second
    segmentIndex := 0

    for {
        segment, err := input.ReadSegment(segmentDuration)
        if err != nil {
            break  // 直播結束
        }

        var wg sync.WaitGroup
        for _, profile := range profiles {
            wg.Add(1)
            go func(p TranscodeProfile) {
                defer wg.Done()

                // FFmpeg 轉碼
                transcodedData := ffmpegTranscode(segment, p)

                // 上傳分片到 Object Storage
                segmentPath := fmt.Sprintf(
                    "live/%s/%s/seg%06d.ts",
                    stream.StreamKey, p.Resolution, segmentIndex,
                )
                t.storage.Upload(segmentPath, transcodedData)
            }(profile)
        }
        wg.Wait()

        // 更新 M3U8 播放列表
        t.updateM3U8(stream, profiles, segmentIndex)

        // 通知 CDN 有新分片
        t.cdn.Purge(stream.StreamKey)

        segmentIndex++
    }

    // 生成最終 VOD M3U8（供回看使用）
    t.generateVODM3U8(stream, segmentIndex)
}

// M3U8 播放列表（保留最近 5 個分片，滑動窗口）
func (t *TranscoderService) updateM3U8(
    stream *StreamInfo, profiles []TranscodeProfile, latestSeg int,
) {
    for _, p := range profiles {
        startSeg := max(0, latestSeg-4)

        m3u8 := "#EXTM3U\n#EXT-X-VERSION:3\n"
        m3u8 += fmt.Sprintf("#EXT-X-TARGETDURATION:2\n")
        m3u8 += fmt.Sprintf("#EXT-X-MEDIA-SEQUENCE:%d\n\n", startSeg)

        for i := startSeg; i <= latestSeg; i++ {
            m3u8 += "#EXTINF:2.0,\n"
            m3u8 += fmt.Sprintf("seg%06d.ts\n", i)
        }

        path := fmt.Sprintf("live/%s/%s/playlist.m3u8",
            stream.StreamKey, p.Resolution)
        t.storage.Upload(path, []byte(m3u8))
    }

    // Master playlist（讓播放器自動選碼率）
    master := "#EXTM3U\n"
    for _, p := range profiles {
        master += fmt.Sprintf(
            "#EXT-X-STREAM-INF:BANDWIDTH=%d,RESOLUTION=%s\n%s/playlist.m3u8\n",
            p.Bitrate*1000, p.Resolution, p.Resolution,
        )
    }
    masterPath := fmt.Sprintf("live/%s/master.m3u8", stream.StreamKey)
    t.storage.Upload(masterPath, []byte(master))
}
```

### 3. 即時聊天室（彈幕）

```go
// WebSocket 廣播（一個直播間對應多個 WebSocket 連線）
type ChatHub struct {
    rooms   map[string]*ChatRoom  // streamKey → Room
    mu      sync.RWMutex
    redisPub *redis.Client  // 跨機器廣播
}

type ChatRoom struct {
    streamKey string
    clients   map[*websocket.Conn]int64  // conn → userID
    mu        sync.RWMutex
    broadcast chan ChatMessage
}

type ChatMessage struct {
    StreamKey string `json:"stream_key"`
    UserID    int64  `json:"user_id"`
    Username  string `json:"username"`
    Content   string `json:"content"`
    SentAt    int64  `json:"sent_at"`
}

func (h *ChatHub) Broadcast(msg ChatMessage) {
    // 本機廣播
    h.mu.RLock()
    room, ok := h.rooms[msg.StreamKey]
    h.mu.RUnlock()

    if ok {
        room.broadcast <- msg
    }

    // 跨節點廣播（透過 Redis Pub/Sub）
    data, _ := json.Marshal(msg)
    h.redisPub.Publish(context.Background(),
        fmt.Sprintf("chat:%s", msg.StreamKey), data)
}

func (r *ChatRoom) Run() {
    for msg := range r.broadcast {
        r.mu.RLock()
        for conn := range r.clients {
            conn.WriteJSON(msg)
        }
        r.mu.RUnlock()
    }
}
```

### 4. 延遲優化策略

| 方案 | 延遲 | 適用場景 |
|------|------|---------|
| **HLS (2s 分片)** | 5–15 秒 | 一般直播（Twitch 預設） |
| **LHLS (Low Latency HLS)** | 1–3 秒 | Apple 低延遲擴展 |
| **DASH-CMAF** | 1–3 秒 | 跨平台低延遲 |
| **WebRTC** | < 500ms | 1對1視訊、小型直播間 |

```go
// Low Latency HLS：分片縮短至 200ms
type LLHLSConfig struct {
    PartDuration    float64  // 0.2 秒（vs 普通 2 秒）
    TargetDuration  int      // 2 秒（包含 10 個 Part）
    EnablePreload   bool     // HTTP/2 Server Push 預載下一分片
}
```

---

## 擴展性設計

### CDN 層級分發

```
主播 → 邊緣接入節點（距離最近）
    ↓ 推送到源站
源站 → 全球 CDN PoP（邊緣緩存）
觀眾 → 就近 CDN PoP 拉取
    ↓ PoP 沒有時回源
    → 源站 → 轉碼結果
```

**關鍵參數**：
- CDN 節點數量：500+ PoP（Cloudflare / Akamai 規模）
- 分片緩存 TTL：2–4 秒（與分片時長一致）
- 源站備援：多地域主動主動

---

## Trade-offs

| 決策 | 選擇 | 理由 |
|------|------|------|
| 推流協議 | **RTMP 接入 → HLS 分發** | RTMP 成熟，HLS 相容性最好 |
| 轉碼架構 | **分散式轉碼叢集** | 10萬路同時轉碼需要 |
| 聊天廣播 | **Redis Pub/Sub + WebSocket** | 跨節點廣播，低延遲 |
| 儲存 | **Object Storage（S3）** | VOD 冷儲存，成本低 |
| 延遲目標 | **5秒（HLS）** | < 1秒需 WebRTC，複雜度倍增 |

---

## 相關頁面

- [[sd-video-streaming]] — 點播視頻（HLS、分塊上傳）
- [[sd-chat-system]] — WebSocket 聊天系統
- [[系統設計核心技術棧]] — Redis Pub/Sub、CDN
