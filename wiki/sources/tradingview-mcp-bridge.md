---
title: "TradingView MCP Bridge：用 Claude Code 控制 TradingView 桌面版"
type: source-summary
tags: [tradingview, mcp, claude-code, pine-script, cdp, chart-analysis, quantitative-trading]
created: 2026-06-26
updated: 2026-06-26
sources: [tradingview-mcp-bridge]
---

# TradingView MCP Bridge：用 Claude Code 控制 TradingView 桌面版

## Origin

- **URL**：https://github.com/tradesdontlie/tradingview-mcp
- **作者**：tradesdontlie（GitHub handle）
- **Stars / Forks**：3,957 ⭐ / 1,931 forks（高熱度研究項目）
- **語言**：JavaScript（Node.js）
- **授權**：MIT

## Key Takeaways

- **用 Chrome DevTools Protocol（CDP）橋接**：TradingView Desktop 是 Electron 應用，所有 Chromium/Electron app 內建 CDP 調試接口。這個工具透過 `--remote-debugging-port=9222` flag 啟動 TradingView，然後用 CDP 在 localhost 讀取並控制圖表——**完全不連接 TradingView 的伺服器**
- **78 個 MCP 工具**：從讀取行情、讀 Pine Script 指標值、控制圖表，到寫入/編譯 Pine Script、管理警報、截圖、回放練習，全部透過 MCP 標準協定暴露給 Claude Code
- **CLI 雙通道**：每個 MCP 工具也有對應的 `tv` CLI 命令（30 個命令、66 個子命令），JSON 輸出可與 `jq` 管道對接
- **Pine Script 完整開發循環**：inject → compile（`pine_smart_compile` 自動偵錯）→ 讀錯誤 → 讀 console → 儲存，Claude 可以在本地完成完整的 Pine Script 開發迭代
- **上下文最佳化設計**：工具輸出預設緊湊格式，「分析圖表」完整工作流只用 5–10KB context，而非 80KB
- **法律風險**：README 明確警告——TradingView ToS 限制自動化數據採集，使用者需自行承擔帳號被封的風險；免責聲明要求不得用於自動交易決策

## 架構

```
Claude Code  ←→  MCP Server (stdio)  ←→  CDP (port 9222)  ←→  TradingView Desktop (Electron)
```

- **連線方式**：MCP over stdio（本地，無需網路）
- **CDP 協定**：`chrome-remote-interface` 套件，`localhost:9222`
- **依賴極簡**：只需 `@modelcontextprotocol/sdk` + `chrome-remote-interface`

## 78 個 MCP 工具分類

### 讀圖表（Chart Reading）

| 工具 | 用途 | 輸出大小 |
|------|------|---------|
| `chart_get_state` | 取得 symbol、timeframe、所有指標名稱+ID（第一步必呼叫）| ~500B |
| `data_get_study_values` | 讀 RSI/MACD/BB/EMA 當前值 | ~500B |
| `quote_get` | 最新價格、OHLC、成交量 | ~200B |
| `data_get_ohlcv` | 價格 K 棒（用 `summary:true` 省 context）| 500B（摘要）/ 8KB（100 棒）|

### 讀 Pine Script 繪圖輸出

| 工具 | 用途 |
|------|------|
| `data_get_pine_lines` | 讀 `line.new()` 輸出的水平支撐/阻力線 |
| `data_get_pine_labels` | 讀 `label.new()` 的文字標註 + 價格 |
| `data_get_pine_tables` | 讀 `table.new()` 的數據表格 |
| `data_get_pine_boxes` | 讀 `box.new()` 的價格區間（{high, low}）|

**使用 `study_filter`** 指定特定指標，避免掃描全部。

### 控制圖表

| 工具 | 功能 |
|------|------|
| `chart_set_symbol` | 切換標的（BTCUSD、AAPL、ES1!、NYMEX:CL1! 等）|
| `chart_set_timeframe` | 切換週期（1/5/15/60/D/W/M）|
| `chart_set_type` | 切換圖表類型（K線/Heikin Ashi/折線/面積/磚形）|
| `chart_manage_indicator` | 加減指標（需用全名如 "Relative Strength Index"）|
| `chart_scroll_to_date` | 跳轉到指定日期 |
| `chart_set_visible_range` | 縮放到精確範圍（unix timestamps）|

### Pine Script 開發循環

| 工具 | 步驟 |
|------|------|
| `pine_set_source` | 1. 注入程式碼到編輯器 |
| `pine_smart_compile` | 2. 編譯（自動偵測 + 錯誤檢查）|
| `pine_get_errors` | 3. 讀編譯錯誤 |
| `pine_get_console` | 4. 讀 `log.info()` 輸出 |
| `pine_save` | 5. 儲存到 TradingView 雲端 |

### 其他能力

- **多窗格布局**：`pane_set_layout`（2x2/3x1 等）、`pane_set_symbol`（各窗格設不同標的）
- **回放模式**：`replay_start/step/autoplay/trade`——步進歷史 K 棒，模擬進出場練習
- **繪圖**：`draw_shape`（趨勢線、水平線、矩形、文字標注）
- **警報**：`alert_create/list/delete`
- **截圖**：`capture_screenshot`（full / chart / strategy_tester 三區域）
- **UI 自動化**：`ui_click/keyboard/hover/scroll/eval`

## 決策樹（CLAUDE.md 定義）

| 用戶說… | Claude 呼叫… |
|---------|-------------|
| "圖表上有什麼？" | `chart_get_state` → `data_get_study_values` → `quote_get` |
| "有什麼價位顯示？" | `data_get_pine_lines` → `data_get_pine_labels` |
| "給我完整分析" | quote → study_values → pine_lines → pine_labels → pine_tables → ohlcv(summary) → screenshot |
| "切到 AAPL 日線" | `chart_set_symbol` → `chart_set_timeframe` |
| "幫我寫 Pine Script…" | `pine_set_source` → `pine_smart_compile` → `pine_get_errors` |
| "從 3/1 開始回放" | `replay_start` → `replay_step` → `replay_trade` |
| "設定 4 圖格局" | `pane_set_layout` → `pane_set_symbol` × 4 |

## 重要限制與法律風險

| 風險 | 說明 |
|------|------|
| **ToS 灰色地帶** | TradingView ToS 限制自動化數據採集，使用可能導致帳號被封 |
| **非官方 API** | 透過未文件化的 TradingView 內部 Electron 接口，任何 TradingView 更新都可能讓工具失效 |
| **不得自動交易** | README 明確聲明：不得用此工具進行自動化交易決策 |
| **需付費訂閱** | TradingView Desktop 付費訂閱必要，此工具不繞過任何付費牆 |

## 安裝

```bash
# Claude Code 一鍵安裝（貼到對話框）
# "Install the TradingView MCP server. Clone https://github.com/tradesdontlie/tradingview-mcp.git,
#  run npm install, add it to my MCP config at ~/.claude/.mcp.json, and launch TradingView
#  with the debug port. Then verify the connection with tv_health_check."

# 手動
git clone https://github.com/tradesdontlie/tradingview-mcp.git
cd tradingview-mcp && npm install

# 啟動 TradingView（Mac）
./scripts/launch_tv_debug_mac.sh

# MCP 設定（~/.claude/.mcp.json）
# { "mcpServers": { "tradingview": { "command": "node", "args": ["/path/to/src/server.js"] } } }
```

## Entities Mentioned

- [[tradesdontlie]] — 作者
- [[Anthropic]] — Claude Code MCP 協定

## Concepts Mentioned

- [[Claude MCP 伺服器整合指南]] — MCP 協定基礎
- [[TradingView MCP Bridge]] — 本頁核心概念
- [[Vibe Coding程式交易實戰]] — 相關使用情境
- [[Pine Script開發工作流]] — 本工具的核心能力之一
