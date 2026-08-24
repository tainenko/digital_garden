---
title: LoRA訓練流程
type: concept
tags: [ai, lora, image-generation, training]
created: 2026-04-20
updated: 2026-04-20
sources: [lora-training-z-image-turbo]
---

# LoRA 訓練流程

以二次元角色 LoRA 訓練為例的三階段完整工作流程。

---

## Phase 1：初次訓練 — 方向確認與工作流建立

### 1. 基底模型驗證
確認 [[Z-Image-Turbo]] 等基底模型是否已能用角色名稱生成目標角色，評估 LoRA 訓練的必要性與方向。

### 2. 資料收集
- **來源**：Danbooru（`solo height:>=1024 width:>=1024 -monochrome -comic`）
- **工具**：imgbrd-grabber 批次下載
- **初始量**：10–15 張即可開始

### 3. 初次訓練
```
工具：Ostris AI Toolkit
基底：Z-Image-Turbo
Trigger Word：自訂（如 zzCharacterName）
Steps：3,000
Sample Interval：每 250 步
硬體：Runpod RTX 5090（~$1–2 USD）
```

---

## Phase 2：迭代改善 — 透過目標設定提升品質

### 評估三指標
1. LoRA 強化效果 vs 基底模型基礎能力
2. 相同 prompt 不同 seed 的一致性
3. 換裝彈性測試

### 關鍵參數調整

| 參數 | 預設 → 調整後 | 原因 |
|------|-------------|------|
| QUANTIZATION | 開啟 → **None** | 提升精度 |
| Low VRAM | 開啟 → **關閉** | 提升速度 |
| TARGET Linear Rank | 32 → **16** | 減少對基底的非預期干擾 |
| Max Step Saves | 4 → **12** | 更多 checkpoint 選擇 |
| Timestep Bias | Balanced → **Low Noise** | 保留背景細節 |
| Cache Latents | 關閉 → **啟用** | 加速訓練 |

### 訓練資料改善
- 改用 [[Nano Banana Pro]] 生成統一畫風的角色設定圖（14–15 張）
- 解決 Danbooru 多畫師導致的畫風不統一問題

### 標籤策略（[[Feature Disentanglement]]）
- **外觀標籤**：trigger word + 髮色髮型特徵
- **服裝標籤**：完整服裝細節
- **情境標籤**：動作、場景、環境（可替換部分）

### Checkpoint 選擇原則
- **目視選擇**，非依賴 Loss 曲線
- 原則：「剛剛好」——最小化對基底模型的非預期影響
- 典型選擇：Step 1,500（3,000步訓練中的中間點）

---

## Phase 3：重現

1. 下載 CivitAI 或 GitHub 上的預訓練 LoRA 檔案
2. 放入 ComfyUI 的 LoRA 資料夾
3. 載入範例工作流程即可

---

## 成本估算

| 項目 | 數值 |
|------|------|
| 硬體 | Runpod RTX 5090 |
| 費用 | ~$1–2 USD |
| 時間 | 1–2 小時 |

---

## 相關頁面

- [[Feature Disentanglement]] — 標籤解耦詳解
- [[訓練資料策略]] — 資料收集與品質控制
- [[Z-Image-Turbo]] — 常用基底模型
- [[Ostris AI Toolkit]] — 訓練工具
- [[Nano Banana Pro]] — 合成訓練資料工具
