---
title: "SD題解：雲端儲存（Dropbox / Google Drive）"
type: topic
tags: [system-design, dropbox, cloud-storage, sync, chunking, golang, hard]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：雲端儲存（Dropbox / Google Drive）

> **難度**: 進階 ｜ **頻率**: 中等 ｜ **代表**: Dropbox, Google Drive, iCloud

---

## RESHADED 快速分析

**R - Requirements**
- 上傳/下載檔案、跨裝置同步、版本歷史、分享連結
- 非功能：一致性（同步後所有裝置看到相同檔案）、大檔案支援（100GB+）

**E - Estimation**
- DAU：5億
- 每用戶平均儲存：10GB → 總儲存 5000 PB = **5 EB**
- 每天上傳 QPS：5億 × 1次 / 86400 ≈ 5800/s
- 讀寫比：約 1:1（每次上傳通常對應多台裝置下載）

---

## 核心挑戰

1. **大檔案上傳**：不能一次傳，需要分塊（Chunking）
2. **去重（Deduplication）**：相同檔案只存一份（Content-addressable Storage）
3. **增量同步（Delta Sync）**：只傳修改的部分，不是整個檔案
4. **衝突處理**：兩台裝置同時修改同一個檔案

---

## Go 實現

### 1. 分塊與 Hash（Content-Addressable Storage）

```go
package storage

import (
    "crypto/sha256"
    "fmt"
    "io"
    "os"
)

const ChunkSize = 4 * 1024 * 1024 // 4MB per chunk

type Chunk struct {
    Index    int
    Hash     string // SHA-256 of chunk content
    Size     int64
    Offset   int64
}

type FileMetadata struct {
    FileID   string
    Name     string
    Size     int64
    Hash     string   // SHA-256 of entire file
    Chunks   []*Chunk
    Version  int
    UpdatedAt time.Time
}

// ChunkFile 將檔案分塊並計算每塊 Hash
func ChunkFile(path string) (*FileMetadata, [][]byte, error) {
    f, err := os.Open(path)
    if err != nil {
        return nil, nil, err
    }
    defer f.Close()

    fileHasher := sha256.New()
    var chunks []*Chunk
    var chunkDatas [][]byte
    var offset int64
    buf := make([]byte, ChunkSize)

    for i := 0; ; i++ {
        n, err := f.Read(buf)
        if n == 0 {
            break
        }
        data := make([]byte, n)
        copy(data, buf[:n])

        // 計算這一塊的 hash
        chunkHash := sha256.Sum256(data)
        hashStr := fmt.Sprintf("%x", chunkHash)

        chunks = append(chunks, &Chunk{
            Index:  i,
            Hash:   hashStr,
            Size:   int64(n),
            Offset: offset,
        })
        chunkDatas = append(chunkDatas, data)
        fileHasher.Write(data)
        offset += int64(n)

        if err == io.EOF {
            break
        }
    }

    fileHash := fmt.Sprintf("%x", fileHasher.Sum(nil))
    return &FileMetadata{
        Hash:   fileHash,
        Chunks: chunks,
        Size:   offset,
    }, chunkDatas, nil
}
```

### 2. 去重上傳（只傳新的 Chunk）

```go
type SyncService struct {
    chunkDB  ChunkRepository  // chunk_hash → S3 key
    metaDB   MetadataDB
    storage  ObjectStorage    // S3
}

// UploadFile 智慧上傳（只上傳 Server 沒有的 Chunk）
func (s *SyncService) UploadFile(ctx context.Context, userID string, meta *FileMetadata, chunks [][]byte) error {
    // 1. 詢問 Server 哪些 Chunk 已存在（去重）
    existingChunks, err := s.chunkDB.CheckExistence(ctx, extractHashes(meta.Chunks))
    if err != nil {
        return err
    }

    // 2. 只上傳不存在的 Chunk
    for i, chunk := range meta.Chunks {
        if existingChunks[chunk.Hash] {
            continue // Server 已有此 Chunk，跳過（去重！）
        }
        s3Key := "chunks/" + chunk.Hash
        if err := s.storage.Put(ctx, s3Key, chunks[i]); err != nil {
            return fmt.Errorf("failed to upload chunk %d: %w", i, err)
        }
        // 記錄 chunk_hash → s3_key 映射
        s.chunkDB.Register(ctx, chunk.Hash, s3Key)
    }

    // 3. 儲存檔案元數據（Chunk 清單）
    return s.metaDB.SaveVersion(ctx, userID, meta)
}

// 去重效果：如果 10 個用戶上傳同一份 PDF，只存 1 份
// 如果修改檔案只改了前 10%，只傳前面幾個 Chunk
```

### 3. 增量同步（Delta Sync）

```go
// GetChangedChunks 比較新舊版本，找出需要上傳的 Chunk
func GetChangedChunks(oldMeta, newMeta *FileMetadata) []int {
    oldHashes := make(map[int]string)
    for _, c := range oldMeta.Chunks {
        oldHashes[c.Index] = c.Hash
    }

    var changedIndices []int
    for _, c := range newMeta.Chunks {
        if oldHashes[c.Index] != c.Hash {
            changedIndices = append(changedIndices, c.Index)
        }
    }
    return changedIndices
}
```

### 4. 版本控制

```go
type FileVersion struct {
    FileID    string
    VersionID string
    UserID    string
    Chunks    []*Chunk
    CreatedAt time.Time
    Message   string // 可選的版本說明
}

type VersionService struct {
    db      VersionDB
    storage ObjectStorage
}

// ListVersions 列出檔案的版本歷史
func (v *VersionService) ListVersions(ctx context.Context, userID, fileID string) ([]*FileVersion, error) {
    return v.db.GetVersions(ctx, fileID)
}

// Restore 還原到特定版本
func (v *VersionService) Restore(ctx context.Context, userID, fileID, versionID string) error {
    version, err := v.db.GetVersion(ctx, fileID, versionID)
    if err != nil {
        return err
    }
    // 以該版本的 Chunk 清單建立新版本（Chunk 本身不需要複製，已在 S3）
    newMeta := &FileMetadata{
        FileID:    fileID,
        Chunks:    version.Chunks,
        UpdatedAt: time.Now(),
    }
    return v.db.SaveVersion(ctx, userID, newMeta)
}
```

### 5. 跨裝置同步（Long Polling）

```go
type SyncNotifier struct {
    cache *redis.Client
}

// WatchChanges 裝置監聽檔案變更（Long Polling）
func (s *SyncNotifier) WatchChanges(ctx context.Context, userID string, since time.Time) ([]*ChangeEvent, error) {
    timeout := time.After(30 * time.Second)
    ticker := time.NewTicker(1 * time.Second)
    defer ticker.Stop()

    for {
        select {
        case <-ctx.Done():
            return nil, ctx.Err()
        case <-timeout:
            return nil, nil // 30 秒沒變更，回傳空（客戶端重新 polling）
        case <-ticker.C:
            changes, err := s.getChangesSince(ctx, userID, since)
            if err != nil {
                return nil, err
            }
            if len(changes) > 0 {
                return changes, nil // 有變更，立刻回傳
            }
        }
    }
}

// NotifyChange 有檔案更新時通知（上傳完成後呼叫）
func (s *SyncNotifier) NotifyChange(ctx context.Context, userID string, event *ChangeEvent) {
    key := fmt.Sprintf("changes:%s", userID)
    data, _ := json.Marshal(event)
    s.cache.LPush(ctx, key, data)
    s.cache.Expire(ctx, key, 7*24*time.Hour)
}
```

### 6. 衝突處理

```go
// ConflictResolution 衝突策略
type ConflictStrategy int

const (
    LastWriteWins ConflictStrategy = iota // 最後寫入勝出（Dropbox 預設）
    CreateConflictCopy                     // 建立衝突副本（"file (conflicted copy).docx"）
    MergeContent                           // 嘗試合併（文字檔可用）
)

func handleConflict(existing, incoming *FileMetadata, strategy ConflictStrategy) *FileMetadata {
    switch strategy {
    case LastWriteWins:
        if incoming.UpdatedAt.After(existing.UpdatedAt) {
            return incoming
        }
        return existing
    case CreateConflictCopy:
        // 兩個版本都保留，重新命名其中一個
        incoming.Name = addConflictSuffix(incoming.Name, incoming.UpdatedAt)
        return incoming // 返回後由上層決定如何儲存兩個版本
    }
    return incoming
}
```

---

## 架構圖

```
用戶裝置（本地 Sync Agent）
    ↓ 偵測檔案變更
    ↓ 分塊 + 計算 Hash
API Server
    ↓ 確認哪些 Chunk 需要上傳（去重）
    ↓
S3（Chunk Storage，按 hash 命名，去重）
    ↓ 上傳完成
Metadata DB（FileID → Chunk 清單 + 版本）
    ↓
Notification Service
    ↓ Long Polling
其他裝置（下載更新的 Chunk，重組檔案）
```

---

## Trade-offs

| 決策 | 選擇 | 理由 |
|------|------|------|
| 分塊大小 | 4MB | 太小 → overhead 多；太大 → 斷線重傳代價高 |
| 去重策略 | Content-addressable（hash 為 key）| 相同內容只存一份，節省大量空間 |
| 同步通知 | Long Polling | 比 Polling 省；比 WebSocket 簡單，夠用 |
| 衝突處理 | 建立衝突副本 | 不丟任何資料，用戶自行決定保留哪個 |

---

## 相關題解

- [[sd-video-streaming|SD題解：影片串流]] — 相似的分塊上傳架構
- [[sd-consistent-hashing|SD題解：一致性雜湊]] — S3 的分散式儲存機制
