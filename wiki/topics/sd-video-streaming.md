---
title: "SD題解：影片串流（YouTube / Netflix）"
type: topic
tags: [system-design, video-streaming, cdn, transcoding, golang, medium]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：影片串流（YouTube / Netflix）

> **難度**: 中級 ｜ **頻率**: 高 ｜ **代表**: YouTube, Netflix, Bilibili

---

## RESHADED 快速分析

**R - Requirements**
- 上傳影片、轉碼為多種解析度（360p/720p/1080p/4K）
- 串流播放（自適應碼率）、暫停/快轉
- 非功能：高可用、低延遲啟動（<2s）、全球分發

**E - Estimation**（YouTube 規模）
- DAU：20億
- 每分鐘上傳：500 小時影片
- 每天觀看：10億小時
- 讀寫比：約 200:1（極度讀重）
- 儲存：500小時/分 × 1440分 × 10GB/小時 × 365天 ≈ **2.6 EB/年**

---

## 架構設計

```
上傳流程：
Client → Upload Service → Object Storage（S3 原始檔）
                              ↓
                         Transcoding Queue（Kafka）
                              ↓
                    Transcoding Workers（FFmpeg）
                              ↓
                    CDN Origins（各解析度）← 全球 CDN Edge

播放流程：
Client → CDN Edge（最近節點）→ 回源到 CDN Origin → S3
```

---

## Go 實現

### 1. 分塊上傳（Multipart Upload）

大型影片（GB 級）不能一次上傳，需分塊：

```go
package upload

import (
    "context"
    "crypto/md5"
    "fmt"
    "io"
    "sync"
)

type UploadSession struct {
    VideoID   string
    TotalSize int64
    ChunkSize int64
    Chunks    map[int]*ChunkInfo
    mu        sync.Mutex
}

type ChunkInfo struct {
    Index    int
    Size     int64
    Checksum string
    Uploaded bool
}

type UploadService struct {
    storage   ObjectStorage
    sessions  map[string]*UploadSession
    mu        sync.RWMutex
    jobQueue  MessageQueue // Kafka
}

// InitUpload 初始化分塊上傳
func (s *UploadService) InitUpload(ctx context.Context, userID, filename string, totalSize int64) (string, error) {
    videoID := generateVideoID()
    chunkSize := int64(5 * 1024 * 1024) // 5MB per chunk
    totalChunks := (totalSize + chunkSize - 1) / chunkSize

    session := &UploadSession{
        VideoID:   videoID,
        TotalSize: totalSize,
        ChunkSize: chunkSize,
        Chunks:    make(map[int]*ChunkInfo),
    }
    for i := 0; i < int(totalChunks); i++ {
        session.Chunks[i] = &ChunkInfo{Index: i}
    }

    s.mu.Lock()
    s.sessions[videoID] = session
    s.mu.Unlock()

    return videoID, nil
}

// UploadChunk 上傳單個分塊
func (s *UploadService) UploadChunk(ctx context.Context, videoID string, chunkIndex int, data io.Reader) error {
    s.mu.RLock()
    session, ok := s.sessions[videoID]
    s.mu.RUnlock()
    if !ok {
        return fmt.Errorf("upload session not found: %s", videoID)
    }

    // 計算 checksum 驗證完整性
    buf, _ := io.ReadAll(data)
    checksum := fmt.Sprintf("%x", md5.Sum(buf))

    // 上傳到 S3 的臨時路徑
    chunkKey := fmt.Sprintf("raw/%s/chunk_%04d", videoID, chunkIndex)
    if err := s.storage.Put(ctx, chunkKey, buf); err != nil {
        return err
    }

    session.mu.Lock()
    session.Chunks[chunkIndex].Checksum = checksum
    session.Chunks[chunkIndex].Uploaded = true
    allDone := s.allChunksUploaded(session)
    session.mu.Unlock()

    // 全部分塊上傳完成 → 觸發合併與轉碼
    if allDone {
        go s.triggerTranscoding(videoID, session)
    }
    return nil
}

func (s *UploadService) allChunksUploaded(session *UploadSession) bool {
    for _, chunk := range session.Chunks {
        if !chunk.Uploaded {
            return false
        }
    }
    return true
}

// triggerTranscoding 發送轉碼任務到 Kafka
func (s *UploadService) triggerTranscoding(videoID string, session *UploadSession) {
    // 先合併所有分塊
    s.storage.ComposeChunks(context.Background(), videoID, len(session.Chunks))

    // 發送轉碼任務
    s.jobQueue.Publish("transcode-jobs", TranscodeJob{
        VideoID:    videoID,
        SourcePath: fmt.Sprintf("raw/%s/original", videoID),
        Profiles:   []string{"360p", "720p", "1080p", "4K"},
    })
}
```

### 2. 轉碼 Worker

```go
package transcoding

import "fmt"

type TranscodeJob struct {
    VideoID    string
    SourcePath string
    Profiles   []string
}

type Profile struct {
    Name       string
    Resolution string
    Bitrate    string
    Codec      string
}

var profiles = map[string]Profile{
    "360p":  {"360p",  "640x360",   "800k",  "h264"},
    "720p":  {"720p",  "1280x720",  "2500k", "h264"},
    "1080p": {"1080p", "1920x1080", "5000k", "h264"},
    "4K":    {"4K",    "3840x2160", "15000k","h265"},
}

type TranscodeWorker struct {
    storage   ObjectStorage
    notifier  EventPublisher // 通知完成
}

// ProcessJob 執行轉碼（實際生產用 FFmpeg）
func (w *TranscodeWorker) ProcessJob(job TranscodeJob) error {
    results := make(chan error, len(job.Profiles))

    // 並行轉碼多個解析度
    for _, profileName := range job.Profiles {
        go func(p string) {
            profile := profiles[p]
            outputPath := fmt.Sprintf("videos/%s/%s/", job.VideoID, p)

            // 實際呼叫 FFmpeg（偽代碼）
            // cmd := exec.Command("ffmpeg",
            //     "-i", sourcePath,
            //     "-vf", fmt.Sprintf("scale=%s", profile.Resolution),
            //     "-b:v", profile.Bitrate,
            //     "-codec:v", profile.Codec,
            //     "-hls_time", "10",        // HLS 分段，每段 10 秒
            //     "-hls_playlist_type", "vod",
            //     outputPath+"playlist.m3u8")

            // 產生 HLS segments (.ts 檔) + playlist (.m3u8)
            err := w.generateHLS(job.VideoID, job.SourcePath, profile, outputPath)
            results <- err
        }(profileName)
    }

    // 等待所有解析度完成
    for range job.Profiles {
        if err := <-results; err != nil {
            return err
        }
    }

    // 發布「轉碼完成」事件
    w.notifier.Publish("video-ready", VideoReadyEvent{
        VideoID:  job.VideoID,
        Profiles: job.Profiles,
    })
    return nil
}

func (w *TranscodeWorker) generateHLS(videoID, source string, profile Profile, outputPath string) error {
    // 偽代碼：呼叫 FFmpeg 生成 HLS
    // 輸出：
    //   videos/{videoID}/{quality}/playlist.m3u8
    //   videos/{videoID}/{quality}/segment_000.ts
    //   videos/{videoID}/{quality}/segment_001.ts  ...
    return nil
}
```

### 3. 自適應碼率串流（ABR）

客戶端根據網路狀況動態切換解析度：

```go
// Master Playlist（HLS 格式）
// 客戶端先下載這個，根據頻寬選擇解析度
func generateMasterPlaylist(videoID string, profiles []string) string {
    var sb strings.Builder
    sb.WriteString("#EXTM3U\n")
    sb.WriteString("#EXT-X-VERSION:3\n\n")

    bandwidths := map[string]int{
        "360p": 800000, "720p": 2500000, "1080p": 5000000, "4K": 15000000,
    }
    resolutions := map[string]string{
        "360p": "640x360", "720p": "1280x720", "1080p": "1920x1080", "4K": "3840x2160",
    }

    for _, p := range profiles {
        sb.WriteString(fmt.Sprintf(
            "#EXT-X-STREAM-INF:BANDWIDTH=%d,RESOLUTION=%s\n",
            bandwidths[p], resolutions[p]))
        sb.WriteString(fmt.Sprintf(
            "https://cdn.example.com/videos/%s/%s/playlist.m3u8\n\n",
            videoID, p))
    }
    return sb.String()
}
```

### 4. 播放進度同步

```go
type PlaybackService struct {
    cache *redis.Client
    db    PlaybackDB
}

// SaveProgress 每 5 秒儲存播放進度
func (p *PlaybackService) SaveProgress(ctx context.Context, userID, videoID string, position float64) {
    key := fmt.Sprintf("progress:%s:%s", userID, videoID)
    p.cache.Set(ctx, key, position, 30*24*time.Hour)

    // 非同步同步到 DB（用於跨裝置同步）
    go p.db.UpsertProgress(userID, videoID, position)
}

// GetProgress 取得上次播放位置（跨裝置）
func (p *PlaybackService) GetProgress(ctx context.Context, userID, videoID string) float64 {
    key := fmt.Sprintf("progress:%s:%s", userID, videoID)
    val, err := p.cache.Get(ctx, key).Float64()
    if err == nil {
        return val
    }
    // Cache miss，查 DB
    progress, _ := p.db.GetProgress(userID, videoID)
    return progress
}
```

---

## 架構圖（完整）

```
[上傳]
用戶 → Upload API → S3（原始檔）
                      ↓ Kafka
               Transcoding Workers（FFmpeg × N）
                      ↓（並行：360p/720p/1080p/4K）
              S3（HLS segments + m3u8 playlist）
                      ↓
                CDN 預熱（主動推到邊緣節點）

[播放]
用戶 → CDN Edge → S3（HLS segments）
播放器自動根據頻寬切換 360p ↔ 720p ↔ 1080p
```

---

## Trade-offs

| 決策 | 選擇 | 理由 |
|------|------|------|
| 串流協定 | HLS | 相容性最好，支援自適應碼率 |
| 儲存 | S3 + CDN | 影片大、讀重，必須全球分發 |
| 轉碼 | 並行多 Worker | 轉碼 CPU 密集，需水平擴展 |
| 分塊上傳 | 5MB/chunk | 失敗可只重傳失敗的 chunk |

---

## 相關題解

- [[sd-distributed-cache|SD題解：分散式快取]] — CDN 本質是快取
- [[sd-social-media-feed|SD題解：社群媒體 Feed]] — CDN 分發媒體內容
