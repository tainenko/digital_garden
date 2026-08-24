---
title: DDD 回溯觸發機制
type: concept
tags: [DDD, AI Agent, 品質門禁, 回溯, 非線性, ForceInjection]
created: 2026-05-25
updated: 2026-05-25
sources: [forceinjection-ddd-skills]
---

# DDD 回溯觸發機制（Backtrack Triggers）

DDD AI Agent 技能流水線的核心品質設計。與傳統的線性流程不同，回溯觸發機制允許下游技能**主動偵測品質問題並觸發上游技能重跑**，使 AI Agent 建模系統能自我修正而非盲目推進。

## 核心設計原則

1. **品質門禁優先於進度**：寧可退回重做，不推進缺陷模型
2. **條件明確、可量化**：每條觸發規則有可偵測的具體指標（百分比、數量等）
3. **防死迴圈**：同一回溯路徑最多執行 3 次；第 3 次觸發升級為「需人工架構決策」
4. **精確路由**：退回的目標技能根據問題性質決定，不是統一退到起點

## 完整回溯觸發條件表

| 偵測技能 | 觸發條件 | 退回目標 | 原因 |
|----------|----------|----------|------|
| ⑧ `ddd-model-review` | 聚合邊界與上下文邊界矛盾 | `ddd-contexts` | 一致性需求橫跨上下文邊界 |
| ⑧ `ddd-model-review` | 術語衝突率 > 20% | `ddd-contexts` | 通用語言定義不足 |
| ⑧ `ddd-model-review` | Invariant 表達率 < 60% | `ddd-aggregates` | 聚合是資料容器而非行為邊界 |
| ⑦ `ddd-domain-interactions` | 事件攜帶其他聚合私有資料 | `ddd-aggregates` | 事件 schema 無法乾淨設計；需邊界調整 |
| ⑥ `ddd-aggregates` | 不變式橫跨多個上下文 | `ddd-contexts` | 一致性需求被分裂到上下文邊界兩側 |
| ⑤ `ddd-context-map` | 循環依賴，或單一上下文有 >3 個上游 | `ddd-subdomains` / `ddd-contexts` | God Context 或子域分類錯誤 |
| ④ `ddd-contexts` | >5 個無法調和的跨上下文術語衝突 | `ddd-discover` | 領域理解不足，需重新發現 |
| ③ `ddd-subdomains` | 無法區分核心域與支撐域 | `ddd-scope` | 業務價值主張不清晰 |

## 最常觸發的 3 條規則

### 規則 1：Invariant 表達率 < 60%（退回 `ddd-aggregates`）

**偵測者**：`ddd-model-review`  
**含義**：設計出的聚合大多數是「資料結構」而非「業務不變式守護者」  
**症狀**：聚合的 `invariants` 欄位為空或過於簡單，所有行為都委託給 Service  
**修正方向**：回到 `ddd-aggregates`，以不變式為出發點重新劃定邊界

範例（會議室案例）：
```
觸發前：ConflictDetectionService 跨聚合查詢 → Aggregate 得分 7/10
觸發後：引入 RoomSchedule 聚合，內化衝突偵測
重驗後：Aggregate 得分 9/10，觸發消除
```

### 規則 2：術語衝突率 > 20%（退回 `ddd-contexts`）

**偵測者**：`ddd-model-review`  
**含義**：多個上下文對同一術語有不同定義，且未建立翻譯層  
**症狀**：「Order」在訂單上下文 vs 履行上下文語義截然不同，但未設 ACL  
**修正方向**：回到 `ddd-contexts`，強化語言邊界定義與 ACL 設計

### 規則 3：God Context 偵測（退回 `ddd-subdomains`）

**偵測者**：`ddd-context-map`  
**條件**：單一上下文有 >3 個上游依賴，或多個上下文形成循環依賴  
**含義**：上下文邊界設計違反了「高內聚、低耦合」原則  
**修正方向**：回到 `ddd-subdomains` 重新分類，或回到 `ddd-contexts` 拆分

## 防無限迴圈設計

```
第 1 次觸發 → 正常退回並重跑目標技能
第 2 次觸發 → 退回並附帶上次失敗的詳細診斷資訊
第 3 次觸發 → 不再退回，標記為「需人工架構決策」，輸出候選決策清單
```

**設計意圖**：AI Agent 可以重試，但不應無限重試同一個問題。第 3 次觸發表示問題超出了技能系統的覆蓋範圍，需要人類架構師介入判斷。

## 與其他品質機制的對比

| 機制 | 觸發方式 | 覆蓋範圍 |
|------|----------|----------|
| DDD 回溯觸發 | 定量指標自動觸發，退回上游重跑 | 模型一致性、邊界完整性 |
| [[OpenSpec工作流]] 的 validate | 格式/結構驗證，不通過則報錯 | 文件格式合規性 |
| TDD Red-Green-Refactor | 測試失敗觸發修改，前向迭代 | 程式碼行為正確性 |
| [[BDD行為驅動開發]] 的 Scenario | 描述驗收標準，人工確認 | 業務需求對齊 |

## 相關概念

- [[DDD AI Agent技能流水線]] — 回溯機制所在的整體系統
- [[DDD領域驅動設計]] — 不變式、限界上下文等核心概念
- [[OpenSpec文件格式與驗證]] — OpenSpec 自己的驗證機制
- [[Superpowers技能框架]] — 另一個以品質門禁為核心的 Skill 系統（TDD + Code Review）
- [[ForceInjection]] — 機制設計者
