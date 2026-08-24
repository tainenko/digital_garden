---
title: How to Write a Good Spec for AI Agents
type: source-summary
tags: [SDD, 規格寫作, AI Agent, 好規格, 邊界]
created: 2026-05-18
updated: 2026-05-18
sources: [addyosmani-good-spec-ai-agents]
---

# How to Write a Good Spec for AI Agents

## Origin
- **作者**：Addy Osmani（Google Chrome 工程主管）
- **日期**：2026-01-13
- **URL**：https://addyosmani.com/blog/good-spec/

## Key Takeaways

- **核心主張**：「好規格不只告訴 AI 要建什麼，而是幫助它自我修正並維持在安全邊界內。」
- **好規格的六大結構區域**：
  1. **Commands**：可執行的完整指令（含所有 flags）
  2. **Testing**：測試框架、覆蓋率期望
  3. **Project structure**：明確的目錄結構
  4. **Code style**：有具體範例的風格規範
  5. **Git workflow**：分支命名、commit 格式
  6. **Boundaries**：三層邊界定義（見下）
- **三層邊界（Three-Tier Boundaries）**：
  - ✅ **Always（永遠執行）**：AI 可安全自主執行的操作
  - ⚠️ **Ask first（先詢問）**：高影響力的變更，需人類確認才執行
  - 🚫 **Never（絕不執行）**：硬性停止（例：「絕不 commit secrets」）
- **模組化提示優於龐大 Context**：把大任務拆成專注的子任務，「指令的詛咒」（curse of instructions）顯示需求過多會讓 AI 表現下滑；大型規格需要目錄和摘要。
- **規格是持續演化的共同 artifact**：讓 AI 把初始 brief 擴展成完整規格，再在整個專案過程中持續修訂。
- **常見反模式**：
  - 規格過於模糊，AI 缺乏錨定點
  - Context 過大卻沒有摘要機制
  - 跳過人工 review，產出脆弱的「紙牌屋」程式碼
  - 忽略六大結構區域的某一項
  - 把快速原型等同於生產級工程

## Entities Mentioned
- Addy Osmani — Google Chrome 工程主管

## Concepts Mentioned
- [[好規格寫作原則]] — 本文核心
- [[Spec驅動開發]] — 規格寫作是 SDD 的基礎能力
- [[SDD適用邊界]] — 三層邊界是 SDD 的安全護欄

## Contradictions / Tensions
- Addy Osmani 強調人工 review 是必要的，但 spec-kit 的「15 分鐘」等宣傳暗示高度自動化——兩者對人類介入程度的期待有落差。

## Questions Raised
- 三層邊界如何隨專案成熟度動態調整？初期「Ask first」的操作，穩定後是否可轉為「Always」？
