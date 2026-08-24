---
title: 富途 Futu
type: entity
tags: [broker, hk-stock, trading-api, opend, claude-code, quantitative-trading]
created: 2026-06-26
updated: 2026-06-26

sources: [futunn-opend-ai-integration, futuhk-futu-skills-intro]
---

# 富途 Futu

## 基本資料

- **全名**：富途控股（Futu Holdings）
- **產品**：富途牛牛（Futubull）— 港美股及 A 股的線上券商 App
- **API 平台**：OpenD — 本地代理網關，開發者透過它存取富途行情與交易 API
- **API 文件**：https://openapi.futunn.com/futu-api-doc/
- **市場覆蓋**：港股、美股、A 股、新加坡、日本（五大市場）

## OpenD AI 整合

富途官方提供符合 [[Claude Code內部運作機制|Claude Code Skills]] 標準的技能包，是台港市場券商中少見的 **AI-native 整合方案**：

- **技能包下載**：https://openapi.futunn.com/skills/opend-skills.zip
- **包含技能**：`futuapi`（行情交易）+ `install-futu-opend`（環境安裝）
- **支援平台**：Claude Code、Cursor、VS Code（Cline）、OpenClaw

### Skills 全覽

| Skill | 中文名 | 狀態 |
|-------|--------|------|
| `futuapi` | 行情交易主體（**56 個 API 介面**：35 行情 + 14 交易 + 7 推送）| ✅ 已上線 |
| `install-futu-opend` | OpenD 安裝輔助 | ✅ 已上線 |
| `futu-news-search` | 資訊搜尋（新聞/公告/研報）| ✅ 已上線 |
| `futu-stock-digest` | 個股解讀（結構化事件摘要）| ✅ 已上線 |
| `futu-comment-sentiment` | 情緒溫度計（牛牛圈多空分析）| ✅ 已上線 |
| `futu-technical-signal` | 技術面異動（MACD/KDJ/RSI）| 🔜 即將推出 |
| `futu-capital-signal` | 資金面異動（主力資金流向）| 🔜 即將推出 |
| `futu-derivative-signal` | 衍生工具異動（期權/認股證）| 🔜 即將推出 |

**委託類型**：13 種（限價、市價、止損、追蹤止損、競價等）

## 對比 Alpaca

[[Alpaca]] 是美股 API-first 券商（雲端 REST）；富途 OpenD 是港美股本地代理架構：

| 維度 | 富途 OpenD | Alpaca |
|------|-----------|--------|
| 通訊架構 | 本地 OpenD 代理 → 富途伺服器 | 直接呼叫雲端 REST API |
| 市場覆蓋 | 港股、美股、A 股 | 美股為主 |
| AI 整合 | 官方 Claude Code Skills | 需自行包裝 |
| 模擬交易 | 預設模擬盤（需明確啟用真實） | Paper Trading 環境獨立 |
| 速率限制 | 15 筆/30 秒 | 依帳戶等級 |

## 速率與限制

- 下單：15 筆 / 30 秒
- 訂閱標的：100–2000（依帳戶等級）

## 相關頁面

- [[富途OpenD Skills整合]]
- [[Claude Code量化交易四層系統]]
- [[Alpaca]]
- [[Claude Code內部運作機制]]
