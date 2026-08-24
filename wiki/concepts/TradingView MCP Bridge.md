---
title: TradingView MCP Bridge
type: concept
tags: [tradingview, mcp, claude-code, pine-script, cdp, chart-analysis, quantitative-trading]
created: 2026-06-26
updated: 2026-06-26
sources: [tradingview-mcp-bridge]
---

# TradingView MCP Bridge

## 是什麼

開源 MCP 伺服器，透過 **Chrome DevTools Protocol（CDP）** 橋接 [[Claude Code內部運作機制|Claude Code]] 與 **TradingView Desktop**，讓 AI Agent 可讀取圖表狀態、控制圖表、開發 Pine Script、管理警報，以及模擬回放練習——全部透過自然語言觸發。

**GitHub**：https://github.com/tradesdontlie/tradingview-mcp（3,957 ⭐）

## 架構原理

```
Claude Code ←→ MCP Server (stdio) ←→ CDP localhost:9222 ←→ TradingView Desktop (Electron)
```

TradingView Desktop 是 Electron（Chromium）應用，所有 Chromium 應用內建 CDP 調試接口。用 `--remote-debugging-port=9222` 啟動 TradingView，即可從 localhost 透過 CDP 讀取並控制其 UI 狀態——**完全不連接 TradingView 伺服器**，所有數據留在本機。

## 78 個 MCP 工具速查

### 讀圖表（起點：先呼叫 `chart_get_state`）

```
chart_get_state          → 取得 symbol/timeframe/所有指標名稱+ID
data_get_study_values    → 讀 RSI/MACD/BB/EMA 當前值（~500B）
quote_get                → 最新 OHLC + 成交量（~200B）
data_get_ohlcv           → K棒數據（用 summary:true 省 context）
```

### 讀 Pine Script 繪圖輸出

```
data_get_pine_lines      → line.new() 水平支撐阻力線
data_get_pine_labels     → label.new() 文字標注 + 價格
data_get_pine_tables     → table.new() 數據表格
data_get_pine_boxes      → box.new() 價格區間
```

**關鍵**：永遠用 `study_filter:"指標名稱"` 指定單一指標，避免讀取全部。

### 控制圖表

```
chart_set_symbol         → 切換標的（AAPL/BTCUSD/ES1! 等）
chart_set_timeframe      → 切換週期（1/5/15/60/D/W/M）
chart_set_type           → K線/HeikinAshi/折線/磚形
chart_manage_indicator   → 加減指標（需全名："Relative Strength Index"）
chart_scroll_to_date     → 跳轉日期
pane_set_layout          → 多窗格布局（2x2/3x1 等）
```

### Pine Script 開發循環

```
pine_set_source      → 1. 注入程式碼
pine_smart_compile   → 2. 編譯（自動偵錯）
pine_get_errors      → 3. 讀錯誤
pine_get_console     → 4. 讀 log.info() 輸出
pine_save            → 5. 儲存到雲端
```

### 其他

```
replay_start/step/trade  → 歷史回放練習
draw_shape               → 趨勢線/水平線/矩形/文字
alert_create/list/delete → 價格警報管理
capture_screenshot        → 截圖（full/chart/strategy_tester）
batch_run                → 批次掃描多標的/多週期
tv stream quote/bars/values/all → 串流輪詢（JSONL 輸出）
```

## CLI 雙通道

每個 MCP 工具也有 `tv` CLI 命令，JSON 輸出可 pipe 到 `jq`：

```bash
tv quote                           # 當前價格
tv symbol AAPL                     # 切換標的
tv ohlcv --summary                 # 價格摘要
tv screenshot -r chart             # 截圖
tv pine compile                    # 編譯 Pine Script
tv stream quote | jq '.close'      # 監控收盤價串流
tv pane layout 2x2                 # 4 圖格局
```

## 上下文最佳化

| 設計 | 節省方式 |
|------|---------|
| Pine lines | 只回傳去重後的價格水位，不含每條線的物件 |
| Pine labels | 每個指標最多 50 條，只含文字 + 價格 |
| OHLCV summary | 統計摘要 + 最後 5 棒（非全量）|
| `study_filter` | 指定單一指標取代掃描全部 |

「分析圖表」完整工作流：~5–10KB context（vs 無最佳化的 ~80KB）

## 法律風險（重要）

| 風險 | 說明 |
|------|------|
| **ToS 灰色地帶** | TradingView ToS 限制自動化數據採集，可能導致帳號被封 |
| **內部 API 不穩定** | 使用未文件化的 Electron 內部接口，TradingView 任何更新都可能讓工具失效 |
| **不得自動交易** | 官方定位為研究/個人工作流工具，不得用於自動化交易決策 |
| **需付費訂閱** | TradingView Desktop 付費訂閱為前提條件 |

## 與富途 Skills 的定位對比

| 維度 | TradingView MCP Bridge | [[富途OpenD Skills整合\|富途 OpenD Skills]] |
|------|----------------------|------------------------------------------|
| 數據來源 | TradingView Desktop（本地 CDP）| 富途 OpenD（本地代理 → 富途伺服器）|
| 交易執行 | ❌ 不可（圖表操作層）| ✅ 可下單（模擬/真實）|
| Pine Script | ✅ 完整開發環境 | ❌ 無 |
| 市場 | TradingView 訂閱覆蓋的所有市場 | 港/美/A 股/新加坡/日本 |
| 法律風險 | 中（ToS 灰色地帶）| 低（官方授權）|
| 安裝難度 | 中（需啟動 CDP 模式）| 低（官方一鍵安裝）|

**互補使用**：TradingView MCP 做圖表分析 + Pine Script 開發 → 富途 Skills 執行下單。

## 安裝快速參考

```bash
git clone https://github.com/tradesdontlie/tradingview-mcp.git
cd tradingview-mcp && npm install

# 啟動 TradingView（Mac）
./scripts/launch_tv_debug_mac.sh   # 自動帶 --remote-debugging-port=9222

# MCP 設定（~/.claude/.mcp.json）
{
  "mcpServers": {
    "tradingview": {
      "command": "node",
      "args": ["/path/to/tradingview-mcp/src/server.js"]
    }
  }
}
```

## 相關頁面

- [[tradesdontlie]]
- [[Claude MCP 伺服器整合指南]]
- [[富途OpenD Skills整合]]
- [[Vibe Coding程式交易實戰]]
- [[Claude Code量化交易四層系統]]
- [[Claude股市期貨分析實戰]]
