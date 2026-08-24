---
title: Curve Finance
type: entity
tags: [DeFi, AMM, 穩定幣, 流動性, 加密貨幣]
created: 2026-05-22
updated: 2026-05-22
sources: [quant67-14-crypto-strategies]
---

# Curve Finance

## 概述

Curve 是 DeFi 中**穩定幣和同類資產（LST、wBTC）流動性**的核心基礎設施。使用 StableSwap 改良 AMM 公式，在資產維持近似比值時提供極低滑點的兌換。

## 核心機制

- **StableSwap 公式**：結合恆定和（x+y=k）與恆定乘積（xy=k），在中間區域提供近恆定匯率
- **交易手續費**：0.04%（vs Uniswap V2/V3 的 0.05-1%）
- **CRV Token**：平台治理代幣，LP 可獲 CRV 激勵
- **veCRV**：鎖倉 CRV 最長 4 年，獲得 Gauge 投票權和協議手續費

## Gauge Weight 系統

veCRV 持有者每週投票決定哪些流動性池獲得多少 CRV 激勵。這創造了「賄賂市場」——協議付費讓 veCRV 投票者把票給自己的池子，以獲取更多流動性（詳見 [[Curve-Convex收益策略]]）。

## 重要事故

- **2023年7月**：reentrancy 漏洞，Curve 自身池損失 ~$70M（波及 JPEG'd、Alchemix、Metronome）

## 生態地位

- DeFi 中穩定幣兌換的**主要流動性來源**（尤其 USDC/USDT/FRAX/DAI）
- LST 生態（stETH/rETH/cbETH）的主要配對流動性
- [[Convex Finance]] 聚合了大量 Curve LP 倉位和 veCRV 投票權

## 相關概念

- [[Curve-Convex收益策略]] — 策略層實作
- [[無常損失與AMM流動性提供]] — AMM 基礎原理
- [[Convex Finance]] — Curve 的最大流動性聚合器
