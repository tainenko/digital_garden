---
title: Feature Disentanglement
type: concept
tags: [ai, lora, training, tagging]
created: 2026-04-20
updated: 2026-04-20
sources: [lora-training-z-image-turbo]
---

# Feature Disentanglement（特徵解耦 / 標籤解耦）

在 LoRA 訓練中，透過分類標籤讓模型獨立學習不同維度的特徵，避免概念互相污染（concept bleeding）。

---

## 核心問題

若不做標籤解耦，模型可能把「特定服裝」和「角色外觀」綁定在一起——導致：
- 換裝時角色外觀改變
- Trigger word 觸發非預期的服裝或場景
- 不同 seed 下角色一致性差

---

## 解法：三層標籤分離

### 第一層：外觀標籤（Appearance）
固定角色的核心身份特徵，每張圖都必須標記：
```
zzArisuTachibana, blue hair bow, sidelocks
```

### 第二層：服裝標籤（Clothing）
描述當前圖片的服裝，**僅在訓練圖片中有該服裝時加入**：
```
blue dress, pleated skirt, short sleeves, white collar,
buttons, brown belt, white socks, brown loafers
```

### 第三層：情境標籤（Context）
動作、場景、環境等可替換的描述：
```
anime, smile, outdoor, school background
```

---

## 效果

| 測試場景 | 未解耦 | 解耦後 |
|----------|--------|--------|
| 換裝測試 | 角色外觀可能改變 | 只有服裝改變，臉部/髮型穩定 |
| 不同 seed | 一致性差 | 一致性高 |
| 背景替換 | 可能帶出固定背景 | 背景可自由控制 |

---

## 實踐要點

1. **外觀標籤需精確**：選擇最能代表該角色獨特外觀的 tag（如特殊髮飾、髮色組合）
2. **服裝標籤需完整**：漏標的服裝細節容易被模型「吸附」到 trigger word
3. **情境標籤避免重複**：避免所有圖片都有相同場景描述，否則場景也會被記憶

---

## 相關頁面

- [[LoRA訓練流程]] — 完整訓練流程（含標籤策略步驟）
- [[訓練資料策略]] — 資料收集與標記
- [[Ostris AI Toolkit]] — 實現此策略的訓練工具
