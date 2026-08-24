---
title: karuppiah7890/interview-questions
type: source-summary
tags: [面試, OOP設計, Kubernetes, SRE, HashiCorp, LinkedIn, Go, Python]
created: 2026-05-12
updated: 2026-05-12
sources: [github-karuppiah7890-interview-questions]
---

# karuppiah7890/interview-questions

## Origin

- **URL**: https://github.com/karuppiah7890/interview-questions
- **Author**: karuppiah7890（Karuppiah，軟體工程師）
- **Description**: "Some of the interview questions I have gotten" — 個人收錄的真實面試題庫
- **Languages**: Python 79.5%、Go 20.5%
- **Date Ingested**: 2026-05-12

## Key Takeaways

- 收錄三個面試來源：**HashiCorp 預評測**（OOP Banking System）、**Acresium SRE Lead**（Kubernetes/分散式系統）、**LinkedIn SE**（截圖式題目）
- Banking System 設計題同時出現於 HashiCorp 與 Coinbase，為跨公司重複題型，具高複習優先性
- Go 實作使用 mutex 確保 thread-safe；Python 實作使用 `sortedcontainers.SortedList/SortedDict` 處理時序查詢
- HashiCorp 版本 Level 3 有 **cashback 機制**（提款金額 2% 退款，24 小時後到帳），與 Coinbase 版本的 `cancel_payment` 不同
- HashiCorp 版本 Level 2 同時有 `TOP_ACTIVITY`（交易量）和 `TOP_SPENDERS`（轉出量）兩種排行榜
- Acresium 面試為 **2025 年** Site Reliability Lead Engineer（Distributed Systems）職位，含大量 Kubernetes 操作截圖
- LinkedIn 面試為 **2025 年** Software Engineer 截圖式筆試（題目以 PNG 截圖保存）

## Entities Mentioned

- [[karuppiah7890]] — 題庫作者，記錄自身面試經歷

## Concepts Mentioned

- [[HashiCorp_Banking_System_OA]] — HashiCorp 預評測 OOP 設計，4 層遞進題目
- [[Acresium_Kubernetes_SRE_面試題]] — Acresium 2025 SRE Lead 面試，K8s + 分散式系統
- [[Banking_System]] — Coinbase 版本，與 HashiCorp 版本高度重疊
- [[Low Level Design OOD設計題型]] — OOP/LLD 設計題通用框架

## Contradictions/Tensions

- Coinbase HA 版本（已在 wiki）的 Level 3 是 `schedule_payment + cancel_payment`；HashiCorp 版本 Level 3 是 `scheduled_transfer（24h 到期）+ cashback`，兩者結構相似但 API 細節不同
- 同一道 Banking System 題目在不同公司有微差異，面試時需確認版本規格

## Questions Raised

- Acresium Kubernetes 面試的具體考題為截圖，未有文字整理；是否值得 OCR 解析？
- LinkedIn 筆試題目也是截圖，題型未知——是否為 LeetCode 型演算法題？
