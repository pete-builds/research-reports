---
title: "MCP Server Ideas for Higher Education on a LiteLLM Gateway"
date: 2026-04-28
updated: 2026-04-28 13:51 EDT
summary: "A scored map of MCP servers worth deploying behind a university LiteLLM gateway, distinguishing community-ready servers (GitHub, Atlassian, Microsoft 365, Snowflake, Grafana, arXiv, Canvas LMS) from the higher-ed gaps (institutional repository, Box, Zoom, library systems) where an in-house build is the right move."
---

## TL;DR

The Model Context Protocol (MCP) ecosystem has matured to the point where universities running a LiteLLM AI gateway can deploy a centrally-administered MCP layer on top of it with low marginal cost. Most enterprise MCP servers a developer or sysadmin would want (GitHub, Microsoft 365, Atlassian, Snowflake, Grafana, Datadog, BigQuery) are now official vendor builds, OAuth-bearing, and remote-hosted: the right move is to **buy** them. The higher-ed gaps are concentrated in scholarly tooling (OpenAlex, Crossref, ORCID, DSpace) and library systems (Alma, FOLIO, EBSCO), where no production MCP exists and clean public APIs make these the highest-leverage **build** targets for a university IT or library team. Canvas LMS sits in the middle: three credible community implementations exist, and the right pattern is fork-and-harden under FERPA review. Anthropic's Skills launch (October 2025) further sharpens the question: reserve MCP for live, authenticated APIs, and use Skills for institutional workflow knowledge.

## Opening

This report answers a practical question: what MCP servers should a university stand up behind its LiteLLM AI gateway, and which are worth building in-house versus adopting from the community? It scores each candidate against six dimensions (audience, build vs buy, gateway fit, prompt-injection risk, FERPA/HIPAA exposure, priority) across six audience categories (developers, sysadmins, library/scholarly, instructional, data/analytics, security/governance). The recommendations are evidence-based: every named MCP server has been verified by fetching its repo or vendor doc, and confidence levels are flagged where the evidence is thin. The audience is a university CIO, AI hub lead, library systems team, or IT architect deciding what to deploy.

## Index

1. [Current Status](#current-status) — Snapshot of LiteLLM, Anthropic, vendor, and ecosystem state
2. [The university LiteLLM gateway context](#1-the-university-litellm-gateway-context) — What this report assumes about the deployment
3. [LiteLLM as an MCP gateway: what it can do](#2-litellm-as-an-mcp-gateway-what-it-can-do) — Capabilities, transports, OAuth, observability
4. [The MCP server landscape in 2026](#3-the-mcp-server-landscape-in-2026) — Three phases, the registry, Skills vs MCP
5. [Recommended MCP servers — by audience](#4-recommended-mcp-servers-for-higher-education--by-audience)
   - 5.1 [Developer / engineer](#41-developer--engineer-mcps)
   - 5.2 [Sysadmin / SRE](#42-sysadmin--sre-mcps)
   - 5.3 [Library / scholarly](#43-library--scholarly-mcps)
   - 5.4 [Instructional / student-facing](#44-instructional--student-facing-mcps)
   - 5.5 [Data / analytics](#45-data--analytics-mcps)
   - 5.6 [Security / governance](#46-security--governance-mcps)
6. [The "build vs buy" matrix](#5-the-build-vs-buy-matrix) — Cross-cut summary and top in-house targets
7. [Industry trends shaping the recommendation](#6-industry-trends-shaping-the-recommendation) — Five forces
8. [Gateway-specific architectural guidance](#7-gateway-specific-architectural-guidance) — Operating recommendations for a LiteLLM-fronted MCP layer
9. [Confidence Assessment](#confidence-assessment) — Evidence rating per area
10. [Open Questions](#open-questions) — Unresolved decisions for the implementer
11. [Sources](#sources) — All citations grouped by category

## Current Status

- LiteLLM ships a production **MCP Server Gateway** that lets the proxy act as both an MCP client (to upstream tool servers) and an MCP server (to LLMs and IDEs like Claude Code), with permission scoping by Key, Team, and Organization, and OAuth 2.0 support for both interactive (PKCE) and machine-to-machine flows ([docs.litellm.ai/docs/mcp](https://docs.litellm.ai/docs/mcp), [docs.litellm.ai/docs/mcp_oauth](https://docs.litellm.ai/docs/mcp_oauth), [DeepWiki: BerriAI/litellm 8.10](https://deepwiki.com/BerriAI/litellm/8.10-mcp-server-gateway)).
- LiteLLM v1.80.18+ uses MCP protocol version `2025-11-25`, supports stdio/SSE/streamable-HTTP transports, prefixes tool names with the server alias to avoid collisions, and logs every `mcp_server_tool_call` through the proxy's existing observability pipeline ([DeepWiki: BerriAI/litellm 8.10](https://deepwiki.com/BerriAI/litellm/8.10-mcp-server-gateway)).
- Anthropic's `modelcontextprotocol/servers` reference repo has shrunk to seven actively maintained reference servers (Everything, Fetch, Filesystem, Git, Memory, Sequential Thinking, Time); twelve former references including GitHub, GitLab, Slack, Postgres, and Sentry are now archived or maintained by the vendors themselves ([github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)).
- Vendors have taken over the most important enterprise MCP servers: GitHub ([github.blog/changelog](https://github.blog/changelog/2026-01-28-github-mcp-server-new-projects-tools-oauth-scope-filtering-and-new-features/)), Atlassian Rovo, Microsoft 365, Snowflake, Grafana, Datadog, and Elastic all ship official servers, most with OAuth and most either remote-hosted or production-ready.
- Higher-ed-specific MCPs are sparse but real: Canvas LMS has multiple credible community implementations (the most mature is Vishal Sachdev's at 115 stars with 88 tools and a hosted preview at `mcp.illinihunt.org` ([github.com/vishalsachdev/canvas-mcp](https://github.com/vishalsachdev/canvas-mcp))), arXiv has Pearl Labs' server at 2.6k stars ([github.com/blazickjp/arxiv-mcp-server](https://github.com/blazickjp/arxiv-mcp-server)), PubMed has a community server at 108 stars ([github.com/JackKuo666/PubMed-MCP-Server](https://github.com/JackKuo666/PubMed-MCP-Server)). Library systems (Alma, FOLIO, EBSCO) have no production MCP server we could verify.
- Anthropic's own product strategy is bifurcating: **Skills** (folders of instructions, scripts, and resources, launched October 2025, Pro/Max/Team/Enterprise + API and Claude Code) handle workflow knowledge that doesn't need a long-running server, while **MCP** continues to be the protocol for live tool calls ([claude.com/blog/skills](https://claude.com/blog/skills)). Picking the right one matters for any university's roadmap.
- Cloudflare's remote MCP hosting (workers-oauth-provider, McpAgent, mcp-remote, AI Playground), launched March 25, 2025, is the dominant pattern for OAuth-protected remote servers ([blog.cloudflare.com](https://blog.cloudflare.com/remote-model-context-protocol-servers-mcp/)). It's the "build your own remote MCP" reference architecture.

## Findings

### 1. The university LiteLLM gateway context

The pattern this report assumes is a university-operated LiteLLM proxy that fronts inference for Claude Code and other agentic IDEs. The proxy exposes both `/v1/chat/completions` (OpenAI format) and `/v1/messages` (Anthropic format), and users authenticate with virtual keys. Stanford and Carnegie Mellon both publicly run gateways on LiteLLM ([uit.stanford.edu](https://uit.stanford.edu/service/ai-api-gateway), [cmu.edu](https://www.cmu.edu/computing/services/ai/tools/ai-gateway/index.html)), and the same architecture is being adopted at peer institutions. The deployment question this report addresses is whether to stand up an MCP gateway alongside the LLM gateway (using LiteLLM's built-in MCP feature, or a separate Webrix/MintMCP-style purpose-built gateway), and which servers should sit behind it.

### 2. LiteLLM as an MCP gateway: what it can do

LiteLLM's MCP Server Gateway is no longer experimental: it's documented at [docs.litellm.ai/docs/mcp](https://docs.litellm.ai/docs/mcp) and the implementation is described in detail at [DeepWiki](https://deepwiki.com/BerriAI/litellm/8.10-mcp-server-gateway). The relevant capabilities for a university deployment:

- **Dual role**: LiteLLM is both an MCP client (it can call upstream MCP servers on behalf of any LLM it routes to) and an MCP server (Claude Code, Cursor, VS Code agents can connect to LiteLLM as if it were one big MCP server).
- **Three-layer access control**: Virtual Key authentication, then permission inheritance from Team, then intersection of Key+Team permissions. Servers can be scoped per Key, Team, or Organization. This maps directly to a college / department / project scoping model ([DeepWiki: BerriAI/litellm 8.10](https://deepwiki.com/BerriAI/litellm/8.10-mcp-server-gateway)).
- **OAuth 2.0**: Both PKCE (interactive, per-user tokens for Claude Code) and client_credentials (M2M, service tokens). Tokens are cached server-side; the gateway brokers OAuth discovery, registration, and refresh ([docs.litellm.ai/docs/mcp_oauth](https://docs.litellm.ai/docs/mcp_oauth)). This is the feature that makes a *centrally-administered* MCP layer feasible at all.
- **Transports**: stdio, SSE, and streamable-HTTP. Stdio servers run as managed subprocesses; remote servers connect via HTTP/SSE. Standard FastMCP + httpx + Docker SSE patterns slot in directly.
- **Observability**: every tool call goes through the standard proxy logging pipeline (`mcp_server_tool_call` handler). Anything LiteLLM logs today (cost, latency, user) extends to MCP calls automatically ([DeepWiki](https://deepwiki.com/BerriAI/litellm/8.10-mcp-server-gateway)).
- **SEP-986 naming compliance** is enforced with warnings for legacy non-compliant server names; future protocol versions may block them ([docs.litellm.ai/docs/mcp](https://docs.litellm.ai/docs/mcp)). Anything built in-house should use the new naming convention from day one.

The trade-off: LiteLLM is "good enough" as an MCP gateway, but it's not a purpose-built MCP gateway like MintMCP or Webrix. The MintMCP comparison piece is candid: "If you're building an application that needs MCP servers as part of its functionality, LiteLLM makes sense" ([mintmcp.com/blog](https://www.mintmcp.com/blog/mintmcp-vs-litellm-mcp-comparison)). Where LiteLLM is already the LLM gateway, the marginal cost of turning on the MCP feature is low and the integration story is one component instead of two.

### 3. The MCP server landscape in 2026

The ecosystem has moved through three phases in eighteen months:

- **Phase 1 (Nov 2024 – early 2025)**: Anthropic published reference servers in `modelcontextprotocol/servers`. Mostly stdio, mostly local, mostly toy.
- **Phase 2 (mid-2025)**: Vendors took over their own connectors. GitHub rewrote Anthropic's reference into Go and made it official ([github.blog/changelog](https://github.blog/changelog/2025-04-04-github-mcp-server-public-preview/)). Cloudflare shipped remote MCP hosting on Workers with built-in OAuth (March 25, 2025; [blog.cloudflare.com](https://blog.cloudflare.com/remote-model-context-protocol-servers-mcp/)). Anthropic launched the Connectors Directory (July 14, 2025) at `claude.ai/directory` with Notion, Canva, Stripe, Linear, Figma and partners ([claude.com/blog/connectors-directory](https://claude.com/blog/connectors-directory)).
- **Phase 3 (late 2025 – 2026)**: Remote-first, OAuth-first. Atlassian's Rovo MCP is GA, OAuth-only, with rate limits scaled by plan ([atlassian.com/platform/remote-mcp-server](https://www.atlassian.com/platform/remote-mcp-server)). Microsoft's M365 MCP Server for Claude went live with delegated Graph permissions and Azure Enterprise app registrations ([office365itpros.com](https://office365itpros.com/2026/04/08/microsoft-365-connector-for-claude/)). Snowflake, Grafana, Datadog, Elastic, and Google (BigQuery via the renamed Toolbox) all ship official servers.

The official Anthropic registry at [registry.modelcontextprotocol.io](https://registry.modelcontextprotocol.io/) is now operational; the `modelcontextprotocol` GitHub org carries 41 repositories including SDKs in 10 languages, an Inspector tool with 9.6k stars, and the Servers collection at 85k stars ([github.com/orgs/modelcontextprotocol/repositories](https://github.com/orgs/modelcontextprotocol/repositories)). PulseMCP and mcp.so / mcpservers.org are the community indexes; PulseMCP is run by an MCP Steering Committee member ([pulsemcp.com](https://www.pulsemcp.com/)).

The other strategic shift: **Anthropic Skills**. Launched October 16, 2025, Skills are folders of instructions, scripts, and resources that load on demand ([claude.com/blog/skills](https://claude.com/blog/skills)). For workflow knowledge that doesn't need a live API call (e.g., "how this institution formats internal change tickets" or "the IT report template"), Skills are the better tool. MCP servers should be reserved for live data and authenticated APIs.

### 4. Recommended MCP servers for higher education — by audience

Format: **What it does | Audience | Build vs buy | LiteLLM gateway fit | Risk | Priority**

#### 4.1 Developer / engineer MCPs

| MCP | What it does | Audience | Build vs buy | Gateway fit | Risk | Priority |
|---|---|---|---|---|---|---|
| **GitHub Enterprise** ([github/github-mcp-server](https://github.com/github/github-mcp-server)) | 60+ tools across 20+ categories: repos, issues, PRs, Actions, code security; OAuth or PAT; remote at `api.githubcopilot.com/mcp/` for github.com, local Docker for GHES | All developers, AI hub staff, central IT, research engineers | **Buy.** GA, MIT-licensed, GHES support via local mode | Excellent. Stdio Docker via LiteLLM, or remote via per-user OAuth. `--read-only` flag for safer defaults | **Medium.** Returns repo content, issue text, PR bodies; treat as injection surface | **P0** |
| **Microsoft Graph / M365** (Anthropic's official `M365 MCP Server for Claude` Enterprise app, 07c030f6-5743-41b7-ba00-0a6e85f37c17) | SharePoint, OneDrive, Outlook, Teams, Calendar via delegated Graph permissions ([office365itpros.com](https://office365itpros.com/2026/04/08/microsoft-365-connector-for-claude/), [support.claude.com](https://support.claude.com/en/articles/12542951-enable-and-use-the-microsoft-365-connector)) | Anyone in institutional M365: most administrative staff, faculty, librarians | **Buy** (Anthropic's connector) plus consider a **read-only Graph MCP** like [eesb99/msgraph-mcp](https://github.com/eesb99/msgraph-mcp) for unattended Claude Code workflows | Anthropic connector is for Claude apps, not Claude Code through LiteLLM. For Claude Code on the gateway, the institution needs to deploy its own Graph MCP or proxy [Lokka](https://github.com/merill/lokka)-style servers. Requires admin-consent workflow with central IT | **Medium.** Email/Teams content is high-sensitivity; FERPA implications if student data lands in OneDrive/Teams | **P0** for M365-using staff |
| **Atlassian Jira/Confluence/Compass** (Atlassian Rovo MCP, GA, OAuth, [atlassian.com/platform/remote-mcp-server](https://www.atlassian.com/platform/remote-mcp-server)) | Search/fetch/edit Jira issues, Confluence pages, Compass scorecards | IT teams that use Atlassian (varies by department) | **Buy.** GA, hosted by Atlassian, OAuth only, 1000 calls/hour at Standard | Easy. Add as remote MCP to LiteLLM with OAuth client_credentials or PKCE per user | **Medium.** Issue/page content can carry injected instructions | **P1** if Atlassian is in use; otherwise N/A |
| **Slack** (Zencoder maintains the former Anthropic reference at [github.com/zencoderai/slack-mcp-server](https://github.com/zencoderai/slack-mcp-server)) | List channels, post, react, history, threads, users | Teams that operate in Slack (universities often run a patchwork by college) | **Buy** for the basic version; OAuth scope is bot-token style; only 67 stars, v0.0.1 from July 2025 — vet before deploying | OK. Stdio via Docker. For unified Slack Enterprise Grid, more robust solution may be needed | **Medium.** Reads channel content. Bot-token model requires careful scope management | **P2.** Wait for Slack to ship an official remote MCP, or for Zencoder's to mature |
| **Box** | No verified production MCP server found in this research | Institution-wide file collaboration | **Build** or wait. Anthropic's Box-as-Skills partnership ([claude.com/blog/skills](https://claude.com/blog/skills)) signals Box may go the Skills route. | If built: standard FastMCP pattern, OAuth 2.0 against Box | **High.** Box stores institutional documents, possibly FERPA/HIPAA | **P2.** Watch for official Box MCP or Box Skill |
| **ServiceNow** | No verified official MCP server found in this research; community projects exist on PulseMCP | ServiceNow customers (central IT) | **Build** if needed. Pattern is straightforward (REST API + FastMCP) | Standard | **Low** if read-only on ticket text; **medium** for write actions | **P3** |

#### 4.2 Sysadmin / SRE MCPs

| MCP | What it does | Audience | Build vs buy | Gateway fit | Risk | Priority |
|---|---|---|---|---|---|---|
| **Grafana** ([github.com/grafana/mcp-grafana](https://github.com/grafana/mcp-grafana)) | Dashboards, datasources (Prometheus/Loki/ClickHouse/Elastic), alerts, incidents, on-call; Apache 2.0; 2.9k stars; 47 releases | Central IT SRE, AI hub ops | **Buy.** Official, mature, multi-transport | Excellent. uvx, Docker, or Helm. SSE supported | **Low.** Read-only by default; "run panel query tools are disabled by default" | **P0** if Grafana is deployed |
| **Datadog** (official; [docs.datadoghq.com/bits_ai/mcp_server/](https://docs.datadoghq.com/bits_ai/mcp_server/)) | Query observability data, metrics, logs, APM | IT teams using Datadog | **Buy.** Official. Notes "under significant development" | OK. Will proxy through LiteLLM | **Low** read-only | **P0** if Datadog is in use |
| **Elastic / Elasticsearch** ([github.com/elastic/mcp-server-elasticsearch](https://github.com/elastic/mcp-server-elasticsearch)) | list_indices, get_mappings, search, ES\|QL, get_shards | Central IT search/log teams | **Buy** but note **deprecated** in favor of Elastic Agent Builder MCP endpoint in Elasticsearch 9.2+ | OK as Docker; plan migration to Agent Builder | **Low** read-only | **P2** — wait for Agent Builder if you can |
| **Kubernetes / OpenShift** | Multiple community MCPs exist; no clear single official server (this research did not verify any specific repo's currency) | Central IT platform teams | **Verify before deploying.** Best to pick one community server and pin a version, or build a minimal kubectl-wrapping FastMCP | Standard | **High.** Kubernetes has cluster-admin reach if scoped wrong; treat as crown jewels | **P1** for ops teams; build cautiously |
| **AWS / Azure read-only** | AWS Bedrock AgentCore servers can be added behind LiteLLM with native AWS SigV4 ([DeepWiki](https://deepwiki.com/BerriAI/litellm/8.10-mcp-server-gateway)). Azure has Microsoft's Lokka and Microsoft Agent 365 Work IQ MCP ([learn.microsoft.com](https://learn.microsoft.com/en-us/microsoft-agent-365/tooling-servers-overview)) | Cloud teams | **Buy** (cloud-vendor-managed) | Excellent for Bedrock; Azure path requires Entra app registration | **Medium.** Cloud APIs are high-blast-radius | **P1** |
| **Infra monitoring (Uptime Kuma, Portainer, Synology, Pi-hole)** | Container/host status, network DNS, NAS storage | Central IT ops, AI hub ops | **Build** small FastMCP wrappers per system; community examples exist | Direct via SSE behind LiteLLM | **Low** (internal, read-only) | **P3** |

#### 4.3 Library / scholarly MCPs

| MCP | What it does | Audience | Build vs buy | Gateway fit | Risk | Priority |
|---|---|---|---|---|---|---|
| **arXiv** ([github.com/blazickjp/arxiv-mcp-server](https://github.com/blazickjp/arxiv-mcp-server)) | Search, download, read, semantic-search, citation graph (via Semantic Scholar), watch_topic; Apache 2.0; 2.6k stars; explicitly warns about prompt injection in paper content | Researchers, library staff supporting research | **Buy.** Mature, well-documented, security-aware | Easy. uvx or Docker; stdio or HTTP | **High.** Repo's own README warns: "paper content retrieved from arXiv is untrusted external input" — wrap in read-only configs, treat as data | **P0** for research-supporting workflows |
| **PubMed** ([github.com/JackKuo666/PubMed-MCP-Server](https://github.com/JackKuo666/PubMed-MCP-Server)) | Article search, metadata by PMID, PDF fetch attempts, deep-paper analysis; MIT; 108 stars | Medical schools, life-sciences researchers, library liaisons | **Buy with caveats.** Smaller community; verify currency before deploying institutionally | Easy. FastMCP-based, Python | **Medium.** PubMed terms of service apply; respect rate limits | **P1** for institutions with health-sciences focus |
| **OpenAlex / Crossref** | No verified production MCP server found in this research; OpenAlex has a clean documented API at [developers.openalex.org](https://developers.openalex.org/) | Library, scholarly comms, bibliometrics, institutional research strategy | **Build.** Both APIs are well-defined, no auth required, ideal in-house target. Wrap OpenAlex's works/authors/institutions/concepts endpoints; add citation traversal | Standard FastMCP, no auth needed for read | **Low.** Public open data, attacker-controlled text via abstract/title is the main injection vector | **P0** — high leverage, low effort |
| **Alma / FOLIO / EBSCO** | No verified production MCP server found in this research | Library catalog ops, e-resource staff | **Build** for any of these. Alma has the Ex Libris APIs; FOLIO has its module APIs; EBSCO has its standard query APIs | Standard | **Medium.** Patron data must stay out (FERPA); restrict to bibliographic and holdings data | **P1** for university libraries |
| **ORCID** | Public API is open; no verified MCP server found in this research | Faculty, researchers, RIS-style profile work | **Build.** Trivial in-house target wrapping the ORCID public API | Standard | **Low.** Public profile data | **P2** |
| **DSpace / institutional repository** | DSpace 7+ has a documented REST API; no verified MCP server found in this research | Library digital scholarship, faculty depositors | **Build.** FastMCP wrapper over the DSpace REST API | Standard | **Low** for read; **medium** for write (deposit) | **P2** |

#### 4.4 Instructional / student-facing MCPs

| MCP | What it does | Audience | Build vs buy | Gateway fit | Risk | Priority |
|---|---|---|---|---|---|---|
| **Canvas LMS** ([github.com/vishalsachdev/canvas-mcp](https://github.com/vishalsachdev/canvas-mcp), 88 tools, 8 agent skills, hosted preview at `mcp.illinihunt.org`, MIT, 115 stars; alternates: [r-huijts/canvas-mcp](https://github.com/r-huijts/canvas-mcp), [DMontgomery40/mcp-canvas-lms](https://github.com/DMontgomery40/mcp-canvas-lms), 54 tools v2.2) | Faculty, teaching assistants, learning designers, course-support staff | **Buy** the Sachdev server (most mature, FERPA-conscious) and **fork** for institution-specific Canvas tenancy. Or **build** if the library or center for teaching has stricter requirements | Easy. Python, stdio or HTTP. Watch for the prompt-injection surface in student-submitted content | **High.** FERPA. Student names, grades, submissions. Read-only by default; instructor-only access enforcement | **P1** with strict FERPA review |
| **Zoom / Webex** | No verified production MCP server found; Zoom has a Marketplace and OAuth API | Anyone running meetings | **Build.** Both have well-documented OAuth REST APIs | Standard via OAuth | **Medium.** Recordings can contain sensitive content | **P2** |
| **Qualtrics** | No verified production MCP server found | Survey researchers, IRB-cleared studies | **Build** if there's demand. Read-only against survey results, with PII filtering | Standard | **High.** Survey responses can be PHI/PII; needs IRB conversation | **P3** |

#### 4.5 Data / analytics MCPs

| MCP | What it does | Audience | Build vs buy | Gateway fit | Risk | Priority |
|---|---|---|---|---|---|---|
| **Google BigQuery** (via [googleapis/genai-toolbox](https://github.com/googleapis/genai-toolbox), Apache 2.0) | MCP server for BigQuery (and 20+ other databases) with prebuilt + custom tools, IAM-aware, OpenTelemetry observability | Research data, institutional analytics | **Buy.** Official Google, multi-database support | Excellent. Binary, Docker, Helm, npm | **Medium.** Query against potentially sensitive datasets; rely on IAM scoping | **P0** if BigQuery is in use |
| **Snowflake** ([github.com/Snowflake-Labs/mcp](https://github.com/Snowflake-Labs/mcp)) | Cortex Search, Cortex Analyst, Cortex Agent, object management, SQL execution, semantic views; Apache 2.0; honors RBAC | Teams on Snowflake | **Buy.** Official Snowflake-Labs | Excellent. uvx, multiple transports, supports SSO/OAuth/MFA | **Medium.** SQL execution permission scoping is critical | **P0** if Snowflake is in use |
| **Tableau / Looker / Power BI** | Tableau has a community MCP; no clear official server verified in this research | BI analysts, institutional research | **Verify** before deploying any community Tableau MCP. Or **build** read-only against the Tableau REST API | Standard | **Medium.** Workbook content may include sensitive | **P2** |
| **Institutional research / data warehouse** | No public MCP — internal data warehouse | Institutional research offices | **Build** if there's specific demand | Standard | **High.** IR data is often confidential | **P2** with stewardship review |

#### 4.6 Security / governance MCPs

| MCP | What it does | Audience | Build vs buy | Gateway fit | Risk | Priority |
|---|---|---|---|---|---|---|
| **CrowdStrike** | No verified production MCP server | Information security office / SOC if CrowdStrike is deployed | **Build** read-only against the Falcon API | Standard | **Medium.** Threat detection data | **P3** |
| **Splunk Enterprise Security** | No clear official MCP server found in this research; Cisco-Splunk has been emphasizing AI workflows | SOC | **Build** if needed; Splunk REST API is well-documented | Standard | **Medium** | **P2** |
| **Threat intel feeds** (CISA KEV, MITRE, OTX) | CISA Known Exploited Vulnerabilities, MITRE ATT&CK, OTX pulses | Security analysts, IT ops, infosec teams | **Build.** Wrap CISA KEV JSON, MITRE ATT&CK STIX, OTX API behind FastMCP; community examples exist | Direct | **Medium.** Community-contributed pulse data is attacker-influenced; treat all returned text as untrusted | **P2** |
| **DLP / sensitivity labels** | Microsoft Purview API exists; no verified MCP server | Compliance, governance | **Build** if the AI hub takes on AI-data-governance scope | Standard | **High.** Purview itself is sensitive | **P3** |

### 5. The "build vs buy" matrix

| Domain | Buy (verified production MCP) | Build (in-house project) | Wait / unclear |
|---|---|---|---|
| Developer | GitHub, Atlassian, M365 (Anthropic connector), Datadog, Grafana, Snowflake, BigQuery (via Toolbox), Elastic | Read-only Graph for Claude Code, ServiceNow if needed, Box | Slack (Zencoder server young) |
| Sysadmin | Grafana, Datadog, Elastic (then migrate), Bedrock AgentCore | Institution-scope infra monitoring wrappers (Uptime Kuma, Portainer, Synology, Pi-hole) if central IT wants them | Kubernetes (community), Azure read-only |
| Library / scholarly | arXiv (with hardening), PubMed (with caveats) | OpenAlex, Crossref, ORCID, Alma, FOLIO, EBSCO, DSpace | — |
| Instructional | Canvas (community, FERPA-hardened) | Zoom, Webex, Qualtrics | — |
| Data | BigQuery, Snowflake | Tableau (build read-only), institutional research systems | — |
| Security | (none verified for buy at university scale) | CrowdStrike, Purview, CISA KEV, MITRE | Splunk |

The **highest-leverage in-house build targets** are the ones with no community MCP and a clean public API:
1. **OpenAlex** — global research catalog, no auth, ideal for library/research workflows
2. **Crossref** — DOI metadata, citations, no auth
3. **ORCID** — public profile data, no auth
4. **DSpace** — institutional repository, REST API
5. **Read-only Graph for unattended Claude Code** — fills the Anthropic-connector gap

### 6. Industry trends shaping the recommendation

1. **Remote-first MCP with OAuth is the new default.** Cloudflare's pattern (workers-oauth-provider + McpAgent + mcp-remote) became the reference architecture in 2025 ([blog.cloudflare.com](https://blog.cloudflare.com/remote-model-context-protocol-servers-mcp/)). Atlassian, GitHub, M365, Snowflake, Datadog all ship hosted/remote OAuth MCP. Local-stdio is becoming the exception, not the norm. Universities should expect MCP servers to be remote, OAuth-bearing, and per-user-scoped.

2. **Vendors are taking over their own MCP servers.** Anthropic's `modelcontextprotocol/servers` repo has *shrunk* — twelve former references (GitHub, GitLab, Slack, Postgres, Sentry, etc.) are archived because the vendor took over ([github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)). The lesson: **don't build an MCP server for a vendor's product unless you've checked they aren't already building one.**

3. **Skills vs MCP is now a real architectural choice.** Anthropic Skills (October 2025) handle workflow knowledge; MCP handles live API calls ([claude.com/blog/skills](https://claude.com/blog/skills)). For institution-specific knowledge ("how we format change tickets", "which Confluence space to file the meeting notes in"), a Skill is lighter weight and doesn't need a server.

4. **MCP marketplaces / registries are consolidating.** The official `registry.modelcontextprotocol.io` is operational ([registry.modelcontextprotocol.io](https://registry.modelcontextprotocol.io/)); PulseMCP, mcp.so, mcpservers.org are the major community indexes. The official registry will become authoritative; institutions should publish any internally-built MCPs there if they're shareable, and consume from there with provenance checks.

5. **Enterprise MCP gateways are diverging from LLM gateways.** MintMCP, Webrix, and others are positioning as MCP-only gateways with SSO/RBAC/audit features ([mintmcp.com/blog](https://www.mintmcp.com/blog/mintmcp-vs-litellm-mcp-comparison)). Where LiteLLM is already the LLM gateway, doubling up on MCP via LiteLLM is a sensible default — but the pattern will likely separate over time (LLM gateway and MCP gateway as distinct systems).

### 7. Gateway-specific architectural guidance

For a university deploying MCP behind LiteLLM:

- **Front everything through LiteLLM's MCP gateway** rather than letting Claude Code instances connect directly to MCP servers. This gives one place to enforce per-Key/Team/Org permissions, one audit log, and one OAuth-token cache.
- **Use OAuth PKCE for user-attributed work** (Claude Code calling GitHub, M365, Atlassian as the user). Use client_credentials only for service-account workflows where attribution is to a system, not a person ([docs.litellm.ai/docs/mcp_oauth](https://docs.litellm.ai/docs/mcp_oauth)).
- **Pair MCP scopes with LiteLLM Teams** that map to colleges or departments. A medical school Team gets PubMed, an engineering Team gets GitHub Enterprise, the AI hub Team gets the union of internal MCPs.
- **Audit log every MCP tool call** through the existing LiteLLM observability hooks. Include user identity, MCP server, tool name, arguments, latency, success/failure. This becomes the FERPA/HIPAA-defensible audit trail.
- **Apply per-user budgets** at the LiteLLM Key level. MCP tool calls don't cost LLM tokens but they cost upstream API quota; budget-bound this with the `x-litellm-end-user-id` header pattern.
- **Treat all MCP outputs as untrusted data.** arXiv, PubMed, GitHub, Confluence, even internal Slack — all of them can carry injected instructions in returned content. The MCP server's own README for arxiv-mcp-server explicitly flags this; it should be the rule, not the exception ([github.com/blazickjp/arxiv-mcp-server](https://github.com/blazickjp/arxiv-mcp-server)).
- **Standardize on FastMCP + httpx + Docker SSE** for any in-house server. This gives a uniform deploy/operate story and makes the build → publish to LiteLLM workflow repeatable.

## Confidence Assessment

| Area | Confidence | Why |
|---|---|---|
| LiteLLM MCP gateway capability | **High** | Multiple primary sources fetched, including official docs and DeepWiki implementation notes |
| Anthropic official MCP server list (current) | **Medium-High** | Verified directly from `modelcontextprotocol/servers` README and the org's repo list (single publisher, but authoritative) |
| GitHub / Atlassian / Microsoft 365 / Snowflake / Grafana / Elastic / Datadog as buyable MCPs | **High** | Each verified at the vendor's own page or repo |
| Cloudflare remote MCP pattern | **Medium-High** | Verified at Cloudflare's launch blog post (single source for date and components) |
| Anthropic Skills vs MCP boundary | **Medium-High** | Verified launch post and capability description; specific Skill availability for individual partners (Box, Notion, Canva) inferred from announcement |
| arXiv MCP, PubMed MCP, Canvas LMS MCP recommendations | **High** | Each repo verified directly with stars, license, and maturity signals |
| OpenAlex / Crossref / ORCID / Alma / FOLIO / DSpace as build-targets | **Medium** | Confirmed *no* production MCP exists for most by negative search; the recommendation to build is sound but the "no MCP exists" claim could be stale within months |
| Slack (Zencoder), ServiceNow, Box, Zoom MCP availability | **Medium** | Slack-Zencoder verified directly. ServiceNow / Box / Zoom: confirmed no clearly-official MCP via search but may exist on PulseMCP / mcp.so under names not surfaced |
| Splunk / CrowdStrike / Tableau / Power BI MCP availability | **Low-Medium** | Search did not surface official servers; not exhaustively verified. Treat the "build" recommendation as provisional |
| Kubernetes / OpenShift MCP recommendation | **Low** | Many community servers exist but none verified as production-ready in this research; verify before adopting |

## Open Questions

1. **Does the institution use Atlassian, ServiceNow, Box, Zoom enterprise-wide, or only by department?** Determines P0/P2 ranking on those rows.
2. **What's the institutional stance on running an OAuth-issuing service?** The Microsoft 365 connector requires central IT to admin-consent the Anthropic Enterprise app in Entra ID; same for any per-user OAuth MCP.
3. **Does the LiteLLM gateway team want to enable the MCP gateway feature on the existing instance, or stand up a parallel deployment?** Affects audit-log architecture.
4. **What's the FERPA review path for a Canvas MCP?** A read-only, instructor-scoped Canvas MCP is potentially deployable, but the sentence "FERPA review for AI tools" is doing a lot of work.
5. **Is there appetite at the university library to build OpenAlex / Crossref / ORCID / DSpace MCPs?** These are the highest-leverage build targets for the higher-ed angle.
6. **What do the health-sciences schools want?** Medical schools have different research/clinical needs (PubMed, ClinicalTrials.gov, NIH RePORTER). Worth a separate scoping conversation.
7. **Is publishing the institution's MCP catalog publicly desirable?** Stanford and CMU have done this with their AI gateway pages; an MCP catalog is a recruiting and faculty-relations signal.
8. **Does central IT plan to track the MCP version (`2025-11-25` etc.) for compliance/upgrade planning?** LiteLLM bumps protocol versions; SEP-986 naming compliance is enforced.

## Sources

### LiteLLM (gateway)
- [docs.litellm.ai/docs/mcp](https://docs.litellm.ai/docs/mcp) — MCP Gateway overview, transports, access control, version 2025-11-25
- [docs.litellm.ai/docs/mcp_oauth](https://docs.litellm.ai/docs/mcp_oauth) — Interactive PKCE and M2M client_credentials OAuth flows
- [DeepWiki: BerriAI/litellm 8.10 MCP Server Gateway](https://deepwiki.com/BerriAI/litellm/8.10-mcp-server-gateway) — Implementation details: tool prefixing, three-layer access control, observability
- [docs.litellm.ai/docs/simple_proxy](https://docs.litellm.ai/docs/simple_proxy) — Proxy navigation showing Agent & MCP Gateway category
- [MintMCP vs LiteLLM MCP comparison](https://www.mintmcp.com/blog/mintmcp-vs-litellm-mcp-comparison) — When to use LiteLLM as MCP gateway

### Anthropic / MCP ecosystem
- [github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) — Reference servers (current 7, archived 12)
- [github.com/orgs/modelcontextprotocol/repositories](https://github.com/orgs/modelcontextprotocol/repositories) — 41 repos, SDKs, registry, inspector
- [registry.modelcontextprotocol.io](https://registry.modelcontextprotocol.io/) — Official MCP Registry
- [claude.com/blog/skills](https://claude.com/blog/skills) — Anthropic Skills launch (October 16, 2025)
- [claude.com/blog/connectors-directory](https://claude.com/blog/connectors-directory) — Connectors Directory launch (July 14, 2025)
- [support.claude.com](https://support.claude.com/en/articles/12542951-enable-and-use-the-microsoft-365-connector) — M365 connector enable guide
- [support.claude.com — M365 Security Guide](https://support.claude.com/en/articles/12684923-microsoft-365-connector-security-guide)
- [pulsemcp.com](https://www.pulsemcp.com/) — Community MCP index, MCP Steering Committee member maintained

### University AI gateway public references
- [Stanford AI API Gateway](https://uit.stanford.edu/service/ai-api-gateway) — LiteLLM-backed institutional gateway
- [Carnegie Mellon AI Gateway](https://www.cmu.edu/computing/services/ai/tools/ai-gateway/index.html) — LiteLLM-backed institutional gateway

### Vendor MCP servers (verified via direct fetch)
- [github.com/github/github-mcp-server](https://github.com/github/github-mcp-server) — GitHub official, GHES support, OAuth, 60+ tools, MIT
- [github.blog: GitHub MCP public preview](https://github.blog/changelog/2025-04-04-github-mcp-server-public-preview/) — Announcement
- [github.blog: GitHub MCP Jan 2026 update](https://github.blog/changelog/2026-01-28-github-mcp-server-new-projects-tools-oauth-scope-filtering-and-new-features/) — Enterprise Server compatibility
- [docs.github.com — GitHub MCP for Enterprise](https://docs.github.com/en/copilot/how-tos/provide-context/use-mcp-in-your-ide/enterprise-configuration)
- [atlassian.com/platform/remote-mcp-server](https://www.atlassian.com/platform/remote-mcp-server) — Atlassian Rovo MCP, GA, OAuth, rate limits
- [office365itpros.com](https://office365itpros.com/2026/04/08/microsoft-365-connector-for-claude/) — M365 MCP enterprise app IDs and architecture
- [github.com/eesb99/msgraph-mcp](https://github.com/eesb99/msgraph-mcp) — Read-only Graph MCP for Claude Code
- [github.com/Snowflake-Labs/mcp](https://github.com/Snowflake-Labs/mcp) — Snowflake official, Cortex tools, Apache 2.0
- [github.com/grafana/mcp-grafana](https://github.com/grafana/mcp-grafana) — Grafana official, Apache 2.0, 2.9k stars
- [docs.datadoghq.com/bits_ai/mcp_server/](https://docs.datadoghq.com/bits_ai/mcp_server/) — Datadog official, in active development
- [github.com/elastic/mcp-server-elasticsearch](https://github.com/elastic/mcp-server-elasticsearch) — Elastic official, deprecated in favor of Agent Builder
- [github.com/googleapis/genai-toolbox](https://github.com/googleapis/genai-toolbox) — Google MCP Toolbox for Databases (BigQuery+)
- [github.com/zencoderai/slack-mcp-server](https://github.com/zencoderai/slack-mcp-server) — Slack MCP (former Anthropic reference)

### Higher-ed / scholarly MCP servers
- [github.com/vishalsachdev/canvas-mcp](https://github.com/vishalsachdev/canvas-mcp) — Canvas LMS MCP, 88 tools, MIT, hosted preview
- [github.com/r-huijts/canvas-mcp](https://github.com/r-huijts/canvas-mcp) — Alternative Canvas MCP
- [github.com/DMontgomery40/mcp-canvas-lms](https://github.com/DMontgomery40/mcp-canvas-lms) — Alternative Canvas MCP, 54 tools
- [atomicjolt.com](https://www.atomicjolt.com/post/automatically-generate-working-code-for-canvas-api-development) — Atomic Canvas API Docs MCP
- [github.com/blazickjp/arxiv-mcp-server](https://github.com/blazickjp/arxiv-mcp-server) — arXiv MCP, Apache 2.0, 2.6k stars
- [github.com/JackKuo666/PubMed-MCP-Server](https://github.com/JackKuo666/PubMed-MCP-Server) — PubMed MCP, MIT, 108 stars
- [developers.openalex.org](https://developers.openalex.org/) — OpenAlex API (build target)

### Hosting / industry context
- [blog.cloudflare.com — Remote MCP servers](https://blog.cloudflare.com/remote-model-context-protocol-servers-mcp/) — Cloudflare workers-oauth-provider, McpAgent, mcp-remote, AI Playground (March 25, 2025)
- [Securing LiteLLM MCP — cloudurable](https://cloudurable.com/blog/securing-litellm-s-mcp-integration-one-gateway-mu/) — Operational walkthrough
- [LiteLLM + MCP Integration — Infralovers](https://www.infralovers.com/blog/2025-10-23-litellm-mcp-integration/) — Architecture and governance considerations
- [LiteLLM and MCP — Rick Hightower / Medium](https://medium.com/@richardhightower/litellm-and-mcp-one-gateway-to-rule-all-ai-models-378a38ce7746) — Pattern explanation

## Update History

- **2026-04-28 13:51 EDT** — Initial publication. Research agent run, ~25 web searches and direct URL fetches, focused on LiteLLM MCP gateway capabilities, Anthropic MCP ecosystem state, and verified MCP servers across six audience categories. Confidence levels assigned per-section.

## How This Report Was Generated

Research agent (Claude Opus 4.7, 1M context). Search tooling: SearXNG metasearch and direct WebFetch against vendor docs and GitHub repos. Search-engine results were noisy on several queries; compensated by direct URL fetches against known-good vendor URLs and GitHub repos. Each named MCP server was verified by fetching its GitHub repo or vendor doc. Build-vs-buy recommendations distinguish "verified production MCP exists" from "no production MCP found in this research" — the latter could be stale within months given the pace of the ecosystem.
