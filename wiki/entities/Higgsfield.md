---
title: Higgsfield
type: entity
tags: [ai工具, 影像生成, mcp, 內容創作, 自動化]
created: 2026-04-30
updated: 2026-04-30
sources: [chase-ai-higgsfield-mcp-carousel]
---

# Higgsfield

## 基本資料

- **類型**：AI 影像／影片生成平台
- **官網**：higgsfield.ai
- **定位**：專業級 AI 視覺內容生成，主打電影感／高品質輸出
- **MCP 支援**：提供 MCP Server，可與 Claude Code 直接整合

---

## 核心能力

- 文字生成圖像（Text-to-Image）
- 文字生成影片（Text-to-Video）
- 風格一致性控制（適合品牌內容批量生產）
- **MCP Server**：讓 Claude Code 可以直接呼叫 Higgsfield API 生成視覺素材

---

## 在內容自動化中的角色

與 Claude Code 搭配使用時，Higgsfield 負責**視覺生成**端：

```
Claude Code（文字策略 + 流程控制）
    ↓ 透過 MCP 呼叫
Higgsfield（圖像／影片生成）
    ↓
輸出：完整輪播貼文視覺素材
```

詳見 [[輪播內容自動化工作流]]。

---

## 相關頁面

- [[輪播內容自動化工作流]] — Claude Code + Higgsfield MCP 完整工作流
- [[IG輪播轉換設計]] — 輪播策略設計（人工或 AI 輔助皆適用）
- [[Claude MCP 伺服器整合指南]] — MCP 協定原理與整合方式
- [[LIFE根系內容系統]] — growithfyn 使用此工具組的內容系統
