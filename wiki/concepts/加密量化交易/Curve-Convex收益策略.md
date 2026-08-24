---
title: Curve-Convex收益策略
type: concept
tags: [DeFi, Curve, Convex, 穩定幣, 收益策略, 加密量化]
created: 2026-05-22
updated: 2026-05-22
sources: [quant67-14-crypto-strategies]
---

# Curve/Convex 收益策略

## 基礎：StableSwap 模型

[[Curve Finance]] 使用改良的 AMM 公式，針對 1:1 錨定資產（穩定幣、LST、wBTC）大幅降低滑點。相比標準 AMM，相同資本提供更深的流動性，[[無常損失與AMM流動性提供|無常損失]] 極低。

## 多層收益結構

| 層次 | 來源 | 典型年化 |
|------|------|---------|
| 第一層 | Curve 池交易手續費（0.04%） | 1–5% |
| 第二層 | CRV token 激勵（gauge weight 決定） | 5–20% |
| 第三層 | [[Convex Finance]] CVX 獎勵 | 3–10% |
| 第四層 | Bribe 機制（veCRV 投票報酬） | **10–30%** |

**總年化**：在有利條件下可達 30%+，但對 CRV/CVX 幣價高度敏感。

## Bribe 機制（賄賂市場）

- Curve gauge weight 決定哪些池子獲得最多 CRV 激勵
- veCRV（鎖倉 CRV 獲得）持有者有投票權
- 協議付費「賄賂」veCRV 持有者把票投給自己的池子
- Convex 聚合了大量 veCRV 投票權，通過它可高效執行 bribe 策略

## Convex 聚合層

```
用戶 LP Token → 存入 Convex → 收到 cvxCRV
                           ↓
              自動領取 CRV + CVX + 手續費
              免除自己鎖倉和 gauge 管理
```

**優點**：降低操作複雜度，散戶也能享受機構級 gauge 投票權

## 再投資循環

1. 提供 Curve 流動性 → 獲得 CRV
2. 通過 Convex 鎖倉 → 獲得 vlCRV 投票權
3. 出售投票給需要 gauge weight 的協議（bribe 收益）
4. 收到 CVX → 再投入或變現

## 風險

| 風險 | 說明 |
|------|------|
| CRV/CVX 幣價暴跌 | 多層激勵都依賴幣價支撐，熊市時收益大幅萎縮 |
| 智能合約風險 | Curve 歷史上曾有 reentrancy 漏洞（2023年，$70M） |
| 流動性集中風險 | Curve 是 DeFi 穩定幣流動性核心，系統性失敗影響整個生態 |
| 去錨事件 | 2022年 stETH 去錨引發清算瀑布，穩定池不一定穩定 |

## 相關概念

- [[無常損失與AMM流動性提供]] — AMM 基礎原理
- [[加密借貸利率套利]] — 另一種 DeFi 固定收益策略
- [[跨鏈橋風險]] — DeFi 整體生態的系統性風險
- [[加密量化投資組合配置]] — AMM LP 配置 20-30%
