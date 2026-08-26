---
title: JetBrains go-modern-guidelines — 讓 AI 寫出現代化 Go 程式碼
type: source-summary
tags: [golang, AI輔助開發, JetBrains, Claude Code, 現代化]
created: 2026-08-25
updated: 2026-08-25
sources: [jetbrains-go-modern-guidelines]
---

# JetBrains go-modern-guidelines

## Origin

- **標題**：讓 AI 寫出「現代化 Go」代碼：JetBrains 發布 go-modern-guidelines
- **原文**：騰訊雲開發者社區（GoLang學習記），2026-03-01
- **官方 Blog**：[JetBrains Blog 2026-08-24](https://blog.jetbrains.com/go/2026/08/24/help-ai-coding-agents-write-up-to-date-code-with-modern-golang-skills/)
- **原始介紹**：[Write Modern Go Code With Junie and Claude Code](https://blog.jetbrains.com/go/2026/02/20/write-modern-go-code-with-junie-and-claude-code/)
- **GitHub**：github.com/JetBrains/go-modern-guidelines

## Key Takeaways

1. **AI 寫 Go 的兩大問題**：訓練資料截止日（錯過新版功能）＋頻率偏差（舊寫法在訓練資料中比例高，AI 偏好重複）
2. **版本感知（Version-Aware）**：工具讀取 `go.mod`，只提供當前版本適用的規則，不推薦超前版本功能
3. **涵蓋範圍**：Go 1.0 ~ 1.27 的現代化規則，隨新版本持續更新
4. **兩種 CLI 指令**：`list`（簡短列出相關規則 ID）、`explain`（詳細 before/after 範例）
5. **支援的 AI 工具**：Junie、Claude Code（`/use-modern-go` 指令）、Cursor、Codex

## Entities Mentioned

- [[JetBrains]] — 發布者，GoLand IDE 開發商
- [[GoLand]] — JetBrains 的 Go IDE，Junie AI 整合其中

## Concepts Mentioned

- [[go-modern-guidelines 現代化規則]] — 核心主題：具體規則與 before/after 範例

## Contradictions / Tensions

- 工具需要 Go 1.25+ toolchain 才能執行，但覆蓋範圍從 Go 1.0 開始——老專案若 toolchain 未升級則無法使用
- 版本感知是優點，但也代表每次升級 Go 版本後需重新確認哪些新規則適用

## Questions Raised

- 除了 JetBrains，Google（gopls）是否也有類似的 AI 現代化規則集？
- 這套規則與 Go 官方的 `modernize` analyzer（`go fix`）的重疊程度如何？
