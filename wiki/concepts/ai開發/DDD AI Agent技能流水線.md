---
title: DDD AI Agent 技能流水線
type: concept
tags: [DDD, AI Agent, Skills, 流水線, 建模, ForceInjection]
created: 2026-05-25
updated: 2026-05-25
sources: [forceinjection-ddd-skills]
---

# DDD AI Agent 技能流水線

ForceInjection 設計的 9 技能 × 5 階段完整 DDD 建模系統，以 AI Agent Skill 形式封裝，解決現有 DDD 工具「各做各的」碎片化問題，提供**從模糊需求到可執行工程規格的閉環路徑**。

## 五階段架構

```
Stage I    Problem Space Discovery   ← 發散探索
Stage II   Strategic Modeling        ← 分析分解
Stage III  Tactical Modeling         ← 精確設計
Stage IV   Model Validation          ← 品質審查
Stage V    Specification Bridge      ← 轉換工程規格
```

## 9個技能清單

### Stage I：問題空間發現

| 技能 | 核心任務 | 輸出產物 |
|------|----------|----------|
| `ddd-scope` | 將模糊需求收斂為可執行的建模輸入 | 問題陳述、目標/非目標、約束、術語種子、風險清單 |
| `ddd-discover` | 協作式領域探索（事件風暴/領域故事） | 事件流、命令/事件候選清單、熱點、歧義點 |

### Stage II：戰略建模

| 技能 | 核心任務 | 輸出產物 |
|------|----------|----------|
| `ddd-subdomains` | 識別業務能力，分類子域 | 能力清單、Core/Supporting/Generic 三分類、核心域聲明 |
| `ddd-contexts` | 設計限界上下文與通用語言 | 上下文目錄、邊界 ADR、詞彙表 |
| `ddd-context-map` | 繪製上下文關係與整合策略 | 關係矩陣、整合模式（ACL/OHS/PL）、契約所有權、失敗模式 |

### Stage III：戰術建模

| 技能 | 核心任務 | 輸出產物 |
|------|----------|----------|
| `ddd-aggregates` | 從不變式設計聚合邊界 | 聚合目錄、不變式表、事務邊界 |
| `ddd-domain-interactions` | 設計建構塊間的協作機制 | 領域事件目錄、服務定義、Repository 介面、Factory 清單 |

### Stage IV：驗證

| 技能 | 核心任務 | 輸出產物 |
|------|----------|----------|
| `ddd-model-review` | 全域模型品質評估與反饋閉環 | 一致性分數、問題清單、回溯觸發條件 |

### Stage V：規格橋接

| 技能 | 核心任務 | 輸出產物 |
|------|----------|----------|
| `ddd-openspec-bridge` | 將 DDD 戰術產物轉換為 OpenSpec 格式 | OpenSpec Proposal / Design / Specs / Tasks |

## 產物流向圖

```
ddd-scope ──────────────────────────────────────────► ddd-discover
     │                                                      │
     └──────────────────────────────────────────────► ddd-subdomains
                                                            │
                                                      ddd-contexts
                                                            │
                                                      ddd-context-map
                                                            │
                                                      ddd-aggregates
                                                            │
                                                  ddd-domain-interactions
                                                            │
                                                      ddd-model-review ──► (回溯)
                                                            │
                                                    ddd-openspec-bridge
                                                            │
                                                       程式實作
```

## 非線性入口：5 種場景

| 場景 | 建議起始技能 | 前提條件 |
|------|------------|---------|
| 全新專案 | `ddd-scope` | 無 |
| 需求明確 | `ddd-discover` | 使用者提供 scope 上下文 |
| 已知子域 | `ddd-contexts` | 有子域分類 |
| 單一上下文深化 | `ddd-aggregates` | 有限界上下文定義 |
| 現有模型稽核 | `ddd-model-review` | 已有建模產物 |

## SKILL.md 契約格式

每個技能強制包含七節：

```
1. 觸發條件（Trigger Conditions）
2. 輸入要求（Input Requirements）
3. 執行步驟（5–7 steps）
4. 輸出產物（Outputs with structural requirements）
5. 驗證清單（Validation Checklist / Exit Criteria）
6. 回溯觸發條件（Backtrack Triggers）
7. 使用範例（Examples）
```

目的：使結構化輸出**可機器解析**，確保技能之間的產物可靠銜接。

## 完整範例：會議室預訂系統

### Stage I — ddd-scope 輸出
- 問題：「會議室資源缺乏統一管理；日程衝突降低效率」
- 目標：時段預訂、衝突偵測、打卡確認
- 非目標：設備維護、視訊會議排程
- 約束：最短 30 分鐘、每人每時段最多 1 個預訂
- 術語種子：Booking / Room / TimeSlot / CheckIn

### Stage II — ddd-subdomains 輸出
| 分類 | 子域 | 說明 |
|------|------|------|
| Core | 預訂與衝突管理 | 競爭差異點，業務核心 |
| Supporting | 會議室資源管理 | 必要但非差異化 |
| Generic | 身份與授權 | 可複用現有系統 |

### Stage III — ddd-aggregates 輸出
| 聚合 | 根實體 | 關鍵不變式 |
|------|--------|-----------|
| Booking | Booking | 同一 Room + TimeSlot 不得有兩個 Confirmed 預訂 |
| Room | Room | 必須有唯一 ID 和非零容量 |

### Stage IV — ddd-model-review 回溯範例
- `ConflictDetectionService` 需要跨聚合查詢 → Aggregate 得分 7/10
- **回溯觸發**：引入 `RoomSchedule` 聚合管理時段佔用
- 調整後：衝突偵測內化到聚合，得分升至 9/10，無新觸發

## 驗證結果（Cargo Shipping 盲跑）

- 加權總分：**85.8%（B+ 合格）**
- 回溯觸發測試：**3/3 通過**
- 已識別 3 個可改善項（主要集中在 ddd-discover 與 ddd-contexts）

## 相關概念

- [[DDD回溯觸發機制]] — 品質門禁詳細規則
- [[DDD領域驅動設計]] — 理論基礎（Eric Evans 戰略/戰術設計）
- [[OpenSpec工作流]] — Stage V 橋接目標
- [[Spec驅動開發]] — 上層方法論
- [[BDD行為驅動開發]] — Scenario 格式來源
- [[ForceInjection]] — 作者組織
