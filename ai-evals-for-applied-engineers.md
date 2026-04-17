---
title: "AI Evals for Applied Engineers: Testing Production AI Systems"
date: 2026-04-12
updated: 2026-04-14 4:36 PM ET
status: complete
canary: r-7f3a9c12
summary: "AI evals are the systematic testing layer for non-deterministic AI systems. Unlike traditional unit tests, evals handle the reality that the same prompt can produce different outputs every time. This report covers the two leading open-source-friendly tools (promptfoo and Braintrust), practical eval patterns for engineers running LiteLLM and building agents, and how evals fit into an applied AI stack between workflow automation and chat interfaces."
---

**TL;DR:** LLMs are non-deterministic, so traditional tests don't work. Use **promptfoo** (open-source CLI, YAML-driven, self-hostable) to compare models, catch prompt regressions, and red-team your AI. Point it at your LiteLLM proxy and run `promptfoo eval`. Step up to **Braintrust** when you need production tracing and team workflows.

## Executive Summary

AI evals are systematic tests designed for non-deterministic systems. Unlike unit tests that check for exact outputs, evals score LLM responses across dimensions like correctness, relevance, safety, and format compliance.

**Why this matters now:** Prompt changes, model swaps, and RAG pipeline updates can silently degrade quality. Without evals, the only detection method is user complaints. Evals close that gap by catching regressions before they ship.

**The landscape in April 2026:**

- **promptfoo** is the leading open-source eval CLI. Acquired by OpenAI in March 2026, it remains MIT-licensed. It runs locally, supports 60+ LLM providers (including native LiteLLM integration), and handles both quality evals and red-team security scanning. Best fit for self-hosted stacks.
- **Braintrust** is the leading commercial platform. It combines evals with production observability: tracing, scoring, datasets, and alerts. Free tier is generous (1M spans/month). Best fit for teams shipping AI to external users.
- **Langfuse** is the open-source alternative to Braintrust (21K GitHub stars, acquired by ClickHouse). Self-hostable but requires more infrastructure.
- **DeepEval**, **Arize Phoenix**, and **LangSmith** serve narrower niches (pytest-style, notebook-first, and LangChain-coupled, respectively).

**Recommended starting point:** Install promptfoo, point it at your LiteLLM proxy, write 5-10 test cases for a real prompt, and run your first eval. The Getting Started Checklist in Section 7 has a four-week plan.

## Index

1. [What Are Evals and Why They Matter](#1-what-are-evals-and-why-they-matter)
   - The core problem evals solve
   - Three gaps evals bridge (comprehension, specification, generalization)
2. [promptfoo: The Open-Source CLI](#2-promptfoo-the-open-source-cli)
   - How it works (YAML config, matrix testing)
   - Key features table
   - Self-hosting via Docker
   - LiteLLM integration
   - Installation options
3. [Braintrust: The Observability Platform](#3-braintrust-the-observability-platform)
   - How it works (SDK tracing, experiments)
   - Key features table
   - Pricing tiers
   - Self-hosting (enterprise only)
   - promptfoo vs. Braintrust comparison
4. [Other Notable Tools](#4-other-notable-tools)
   - Langfuse (open-source observability)
   - LangSmith (LangChain ecosystem)
   - Arize Phoenix (notebook-first)
   - DeepEval (pytest-style)
   - Quick comparison table
5. [Practical Eval Patterns](#5-practical-eval-patterns)
   - What to test: model comparison, prompt regression, RAG quality, agent behavior, red teaming
   - Component vs. end-to-end evals
   - LLM-as-judge best practices
   - A/B testing prompts
6. [How Evals Connect to the Stack](#6-how-evals-connect-to-the-stack)
   - LiteLLM (model gateway)
   - n8n (automated eval pipelines)
   - Local vs. cloud model comparison
   - Open WebUI (pre-flight checks)
7. [Getting Started Checklist](#7-getting-started-checklist)
   - Week 1: First eval
   - Week 2: Model comparison
   - Week 3: CI/CD integration
   - Week 4: Expand coverage

---

## 1. What Are Evals and Why They Matter

Traditional software testing is deterministic: given input X, you expect output Y. LLMs break this model. The same prompt can produce different answers every run, and "correct" is often subjective. Evals are the testing framework built for this reality.

**The core problem evals solve:** You changed a prompt, swapped a model, or updated your RAG pipeline. Did things get better or worse? Without evals, the answer is "vibes-based testing" where you manually check a few examples and hope for the best. [source: [Pragmatic Engineer - A pragmatic guide to LLM evals for devs](https://newsletter.pragmaticengineer.com/p/evals)]

**Evals are not academic ML benchmarks.** They are practical tests that answer questions like:

- Does my customer support agent actually answer the question asked?
- When I switch from Claude to GPT-5, do my structured outputs still parse correctly?
- Is my RAG pipeline returning relevant context, or hallucinating citations?
- Did my prompt change break edge cases that were working before?

**Three gaps evals bridge** (per the Pragmatic Engineer):

1. **Comprehension gap**: Understanding how your model behaves across real data at scale, not just the five examples you tested by hand.
2. **Specification gap**: Verifying that your prompt actually instructs the LLM to do what you intend.
3. **Generalization gap**: Confirming consistent performance across diverse inputs, not just the happy path.

**Confidence: High.** This framing is consistent across every major source reviewed.

---

## 2. promptfoo: The Open-Source CLI

### What It Is

promptfoo is an open-source (MIT-licensed) CLI and library for evaluating and red-teaming LLM applications. Founded in 2024 by Ian Webster and Michael D'Angelo, it was acquired by OpenAI in March 2026 for its security testing capabilities. OpenAI has committed to keeping it open source. [source: [OpenAI announcement](https://openai.com/index/openai-to-acquire-promptfoo/), [TechCrunch](https://techcrunch.com/2026/03/09/openai-acquires-promptfoo-to-secure-its-ai-agents/)]

Used by over 25% of Fortune 500 companies and trusted by both OpenAI and Anthropic prior to the acquisition. [source: [CNBC](https://www.cnbc.com/2026/03/09/open-ai-cybersecurity-promptfoo-ai-agents.html)]

### How It Works

Everything is driven by a YAML config file (`promptfooconfig.yaml`) that defines three things: prompts, providers (models), and test cases. promptfoo runs every combination and scores the results.

```yaml
# promptfooconfig.yaml
prompts:
  - "Summarize this text in {{language}}: {{input}}"

providers:
  - openai:gpt-4o
  - anthropic:messages:claude-sonnet-4-5

tests:
  - vars:
      language: French
      input: "The quick brown fox jumps over the lazy dog."
    assert:
      - type: contains
        value: "renard"
      - type: llm-rubric
        value: "Output is a grammatically correct French sentence"
```

Run with `promptfoo eval`, view results with `promptfoo view`. The web UI shows side-by-side comparisons across models and prompts. [source: [promptfoo Getting Started](https://www.promptfoo.dev/docs/getting-started/)]

### Key Features

| Feature | Detail |
|---------|--------|
| **Matrix testing** | Every [prompt] x [provider] x [test case] combination, automatically |
| **60+ providers** | OpenAI, Anthropic, Google, Azure, Ollama, LiteLLM, and custom APIs |
| **Assertion types** | `contains`, `similar`, `llm-rubric`, `is-json`, `python`, `javascript`, custom |
| **LLM-as-judge** | Built-in `llm-rubric` assertion type for subjective evaluation |
| **Red teaming** | Scans for 50+ vulnerability types: jailbreaks, injection, RAG poisoning, bias |
| **CI/CD** | GitHub Action (`promptfoo/promptfoo-action`), plus CLI for GitLab/Jenkins |
| **Caching** | Results cached locally, live reload during development |
| **Export** | JSON, YAML, CSV, HTML output formats |

[source: [promptfoo Intro](https://www.promptfoo.dev/docs/intro/), [promptfoo Guides](https://www.promptfoo.dev/docs/guides/)]

### Self-Hosting

promptfoo runs entirely locally by default. The CLI talks directly to LLM APIs from your machine. No data leaves your network unless you choose to share results. [source: [promptfoo Intro](https://www.promptfoo.dev/docs/intro/)]

For a shared web UI, promptfoo offers Docker deployment:

```bash
docker pull ghcr.io/promptfoo/promptfoo:latest

docker run -d \
  --name promptfoo \
  -p 3000:3000 \
  -v /path/to/data:/home/promptfoo/.promptfoo \
  -e OPENAI_API_KEY=sk-xxx \
  ghcr.io/promptfoo/promptfoo:latest
```

Docker Compose and Helm charts are also available. Data persists in a SQLite database at `/home/promptfoo/.promptfoo/promptfoo.db`. [source: [promptfoo Self-Hosting](https://www.promptfoo.dev/docs/usage/self-hosting/)]

**Limitation:** The self-hosted server uses SQLite and an in-memory job queue, so it is limited to a single replica. The docs explicitly state self-hosting "is not recommended for production use cases" that need RBAC, multi-team support, or horizontal scaling. For those, there is an Enterprise tier. [source: [promptfoo Self-Hosting](https://www.promptfoo.dev/docs/usage/self-hosting/)]

**For a homelab, this is fine.** Single-user, single-replica is exactly the homelab use case.

### LiteLLM Integration

promptfoo has a native LiteLLM provider. Point it at your LiteLLM proxy:

```yaml
providers:
  - id: litellm:gpt-4o
    config:
      apiBaseUrl: http://localhost:4000
  - id: litellm:claude-sonnet-4-5
    config:
      apiBaseUrl: http://localhost:4000
```

Or use environment variables: `LITELLM_API_BASE_URL` and `LITELLM_API_KEY`. Since LiteLLM uses the OpenAI format, you can also use the `openai:chat:` provider prefix pointed at your proxy. [source: [promptfoo LiteLLM Provider](https://www.promptfoo.dev/docs/providers/litellm/)]

### Install

```bash
# npm (recommended)
npm install -g promptfoo

# Homebrew
brew install promptfoo

# pip
pip install promptfoo

# No install needed
npx promptfoo@latest init --example getting-started
```

**Confidence: High.** All claims verified against official documentation.

---

## 3. Braintrust: The Observability Platform

### What It Is

Braintrust is a commercial AI evaluation and observability platform. Where promptfoo is a developer CLI, Braintrust is a full platform with tracing, scoring, datasets, prompt management, and production monitoring. Raised $80M Series B in 2025 at an $800M valuation. Used by Vercel, Notion, Coursera, Dropbox, and Replit. [source: [Braintrust home](https://www.braintrust.dev/), [Braintrust pricing](https://www.braintrust.dev/pricing)]

### How It Works

Braintrust provides SDKs (Python, TypeScript, Go, Ruby, C#) that wrap your LLM calls. Every call is traced automatically. You define scoring functions (code-based, LLM-as-judge, or human annotation) and run experiments against versioned datasets.

The key differentiator: Braintrust connects **production observability** to **evaluation**. You can take a production trace that failed, add it to a dataset with one click, and use it as a regression test going forward. [source: [Braintrust home](https://www.braintrust.dev/)]

### Key Features

| Feature | Detail |
|---------|--------|
| **Tracing** | Every LLM call, tool call, and agent step traced with latency/cost/quality |
| **Scoring** | LLM-as-judge ("autoevals"), custom code scorers, human annotation |
| **Datasets** | Versioned test sets, one-click creation from production traces |
| **Experiments** | Side-by-side prompt/model comparisons with statistical analysis |
| **CI/CD** | GitHub Action that posts eval diffs directly on pull requests |
| **Loop** | Built-in AI agent that can run evals and iterate on prompts autonomously |
| **Prompt management** | Version and deploy prompts through the platform |
| **Alerts** | Automated quality alerts on production regressions |

[source: [Braintrust home](https://www.braintrust.dev/), [Braintrust CI/CD article](https://www.braintrust.dev/articles/best-ai-evals-tools-cicd-2025)]

### Pricing

| Tier | Cost | Includes |
|------|------|----------|
| **Free** | $0 | Unlimited users, 1M trace spans/mo, 10K scores/mo, 14-day retention |
| **Pro** | $249/mo | Unlimited spans, 50K scores/mo, 1-month retention |
| **Enterprise** | Custom | Full self-hosted, HIPAA, SSO, RBAC, custom retention |

[source: [Braintrust pricing](https://www.braintrust.dev/pricing)]

### Self-Hosting

Braintrust uses a hybrid architecture separating the control plane (UI, auth) from the data plane (traces, datasets, eval results). Enterprise customers can run the data plane in their own AWS/GCP/Azure VPC via Terraform modules. Full self-hosting requires an enterprise agreement. [source: [Braintrust self-hosted article](https://www.braintrust.dev/articles/best-self-hosted-ai-evals-tools-2026)]

**For a homelab:** The free tier is generous enough for individual use. Self-hosting is enterprise-only, so this is not a "run it on your NAS" tool.

### promptfoo vs. Braintrust

| Dimension | promptfoo | Braintrust |
|-----------|-----------|------------|
| **Model** | Open-source CLI | Commercial platform (free tier) |
| **Strength** | Eval + red team testing | Eval + production observability |
| **Self-hosted** | Yes, Docker/Helm (free) | Enterprise only |
| **Config** | YAML files in your repo | SDK wrapping + web UI |
| **Best for** | Dev/CI testing, security scanning | Production monitoring, team workflows |
| **LiteLLM** | Native provider support | Use OpenAI-compatible SDK config |
| **Learning curve** | Low (YAML + CLI) | Medium (SDK integration) |
| **Collaboration** | Basic sharing | Full team workflows, annotation |

**Confidence: High.** Comparison based on verified feature sets from official sources.

---

## 4. Other Notable Tools

### Langfuse (Open Source)

The most popular open-source LLM observability platform (21K+ GitHub stars). Acquired by ClickHouse in early 2026. Offers tracing, evaluation, prompt management, and datasets. Self-hosts via Docker Compose in about 5 minutes, but production deployments require managing ClickHouse, Redis, PostgreSQL, and S3. MIT licensed. Strong LiteLLM integration. [source: [Langfuse GitHub](https://github.com/langfuse/langfuse), [Langfuse self-hosting](https://langfuse.com/self-hosting)]

**Best for:** Teams wanting open-source observability with self-hosting. Think of it as "open-source Braintrust" with more infrastructure to manage.

### LangSmith (LangChain ecosystem)

The observability and eval platform from the LangChain team. Deep integration with LangChain/LangGraph. Per-trace pricing. Cloud-hosted with playground features. Launched Sandboxes in March 2026 (partnership with NVIDIA). [source: [Braintrust comparison article](https://www.braintrust.dev/articles/langfuse-alternatives-2026)]

**Best for:** Teams already committed to the LangChain framework. If you are not using LangChain, the vendor coupling is hard to justify.

### Arize Phoenix (Open Source)

Notebook-first, open-source evaluation tool from the Arize team. Local-first with Pandas DataFrame integration. Strong for data scientists who want transparency and low abstraction. ELv2 license. [source: [Hamel Husain eval tools review](https://hamel.dev/blog/posts/eval-tools/)]

**Best for:** Data science teams comfortable with Jupyter notebooks who want maximum transparency.

### DeepEval (Open Source)

Python-first eval framework that works like pytest for LLMs. 50+ built-in metrics (G-Eval, hallucination, answer relevancy). Runs evaluator models locally. Native pytest integration for CI/CD. The cloud component (Confident AI) is separate and commercial. [source: [DeepEval GitHub](https://github.com/confident-ai/deepeval), [DeepEval docs](https://deepeval.com/docs/getting-started)]

**Best for:** Python developers who want evals that feel like writing pytest tests.

### Quick Comparison

| Tool | License | Self-Host | Primary Strength | Homelab Fit |
|------|---------|-----------|-----------------|-------------|
| **promptfoo** | MIT | Yes (Docker) | Eval CLI + red teaming | Excellent |
| **Braintrust** | Commercial | Enterprise only | Observability + team evals | Good (free tier) |
| **Langfuse** | MIT | Yes (Docker Compose) | Open-source observability | Good (more infra) |
| **LangSmith** | Commercial | No | LangChain integration | Poor (vendor lock) |
| **Arize Phoenix** | ELv2 | Yes | Notebook-first analysis | Niche |
| **DeepEval** | Apache 2.0 | Yes (CLI) | Pytest-style LLM testing | Good |

**Confidence: High** for promptfoo, Braintrust, Langfuse. **Medium** for LangSmith, Phoenix, DeepEval (less deeply verified).

---

## 5. Practical Eval Patterns

### What to Test

For someone running LLMs via LiteLLM, building agents in Claude Code, and automating with n8n, here are the practical eval categories:

**1. Model comparison evals**
You have LiteLLM routing to multiple models. Which one is actually better for your use case?

```yaml
# Compare models on your actual prompts
providers:
  - id: litellm:claude-sonnet-4-5
    config:
      apiBaseUrl: http://litellm:4000
  - id: litellm:gpt-4o
    config:
      apiBaseUrl: http://litellm:4000
  - id: litellm:llama-3.3-70b
    config:
      apiBaseUrl: http://litellm:4000

tests:
  - vars:
      task: "Extract the invoice number, date, and total from this email..."
    assert:
      - type: is-json
      - type: llm-rubric
        value: "All three fields are correctly extracted"
```

**2. Prompt regression testing**
You changed a system prompt. Did it break anything?

Run the same test suite before and after the change. promptfoo shows a diff view highlighting regressions. Put this in CI so it runs on every commit that touches a prompt file.

**3. RAG quality evals**
Test that your retrieval pipeline returns relevant context and the LLM uses it correctly. Assert that answers cite the right sources and do not hallucinate facts not in the retrieved documents.

**4. Agent behavior evals**
For Claude Code agents or n8n-orchestrated workflows: does the agent choose the right tools? Does it handle edge cases gracefully? Does it know when to ask for clarification vs. proceed?

**5. Security/red team evals**
Does your customer-facing LLM resist jailbreaks? Can users extract system prompts? promptfoo scans for 50+ vulnerability types automatically.

### Component vs. End-to-End Evals

**Component evals** test individual pieces:
- Does the prompt produce well-structured output?
- Does the retriever return relevant documents?
- Does the parser handle edge-case formats?

**End-to-end evals** test the full pipeline:
- Given a user question, does the complete system (retrieval + prompt + model + post-processing) produce a correct, useful answer?

**Start with end-to-end evals.** They catch the most impactful failures. Add component evals when you need to diagnose where things break. [source: [Pragmatic Engineer](https://newsletter.pragmaticengineer.com/p/evals)]

### LLM-as-Judge

When there is no single "right answer" (summarization, conversation quality, tone), use another LLM to evaluate. This is the `llm-rubric` assertion in promptfoo or autoevals in Braintrust.

**Best practices:**
- Use **PASS/FAIL** judgments, not 1-5 scales. Binary forces clarity. [source: [Pragmatic Engineer](https://newsletter.pragmaticengineer.com/p/evals)]
- Write specific rubrics: "The summary mentions the key decision and the deadline" is better than "The summary is good."
- Validate the judge against human labels. If your LLM judge disagrees with domain experts more than 15-20% of the time, the judge needs tuning.
- Use a strong model as the judge (Claude Opus, GPT-4o) even if the system under test uses a cheaper model. [source: [Evidently AI LLM-as-Judge guide](https://www.evidentlyai.com/llm-guide/llm-as-a-judge)]

### A/B Testing Prompts

promptfoo's matrix testing makes this trivial. Define two prompts, same test cases, same models. Compare scores side-by-side. The web UI highlights which prompt won on each test case.

```yaml
prompts:
  - file://prompts/v1-system-prompt.txt
  - file://prompts/v2-system-prompt.txt

providers:
  - litellm:claude-sonnet-4-5

tests:
  - vars: { query: "What's your refund policy?" }
    assert:
      - type: llm-rubric
        value: "Answer is helpful, accurate, and references the actual policy"
```

**Confidence: High.** Patterns drawn from multiple verified sources and official documentation.

---

## 6. How Evals Connect to the Stack

In the applied AI stack, evals sit at **Layer 10.5**: after workflow automation (n8n, Layer 10) and before chat interfaces (Open WebUI, Layer 11). They are the quality gate between "I built something" and "I shipped something."

### LiteLLM (Model Gateway, Layer 6)

LiteLLM gives you a single endpoint for dozens of models. Evals let you **compare those models on your actual workloads** instead of relying on public benchmarks. What matters is not "Claude scores 92% on MMLU" but "Claude extracts invoice fields correctly 97% of the time in my pipeline."

**Pattern:** Use promptfoo's LiteLLM provider to point all evals through your existing proxy. Same API keys, same routing, same rate limits. Your eval environment matches production exactly.

### n8n (Workflow Automation, Layer 10)

n8n orchestrates multi-step AI workflows. Evals verify those workflows produce correct outputs.

**Pattern: Automated eval pipelines.** Build an n8n workflow that:
1. Triggers on a schedule or webhook
2. Pulls test cases from a dataset
3. Runs them through your AI workflow
4. Scores results (via promptfoo CLI or direct LLM-as-judge calls)
5. Posts a summary to Slack/email if quality drops below threshold

This turns evals from a manual developer activity into an automated quality monitoring system.

### Local Models vs. Cloud (Quality Comparison)

Running Ollama locally? Use evals to answer the critical question: "Is the local model good enough for this task, or do I need to pay for the cloud model?"

```yaml
providers:
  - id: litellm:llama-3.3-70b    # Local via Ollama
    config:
      apiBaseUrl: http://litellm:4000
  - id: litellm:claude-sonnet-4-5  # Cloud via Anthropic
    config:
      apiBaseUrl: http://litellm:4000

tests:
  - vars: { task: "Classify this support ticket..." }
    assert:
      - type: llm-rubric
        value: "Classification is correct and confidence is calibrated"
```

If the local model scores within 5% of the cloud model on your specific task, you just saved yourself API costs. If it does not, you have data to justify the cloud spend.

### Open WebUI (Chat Interface, Layer 11)

Open WebUI is where users interact with models. Evals ensure the models behind that interface are performing well before users see them. Think of evals as the pre-flight check before exposing a model through the chat UI.

---

## 7. Getting Started Checklist

For someone with LiteLLM already running and familiarity with n8n:

**Week 1: Install and run your first eval**
- [ ] `npm install -g promptfoo` (or `brew install promptfoo`)
- [ ] `promptfoo init` in a new directory
- [ ] Edit `promptfooconfig.yaml` to point at your LiteLLM proxy
- [ ] Define 5-10 test cases for a real prompt you use
- [ ] Run `promptfoo eval` and `promptfoo view`

**Week 2: Compare models**
- [ ] Add 2-3 providers (local model, cloud model, different cloud model)
- [ ] Add `llm-rubric` assertions for subjective quality
- [ ] Compare results side-by-side
- [ ] Decide which model wins for this specific task

**Week 3: Add to CI/CD or automation**
- [ ] If using GitHub: add `promptfoo/promptfoo-action` to your workflow
- [ ] If using n8n: build an eval pipeline that runs on a schedule
- [ ] Set a quality threshold and alert on regressions

**Week 4: Expand coverage**
- [ ] Add red team testing: `promptfoo redteam init`
- [ ] Build test suites for each major AI workflow
- [ ] Consider Braintrust free tier for production tracing if shipping to users
- [ ] Optionally, self-host the promptfoo web UI via Docker for persistent result storage

---

## Sources

- [promptfoo Documentation](https://www.promptfoo.dev/docs/intro/)
- [promptfoo Getting Started](https://www.promptfoo.dev/docs/getting-started/)
- [promptfoo Self-Hosting](https://www.promptfoo.dev/docs/usage/self-hosting/)
- [promptfoo LiteLLM Provider](https://www.promptfoo.dev/docs/providers/litellm/)
- [promptfoo GitHub](https://github.com/promptfoo/promptfoo)
- [OpenAI Acquires promptfoo](https://openai.com/index/openai-to-acquire-promptfoo/)
- [TechCrunch: OpenAI acquires Promptfoo](https://techcrunch.com/2026/03/09/openai-acquires-promptfoo-to-secure-its-ai-agents/)
- [CNBC: OpenAI to buy Promptfoo](https://www.cnbc.com/2026/03/09/open-ai-cybersecurity-promptfoo-ai-agents.html)
- [Braintrust Home](https://www.braintrust.dev/)
- [Braintrust Pricing](https://www.braintrust.dev/pricing)
- [Braintrust Self-Hosted AI Evals Tools 2026](https://www.braintrust.dev/articles/best-self-hosted-ai-evals-tools-2026)
- [Braintrust CI/CD Evals](https://www.braintrust.dev/articles/best-ai-evals-tools-cicd-2025)
- [Langfuse GitHub](https://github.com/langfuse/langfuse)
- [Langfuse Self-Hosting](https://langfuse.com/self-hosting)
- [DeepEval GitHub](https://github.com/confident-ai/deepeval)
- [DeepEval Docs](https://deepeval.com/docs/getting-started)
- [Hamel Husain: Selecting The Right AI Evals Tool](https://hamel.dev/blog/posts/eval-tools/)
- [Pragmatic Engineer: A pragmatic guide to LLM evals for devs](https://newsletter.pragmaticengineer.com/p/evals)
- [Evidently AI: LLM-as-a-Judge Guide](https://www.evidentlyai.com/llm-guide/llm-as-a-judge)
- [Arize: Comparing LLM Evaluation Platforms](https://arize.com/llm-evaluation-platforms-top-frameworks/)
