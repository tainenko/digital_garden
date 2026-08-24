---
title: Max Fee（Maximize Block Fee with Transaction Selection）
type: concept
tags: [CodeSignal, Coinbase, OA, Golang, Greedy, TreeDP, DAG]
created: 2026-05-04
updated: 2026-05-04
sources: []
---

# Max Fee（Maximize Block Fee with Transaction Selection）

## 題目說明

**難度**：Hard｜**類型**：Coding OA / Onsite

每個 Block 有最大容量 **100**。給定 N 筆交易，每筆交易有 `id`、`size`、`fee`，選出一組子集使得：

- `total_size ≤ 100`
- `total_fee` 最大

---

## Level 1 — 無依賴，Greedy 選擇

**規則**：
- 禁止使用 heap / priority queue（面試官明確要求 production-friendly 解法）
- 允許 greedy（雖然 greedy 在嚴格 0/1 knapsack 下不保證最優，但面試接受）

**解法**：按 `fee/size` 密度排序，依序貪心填入直到超過容量。

```go
type Transaction struct {
    ID   string
    Size int
    Fee  int
}

func selectTransactions(txns []Transaction) []Transaction {
    sort.Slice(txns, func(i, j int) bool {
        // 按 fee density 降序
        di := float64(txns[i].Fee) / float64(txns[i].Size)
        dj := float64(txns[j].Fee) / float64(txns[j].Size)
        return di > dj
    })

    var selected []Transaction
    capacity := 100
    for _, t := range txns {
        if t.Size <= capacity {
            selected = append(selected, t)
            capacity -= t.Size
        }
    }
    return selected
}
```

---

## Level 2 — 加入 parentId 依賴鏈

**新增規則**：
- 交易可有可選的 `parentId` 欄位
- **Child 只有在所有 ancestor 都被選中時才能入選**
- 依賴結構是 forest（多棵樹，無環）

**解法思路**：

1. **建樹**：將 transactions 組織成 parent → children 的樹狀結構
2. **DFS + Greedy**：從 root 開始，若 parent 被選，才考慮 children
3. **必選 parent**：若選了 child，其 parent chain 必須全部進入

```go
type TxNode struct {
    ID       string
    Size     int
    Fee      int
    Children []*TxNode
}

func selectWithDependency(txns []Transaction) []Transaction {
    // 1. 建立 id → node 映射
    nodes := make(map[string]*TxNode)
    for i := range txns {
        nodes[txns[i].ID] = &TxNode{
            ID: txns[i].ID, Size: txns[i].Size, Fee: txns[i].Fee,
        }
    }

    // 2. 建樹，找 roots
    var roots []*TxNode
    for _, t := range txns {
        if t.ParentID == "" {
            roots = append(roots, nodes[t.ID])
        } else if parent, ok := nodes[t.ParentID]; ok {
            parent.Children = append(parent.Children, nodes[t.ID])
        }
    }

    // 3. DFS greedy：選 root 後才考慮 children
    var selected []Transaction
    capacity := 100

    var dfs func(node *TxNode) bool
    dfs = func(node *TxNode) bool {
        if node.Size > capacity {
            return false
        }
        capacity -= node.Size
        selected = append(selected, Transaction{ID: node.ID, Size: node.Size, Fee: node.Fee})
        for _, child := range node.Children {
            dfs(child) // 子節點依照 fee density 排序再遞迴
        }
        return true
    }

    // 按 root 的 fee density 排序
    sort.Slice(roots, func(i, j int) bool {
        return float64(roots[i].Fee)/float64(roots[i].Size) >
               float64(roots[j].Fee)/float64(roots[j].Size)
    })
    for _, root := range roots {
        dfs(root)
    }
    return selected
}
```

---

## 複雜度

| Level | 時間 | 空間 |
|-------|------|------|
| Level 1 | O(N log N) | O(N) |
| Level 2 | O(N log N) | O(N) |

---

## 相關頁面

- [[Coinbase_OA題目總表]] — 所有 Coinbase OA 題目索引
- [[Coinbase_HA_總覽]] — OA 格式說明
