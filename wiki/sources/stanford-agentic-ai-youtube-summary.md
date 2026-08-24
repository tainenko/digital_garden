---
title: 一部影片看完 Stanford AI 系統課程，從 LLM 到 Agentic Workflow
type: source-summary
tags: [stanford, llm, agentic-ai, rag, prompt-engineering, react-pattern, ai-agents]
created: 2026-05-15
updated: 2026-05-15
sources: [stanford-agentic-ai-youtube-summary]
---

# 一部影片看完 Stanford AI 系統課程，從 LLM 到 Agentic Workflow

## Origin

- **Title:** 一部影片看完 Stanford AI 系統課程，從 LLM 到 Agentic Workflow
- **Platform:** YouTube
- **URL:** https://www.youtube.com/watch?v=eKW9ITaltWw
- **Date ingested:** 2026-05-15
- **Original source:** Stanford Webinar — "Agentic AI: A Progression of Language Model Usage"
- **Language:** 繁體中文（摘要 Stanford 英文原課）

## Key Takeaways

- **LLM 基礎運作**：預訓練（Pre-training）= 海量語料 + next-token prediction；後訓練（Post-training）= 指令微調（SFT）+ RLHF 對齊人類偏好
- **LLM 五大限制**：幻覺（hallucinations）、知識截止日（knowledge cutoff）、無法引用來源、無法存取私有資料、Context 視窗有限
- **Prompting 六大策略**：明確指令、Few-Shot 範例、Chain-of-Thought、任務分解、提供上下文參考資料、系統性評估
- **RAG**：將外部知識庫預處理 → 向量儲存 → Similarity Search，注入至 Prompt，解決知識截止與私有資料問題
- **Tool Usage**：LM 生成 function call signature，呼叫 API、程式碼執行、網頁搜尋等外部工具
- **ReAct Pattern**：Reasoning（CoT 推理）+ Acting（工具/RAG/程式碼呼叫）交互循環，是從 Prompting 跨入 Agentic 的關鍵橋樑
- **Agentic 四大設計模式**：Planning（多步規劃）、Reflection（自我審查與迭代改善）、Tool Integration（工具整合）、Multi-agent Collaboration（多代理協作）
- **進化路徑**：Base LLM → Prompting → RAG → Tool Use → Single Agent（ReAct）→ Multi-agent

## Entities Mentioned

- [[Stanford]] — 原課程出處
- [[Anthropic]] — Claude 系列為 Agentic AI 代表性系統
- [[Andrej Karpathy]] — LLM 預訓練概念推廣者

## Concepts Mentioned

- [[LLM限制與解決方案]] — 幻覺/截止日/無私有資料/Context限制 + 對應解法
- [[ReAct Pattern]] — CoT Reasoning + Tool Acting 交互循環
- [[RAG檢索增強生成實戰]] — 外部知識注入標準做法
- [[Prompt Engineering進階]] — 六大 Prompting 策略
- [[AI Agent核心架構 Model+Context+Tools]] — Agentic 四大模式
- [[Context Engineering最佳實踐]] — Context 視窗限制的工程應對

## Contradictions/Tensions

- 課程以 RAG 為解決知識截止的主要方案，但 RAG 本身的 retrieval 品質問題（若 retrieve 不到或 snippet 誤導，仍會答錯）在影片中未深入討論；可對照 [[Context Engineering最佳實踐]] 的 Contextual Retrieval 做法

## Questions Raised

- Fine-Tuning 在此進化路徑中的位置？（影片標題提及但摘要中未詳述）
- Multi-agent 系統的協調機制（task routing、結果整合）如何設計？
- 如何評估 Agentic 系統的正確性？（對照 [[AI Agent評測基準]]）
