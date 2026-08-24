---
title: "Z-Image-Turbo 二次元人物 LoRA 訓練經驗完整分享"
source: "https://vocus.cc/article/695e5799fd89780001575129"
author:
  - "[[蘇嘉冠 JiaKuan Su]]"
published: 2026-01-10
created: 2026-04-20
tags:
  - clippings
  - ai
  - lora
  - image-generation
---

## Part I：初次訓練 — 方向確認與工作流建立

**角色選擇**：以《偶像大師灰姑娘女孩 U149》的橘愛莉絲為範例（12歲，141cm）。訓練前先確認 Z-Image-Turbo 基底模型能否用角色名稱生成目標角色。

**資料收集**：
- 主要來源：Danbooru 圖庫
- 條件：單人、1024px 以上、彩色、無文字、統一服裝、多角度
- 搜尋指令：`tachibana_arisu solo height:>=1024 width:>=1024 -monochrome -comic`
- 工具：imgbrd-grabber 批次下載
- 初始資料量：10–15 張

**訓練設定**：
- 工具：Ostris AI Toolkit
- 基底模型：Z-Image-Turbo
- Trigger word：zzArisuTachibana
- 訓練步數：3,000
- 樣本間隔：每 250 步
- 硬體：Runpod RTX 5090，約 $1–2 USD，1–2 小時

---

## Part II：迭代改善 — 透過目標設定提升品質

**評估目標**：
1. LoRA 強化 vs 基底模型比較
2. 相同 prompt 不同 seed 的一致性測試
3. 換裝測試（驗證彈性）

**關鍵參數調整**：
- QUANTIZATION：Transformer 和 Text Encoder 設為 None
- Low VRAM：關閉（提升速度）
- TARGET Linear Rank：從 32 降至 **16**
- Max Step Saves：從 4 提升至 **12**
- Batch Size：VRAM 允許時提升至 2
- Timestep Bias：從 Balanced 改為 **Low Noise**（保留背景細節）
- Resolution：只保留 512px
- Cache Latents：啟用（加速）

**訓練資料改善策略**：
- Danbooru 問題：多畫師、多畫風 → 角色外觀不一致
- 解法：用 Nano Banana Pro 生成「角色設定圖」再延伸變化，14–15 張，統一畫風

**標籤策略（Feature Disentanglement）**：
- 外觀標籤：zzArisuTachibana, blue hair bow, sidelocks
- 服裝標籤：blue dress, pleated skirt, short sleeves, white collar, buttons, brown belt, white socks, brown loafers
- 其他：anime, 動作, 場景, 環境描述

**模型選擇**：以 250 步間隔的樣本圖作為選擇依據（而非 Loss 曲線）。原則：「剛剛好」，最小化對基底模型的非預期影響。最終選定：Step 1,500

---

## Part III：重現

下載 CivitAI 或 GitHub 預訓練模型，放入 ComfyUI LoRA 資料夾，載入範例工作流程即可重現。
