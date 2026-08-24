---
title: Superpowers 技能框架
type: concept
tags: [superpowers, tdd, claude-code, multi-agent, code-review, git-worktree]
created: 2026-04-28
updated: 2026-04-28
sources: [openspec-superpowers-multi-source]
---

# Superpowers 技能框架

[[Superpowers]] 透過 CLAUDE.md 配置，將專業工程紀律「安裝」到 Claude Code 中，使 AI 在執行任何任務時自動遵守 TDD、code review、branch isolation 等規範。

## 核心工程紀律

### TDD（測試驅動開發）
```
RED  → 先寫一個會失敗的測試
GREEN → 寫最小實作讓測試通過
REFACTOR → 在測試保護下整理程式碼
```

### 自動 Code Review
- 獨立的 reviewer agent（非 implementer 本身）進行審查
- 客觀評估：邏輯正確性、安全性、可讀性
- 避免「自己改自己」的確認偏誤

### Git Worktree 隔離
- 每個功能在獨立 worktree 中開發
- 主分支保持乾淨
- 完成後 merge 或丟棄

### Pre-Completion Verification
- 提交前強制執行驗證清單
- 確保所有測試通過、lint 無錯誤

## 多 Agent 架構

```
Controller Agent
    ├── Implementer Agent（寫程式碼）
    └── Reviewer Agent（審查程式碼）
```

- **優點**：reviewer 與 implementer 分離，避免自我審查盲點
- **代價**：token 消耗比單 agent（如 [[OpenSpec]]）高出約 3–5 倍

## 5 階段工作流

| 階段 | 內容 |
|------|------|
| Brainstorming | 需求探索，明確範圍 |
| Git Worktree | 建立隔離開發環境 |
| Writing Plans | 逐步執行計畫 |
| Subagent Development | Controller 指揮 Implementer 開發、Reviewer 審查 |
| Finishing Branch | Merge 或丟棄，清理 worktree |

## Token 成本考量

| 情境 | 建議 |
|------|------|
| 個人原型（< 2h） | 不需要 Superpowers |
| 個人功能（2–8h） | Superpowers 單獨使用即可 |
| 團隊功能（4–16h）| Superpowers + [[OpenSpec]] |
| 大型/平行開發 | Full triple stack + worktrees |

## 安裝與配置

```bash
# 在 Claude Code 中安裝
/plugin install superpowers@claude-plugins-official

# 配置 .claude/settings.json
{
  "mcpServers": { ... }
}
```

CLAUDE.md 中不需要明確提及 Superpowers，安裝後自動生效。

## 相關頁面

- [[Superpowers]] — 框架整體介紹
- [[OpenSpec工作流]] — 搭配的規格管理框架
- [[Spec驅動開發]] — 方法論背景
- [[Vibe Coding風險與限制]] — Superpowers 緩解的具體風險
