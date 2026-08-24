---
title: Claude Code 量化交易四層系統
type: concept
tags: [claude-code, algorithmic-trading, alpaca, options, automation, taiwan]
created: 2026-05-06
updated: 2026-05-06
sources: [gucci-claude-code-4-ai-traders]
---

# Claude Code 量化交易四層系統

[[追日Gucci]] 示範的完整框架：用 Claude Code + [[Alpaca]] 在一個下午建立四個自動化 AI 交易員，不需要寫程式。

## 核心主張：三大鴻溝

散戶 vs 法人的差距是三道鴻溝——三者都能用 AI 打平：

| 鴻溝 | 法人優勢 | Claude Code 解法 |
|------|---------|----------------|
| 資訊差 | 完整資料管道、量化團隊年分析數百份報告 | Level 3：SEC/Capitol Trades 追蹤工具 |
| 紀律差 | 演算法執行，不受情緒干擾 | Level 2：規則化機器人 |
| 方便差 | 24/7 自動系統 | Level 4：選擇權掃描 + Routines 排程 |

---

## Level 1：券商連線與第一筆下單（5 分鐘）

**工具**：[[Alpaca]]（API-first 券商，支援 Paper Trading）

**所需憑證**：
- API Key
- Secret Key
- API Endpoint（Paper Trading 用虛擬資金跑真實市場資料）

**自然語言指令範例**：
```
「幫我查 NVDA 現在的價格，然後買入 1 股」
「顯示我的帳戶餘額和持倉」
「以市價單買入 1 股 NVIDIA」
```

**意義**：建立「自然語言 → 券商執行」的完整管道，是後三層的基礎。

---

## Level 2：底撈機器人（解決紀律差 + 方便差）

### 策略邏輯：NVDA 跌幅分級買入

```
開盤 30 分鐘後檢查：
跌 1% → 買 1 股
跌 2% → 買 2 股
跌 3% → 買 3 股
跌 5%+ → 最多 4 股（上限）
每日只進場一次（不重複買）
```

### 出場：Trailing Stop（移動停損）

```
買入後每 5 分鐘監控
停損點 = 歷史最高價 - $5
價格跌破停損點 → 自動出場
```

**效果**：上漲時停損點跟著上移（鎖利），下跌時保護已有獲利。

### 自動化：Claude Code Routines

- **進場邏輯**：每日開盤後 30 分鐘執行一次
- **出場監控**：交易時段內每 5 分鐘執行
- **收盤自動暫停**：非交易時段跳過

---

## Level 3：資訊優勢——看到隱藏的牌

### Level 3a：內線交易追蹤（SEC Form 4）

**資料來源**：美國 SEC Form 4 申報（高管買賣須於 2 個工作日內申報）

**Claude Code 建立的工具**：
```
監控大型科技股
解析 Form 4 申報，提取高管交易記錄
標記 CEO、CFO 等 C-suite 人員的買賣
高亮「同日 3 人以上同向操作」的協調訊號
產出 Markdown 報告 + 30 天趨勢視覺化
```

**範例發現**：
- NVIDIA：3 月 18 日，包含 Jensen Huang 在內的 6 位高管同步大量賣出
- 多人協調賣出 = 需要關注的潛在訊號

**限制**：申報有 2 個工作日延遲，不能即時跟單；適合用於趨勢參考而非精確時機。

### Level 3b：國會交易追蹤（Capitol Trades）

**資料來源**：Capitol Trades 網站（依 STOCK Act 申報）

**Claude Code 建立的分析工具**：
```
監控議員交易活動
計算每位議員的報酬率
排行最活躍/績效最佳的交易員
回測排名最高者近一年的交易結果
```

**核心洞察**：不盲目跟單，而是用 AI 驗證任何策略的歷史有效性。

---

## Level 4：選擇權機會雷達（最進階）

### Level 4a：方向性選擇權掃描

**掃描框架**：
```
1. 掃描大型科技股
2. 判斷趨勢方向（上升/下降）
3. IV 高 → 賣方策略（Credit Spread）
   IV 低 → 買方策略（Debit Spread）
4. 到期日：30–45 天
5. 第一腳 Delta ≈ 0.30
6. 第二腳在股價 3% 範圍內
7. 篩選期望報酬率 + 勝率（PoP）
8. 出場目標：收取 premium 的 50%
```

**範例輸出**：Microsoft Debit Call Spread（Buy 430 / Sell 455）
- 期望值：$3.77
- 勝率：49.2%
- 最大獲利：$1,646
- 最大虧損：$854

### Level 4b：財報 IV Crush Iron Condor

**機制**：財報前 IV 飆升 → 財報後 IV 崩潰 → 賣方收取 premium

**四腳結構**：
```
下方：Sell Put（高 strike） + Buy Put（低 strike）
上方：Sell Call（低 strike） + Buy Call（高 strike）
到期日：7–21 天
Delta 中性定位
```

**範例**：Amazon Iron Condor
```
Sell Put 240 / Buy Put 232.5
Sell Call 265 / Buy Call 272.5
收取 Premium：$369（最大獲利）
損益兩平區間：$238–$270
最大虧損：$3,100
勝率：~40%
```

⚠️ 注意風報比：Max Loss $3,100 vs Max Profit $369，勝率需要夠高才能長期正期望值。

---

## 完整自動化管道

```
掃描訊號（Claude Code Routines，每 15 分鐘）
    ↓
篩選符合條件的機會
    ↓
Telegram Bot 推播（含交易細節 + P&L 圖）
    ↓
用戶確認後 → Alpaca 執行四腳組合單
```

**結果**：交易員在工作或睡眠中被動接收訊號，主動決策是否執行。

---

## 工具清單

| 工具 | 功能 |
|------|------|
| Claude Code（Auto/Opus/Sonnet 4.6） | 自然語言 → 工具建立 |
| [[Alpaca]] | 執行下單（Paper + 實盤） |
| Yahoo Finance | 選擇權資料（含 Open Interest） |
| SEC API | Form 4 內線申報資料 |
| Capitol Trades | 國會交易追蹤 |
| Telegram Bot API | 推播通知 |
| Claude Code Routines | 排程與自動化 |

---

## 注意事項

- 所有策略為技術示範，不構成投資建議
- Paper Trading 驗證策略後才考慮實盤
- 選擇權策略複雜度高，需充分理解 Greeks 再執行
- 相關護欄設定見 [[Vibe Coding程式交易實戰]]

## 相關頁面

- [[選擇權交易策略]] — Iron Condor、IV Crush、Trailing Stop 概念詳解
- [[Alpaca]] — API-first 券商
- [[追日Gucci]] — 作者
- [[Vibe Coding程式交易實戰]] — AI 改動交易邏輯的防護措施
- [[Claude股市期貨分析實戰]] — 分析工具角度的互補框架
- [[Boris的Claude Code完整工作流]] — Claude Code Routines 排程的上層設計
