---
title: Claude Code創始人Boris揭秘完整工作流設定
type: source-summary
tags: [boris, claude-code, anthropic, workflow, setup]
created: 2026-05-06
updated: 2026-05-06
sources: [boris-claude-code-setup-youtube]
---

# Claude Code創始人Boris揭秘完整工作流設定

## Origin

- **標題**：🚀Claude Code创始人Boris亲自揭秘：他的设置竟然如此简单！开箱即用才是最强工作流，复利工程思维让效率翻倍！Opus 4.5计划模式+iTerm2+斜杠命令+GitHub Actions
- **URL**：https://www.youtube.com/watch?v=Xm-n4m7IaZk
- **主題**：Boris Cherny（Claude Code 創造者，Anthropic Staff Engineer）公開分享個人工作流設定
- **原始素材**：Boris 在 X（原 Twitter）的爆紅推文（8M 次瀏覽）及 howborisusesclaudecode.com 完整整理

## Key Takeaways

1. **設定極簡，開箱即用才是最強**：Boris 個人幾乎不客製化 Claude Code，強調 Claude Code 預設狀態下就能高效運作
2. **5 個並行終端 + Git Worktree**：同時跑 5 個 Claude Code 實例，每個在獨立 worktree，iTerm2 通知驅動切換
3. **Opus 優先，理由是整體成本反而更低**：更聰明的模型需要更少引導，最終 token 消耗更少
4. **計畫模式（Plan Mode）幾乎每次必用**：Shift+Tab 兩次進入，討論到計畫確認後切換 auto-accept
5. **驗證是 #1 優先級**：給 Claude 能自我驗證的方式，品質提升 2–3 倍
6. **複利工程思維（Compounding Engineering）**：CLAUDE.md + @.claude PR 評論 → 每次小改善累積成系統性優勢
7. **Slash Commands 工業化**：存在 `.claude/commands/`，團隊共用，最常用：`/commit-push-pr`、`/go`
8. **PostToolUse Hook 自動格式化**：Write/Edit 之後自動跑 formatter，零人工干預
9. **MCP 整合**：Slack、BigQuery、Sentry 三大 MCP，Boris 本人 6 個月沒親手寫 SQL
10. **`/loop` 和 `/schedule` 進行長期自動化**：/babysit（5分鐘）、/slack-feedback（30分鐘）等定期任務

## Entities Mentioned

- [[Boris]] — Claude Code 創造者，本文主角
- [[Anthropic]] — Boris 任職公司
- [[Erik Schluntz]] — 提出 /btw 命令（1.5M 瀏覽），Anthropic MTS

## Concepts Mentioned

- [[複利工程思維]] — Dan Shipper 提出，Boris 實踐的核心哲學
- [[Boris的Claude Code完整工作流]] — 完整設定詳解
- [[CoWork桌面工具指南]] — 四大心法的 GUI 應用
- [[CLAUDE.md撰寫最佳實踐]] — Git 化 CLAUDE.md 實踐
- [[Claude Code Hooks 深度設定]] — PostToolUse 自動格式化
- [[Claude MCP 伺服器整合指南]] — Slack / BigQuery / Sentry 整合
- [[生產環境Vibe Coding四大策略]] — 驗證優先的理念共鳴

## Contradictions / Tensions

- Boris 主張「開箱即用」，但他實際使用的設定（worktrees、hooks、MCPs、slash commands）對新手並不算簡單——「簡單」指的是不需要深度客製化 *Claude Code 本身的行為*，而非環境設定
- Opus 成本邏輯在簡單任務上不一定成立；Boris 的場景是複雜、多步推理的工程任務

## Questions Raised

- Git worktree 命名策略（za/zb/zc）的最佳實踐是什麼？
- `/schedule`（雲端執行）與 `/loop`（本機執行）如何選擇？
- Opus 4.7 xhigh effort 模式下的實際成本與品質對比？
- Chrome 擴充套件自我驗證的具體設定方式？
