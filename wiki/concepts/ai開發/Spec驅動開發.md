---
title: Spec 驅動開發（SDD）
type: concept
tags: [spec-driven-development, sdd, openspec, superpowers, ai-coding, vibe-coding, spec-kit]
created: 2026-04-28
updated: 2026-05-26
sources: [openspec-superpowers-multi-source, xiediandongxiba-github-spec-kit-sdd, scott-hsiao-openspec-vocus]
---

# Spec 驅動開發（SDD）

Spec-Driven Development（SDD）是一種在 AI 動手寫程式之前，先用結構化規格文件定義「要做什麼」與「為什麼這樣做」的開發方法論。是對純 [[Vibe Coding基礎概念]] 的「成熟化」演進。

## 解決的三道牆

| 問題 | 描述 | 解法工具 |
|------|------|---------|
| **Wall 1** 需求錯位 | AI 在 code review 前就做了錯誤的功能 | OpenSpec（propose 先確認）|
| **Wall 2** 缺工程紀律 | 沒有測試、沒有 branch、沒有 code review | [[Superpowers]] |
| **Wall 3** 決策消失 | 技術決策在每次迭代後蒸發，下次 AI 不知道為何這樣設計 | [[OpenSpec]] Delta/Archive |

## 核心原則

1. **先寫規格，再寫程式**：讓 AI 理解「行為期望」而非「實作步驟」
2. **行為描述 > 偽程式碼**：規格寫 "when user submits form, system validates email format"，不寫 `validateEmail(email)`
3. **歸檔決策**：為什麼選 PostgreSQL 而非 MongoDB？這些理由必須記錄在案

## 主要框架

### 三工具規模定位（SDD 工具選擇速查）

| 框架 | 適用規模 | 核心特色 |
|------|---------|---------|
| [[BMAD]] | 大型組織 | 詳細流程、角色分工明確、適合已有敏捷體制的團隊 |
| [[GitHub-spec-kit]] | 0→1 新專案 | 官方工具鏈、三步工作流、constitution.md 項目憲法 |
| [[OpenSpec]] | 成長期 1→100 | 輕量、Brownfield-first、SOHO/中小型、決策追溯 |

### 完整工具清單

| 框架 | 重點 | 適用情境 |
|------|------|---------|
| [[GitHub-spec-kit]] | 官方工具鏈、三步工作流、項目憲法 | 0→1 新產品、企業級、多團隊協作 |
| [[OpenSpec]] | 變更管理、決策追溯、Delta Spec | 需要合規記錄、跨平台、長期維護 |
| [[BMAD]] | 詳細流程、角色分工 | 大型組織、已有敏捷流程的團隊 |
| [[Superpowers]] | 程式碼品質執行、強制 TDD | Claude Code 用戶、需要 code review |
| GSD（Get Shit Done） | 元提示 + 情境工程 | 快速原型、個人開發 |

## GitHub spec-kit vs OpenSpec 對照

| 面向 | [[GitHub-spec-kit]] | [[OpenSpec]] |
|------|---------------------|--------------|
| 來源 | GitHub 官方 | Fission-AI 社群 |
| 工作流步驟 | 3 步（/specify → /plan → /tasks） | 8 步（explore → … → archive） |
| 核心文件 | spec.md + plan.md + tasks.md | proposal.md + spec.md（Delta Headers）|
| 架構約束 | constitution.md 項目憲法 | config.yaml Context Anchor |
| AI 支援 | Claude / Copilot / Gemini / Cursor | Claude Code 為主 |
| 決策追溯 | 改動先改規範（單向） | Delta Spec + Archive（雙向可追溯）|
| 適用邊界 | 明確標示不適合 Hotfix / Demo | 任何規模均有對應操作 |
| 安裝 | `uvx specify init`（需 Python + uv） | Claude Code Slash Command |

## 規格文件類型

### OpenSpec 格式
- **proposal.md**：問題描述、目標、邊界
- **behavioral specs**：行為期望（given/when/then 格式）
- **design.md**：技術選型與架構決策
- **task checklist**：可執行的實作任務清單

### GitHub spec-kit 格式
- **spec.md**：功能規範（WHAT & WHY，不含 HOW）
- **plan.md**：架構方案與技術設計
- **research.md**：技術選型調研
- **data-model.md**：資料模型定義
- **contracts/**：API 契約
- **tasks.md**：依賴排序的任務清單，`[P]` 標記可並行項

## 規範寫作三條鐵律（GitHub spec-kit）

1. **專注 WHAT & WHY，避免 HOW**：寫「支援實時消息推送」，不寫「用 WebSocket 實作」
2. **用 `[需要澄清]` 代替猜測**：不確定的需求明確標記，讓人類決定而非 AI 猜測
3. **驗收標準必須可測試、可量化**：「回應時間 < 200ms」而非「系統要快」

## SDD 適用邊界（GitHub spec-kit 的明確分類）

| 類型 | 適合 SDD | 說明 |
|------|---------|------|
| 0→1 新產品 | ✅ | 規格先行避免方向偏差 |
| 企業級項目 | ✅ | 文件完整性、多團隊協作 |
| 技術選型實驗 | ✅ | 先規格再評估 |
| 線上緊急 Hotfix | ❌ | 時間壓力下過度 overhead |
| 一次性腳本 | ❌ | 工程化成本高於收益 |
| 個人學習 Demo | ❌ | 不需要正式規格流程 |

## 製造業類比：SDD 是軟體的工程變更管理

製造業有一套成熟的「工程變更管理」流程，SDD 的三段式工作流與之對應：

| 製造業概念 | 說明 | SDD 對應 |
|-----------|------|---------|
| BOM（物料清單） | 產品完整組成清單 | `specs/`（系統現狀規格）|
| ECR（工程變更請求） | 提出變更需求 | `proposal.md`（為什麼改、改什麼）|
| ECO（工程變更單） | 核准後的正式變更文件 | `spec.md`（Delta Headers，已審核）|
| ECN（工程變更通知） | 通知相關方變更生效 | `openspec archive`（合回主規格，全員可見）|

這個類比說明 SDD 不是新概念，而是將製造業數十年的品質管理實踐移植到軟體開發——差別在於 SDD 的流程更輕量，門檻更低。

## 常見陷阱

- 把規格寫成偽程式碼而非行為描述
- 跳過 brainstorming 階段，讓 AI 猜測技術選型
- 不跑 archive，導致舊版規格無法追溯
- 對 30 分鐘任務跑完整 SDD 流程（過度工程化）

## 相關頁面

- [[SDD適用邊界]] — 何時用 SDD、何時不用，四大失敗模式與替代方案
- [[SDD成熟度三層次]] — Spec-first / Spec-anchored / Spec-as-source 的選擇框架
- [[好規格寫作原則]] — 六大結構區域、三層邊界、三條鐵律
- [[EARS需求語法]] — 需求精確化的語法工具（五種句型）
- [[BDD行為驅動開發]] — 驗收標準的標準格式（Given/When/Then）
- [[GitHub-spec-kit]] — GitHub 官方 SDD 工具，三步工作流，適合 0→1
- [[OpenSpec]] — 社群 SDD 框架，決策追溯，適合 1→100
- [[BMAD]] — 大型組織 SDD 框架，詳細角色分工
- [[Superpowers]] — SDD 的程式碼品質執行層
- [[OpenSpec工作流]] — OpenSpec 具體 8 階段操作
- [[Vibe Coding基礎概念]] — SDD 是 Vibe Coding 的進化版
- [[Vibe Coding風險與限制]] — SDD 正是為了緩解這些風險
