---
title: Claude Code Plugins 插件系統
type: concept
tags: [claude-code, plugins, distribution, skills, commands, hooks]
created: 2026-05-07
updated: 2026-05-07
sources: [github-huangjia2019-claude-code-engineering]
---

# Claude Code Plugins 插件系統

## 什麼是 Plugin

Plugin 是 Skills、Commands、Hooks（以及可選的 MCP Server）的打包封裝，形成一個含 manifest 的可安裝、可分發的能力包。

> "寫出有用的 Skills 是工藝，把它打包分發才是工程。"
> — [[黃佳 huangjia2019]]，第 23 講

## 插件目錄結構

```
my-plugin/
├── manifest.json          # 插件描述：名稱、版本、作者、依賴
├── agents/                # 內嵌的 Sub-Agent 定義
│   └── reviewer.md
├── commands/              # Slash 命令
│   ├── review.md
│   └── test.md
├── hooks/                 # 事件 Hooks
│   ├── pre-commit.sh
│   └── post-test.sh
└── README.md
```

## manifest.json 格式

```json
{
  "name": "my-team-toolkit",
  "version": "1.0.0",
  "description": "團隊標準化開發工具包",
  "author": "team@example.com",
  "capabilities": ["skills", "commands", "hooks"],
  "dependencies": {
    "mcp-servers": ["github", "notion"]
  }
}
```

## Plugin vs 散裝組件

| 面向 | 散裝（Skills/Commands/Hooks 各自放）| Plugin（打包）|
|------|-----------------------------------|--------------|
| 安裝方式 | 手動複製各個目錄 | 一條命令安裝 |
| 版本管理 | 各自獨立，容易不一致 | 統一 manifest 版本號 |
| 分發方式 | 口耳相傳、Slack 貼連結 | npm / git / 內部 registry |
| 依賴聲明 | 無標準 | manifest 明確聲明 MCP 依賴 |
| 適合場景 | 個人實驗 | 團隊/社群共用 |

## 分發策略

### 1. 團隊內部分發
```bash
# git submodule
git submodule add https://github.com/team/claude-toolkit .claude/plugins/toolkit

# 或 npm（若發布為 npm package）
npm install @team/claude-toolkit
```

### 2. 開源社群（[[Skillsmap]]）
- 發布到 Skillsmap 市場（10,000+ Skills）
- 標準化 manifest 讓平台可索引、一鍵安裝

### 3. 企業私有 Registry
- 搭配 Managed Settings 強制安裝特定 Plugin
- 版本鎖定確保合規

## 設計原則

1. **模組化**：每個 Plugin 解決一個明確的問題域
2. **最小依賴**：只聲明真正需要的 MCP Server
3. **版本兼容性**：遵循 semver，breaking change 升 major
4. **文檔即合約**：README 描述觸發條件、副作用、權限需求

## 與行業標準的關係

第 14 講提到 Skills 出圈的趨勢——不同平台（Claude、Cursor、Copilot）對「可重用 AI 工作流」的概念正在收斂，但格式尚未統一。Plugin 是 [[Anthropic]] 生態內的標準化嘗試。

## 相關頁面

- [[Claude Code工程化架構與全景]]
- [[Skillsmap]]
- [[Claude Code Hooks 深度設定]]
- [[Claude Code Subagents 完整指南]]
- [[黃佳 huangjia2019]]
