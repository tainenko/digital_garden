---
title: Context Engineering 最佳實踐
type: concept
tags: [context-engineering, kv-cache, memory, compression, rag, agent, token-optimization]
created: 2026-05-07
updated: 2026-05-07
sources: [s09g-ai-agent-bootcamp]
---

# Context Engineering 最佳實踐

Context Engineering 是 AI Agent 開發的核心工程挑戰：在有限的 context window 內，讓模型獲得最有用的資訊，同時最小化延遲與成本。

來源：[[Bojie Li (s09g)]] 的「AI Agent 實戰營」Week 2–3。

---

## KV Cache 優化

### 為什麼 KV Cache 至關重要

LLM 推理時會計算每個 token 的 Key-Value 對，可以快取重複部分。KV Cache 命中 → 省略重複計算 → 延遲與成本大幅下降。

**不良 context 設計會摧毀 KV Cache**：
- 每次請求在 context 開頭插入動態時間戳 → Cache Miss
- 每輪對話改變系統提示順序 → Cache Miss
- 把用戶 ID 等動態資訊放在靜態系統提示之前 → Cache Miss

**KV Cache 友好的設計原則**：
```
[靜態部分] → [較穩定部分] → [動態部分]
系統提示    → 對話歷史     → 當前用戶輸入
```

---

## Context 壓縮策略

當對話過長時，選擇性保留資訊：

| 策略 | 方法 | 適用場景 |
|------|------|---------|
| **摘要壓縮** | 讓 LLM 摘要舊對話 | 長期對話 |
| **關鍵資訊提取** | 提取重要事實、決策、結果 | 任務型對話 |
| **語意壓縮** | 向量相似度篩選最相關片段 | 知識庫整合 |
| **滑動視窗** | 只保留最近 N 輪 | 即時對話 |

---

## 用戶記憶系統

讓 Agent 跨 session 記住用戶偏好：

**兩層記憶架構**：
1. **短期記憶**：當前 session 的對話歷史
2. **長期記憶**：結構化 JSON 卡片（用戶偏好、重要事件、行為模式）

**Contextual Retrieval 技術**（Anthropic 提出）：
- 為知識庫每個片段生成「上下文感知前置摘要」
- 讓檢索時能理解片段在原文中的語義位置
- 效果：檢索失敗率降低 **49–67%**

---

## Prompt Engineering 量化

常見但未被量化的 prompt 設計選擇，實際影響 Agent 任務完成率：

| 變數 | 影響 |
|------|------|
| 語氣（禮貌 vs 直接）| 任務完成率差異顯著 |
| 指令組織方式（清單 vs 段落）| 影響遵循率 |
| 工具描述詳細程度 | 影響工具選擇準確率 |
| Few-shot 範例數量 | 邊際效益遞減點不同 |

---

## Log 脫敏（Log Sanitization）

Agent 在執行過程中會記錄大量 log，包含敏感資訊（API Key、個人資料）。

**智能脫敏原則**：
- 識別敏感模式（信用卡號、姓名、Token）→ 替換為占位符
- 保留除錯所需的上下文結構
- 不能過度脫敏導致 log 失去診斷價值

---

## 相關頁面

- [[AI Agent核心架構 Model+Context+Tools]] — 整體框架
- [[RAG檢索增強生成實戰]] — 知識庫與檢索
- [[Claude Code 記憶體系統深度指南]] — Claude Code 的記憶系統
- [[CLAUDE.md撰寫最佳實踐]] — prompt 工程的靜態面
- [[Bojie Li (s09g)]] — 課程來源
