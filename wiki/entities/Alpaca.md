---
title: Alpaca
type: entity
tags: [broker, api, algorithmic-trading, paper-trading, fintech]
created: 2026-05-06
updated: 2026-05-06
sources: [gucci-claude-code-4-ai-traders]
---

# Alpaca

## 基本資訊

- **類型**：API-first 美國股票券商
- **定位**：專為自動化交易設計，非傳統零售券商
- **官網**：alpaca.markets

## 核心特性

### Paper Trading（模擬帳戶）
- 使用**真實市場資料**，帳戶為虛擬資金（預設 $100,000）
- 不需要存入真實資金即可測試策略
- 與實盤 API 完全相同，程式碼切換容易

### API-First 設計
- 三個憑證即可開始：API Key、Secret Key、API Endpoint
- 支援自然語言驅動（透過 Claude Code）
- 適合搭配 LLM / 自動化腳本

## 在 Claude Code 量化系統中的角色

[[Claude Code量化交易四層系統]] 的執行層：

| 功能 | 說明 |
|------|------|
| 帳戶查詢 | 餘額、持倉、歷史交易 |
| 價格查詢 | 即時股票報價 |
| 市價單 | 即時執行買入/賣出 |
| 限價單 | 指定價格執行 |
| 四腳組合單 | Iron Condor 等多腳選擇權策略 |
| Trailing Stop | 移動停損自動出場 |

## 相關頁面

- [[Claude Code量化交易四層系統]] — 完整 4 層 AI 交易系統
- [[選擇權交易策略]] — Alpaca 執行的策略類型
- [[追日Gucci]] — 使用 Alpaca 的示範案例
