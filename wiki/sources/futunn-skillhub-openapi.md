---
title: "富途 Skill Hub：futuapi 官方技能頁"
type: source-summary
tags: [futu, opend, claude-code, skills, trading-api, quantitative-trading]
created: 2026-06-26
updated: 2026-06-26
sources: [futunn-skillhub-openapi]
---

# 富途 Skill Hub：futuapi 官方技能頁

## Origin

- **URL**：https://www.futunn.com/hk/skillhub/openapi
- **平台**：富途牛牛香港官方 Skill Hub（技能市場頁）
- **性質**：`futuapi` 技能的官方展示頁，含精確的 API 數量與功能分類

## Key Takeaways

- **56 個 API 介面**：這是目前最精確的數字，分為行情 35 個 + 交易 14 個 + 推送 7 種
- **與前兩篇文件的數字差異**：先前 API 文件說「14 行情 + 8 交易 + 5 訂閱腳本」，這裡是「35 行情 + 14 交易 + 7 推送」——前者是高層腳本數量，後者是底層 API 介面數量（更細粒度）
- **MACD 策略回測**：官方範例包含「寫一個 MACD 金叉買入、死叉沽出的策略，回測 TSLA 過去 3 年的表現」——說明 Skills 可用於策略回測，不只是即時查詢與下單
- **3 分鐘安裝、零程式碼**：官方定位是無需技術背景的投資者也能使用
- **本地加密傳輸**：透過富途牛牛 OpenD 網關本地加密，不過第三方伺服器

## 精確 API 數量

| 類別 | 數量 | 涵蓋功能 |
|------|------|---------|
| 行情（市場數據） | 35 個介面 | 快照、K線、盤口、逐筆、選股、板塊分析、資金流向 |
| 交易 | 14 個介面 | 下單/撤單、持倉、帳戶資金、交易歷史、模擬交易 |
| 推送通知 | 7 種 | 價格推送、K線更新、逐筆成交、價格提醒、訂單狀態通知 |
| **合計** | **56 個** | |

## 官方自然語言範例

```
查看騰訊（0700）的陰陽燭
```
```
用模擬帳戶買入 100 股蘋果
```
```
寫一個 MACD 金叉買入、死叉沽出的策略，回測 TSLA 過去 3 年的表現
```

第三個範例尤其重要：說明 Skills 已可支援**策略程式碼生成 + 歷史回測**，而非只是即時查詢。

## 安裝

支援平台：OpenClaw、Claude Code、Cursor  
安裝文件：https://openapi.futunn.com/futu-api-doc/hk/intro/ai.html

## 數字差異說明

| 來源 | 行情 | 交易 | 推送/訂閱 | 備註 |
|------|------|------|----------|------|
| API 文件頁（第一篇） | 14 腳本 | 8 腳本 | 5 腳本 | 高層腳本/功能模組數量 |
| Skill Hub 展示頁（本篇）| 35 介面 | 14 介面 | 7 種 | 底層 API 介面數量（更細粒度）|

兩者並不矛盾：一個高層腳本可能封裝多個底層 API 介面。

## Contradictions/Tensions

- 「回測」功能需要歷史數據 API + 本地計算，Skill 是否自帶回測引擎或只是生成回測程式碼由 Claude 執行？需實際測試確認
- MACD 範例中提到「過去 3 年」，但 K 線 API 的歷史數據深度限制未在此頁說明

## Questions Raised

- 35 個行情介面的完整清單？（官方文件是否有詳細列表？）
- 回測功能是 Skills 自帶，還是 Claude 生成 Python 腳本後本地執行？
- Skill Hub 是否有其他第三方 Skills，還是目前只有官方 `futuapi`？

## Concepts Mentioned

- [[富途OpenD Skills整合]]
- [[富途 Futu]]
