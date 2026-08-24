---
title: AI伺服器供應鏈
type: concept
tags: [ai-server, supply-chain, semiconductor, pmic, gpu, investment]
created: 2026-04-20
updated: 2026-04-20
sources: [2026-04-20_unclestocknotes_umc-price-hike-h2-2026.md]
---

# AI 伺服器供應鏈

AI 伺服器不只是 GPU——完整的供應鏈涵蓋從先進製程到成熟製程的各類元件，形成一條被廣泛低估的需求鏈。

## 供應鏈全貌

```
AI 伺服器
├── 計算層（先進製程，TSMC 3nm/5nm）
│   ├── GPU（NVIDIA H100/B200/Vera Rubin）
│   ├── CPU（Intel/AMD）
│   └── 網路 ASIC（Broadcom/Marvell）
│
├── 電源層（成熟製程，8吋 BCD）← 容易被忽視
│   ├── PMIC（電源管理IC）
│   ├── 電壓調節模組（VR）
│   └── 電源供應器（PSU）
│
├── 管理層（成熟製程）
│   └── BMC（主機板管理控制器）
│
├── 連接層（成熟/特殊製程）
│   ├── 高速介面控制器（PCIe/NVLink）
│   └── 網路交換器 ASIC
│
└── 儲存層
    ├── HBM（高頻寬記憶體）
    └── SSD 控制器
```

## 為什麼成熟製程元件被低估

投資人目光聚焦在 GPU/先進製程，但：
1. GPU 每台伺服器可能只有幾顆，但 PMIC 可能有數十顆
2. 成熟製程產能萎縮（8吋廠關閉），需求卻在爆增
3. PMIC 交期已從 21 週拉長至 35 週（+14週）

詳見[[成熟製程AI需求]]與[[PMIC供需動態]]。

## 2026 供應鏈緊張點

| 元件 | 緊張原因 | 影響廠商 |
|------|---------|---------|
| PMIC | 8吋 BCD 供給萎縮 + AI 需求爆增 | [[UMC 聯電]]、Richtek、MPS |
| BMC | 供應商優先 AI 高毛利訂單 | ASPEED（亞信）|
| HBM | 先進製程產能有限 | SK Hynix、Samsung、Micron |
| 電源供應器 | 600kW 機架要求全新設計 | Delta（台達電） |

## 相關概念

- [[成熟製程AI需求]] — 成熟製程元件的需求邏輯
- [[PMIC供需動態]] — 最緊張的成熟製程元件

## 來源

- [[unclestocknotes-umc-price-hike-h2-2026|UMC H2 2026 晶圓調漲分析]]
