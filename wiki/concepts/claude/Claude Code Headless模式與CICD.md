---
title: Claude Code Headless 模式與 CI/CD
type: concept
tags: [claude-code, headless, cicd, github-actions, automation]
created: 2026-05-07
updated: 2026-05-07
sources: [github-huangjia2019-claude-code-engineering]
---

# Claude Code Headless 模式與 CI/CD

## 什麼是 Headless 模式

Claude Code 不需要人工守著終端——透過 Headless 模式，它可以嵌入 GitHub Actions、排程任務、自動化流水線，成為「永不休息的 CI 同事」。

> "當 Claude Code 學會無人值守，你的 CI/CD 流水線就多了一個永遠不需要休息的同事。"
> — [[黃佳 huangjia2019]]，第 19 講

## 啟動方式

```bash
# 非互動模式，接受 stdin 輸入
claude --headless

# 或透過 Agent SDK 程式化呼叫
claude -p "your prompt here" --output-format json
```

## 核心應用：自動 PR Review（GitHub Actions）

```yaml
# .github/workflows/claude-review.yml
name: Claude PR Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Claude Code Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          claude --headless -p "Review the diff of this PR for bugs, security issues, and style violations. Be concise." \
            --output-format json
```

## 安全考量

Headless 模式的風險高於互動模式（無人在旁審核），需搭配：

1. **權限最小化**：`allowedTools` 只開放必要工具
2. **Rules deny 規則**：明確禁止危險操作（刪除、push force 等）
3. **Token 管理**：`ANTHROPIC_API_KEY` 存於 Secrets，不硬編碼
4. **輸出稽核**：結合 `PostToolUse` Hooks 記錄所有動作

詳見 [[Claude Code Rules規則系統]]、[[Claude Code Hooks 深度設定]]。

## Headless vs 互動模式

| 面向 | Headless | 互動（Interactive）|
|------|---------|-------------------|
| 觸發方式 | CI/CD 系統、排程 | 人工在終端輸入 |
| 監督 | 無人守候 | 即時人工審核 |
| 輸出格式 | JSON（結構化）| 串流文字 |
| 適合場景 | PR Review、自動測試修復、定期報告 | 開發、調試、探索 |
| 安全門檻 | 需更嚴格的 Rules + Hooks | 人工介入可即時修正 |

## 與 Agent SDK 的關係

Headless 模式是 Headless 調用的 CLI 介面；[[Claude Code Subagents 完整指南|Agent SDK]] 是程式化介面（Python `query()` 方法）。兩者底層相同，Agent SDK 提供更精細的控制（自定義工具 `@tool`、Hook 攔截、選項配置）。

## 相關頁面

- [[Claude Code工程化架構與全景]]
- [[Claude Code Rules規則系統]]
- [[Claude Code Hooks 深度設定]]
- [[Claude Code Subagents 完整指南]]
- [[黃佳 huangjia2019]]
