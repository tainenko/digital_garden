---
title: PMIC供需動態
type: concept
tags: [pmic, semiconductor, supply-chain, ai-server, investment, 2026]
created: 2026-04-20
updated: 2026-04-20
sources: [2026-04-20_unclestocknotes_umc-price-hike-h2-2026.md]
---

# PMIC 供需動態

電源管理 IC（Power Management IC）在 AI 伺服器爆發需求下，出現供給緊張、交期拉長、報價上升的結構性失衡。

## 什麼是 PMIC

將輸入電源（如 12V / 48V）轉換為各元件所需電壓，是所有電子設備的基礎元件。AI 伺服器對 PMIC 的需求特別高，因為：
- 更多高功耗晶片 → 更多電源轉換級數
- NVIDIA Vera Rubin 單機架 600kW → PMIC 規格大幅升級
- 效率要求提升（Vicor 架構達 98% @ 2,000W）

## 交期變化（2026 年）

| 元件 | 正常交期 | 目前交期 | 拉長幅度 |
|------|---------|---------|---------|
| PMIC | 21–26 週 | 35–40 週 | **+14 週** |
| BMC | 11–16 週 | 21–26 週 | **+10 週** |

交期拉長 = 供不應求的明確訊號。

## 供給面壓力

1. **8 吋 BCD 產能外流**：供應商優先接高毛利 AI PMIC 訂單，排擠一般 BMC/PMIC
2. **原物料漲價**：黃金、銅等金屬成本上升
3. **廠商縮減 8 吋**：三星關閉 S7、TSMC 減少 8 吋投資
4. **晶圓漲價傳導**：[[UMC 聯電]]、SMIC、VIS 調漲 BCD 製程 10–20%

## 需求面驅動

1. **AI 伺服器出貨量**：2026 年預估成長 28%
2. **單台功耗提升**：每台 AI 伺服器需 250–900kW 電力供應
3. **Vera Rubin 帶動**：NVIDIA Vera Rubin 的超高功耗設計需要全新 PMIC 架構

## 主要受益廠商（PMIC 設計）

| 廠商 | 屬性 |
|------|------|
| MPS（Monolithic Power Systems） | 美股，AI PMIC 龍頭 |
| Richtek / 立錡 | 台股 6286，台系龍頭 |
| Renesas | 日系 |
| Infineon | 歐系 |
| ADI（Analog Devices） | 美系 |
| onsemi | 美系 |
| TI（Texas Instruments） | 美系 |

## 相關概念

- [[成熟製程AI需求]] — PMIC 是成熟製程需求最集中的元件
- [[AI伺服器供應鏈]] — 供應鏈全貌
- [[UMC 聯電]] — PMIC 的晶圓代工供應商

## 來源

- [[unclestocknotes-umc-price-hike-h2-2026|UMC H2 2026 晶圓調漲分析]]
