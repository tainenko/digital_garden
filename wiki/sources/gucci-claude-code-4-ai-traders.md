---
title: 我用 Claude Code 一個下午做出 4 個 AI 交易員（不用寫程式）
type: source-summary
tags: [gucci, claude-code, algorithmic-trading, alpaca, options, insider-trading]
created: 2026-05-06
updated: 2026-05-06
sources: [gucci-claude-code-4-ai-traders]
---

# 我用 Claude Code 一個下午做出 4 個 AI 交易員（不用寫程式）

## Origin

- **作者**：[[追日Gucci]]（AI效率革命聯盟）
- **發布日期**：2026-04-25
- **影片日期**：2026-04-24，片長 58:30
- **URL**：https://www.guccidgi.com/2026/04/claude-code-4-ai/
- **類型**：YouTube 教學影片 + 文字整理

## Key Takeaways

1. **三大鴻溝**：散戶 vs 法人的差距是資訊差、紀律差、方便差——三者都能用 Claude Code + Alpaca 解決
2. **Level 1（5分鐘）**：Alpaca paper trading 帳戶 + API Key → 自然語言下單，建立「指令→執行」管道
3. **Level 2（底撈機器人）**：NVDA 跌幅分級買入（1%/2%/3%/5%+）+ Trailing Stop 出場 + Claude Code Routines 全自動排程
4. **Level 3a（內線追蹤）**：SEC Form 4 資料 → 解析高管買賣 → 標記多人同日同向操作
5. **Level 3b（國會交易追蹤）**：Capitol Trades → 計算議員績效 → 回測最強交易員策略
6. **Level 4a（方向性選擇權掃描）**：趨勢判斷 + IV 高低選 Credit/Debit Spread，自動篩選期望值與勝率
7. **Level 4b（財報 IV Crush Iron Condor）**：財報前 IV 飆升，財報後 IV 暴跌，賣出鐵鷹式收取 premium
8. **完整自動化管道**：Alpaca 執行 + Claude Code Routines 排程 + Telegram Bot 推播，交易員在工作/睡眠中被動接收訊號
9. **工具最少化**：整個系統只用 Claude Code（Auto/Opus/Sonnet 4.6）+ Alpaca + Yahoo Finance，無需特殊訂閱
10. **教育性申明**：所有策略為技術示範，非投資建議，需獨立驗證後依自身風險承受度決策

## Entities Mentioned

- [[追日Gucci]] — 作者
- [[Alpaca]] — API-first 券商，支援 Paper Trading，本文核心執行平台
- [[Anthropic]] — Claude Code 開發者

## Concepts Mentioned

- [[Claude Code量化交易四層系統]] — 本文主要新概念
- [[選擇權交易策略]] — Iron Condor / IV Crush / Credit-Debit Spread / Trailing Stop
- [[Vibe Coding程式交易實戰]] — 相關護欄策略（前文）
- [[Claude股市期貨分析實戰]] — 分析工具角度的互補

## Contradictions / Tensions

- 影片強調「不用寫程式」，但實際上用戶需要理解 API Key 設定、Paper Trading 機制、選擇權 Greeks（Delta）等概念，並非完全零門檻
- Iron Condor 的 PoP 僅 40%（Max Loss $3,100 vs Max Profit $369），風報比相對不利，文中未深度討論
- 國會交易追蹤策略建議「獨立驗證」——暗示直接跟隨有效性存疑

## Questions Raised

- Trailing Stop 在 Alpaca 的實際執行細節（市價還是限價觸發）？
- Claude Code Routines 的排程如何在市場收盤時自動暫停？
- SEC Form 4 資料的延遲問題（申報截止 2 個工作日），是否足夠 actionable？
- OpenClaw APP 的具體功能與 Claude Code 的互補關係？
