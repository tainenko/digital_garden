---
title: Serverless-First 架構
type: concept
tags: [system-design, serverless, edge-computing, stateful-serverless, cost, 2026]
created: 2026-05-15
updated: 2026-05-15
sources: [dev-to-system-design-2026]
---

# Serverless-First 架構

2026 系統設計的第二支柱：以 Serverless 為預設執行環境，而非作為例外選項。

## 核心思想

傳統架構思維：「我需要幾台伺服器？」
Serverless-First 思維：「只有確認 Serverless 無法滿足時，才考慮傳統 VM」

## 三個關鍵驅動因素

### 1. Pay-per-use 強制成本紀律
- 不執行就不付錢，消除 idle capacity 浪費
- 天然對齊 [[FinOps與GreenOps]] 的成本優化目標
- 缺點：高流量穩定場景下 per-request 成本可能高於預留容量

### 2. Stateful Serverless 成熟化
傳統 Serverless 的核心限制是「無狀態」。2026 年 Stateful Serverless 模式成為主流：

| 模式 | 說明 | 代表技術 |
|------|------|---------|
| Durable Functions | 長時間執行的有狀態工作流 | Azure Durable Functions |
| Durable Objects | 每個物件維護自身狀態，地理分散 | Cloudflare Durable Objects |
| Actor Model | 每個 Actor 封裝狀態 + 行為 | Dapr、Orleans |
| Workflow Engine | 複雜 Agentic 流程的有狀態協調 | Temporal、LangGraph |

### 3. 邊緣運算（Edge Computing）
- 2026 年 75% 企業資料在邊緣處理（IDC 預測）
- 推動力：AI 推理延遲要求、資料主權法規、頻寬成本
- 架構模式：CDN 邊緣函數處理輕量邏輯，回源至核心 Serverless 處理複雜計算

```
[User]
  ↓ <5ms
[Edge Function] → 快取命中、A/B、身份驗證、輕量 AI 推理
  ↓ <50ms（僅 cache miss 或複雜請求）
[Core Serverless] → 業務邏輯、資料庫、完整 AI 推理
  ↓
[Data Layer / Lakehouse]
```

## Serverless-First 適用邊界

| 適合 | 不適合 |
|------|--------|
| 事件驅動、非同步工作流 | 超低延遲 (<1ms) 需求 |
| 流量高度不均勻 | 持續高吞吐量穩定負載 |
| AI 推理端點 | 需要 GPU 長時間佔用的訓練任務 |
| 資料處理管道 | 大量共享記憶體的運算 |

## 與 AI-Native 架構的整合

- DSLM 推理部署於邊緣 Serverless（低延遲、資料不外傳）
- Agentic Workflow 使用 Durable Functions / Temporal 管理多步有狀態執行
- RAG 的向量搜尋可部署於 Serverless 函數，但冷啟動延遲需監控

## 冷啟動問題的處理

- 預熱（Provisioned Concurrency / Warm instances）
- 輕量 Runtime 選擇（Go、Rust 取代 JVM 系語言）
- 邊緣函數通常無冷啟動問題（V8 Isolate 模型）

## 相關頁面

- [[FinOps與GreenOps]] — Serverless 的成本優化策略
- [[AI-Native架構]] — AI 推理在 Serverless 環境的部署
- [[Observability 3.0 Causal Tracing]] — Serverless 的可觀測性挑戰（分散式追蹤）
- [[分散式系統基礎概念]] — 無狀態 vs 有狀態的基礎討論
- [[sd-cdn|CDN 設計]] — 邊緣層設計模式
