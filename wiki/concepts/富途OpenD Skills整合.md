---
title: 富途 OpenD Skills 整合
type: concept
tags: [futu, opend, claude-code, skills, trading-api, quantitative-trading]
created: 2026-06-26
updated: 2026-06-26
sources: [futunn-opend-ai-integration, futuhk-futu-skills-intro, futunn-skillhub-openapi]
---

# 富途 OpenD Skills 整合

## 是什麼

[[富途 Futu]] 官方提供的 [[Claude Code內部運作機制|Claude Code Skills]] 技能包，讓 AI Agent 可透過自然語言或斜線指令直接操作富途 OpenD，進行港美股行情查詢、下單交易與策略開發。

技能包下載：`https://openapi.futunn.com/skills/opend-skills.zip`

## 架構

```
用戶自然語言 / /futuapi 指令
        ↓
  Claude Code（技能路由）
        ↓
  futuapi Skill（Python/Shell 腳本）
        ↓
  本地 OpenD 網關（須先手動登入）
        ↓
  富途伺服器（行情 + 交易）
```

OpenD 是必要的本地代理——所有 API 呼叫都經過它，開發者無法繞過直接連富途伺服器。

## Skills 全覽

### 已上線

| Skill | 斜線指令 | 功能概要 |
|-------|---------|---------|
| `futuapi` | `/futuapi` | 行情交易主體（**56 個 API 介面**：35 行情 + 14 交易 + 7 推送，13 種委託類型） |
| `install-futu-opend` | `/install-futu-opend` | OpenD 安裝輔助（OS 偵測、自動下載、SDK 升級）|
| `futu-news-search` | 自然語言觸發 | 搜尋富途平台新聞、公告、研報。例：「Tesla 最近有咩新聞？」 |
| `futu-stock-digest` | 自然語言觸發 | 個股結構化摘要：重要事件 + 市場解讀 + 來源連結 |
| `futu-comment-sentiment` | 自然語言觸發 | 分析牛牛圈討論的多空情緒與討論熱度 |

### 即將推出

| Skill | 中文名 | 功能 |
|-------|--------|------|
| `futu-technical-signal` | 技術面異動 | MACD/KDJ/RSI 形態識別買賣訊號 |
| `futu-capital-signal` | 資金面異動 | 主力資金分佈、沽空活動、資金流向追蹤 |
| `futu-derivative-signal` | 衍生工具異動 | 期權持倉變化、隱波異動、認股證訊號 |

## 安裝

**一鍵安裝**（推薦，無需程式基礎）——貼到 Agent 對話框：
```
根據指引安裝 Futu Skills：https://www.futunn.com/skills/futu-install.md
```

**手動安裝**：
```bash
curl -O https://openapi.futunn.com/skills/opend-skills.zip
unzip opend-skills.zip
cp -r skills/ ~/.claude/skills/   # Claude Code / Cursor / VS Code
```

安裝後流程：啟動 OpenD（手動登入）→ `/futuapi` 或自然語言開始對話。

## 重要限制與安全設計

| 限制 | 詳情 |
|------|------|
| 預設模擬盤 | 交易操作預設走模擬帳戶，需明確確認才執行真實下單 |
| 下單速率 | 15 筆 / 30 秒上限 |
| 訂閱上限 | 100–2000 標的（依帳戶等級） |
| OpenD 必須先登入 | Agent 無法自動登入，須用戶手動先開啟 OpenD |

**四層安全框架**：
1. 本地執行——所有數據經 OpenD，不過第三方伺服器
2. 逐筆確認——AI 無法獨立完成交易，每筆需用戶明確授權
3. 細粒度權限——可只開啟唯讀 Skills，不授予下單能力
4. 機構級風控——與富途牛牛 App 相同的風控保護

**預設模擬盤**是關鍵安全設計——防止 Agent 在生產帳戶誤操作，與 [[Claude Code量化交易四層系統]] 中推薦的「先 Paper Trading 驗證」原則一致。

## 與 Alpaca 方案比較

| 維度 | 富途 OpenD Skills | Alpaca + Claude Code |
|------|------------------|----------------------|
| 官方支援 | ✅ 官方提供 Skills | ❌ 需自行包裝 API |
| 市場 | 港股、美股、A 股、新加坡、日本 | 美股 |
| 通訊 | 本地 OpenD 代理 | 雲端 REST |
| 延遲 | 較低（本地代理） | 網路 RTT |
| 開箱即用 | ✅ 高 | 中（需自訂） |

## 在量化工作流中的位置

結合 [[Claude Code量化交易四層系統]] 的框架：

- **Level 1（券商連線）**：OpenD Skills 直接解決這層——行情訂閱 + 下單執行都有現成腳本
- **Level 2（訊號生成）**：`futu-technical-signal`、`futu-capital-signal`、`futu-derivative-signal` 上線後可覆蓋此層；`futuapi` 本身也支援策略回測（官方範例：「寫 MACD 金叉策略，回測 TSLA 過去 3 年」）
- **Level 3–4**（策略邏輯 + 風控）：仍需自行開發；但情緒分析（`futu-comment-sentiment`）可作為 Level 3 的確認層

## 相關頁面

- [[富途 Futu]]
- [[Claude Code量化交易四層系統]]
- [[Claude股市期貨分析實戰]]
- [[Vibe Coding程式交易實戰]]
- [[Alpaca]]
