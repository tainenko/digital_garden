---
title: codejunkie99
type: entity
tags: [agent-harness, rust, open-source, author]
created: 2026-06-29
updated: 2026-06-29
sources: [github-codejunkie99-agentic-harness]
---

# codejunkie99

## 基本資料

- **GitHub**：https://github.com/codejunkie99
- **身份**：開源開發者
- **代表作**：[[github-codejunkie99-agentic-harness|agentic-harness]] — Rust 原生的 Agent 執行時 / SDK / CLI（Apache-2.0，v0.1.1，~83 ⭐）

## 技術主張

- **Rust 原生路線**：用單一語言寫 agent，編譯出**單一可攜 binary**，從 laptop → CI → 遠端 Linux 沙盒 → Cloudflare 邊緣皆可跑。
- **四大賣點**：one toolchain（Cargo）、one artifact（自帶 metadata 的執行檔）、typed end-to-end（跨 handler 邊界編譯期檢查）、easy to test（檔案/搜尋/編輯/shell 皆為普通方法）。
- 與 [[alienzhou]]（TypeScript 路線的 [[Agent Harness實作|Zero2Agent]]）形成 Agent Harness 落地語言的對照組。

## 相關頁面

- [[Agent Harness實作]]
- [[ToolContext模式]]
- [[ReAct Pattern]]
- [[alienzhou]]
