---
title: AI Agent 實戰營（Bojie Li / 圖靈社區）
type: source-summary
tags: [ai-agent, context-engineering, rag, mcp, sft, rlhf, multimodal, multi-agent, 實戰, 課程]
created: 2026-05-07
updated: 2026-05-07
sources: [s09g-ai-agent-bootcamp]
---

# AI Agent 實戰營（Bojie Li / 圖靈社區）

## Origin

- **標題**：AI Agent 實戰營（《AI Agent 实战营》）
- **作者**：Bojie Li（李博杰，s09g）— Chief Scientist, Pine AI
- **來源**：
  - GitHub: https://github.com/s09g/ai-agent-book-projects
  - 課程頁: https://01.me/2025/08/ai-agent-bootcamp/
  - 出版合作：圖靈社區（Turing Community）
- **擷取日期**：2026-05-07

## Key Takeaways

- **核心框架**：**Agent = Model + Context + Tools**
  - Model：決策大腦（理解與推理）
  - Context：作業系統（指令、歷史、推理鏈、工具記錄）
  - Tools：雙手（感知環境、執行操作、呼叫外部）
- **LLM 樣本效率**：LLM 透過先驗知識比傳統 RL（Q-learning）高出 **250–400×** 的樣本效率
- **KV Cache 是成本瓶頸**：不良 context 設計會摧毀 KV Cache 效率，造成延遲/成本暴增
- **Contextual Retrieval 效果**：Anthropic 技術，為檢索文本加前置摘要，讓檢索失敗率降低 **49–67%**
- **Coding Agent 純 Python 17 工具**：Week 5 實作生產級 Coding Agent，不依賴 CLI，完整工具鏈
- **Agentic RAG vs 非 Agentic RAG**：ReAct 驅動的迭代檢索顯著提升複雜法律問答準確度
- **AdaptThink**：讓推理模型自適應選擇 Thinking vs NoThinking，推理成本降低 **45–69%**，準確率提升
- **SFT vs RL**：Week 7 系統比較兩種後訓練方式在不同任務上的強弱與適用場景
- **Multi-Agent 直連**：Week 9 雙 Agent 架構（語音 + 電腦操作）透過 WebSocket 直接通訊，無需協調者
- **適用 API 平台**：Kimi（Moonshot）、SiliconFlow（開源模型）、火山引擎（豆包）、OpenRouter（國際模型）

## 九週課程摘要

| 週次 | 主題 | 核心項目 |
|------|------|---------|
| Week 1 | Agent 基礎 | Web Search Agent、RL vs LLM 樣本效率對比、Context Ablation |
| Week 2 | Context Engineering | KV Cache 優化、Context 壓縮、用戶記憶系統、Log 脫敏 |
| Week 3 | 知識庫與檢索 | Dense/Sparse Embedding、混合檢索管道、Agentic RAG、Contextual Retrieval、GraphRAG/RAPTOR |
| Week 4 | 工具生態 | MCP Server（感知/執行/協作）、事件驅動 Agent、非同步 Agent |
| Week 5 | Coding Agent | 純 Python 17 工具 Coding Agent（生產級）|
| Week 6 | 評測基準 | Terminal-Bench、SWE-bench、GAIA、OSWorld、Android World、ELO 系統 |
| Week 7 | 模型後訓練 | AdaptThink、ReTool（DAPO）、SFT vs RL、verl 框架、17 個訓練項目 |
| Week 8 | 多模態互動 | 實時語音對話（多 Provider）、Browser Use 自動化、Claude Quickstarts |
| Week 9 | 多 Agent 協作 | 雙 Agent（語音 + 電腦操作）WebSocket 直連架構 |

## Concepts Mentioned

- [[AI Agent核心架構 Model+Context+Tools]] — 核心框架詳解
- [[Context Engineering最佳實踐]] — Week 2 深度展開
- [[AI Agent評測基準]] — Week 6 詳解
- [[RAG檢索增強生成實戰]] — Week 3 擴充
- [[Claude MCP 伺服器整合指南]] — Week 4 工具生態對應
- [[Bojie Li (s09g)]] — 作者實體

## Contradictions/Tensions

- Week 7 AdaptThink 主張「選擇性推理」（有些題不需要長思維鏈）——與「鏈式思考越長越好」的傳統觀點有張力
- Week 5 Coding Agent 強調純 Python 不依賴 CLI，但 Claude Code（Week 8 有提及）本身就是 CLI 工具——兩種路徑並存，各有適用場景

## Questions Raised

- Pine AI 的「電話談判 Agent」具體產品形態？
- AdaptThink 的選擇邏輯如何判斷「何時不需要思考」？
- verl 框架（Week 7）與其他 RLHF 框架（TRL、OpenRLHF）相比的優勢？
