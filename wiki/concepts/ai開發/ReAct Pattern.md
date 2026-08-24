---
title: ReAct Pattern
type: concept
tags: [react, reasoning, acting, agent, tool-use, chain-of-thought, llm]
created: 2026-05-15
updated: 2026-05-15
sources: [stanford-agentic-ai-youtube-summary]
---

# ReAct Pattern

**ReAct = Reasoning + Acting**。將 Chain-of-Thought 推理與外部工具呼叫交互循環，是從純 Prompting 跨入 Agentic AI 的關鍵設計橋樑。

## 核心概念

傳統 LLM 呼叫是單次的：輸入 → 模型 → 輸出。
ReAct 打破這個邊界，讓模型在輸出中夾帶「行動」，行動的結果再回饋為新輸入，形成循環：

```
Thought → Action → Observation → Thought → Action → ...→ Final Answer
```

- **Thought（推理）**：CoT 內部推理，決定下一步要做什麼
- **Action（行動）**：呼叫外部工具（API、搜尋、程式碼執行）
- **Observation（觀察）**：工具回傳的結果，成為下一輪推理的輸入

## 為什麼重要

| 純 CoT | 純 Tool Use | ReAct |
|-------|------------|-------|
| 推理但資訊封閉 | 執行但無推理過程 | 推理指導行動，行動補充推理 |
| 幻覺風險高 | 不知何時停止 | 推理可見、可審計 |

ReAct 讓模型的「思考過程」可追蹤，大幅提升可解釋性與可除錯性。

## 典型工具組合

- **RAG**：搜尋知識庫取得相關文件 chunk
- **Web Search**：取得即時資訊
- **Code Execution**：計算、資料處理、驗證
- **API 呼叫**：外部服務（天氣、資料庫、日曆）

## ReAct 在 Agentic 架構中的位置

ReAct 是 Single Agent 的核心執行迴圈：

```
[目標輸入]
    ↓
[Thought] 分析任務，決定下一步工具
    ↓
[Action] 呼叫工具
    ↓
[Observation] 收到結果
    ↓（若未達目標，繼續循環）
[Thought] 根據新觀察重新推理
    ↓
[Final Answer]
```

Multi-agent 系統中，每個子 Agent 內部也採用 ReAct 執行單一子任務。

## 常見問題

- **無限循環**：需設置最大步數限制或停止條件
- **工具錯誤累積**：Observation 含錯誤資訊時，後續推理也會偏差
- **效率**：每次 Action 增加延遲，需評估是否值得

## 相關頁面

- [[LLM限制與解決方案]] — ReAct 解決幻覺與資訊封閉問題的脈絡
- [[Prompt Engineering進階]] — CoT 與 ReAct 完整 Prompt 實作
- [[AI Agent核心架構 Model+Context+Tools]] — ReAct 是 Tools 維度的核心執行模式
- [[LangGraph Agent工作流設計]] — LangGraph 以 ReAct 為基礎的 Graph-based Agent 實作
- [[RAG檢索增強生成實戰]] — ReAct 最常見的 Action 之一
