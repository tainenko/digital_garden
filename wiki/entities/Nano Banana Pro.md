---
title: Nano Banana Pro
type: entity
tags: [ai, image-generation, data-preparation]
created: 2026-04-20
updated: 2026-04-20
sources: [lora-training-z-image-turbo]
---

# Nano Banana Pro

用於生成統一畫風角色設定圖的 AI 圖像生成工具，可解決 Danbooru 多畫師導致的畫風不一致問題。

---

## 基本資訊

- **類型**：AI 圖像生成工具
- **主要用途**：為 LoRA 訓練生成統一畫風的訓練資料

## 核心用途：解決訓練資料畫風不統一

**問題**：直接從 Danbooru 下載訓練資料時，由於來源畫師不同，角色外觀（髮型細節、眼睛形狀、身體比例）在不同圖片中不一致，導致訓練出的 LoRA 角色一致性差。

**解法**：使用 Nano Banana Pro 先生成 14–15 張統一畫風的「角色設定圖」（不同角度、表情、動作），再以此作為訓練資料，確保所有圖片的角色外觀一致。

## 效果對比

| 資料來源 | 優點 | 缺點 |
|----------|------|------|
| Danbooru | 資料量大、多樣性高 | 多畫師 → 畫風不統一 → 角色一致性差 |
| Nano Banana Pro 生成 | 畫風統一、角色一致性強 | 需要額外生成步驟 |

## 相關頁面

- [[訓練資料策略]] — 完整的資料準備策略
- [[LoRA訓練流程]] — 訓練整體流程
- [[Z-Image-Turbo]] — 配套基底模型
