---
title: "AI Hallucination Metrics in 2026: Still a Thing, Getting Weirder"
date: 2026-04-01
updated: 2026-04-01
status: complete
canary: r-8b2d4e91
summary: "Hallucination rates on simple summarization tasks have dropped 96% since 2021 (from 21.8% to 0.7% for the best models). But the picture is more complicated than the headline: reasoning models actually hallucinate more than their non-reasoning counterparts, domain-specific hallucination rates remain dangerously high (18.7% legal, 15.6% medical), and AI-generated hallucinations are now contaminating peer-reviewed science. It is still very much a thing."
---

**TL;DR:** The best models now hallucinate under 1% on simple summarization benchmarks, down from 21.8% in 2021. But "reasoning" models (o3, GPT-5, Claude Sonnet 4.5) hallucinate at 10-50% on harder tasks because thinking harder means more invented inferences. Domain-specific rates remain high: 18.7% on legal questions, 15.6% on medical. In the real world, 47% of enterprise AI users report making a major business decision based on hallucinated content. GPTZero found 50+ hallucinated papers slipped past peer review at ICLR 2026. Hallucination is not solved. It has shifted from "obvious nonsense" to "confidently wrong in ways that fool experts."

## Table of Contents

1. [The Numbers: Model-by-Model](#1-the-numbers-model-by-model)
2. [The Reasoning Model Paradox](#2-the-reasoning-model-paradox)
3. [Domain-Specific Rates](#3-domain-specific-rates)
4. [Key Benchmarks](#4-key-benchmarks)
5. [Real-World Incidents](#5-real-world-incidents)
6. [Enterprise Impact](#6-enterprise-impact)
7. [Bottom Line](#7-bottom-line)

## 1. The Numbers: Model-by-Model

**Vectara HHEM Leaderboard** (summarization task, lower is better):

| Model | Hallucination Rate | Notes |
|-------|-------------------|-------|
| Gemini 2.0 Flash | ~0.7% | Best on record |
| GPT-4o family | 0.8-2.0% | Consistently strong |
| Claude 4.6 Sonnet | ~3% | Constitutional AI helps |
| Claude Opus | ~10.1% | Trades accuracy for depth |
| GPT-5 | >10% | Reasoning model penalty |
| Claude Sonnet 4.5 | >10% | Reasoning model penalty |
| Grok-4 | >10% | Reasoning model penalty |
| Gemini 3 Pro | >10% | Reasoning model penalty |
| OpenAI o3 | 33-51% | PersonQA/SimpleQA benchmarks |

The best-performing models improved from 21.8% hallucination in 2021 to 0.7% in 2025, a 96% reduction over four years. [source: [Suprmind Research Report](https://suprmind.ai/hub/insights/ai-hallucination-statistics-research-report-2026/)]

**Confidence: High.** Multiple independent sources (Vectara, Artificial Analysis, AllAboutAI) converge on these ranges.

## 2. The Reasoning Model Paradox

This is the most counterintuitive finding of 2026. Models marketed as strong "reasoners" hallucinate significantly more on grounded tasks.

Every reasoning model tested on Vectara's refreshed 7,700-article dataset exceeded 10% hallucination. The mechanism: reasoning models invest computational effort into "thinking through" answers, which during summarization leads them to add inferences, draw connections, and generate insights that go beyond what is in the source document. They are not failing to understand the text. They are succeeding at generating plausible-sounding extensions of it. [source: [Vectara Blog](https://www.vectara.com/blog/introducing-the-next-generation-of-vectaras-hallucination-leaderboard)]

Most dramatically, OpenAI's o3 series hit 33-51% hallucination on PersonQA and SimpleQA, more than double the earlier o1 models (~16%). [source: [ModelsLab](https://modelslab.com/blog/llm/llm-hallucination-rates-2026)]

**Confidence: High.** Vectara's methodology is well-documented and the reasoning model finding has been independently confirmed.

## 3. Domain-Specific Rates

General-purpose hallucination rates mask severe domain-specific problems:

| Domain | Hallucination Rate |
|--------|-------------------|
| Legal questions | 18.7% |
| Medical queries | 15.6% |
| Legal citations | 6.4% |
| Programming content | 5.2% |
| Citation accuracy (Perplexity, best) | 37% |
| Citation accuracy (Grok-3, worst) | 94% |

The citation accuracy numbers are particularly striking. In a Columbia Journalism Review study, even the best-performing model (Perplexity) hallucinated 37% of citations, while Grok-3 hallucinated 94%. [source: [AllAboutAI Hallucination Report](https://www.allaboutai.com/resources/ai-statistics/ai-hallucinations/)]

**Confidence: High** for the domain rates. **Medium** for the citation accuracy study, as the methodology may differ from Vectara's.

## 4. Key Benchmarks

The hallucination measurement ecosystem has matured significantly:

- **Vectara HHEM Leaderboard**: The longest-running benchmark. Uses HHEM-2.3 model on a refreshed 7,700-article dataset. Measures hallucination in summarization tasks. [source: [GitHub](https://github.com/vectara/hallucination-leaderboard)]
- **AA-Omniscience** (Artificial Analysis, Nov 2025): 6,000 questions across 42 topics in 6 domains. Unique in that it penalizes wrong answers but does not penalize refusal, rewarding models that know their limits. [source: [Artificial Analysis](https://artificialanalysis.ai/evaluations/omniscience)]
- **OpenAI SimpleQA / PersonQA**: OpenAI's own benchmarks for factual accuracy on simple questions and person-related facts.
- **Google DeepMind FACTS Benchmark**: Focuses on factual accuracy in generation.
- **HalluHard** (Swiss-German consortium): Stress-tests models on adversarial hallucination scenarios.

**Confidence: High.** These are all publicly documented benchmarks with known methodologies.

## 5. Real-World Incidents

### ICLR 2026 Hallucination Contamination

GPTZero scanned 300 of 20,000 papers submitted to ICLR 2026 and found 50+ contained at least one obvious AI hallucination (fabricated citations, invented references). Each paper had been reviewed by 3-5 peer experts, most of whom missed the fake citations. Some citations pointed to `example.com`. GPTZero is now collaborating with ICLR program chairs to scan all 20,000 submissions. [source: [GPTZero](https://gptzero.me/news/iclr-2026/), [BetaKit](https://betakit.com/start-up-investigation-reveals-50-peer-reviewed-papers-contained-hallucinated-citations/)]

This followed a similar finding at NeurIPS 2025, where GPTZero found 100+ hallucinated citations in accepted papers. [source: [Fortune](https://fortune.com/2026/01/21/neurips-ai-conferences-research-papers-hallucinations/)]

### Confidence Language Inversion

Research has found that AI models are 34% more likely to use confident language ("definitely," "certainly," "without doubt") when generating incorrect information than when providing factual information. This makes hallucinations harder to detect by humans. [source: [Suprmind](https://suprmind.ai/hub/insights/ai-hallucination-statistics-research-report-2026/)]

**Confidence: High** for the ICLR finding (primary source). **Medium** for the confidence language inversion (single source).

## 6. Enterprise Impact

- 47% of enterprise AI users admitted to making at least one major business decision based on hallucinated content. [source: [Suprmind](https://suprmind.ai/hub/insights/ai-hallucination-statistics-research-report-2026/)]
- Enterprise risk framing is shifting: the concern is no longer just "will it hallucinate?" but "will it hallucinate confidently while operating autonomously at scale?" [source: [VKTR](https://www.vktr.com/ai-technology/wiring-the-intelligent-enterprise-for-the-year-ahead-2026-will-be-a-year-of-reckoning-for-ai/)]
- RAG (retrieval-augmented generation) remains the primary enterprise mitigation strategy, but it reduces rather than eliminates hallucination. [source: [Contextual AI](https://contextual.ai/blog/why-does-enterprise-ai-hallucinate)]
- Duke University Libraries published a piece in January 2026 titled "It's 2026. Why Are LLMs Still Hallucinating?" arguing the problem is architectural, not fixable by scaling alone. [source: [Duke Libraries](https://blogs.library.duke.edu/blog/2026/01/05/its-2026-why-are-llms-still-hallucinating/)]

**Confidence: Medium** on the 47% stat (single source, survey methodology unknown). **High** on the architectural argument (widely held view).

## 7. Bottom Line

Hallucination is not solved. It has improved dramatically on simple tasks (0.7% vs 21.8% four years ago), but the improvement is uneven and sometimes inverted:

1. **Simple summarization**: Largely solved for top models (<1-3%).
2. **Reasoning tasks**: Getting worse as models get "smarter." The thinking-harder penalty is real.
3. **Domain-specific accuracy**: Still dangerously high in law, medicine, and citations.
4. **Real-world contamination**: Hallucinated content is now polluting peer-reviewed science.
5. **Enterprise risk**: Nearly half of enterprise users have acted on hallucinated content.
6. **Detection difficulty**: Models are more confident when wrong, making human detection harder.

The trajectory is not a simple downward line. It is more like: models are getting better at being right on easy things and worse at being wrong on hard things, with increasing confidence either way.

---

*Research compiled 2026-04-01 13:40 ET. All claims sourced from web search results dated 2025-2026.*
