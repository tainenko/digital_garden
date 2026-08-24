---
title: Cursor
type: entity
tags: [tool, ai-coding, ide, vibe-coding]
created: 2026-04-27
updated: 2026-04-27
sources: [abmedia-vibe-coding-complete-guide-2026, simular-cursor-vibe-coding]
---

# Cursor

以 VS Code 為底層架構的 AI 代碼編輯器，2026 年 Vibe Coding 最主流的 IDE 工具。估值 92 億美元（2026）。

## 核心功能

| 功能 | 操作 | 說明 |
|------|------|------|
| AI 對話面板 | Cmd/Ctrl + I | 自然語言指令，支援多檔案修改 |
| 行內編輯 | Cmd/Ctrl + K | 局部修改，附 diff 預覽 |
| Autocomplete | Tab 接受 | 智慧代碼預測 |
| 圖片輸入 | 直接上傳 | 分析設計稿自動生成 UI |
| 除錯 | 貼 error message | 自動分析 stack trace 並修復 |

## 定位

- **目標用戶**: 有程式基礎的專業開發者
- **差異化**: 對整個專案的 context 理解最深
- **底層**: VS Code，所有 VS Code 插件相容

## 操作最佳實踐

- 複雜需求拆成小步驟，逐步下指令
- 新開發階段開新對話（避免舊 context 污染）
- 手動選取相關檔案提供精確 context
- 每次修改後視覺確認結果再 Accept

## 相關頁面

- [[Vibe Coding工具比較]] — 與 Bolt.new、Windsurf 等的定位比較
- [[Vibe Coding基礎概念]] — Vibe Coding 工作流
- [[Bolt.new]] — 非技術用戶替代方案
