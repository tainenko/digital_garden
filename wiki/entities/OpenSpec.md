---
title: OpenSpec
type: entity
tags: [openspec, spec-driven-development, ai-coding, fission-ai, open-source]
created: 2026-04-28
updated: 2026-05-26
sources: [openspec-superpowers-multi-source, kaochenlong-spectra-openspec]
---

# OpenSpec

由 **Fission-AI** 開發的開源 spec 驅動開發（SDD）框架。核心理念：在 AI 動手寫程式之前，先把需求轉化為結構化文件，避免需求錯位、讓技術決策可追溯。

## 核心定位

> OpenSpec 解答「為什麼做這個改動？」

- **主要解決**：Wall 3——設計理由在每次迭代後消失
- **特點**：平台中立（ChatGPT、GitHub Copilot、Claude Code 皆可用）
- **Token 效率**：高（單 agent 模式）

## 工作流（8 階段）

詳見 [[OpenSpec工作流]]。

```
explore → proposal → design → specs → tasks → apply → verify → archive
```

- `explore`：與 AI 一起探索、腦力激盪
- `propose`：生成四份規格文件（proposal.md / behavioral specs / design.md / task checklist）
- `apply`：逐任務實作
- `archive`：版本化決策記錄（Delta Spec System）

## Delta Spec System

OpenSpec 的關鍵特性：每次修改只寫「描述差異」的 delta spec，審核通過後再 sync 回主規格。這讓「為什麼改」的歷史得以保存。

## v1.0 主要變化（2026-02）

- **非線性工作流**：舊版強制線性執行；v1.0 允許任意順序，系統以 DAG 追蹤文件依賴關係
- **指令前綴**：`/opsx:` 取代 `/openspec:`；新增 `/opsx:explore`、`/opsx:new`、`/opsx:continue`、`/opsx:ff`
- **配置統一**：`openspec/config.yaml`（取代舊版 `project.md`）
- **Skills 系統**：從 slash commands 遷移至 `.claude/skills/` 目錄，相容多工具
- **即時進度**：執行前自動讀取 `tasks.md` 計算完成率
- **Spectra GUI**：配套桌面管理器，見 [[Spectra]]

## 安裝

```bash
npm install -g @fission-ai/openspec@latest
openspec init
```
需要 Node.js 20.19.0+。

## 與 Superpowers 的關係

| 面向 | OpenSpec | [[Superpowers]] |
|------|----------|-----------------|
| 解決什麼 | 決策可追溯 | 程式碼品質 |
| TDD 執行 | 無 | 強制 |
| Agent 數量 | 單一 | 多個 |
| Token 成本 | 低 | 較高 |
| 平台相依 | 無 | 需 subagent 支援 |

## 相關頁面

- [[OpenSpec工作流]] — 8 階段流程與 v1.0 新指令
- [[Spectra]] — 配套桌面 GUI 管理器
- [[Spec驅動開發]] — 方法論背景
- [[Superpowers]] — 可搭配的程式碼品質框架
- [[Vibe Coding工具比較]] — 工具層與框架層的定位差異
- [[DDD AI Agent技能流水線]] — ForceInjection 的 ddd-openspec-bridge 將 DDD 建模成果導入 OpenSpec
