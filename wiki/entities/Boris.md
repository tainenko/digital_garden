---
title: Boris
type: entity
tags: [anthropic, claude-code, engineer, creator]
created: 2026-04-30
updated: 2026-05-06
sources: [boris-cowork-startup-ideas-podcast, boris-claude-code-setup-youtube]
---

# Boris

## 基本資訊

- **全名**：Boris Cherny
- **職位**：[[Anthropic]] Staff Engineer
- **身份**：Claude Code 創造者
- **知名事蹟**：一篇講自己怎麼用 Claude Code 的 X 推文獲得 **8 百萬次瀏覽、99,000 個書籤**

## 核心理念

### 設定哲學：開箱即用才是最強

> 「我的設定可能出乎意料地樸素。Claude Code 開箱即用效果很好，我個人不太客製化。Claude Code 團隊裡每個人用法都非常不同。沒有唯一正確的方式。」

Boris 的設定極簡——不代表沒有設定，而是不需要**深度客製化 Claude 的行為**，只需要建立好工作流框架。

## 四大心法（爆紅推文核心）

### 1. 計畫模式是最被低估的功能
幾乎每個 session 都從計畫模式（Plan Mode，Shift+Tab ×2）開始。與 Claude 來回討論計畫、確認方向後，切換 auto-accept 自動執行。

> 「好的計畫非常重要，能避免後期大量問題。計畫確定了，程式碼就確定了。」

### 2. 永遠用 Opus + 思考模式
違反直覺的成本邏輯：Opus 每 token 更貴，但更聰明、不需要太多引導、工具使用更準確，最終消耗的 token 更少，整體成本往往比 Sonnet 更低。

> 「這是我用過最好的 coding 模型。就算比 Sonnet 慢，因為引導成本更低，最終幾乎總是更快。」

### 3. CLAUDE.md check in 到 Git，全團隊共同維護
- Claude 做錯某件事 → 立刻加進 CLAUDE.md
- Code Review 發現問題 → `@.claude add to CLAUDE.md`，Claude 自動更新
- 同一件事不用說第二次，這是[[複利工程思維]]的核心實踐

### 4. 給 Claude 驗證自己成果的方式
讓 Claude 用 Chrome 擴充套件「看到」自己做出來的東西。驗證是 #1 優先級，品質提升 **2–3 倍**。

> 「就像工程師不能寫了程式卻永遠不跑。能自我驗證的 Claude，成果品質大幅提升。」

## 工作方式

- 同時跑 **5 個終端 Claude Code**（各自在獨立 git worktree）+ **5–10 個 claude.ai/code** web sessions
- 過去兩個月，Claude Code 寫了他 **100% 的程式碼**，自己一行都沒手寫
- **不親手寫 SQL**，已超過 6 個月（全靠 BigQuery MCP）
- 管理者心態：在任務需要決策時介入，其餘時間讓 Claude 並行執行
- 常用 slash commands：`/commit-push-pr`、`/go`、`/loop`
- 定期任務：`/babysit`（5分鐘）、`/slack-feedback`（30分鐘）、`/pr-pruner`（1小時）

## 複利工程思維

Boris 是[[複利工程思維]]（Compounding Engineering，Dan Shipper 概念）的主要推廣者：每一次對工作流的小改善（CLAUDE.md 更新、新 slash command、新 hook）都會在未來所有 session 和所有團隊成員中自動生效，形成複利效應。

Anthropic 工程師今年人均程式碼產出提升 **200%**，複利工程是重要驅動力之一。

## 相關頁面

- [[Boris的Claude Code完整工作流]] — 完整設定詳解（新）
- [[複利工程思維]] — Boris 實踐的核心哲學（新）
- [[CoWork桌面工具指南]] — GUI 版本的工作方式
- [[Claude Code 入門完整指南]] — Claude Code 詳細教程
- [[CLAUDE.md撰寫最佳實踐]] — CLAUDE.md 最佳實踐
- [[Claude Code多人團隊協作指南]] — 團隊並行工作流
