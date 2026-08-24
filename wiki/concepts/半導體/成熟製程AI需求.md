---
title: 成熟製程AI需求
type: concept
tags: [semiconductor, ai, mature-node, pmic, bmc, investment, supply-chain]
created: 2026-04-20
updated: 2026-04-20
sources: [2026-04-20_unclestocknotes_umc-price-hike-h2-2026.md]
---

# 成熟製程AI需求

AI 基礎建設的需求不只來自先進製程 GPU——每台 AI 伺服器還需要大量成熟製程（28nm 以上，尤其 8 吋 BCD）元件，形成容易被忽視的結構性需求。

## 核心邏輯

```
AI 伺服器需求爆增
    ↓
GPU（先進製程，TSMC 3nm/5nm）← 大家都知道
    +
成熟製程元件（8吋 BCD）← 容易被忽視
    ↓
PMIC、BMC、高速介面控制器 全面吃緊
    ↓
成熟製程晶圓漲價（UMC、SMIC、VIS）
```

## AI 伺服器需要哪些成熟製程元件

| 元件 | 功能 | 製程 |
|------|------|------|
| **PMIC**（電源管理IC） | 伺服器各級電壓轉換與分配 | 8吋 BCD |
| **BMC**（主機板管理控制器） | 遠端監控、故障偵測 | 成熟邏輯製程 |
| **高速傳輸介面控制器** | PCIe、NVLink 等高速連接 | 成熟/特殊製程 |
| **電壓調節模組（VR）** | CPU/GPU 旁的即時電壓調節 | 成熟製程 |

## NVIDIA Vera Rubin 的放大效應

Vera Rubin 的極端功耗需求使 PMIC 規格大幅升級：
- 單顆晶片 TDP：**2,000W**
- 單機架：最高 **600kW**
- 電源密度：700kW/m²（現行高密度部署的 10 倍）
- 採用 48V 直接供電到晶片，需要全新 PMIC 架構

主要 PMIC 供應商：ADI、MPS、Richtek、Infineon、Renesas、onsemi、TI 等

## 供給端的結構性萎縮

8 吋廠產能在縮減，不是擴充：

| 事件 | 影響 |
|------|------|
| 三星 S7 八吋廠計畫關閉 | 全球 8 吋產能減少 |
| TSMC、三星優先先進製程 | 8 吋投資意願低 |
| 2026 全球 8 吋產能預估 | 年減 2.4% |
| 8 吋廠利用率 | 75–80%（2025）→ 85–90%（2026）|

**供需缺口 = 需求增（AI）+ 供給減（廠商轉向）**

## 投資意涵

- 成熟製程漲價有結構性支撐，不是短期現象
- 受益方：[[UMC 聯電]]、SMIC、VIS、Tower Semiconductor
- 下游受益：PMIC 設計廠（Richtek/立錡、MPS）

## 相關概念

- [[PMIC供需動態]] — 成熟製程需求最集中的元件
- [[AI伺服器供應鏈]] — 更大的供應鏈框架
- [[UMC 聯電]] — 漲價的直接受益者

## 來源

- [[unclestocknotes-umc-price-hike-h2-2026|UMC H2 2026 晶圓調漲分析]]
