---
title: 加密貨幣量化交易策略全攻略（quant67）
type: source-summary
tags: [加密貨幣, 量化交易, DeFi, 套利, AMM, 資金費率, 跨鏈橋, 風險管理]
created: 2026-05-22
updated: 2026-05-22
sources: [quant67-14-crypto-strategies]
---

# 加密貨幣量化交易策略全攻略（quant67）

## Origin

- **Title**: 14個主流加密貨幣量化交易策略
- **Author**: quant67.com
- **URL**: https://quant67.com/post/quant/14-crypto-strategies/14-crypto-strategies.html
- **Date Ingested**: 2026-05-22
- **Audience**: 量化交易工程師團隊（非散戶）

## Key Takeaways

- **六大策略類別**：資金費率套利、跨交易所套利（搬磚）、三角與跨鏈套利、AMM 流動性提供、Curve/Convex 收益策略、借貸利率套利——外加流動性挖礦風險評估
- **資金費率套利**是最成熟的 delta-neutral 策略，年化 8-30%，建議配置 30-40%，核心工具：long spot + short perp
- **無常損失**是 AMM LP 的最大隱患，公式 IL(r) = 2√r/(1+r) - 1，Uniswap V3 集中流動性在經濟上等同「賣出波動率」
- **跨鏈橋已損失超 $2.5B**（2021-2024）：Wormhole $320M、Ronin $625M、Nomad $190M、Multichain $126M；橋接風險需視為配置上限，而非可回避的技術問題
- **策略衰減快**：加密因子半衰期 6-12 個月（vs 股票 5+ 年），需持續研發
- **容量與報酬反比**：最高收益策略（MEV、小幣套利）容量僅數百萬美元；主流策略（BTC 資金費率）容量 10 億美元級但報酬較低
- **「生存本身即 alpha」**：避免交易所倒閉、橋接被盜、清算瀑布比追求邊際 bps 更重要
- **24/7 運維**：自動 kill-switch 是必要安全措施，失控進程可在人類醒來前蒸發資金
- **千億美元規模所需人力**：DevOps 2-3、安全/冷錢包 2、合規/AML 2、研究/策略 4-6、執行/做市 3-5

## Entities Mentioned

- [[Curve Finance]] — StableSwap AMM，穩定幣交易的核心基礎設施
- [[Convex Finance]] — Curve LP 聚合器，CVX reward wrapping
- [[Fireblocks]] — 機構離交易所結算網絡，消除搬磚的鏈上確認延遲
- [[ccxt]] — Python/JS 加密交易所統一 API 庫

## Concepts Mentioned

- [[資金費率套利]] — long spot + short perp 的 delta-neutral 策略
- [[無常損失與AMM流動性提供]] — AMM LP 經濟學與 IL 公式
- [[Curve-Convex收益策略]] — 多層 yield 疊加：手續費 + CRV + CVX + bribes
- [[加密借貸利率套利]] — 跨協議借貸利差捕獲
- [[跨鏈橋風險]] — 橋接協議安全事故歷史與風險計量
- [[加密量化投資組合配置]] — 風險預算分配框架與硬性上限
- [[加密量化基礎設施]] — 工程技術棧與團隊配置

## Contradictions/Tensions

- 文章建議搬磚（跨交易所套利）配置僅 10%，與許多散戶量化策略的高度聚焦相反——機構已用 Fireblocks/Copper 離交易所結算徹底改變競爭格局
- Curve/Convex 的多層 yield（尤其 bribe 機制 10-30%）在 token 價格下行時可能迅速崩潰；文章警告但沒有量化 CRV/CVX 的尾部風險場景

## Questions Raised

- MEV 策略（未涵蓋）：Flashbots/私有 mempool 在 2025-2026 的競爭格局如何？
- stETH 在 2022 去錨後，LST/LRT 套利的復甦程度如何？
- 隨著 CEX 合規化（MiCA/SFC），機構搬磚的門檻是否進一步提高？
