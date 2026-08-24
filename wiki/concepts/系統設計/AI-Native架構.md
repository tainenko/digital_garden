---
title: AI-Native 架構
type: concept
tags: [system-design, ai-native, dslm, rag, agentic, mlops, 2026]
created: 2026-05-15
updated: 2026-05-15
sources: [dev-to-system-design-2026]
---

# AI-Native 架構

2026 系統設計的第一支柱：將 AI 能力嵌入架構的結構性核心，而非作為後置功能加入。

## 核心思想

傳統架構：Application → 偶爾呼叫 AI API
AI-Native 架構：AI 推理層嵌入請求路徑（request path），作為系統的一等公民

```
[User Request]
     ↓
[AI Orchestrator]
  ├── RAG Pipeline（知識增強）
  ├── Agent Execution（多步推理）
  └── DSLM Inference（領域專屬模型）
     ↓
[Response + Feedback Loop]
     ↓
[Training Pipeline]（運營資料回饋）
```

## 三個關鍵元素

### 1. RAG 嵌入請求路徑
- 不是「可選的知識庫查詢」而是每個請求的標準處理步驟
- 需要低延遲向量搜尋（sub-10ms retrieval）才能滿足端對端 SLA
- 對照 [[RAG檢索增強生成實戰]] 的工程實作

### 2. 領域專屬語言模型（DSLM）
- 從通用 LLM（GPT-4、Claude）轉向針對特定業務領域微調的小型模型
- 優勢：推理成本低、延遲小、領域準確率高、資料不外傳
- 挑戰：需要內部訓練資料、MLOps 基礎設施、持續評估流程
- 與 [[LLM限制與解決方案]] 中 Fine-Tuning 路徑呼應

### 3. 持續訓練回饋迴圈
- 運營資料（用戶行為、模型輸出評分、錯誤案例）→ 自動回饋訓練 Pipeline
- 需要 Feature Store 儲存實時特徵，供推理與訓練兩端共用
- 這是 AI-Native 與「裝上 AI 的傳統架構」的本質差別

## AI-Native 架構必備元件

| 元件 | 職責 | 對應工具 |
|------|------|---------|
| Agent Orchestrator | 管理多步 Agentic 推理流程 | LangGraph、Temporal |
| RAG System | 即時知識增強 | pgvector、Pinecone、Weaviate |
| Feature Store | 跨訓練/推理共享特徵 | Feast、Tecton |
| Model Registry | 版本管理、A/B 測試、回滾 | MLflow、Vertex AI |
| Causal Tracing Engine | 追蹤 AI 決策路徑（Observability 3.0）| 見 [[Observability 3.0 Causal Tracing]] |

## 設計師角色轉變

- 過去：定義元件、選型技術棧、設計 API
- 現在：為自治代理定義「約束邊界」（constraints）與「編排規則」（orchestration logic）
- 關鍵新技能：Prompt 工程化、Agent 評測（Evals）、訓練資料品質管理

## 相關頁面

- [[Observability 3.0 Causal Tracing]] — AI 系統的可觀測性要求
- [[RAG檢索增強生成實戰]] — RAG 完整工程實作
- [[ReAct Pattern]] — Agentic 推理的核心執行模式
- [[AI Agent核心架構 Model+Context+Tools]] — Agent 三層架構
- [[LLM限制與解決方案]] — DSLM 與 Fine-Tuning 的選擇脈絡
- [[Serverless-First架構]] — AI 推理的部署環境
