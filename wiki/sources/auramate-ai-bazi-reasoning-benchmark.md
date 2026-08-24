---
title: "AI 算八字到底准不准？我们让 AI 和人类顶级命理师正面对决"
type: source-summary
tags: [bazi, llm-benchmark, chinese-metaphysics, reasoning, auramate]
created: 2026-06-11
updated: 2026-06-11
sources: [auramate-ai-bazi-reasoning-benchmark]
---

# AI 算八字到底准不准？

## Origin
- **Title**: AI 算八字到底准不准？我们让 AI 和人类顶级命理师正面对决
- **Publisher**: AuraMate Research
- **Date**: 2026-02-01
- **URL**: https://auramate.net/article/ai-bazi-reasoning-benchmark
- **Related paper**: BaziQA-Benchmark — Evaluating Symbolic and Temporally Compositional Reasoning in LLMs (arXiv:2602.12889)

## Key Takeaways
- **Benchmark source**: 200 MCQs drawn from five consecutive Global Diviner Competitions (2021–2025), covering 9 life domains; 25% random baseline.
- **9 domains tested**: career timing, wealth/investment, romance, family/children, health, personality, academics, yearly fortune, holistic assessment.
- **AI parity with mid-tier humans**: Top general LLMs (DeepSeek-V3, Gemini-3-Pro, GPT-5.1) score 36–38.5%; human champions range 37.5–50%. AI already surpasses lower-ranked human competitors.
- **Historical milestone**: In 2023, the strongest general AI reached 36.0%, surpassing the third-place human finalist at 32.5%.
- **AuraMate SRP**: Proprietary Structured Reasoning Protocol — three sequential steps: (1) comprehensive pattern assessment, (2) force prioritization within temporal context, (3) event inference — yields +3 to +30 pp gains across domains.
- **AuraMate 灵伴 scores**: 42.0% in 2025; five-year average 37.5%.
- **Open benchmark**: Full dataset and evaluation code publicly released; "only AI that passes objective verification deserves user trust."
- **Framing**: The paper frames Bazi as a *symbolic + temporally compositional reasoning* task — calendar cycle arithmetic, Heavenly Stems/Earthly Branches interactions, Five-Element force dynamics — not merely cultural knowledge recall.

## Entities Mentioned
- [[AuraMate]] — publisher and developer of 灵伴 app + SRP methodology
- [[Shanghai Jiao Tong University]] — affiliated research institution
- DeepSeek-V3, DeepSeek-R1, Gemini-3-Pro, GPT-5.1 — evaluated models

## Concepts Mentioned
- [[BaziQA-Benchmark]] — the MCQ evaluation dataset and methodology
- [[八字推命]] — the Bazi fortune-telling domain being evaluated
- [[AI Agent評測基準]] — broader context of LLM evaluation benchmarks

## Contradictions / Tensions
- Human champion ceiling (50%) is still meaningfully above best AI (42%), suggesting the task is not solved — but the gap is closing fast.
- MCQ format eliminates subjectivity but may not capture the full complexity of real-world Bazi consultation (open-ended interpretation, client interaction).

## Questions Raised
- Does SRP generalize to other symbolic/calendrical reasoning domains (e.g., astrology, I Ching, Vedic jyotish)?
- What explains the +3 to +30 pp variance across domains — which domains benefit most from structured reasoning?
- How does the benchmark hold up against data contamination risk (competition questions may appear in training data)?
