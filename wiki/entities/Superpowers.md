---
title: Superpowers
type: entity
tags: [superpowers, tdd, claude-code, ai-coding, open-source, obra]
created: 2026-04-28
updated: 2026-04-28
sources: [openspec-superpowers-multi-source]
---

# Superpowers

由 **obra** 開發的開源技能框架，安裝在 Claude Code（或其他支援 subagent 的平台）中，強制執行專業工程紀律：TDD、code review、git worktree 隔離。

## 核心定位

> Superpowers 解答「這段程式碼正確嗎？」

- **主要解決**：Wall 2——缺乏工程紀律（沒有測試、沒有 code review）
- **特點**：通過 CLAUDE.md 配置自動激活，AI 執行時不需手動提醒
- **Agent 架構**：多 agent（controller + implementer + reviewer）

## 工作流（5 階段）

詳見 [[Superpowers技能框架]]。

```
brainstorming → git worktree → writing plans → subagent development → finishing branch
```

## 強制執行的工程紀律

- **TDD**：RED（先寫失敗測試）→ GREEN（實作通過）→ REFACTOR（整理）
- **自動 Code Review**：由獨立 reviewer agent 進行客觀審查
- **Git Worktree 隔離**：每個功能在獨立 worktree 開發，避免污染主分支
- **完成前驗證**：pre-completion checklist

## 安裝

在 Claude Code 中執行：
```
/plugin install superpowers@claude-plugins-official
```
然後配置 `.claude/settings.json` 中的 MCP server。

## 與 OpenSpec 的關係

兩者不會自動串聯——在 `openspec apply` 階段啟動 Superpowers 的紀律，需要在專案的 CLAUDE.md 中明確加入規則。

| 面向 | [[OpenSpec]] | Superpowers |
|------|-------------|-------------|
| 解決什麼 | 決策可追溯 | 程式碼品質 |
| TDD 執行 | 無 | 強制 |
| Agent 數量 | 單一 | 多個（3+）|
| Token 成本 | 低 | 較高 |
| Git 管理 | 手動 | 自動 worktree |

## 何時值得用

- **Superpowers 足夠**：2–8 小時的個人功能開發，TDD 就能防止大多數問題
- **需要加 OpenSpec**：4 小時以上的團隊協作、需要合規審計記錄的企業專案

## 相關頁面

- [[Superpowers技能框架]] — 技術細節
- [[Spec驅動開發]] — 方法論背景
- [[OpenSpec]] — 可搭配的規格管理框架
- [[Vibe Coding風險與限制]] — Superpowers 正是為了緩解這些風險
