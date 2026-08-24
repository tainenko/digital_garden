---
title: "富途 OpenD AI 整合指南：用自然語言操作券商 API"
type: source-summary
tags: [futu, opend, claude-code, skills, trading-api, quantitative-trading, hk-stock]
created: 2026-06-26
updated: 2026-06-26
sources: [futunn-opend-ai-integration]
---

# 富途 OpenD AI 整合指南：用自然語言操作券商 API

## Origin

- **URL**：https://openapi.futunn.com/futu-api-doc/hk/intro/ai.html
- **平台**：富途證券（Futu / 富途牛牛）官方 API 文件
- **版本**：Futu API Doc v10.8
- **摘要**：官方提供 Claude Code Skills 標準的技能包，讓 AI Agent 可透過自然語言直接呼叫富途 OpenD 進行行情查詢、交易下單、策略回測

## Key Takeaways

- **富途官方提供 Claude Code Skills**：直接支援 [[Claude Code內部運作機制|Claude Code]] Skills 標準，是台港市場券商中少見的 AI-native 整合方案
- **兩個技能模組**：`futuapi`（行情交易主體）和 `install-futu-opend`（環境安裝輔助），合計打包為 `opend-skills.zip`
- **futuapi 技能覆蓋範圍**：14 個行情腳本 + 8 個交易腳本 + 5 個訂閱腳本，涵蓋快照/K線/盤口/逐筆/分時/選股/下單/持倉/資金/實時推送
- **多平台支援**：Claude Code、Cursor、VS Code（Cline）、OpenClaw 均可安裝使用
- **安全預設**：交易操作預設走模擬盤，真實下單需明確二次確認——防止 Agent 誤操作
- **速率限制**：下單頻率上限 15 筆/30 秒；訂閱上限 100–2000 標的（依帳戶等級）

## 兩大技能模組

### futuapi — 行情交易助手

| 類別 | 功能 |
|------|------|
| 行情快照 | 港美 A 股即時報價、漲跌幅、成交量 |
| K 線數據 | 歷史與即時 K 線（日/週/分鐘等週期） |
| 盤口數據 | 買賣盤十檔報價 |
| 逐筆成交 | Tick-by-tick 交易流水 |
| 分時數據 | 當日走勢圖數據 |
| 條件選股 | 多因子篩選股票（篩選器） |
| 訂單管理 | 下單/撤單/改單（預設模擬盤） |
| 持倉/資金 | 查詢持倉明細、帳戶可用資金 |
| 實時訂閱 | 行情推送、K 線推送 |

### install-futu-opend — 安裝輔助

- 互動式平台選擇（港/美/A 股）
- OS 自動偵測
- 自動下載並配置 OpenD 本地網關
- SDK 升級引導

## 安裝方式

**一鍵安裝**（推薦）：複製官方提供的安裝指令文字，貼到 AI Agent 的對話框，Agent 自動下載並設定

**手動安裝**：
```
下載：https://openapi.futunn.com/skills/opend-skills.zip
解壓後將 skills/ 目錄複製到：
  Claude Code / VS Code / Cursor：~/.claude/skills/
  專案層級：{專案根目錄}/.claude/skills/
  OpenClaw：~/.openclaw/skills/
```

**觸發方式**：
- `/futuapi` — 啟動行情交易助手
- `/install-futu-opend` — 啟動安裝輔助
- 自然語言描述（中文）也可自動匹配

## 使用前提

1. 須先手動登入 OpenD 本地網關（OpenD 是富途的本地代理程式，API 透過它與富途伺服器通訊）
2. 交易功能預設為模擬盤
3. 真實下單須在 Agent 確認對話中明確回應「真實交易」

## Contradictions/Tensions

- 文件描述功能豐富，但具體腳本名稱和參數在官方頁面未完整列出，需下載 ZIP 後查看
- Skills 標準依賴 Claude Code Skills 架構，其他 IDE 支援程度可能有落差

## Questions Raised

- opend-skills.zip 內的腳本是否開源？是否可自訂修改？
- 支援的市場範圍？（港股確定，美股/A 股範圍？）
- 與[[Claude Code量化交易四層系統]]中 Alpaca 的整合方式有何不同？（Futu 走本地 OpenD；Alpaca 走雲端 REST API）
