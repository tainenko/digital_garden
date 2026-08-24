---
title: LLM 限制與解決方案
type: concept
tags: [llm, hallucination, rag, fine-tuning, context-window, knowledge-cutoff]
created: 2026-05-15
updated: 2026-05-15
sources: [stanford-agentic-ai-youtube-summary]
---

# LLM 限制與解決方案

從 Stanford Agentic AI 課程整理的系統性框架：五大基礎限制及其對應解法。

## 五大限制 × 解決方案對照

| 限制 | 說明 | 解決方案 |
|------|------|---------|
| **幻覺（Hallucination）** | 模型以高信心輸出錯誤資訊，沒有內建的事實核查機制 | Grounding（提供可信上下文）、RAG、引用來源要求 |
| **知識截止日（Knowledge Cutoff）** | 只知道訓練資料截止前的事件，無法感知最新資訊 | RAG（動態注入最新文件）、Tool Use（Web Search）|
| **無法引用來源** | 無法對輸出追溯依據，難以驗證 | RAG（附帶 source chunks）、結構化輸出要求 |
| **無法存取私有資料** | 訓練資料不含企業內部文件、資料庫、程式碼庫 | RAG（私有知識庫向量化）、Fine-Tuning（固化知識）|
| **Context 視窗有限** | 無法同時處理超長文件或極長對話歷史 | Chunking + 摘要壓縮、Memory 系統、Contextual Retrieval |

## 進化路徑

```
Base LLM
  └── Prompting 優化（CoT / Few-Shot / 任務分解）
        └── RAG（外部知識注入）
              └── Tool Use（動態外部呼叫）
                    └── ReAct（推理+行動循環）
                          └── Single Agent
                                └── Multi-agent Collaboration
```

每一層都在解決前一層的殘餘限制，而非替代前者。

## Prompting 六大策略

解決「模型能力未充分激發」的問題，不需改動模型本身：

1. **明確指令** — 具體說明任務，減少模型猜測
2. **Few-Shot 範例** — 示範期望的輸出格式與風格
3. **Chain-of-Thought** — 讓模型先推理再回答
4. **任務分解** — 複雜任務拆分為多個簡單子任務
5. **上下文參考資料** — 提供可信文件防止幻覺
6. **系統性評估** — 持續衡量並迭代 Prompt 效果

## 限制的殘餘問題

- RAG 的 retrieve 品質決定答案品質：若相似度搜尋未找到相關 chunk，仍會幻覺
- Fine-Tuning 固化知識但無法即時更新；適合穩定領域知識，不適合時效性資料
- Context 視窗擴大（如 Gemini 2M tokens）緩解但未根除長文處理問題

## 相關頁面

- [[ReAct Pattern]] — 從 Prompting 跨入 Agentic 的橋樑
- [[RAG檢索增強生成實戰]] — RAG 完整工程實作
- [[Prompt Engineering進階]] — CoT、Few-Shot、ReAct 完整技巧
- [[Context Engineering最佳實踐]] — Context 視窗工程應對
- [[AI Agent核心架構 Model+Context+Tools]] — Agentic 系統整體設計
- [[語意相似度評分SSR]] — 數值分布失真（直接打分塌向中間值）的具體案例與緩解法
