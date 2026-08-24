---
title: Fireblocks
type: entity
tags: [加密貨幣, 機構, 安全, 離交易所結算, 基礎設施]
created: 2026-05-22
updated: 2026-05-22
sources: [quant67-14-crypto-strategies]
---

# Fireblocks

## 概述

Fireblocks 是面向機構的**數字資產基礎設施平台**，提供私鑰管理（MPC 技術）、資產轉帳、以及**離交易所結算網絡（Off-Exchange Settlement Network）**。

## 核心功能：離交易所結算

跨交易所套利（搬磚）最大的障礙是鏈上確認延遲（BTC ~60分鐘、ETH ~3-6分鐘）。Fireblocks 通過以下方式消除這個障礙：

- 主流 CEX（Binance、OKX 等）直接整合 Fireblocks 結算層
- **資金無需在鏈上移動**，通過 Fireblocks 的 ILP（Inter-Ledger Protocol）在賬本記錄轉移
- 幾乎零延遲的結算，使機構能夠執行鏈上用戶無法完成的高頻跨交易所套利

## 競爭者

- **Copper**（英國）：類似定位，歐洲機構市場份額較高
- 兩者共同形成機構級套利的「護城河」，普通個人幾乎無法與之競爭

## 其他功能

- **MPC 私鑰管理**：無單點故障的分散式密鑰管理
- **策略合規工作流**：AML 篩查、交易審批流程
- **API 整合**：支援主流 DeFi 協議的授權管理

## 相關概念

- [[加密量化基礎設施]] — 機構級安全架構
- [[跨鏈橋風險]] — Fireblocks 可部分替代橋接需求
- [[加密量化投資組合配置]] — 機構競爭格局影響散戶搬磚可行性
