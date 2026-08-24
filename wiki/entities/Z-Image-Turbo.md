---
title: Z-Image-Turbo
type: entity
tags: [ai, image-generation, base-model, flux]
created: 2026-04-20
updated: 2026-04-20
sources: [lora-training-z-image-turbo]
---

# Z-Image-Turbo

二次元風格的 Flux 衍生基底模型，專為動漫/二次元圖像生成優化。

---

## 基本資訊

- **類型**：Flux 架構衍生模型
- **風格定位**：二次元動漫角色生成
- **用途**：作為 LoRA 訓練的基底模型（Base Model）

## 特性

- 預訓練對動漫角色有一定認知能力（可通過角色名稱直接生成已知角色）
- 訓練前建議確認模型是否已知悉目標角色，以決定訓練方向
- 與 [[Ostris AI Toolkit]] 配合使用進行 LoRA 微調

## 使用建議

- Linear Rank 建議設為 **16**（而非預設 32），減少對基底模型的非預期影響
- Timestep Bias 設為 **Low Noise** 可保留背景細節
- 推論工作流可透過 ComfyUI 重現

## 相關頁面

- [[LoRA訓練流程]] — 以此模型為基底的完整訓練流程
- [[Ostris AI Toolkit]] — 配套訓練工具
- [[Nano Banana Pro]] — 訓練資料生成工具
