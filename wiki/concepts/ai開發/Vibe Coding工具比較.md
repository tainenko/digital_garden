---
title: Vibe Coding 工具比較
type: concept
tags: [vibe-coding, tools, cursor, bolt, replit, comparison, openspec, superpowers]
created: 2026-04-27
updated: 2026-04-28
sources: [abmedia-vibe-coding-complete-guide-2026, simular-cursor-vibe-coding, openspec-superpowers-multi-source]
---

# Vibe Coding 工具比較

Vibe Coding 工具依「技術門檻 vs 控制深度」分為三層：**瀏覽器即用**（給完全不懂程式的人）、**AI 增強 IDE**（給有基礎的開發者）、**Terminal / CLI**（給進階工程師）。

## 主流工具一覽

| 工具 | 類型 | 定位 | 月費 | 最適合 |
|------|------|------|------|--------|
| **[[Cursor]]** | AI Code Editor | 深度專案理解，VS Code 底層 | ~$20 | 有程式基礎的開發者 |
| **Claude Code** | CLI Tool | Terminal 原生，自主執行 | API 計費 | 進階工程師 |
| **GitHub Copilot** | IDE Plugin | 廣泛 IDE 支援，GitHub 整合 | $10–$19 | 既有 VS Code 用戶 |
| **[[Bolt.new]]** | Browser IDE | 零安裝，一提示生成應用 | 免費起 | 非技術創辦人 |
| **Windsurf** | Code Editor | Cascade 多步驟工作流 | $15 | 預算有限的獨立開發者 |
| **[[Replit]]** | Cloud IDE | 協作、教育導向 | 免費起 | 初學者、學生 |

## 選擇指南

```
完全沒有程式背景？
  └─ 想快速驗證一個 app 概念 → Bolt.new
  └─ 想學習同時做東西 → Replit

有基本程式概念？
  └─ 預算有限 → Windsurf ($15/月)
  └─ 想要最完整的體驗 → Cursor (~$20/月)
  └─ 已在用 VS Code → GitHub Copilot

進階工程師？
  └─ 喜歡 terminal 工作流 → Claude Code
  └─ 需要深度 codebase 理解 → Cursor
```

## Cursor 操作重點

- **Cmd+I**：開啟 AI 對話面板（主要工作入口）
- **Cmd+K**：行內快速編輯 + diff 預覽
- **支援圖片輸入**：可上傳設計稿讓 AI 對應實作
- **Context 管理**：手動選取相關檔案，讓 AI 聚焦

## 工具選擇的核心原則

1. **不要一次用太多工具**：選一個熟透，再考慮其他
2. **工具不能取代判斷力**：不論哪種工具，都需要你驗證輸出
3. **從需求倒推**：快速原型選 Bolt.new；長期專案選 Cursor

## Spec 驅動框架層（框架 > 工具）

工具（Cursor / Claude Code）解決「AI 怎麼寫程式」，而 Spec 驅動框架解決「AI 應該做什麼 + 有沒有遵守工程紀律」。

| 框架 | 安裝在 | 核心價值 |
|------|--------|---------|
| [[OpenSpec]] | npm 全域安裝，任何 AI 皆可用 | 決策可追溯、需求不偏移 |
| [[Superpowers]] | Claude Code plugin | 強制 TDD + code review |

**建議疊加方式**：
- `< 2h` 原型 → Claude Code 單獨
- `2–8h` 個人功能 → Claude Code + Superpowers
- `4–16h` 團隊功能 → Claude Code + OpenSpec + Superpowers

## 相關頁面

- [[Vibe Coding基礎概念]] — 工作流總覽
- [[Vibe Coding風險與限制]] — 各工具的安全注意事項
- [[Spec驅動開發]] — 框架層方法論
- [[OpenSpec]] — Spec 驅動框架
- [[Superpowers]] — TDD 執行框架
- [[Cursor]] — Cursor 詳細介紹
- [[Bolt.new]] — Bolt.new 詳細介紹
- [[Replit]] — Replit 詳細介紹
