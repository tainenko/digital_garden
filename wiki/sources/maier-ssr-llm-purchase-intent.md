---
title: "LLMs Reproduce Human Purchase Intent via Semantic Similarity Elicitation of Likert Ratings"
type: source-summary
tags: [llm, synthetic-consumers, market-research, semantic-similarity, embeddings, likert, pymc-labs, colgate]
created: 2026-06-16
updated: 2026-06-16
sources: [maier-ssr-llm-purchase-intent]
---

# LLMs Reproduce Human Purchase Intent via Semantic Similarity Elicitation of Likert Ratings

## Origin
- **Title**: LLMs Reproduce Human Purchase Intent via Semantic Similarity Elicitation of Likert Ratings
- **Authors**: Benjamin F. Maier, Ulf Aslak, Luca Fiaschi, Nina Rismal, Kemble Fletcher, Christian C. Luhmann, Robbie Dow, Kli Pappas, Thomas V. Wiecki
- **Affiliations**: [[PyMC Labs]] × Colgate-Palmolive
- **Date**: 2025-10-09 (v1); revised 2025-10-27
- **arXiv**: 2510.08338 — https://arxiv.org/abs/2510.08338
- **Code**: https://github.com/pymc-labs/semantic-similarity-rating (official SSR implementation)
- **Discovery link**: BAAI Hub https://hub.baai.ac.cn/paper/d3108d63-8261-4aaa-bd34-ee0a9b888346

## Key Takeaways
- **Problem**: Consumer research costs businesses billions/year and suffers from sample-panel bias + scalability limits. Asking an LLM directly for a 1–5 Likert score (Direct Likert Rating, DLR) produces **unrealistic distributions** — models collapse toward the neutral midpoint, so the synthetic distribution looks nothing like the human one (KS similarity ≈ 0.26–0.39).
- **Core method — Semantic Similarity Rating (SSR)**: Don't ask the LLM for a number. Ask it for a **free-text expression of purchase intent**, embed that text, and map it to a Likert *probability distribution* by comparing (cosine similarity) against pre-written **anchor/reference statements** — one per scale point. See [[語意相似度評分SSR]].
- **Headline result**: SSR achieves **~90% of human test–retest reliability** (correlation attainment ρ ≈ 90% GPT-4o / 92% Gemini-2.0-flash) while preserving realistic distributions (**KS similarity > 0.85**, up to 0.88).
- **Validation dataset**: 57 real Colgate-Palmolive consumer surveys, **9,300 unique U.S. participants**, evaluating personal-care product concepts (presented as concept images with text + optional artwork).
- **Bonus**: Synthetic respondents emit a **qualitative free-text rationale** alongside the structured rating — interpretability that pure numeric scoring can't give.
- **Models**: Response generation with **GPT-4o** and **Gemini-2.0-flash**; embeddings via **OpenAI text-embedding-3-small** (large gave near-identical results — method is robust to embedding choice).
- **Method ladder (worst→best)**: DLR (direct integer) ρ≈80% → FLR (free-text then a 2nd "Likert expert" LLM maps to integer) ρ≈85–90% → **SSR** ρ≈90–92%. The jump from FLR to SSR comes from mapping to a *distribution* via embeddings rather than a single integer.

## How SSR works (condensed)
1. Persona prompt: LLM impersonates a consumer with full demographics (age, gender, income, location, ethnicity); n=2 samples per prompt for stability.
2. Elicit free-text purchase-intent response to the concept.
3. Embed response; cosine-similarity vs. 5 anchor statements (one per Likert point), across 6 anchor sets.
4. Build a PMF by subtracting the min similarity across the reference set, normalize to sum 1 (no softmax; T=1, ε=0 in experiments).
→ Full mechanics on [[語意相似度評分SSR]].

## Entities Mentioned
- [[PyMC Labs]] — Bayesian/probabilistic ML consultancy; co-developer of SSR
- [[Thomas Wiecki]] — PyMC Labs founder, senior author; PyMC core creator
- Colgate-Palmolive — supplied the 57-survey / 9,300-respondent ground-truth dataset

## Concepts Mentioned
- [[語意相似度評分SSR]] — the core SSR method (anchors + embeddings → Likert PMF)
- [[合成消費者調查模擬]] — broader framing: LLMs as synthetic survey respondents
- [[RAG檢索增強生成實戰]] — shared embedding/cosine-similarity machinery
- [[AI Agent評測基準]] / [[BaziQA-Benchmark]] — sibling "LLM-vs-human" evaluation framing
- [[LLM限制與解決方案]] — distribution-distortion is another LLM failure mode + mitigation

## Contradictions / Tensions
- **vs. naive "just ask the LLM"**: Direct numeric Likert (DLR) is the obvious approach and it *fails on distribution* (KS≈0.26) even when its rank correlation looks okay (ρ≈80%). Good ranking ≠ realistic distribution — a caution for any LLM-as-survey-panel effort.
- **vs. supervised baseline**: A LightGBM classifier on demographics + product features reaches only ρ≈65%, below all LLM prompting methods — the LLM's world knowledge matters.
- **Demographic fidelity gap**: Not all demographic patterns replicate; gender and region effects are poorly captured, so SSR is better at aggregate intent than at sub-population slicing.

## Questions Raised
- Reference statements are **hand-designed** and performance varies by anchor set — can they be optimized automatically/iteratively?
- Does SSR generalize beyond purchase intent to other Likert constructs (satisfaction, trust, relevance)?
- How well does it hold outside personal care / outside the LLM's training-data domain (hallucination risk on unfamiliar products)?
- It cannot model real-world contingencies (budget constraints, actual marketing exposure) — how far can synthetic panels substitute for real ones before decisions degrade?
