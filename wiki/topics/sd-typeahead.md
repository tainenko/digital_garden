---
title: "SD題解：搜尋自動補全（Typeahead / Autocomplete）"
type: topic
tags: [system-design, typeahead, trie, autocomplete, golang, medium]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：搜尋自動補全（Typeahead / Autocomplete）

> **難度**: 中級 ｜ **頻率**: 高 ｜ **代表**: Google 搜尋框、Twitter 搜尋

---

## RESHADED 快速分析

**R - Requirements**
- 用戶輸入前綴，回傳最多 10 個建議（按熱度排序）
- P99 延遲 < 100ms
- 建議要反映最新趨勢（不能太舊）

**E - Estimation**
- DAU：5億
- 每次搜尋平均輸入 5 個字元 → 5次 API 請求
- QPS：5億 × 5次 / 86400 ≈ **29,000 QPS**（讀重）
- 資料量：Google 每天 100億次搜尋 → 假設 5億種不重複關鍵字

---

## 核心資料結構：Trie（前綴樹）

```
輸入: "be"
Trie:
    root
     └─ b
         └─ e  ← 在這個節點找最熱門的建議
             ├─ a → bear（熱度:100）
             ├─ e → beef（熱度: 85）
             └─ s → best（熱度: 200）  ← 最熱門

建議結果: ["best", "bear", "beef"]
```

---

## Go 實現

### 基本 Trie（含熱度排序）

```go
package typeahead

import (
    "sort"
    "strings"
)

type TrieNode struct {
    children map[rune]*TrieNode
    isEnd    bool
    word     string // 完整詞彙
    freq     int64  // 搜尋頻率
}

type Trie struct {
    root *TrieNode
}

func NewTrie() *Trie {
    return &Trie{root: &TrieNode{children: make(map[rune]*TrieNode)}}
}

// Insert 插入詞彙及其頻率
func (t *Trie) Insert(word string, freq int64) {
    node := t.root
    for _, ch := range strings.ToLower(word) {
        if _, ok := node.children[ch]; !ok {
            node.children[ch] = &TrieNode{children: make(map[rune]*TrieNode)}
        }
        node = node.children[ch]
    }
    node.isEnd = true
    node.word = word
    node.freq = freq
}

// Search 找出前綴對應的 topK 建議
func (t *Trie) Search(prefix string, topK int) []string {
    node := t.root
    for _, ch := range strings.ToLower(prefix) {
        if _, ok := node.children[ch]; !ok {
            return nil // 前綴不存在
        }
        node = node.children[ch]
    }

    // DFS 收集所有以此前綴開頭的詞
    var results []suggestion
    t.dfs(node, &results)

    // 按頻率排序，取 topK
    sort.Slice(results, func(i, j int) bool {
        return results[i].freq > results[j].freq
    })
    if len(results) > topK {
        results = results[:topK]
    }

    words := make([]string, len(results))
    for i, r := range results {
        words[i] = r.word
    }
    return words
}

type suggestion struct {
    word string
    freq int64
}

func (t *Trie) dfs(node *TrieNode, results *[]suggestion) {
    if node.isEnd {
        *results = append(*results, suggestion{node.word, node.freq})
    }
    for _, child := range node.children {
        t.dfs(child, results)
    }
}
```

### 優化：在每個節點快取 TopK

DFS 每次都要遍歷很慢。更快的方式：每個節點預存 TopK 建議：

```go
type OptimizedTrieNode struct {
    children map[rune]*OptimizedTrieNode
    topK     []suggestion // 預計算的 TopK 結果
}

type OptimizedTrie struct {
    root *OptimizedTrieNode
    k    int
}

// Search O(L) where L = prefix length，不需要 DFS！
func (t *OptimizedTrie) Search(prefix string) []string {
    node := t.root
    for _, ch := range strings.ToLower(prefix) {
        if _, ok := node.children[ch]; !ok {
            return nil
        }
        node = node.children[ch]
    }
    result := make([]string, len(node.topK))
    for i, s := range node.topK {
        result[i] = s.word
    }
    return result
}

// updateTopK 插入詞彙後更新路徑上所有節點的 TopK
func (t *OptimizedTrie) Insert(word string, freq int64) {
    node := t.root
    path := []*OptimizedTrieNode{node}

    for _, ch := range strings.ToLower(word) {
        if _, ok := node.children[ch]; !ok {
            node.children[ch] = &OptimizedTrieNode{
                children: make(map[rune]*OptimizedTrieNode),
            }
        }
        node = node.children[ch]
        path = append(path, node)
    }
    node.topK = upsertTopK(node.topK, suggestion{word, freq}, t.k)

    // 沿路徑向上更新所有祖先節點
    for i := len(path) - 2; i >= 0; i-- {
        path[i].topK = upsertTopK(path[i].topK, suggestion{word, freq}, t.k)
    }
}

func upsertTopK(topK []suggestion, s suggestion, k int) []suggestion {
    for i, existing := range topK {
        if existing.word == s.word {
            topK[i].freq = s.freq
            sort.Slice(topK, func(i, j int) bool { return topK[i].freq > topK[j].freq })
            return topK
        }
    }
    topK = append(topK, s)
    sort.Slice(topK, func(i, j int) bool { return topK[i].freq > topK[j].freq })
    if len(topK) > k {
        return topK[:k]
    }
    return topK
}
```

### API 服務層

```go
type TypeaheadService struct {
    trie  *OptimizedTrie
    cache *redis.Client
}

// GetSuggestions 帶 Redis 快取的查詢
func (s *TypeaheadService) GetSuggestions(ctx context.Context, prefix string) ([]string, error) {
    prefix = strings.ToLower(strings.TrimSpace(prefix))
    if len(prefix) == 0 {
        return nil, nil
    }

    // 1. 查 Cache
    cacheKey := "suggest:" + prefix
    if cached, err := s.cache.LRange(ctx, cacheKey, 0, 9).Result(); err == nil && len(cached) > 0 {
        return cached, nil
    }

    // 2. 查 Trie
    suggestions := s.trie.Search(prefix, 10)

    // 3. 回填 Cache（TTL 5 分鐘）
    if len(suggestions) > 0 {
        pipe := s.cache.Pipeline()
        pipe.Del(ctx, cacheKey)
        for _, s := range suggestions {
            pipe.RPush(ctx, cacheKey, s)
        }
        pipe.Expire(ctx, cacheKey, 5*time.Minute)
        pipe.Exec(ctx)
    }

    return suggestions, nil
}

// UpdateFrequency 用戶實際搜尋後更新詞頻（非同步批次更新）
func (s *TypeaheadService) UpdateFrequency(term string) {
    // 不立即更新 Trie（成本高），而是放入 Redis 計數器
    // 定期（如每小時）批次重建 Trie
    s.cache.Incr(context.Background(), "freq:"+term)
}
```

---

## 系統架構

```
Client（每輸入一個字元觸發請求，加 debounce 300ms）
    ↓
API Server → Redis Cache → Trie Service（記憶體）
                               ↑
                         定期從 DB 重建（每小時）
                               ↓
                           頻率 DB（Cassandra/BigQuery）
                           ← 用戶搜尋日誌 streaming
```

**Trie 重建策略**：
- 每小時從 DB 拉最新詞頻數據
- 在後台建新 Trie
- Atomic 替換（指針切換，無需停服）

---

## Trade-offs

| 決策 | 選擇 | 理由 |
|------|------|------|
| 資料結構 | Trie + 預存 TopK | O(L) 查詢速度，比每次 DFS 快 |
| 快取 | Redis（前綴 key）| 熱門前綴不用每次查 Trie |
| 更新策略 | 批次更新（非即時）| 詞頻不需實時，避免頻繁重建 Trie |
| Client 優化 | Debounce 300ms | 減少請求數，打字快時不需每個字都查 |

---

## 相關題解

- [[sd-distributed-cache|SD題解：分散式快取]] — Redis 快取策略
- [[sd-url-shortener|SD題解：URL Shortener]] — 類似的高讀 QPS 設計
