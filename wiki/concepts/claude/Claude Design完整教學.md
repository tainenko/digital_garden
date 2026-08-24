---
title: Claude Design 完整教學
type: concept
tags: [claude, design, prototype, figma, no-code]
created: 2026-05-07
updated: 2026-05-07
sources: [aiposthub-claude-tutorial-2026]
---

# Claude Design 完整教學

## 什麼是 Claude Design

[[Anthropic]] 於 **2026 年 4 月**正式推出 Claude Design，讓沒有設計背景的人用一段文字就能生成可互動的視覺成品，不需學習 Figma 或任何設計工具。

## 六大應用場景

1. **App 原型**：描述功能需求，直接輸出可點擊的 App UI
2. **Landing Page**：一句話生成行銷頁，含版面、配色、文案
3. **簡報**：輸入主題與大綱，Claude Design 生成完整投影片
4. **資訊圖表**：把數據或概念視覺化
5. **UI 元件**：生成符合需求的按鈕、表單、卡片等元件
6. **品牌素材**：Logo 草稿、社群封面等

## Claude Design vs Figma MCP

| 面向 | Claude Design | Figma MCP |
|------|--------------|-----------|
| 目標用戶 | 非設計師、PM、企劃 | 設計師、工程師 |
| 使用方式 | 純對話，零工具門檻 | 需安裝 Figma MCP + Claude Code |
| 輸出 | 可互動原型、靜態設計 | 符合 Design System 的前端代碼 |
| 雙向編輯 | 否（單向生成） | 是（改圖即改 Code）|
| 適合階段 | 概念發想、快速原型 | 設計交接、前端實作 |

## Figma MCP 工作流（設計師版）

若需要將設計稿 100% 轉為前端代碼，改用 Figma MCP + Claude Code：

1. 在 Figma 完成設計
2. 安裝 Figma MCP（Beta）
3. 在 Claude Code 呼叫 Figma 資源
4. Claude Code 讀取 Design System，一鍵輸出符合規範的前端代碼
5. 雙向：改 Figma → Code 同步更新；改 Code → 可反映回 Figma

## 費用

Claude Design 的費用依 Claude 方案而定（具體方案請查閱 [[AI郵報]] 原文）。

## 相關頁面

- [[Claude生態系四種應用]]
- [[Claude Code 入門完整指南]]
- [[Vibe Coding基礎概念]]
- [[AI郵報]]
