---
title: Observability 3.0 與 Causal Tracing
type: concept
tags: [system-design, observability, causal-tracing, agentic, distributed-systems, 2026]
created: 2026-05-15
updated: 2026-05-15
sources: [dev-to-system-design-2026]
---

# Observability 3.0 與 Causal Tracing

2026 可觀測性的新標準：將應用日誌、基礎設施指標、AI 模型決策三者整合為單一可追蹤的因果鏈。

## 為什麼需要 Observability 3.0

傳統 Observability（Logs + Metrics + Traces）為人類決策的系統設計：
- 請求失敗 → 找 log → 找對應 trace → 找 metric 相關性 → 人工判斷

**Agentic 系統的新問題：**
- AI 模型做了某個決策，導致下游行為異常 — 但傳統 trace 無法捕捉「模型為何這樣推理」
- 多個 AI Agent 協作時，錯誤的責任歸屬不清
- AI 的 non-determinism 使「重現問題」本身就很困難

## Causal Tracing（因果追蹤）

### 定義
將一個端對端事務的完整因果鏈記錄下來，包含：
1. **Application logs** — 服務調用、錯誤、業務事件
2. **Infrastructure metrics** — CPU/Memory/網路延遲
3. **AI model decisions** — 模型輸入、輸出、推理步驟（Thought/Action/Observation）、信心分數

### 概念架構

```
[User Request]
     ↓
[Trace ID 生成]
     ↓
[Service A] → log(trace_id, event, context)
     ↓
[AI Orchestrator]
  ├── log(trace_id, prompt, model, temperature)
  ├── log(trace_id, thought_step_1, tool_called)
  ├── log(trace_id, observation_1, confidence)
  └── log(trace_id, final_decision, reasoning_path)
     ↓
[Service B / External Tool]
     ↓
[Causal Trace Store] ← 將以上所有串聯
     ↓
[Causal Analysis Dashboard]
  → "為什麼 AI 選擇了工具 X 而非 Y？"
  → "哪個推理步驟導致最終錯誤？"
```

### 與 OpenTelemetry 的關係
- OTel 提供基礎的 Trace/Span 結構，Causal Tracing 是其 **AI 語義擴展**
- 需要自定義 Span attributes 記錄模型決策資訊
- 目前尚無統一標準（2026 年仍在演進中）

## Observability 3.0 的三個層次

| 層次 | 傳統 Observability | Observability 3.0 |
|------|-------------------|--------------------|
| **資料收集** | Logs + Metrics + Traces | + AI 決策路徑、Prompt/Response、Reasoning Chain |
| **關聯分析** | 人工查詢關聯 | 自動化因果推斷（Causal Inference）|
| **除錯模式** | 找問題在哪個服務 | 找問題在哪個推理步驟、哪個 Tool Call |

## 實作建議

1. **每個 AI 呼叫都附帶 Trace ID**，與服務層 Trace 串聯
2. **記錄完整 Prompt 與 Response**（注意 PII 脫敏，對照 [[Context Engineering最佳實踐]]）
3. **ReAct 的每個 Thought/Action/Observation 都是一個 Span**
4. **設置 AI 決策的 Anomaly Detection**：推理步驟異常增加、信心分數持續低落
5. **Causal Trace Store 使用 Lakehouse** 長期保留（對照 [[Data Mesh與Lakehouse]]）

## 面試中的可觀測性問題

當題目涉及 AI 系統時，主動提出 Causal Tracing 需求：
- 「這個 Agentic 系統需要記錄每個 Agent 的決策路徑，才能在出錯時進行 root cause analysis」
- 「AI 推理的 non-determinism 要求我們保留完整的 input/output log，而不只是 error log」

## 相關頁面

- [[AI-Native架構]] — Causal Tracing Engine 是 AI-Native 架構的必備元件
- [[ReAct Pattern]] — ReAct 的 Thought/Action/Observation 是 Causal Tracing 的天然 Span 邊界
- [[Serverless-First架構]] — 分散式 Serverless 環境下的追蹤挑戰
- [[Data Mesh與Lakehouse]] — Causal Trace 的長期儲存
- [[OpenTelemetry分散式追蹤]] — OTel 作為 Causal Tracing 的基礎設施
