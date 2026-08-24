---
title: "富途 Skills 入門指南：讓 AI Agent 幫你炒股"
type: source-summary
tags: [futu, opend, claude-code, skills, trading-api, quantitative-trading, hk-stock, sentiment-analysis]
created: 2026-06-26
updated: 2026-06-26
sources: [futuhk-futu-skills-intro]
---

# 富途 Skills 入門指南：讓 AI Agent 幫你炒股

## Origin

- **URL**：https://www.futuhk.com/blog/detail-futu-skills-102-260475002
- **平台**：富途香港（futuhk.com）官方部落格
- **性質**：面向一般投資者的使用者導向介紹，無需程式基礎

## Key Takeaways

- **Skills = 專業投資能力插件**：賦予 AI Agent 直連富途交易基礎設施 + 機構級市場數據的能力；與富途牛牛內建 AI（固定 Q&A）的關鍵差異在於「完全可自訂」的本地自動化
- **五大市場覆蓋**：港股、美股、A 股、新加坡、日本——比先前文件（只提港美 A 股）多了新加坡和日本
- **13 種委託類型**：限價、市價、止損、追蹤止損、競價等——說明 Skills 對接的是完整交易能力，不只是簡單下單
- **資訊分析三件套已上線**：`futu-news-search`、`futu-stock-digest`、`futu-comment-sentiment` 已可用，是行情 + 情緒分析的組合拳
- **三大訊號技能即將推出**：技術面（MACD/KDJ/RSI 形態）、資金面（主力資金流向）、衍生工具（期權/認股證異動）
- **安全框架四層保護**：本地執行（不過第三方伺服器）/ 逐筆確認 / 細粒度權限（可只開只讀）/ 機構級風控
- **安裝無需寫程式**：一條指令完成安裝，自然語言操作；安裝觸發語句：`根據指引安裝 Futu Skills：https://www.futunn.com/skills/futu-install.md`

## 六個具名 Skills

| Skill 名稱 | 中文名 | 狀態 | 功能 |
|-----------|--------|------|------|
| `futu-news-search` | 資訊搜尋 | ✅ 已上線 | 搜尋富途平台上的新聞、公告、研究報告 |
| `futu-stock-digest` | 個股解讀 | ✅ 已上線 | 結構化摘要：重要事件 + 市場解讀 + 來源連結 |
| `futu-comment-sentiment` | 情緒溫度計 | ✅ 已上線 | 分析牛牛圈討論的多空情緒與熱度 |
| `futu-technical-signal` | 技術面異動 | 🔜 即將推出 | MACD/KDJ/RSI 形態識別買賣訊號 |
| `futu-capital-signal` | 資金面異動 | 🔜 即將推出 | 主力資金分佈、沽空活動、資金流向追蹤 |
| `futu-derivative-signal` | 衍生工具異動 | 🔜 即將推出 | 期權持倉變化、隱波異動、認股證訊號 |

（另有 `futuapi` 行情交易主體和 `install-futu-opend` 安裝輔助，見[[futunn-opend-ai-integration]]）

## 自然語言範例

| 場景 | 範例指令 |
|------|---------|
| 新聞查詢 | "Tesla 最近有咩新聞？" |
| 個股解讀 | "提供一份 Tesla 的最新解讀" |
| 情緒查詢 | "牛牛圈而家睇好定睇淡比亞迪？" |
| 下單 | "幫我以 190 美元限價買入 50 股" |

## 三步工作流

1. **多維度分析**：一條指令觸發個股解讀 + 資訊搜尋 + 情緒分析 + 技術訊號的綜合報告
2. **執行**：在同一對話中下達買賣指令，AI 準備好但每筆交易仍需用戶手動確認
3. **自動化**：排程每日報告、設定條件提醒、批次處理自選股——在本地運行，不需手動開 App

## 安全框架詳述

- **本地執行**：所有數據經 OpenD 處理，不經過第三方伺服器
- **逐筆確認**：AI 無法獨立完成交易，每筆都需要用戶明確授權
- **細粒度權限**：可只開啟唯讀 Skills（不授予下單能力）
- **機構級風控**：與富途牛牛 App 內交易相同的風控保護

## Contradictions/Tensions

- 這篇文件比[[futunn-opend-ai-integration|OpenD AI 整合指南]]多出「新加坡、日本」兩個市場，兩者說法不完全一致，實際支援範圍需以最新官方文件為準
- 六個具名 Skills 與先前文件的「14 行情 + 8 交易 + 5 訂閱腳本」分類體系不同——前者是功能模組維度，後者是腳本數量維度

## Questions Raised

- `futu-comment-sentiment` 分析的牛牛圈是港股為主？還是也涵蓋美股討論？
- 三大即將推出的訊號技能（技術/資金/衍生工具）上線時間表？
- 可否將多個 Skills 串接成自動化 pipeline，例如「技術訊號觸發 → 情緒確認 → 自動生成交易建議」？

## Entities Mentioned

- [[富途 Futu]] — 技能提供方
- [[Anthropic]] — Claude Code 底層

## Concepts Mentioned

- [[富途OpenD Skills整合]] — 技術架構
- [[Claude股市期貨分析實戰]] — 相關使用情境
- [[Vibe Coding程式交易實戰]] — 同類自動化交易方法論
