---
title: PyMC Labs
type: entity
tags: [company, bayesian, probabilistic-ml, llm-application, market-research]
created: 2026-06-16
updated: 2026-06-16
sources: [maier-ssr-llm-purchase-intent]
---

# PyMC Labs

貝氏 / 機率機器學習顧問公司，由 [[Thomas Wiecki]] 創辦，背後與開源專案 **PyMC**（Python 機率程式設計庫）關係密切。近年將機率方法延伸到 **LLM 應用 / 合成消費者研究**。

## Key Facts
- 與 **Colgate-Palmolive** 合作發表論文《LLMs Reproduce Human Purchase Intent via Semantic Similarity Elicitation of Likert Ratings》（arXiv:2510.08338，2025-10）。
- 提出 [[語意相似度評分SSR]]（Semantic Similarity Rating），用 embedding 相似度把 LLM 自由文字映射成 Likert 分布。
- 開源官方實作：https://github.com/pymc-labs/semantic-similarity-rating

## 研究與產品方向
- **合成消費者調查模擬**：用 LLM 取代/補充昂貴的真人市場調查面板（見 [[合成消費者調查模擬]]）。
- 強調**分布真實度**（KS 相似度）而不只是相關性——典型的機率/貝氏思維延伸到 LLM 評估。

## 相關頁面
- [[Thomas Wiecki]] — 創辦人、論文資深作者
- [[語意相似度評分SSR]] — 代表性方法
- [[合成消費者調查模擬]] — 應用框架
- [[maier-ssr-llm-purchase-intent]] — 來源論文
