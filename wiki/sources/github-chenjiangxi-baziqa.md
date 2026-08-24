---
title: "BaziQA Dataset — GitHub: chenjiangxi/baziqa"
type: source-summary
tags: [bazi, llm-benchmark, dataset, open-source]
created: 2026-06-11
updated: 2026-06-11
sources: [github-chenjiangxi-baziqa]
---

# BaziQA Dataset (GitHub)

## Origin
- **Repository**: https://github.com/chenjiangxi/baziqa
- **License**: MIT
- **Paper**: arXiv:2602.12889
- **Related project**: [[AuraMate]] 灵伴

## Key Takeaways
- **Scale**: 90 subjects, 450 questions, ~291 KB; two components:
  - *Contest8 (2021–2025)*: 5 yearly competitions × 8 subjects × 5 MCQs = 200 contest questions
  - *Celebrity50*: 50 public figures × 5 life domains = 250 detailed fortune profiles
- **Data format**: JSON per subject — birth year/month/day/hour, gender, location; four-option MCQ with verified answers.
- **5 life domains in Celebrity50**: Romance (感情), Wealth (财富), Family Relations (六亲), Career (事业), Health (健康).
- **Evaluation results on Contest8 (2021–2025)**:

| Model | Method | Avg Accuracy |
|-------|--------|-------------|
| DeepSeek-Chat-v3 | Structured | 38.00% |
| DeepSeek-Chat-v3 | Multi-turn | 36.70% |
| DeepSeek-R1 | Structured | 35.00% |
| DeepSeek-R1 | Multi-turn | 34.10% |

- **Method comparison**: Structured prompting consistently outperforms multi-turn dialogue (+1–2 pp), confirming the value of [[BaziQA-Benchmark#Structured Reasoning Protocol|SRP-style]] structured input.
- **Random baseline**: 25% (4-option MCQ).

## Entities Mentioned
- [[AuraMate]] — related commercial application
- DeepSeek-Chat-v3, DeepSeek-R1 — evaluated models

## Concepts Mentioned
- [[BaziQA-Benchmark]] — the benchmark this dataset implements
- [[八字推命]] — the Bazi domain

## Questions Raised
- Celebrity50 profiles — are answers ground-truthed against historical records, or expert consensus?
- How does DeepSeek-R1's lower score (despite strong reasoning) compare to its performance on other symbolic reasoning tasks — is CoT harmful here?
