---
title: 行為面試CARL法
type: concept
tags: [behavioral-interview, CARL, STAR, leadership, staff-engineer, principal-engineer]
created: 2026-04-21
updated: 2026-04-21
sources: [behavioral-interview-senior-eng-leadership]
---

# 行為面試CARL法

Senior/Staff/Principal 面試的行為輪次，決定的不只是「過關」，而是你的 **leveling**。

---

## CARL vs STAR

| | CARL（推薦）| STAR（傳統）|
|-|-----------|-----------|
| Context | 背景 + **重要性** | Situation |
| Actions | 複數步驟（你做了什麼）| Task + Action |
| Results | 可量化的改變 | Result |
| **Learnings** | **可遷移的洞察** | ❌ 無此步驟 |

**為什麼 Learnings 關鍵？**  
L6+ 面試官在篩選的是：你能否從經驗中提取原則，並在新情境中複用。

> 「This taught me to front-load alignment work with stakeholders before committing to an architecture...」——這句話讓面試官知道他在跟一個能反思的 principal 說話。

---

## 三步準備法則：Decode → Select → Deliver

### Decode（解碼問題真正的意圖）

不同問法，測的是同一個訊號：

| 問法 | 真正在測什麼 |
|------|------------|
| 「說一個你和 manager 意見不合的例子」 | 如何處理 authority figures |
| 「說一個你失敗的例子」 | 自我認知 + 成長心態 |
| 「說一個你影響了非直屬成員的例子」 | 影響力不靠授權 |
| 「說一個模糊的專案最後如何推進的例子」 | 模糊環境的決策力 |

### Select（選最大 scope 的故事）

優先順序：
1. **Scope（最關鍵）**：跨越多個 team、持續多年、有明確業務影響
2. Relevance：自然對應問題訊號
3. Uniqueness：同一場面試中不重複故事
4. Recency：近期優先，但特別精彩的舊故事可接受

### Deliver（可重現的行為模式）

不要背稿，要讓面試官看到 **repeatable behaviors**——你在類似情況下會怎麼做的預測。

---

## 故事庫建立

### 核心故事（3-4 個，可塑形）

| 故事類型 | 展示的訊號 |
|---------|----------|
| 多年跨 team 重大技術決策 | 長期 ownership + org-level impact |
| 有業務影響的技術重構 | 技術與商業的橋接能力 |
| 優先排序衝突解決（eng vs product） | 溝通 + alignment 能力 |
| 在關鍵模糊中推進的決策 | 模糊環境中的方向感 |

### 補充覆蓋面向

每個面向準備 1-2 個例子（可從核心故事延伸）：
- **Scope** — 你承擔過多大的事
- **Ownership** — 你如何 take accountability
- **Communication** — 說服不同背景的 stakeholders
- **Growth** — 你讓周圍的人如何成長
- **Ambiguity** — 資訊不完整時如何決策
- **Perseverance** — 遇到巨大阻礙時如何堅持
- **Leadership** — 不靠授權的影響力

---

## CARL 故事範本

```
Context:
2023年Q2，我們的 checkout service p99 latency 從 200ms 飆到 2s，
影響 3 個 product teams，預估每天損失 $X 轉換率。

Actions:
1. 用 pprof 定位到 hot path 有大量 GC pressure
2. 說服 3 個 team 的 TL 暫停 roadmap 一週專注優化
3. 引入 sync.Pool + pre-allocation，重新設計 request handler
4. 建立 p99 SLO dashboard，讓所有 team 可見

Results:
p99 latency 降回 180ms，轉換率回升，建立了跨 team 的 latency
SLO 標準，後來被 5 個其他 service 採用。

Learnings:
跨 team 影響需要先讓數據可見（shared dashboard），再談改動——
用客觀數據建立 urgency 比靠個人說服有效 10 倍。
```

---

## 常見錯誤

- ❌ 說「我們做了 X」——用「我做了 X，因為 Y，但需要協調 A、B、C」
- ❌ 只說 team-level 影響——L6 需要跨 org 影響
- ❌ 沒有 Learnings——讓故事停在結果，而非反思
- ❌ 背太熟——聽起來像念稿而非真實經歷

---

## 相關頁面
- [[Principal工程師面試框架]] — 面試結構與 leveling 邏輯
- [[golang-principal-interview|Golang Principal Engineer 面試完整指南]]
