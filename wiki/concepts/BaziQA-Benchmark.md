---
title: BaziQA-Benchmark
type: concept
tags: [llm-benchmark, bazi, symbolic-reasoning, temporal-reasoning, evaluation]
created: 2026-06-11
updated: 2026-06-11
sources: [auramate-ai-bazi-reasoning-benchmark, github-chenjiangxi-baziqa]
---

# BaziQA-Benchmark

A multiple-choice benchmark for evaluating **symbolic and temporally compositional reasoning** in LLMs, using Chinese Bazi (八字) fortune-telling as the evaluation domain.

- **Paper**: arXiv:2602.12889 — *BaziQA-Benchmark: Evaluating Symbolic and Temporally Compositional Reasoning in LLMs*
- **Dataset**: github.com/chenjiangxi/baziqa (MIT, ~291 KB)
- **Creator**: [[AuraMate]] / Shanghai Jiao Tong University

## Why Bazi as a Benchmark?

Bazi is not just cultural knowledge recall — it requires:
1. **Calendar arithmetic**: converting Gregorian birth datetime → Heavenly Stems + Earthly Branches (天干地支) across year/month/day/hour pillars
2. **Symbolic rule application**: Five-Element (五行) interactions (生/剋/合/沖), Nayin cycles, 10-year Luck Pillar (大運) progressions
3. **Temporal composition**: reasoning about how elemental forces shift across life stages and annual cycles
4. **Multi-domain inference**: translating abstract elemental patterns into life-domain predictions (career, wealth, health, etc.)

This makes it a rigorous test of **compositional symbolic reasoning** — distinct from factual recall or language understanding benchmarks.

## Dataset Structure

| Component | Subjects | Questions | Notes |
|-----------|---------|-----------|-------|
| Contest8 (2021–2025) | 40/yr × 5 yr | 200 | From Global Diviner Competition; MCQ, verified answers |
| Celebrity50 | 50 | 250 | 5 domains per person; public figures |
| **Total** | **90** | **450** | ~291 KB JSON |

- **Format**: JSON — birth year/month/day/hour, gender, location; 4-option MCQ with correct answer
- **Random baseline**: 25%
- **9 evaluated domains**: career timing, wealth/investment, romance, family/children, health, personality, academics, yearly fortune, holistic assessment

## Leaderboard (Contest8 2021–2025)

| System | Method | Accuracy |
|--------|--------|---------|
| AuraMate 灵伴 (2025) | SRP | **42.0%** |
| DeepSeek-Chat-v3 | Structured | 38.00% |
| GPT-5.1 / Gemini-3-Pro | — | ~36–38.5% |
| DeepSeek-Chat-v3 | Multi-turn | 36.70% |
| DeepSeek-R1 | Structured | 35.00% |
| DeepSeek-R1 | Multi-turn | 34.10% |
| Human 3rd-place (2023) | — | 32.5% |
| Human champion range | — | 37.5–50% |
| Random baseline | — | 25% |

**Notable**: DeepSeek-R1 (reasoning model) underperforms DeepSeek-Chat-v3 — extended chain-of-thought may be counterproductive for this symbolic rule-lookup task.

## AuraMate Structured Reasoning Protocol (SRP)

Three sequential steps proprietary to [[AuraMate]]:
1. **Comprehensive pattern assessment** — identify all active elemental configurations in the chart
2. **Force prioritization within temporal context** — rank dominant elements given current Luck Pillar + annual pillar
3. **Event inference** — map force dynamics to life-domain outcomes

Gains over vanilla LLM prompting: **+3 to +30 percentage points** (variance by domain).

## Limitations & Tensions
- MCQ format avoids subjectivity but underrepresents real consultation complexity
- Data contamination risk: competition questions may appear in LLM training data
- Human champion ceiling (~50%) still well above best AI (42%), so task is unsolved
- Celebrity50 ground truth methodology unclear (historical records vs. expert consensus)

## Related
- [[AI Agent評測基準]] — broader LLM benchmark landscape including SWE-bench, GAIA, etc.
- [[AuraMate]] — benchmark creator
- [[八字推命]] — the Bazi domain being evaluated
