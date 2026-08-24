---
title: Ostris AI Toolkit
type: entity
tags: [ai, lora, training-tool]
created: 2026-04-20
updated: 2026-04-20
sources: [lora-training-z-image-turbo]
---

# Ostris AI Toolkit

用於 Flux 系列模型 LoRA 訓練的開源工具套件。

---

## 基本資訊

- **類型**：開源訓練工具
- **主要用途**：Flux / Z-Image-Turbo 等模型的 LoRA 微調訓練
- **配套推論工具**：ComfyUI

## 關鍵訓練參數

| 參數 | 說明 |
|------|------|
| TARGET Linear Rank | LoRA 矩陣秩，越低對基底影響越小（建議 16） |
| Timestep Bias | Balanced / Low Noise；Low Noise 可保留背景細節 |
| Max Step Saves | 儲存中間檢查點數量，建議提升至 12 以方便選模 |
| Cache Latents | 啟用後加速訓練 |
| QUANTIZATION | 建議 Transformer + Text Encoder 皆設為 None |
| Low VRAM Mode | VRAM 充足時關閉以提升速度 |

## 使用流程

1. 設定基底模型（[[Z-Image-Turbo]] 等）
2. 配置 Trigger Word（如 `zzArisuTachibana`）
3. 準備訓練資料（Danbooru 或 [[Nano Banana Pro]] 生成）
4. 設定標籤（[[Feature Disentanglement]]）
5. 執行訓練（建議 3,000 步，每 250 步取樣）
6. 依樣本圖目視選出最佳 checkpoint

## 相關頁面

- [[Z-Image-Turbo]] — 常用基底模型
- [[LoRA訓練流程]] — 完整訓練流程
- [[Feature Disentanglement]] — 標籤策略
