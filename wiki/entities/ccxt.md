---
title: ccxt
type: entity
tags: [加密貨幣, 量化交易, 開源工具, Python, JavaScript]
created: 2026-05-22
updated: 2026-05-22
sources: [quant67-14-crypto-strategies]
---

# ccxt

## 概述

**ccxt**（CryptoCurrency eXchange Trading）是 Python/JavaScript/PHP 的開源加密交易所統一 API 庫，支援 100+ 交易所的標準化接口。

## 主要用途

- 以統一 API 訪問多個 CEX 的行情、下單、帳戶數據
- 跨交易所資金費率比較（[[資金費率套利]] 的核心工具）
- 跨交易所套利（搬磚）的報價抓取
- 回測時的歷史 OHLCV 數據拉取

## 代碼示例（資金費率掃描）

```python
import ccxt

exchange_ids = ['binance', 'okx', 'bybit', 'gate']
for eid in exchange_ids:
    ex = getattr(ccxt, eid)()
    markets = ex.fetch_funding_rates()
    for symbol, info in markets.items():
        rate = info['fundingRate']
        annualized = rate * 3 * 365  # 每8小時 → 年化
        if annualized > 0.10:  # 年化 > 10% 才值得考慮
            print(f"{eid} {symbol}: {annualized:.1%}")
```

## 局限性

- 統一接口犧牲了各交易所的特有功能（如 Binance 的 Portfolio Margin）
- WebSocket 支援品質因交易所而異
- 機構級延遲要求（HFT）需改用交易所原生 SDK 或直連 co-location

## 相關概念

- [[資金費率套利]] — ccxt 的主要應用場景
- [[加密量化基礎設施]] — 完整工程棧中的 API 層
