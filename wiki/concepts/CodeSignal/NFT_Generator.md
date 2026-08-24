---
title: NFT Generator（Generate NFT Metadata and Ensure Uniqueness）
type: concept
tags: [CodeSignal, Coinbase, OA, Golang, Math, CartesianProduct, MixedRadix]
created: 2026-05-04
updated: 2026-05-04
sources: []
---

# NFT Generator（Generate NFT Metadata and Ensure Uniqueness）

## 題目說明

**難度**：Hard｜**類型**：Coding OA / Onsite

實作一個 NFT metadata generator。每個 NFT 從多個 trait category 中各取一個 value，生成 n 個**唯一且確定性**的 NFT。

---

## Input / Output

**Input**（單一 JSON 物件）：
```json
{
  "collectionName": "CryptoPunks",
  "traits": [
    {"name": "Background", "values": ["Blue", "Green", "Red"]},
    {"name": "Eyes",       "values": ["Small", "Big"]}
  ],
  "n": 4,
  "seed": 42
}
```

**Output**（JSON array）：
```json
[
  {"name": "CryptoPunks #1", "attributes": [{"trait_type": "Background", "value": "Blue"},  {"trait_type": "Eyes", "value": "Small"}]},
  {"name": "CryptoPunks #2", "attributes": [{"trait_type": "Background", "value": "Green"}, {"trait_type": "Eyes", "value": "Big"}]},
  ...
]
```

若 n > 可能的唯一組合數，回傳 `"ERROR"`。

---

## 規則

1. **唯一性**：兩個 NFT 如果每個 trait category 的 value 都相同，視為重複，不可出現
2. **確定性**：相同 input → 相同 output（使用 seed）
3. **不重試**：不能用「生成 → 檢查是否重複 → 重試」的方式，必須直接生成唯一組合
4. **重複 value 不增加組合數**：同一 category 內若有兩個 "Gold"，視為一個
5. **命名**：`"<collectionName> #<index>"` 從 1 開始

---

## 解題思路

### 核心：Mixed-Radix Decoding（混合進位解碼）

把所有可能組合映射為整數 `0 ~ total_combinations - 1`。

```
total_combinations = product(unique_values_count per category)
```

整數 k → NFT：對每個 category，`k % unique_count` 決定選哪個 value，然後 `k /= unique_count`。

### 確定性唯一生成（不需 retry）

使用**等差數列模 total_combinations**，選一個與 total_combinations 互質的步長 step：

```
indices = [(seed + i * step) % total_combinations  for i in range(n)]
```

若 step 與 total_combinations 互質（coprime），等差數列在 total 個元素內會遍歷所有可能，保證唯一。

---

## Go 完整解法

```go
package main

import (
    "fmt"
    "math/big"
)

type Trait struct {
    Name   string
    Values []string
}

type NFTAttribute struct {
    TraitType string `json:"trait_type"`
    Value     string `json:"value"`
}

type NFT struct {
    Name       string         `json:"name"`
    Attributes []NFTAttribute `json:"attributes"`
}

func generateNFTs(collectionName string, traits []Trait, n int, seed int64) ([]NFT, error) {
    // 1. 對每個 category 去重，建立 unique values 清單
    uniqueVals := make([][]string, len(traits))
    for i, t := range traits {
        seen := map[string]bool{}
        for _, v := range t.Values {
            if !seen[v] {
                seen[v] = true
                uniqueVals[i] = append(uniqueVals[i], v)
            }
        }
    }

    // 2. 計算 total_combinations（用 big.Int 防 overflow）
    total := big.NewInt(1)
    for _, uv := range uniqueVals {
        if len(uv) == 0 {
            return nil, fmt.Errorf("ERROR")
        }
        total.Mul(total, big.NewInt(int64(len(uv))))
    }

    if big.NewInt(int64(n)).Cmp(total) > 0 {
        return nil, fmt.Errorf("ERROR")
    }

    // 3. 找與 total 互質的 step（從 total/2 附近找）
    step := findCoprime(total)

    // 4. 生成 n 個唯一 index，decode 成 NFT
    nfts := make([]NFT, n)
    cur := new(big.Int).SetInt64(seed)
    cur.Mod(cur, total)

    for i := 0; i < n; i++ {
        nfts[i] = decodeIndex(new(big.Int).Set(cur), uniqueVals, collectionName, i+1)
        cur.Add(cur, step)
        cur.Mod(cur, total)
    }
    return nfts, nil
}

func decodeIndex(idx *big.Int, uniqueVals [][]string, name string, num int) NFT {
    attrs := make([]NFTAttribute, len(uniqueVals))
    for i := len(uniqueVals) - 1; i >= 0; i-- {
        size := big.NewInt(int64(len(uniqueVals[i])))
        mod := new(big.Int)
        idx.DivMod(idx, size, mod)
        attrs[i] = NFTAttribute{
            TraitType: fmt.Sprintf("Trait%d", i), // 替換為實際 trait name
            Value:     uniqueVals[i][mod.Int64()],
        }
    }
    return NFT{Name: fmt.Sprintf("%s #%d", name, num), Attributes: attrs}
}

func gcd(a, b *big.Int) *big.Int {
    a, b = new(big.Int).Set(a), new(big.Int).Set(b)
    for b.Sign() != 0 {
        a, b = b, new(big.Int).Mod(a, b)
    }
    return a
}

func findCoprime(total *big.Int) *big.Int {
    // 從 total/2+1 開始往上找第一個與 total 互質的數
    candidate := new(big.Int).Div(total, big.NewInt(2))
    candidate.Add(candidate, big.NewInt(1))
    one := big.NewInt(1)
    for {
        if gcd(candidate, total).Cmp(one) == 0 {
            return candidate
        }
        candidate.Add(candidate, one)
    }
}
```

---

## 邊界條件

| 情境 | 處理方式 |
|------|---------|
| n > total_combinations | 回傳 ERROR |
| category 內有重複 value | 去重後計算 total |
| traits 為空（0 個 category） | total = 1，只能生成 1 個 NFT（空 attributes）|
| total_combinations 極大 | 使用 big.Int |

---

## 複雜度

| 操作 | 時間 | 空間 |
|------|------|------|
| 初始化 | O(K × V)，K = category 數，V = max values 數 | O(K × V) |
| 每個 NFT 生成 | O(K) | O(K) |
| 總計 | O(n × K) | O(n × K) |

---

## 相關頁面

- [[Coinbase_OA題目總表]] — 所有 Coinbase OA 題目索引
