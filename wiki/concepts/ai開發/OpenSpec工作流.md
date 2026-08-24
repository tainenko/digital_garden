---
title: OpenSpec 工作流
type: concept
tags: [openspec, workflow, spec-driven-development, ai-coding]
created: 2026-04-28
updated: 2026-05-26
sources: [openspec-superpowers-multi-source, kaochenlong-spectra-openspec]
---

# OpenSpec 工作流

[[OpenSpec]] 的完整開發週期分為 8 個階段，確保每個變更從探索到歸檔都有完整記錄。

## 8 階段流程

```
explore → proposal → design → specs → tasks → apply → verify → archive
```

### 1. Explore（探索）
- 與 AI 一起理解現有 codebase 或問題空間
- 指令：`openspec explore`
- 輸出：理解報告、潛在方向

### 2. Proposal（提案）
- 定義「要改什麼」，生成四份文件：
  - `proposal.md`：問題、目標、邊界
  - Behavioral specs：given/when/then 行為描述
  - `design.md`：技術選型與架構決策
  - Task checklist：可執行任務清單
- 指令：`openspec propose`
- **關鍵**：在這階段與 AI 確認方向，避免後期返工

### 3. Design（設計）
- 細化技術選型，回答「怎麼做」
- 包含 API 介面、資料模型、邊界條件

### 4. Specs（規格）
- 將 design 轉為正式的行為規格
- 為後續測試提供基礎

### 5. Tasks（任務拆分）
- 將規格分解為具體、可執行的小任務
- 每個任務有明確的完成標準

### 6. Apply（實作）
- AI 逐任務實作，指令：`openspec apply`
- 若同時使用 [[Superpowers]]，此階段會觸發 TDD 流程

### 7. Verify（驗證）
- 確認實作符合 behavioral specs
- 手動測試 + 自動化測試

### 8. Archive（歸檔）
- **最關鍵、最常被跳過的步驟**
- 指令：`openspec archive` 或 `/opsx:archive`
- 把 delta spec 同步回主規格，保存決策歷史
- 不執行此步驟 → 設計理由消失（Wall 3）

## Delta Spec System

OpenSpec 的核心機制：

```
主規格（main spec）
    ↓
Delta Spec（只描述這次變更）
    ↓ 審核通過後
    ↓ openspec archive
主規格（更新版）+ 歷史決策記錄
```

優點：可查詢「六個月前為什麼選這個技術」。

## DDD × OpenSpec 映射

[[DDD領域驅動設計]] 概念可直接對應至 OpenSpec 文件結構：

| DDD 概念 | OpenSpec 對應 |
|---------|-------------|
| Bounded Context | `specs/` 子目錄（每個 Capability 一個目錄）|
| Domain Service / Command | `### Requirement:` 段落 |
| Aggregate Behavior | `#### Scenario:` Gherkin 場景 |
| Application Service | `design.md` Technical Design 段落 |
| Tactical Task | `tasks.md` 可執行任務項目 |

這個映射讓有 DDD 背景的工程師能直接以領域語言填寫 OpenSpec 文件。

## `/opsx:propose` 一鍵生成（v1.3 舊版）

OpenSpec 1.3.0 的 `/opsx:propose <description>` 單一指令同時產出四份文件初稿：

```
/opsx:propose "新增購物車優惠券功能"
  → proposal.md（Why / What Changes / Capabilities）
  → design.md（架構、API、資料模型）
  → specs/<capability>/spec.md（Delta Headers + Requirement + Scenario）
  → tasks.md（Milestone 分組任務清單）
```

人工審查這四份初稿後，執行 `openspec validate <change-name>` 確認格式合規，再進入 Apply 階段。詳細文件格式規範見 [[OpenSpec文件格式與驗證]]。

---

## v1.0 新指令系統（2026-02）

OpenSpec 1.0 從線性工作流演進為**非線性、狀態追蹤**架構，指令前綴改為 `/opsx:`。

### 核心設計改變

| 面向 | 舊版（≤1.3） | v1.0 |
|------|------------|------|
| 執行順序 | 強制線性 | 任意順序，系統維護狀態 |
| 文件生成 | 一次全部 | 依 DAG 依賴逐步或快進 |
| 配置檔 | `project.md` | `config.yaml` |
| 指令前綴 | `/openspec:` | `/opsx:` |
| 整合方式 | Slash commands | Skills（`.claude/skills/`）|

### 新指令速查

| 指令 | 說明 |
|------|------|
| `/opsx:explore` | 腦力激盪探索，**不**生成文件 |
| `/opsx:new` | 建立提案（取代舊版 `/openspec:proposal`）|
| `/opsx:continue` | 依 DAG 依賴關係逐一生成下一份文件 |
| `/opsx:ff` | Fast-forward：一次生成所有尚未產出的文件 |
| `/opsx:apply` | AI 實作（同舊版語意）|
| `/opsx:archive` | 歸檔，delta spec 合回主規格（同舊版語意）|

### DAG 文件依賴關係

```
proposal.md
    ↓
spec.md（依賴 proposal.md）
design.md（依賴 proposal.md）
    ↓
tasks.md（依賴 spec.md + design.md）
```

`/opsx:continue` 根據此 DAG 拓撲排序決定下一步；`/opsx:ff` 快進到全部完成。

### 即時進度追蹤

每次執行前，系統自動讀取 `tasks.md` 計算完成率，顯示如「3/5 tasks complete」，不再仰賴 AI 自行判斷。

## 與 Superpowers 整合

兩者不自動串聯，需在 CLAUDE.md 明確宣告：

```markdown
# CLAUDE.md 整合範例
在 openspec apply 階段，強制執行：
- 先寫失敗測試（RED）
- 實作通過（GREEN）
- 重構（REFACTOR）
- 自動 code review
```

## 相關頁面

- [[OpenSpec]] — 框架整體介紹與 v1.0 變化摘要
- [[Spectra]] — v1.0 配套桌面 GUI，視覺化呈現此工作流的 specs/changes/tasks
- [[Superpowers技能框架]] — apply 階段的程式碼品質執行
- [[Spec驅動開發]] — 方法論背景
- [[DDD AI Agent技能流水線]] — Stage V `ddd-openspec-bridge` 將 DDD 戰術產物對接此工作流的 Proposal/Design/Specs/Tasks
