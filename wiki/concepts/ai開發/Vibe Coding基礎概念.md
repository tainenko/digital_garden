---
title: Vibe Coding 基礎概念
type: concept
tags: [vibe-coding, ai-coding, workflow, definition]
created: 2026-04-27
updated: 2026-04-27
sources: [abmedia-vibe-coding-complete-guide-2026, wikipedia-vibe-coding, airabbi-vibe-coding-intro]
---

# Vibe Coding 基礎概念

**Vibe Coding** 是一種 AI 輔助的軟體開發方式：開發者用自然語言描述需求，AI 生成程式碼，開發者的角色從「寫手」變成「引導者」。

## 起源

- **提出者**: [[Andrej Karpathy]]，2025 年 2 月
- **原始精神**: 「完全給 vibes，不看 code diff，直接接受 AI 的所有修改」
- **建立在**: 他 2023 年的預言——「最熱門的新程式語言是英文」

## 與傳統開發的對比

| 面向 | 傳統開發 | Vibe Coding |
|------|---------|-------------|
| 輸入 | 逐行寫程式碼 | 自然語言描述 |
| 開發者角色 | 實作者 | 引導者 + 驗證者 |
| 偵錯方式 | 自己讀 code 找問題 | 貼 error message 給 AI |
| 進入門檻 | 需要程式語言知識 | 不需要，但批判性思維仍重要 |
| 速度 | 慢但精確 | 快但需要迭代驗證 |

## 核心工作流（四步驟）

```
1. 描述需求  →  用自然語言說清楚「我要做什麼」
2. AI 生成   →  模型產出多檔案程式碼、處理相依套件
3. 測試驗證  →  執行並觀察結果是否符合預期
4. 迭代修正  →  描述問題，AI 根據 feedback 修正
```

## 適合對象

- **有程式基礎的開發者**：加速重複性工作、快速上手陌生框架
- **非技術創辦人**：建立 MVP 而不需聘請工程師
- **設計師 / 產品經理**：製作可互動的原型取代靜態設計稿
- **一般工作者**：自動化日常任務（報表、資料處理、排程寄信）

## 適合 vs 不適合的情境

| 適合 | 不適合 |
|------|--------|
| 快速原型、MVP | 金融 / 認證等安全敏感系統 |
| 內部工具、一次性腳本 | 高效能需求核心路徑 |
| 學習新框架 | 需要長期維護的核心基礎設施 |
| 個人 side project | 複雜多人協作架構 |

## 市場現況（2026）

- 92% 美國開發者已採用某種 AI 輔助編程
- Y Combinator 2025 Winter：25% 新創的程式碼 95%+ 由 AI 生成
- 40% 新 SaaS MVP 主要透過 Vibe Coding 建立
- AI coding 工具市場規模估計 85 億美元

## 相關頁面

- [[Vibe Coding工具比較]] — 主流工具的選擇指南
- [[Vibe Coding風險與限制]] — 安全性、技術債、技能退化
- [[Vibe Coding開源生態衝擊]] — 對開源社群的影響
- [[Cursor]] — 最主流的 Vibe Coding IDE
- [[Andrej Karpathy]] — 概念提出者
