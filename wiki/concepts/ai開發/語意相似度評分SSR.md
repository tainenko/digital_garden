---
title: 語意相似度評分 (Semantic Similarity Rating, SSR)
type: concept
tags: [llm, semantic-similarity, embeddings, likert, synthetic-consumers, market-research]
created: 2026-06-16
updated: 2026-06-16
sources: [maier-ssr-llm-purchase-intent]
---

# 語意相似度評分 (Semantic Similarity Rating, SSR)

一種讓 LLM 模擬人類問卷評分（Likert 量表）的方法。**不直接問 LLM 要分數**，而是請它寫自由文字回應，再用 embedding 相似度把文字映射成 Likert 機率分布。出自 [[PyMC Labs]] × Colgate-Palmolive 論文（[[maier-ssr-llm-purchase-intent]]）。

---

## 為什麼需要 SSR

直接叫 LLM 打 1–5 分（Direct Likert Rating, DLR）會產生**不真實的分布**：模型傾向集中在中間值（neutral），合成分布與人類分布差距極大（KS 相似度僅 ≈0.26–0.39）。
→ 排序相關性（ρ）可能還行（≈80%），但**分布形狀完全失真**。「排得對」不等於「分布像」。

這是 [[LLM限制與解決方案]] 中數值輸出失真的一個具體案例。

---

## 方法三要素

1. **自由文字引出 (Textual elicitation)**：請 LLM 以特定人口屬性 persona（年齡/性別/收入/地點/族裔）對產品概念寫出購買意願的自由文字。每個 prompt 抽 n=2 樣本以求穩定。
2. **參考錨句 (Reference / anchor statements)**：為 Likert 每一個刻度（1–5）各寫一句短、通用、跨領域的錨句，表達由「不太可能買」到「很可能買」的意願強度。論文用 **6 組**錨句集，每組 5 句。
3. **Embedding 映射**：將回應與每個錨句都轉成向量，用 cosine 相似度比對，映射為 Likert 上的機率分布（PMF）。

---

## 映射數學

對回應 σ 與第 i 組第 r 個錨句的 embedding 取 cosine 相似度：

```
γ(σ, t) = (v_σ · v_t) / (|v_σ| |v_t|)
```

PMF（式 8）：對每個刻度的相似度**減去該參考集內的最小相似度**（修正低變異），再正規化使總和為 1：

```
p(r) ∝ γ(σ_r, t) − min_ℓ γ(σ_ℓ, t) + ε·δ
Σ_r p(r) = 1
```

- 有一個 temperature 類參數控制 PMF 的離散程度；實驗用 **T=1, ε=0**。
- **不用 softmax**——正規化發生在「減最小值」之後。

---

## 模型與設定

| 元件 | 選擇 | 備註 |
|------|------|------|
| 生成模型 | GPT-4o、Gemini-2.0-flash | 早期試過 gemini-1.5/2.5、o3 |
| Embedding | OpenAI text-embedding-3-small | 換 large 結果幾乎相同（方法對 embedding 穩健）|
| 取樣 | T_LLM ∈ {0.5, 1.5}, top_p=0.9, n=2 | persona 提供完整人口屬性效果最佳 |

---

## 方法階梯（由差到好）

| 方法 | 作法 | 相關達成率 ρ | KS 相似度 |
|------|------|-------------|-----------|
| **DLR** Direct Likert | LLM 直接輸出整數 1–5 | ≈80% | 0.26–0.39 ❌ 分布失真 |
| **FLR** Follow-up Likert | 先寫自由文字 → 第二個「Likert 專家」LLM 轉成整數 | 85–90% | — |
| **SSR** | 自由文字 → embedding → **機率分布** | **90–92%** | **0.80–0.88** ✅ |
| LightGBM（監督式基準）| 人口+產品特徵分類器 | 65% | — |

關鍵躍升來自 FLR→SSR：映射到**分布**（embedding）而非單一整數。

---

## 限制

- 錨句**人工設計**，效果隨錨句集而變動。
- 並非所有人口模式都能複製——**性別、地區效應較差**，適合總體意願、不適合細分群。
- 受訓練資料領域限制，陌生產品可能幻覺。
- 無法模擬真實情境約束（預算、實際行銷曝光）。
- Embedding 模型與相似度度量的選擇會影響表現。

## 未來方向

動態/迭代優化錨句、推廣到其他 Likert 構念（滿意度/信任/相關性）、自動調參、多階段批判校準 pipeline、SSR + 輕量微調混合法。

---

## 相關頁面
- [[合成消費者調查模擬]] — SSR 所服務的大框架
- [[RAG檢索增強生成實戰]] — 共用 embedding + cosine 相似度機制
- [[Prompt Engineering進階]] — persona 提示與結構化輸出
- [[AI Agent評測基準]]、[[BaziQA-Benchmark]] — 同屬「LLM 對齊人類」評測
- [[LLM限制與解決方案]] — 數值分布失真作為一種 LLM 失效模式
