---
title: LoRA 人像訓練最佳實踐（Flux / Z-Image-Turbo）
type: topic
tags: [ai, lora, portrait, flux, z-image-turbo, training]
created: 2026-04-20
updated: 2026-04-20
sources: [lora-training-z-image-turbo]
---

# LoRA 人像訓練最佳實踐（Flux / Z-Image-Turbo）

針對寫實人像與二次元角色 LoRA，整合 Flux.1-dev、Z-Image-Turbo 兩條主流路線的完整參數指南。

---

## 模型選擇：先確定路線

| 模型 | 最適用途 | 推理速度 | 訓練難度 |
|------|---------|---------|---------|
| **Flux.1-dev** | 寫實人像、高品質角色 | 慢（20–50步） | 中 |
| **Z-Image-Turbo** | 二次元角色、快速迭代 | 快（4–8步） | 低 |
| Flux.1-schnell | — | 極快 | ⚠️ 不建議訓練 LoRA |
| Pony | 動漫/插畫風格 | 中 | 中 |

> **核心決策**：追求寫實人像 → Flux.1-dev；追求二次元一致性 + 快速生成 → [[Z-Image-Turbo]]。

---

## 訓練資料準備（最關鍵步驟）

### 圖片數量

| 場景 | 建議數量 |
|------|---------|
| 最低可用 | 15 張 |
| 一般角色 LoRA | 20–25 張 |
| Z-Image-Turbo | 30–50 張（上限 120，超過容易過擬合） |
| Flux.1-dev 精細版 | 25–40 張 |

> 黃金法則：**品質 > 數量**。20 張角度多樣的高解析圖，遠勝 100 張混雜低品質圖。

### 解析度與格式

- **Flux.1-dev**：1024×1024 px（強制，PNG 格式）
- **Z-Image-Turbo**：512px 或 1024×1024 均可；建議 1024 以獲得最佳細節
- **格式**：一律 PNG，JPEG 壓縮會影響細節學習

### 構圖多樣性（影響臉部一致性的最大因素）

必須覆蓋以下角度：

```
臉部角度：
  - 正面 ×2–3 張
  - 左 3/4 側臉 ×2–3 張
  - 右 3/4 側臉 ×2–3 張
  - 左正側面 ×1–2 張（可選）
  - 右正側面 ×1–2 張（可選）

景別：
  - 特寫（臉部佔畫面 ≥15%）×4–6 張
  - 半身 ×4–6 張
  - 全身 ×2–4 張

表情：
  - 自然表情、微笑、嚴肅 ×各1–2 張

光線：
  - 自然光、室內光、側光 ×各1–2 張（Flux 寫實尤其重要）
```

> ⚠️ **臉部佔比**：確保臉部面積佔圖片至少 15%，否則模型難以提取身份特徵。

### 資料來源策略

**Danbooru（二次元）**：
- 優點：量大免費、標籤豐富
- 缺點：多畫師 → 畫風不統一 → 角色外觀各圖差異大
- 搜尋條件：`solo height:>=1024 width:>=1024 -monochrome -comic`
- **適合**：第一輪快速驗證

**[[Nano Banana Pro]] 合成資料**：
- 先生成 14–15 張統一畫風的「角色設定圖」
- 解決多畫師問題，顯著提升角色一致性
- **適合**：正式訓練、重視一致性時

**自拍/真人照片（Flux 寫實路線）**：
- 需覆蓋上述多角度構圖
- 建議包含不同服裝（減少服裝與臉部特徵的耦合）
- 移除遮臉、強逆光、極端角度的照片

---

## 標籤策略（Caption Strategy）

### Flux vs SD1.5 的差異

Flux 的 Text Encoder（T5）理解自然語言，**不要使用 tag 風格**，要用完整句子描述。

**好的 Caption（Flux）**：
> `A woman with curly red hair and freckles, wearing a blue denim jacket, smiling in a park.`

**不好的 Caption（Flux）**：
> `1girl, red hair, freckles, denim jacket, smile, outdoors`

### 三層解耦（Feature Disentanglement）

參見 [[Feature Disentanglement]]，核心原則：

| 層 | 內容 | 目標 |
|----|------|------|
| **身份層**（每張必有） | Trigger word + 獨特外觀特徵（髮色、臉部特徵） | 鎖定角色身份 |
| **服裝層**（依圖填寫） | 當前服裝的完整描述 | 讓服裝可替換 |
| **情境層**（依圖填寫） | 場景、動作、光線、表情 | 讓場景可控 |

> **關鍵邏輯**：你想要在推論時自由控制的屬性，就要在 Caption 中描述它；你想要「燒進」角色身份的屬性，就不要在 Caption 中描述（讓模型自動關聯到 trigger word）。

### Trigger Word 設計

- 選用罕見詞組，避免與基底模型現有概念衝突
- 推薦格式：`zz角色名英文`（如 `zzArisuTachibana`、`zzJohnSmith`）
- 避免使用已知名詞（如 `john`、`girl`）

### 雙模型 Caption（進階）

同時使用 WD14（tag）+ LLM（自然語言）生成 caption，確保 Clip 和 T5 兩個 encoder 都能獲取資訊。工具：JoyCaption、Florence-2。

---

## 訓練參數

### Flux.1-dev 推薦設定

```yaml
# Ostris AI-Toolkit 格式
network:
  type: lora
  linear: 32          # Rank，人像建議 16–32
  linear_alpha: 16    # Alpha = Rank / 2

train:
  steps: 1500         # 約 20–25 repeats × 20–25 張圖
  lr: 2e-4            # 比 SDXL 高一個數量級
  optimizer: adamw8bit
  noise_scheduler: flowmatch
  ema: true           # 強烈建議開啟
  gradient_checkpointing: true
  cache_latents: true

  # 樣本監控
  sample_every: 250
  save_every: 500
  max_step_saves: 12

dataset:
  resolution: [1024, 1024]
```

### Z-Image-Turbo 推薦設定

```yaml
network:
  type: lora
  linear: 16          # Rank，Z-Turbo 用 8–16 足夠
  linear_alpha: 16    # Rank = Alpha（ratio 1.0）

train:
  steps: 3000
  lr: 2e-4
  quantization: none  # Transformer + Text Encoder 皆設 None
  low_vram: false     # VRAM 充足時關閉
  timestep_bias: low_noise  # 保留背景細節
  cache_latents: true

  sample_every: 250   # 每 250 步目視檢查
  max_step_saves: 12

dataset:
  resolution: [512]   # 或 [1024, 1024]
```

### 參數速查對比

| 參數 | Flux.1-dev | Z-Image-Turbo | 說明 |
|------|-----------|--------------|------|
| Rank | 16–32 | 8–16 | 低 rank 足夠，更高 rank 少有提升 |
| Alpha | = Rank/2 | = Rank | Flux 用半值，Z-Turbo 用等值 |
| Learning Rate | 1e-4 ~ 2e-4 | 2e-4 ~ 4e-4 | 比 SDXL 高 10倍 |
| Steps | 1000–2000 | 2000–3000 | 依圖片數量調整 |
| Resolution | 1024px | 512 或 1024px | Z-Turbo 512 即可 |
| Quantization | None | None | 開啟會降低精度 |
| EMA | 開啟 | 可選 | 提升穩定性 |
| Timestep Bias | Balanced | Low Noise | Low Noise 保留背景 |
| Cache Latents | 開啟 | 開啟 | 加速訓練 |

### Steps 計算公式

```
目標 repeats = 20–25
Steps ≈ 圖片數 × 目標 repeats

範例：20 張圖 × 25 repeats = 500 steps（較快）
範例：20 張圖 × 75 repeats = 1500 steps（推薦）
```

---

## Checkpoint 選擇（靠目視，不靠 Loss）

Loss 曲線無法反映視覺品質。正確做法：

1. 每 250 步生成一組固定 seed 的樣本圖
2. 用同一組 prompt 比較不同步數的輸出
3. 選擇「剛好」的 checkpoint——角色特徵穩定，但未過擬合到訓練圖的固定動作/場景
4. 典型最佳 checkpoint：總步數的 50%–70%（如 3000 步訓練選 Step 1500–2000）

**過擬合的訊號**：
- 不管 prompt 怎麼寫，生成結果雷同
- 角色的光線/背景幾乎固定
- 換裝後臉部也跟著變化

**欠擬合的訊號**：
- Trigger word 影響力弱
- 角色特徵不穩定（不同 seed 差異大）

---

## 常見問題排除

| 問題 | 原因 | 解法 |
|------|------|------|
| 臉部不一致（不同 seed 差異大） | 訓練圖角度不夠多 / 臉部佔比太小 | 補充多角度圖；確保臉部 ≥ 15% |
| 換裝時臉也變了 | 服裝特徵未標籤，被燒進身份 | 完整填寫服裝層 caption |
| 固定背景洩漏（背景總是一樣） | 所有訓練圖背景相同 | 增加背景多樣性；或在 caption 中標記背景 |
| 過擬合（輸出變化少） | LR 過高 / 步數過多 | 降 LR（2e-4 → 8e-5）或用更早的 checkpoint |
| 訓練圖數量少（<20 張）的過擬合 | Z-Image-Turbo 特有問題 | 使用 cosine LR decay；加輕微資料增強（color jitter） |
| Flux LoRA 效果平淡 | Caption 用了 tag 格式而非自然語言 | 改用完整自然語言描述 |
| 風格漂移 | 訓練資料風格不統一 | 改用 [[Nano Banana Pro]] 生成統一畫風資料 |

---

## 工具選擇

| 工具 | 特點 | 適合場景 |
|------|------|---------|
| **[[Ostris AI Toolkit]]** | 速度快（比 SimpleTuner 快 20–30%）、設定簡單 | 快速迭代、Z-Image-Turbo 訓練 |
| **SimpleTuner** | 最穩定、文件完整、參數細粒度控制 | 正式生產、需要可重現結果 |
| **kohya-ss** | GUI 介面、適合新手 | 不熟悉 CLI 的用戶 |
| **FluxGym** | 低 VRAM 支援（12–16GB）、極簡界面 | VRAM 不足時 |
| **CivitAI Trainer** | 無需本地 GPU、$2/次 | 一次性嘗試、沒有本地硬體 |

### 雲端 GPU 成本估算

| 平台 | 硬體 | 費用 | 時間 |
|------|------|------|------|
| Runpod | RTX 5090 | ~$1–2 USD | 1–2 小時 |
| Google Colab | L4 | ~$1.4 USD | ~4.5 小時 |
| CivitAI | 內建 | 2000 buzz（≈$2） | 視佇列 |

---

## 三步快速上手

### Flux.1-dev 寫實人像
```
1. 收集 20–25 張多角度照片（1024px，PNG）
2. 用 JoyCaption / Florence-2 生成自然語言 caption
   → 身份特徵不標（讓模型自動學），服裝/場景完整標
3. Ostris AI-Toolkit：rank 32, alpha 16, lr 2e-4, 1500 steps
   → 每 250 步目視檢查，選 Step 750–1250
```

### Z-Image-Turbo 二次元角色
```
1. Nano Banana Pro 生成 15 張統一畫風角色設定圖（多角度）
2. 三層標籤：trigger word + 外觀 / 服裝 / 情境分離
3. Ostris AI-Toolkit：rank 16, alpha 16, lr 2e-4,
   timestep_bias=low_noise, 3000 steps
   → 目視選 Step 1500 左右
```

---

## 相關頁面

- [[LoRA訓練流程]] — Z-Image-Turbo 三階段詳細流程
- [[Feature Disentanglement]] — 標籤解耦原理與實作
- [[訓練資料策略]] — 資料收集與品質控制
- [[Z-Image-Turbo]] — 基底模型特性
- [[Ostris AI Toolkit]] — 訓練工具參數參考
- [[Nano Banana Pro]] — 統一畫風資料生成
