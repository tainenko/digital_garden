---
title: AI 時代，一定要學會使用 GitHub spec kit — SDD 規格驅動開發
type: source-summary
tags: [spec-kit, SDD, 實戰, GitHub, Claude Code]
created: 2026-05-18
updated: 2026-05-18
sources: [milkmidi-github-spec-kit]
---

# AI 時代，一定要學會使用 GitHub spec kit — SDD 規格驅動開發

## Origin
- **作者**：Milk Midi（Medium）
- **日期**：2025-10-08
- **URL**：https://milkmidi.medium.com/ai-時代-一定要學會使用-github-spec-kit-sdd-規格驅動開發-f2df57cfdf3c

## Key Takeaways

- **SDD 的核心價值**：「有明確的規格可以讓團隊有個對焦的文件」——解決 AI 開發中各自理解不同的問題。
- **GitHub spec-kit 完整六步實戰**：

  | 步驟 | 指令 | 說明 |
  |------|------|------|
  | 1 | `uvx specify init` | 初始化專案，選 AI 助手 |
  | 2 | `/speckit.constitution` | 建立項目憲法（生成 `.specify/memory/constitution.md`） |
  | 3 | `/speckit.specify` | 定義功能需求（WHAT & WHY，不談技術棧）|
  | 4 | `/speckit.plan` | 技術實施計畫（需 RD 專業判斷）|
  | 5 | `/speckit.tasks` | 自動生成含 TDD 框架的任務清單 |
  | 6 | `/speckit.implement` | AI 執行開發 |

- **額外指令 `/speckit.clarify`**：需求澄清，讓 AI 主動問問題而非猜測——這是 spec-kit 文件中較少被提到的重要指令。
- **技術規劃是人類責任**：`/plan` 階段的技術選型（用 Vite 或 Next.js？避免 setInterval？）需要 RD 的專業經驗，不能全丟給 AI。
- **憲法的三次強調**：作者連續強調「使用 MVP，不要 overdesign」——constitution.md 的核心心法。
- **Yolo 模式警示**：`claude --dangerously-skip-permissions` 可加速開發，但作者明確提醒這是有風險的操作。
- **現實提醒**：「勿相信『不需程式背景也能開發百萬級應用』的說法」——技術背景仍是必要的。

## Concepts Mentioned
- [[GitHub-spec-kit]] — 本文主角
- [[Spec驅動開發]] — SDD 方法論背景
- [[SDD適用邊界]] — 「不需技術背景」是假命題

## Questions Raised
- `/speckit.clarify` 的最佳觸發時機是什麼？應在 specify 之前還是之後？
