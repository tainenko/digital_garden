---
title: OpenSpec 文件格式與驗證
type: concept
tags: [openspec, sdd, spec-format, validation, gherkin, delta-spec]
created: 2026-05-15
updated: 2026-05-26
sources: [github-forceinjection-openspec-practise, kaochenlong-openspec]
---

# OpenSpec 文件格式與驗證

OpenSpec 1.3.0 的文件格式規則與 `openspec validate` 的強制檢查機制。這是 [[OpenSpec工作流]] 的執行細節，也是 AI 輸出品質的約束邊界。

## 四份核心文件

每個 Change 包含四份文件，由 `/opsx:propose <description>` 一鍵生成初稿：

```
openspec/changes/<change-name>/
├── proposal.md   ← 強制格式
├── design.md     ← 無嚴格格式
├── specs/        ← 強制格式（capability 子目錄 + Delta Headers）
│   └── <capability>/spec.md
└── tasks.md      ← 無嚴格格式
```

---

## proposal.md 格式規範

**強制必備三個段落：**

```markdown
## Why
### Background
<專案背景與動機>

### Problem Statement
<具體問題描述>

### Alternatives
<評估過的替代方案與捨棄原因>

## What Changes
<本次變更新增/修改的資源與能力概覽>

## Capabilities
<能力列表——此段直接驅動 specs/ 子目錄的生成>
- Capability A
- Capability B
```

**關鍵原則：先寫 Why，再寫 What。** Capabilities 段落決定 specs/ 下會產生哪些子目錄。

---

## spec.md 格式規範（Delta Spec）

這是格式最嚴格的文件，`openspec validate` 的主要檢查對象。

**強制結構：Delta Header + Requirement + Scenario**

```markdown
## ADDED Requirements
（或 MODIFIED / REMOVED）

### Requirement: <需求標題>
**Priority**: P0 | P1 | P2
**Rationale**: <為什麼需要此需求>

#### Scenario: <場景標題>
Given <前置條件>
When <觸發動作>
Then <期望結果>

#### Scenario: <另一個場景>
Given ...
When ...
Then ...
```

**Delta Headers 必須完全符合以下三種之一：**
- `## ADDED Requirements`
- `## MODIFIED Requirements`
- `## REMOVED Requirements`

**每個 Requirement 至少需要一個 Scenario。**

---

## design.md 與 tasks.md

無嚴格格式要求，但有推薦結構：

**design.md 建議涵蓋：**
- Architecture（元件關係）
- Data Model
- API 介面定義
- Security 考量
- Deployment 策略

**tasks.md 建議格式：**
- 以 Milestone 分組
- 每個任務有 Definition of Done
- 可執行清單（checkbox 格式）

---

## openspec validate 驗證規則

**指令：** `openspec validate <change-name>`

**三類常見錯誤：**

| 錯誤類型 | 說明 | 修正方式 |
|---------|------|---------|
| Missing Delta headers | spec.md 缺少 `## ADDED/MODIFIED/REMOVED Requirements` | 加入對應的 Delta Header 段落 |
| Malformed requirement title | Requirement 標題未使用 `### Requirement:` 前綴 | 改為 `### Requirement: <標題>` |
| Missing scenario block | Requirement 下無 Gherkin 場景 | 至少加一個 `#### Scenario:` 並填入 Given/When/Then |

**另一個常見錯誤：** specs 文件放在 capability 子目錄以外的根層級——specs 必須依 capability 分目錄組織。

---

## Gherkin 場景撰寫原則

- **具體而非抽象**：`Given a product with stock quantity 0` 勝過 `Given a product is out of stock`
- **行為描述而非實作描述**：`Then the response status is 409 Conflict` 勝過 `Then the system calls rejectOrder()`
- **邊界條件納入**：超出庫存、重複訂單、非法輸入都應有對應場景
- **跨語言可驗證**：同一 Scenario 在 Node.js 和 Python 實作中都必須通過，這是語言無關性的具體保證

---

## 與 AI 協作的使用模式

```
/opsx:propose "新增購物車優惠券功能"
  → AI 生成 proposal.md + design.md + specs/ + tasks.md 初稿
  → 人工審查 Why / Capabilities / Scenario 是否準確
  → openspec validate <change-name>  # 強制格式檢查
  → 修正錯誤後進入 /opsx:apply
```

`config.yaml` 在此流程中作為 **Context Anchor**，自動注入技術棧與架構約束至每次 AI 請求，避免 AI 每次重新猜測專案背景（詳見 [[OpenSpec三大角色]]）。

---

## 目錄結構（v1.0，`openspec init` 後）

```
openspec/
├── config.yaml     ← 技術棧、架構慣例、約束（AI 的持久背景知識，v1.0 正式命名）
├── specs/          ← 當前系統規格的唯一真相源
└── changes/        ← 進行中的提案（含 archive/ 子目錄存放完成的變更）
```

Skills 整合檔案存放於 `.claude/skills/`（v1.0 從 slash commands 遷移至 Skills 系統）。

> **命名演進**：`config.yaml` 是 v1.0 的正式名稱。高見龍的 OpenSpec 入門文章（2026-01）提到的 `AGENTS.md` + `project.md` 為中間過渡版本命名，v1.0（2026-02）已統一為 `config.yaml`。

---

## CLI 指令完整表

| 指令 | 用途 |
|------|------|
| `openspec list` | 顯示活躍的 changes 或既有 specs |
| `openspec validate [name]` | 格式合規檢查（三類錯誤見下節） |
| `openspec show [name]` | 顯示指定提案的詳細資訊 |
| `openspec view` | 開啟互動式儀表板介面 |
| `openspec apply` | 觸發 AI 依規格實作（見 [[OpenSpec工作流]]）|
| `openspec archive` | 歸檔提案，delta spec 合回主規格 |

---

## 何時不需要 Proposal

以下情境可直接修改程式碼，**繞過 SDD 工作流**：

1. **Bug fix**：讓程式碼行為符合現有規格（規格本身沒錯，程式碼才是問題）
2. **錯字與格式調整**：文字修正、空白、縮排
3. **非破壞性依賴更新**：patch/minor 版本升級，不改變外部行為
4. **設定變更**：保留規格所描述行為的環境/配置調整
5. **補充測試**：為已記錄功能追加測試案例（不新增功能）

**核心判斷原則**：若這個改動不改變「系統應該做什麼」（specs 所描述的行為），就不需要 Proposal。

---

## 相關頁面

- [[OpenSpec三大角色]] — Context Anchor / Contract Guardian / Collaboration Middleware 的整體框架
- [[OpenSpec工作流]] — 8 階段流程與 Delta Spec System
- [[Spec驅動開發]] — SDD 方法論與三道牆
- [[DDD領域驅動設計]] — Bounded Context / Aggregate 到 spec.md 的映射
