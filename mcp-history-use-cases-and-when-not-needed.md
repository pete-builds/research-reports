---
title: "MCP: History, Use Cases, and When It's Not Needed"
date: 2026-06-14
updated: 2026-06-14 8:32 AM ET
summary: "A grounded, citeable reference on the Model Context Protocol: its November 2024 origin at Anthropic, the M×N integration problem it solves, where it fits, and the concrete cases where a plain API call or CLI is the better choice."
---

## TL;DR

The Model Context Protocol (MCP) is an open standard Anthropic released on November 25, 2024 to give AI applications a uniform way to connect to external data and tools, replacing one-off per-integration glue with a single protocol ([source](https://www.anthropic.com/news/model-context-protocol)). By mid-2025 it had been adopted by OpenAI, Google, and Microsoft, turning a vendor spec into de facto industry infrastructure, and in December 2025 Anthropic donated it to a Linux Foundation fund ([source](https://en.wikipedia.org/wiki/Model_Context_Protocol)). It is genuinely useful when one integration must be reused across many AI clients or when tools need dynamic discovery, but it is overkill (added latency, token-bloat, security surface) when you have a fixed, known set of tools that a direct API call or CLI can hit more cheaply ([source](https://www.improving.com/thoughts/when-mcp-is-not-the-right-choice/)).

## Current Status

- MCP is the dominant cross-vendor standard for connecting LLM applications to external tools and data, supported by Anthropic's Claude, OpenAI's ChatGPT, Google's Gemini, and Microsoft's Windows/Copilot/Azure stack ([source](https://modelcontextprotocol.io/introduction)) ([source](https://en.wikipedia.org/wiki/Model_Context_Protocol)).
- The current spec revision is `2025-11-25` (released on MCP's one-year anniversary), following `2025-06-18` and `2025-03-26`; the original was `2024-11-05` ([source](https://blog.modelcontextprotocol.io/posts/2025-11-25-first-mcp-anniversary/)) ([source](https://modelcontextprotocol.io/specification/2025-06-18)).
- Transport has consolidated on **stdio** (local) and **Streamable HTTP** (remote); the old HTTP+SSE transport was deprecated in the `2025-03-26` revision and kept only for backwards compatibility ([source](https://modelcontextprotocol.io/docs/learn/architecture)).
- Governance moved out of Anthropic: in December 2025 MCP was donated to the Agentic AI Foundation (AAIF), a directed fund under the Linux Foundation co-founded by Anthropic, Block, and OpenAI ([source](https://en.wikipedia.org/wiki/Model_Context_Protocol)).
- An official MCP Registry launched in preview on September 8, 2025 and grew to nearly 2,000 server entries within months ([source](https://blog.modelcontextprotocol.io/posts/2025-09-08-mcp-registry-preview/)) ([source](https://blog.modelcontextprotocol.io/posts/2025-11-25-first-mcp-anniversary/)).
- The biggest unresolved debates are security (prompt injection via tool output, tool poisoning, supply-chain "rug pulls") and right-sizing (when MCP is the wrong tool versus a direct API) ([source](https://simonwillison.net/2025/Apr/9/mcp-prompt-injection/)) ([source](https://www.improving.com/thoughts/when-mcp-is-not-the-right-choice/)).

## Table of Contents

- [TL;DR](#tldr)
- [Current Status](#current-status)
- [Findings](#findings)
  - [1. What MCP Is](#1-what-mcp-is)
  - [2. History & Timeline](#2-history--timeline)
  - [3. Why It's Needed (the problem it solves)](#3-why-its-needed-the-problem-it-solves)
  - [4. Use Cases (where it fits well)](#4-use-cases-where-it-fits-well)
  - [5. When It's NOT Needed (criticisms, alternatives, anti-patterns)](#5-when-its-not-needed-criticisms-alternatives-anti-patterns)
  - [6. Ecosystem & Existing Tooling (SDKs, registry, adoption)](#6-ecosystem--existing-tooling-sdks-registry-adoption)
  - [7. Security Considerations](#7-security-considerations)
- [Confidence Assessment](#confidence-assessment)
  - [High Confidence](#high-confidence)
  - [Medium Confidence](#medium-confidence)
- [Open Questions](#open-questions)
- [Sources](#sources)
- [Update History](#update-history)
- [How This Report Was Generated](#how-this-report-was-generated)

## Findings

### 1. What MCP Is

MCP is an open-source standard for connecting AI applications to external systems: data sources (files, databases), tools (search, calculators, APIs), and workflows (specialized prompts) ([source](https://modelcontextprotocol.io/introduction)). The official analogy is "a USB-C port for AI applications": just as USB-C standardizes how devices connect, MCP standardizes how AI applications connect to external systems ([source](https://modelcontextprotocol.io/introduction)).

**Architecture.** MCP follows a client-server model with three participants ([source](https://modelcontextprotocol.io/docs/learn/architecture)):
- **Host**: the AI application (e.g., Claude Desktop, Claude Code, VS Code) that coordinates one or more clients.
- **Client**: a connector inside the host that maintains a dedicated 1:1 connection to one server.
- **Server**: a program that exposes context and capabilities. Servers can run locally or remotely.

It is built on **JSON-RPC 2.0** and is a **stateful** protocol with capability negotiation at connection time ([source](https://modelcontextprotocol.io/specification/2025-06-18)). The spec explicitly takes inspiration from the **Language Server Protocol (LSP)**: just as LSP standardized language support across editors, MCP standardizes context/tool integration across AI apps ([source](https://modelcontextprotocol.io/specification/2025-06-18)).

**Two layers** ([source](https://modelcontextprotocol.io/docs/learn/architecture)):
- **Data layer**: the JSON-RPC protocol defining lifecycle management, primitives, and notifications.
- **Transport layer**: the communication channel and authorization (stdio or Streamable HTTP).

**Primitives the server exposes** ([source](https://modelcontextprotocol.io/docs/learn/architecture)):
- **Tools**: executable functions the model can invoke (file ops, API calls, DB queries).
- **Resources**: data sources providing context (file contents, DB records, API responses).
- **Prompts**: reusable interaction templates (system prompts, few-shot examples).

**Primitives the client exposes back to servers** ([source](https://modelcontextprotocol.io/docs/learn/architecture)):
- **Sampling**: a server can request an LLM completion from the host, so the server stays model-independent (no bundled LLM SDK).
- **Elicitation**: a server can ask the user for more input or confirmation.
- **Roots / Logging**: filesystem/uri boundary inquiries and debug logging.

Each primitive has `*/list` (discovery), `*/get` (retrieval), and where relevant `tools/call` (execution). Discovery is dynamic: a server can send a `notifications/tools/list_changed` message and the client re-fetches, so available tools can change at runtime ([source](https://modelcontextprotocol.io/docs/learn/architecture)).

### 2. History & Timeline

Reverse chronological (newest first):

| Date (ET where applicable) | Milestone | Source |
|---|---|---|
| April 2026 | AAIF held the MCP Dev Summit North America in NYC, ~1,200 attendees | ([source](https://en.wikipedia.org/wiki/Model_Context_Protocol)) |
| Dec 2025 | Anthropic donated MCP to the Agentic AI Foundation (AAIF), a Linux Foundation directed fund co-founded by Anthropic, Block, and OpenAI | ([source](https://en.wikipedia.org/wiki/Model_Context_Protocol)) |
| Nov 25, 2025 | Spec revision `2025-11-25` (one-year anniversary): tasks (SEP-1686), simplified OAuth authorization (SEP-991), sampling-with-tools (SEP-1577), URL-mode elicitation; backwards compatible | ([source](https://blog.modelcontextprotocol.io/posts/2025-11-25-first-mcp-anniversary/)) |
| Sep 8, 2025 | Official MCP Registry launched in preview | ([source](https://blog.modelcontextprotocol.io/posts/2025-09-08-mcp-registry-preview/)) |
| Jun 18, 2025 | Spec revision `2025-06-18` (current stable family prior to Nov) | ([source](https://modelcontextprotocol.io/specification/2025-06-18)) |
| May 19, 2025 | Microsoft Build 2025: MCP made a first-class standard across Windows 11, GitHub, Copilot Studio, Dynamics 365, and Azure AI Foundry | ([source](https://blogs.windows.com/windowsdeveloper/2025/05/19/advancing-windows-for-ai-development-new-platform-capabilities-and-tools-introduced-at-build-2025/)) ([source](https://devblogs.microsoft.com/foundry/announcing-model-context-protocol-support-preview-in-azure-ai-foundry-agent-service/)) |
| Apr 9, 2025 | Google DeepMind (Demis Hassabis) committed to MCP support in Gemini models and SDK; later shipped native SDK support, surfaced at Google I/O 2025 | ([source](https://techcrunch.com/2025/04/09/google-says-itll-embrace-anthropics-standard-for-connecting-ai-models-to-data/)) ([source](https://en.wikipedia.org/wiki/Model_Context_Protocol)) |
| Apr 9, 2025 | Simon Willison publishes prominent prompt-injection / tool-poisoning critique | ([source](https://simonwillison.net/2025/Apr/9/mcp-prompt-injection/)) |
| Apr 2025 | Security researchers publish analysis of prompt injection and "poisoned tools" enabling data exfiltration | ([source](https://en.wikipedia.org/wiki/Model_Context_Protocol)) |
| Mar 26, 2025 | Spec revision `2025-03-26`: introduced **Streamable HTTP** transport and **deprecated** the old HTTP+SSE transport (PR #206). Same day, OpenAI (Sam Altman) announced MCP support across products | ([source](https://modelcontextprotocol.io/docs/learn/architecture)) ([source](https://techcrunch.com/2025/03/26/openai-adopts-rival-anthropics-standard-for-connecting-ai-models-to-data/)) |
| Nov 25, 2024 | Anthropic announced and open-sourced MCP (spec revision `2024-11-05`), with local server support in Claude Desktop and an open-source server repo. Launch partners: Block, Apollo, Zed, Replit, Codeium, Sourcegraph | ([source](https://www.anthropic.com/news/model-context-protocol)) |

Notable quotes: Sam Altman, March 26, 2025: "People love MCP and we are excited to add support across our products. [It's] available today in the Agents SDK and support for [the] ChatGPT desktop app [and] Responses API [is] coming soon!" ([source](https://techcrunch.com/2025/03/26/openai-adopts-rival-anthropics-standard-for-connecting-ai-models-to-data/)). Block CTO Dhanji R. Prasanna at launch: "Open technologies like the Model Context Protocol are the bridges that connect AI to real-world applications, ensuring innovation is accessible." ([source](https://www.anthropic.com/news/model-context-protocol))

### 3. Why It's Needed (the problem it solves)

**Official framing.** Before MCP, "every new data source requires its own custom implementation, making truly connected systems difficult to scale." MCP replaces fragmented one-off integrations with a single protocol so a tool is built once and reused everywhere ([source](https://www.anthropic.com/news/model-context-protocol)).

**The M×N integration problem.** With M AI clients (models/apps) and N tools/data sources, the naive approach requires a custom connector for every client-tool pair: M × N integrations, all expensive and duplicative. A shared protocol collapses this to **M + N**: each tool ships one MCP server, each client ships one MCP client, and any client can reach any server. Adding a new tool means writing one server that is immediately usable by every existing client, with no client code changes because discovery is standardized ([source](https://medium.com/@edgar_muyale/the-m-n-integration-problem-explained-and-how-mcp-fixes-it-523083a59037)) ([source](https://www.softwareseni.com/how-mcp-reduces-ai-tool-integration-from-mxn-custom-connectors-to-mn-standard-interfaces/)). (Confidence: High on the framing; the M×N→M+N reduction is consistently described across multiple analyses and matches the official "build once, integrate everywhere" language ([source](https://modelcontextprotocol.io/introduction)).)

**The LSP precedent.** This is the same economic argument that drove LSP adoption in editors: standardize the interface once so the ecosystem stops re-implementing the same integration N times ([source](https://modelcontextprotocol.io/specification/2025-06-18)).

### 4. Use Cases (where it fits well)

Grounded in the official docs and ecosystem examples ([source](https://modelcontextprotocol.io/introduction)) ([source](https://modelcontextprotocol.io/docs/learn/architecture)):

- **One integration reused across many clients.** The strongest fit: you maintain a single MCP server and it works in Claude, ChatGPT, VS Code, Cursor, and more. "Build once and integrate everywhere" ([source](https://modelcontextprotocol.io/introduction)).
- **IDE / coding-agent tool access.** VS Code acts as a host instantiating clients per server (e.g., a Sentry server plus a local filesystem server) ([source](https://modelcontextprotocol.io/docs/learn/architecture)).
- **Personal-assistant style access to user apps.** Agents reaching Google Calendar and Notion to act as a personalized assistant ([source](https://modelcontextprotocol.io/introduction)).
- **Enterprise chat over many databases.** A chatbot connecting to multiple internal databases so users analyze data conversationally ([source](https://modelcontextprotocol.io/introduction)).
- **Dynamic tool discovery at runtime.** When the available toolset changes (per server state, dependencies, or user permissions), the `list_changed` notification keeps clients current without polling ([source](https://modelcontextprotocol.io/docs/learn/architecture)).
- **Mixed local + remote resources.** stdio servers for local machine access; Streamable HTTP servers for hosted/SaaS access, both speaking the same JSON-RPC data layer ([source](https://modelcontextprotocol.io/docs/learn/architecture)).

### 5. When It's NOT Needed (criticisms, alternatives, anti-patterns)

This is the under-covered half of the MCP conversation. The recurring theme across practitioners: **MCP's value is discoverability and reuse across many clients; if you don't need those, you're paying for them anyway.**

**Fixed, known toolset → use a direct API or CLI.** "For production API integrations where available tools are known, fixed, and controlled by the operator, MCP's discoverability overhead is pure cost: a direct API call or CLI command is faster, cheaper, uses less context, and is easier to authenticate and audit." ([source](https://nocodeapi.com/tutorials/you-probably-dont-need-mcp-when-direct-apis-beat-protocol-complexity/))

**AI as a peripheral feature → just call the LLM API.** "If AI is just an occasional enhancement, like adding a chatbot to a settings page, MCP's architecture is overkill: a simple call to your LLM provider's API with some context from your database is enough." ([source](https://www.improving.com/thoughts/when-mcp-is-not-the-right-choice/))

**Don't insert an AI middleman for deterministic work.** "If the task can be solved with a few lines of code calling a reliable API, introducing an AI middleman is usually adding unnecessary risk and latency: don't use an agent to fetch a user's profile from your database when your backend can do it directly in 5ms." ([source](https://nocodeapi.com/tutorials/you-probably-dont-need-mcp-when-direct-apis-beat-protocol-complexity/))

**Latency / throughput overhead.** MCP adds a protocol layer whose per-call cost compounds at volume. The general point: for known/fixed toolsets, "the protocol layer adds nothing except complexity" while direct API calls stay stateless with "nothing running, nothing to maintain" ([source](https://nocodeapi.com/tutorials/you-probably-dont-need-mcp-when-direct-apis-beat-protocol-complexity/)). For "customer-facing latency-sensitive applications," MCP's per-call overhead degrades real-time UI; sidecar/non-blocking patterns do better ([source](https://www.improving.com/thoughts/when-mcp-is-not-the-right-choice/)). (Note: a widely circulated benchmark claiming a 500-tool batch takes ~50s via API vs ~25min via MCP with 10-20x overhead surfaced in secondary summaries but was NOT found on the cited primary page during verification; treat those specific numbers as unverified.)

**Token / context-window cost of tool schemas.** Large tool-schema payloads eat context before the agent sees user input. Per Repello AI, citing Perplexity's CTO Denis Yarats, "in a typical MCP-heavy deployment, tool schemas and protocol overhead consume up to 72% of available context before any user intent is processed," and Perplexity moved toward APIs and CLIs over MCP for agents ([source](https://repello.ai/blog/mcp-vs-cli)). (Confidence: Medium: vendor-originated figure, single corroborating source after the originally cited page returned 404.)

**Minimal/static integrations → direct function calls or basic RAG.** "Stable, limited tools create bloated schemas without dynamic benefits; direct function calls or basic RAG pipelines handle these more efficiently." ([source](https://www.improving.com/thoughts/when-mcp-is-not-the-right-choice/))

**Regulated/enterprise gaps.** MCP's core spec historically lacked built-in SSO, audit trails, and fine-grained authorization, pushing regulated shops to custom gateways beyond the spec ([source](https://www.improving.com/thoughts/when-mcp-is-not-the-right-choice/)). The `2025-11-25` revision narrows this with OAuth-based authorization simplification and enterprise IdP policy controls, but enterprise hardening is still an active area ([source](https://blog.modelcontextprotocol.io/posts/2025-11-25-first-mcp-anniversary/)).

**Ungoverned sprawl.** Servers stood up "without clear ownership or rules" lead to inconsistent configs and credential sprawl; for small/early-stage use, "simple direct LLM API calls are usually enough." ([source](https://www.improving.com/thoughts/when-mcp-is-not-the-right-choice/))

**Rule of thumb (synthesis, Medium confidence):** Reach for MCP when (a) the same integration must serve multiple AI clients, or (b) the toolset is dynamic/discovery-driven. Skip it when the tools are few, fixed, latency-sensitive, or callable deterministically from your own backend.

### 6. Ecosystem & Existing Tooling (SDKs, registry, adoption)

**Official SDKs** (from the official SDK page) ([source](https://modelcontextprotocol.io/docs/sdk)):
- Tier 1: TypeScript, Python, C#, Go
- Tier 2: Java, Rust
- Tier 3: Swift, Ruby, PHP
- TBD: Kotlin

All SDKs support building servers (tools/resources/prompts), building clients, local and remote transports, and type-safe protocol compliance ([source](https://modelcontextprotocol.io/docs/sdk)).

**Registry.** Official MCP Registry launched in preview Sep 8, 2025 at `registry.modelcontextprotocol.io`, maintained by a registry working group (lead David Soria Parra) with contributors from Anthropic, GitHub, Block, and others. It is open source so organizations can build public/private sub-registries from the upstream data ([source](https://blog.modelcontextprotocol.io/posts/2025-09-08-mcp-registry-preview/)).

**Scale (one year in).** Servers went from "a few experimental ones" to thousands; registry grew to nearly 2,000 entries (407% growth from the September batch); community Discord has 2,900+ contributors with 100+ joining weekly; 58 maintainers including 9 core/lead in the steering group; 17 SEPs processed in roughly one quarter ([source](https://blog.modelcontextprotocol.io/posts/2025-11-25-first-mcp-anniversary/)).

**Cross-vendor adoption** (verified): Anthropic (origin), OpenAI (Mar 26, 2025), Google/Gemini (committed Apr 9, 2025; native SDK support shipped), Microsoft (Build 2025, May 19, 2025, across Windows 11/GitHub/Copilot Studio/Azure AI Foundry/Dynamics 365). Client support spans Claude, ChatGPT, VS Code, and Cursor ([source](https://modelcontextprotocol.io/introduction)) ([source](https://techcrunch.com/2025/03/26/openai-adopts-rival-anthropics-standard-for-connecting-ai-models-to-data/)) ([source](https://techcrunch.com/2025/04/09/google-says-itll-embrace-anthropics-standard-for-connecting-ai-models-to-data/)) ([source](https://blogs.windows.com/windowsdeveloper/2025/05/19/advancing-windows-for-ai-development-new-platform-capabilities-and-tools-introduced-at-build-2025/)).

### 7. Security Considerations

MCP's own spec states tools "represent arbitrary code execution and must be treated with appropriate caution," that tool annotations should be considered untrusted unless from a trusted server, and that hosts must obtain explicit user consent before invoking tools or exposing data. It also notes MCP "cannot enforce these security principles at the protocol level" ([source](https://modelcontextprotocol.io/specification/2025-06-18)).

Key real-world risk categories (verified):

- **Prompt injection via tool output.** Simon Willison frames the danger as the toxic combination of private data + untrusted instructions + an exfiltration vector. He stresses these issues are "not inherent to the MCP protocol itself: they're present any time we provide tools to an LLM that can be exposed to untrusted inputs," and that no convincing general mitigation for prompt injection exists ([source](https://simonwillison.net/2025/Apr/9/mcp-prompt-injection/)). (This maps to Willison's broader "lethal trifecta" concept.)
- **Tool poisoning.** Malicious instructions hidden in a tool description (visible to the model, not the user) can trick the agent, e.g., reading `~/.cursor/mcp.json` and exfiltrating it ([source](https://simonwillison.net/2025/Apr/9/mcp-prompt-injection/)).
- **Rug pulls / tool shadowing.** A tool can mutate its definition after install (safe on Day 1, malicious by Day 7) or override calls to a trusted tool. Mitigation: clients should pin and diff tool descriptions and alert on change ([source](https://simonwillison.net/2025/Apr/9/mcp-prompt-injection/)).
- **Supply chain.** Third-party servers are an install-time risk; a critical OS-command-injection CVE in `mcp-remote` (CVE-2025-6514, 437,000+ downloads) is cited as evidence of the supply-chain dimension ([source](https://www.truefoundry.com/blog/mcp-security-risks-best-practices)). (Confidence: Medium: specific CVE/download figure from a single vendor blog; not independently verified against the CVE record in this pass.)

This aligns with Pete's own workspace audit classifying web-search and community-data MCP servers (searxng, threatintel, github) as higher injection risk than internal authenticated servers.

## Confidence Assessment

### High Confidence
- MCP announced Nov 25, 2024 by Anthropic; "USB-C for AI" framing; client-server architecture; primitives (tools/resources/prompts plus client-side sampling/elicitation/roots/logging); JSON-RPC 2.0; LSP inspiration. (Official Anthropic + modelcontextprotocol.io.)
- Transports: stdio + Streamable HTTP; HTTP+SSE deprecated in `2025-03-26`. (Official architecture doc.)
- OpenAI adoption Mar 26, 2025 (Altman quote verbatim); Google commitment Apr 9, 2025; Microsoft Build May 19, 2025. (TechCrunch + Microsoft blogs + Wikipedia.)
- Spec revisions `2024-11-05` → `2025-03-26` → `2025-06-18` → `2025-11-25`. (Official spec + anniversary blog.)
- Official SDK list and tiers; registry launch Sep 8, 2025; donation to Linux Foundation AAIF Dec 2025. (Official sources + Wikipedia.)
- The M×N → M+N framing and the "when not to use" patterns (fixed toolset, peripheral AI feature, deterministic work). (Multiple corroborating analyses.)

### Medium Confidence
- Specific latency figures (10-20x overhead; 500-tool batch 50s vs 25min): single-source benchmark.
- The "72% of context window" tool-schema figure attributed to Perplexity's CTO (Denis Yarats), per Repello AI: vendor-originated; the alternative source originally cited (junia.ai) returned 404 during verification and was removed.
- CVE-2025-6514 / 437,000 downloads and the "30%+ of 1,800 servers have an exploitable vuln" stat: single secondary sources, not verified against primary CVE/paper records.
- Registry "407% growth / nearly 2,000 entries" exact figures: from the official anniversary blog (reliable) but point-in-time.

## Open Questions

- Has the `2025-11-25` authorization work (OAuth simplification, enterprise IdP controls) materially closed the SSO/audit gaps regulated shops cited? Not assessed here.
- Independent reproduction of the latency/token-overhead benchmarks against a current SDK build would strengthen the "when not needed" case beyond single-vendor claims.
- Exact removal timeline for HTTP+SSE across major hosted servers (some vendor deadlines cited for 2026) was not exhaustively verified.

## Sources

**Official Docs / Spec**
- [modelcontextprotocol.io/introduction](https://modelcontextprotocol.io/introduction)
- [modelcontextprotocol.io/docs/learn/architecture](https://modelcontextprotocol.io/docs/learn/architecture)
- [modelcontextprotocol.io/specification/2025-06-18](https://modelcontextprotocol.io/specification/2025-06-18)
- [modelcontextprotocol.io/docs/sdk](https://modelcontextprotocol.io/docs/sdk)
- [MCP Registry preview (blog)](https://blog.modelcontextprotocol.io/posts/2025-09-08-mcp-registry-preview/)
- [One Year of MCP: November 2025 spec release (blog)](https://blog.modelcontextprotocol.io/posts/2025-11-25-first-mcp-anniversary/)

**Vendor Announcements**
- [Anthropic: Introducing the Model Context Protocol](https://www.anthropic.com/news/model-context-protocol)
- [Microsoft: Advancing Windows for AI development (Build 2025)](https://blogs.windows.com/windowsdeveloper/2025/05/19/advancing-windows-for-ai-development-new-platform-capabilities-and-tools-introduced-at-build-2025/)
- [Microsoft Foundry: MCP support (preview) in Azure AI Foundry Agent Service](https://devblogs.microsoft.com/foundry/announcing-model-context-protocol-support-preview-in-azure-ai-foundry-agent-service/)

**Technical Analyses**
- [Simon Willison: MCP has prompt injection security problems](https://simonwillison.net/2025/Apr/9/mcp-prompt-injection/)
- [Improving: When MCP Is Not The Right Choice](https://www.improving.com/thoughts/when-mcp-is-not-the-right-choice/)
- [NoCodeAPI: You Probably Don't Need MCP](https://nocodeapi.com/tutorials/you-probably-dont-need-mcp-when-direct-apis-beat-protocol-complexity/)
- [Repello AI: MCP vs CLI](https://repello.ai/blog/mcp-vs-cli)
- [TrueFoundry: MCP Security Risks & Best Practices](https://www.truefoundry.com/blog/mcp-security-risks-best-practices)
- [The M×N Integration Problem Explained (Medium)](https://medium.com/@edgar_muyale/the-m-n-integration-problem-explained-and-how-mcp-fixes-it-523083a59037)
- [SoftwareSeni: M×N to M+N](https://www.softwareseni.com/how-mcp-reduces-ai-tool-integration-from-mxn-custom-connectors-to-mn-standard-interfaces/)

**News Coverage**
- [TechCrunch: OpenAI adopts rival Anthropic's standard](https://techcrunch.com/2025/03/26/openai-adopts-rival-anthropics-standard-for-connecting-ai-models-to-data/)
- [TechCrunch: Google to embrace Anthropic's standard](https://techcrunch.com/2025/04/09/google-says-itll-embrace-anthropics-standard-for-connecting-ai-models-to-data/)
- [Wikipedia: Model Context Protocol](https://en.wikipedia.org/wiki/Model_Context_Protocol)

## Update History

- 2026-06-14 8:32 AM ET: Initial report (Fresh Mode).

## How This Report Was Generated

Generated by the Research agent (anti-hallucination guardrails: every factual claim grounded in a fetched source, confidence-flagged, "I don't know" over guessing). Tier 1 searxng tools were unavailable this session; primary sourcing was direct WebFetch of official MCP/Anthropic pages plus WebSearch cross-checks (Tier 3) with WebFetch verification of the load-bearing URLs. Timestamps in ET. Spawned `research-polish` (linkify + ToC) and `research-verify` (independent URL/claim verification) after drafting. Not published to git.
