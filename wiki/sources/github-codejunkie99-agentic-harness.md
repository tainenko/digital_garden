---
title: "GitHub: codejunkie99/agentic-harness（Rust-native Agent Harness）"
type: source-summary
tags: [agent-harness, rust, sdk, cli, mcp, runtime, open-source]
created: 2026-06-29
updated: 2026-06-29
sources: [github-codejunkie99-agentic-harness]
---

# GitHub: codejunkie99/agentic-harness

## Origin

- **Title**：Agentic Harness — a Rust-native runtime, SDK, and CLI for software agents
- **Author**：[[codejunkie99]]
- **License**：Apache-2.0
- **Language**：Rust（99.8%），以 Cargo 為建置系統
- **Version / Date**：v0.1.1（2026-05-12）
- **熱度**：~83 ⭐ / 12 forks（擷取當下）
- **URL**：https://github.com/codejunkie99/agentic-harness

一句話定位：**「build agents that read a repo, plan, edit files, run tests, and report back — then ship the same binary to your laptop, CI, a remote Linux sandbox, or the edge.」** 用 Rust 寫一次、編譯出單一可執行檔，到處跑（laptop / CI / 遠端沙盒 / Cloudflare 邊緣）。

## Key Takeaways

1. **Rust 原生、無 JavaScript**：與多數 Agent 框架（TypeScript/Python）相反，全鏈路 Rust。賣點是「one toolchain（Cargo 管 build/test/ship）、one artifact（自帶 metadata 的單一執行檔）、typed end-to-end（跨 handler 邊界編譯期檢查）、easy to test（file/search/edit/shell 都是普通方法）」。
2. **三層架構**：你的 Rust 程式碼 → Harness（SDK / CLI / HTTP）→ 執行目標（native / CI / 遠端沙盒 / Cloudflare Workers）。同一份 binary 可部署到多個目標。
3. **八個核心抽象**：`AgentApp`（根註冊表）、`Session`（持久對話，綁定 agent 名稱 + ID）、`Task`（一次性子 session，全新歷史、共用 workspace）、`Role`（per-call system-prompt 疊加，從 `.agentic-harness/roles/` 載入）、`Skill`（workspace Markdown 自動發現的行為描述子）、`SessionEnv`（local / HttpSessionEnv / Cloudflare）、`ModelClient`（provider-neutral trait，可換後端不動 handler）、`Connector`（接第三方沙盒/MCP/模型閘道的 recipe）。
4. **Agent 身分用 URL 路徑**：`POST /agents/<name>/<id>`，以 `<id>` 維持 session 連續性；重用 ID 即可續接對話。
5. **MCP 整合**：透過 `McpServerOptions` 掛載第三方工具（範例：連 `https://mcp.sentry.io/mcp`，帶 Bearer token），再 `session.with_tools(...)`。
6. **Schema-guided 輸出**：內建 `---RESULT_START---` / `---RESULT_END---` 標記，從推理文字中自動抽取 typed JSON（呼應 [[Agent Harness實作]] 的「輸出契約」設計哲學）。
7. **長任務的 context 管理**：自動 session compaction（壓縮）；自動 session 持久化，產出 run artifacts（briefs / diffs / logs）。
8. **平行 Task 執行**：可同時跑多個子 session 做分散式分析。
9. **CLI 命令**：`guide`（TUI 入口）、`code`（在 workspace 跑 coding-agent 迴圈）、`new <path>`（從模板 scaffold：coding / code-review / test-fixer / docs-writer）、`dev`（watch + auto-reload）、`run <name>`（單次呼叫任一 agent）、`serve` / `host`（HTTP 伺服器）、`build`（編 native / Node / Cloudflare）、`sandbox`（管遠端 Linux 沙盒）、`add`（裝 connector recipe）、`doctor`（環境診斷）。
10. **Connector 生態**：CLI 幫手可生成 Rust adapter 接第三方沙盒（e2b、Daytona 等）。

## 最小範例（webhook agent）

```rust
use agentic_harness::prelude::*;

fn app() -> Result<AgentApp, AgenticHarnessError> {
    Ok(AgentApp::new()
        .with_workspace(".")
        .load_workspace_context()?
        .agent(AgentDefinition::webhook("hello", |ctx: AgentContext| {
            let payload: HelloPayload = ctx.payload()?;
            Ok(json!({ "message": format!("Hello, {name}!") }))
        })))
}
```

呼叫：`agentic-harness run hello --workspace ./my-agent --id demo --payload '{"name":"Ada"}'`

## Entities mentioned

- [[codejunkie99]] — 作者
- [[Anthropic]] — `ModelClient` provider-neutral，可接 Claude 等後端

## Concepts mentioned

- [[Agent Harness實作]] — 本 repo 是 Rust 版的「宿主層」實作，與 Zero2Agent（TypeScript）形成語言/工程對比
- [[ToolContext模式]] — `AgentContext` 顯式注入 workspace / payload / id，與 ToolContext 同源思路
- [[ReAct Pattern]] — coding-agent 迴圈底層
- [[Claude MCP 伺服器整合指南]] — MCP 掛載方式可對照

## Contradictions / tensions

- **語言選型張力**：[[alienzhou]] 的 [[Agent Harness實作|Zero2Agent]] 主張 TypeScript 生態（pnpm monorepo、Anthropic SDK），本 repo 主張 Rust「無 JS、單一 binary、編譯期型別安全」。兩者對「Agent Harness 該用什麼語言落地」給出相反答案——TS 重生態與迭代速度，Rust 重部署可攜性與型別保證。
- **成熟度**：v0.1.1、~83 ⭐，遠不及 Claude Code / Codex 等成熟產品；定位偏「可自架的開源 runtime」而非開箱即用工具。

## Questions raised

- Rust 的編譯期型別安全在「LLM 輸出本質非結構化」的場景下，實際省下多少 runtime 錯誤？schema-guided 抽取是否足以彌合？
- 「同一 binary 跑到 Cloudflare Workers 邊緣」對 agent 工作負載（長 context、檔案系統操作）的真實限制為何？
- Connector 生態（e2b / Daytona）與富途/TradingView 那類 MCP 整合相比，可組合性如何？
- 與 [[alienzhou-zero2agent|Zero2Agent]] 的教學取向不同，本 repo 偏「成品框架」——是否有等價的設計決策文件可供學習？
