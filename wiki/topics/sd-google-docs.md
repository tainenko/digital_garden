---
title: "SD題解：協作文件（Google Docs）"
type: topic
tags: [system-design, google-docs, OT, CRDT, collaboration, golang, hard]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：協作文件（Google Docs）

> **難度**: 進階（最難之一）｜ **頻率**: 中等 ｜ **代表**: Google Docs, Notion, Figma

---

## RESHADED 快速分析

**R - Requirements**
- 多人同時編輯同一份文件，即時看到彼此的變更
- 離線編輯後重新連線，合併衝突
- 版本歷史、游標位置顯示

**E - Estimation**
- 同時編輯者：平均 5 人/文件
- 每次按鍵 = 1 個操作 → 5人 × 每秒 2 次按鍵 = 10 ops/秒/文件
- 熱門文件（如共同筆記）：可能 100 人同時編輯

---

## 核心挑戰：同時編輯的衝突

```
初始文件：  "Hello World"
User A 刪除第6個字母 'W'：   "Hello orld"  (delete at pos 6)
User B 在第5個位置插入 '!':  "Hello! World" (insert '!' at pos 5)
```

如果直接套用兩個操作：
- 先 A 再 B：刪 W 後，在 pos 5 插入 '!' → "Hello!orld"
- 先 B 再 A：插入後，刪 pos 6 的 'W' → "Hello! orld"
- 兩者結果不同！

---

## 兩種解法

| | OT（Operational Transformation）| CRDT（Conflict-free Replicated Data Type）|
|--|--|--|
| 原理 | 傳輸操作前先「轉換」以補償之前的操作 | 特殊資料結構，合併天然無衝突 |
| 實作難度 | 高（Google Docs 使用）| 中（Figma 使用）|
| 網路要求 | 需要中央 Server 排序 | 可去中心化（P2P）|
| 成熟度 | 工業界驗證 20+ 年 | 較新，Yjs 等庫逐漸成熟 |

---

## Go 實現

### 1. 基礎操作類型

```go
package collab

type OpType string

const (
    OpInsert OpType = "insert"
    OpDelete OpType = "delete"
    OpRetain OpType = "retain" // 保持不變（用於指定位置）
)

type Operation struct {
    Type     OpType
    Position int    // 操作發生的位置
    Content  string // 插入的內容（Delete 時為空）
    Length   int    // 刪除的長度（Insert 時為 0）

    // OT 需要的元數據
    ClientID  string
    Revision  int    // 基於哪個版本產生的操作
    Timestamp int64
}
```

### 2. Operational Transformation 核心

```go
// Transform 將操作 op 轉換，使其在 other 操作已執行後仍然正確
// 回傳轉換後的 op'，可以在 other 之後安全執行
func Transform(op, other *Operation) *Operation {
    transformed := *op // 複製

    switch other.Type {
    case OpInsert:
        // other 在 op 之前插入 → op 的位置需要往後移
        if other.Position <= op.Position {
            transformed.Position += len(other.Content)
        }
        // other 在 op 之後插入 → op 的位置不變

    case OpDelete:
        if op.Type == OpInsert {
            if other.Position < op.Position {
                // other 在 op 之前刪除 → op 位置往前移
                deleteEnd := other.Position + other.Length
                if deleteEnd <= op.Position {
                    transformed.Position -= other.Length
                } else {
                    transformed.Position = other.Position
                }
            }
        } else if op.Type == OpDelete {
            // 兩個 Delete 重疊的情況
            otherEnd := other.Position + other.Length
            opEnd := op.Position + op.Length

            if other.Position >= opEnd || otherEnd <= op.Position {
                // 不重疊
                if other.Position < op.Position {
                    transformed.Position -= other.Length
                }
            } else {
                // 重疊：調整刪除範圍
                overlapStart := max(other.Position, op.Position)
                overlapEnd := min(otherEnd, opEnd)
                overlap := overlapEnd - overlapStart
                transformed.Length -= overlap
                if other.Position <= op.Position {
                    transformed.Position = other.Position
                }
            }
        }
    }

    return &transformed
}
```

### 3. Server 端操作排序

Server 是中央協調者，確保所有客戶端看到相同的操作順序：

```go
type DocumentServer struct {
    docs    map[string]*Document
    clients map[string]*Client // clientID → WebSocket conn
    mu      sync.RWMutex
}

type Document struct {
    ID        string
    Content   []rune
    History   []*Operation // 操作歷史
    Revision  int
    mu        sync.Mutex
}

// ApplyOperation Server 接收並廣播操作
func (s *DocumentServer) ApplyOperation(docID, clientID string, op *Operation) error {
    s.mu.Lock()
    doc := s.docs[docID]
    s.mu.Unlock()

    doc.mu.Lock()
    defer doc.mu.Unlock()

    // 1. 轉換操作（補償客戶端不知道的操作）
    // 客戶端基於 revision=N 產生操作，但 Server 可能已在 N 之後執行了其他操作
    transformedOp := op
    for _, historyOp := range doc.History[op.Revision:] {
        transformedOp = Transform(transformedOp, historyOp)
    }

    // 2. 執行操作
    if err := doc.apply(transformedOp); err != nil {
        return err
    }

    // 3. 更新版本號
    transformedOp.Revision = doc.Revision
    doc.History = append(doc.History, transformedOp)
    doc.Revision++

    // 4. 廣播給所有其他客戶端
    go s.broadcast(docID, clientID, transformedOp)

    return nil
}

func (doc *Document) apply(op *Operation) error {
    switch op.Type {
    case OpInsert:
        if op.Position > len(doc.Content) {
            return fmt.Errorf("position out of bounds")
        }
        runes := []rune(op.Content)
        doc.Content = append(
            doc.Content[:op.Position],
            append(runes, doc.Content[op.Position:]...)...,
        )
    case OpDelete:
        end := op.Position + op.Length
        if end > len(doc.Content) {
            end = len(doc.Content)
        }
        doc.Content = append(doc.Content[:op.Position], doc.Content[end:]...)
    }
    return nil
}
```

### 4. 客戶端（含離線緩衝）

```go
type DocumentClient struct {
    docID       string
    conn        *websocket.Conn
    localBuffer []*Operation // 離線時緩衝的操作
    revision    int
    localContent []rune
    mu           sync.Mutex
}

// LocalEdit 用戶在本地輸入
func (c *DocumentClient) LocalEdit(op *Operation) {
    c.mu.Lock()
    defer c.mu.Unlock()

    op.Revision = c.revision
    op.ClientID = "client-1"

    // 立即套用到本地（無需等 Server 確認，樂觀更新）
    c.applyLocal(op)

    // 加入待發送緩衝
    c.localBuffer = append(c.localBuffer, op)

    // 發送給 Server
    go c.sendToServer(op)
}

// ReceiveFromServer 收到 Server 廣播的其他人操作
func (c *DocumentClient) ReceiveFromServer(serverOp *Operation) {
    c.mu.Lock()
    defer c.mu.Unlock()

    // 需要轉換 localBuffer 中的待確認操作
    transformedServerOp := serverOp
    var newBuffer []*Operation
    for _, localOp := range c.localBuffer {
        transformedLocalOp := Transform(localOp, transformedServerOp)
        transformedServerOp = Transform(transformedServerOp, localOp)
        newBuffer = append(newBuffer, transformedLocalOp)
    }
    c.localBuffer = newBuffer

    // 更新本地文件
    c.applyLocal(transformedServerOp)
    c.revision = serverOp.Revision + 1
}
```

### 5. 游標位置同步

```go
type CursorPosition struct {
    ClientID string `json:"client_id"`
    UserName string `json:"user_name"`
    Color    string `json:"color"`
    Position int    `json:"position"`
}

// 游標位置也需要 OT 轉換（位置會因為插入/刪除而偏移）
func transformCursor(cursor *CursorPosition, op *Operation) *CursorPosition {
    transformed := *cursor
    if op.Type == OpInsert && op.Position <= cursor.Position {
        transformed.Position += len(op.Content)
    } else if op.Type == OpDelete && op.Position < cursor.Position {
        deleteEnd := op.Position + op.Length
        if deleteEnd <= cursor.Position {
            transformed.Position -= op.Length
        } else {
            transformed.Position = op.Position
        }
    }
    return &transformed
}
```

---

## 架構圖

```
User A              User B              Server
  │ 輸入 'H'          │                   │
  │──op(insert H)────→│                   │
  │                   │──────────────────→│
  │                   │ 輸入 'e'           │ 轉換 + 儲存
  │←── ack(rev=1) ────│←── op(insert e) ──│
  │                   │                   │ 廣播給所有人
  │←─ op(insert e, transformed) ──────────│
```

---

## Trade-offs

| 決策 | 選擇 | 理由 |
|------|------|------|
| 衝突解決 | OT | 工業界成熟方案；CRDT 更適合 P2P |
| 通訊 | WebSocket | 雙向即時，操作需要即時廣播 |
| 本地更新 | 樂觀更新 | 不等 Server 確認，打字不卡頓 |
| 儲存格式 | 操作歷史（Event Sourcing）| 支援版本回放，任意時間點還原 |

---

## 相關題解

- [[sd-chat-system|SD題解：即時聊天]] — WebSocket 連線管理
- [[sd-dropbox|SD題解：雲端儲存]] — 版本控制概念
