---
title: "Z-Image-Turbo 二次元人物 LoRA 訓練經驗完整分享"
type: source
source: "https://vocus.cc/article/695e5799fd89780001575129"
author: "[[蘇嘉冠 JiaKuan Su]]"
published: 2026-01-10
ingested: 2026-04-20
tags: [ai, lora, image-generation, training]
---

# Z-Image-Turbo 二次元人物 LoRA 訓練經驗完整分享

原始來源：https://vocus.cc/article/695e5799fd89780001575129

---

## 核心論點

以《偶像大師灰姑娘女孩 U149》橘愛莉絲為案例，系統性分享 LoRA 訓練的三個階段：初次訓練建立工作流、迭代改善提升品質、最終重現模型。

---

## Key Takeaways

1. **基底模型驗證先行**：訓練前先確認 Z-Image-Turbo 是否已知悉目標角色，避免訓練方向錯誤。
2. **資料品質 > 資料數量**：Danbooru 多畫師導致畫風不統一；改用 [[Nano Banana Pro]] 生成 14–15 張統一畫風的「角色設定圖」，效果顯著提升。
3. **Feature Disentanglement**：將外觀、服裝、場景分類標籤，讓模型學會獨立控制各維度，提升換裝彈性。
4. **步數選擇靠目視，非 Loss**：以 250 步間隔的樣本圖作為選擇依據；Loss 曲線不代表視覺品質。最終選 Step 1,500（共訓練 3,000 步）。
5. **關鍵參數**：Linear Rank 降至 16（減少對基底的干擾）、Timestep Bias 改為 Low Noise（保留背景細節）。
6. **成本極低**：Runpod RTX 5090，約 $1–2 USD，1–2 小時。

---

## 工具鏈

| 工具 | 用途 |
|------|------|
| [[Ostris AI Toolkit]] | LoRA 訓練主工具 |
| [[Z-Image-Turbo]] | 基底模型（二次元風格） |
| imgbrd-grabber | Danbooru 批次下載 |
| [[Nano Banana Pro]] | 生成統一畫風訓練資料 |
| ComfyUI | 推論與工作流重現 |
| Runpod | 雲端 GPU（RTX 5090） |

---

## 訓練參數摘要

| 參數 | 設定值 |
|------|--------|
| Training Steps | 3,000 |
| Sample Interval | 每 250 步 |
| TARGET Linear Rank | **16**（從 32 降） |
| Max Step Saves | **12**（從 4 升） |
| Batch Size | 2（VRAM 允許時） |
| Timestep Bias | **Low Noise** |
| Resolution | 512px |
| Cache Latents | 啟用 |
| QUANTIZATION | None（Transformer + Text Encoder） |
| Selected Step | **1,500** |

---

## 相關概念

- [[LoRA訓練流程]] — 完整三階段工作流
- [[Feature Disentanglement]] — 標籤解耦技術
- [[訓練資料策略]] — Danbooru vs 合成資料的選擇

---

## 開放問題

- Step 1,500 的選擇是否有更系統化的標準（如 FID score）？
- Nano Banana Pro 生成資料對非動漫角色的效果？
- Z-Image-Turbo 以外的 Flux 衍生模型是否適用相同參數？
