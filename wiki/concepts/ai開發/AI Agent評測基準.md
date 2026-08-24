---
title: AI Agent 評測基準
type: concept
tags: [ai-agent, benchmark, evaluation, swe-bench, gaia, terminal-bench, osworld, elo]
created: 2026-05-07
updated: 2026-05-07
sources: [s09g-ai-agent-bootcamp]
---

# AI Agent 評測基準

評測 AI Agent 能力的標準化基準。來源：[[Bojie Li (s09g)]] 的「AI Agent 實戰營」Week 6。

---

## 主要基準對比

| 基準 | 環境 | 任務類型 | 難度 | 特點 |
|------|------|---------|------|------|
| **SWE-bench** | GitHub Issue | 程式碼修復 | 高 | 評測真實 Bug 修復能力 |
| **SWE-bench Lite** | GitHub Issue | 程式碼修復 | 中 | 300 個精選題，更快迭代 |
| **SWE-bench Verified** | GitHub Issue | 程式碼修復 | 高 | 人工驗證的子集 |
| **GAIA** | 各種工具 | 通用問答 | 三級 | 450+ 題，現實生活任務 |
| **Terminal-Bench** | 真實終端機 | 系統操作 | 高 | Docker 沙盒，~100 任務 |
| **OSWorld** | 完整 OS | 桌面操作 | 高 | 檔案管理、系統設定 |
| **Android World** | Android 模擬器 | App 操作 | 中高 | UI 互動、跨 App 任務 |
| **Tau2-bench** | 工具增強 | 複雜推理 | 高 | 計算、搜尋、數據處理三類 |

---

## SWE-bench

**評測目標**：LLM 能否解決真實 GitHub issue，生成有效 patch

**運作方式**：
1. 提供 GitHub issue 描述 + 程式碼庫
2. Agent 生成修復 patch
3. 自動跑測試驗證 patch 是否正確

**版本**：
- Full：2,294 個 issue
- Lite：300 個精選（更快、更便宜）
- Verified：人工確認可解的子集
- Multimodal：包含圖片、截圖的 issue

**意義**：目前最受重視的 Coding Agent 基準，Claude、GPT、Gemini 都以此比拼。

---

## GAIA

**評測目標**：下一代通用 AI 助理的基礎能力

**特點**：
- 450+ 個現實生活問題（不是合成題）
- 三個難度層（Level 1–3）
- Level 1：1 個工具步驟可解
- Level 3：需要多工具協作、複雜推理

**範例任務**：
- 「找出某篇論文的作者，並查詢他們最近的發表」
- 「計算某段 YouTube 影片的平均音量」

---

## Terminal-Bench

**評測目標**：Agent 在真實終端機環境中執行任務

**環境**：Docker 沙盒（隔離，可重置）

**任務類型**：
- 編譯並運行程式碼
- 訓練 ML 模型
- 搭建伺服器
- 系統管理操作

約 100 個任務，強調真實工程場景。

---

## ELO 評分系統

當多個 Agent 或模型難以用單一分數比較時，用 ELO 評分進行「頭對頭」相對比較：

```
Agent A vs Agent B → 比較兩者在同一任務的表現
根據勝負更新 ELO 分數
多輪比較後形成穩定排名
```

**優點**：比單一 benchmark 更能捕捉整體能力差異；適合評估不同版本的進步。

---

## 評測的局限

- **分佈轉移**：基準題目一旦公開，模型可能被針對性訓練
- **任務代表性**：SWE-bench 偏 Python，不代表所有語言/框架能力
- **成本問題**：完整 SWE-bench 跑一次需要數百美元 API 費用
- **作弊風險**：訓練資料可能包含基準答案

---

## 相關頁面

- [[AI Agent核心架構 Model+Context+Tools]] — Agent 整體架構
- [[Claude Code Headless模式與CICD]] — 自動化評測環境
- [[Bojie Li (s09g)]] — 課程來源
- [[語意相似度評分SSR]] — 「LLM 對齊人類判斷」的另一種評測（重現反應分布而非答對題）
- [[合成消費者調查模擬]] — LLM 作為合成受訪者的評估框架
