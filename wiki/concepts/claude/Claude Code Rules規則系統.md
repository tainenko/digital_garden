---
title: Claude Code Rules 規則系統
type: concept
tags: [claude-code, rules, permissions, security, governance]
created: 2026-05-07
updated: 2026-05-07
sources: [github-huangjia2019-claude-code-engineering]
---

# Claude Code Rules 規則系統

## 核心區分

> "指令規則告訴 Claude 該怎麼做，權限規則告訴 Claude 能做什麼。"
> — [[黃佳 huangjia2019]]，第 20 講

Rules 系統由兩個完全獨立的子系統組成：

| 子系統 | 解決的問題 | 設定位置 |
|--------|----------|---------|
| **指令規則** | Claude 應該遵循什麼規範、風格、流程 | `.claude/rules/*.md` |
| **權限規則** | Claude 被允許/禁止使用哪些工具與操作 | `settings.json` → `permissions` |

## 指令規則（Instruction Rules）

存放於 `.claude/rules/` 目錄，按**路徑匹配**動態載入。

```
.claude/rules/
├── global.md          # 全域規則，任何 session 都載入
├── backend.md         # 只在 backend/ 目錄下載入
├── frontend.md        # 只在 frontend/ 目錄下載入
└── security.md        # 安全審查相關規則
```

**vs CLAUDE.md 的分工：**

| 面向 | CLAUDE.md | `.claude/rules/*.md` |
|------|-----------|----------------------|
| 範圍 | 全局（整個 Project）| 按目錄路徑動態載入 |
| 性質 | 永久生效 | 條件性生效 |
| 用途 | 項目整體規範、背景知識 | 領域專屬規則（前端/後端/安全…）|

詳見 [[Claude Code 記憶體系統深度指南]]。

## 權限規則（Permission Rules）

三級評估框架，由嚴到寬依序：

```
deny → ask → allow
```

### deny（明確禁止）
任何情況都不執行，Claude 必須拒絕：

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force)",
      "Write(/etc/*)"
    ]
  }
}
```

### ask（需要確認）
執行前必須向用戶確認，適合高風險但合法的操作：

```json
{
  "permissions": {
    "ask": [
      "Bash(git commit *)",
      "Write(*.env)"
    ]
  }
}
```

### allow（預先授權）
無需每次確認，加速日常開發流程：

```json
{
  "permissions": {
    "allow": [
      "Read(*)",
      "Bash(npm run *)",
      "Bash(pytest *)"
    ]
  }
}
```

## 三層設定繼承

```
~/.claude/settings.json          ← 用戶全局（個人習慣）
    ↓ 覆蓋
.claude/settings.json            ← 項目級（團隊共享，git commit）
    ↓ 覆蓋
.claude/settings.local.json      ← 本地覆蓋（個人，加入 .gitignore）
```

## 企業治理（縱深防禦）

大型團隊的 Rules 策略：

1. **Managed Settings**：平台/管理員強制下發的規則，個人不可覆蓋
2. **Project deny list**：團隊共識的禁止操作清單（commit 入 git）
3. **Personal allow list**：個人開發效率加速（本地 settings）
4. **Hooks 補充監控**：Rules 管不到的細粒度行為由 `PreToolUse` Hooks 攔截

詳見 [[Claude Code Hooks 深度設定]]。

## 與 Headless 模式的搭配

在無人值守的 CI 環境中，Rules 的 deny 清單是最後防線：

```json
{
  "permissions": {
    "deny": [
      "Bash(rm *)",
      "Bash(git push*)",
      "Bash(curl * | bash)"
    ],
    "allow": [
      "Read(*)",
      "Bash(npm test)",
      "Bash(pytest)"
    ]
  }
}
```

詳見 [[Claude Code Headless模式與CICD]]。

## 相關頁面

- [[Claude Code工程化架構與全景]]
- [[Claude Code 記憶體系統深度指南]]
- [[Claude Code Hooks 深度設定]]
- [[Claude Code Headless模式與CICD]]
- [[CLAUDE.md撰寫最佳實踐]]
- [[黃佳 huangjia2019]]
