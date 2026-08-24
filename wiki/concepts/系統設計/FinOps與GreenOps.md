---
title: FinOps 與 GreenOps
type: concept
tags: [system-design, finops, greenops, cost-optimization, carbon-aware, cloud, 2026]
created: 2026-05-15
updated: 2026-05-15
sources: [dev-to-system-design-2026]
---

# FinOps 與 GreenOps

2026 系統設計的第四支柱：雲端成本與碳排放的系統性治理，從事後分帳演進為架構設計的一等約束。

## FinOps：雲端財務工程化

### 核心理念
「財務責任」不是 Finance 的問題，而是工程決策的一部分。每個架構選擇都有成本後果，需要在設計階段量化。

### 四大 FinOps 實踐

| 實踐 | 說明 |
|------|------|
| **Service Right-sizing** | 持續監控實際使用率，自動縮減過度配置的資源規格 |
| **Serverless 顆粒度優化** | 函數切割過細導致呼叫開銷 > 執行時間；找到最佳顆粒度 |
| **自動資料分層** | Hot/Warm/Cold 資料自動遷移至對應成本儲存層（S3 Standard → Glacier）|
| **廠商中立多雲策略** | 避免單一雲廠商鎖定，保留議價空間與遷移彈性 |

### 成本可觀測性
- 每個服務、每個 API 端點的成本追蹤（細粒度 cost attribution）
- 成本與業務指標對齊：Cost per Transaction、Cost per Active User
- 異常成本自動告警（避免 runaway jobs 燒掉預算）

## GreenOps：碳排放工程化

### 為什麼 2026 是關鍵年
- ESG 法規要求企業揭露 Scope 3 碳排放（含雲端運算）
- 大型雲廠商（AWS、GCP、Azure）開始提供碳排放 API
- AI 訓練的能耗已成為重大碳排放來源

### 三個 GreenOps 模式

**1. Carbon-aware Scheduling（碳感知排程）**
- 非時效性工作（批次訓練、資料管道、備份）調度至低碳排時段/區域
- 工具：Carbon Aware SDK（微軟開源）、Google Carbon Footprint API
- 原則：同等成本下，選擇再生能源比例更高的 Region

**2. 能源高效執行層**
- 客製化晶片（AWS Graviton、Google TPU）在相同工作負載下能耗更低
- Serverless 本身即 GreenOps：idle 時不耗能，資源利用率更高

**3. 模型效率優化**
- DSLM（小型領域模型）vs 通用大模型：推理能耗相差 10–100 倍
- 量化（Quantization）、蒸餾（Distillation）降低 AI 推理碳足跡

## FinOps × GreenOps 的協同

成本優化與碳優化通常方向一致：
- 減少 idle 資源 → 降成本 + 降碳排
- 使用更高效的執行環境 → 降成本 + 降碳排
- 例外：跨 Region 遷移至低碳區域可能增加資料傳輸成本

## 系統設計面試中的 FinOps 思路

面試時主動提出成本考量可以展示工程師成熟度：
- 「此設計的成本模型是 X，預估月費 Y」
- 「Serverless 在這個流量模式下比固定伺服器省 Z%」
- 「RAG 向量索引的儲存成本需要 Cold tier 策略」

## 相關頁面

- [[Serverless-First架構]] — Serverless 是 FinOps 的天然盟友
- [[AI-Native架構]] — DSLM 的推理成本 vs 通用 LLM
- [[Data Mesh與Lakehouse]] — 資料儲存的 Hot/Warm/Cold 分層
- [[系統設計面試模板]] — 面試中加入成本估算的環節
