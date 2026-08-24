---
title: OpenSpec 三大角色
type: concept
tags: [openspec, sdd, context-anchor, contract-guardian, collaboration-middleware, ai-coding]
created: 2026-05-15
updated: 2026-05-15
sources: [github-forceinjection-openspec-practise]
---

# OpenSpec 三大角色

OpenSpec 在 AI 輔助開發中扮演三個相互獨立、缺一不可的角色。理解這三個角色，才能解釋為什麼「先寫規格再寫程式」不只是工程紀律，而是解決 AI 協作結構性問題的必要機制。

---

## 角色一：Context Anchor（情境錨點）

**問題：** LLM 沒有跨對話記憶。每次新對話，AI 不知道專案的技術棧、架構決策、已有的設計原則。開發者必須不斷重複解釋背景，且解釋的品質影響 AI 的建議品質。

**OpenSpec 的解法：** `openspec/config.yaml` 作為專案的**持久外部記憶**，在每次 AI 請求時自動注入。

```yaml
# openspec/config.yaml 示意
project:
  name: ecommerce-mini
  description: MVP 電商平台
  tech_stack:
    - Node.js 20 / Express
    - In-memory storage (v1), File persistence (v2)
  architecture_rules:
    - Domain layer must be pure (no HTTP dependencies)
    - Repository pattern for all data access
  constraints:
    - No external database in v1
```

**效果：** AI 每次提案時都「記得」這個專案用什麼技術、有哪些架構約束，不再重複犯「建議用 MongoDB」這類忽略現有決策的錯誤。

**與 [[CLAUDE.md撰寫最佳實踐]] 的關係：** CLAUDE.md 是 Claude Code 的 Context Anchor；config.yaml 是 OpenSpec 跨工具的 Context Anchor——兩者服務的目的相同，格式不同。

---

## 角色二：Contract Guardian（契約守護者）

**問題：** AI 輸出具有高度變異性（non-determinism）。同一個需求描述，不同時間或不同工具可能產生截然不同的 API 設計、資料模型、錯誤碼——導致客戶端破壞性變更。

**OpenSpec 的解法：** spec.md 中的 Scenario 定義了具體的**介面契約**，AI 的實作必須通過這些場景的測試。

```markdown
#### Scenario: Out-of-stock item cannot be ordered
Given a product with stock quantity 0
When a customer places an order for that product
Then the response status is 409 Conflict
And the response body contains error code "OUT_OF_STOCK"
```

**效果：**
- 無論底層實作改為 File-based 還是 Database，HTTP 契約保持不變
- Node.js 版和 Python 版產生相同的 409 Conflict 回應
- AI 沒有自由發揮 API 設計的空間——「契約」已定義

**語言無關性證明：** ForceInjection/OpenSpec-practise 用同一份 spec 驅動 Express 和 FastAPI 兩套實作，兩者通過完全相同的 E2E 測試場景，這正是 Contract Guardian 角色的直接展示。

---

## 角色三：Collaboration Middleware（協作中介層）

**問題：** 人類與 AI 之間的協作沒有標準協議。人類說「做個購物車功能」，AI 無法確認這句話涵蓋多少邊界案例、使用什麼技術、優先級是什麼——直到程式碼寫出來才發現偏差。

**OpenSpec 的解法：** 規格文件成為人類意圖與 AI 執行之間的**雙向交換格式**。

```
人類表達意圖（proposal.md: Why + Capabilities）
    ↓
AI 提案架構（design.md + specs/ 初稿）
    ↓
人類審查並確認（驗證 Scenario 是否準確）
    ↓
AI 實作（/opsx:apply）
    ↓
自動化測試驗證契約合規（Verify）
    ↓
結果回饋下一輪迭代（Archive + 新 Proposal）
```

**四階段開發週期的對應：**

| 階段 | 人類角色 | AI 角色 |
|------|---------|--------|
| Intent Alignment | 描述目標，審查 spec 初稿 | 生成 proposal + design + specs |
| Spec-Driven Implementation | 批准 tasks.md | 嚴格按 spec 實作程式碼 |
| Automated Verification | 設定驗收標準 | 生成並執行 E2E + 效能測試 |
| Production Evolution | 決定下一個 Proposal | 在現有架構上安全擴展 |

**關鍵洞察：** AI 不在「模糊問題空間」中運作，而是「在明確邊界內執行精確定義的任務」。這消除了 Vibe Coding 的隨機性，而不是消除 AI 的參與。

---

## 三角色的協同

```
config.yaml（Context Anchor）
    → 每次 AI 請求都知道專案背景

spec.md + Scenarios（Contract Guardian）
    → 每次 AI 實作都有可驗證的成功標準

proposal/design/tasks（Collaboration Middleware）
    → 每次人機協作都有結構化的交換格式
```

缺少任何一個角色：
- 無 Context Anchor → AI 重複忽略架構約束
- 無 Contract Guardian → AI 輸出變異性高，破壞性變更頻繁
- 無 Collaboration Middleware → 需求偏差在實作完成後才被發現（Wall 1）

---

## 相關頁面

- [[OpenSpec文件格式與驗證]] — 每個角色對應的具體文件格式
- [[OpenSpec工作流]] — 三大角色在 8 階段中的分佈
- [[Spec驅動開發]] — 三大角色解決的三道牆
- [[CLAUDE.md撰寫最佳實踐]] — Claude Code 生態中的 Context Anchor 實作
- [[Context Engineering最佳實踐]] — Context 管理的更廣泛工程視角
