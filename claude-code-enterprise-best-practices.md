---
title: "Claude Code Enterprise Best Practices: Pre-Contract University Deployment"
date: 2026-04-24
updated: 2026-04-24 11:37 AM ET
summary: "Actionable strategy for deploying Claude Code at Cornell during the pre-contract interim, covering Anthropic's enterprise features, LLM gateway integration with the Cornell AI API Gateway, managed settings, cost governance, and what is known (and not known) about the Internet2 contract."
---

## Current Status

- Cornell already operates a self-hosted LiteLLM-based AI API Gateway at `api.ai.it.cornell.edu`, providing authenticated, metered access to models across Azure, AWS, and GCP. [source: [Cornell AI API Gateway Confluence](https://confluence.cornell.edu/spaces/citai/pages/541787315/AI+API+Gateway)]
- Claude Code natively supports LiteLLM gateways via the `ANTHROPIC_BASE_URL` environment variable, meaning Cornell developers can use Claude Code through the existing gateway today. [source: [Claude Code LLM Gateway docs](https://code.claude.com/docs/en/llm-gateway)]
- Anthropic offers three relevant plan tiers for organizations: Team ($150/seat/mo Premium), Enterprise (contact sales), and Anthropic Console (pay-as-you-go API). All three include Claude Code access. [source: [Claude Code third-party integrations](https://code.claude.com/docs/en/third-party-integrations)]
- Internet2 is reported to be negotiating higher education AI vendor contracts, but no public documentation of a completed Anthropic-specific agreement was found during this research. This is flagged as a gap.
- Claude Code enterprise features (managed settings, server-managed policies, OpenTelemetry monitoring, managed permissions) are mature and available today on Team and Enterprise plans. [source: [Claude Code server-managed settings](https://code.claude.com/docs/en/server-managed-settings)]
- The Cornell AI Gateway is currently in "preview" status with a $10/month free tier per team and KFS-based billing. [source: [Cornell AI API Gateway Confluence](https://confluence.cornell.edu/spaces/citai/pages/541787315/AI+API+Gateway)]

---

## Table of Contents

1. [Claude Code Enterprise Features and Licensing](#1-claude-code-enterprise-features-and-licensing)
2. [Internet2 Contract Status](#2-internet2-contract-status)
3. [University Deployment Patterns](#3-university-deployment-patterns)
4. [Pre-Contract Deployment Strategy for Cornell](#4-pre-contract-deployment-strategy-for-cornell)
5. [Technical Configuration Best Practices](#5-technical-configuration-best-practices)
6. [Cost Management and Governance](#6-cost-management-and-governance)
7. [Risk Mitigation](#7-risk-mitigation)

---

## Findings

### 1. Claude Code Enterprise Features and Licensing

Anthropic offers Claude Code access through several organizational plans. The choice depends on whether you want subscription-based access or API-based pay-as-you-go.

**Plan Comparison for Organizations:**

| Feature | Claude for Teams | Claude for Enterprise | Anthropic Console (API) |
|---|---|---|---|
| Billing | $150/seat Premium, PAYG available | Contact sales | Pay-as-you-go |
| Authentication | Claude.ai SSO or email | Claude.ai SSO + domain capture | API key |
| Claude Code included | Yes | Yes | Yes |
| Claude on web included | Yes | Yes | No |
| SSO | Standard | Advanced + domain capture | Via Console SSO |
| Managed settings | Server-managed via admin console | Server-managed via admin console | Not available |
| SCIM provisioning | No | Yes | No |
| Role-based permissions | Basic | Full | Claude Code or Developer roles |
| Audit logs | Basic | Full + compliance API | Usage dashboard |
| IP allowlisting | No | Yes | No |
| Custom data retention | No | Yes | No |

[source: [Claude Code third-party integrations](https://code.claude.com/docs/en/third-party-integrations), [Claude Code authentication](https://code.claude.com/docs/en/authentication)]

**Key Enterprise Features:**

**Server-Managed Settings (Teams and Enterprise):** Administrators can centrally configure Claude Code through a web interface on Claude.ai. Settings are delivered to clients when users authenticate. This includes permission deny lists, hook configurations, environment variables, and managed-only settings that cannot be overridden by users. [source: [Claude Code server-managed settings](https://code.claude.com/docs/en/server-managed-settings)]

**Managed Permissions:** Organizations can deploy permissions that users cannot override, including `allowManagedPermissionRulesOnly` (prevents user-defined permission rules), `allowManagedHooksOnly` (restricts hooks to managed-only), `disableBypassPermissionsMode`, and `disableAutoMode`. [source: [Claude Code permissions](https://code.claude.com/docs/en/permissions)]

**OpenTelemetry Monitoring:** Claude Code exports metrics, events, and traces via OpenTelemetry. Administrators can set telemetry configuration via managed settings that cannot be overridden. Metrics include token usage, cost tracking, tool activity, and session-level tracking. [source: [Claude Code monitoring](https://code.claude.com/docs/en/monitoring-usage)]

**Organization-Wide CLAUDE.md:** A managed policy CLAUDE.md can be deployed to all machines via MDM or configuration management. Locations: `/Library/Application Support/ClaudeCode/CLAUDE.md` (macOS), `/etc/claude-code/CLAUDE.md` (Linux/WSL), `C:\Program Files\ClaudeCode\CLAUDE.md` (Windows). These cannot be excluded by users. [source: [Claude Code memory](https://code.claude.com/docs/en/memory)]

**Cloud Provider Backends:** Claude Code can run on Anthropic directly, Amazon Bedrock, Google Vertex AI, or Microsoft Foundry. Organizations already invested in a cloud provider can use their existing contracts and billing. [source: [Claude Code third-party integrations](https://code.claude.com/docs/en/third-party-integrations)]

**Self-Serve Enterprise:** Since February 12, 2026, Enterprise plans can be purchased self-serve, removing the sales-only gate. [source: previous research report at `research-reports/claude-code-enterprise-security-features.md`]

---

### 2. Internet2 Contract Status

**What is known:**

- Internet2 is the research and education networking consortium that brokers NET+ cloud contracts for its member institutions (Cornell is a member).
- Internet2 has historically brokered cloud contracts with major vendors (AWS, Azure, GCP, Box, Zoom) that member universities can leverage for reduced pricing and standardized terms.
- There are reports that Internet2 is working on AI vendor contracts for higher education, but no public announcement of a completed Anthropic-specific agreement was found during this research.
- Cornell's AI API Gateway documentation mentions that CIT is "working on contracts with other providers as well, prioritized by our customer's needs," which could include direct Anthropic contracts or Internet2-brokered agreements. [source: [Cornell AI API Gateway Confluence](https://confluence.cornell.edu/spaces/citai/pages/541787315/AI+API+Gateway)]

**What is not known:**

- Whether an Internet2 + Anthropic contract exists, is in negotiation, or is merely anticipated.
- The terms, pricing, or timeline of any such agreement.
- Whether Cornell's procurement office has a separate direct negotiation track with Anthropic.
- How other R1 universities are handling Anthropic contracts (no public case studies were found).

**Confidence: Low.** I was unable to find any public documentation of an Internet2 + Anthropic contract. The Internet2 website did not surface AI-specific contract pages at any URL pattern tested. This information may exist behind member-only portals or may simply not be publicly available yet.

**Recommendation:** Pete should contact Cornell's CIT AI Program team (`ai-support@cornell.edu`) to ask directly about the status of any Internet2 or direct Anthropic contract negotiations. The procurement team will have visibility that is not publicly available.

---

### 3. University Deployment Patterns

While specific university Claude Code deployment case studies were not publicly available, the research reveals a clear pattern for how universities are approaching AI developer tools:

**Pattern 1: Centralized API Gateway (Cornell's current approach)**

Cornell's model is representative of the emerging best practice: a self-hosted LiteLLM gateway that provides a single entry point for all AI model access. This gives the institution:
- Centralized cost tracking via KFS account billing
- Data privacy guarantees (no training on institutional data)
- Model-agnostic access (switch providers without changing client config)
- Per-team budgets and rate limits
- A $10/month free tier for exploration

[source: [Cornell AI API Gateway Confluence](https://confluence.cornell.edu/spaces/citai/pages/541787315/AI+API+Gateway)]

**Pattern 2: Cloud Provider Contracts**

Universities with existing AWS/Azure/GCP contracts can access Claude models through Bedrock, Foundry, or Vertex AI using existing cloud agreements. This avoids a separate Anthropic contract entirely. Claude Code natively supports all three backends. [source: [Claude Code third-party integrations](https://code.claude.com/docs/en/third-party-integrations)]

**Pattern 3: Team/Enterprise Plans for Specific Units**

Individual departments or labs can purchase Claude Team plans ($150/seat) self-serve. This provides Claude Code access with centralized billing at the department level while the institution works on broader contracts.

**Data Classification:** Cornell's AI Gateway permits Moderate and Low risk data. For FERPA data, teams must notify the AI program. High risk data requires direct coordination with the IT Security Office. [source: [Cornell AI API Gateway Confluence](https://confluence.cornell.edu/spaces/citai/pages/541787315/AI+API+Gateway)]

---

### 4. Pre-Contract Deployment Strategy for Cornell

Given that Cornell already has a functioning LiteLLM gateway with Anthropic model access, the interim strategy is straightforward. There are two parallel paths, depending on what level of access is needed.

**Path A: API Gateway Route (Available Now)**

This is what Pete is already doing. Claude Code connects to the Cornell AI Gateway via `ANTHROPIC_BASE_URL`. This route:
- Works today with no contract changes
- Bills through existing KFS accounts
- Uses Cornell's cloud provider contracts (Azure, AWS, GCP) for underlying model access
- Provides data privacy guarantees through existing cloud vendor agreements
- Does not require individual Anthropic accounts

Configuration (already documented in Pete's setup):
```bash
export ANTHROPIC_BASE_URL="https://api.ai.it.cornell.edu/"
export ANTHROPIC_AUTH_TOKEN="sk-YOURKEYHERE"
export CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1
```
[source: Cornell CLAUDE-SETUP.md at `Cornell/Documentation/CLAUDE-SETUP.md`]

**Limitations of Path A:**
- No server-managed settings (requires Teams/Enterprise subscription)
- No web-based Claude access
- No SSO integration beyond the gateway's own auth
- The gateway service is in "preview" status
- Limited to models available through cloud provider contracts
- Per-request charge of $0.002 adds up for Claude Code's high request volume
- Model access restricted to what Cornell's LiteLLM instance exposes (`all_cornell_models`)

**Path B: Claude for Teams (Small-Scale Pilot)**

For teams that need full Claude Code enterprise features before a broad contract:
1. Purchase Claude for Teams seats ($150/seat/month) for AI Hub developers
2. Use server-managed settings to enforce organizational policies
3. Track usage centrally via OpenTelemetry
4. Evaluate whether a full Enterprise plan is warranted

This provides an evaluation baseline for an eventual enterprise rollout without waiting for Internet2 or procurement.

**Path C: Anthropic Console (API-Only)**

For developer-focused teams that only need Claude Code (not web Claude):
1. Create an Anthropic Console organization
2. Invite users with the "Claude Code" role (restricts to Claude Code API keys only)
3. Pay-as-you-go billing
4. Evaluate costs over a defined pilot period

[source: [Claude Code authentication](https://code.claude.com/docs/en/authentication)]

**Recommended Approach:** Continue with Path A (gateway) for daily work. Pursue Path B (Teams pilot) for a small group of AI Hub developers to evaluate enterprise features. Use the pilot data to justify the enterprise contract conversation, whether through Internet2, direct procurement, or a cloud provider backend.

---

### 5. Technical Configuration Best Practices

**For Path A (Gateway Route):**

1. **Separate config directories.** Keep work and personal configs isolated using `CLAUDE_CONFIG_DIR`. Pete already does this with `~/.claude-work`. [source: `Cornell/Documentation/CLAUDE-SETUP.md`]

2. **Pin model versions.** When using cloud provider backends, set `ANTHROPIC_DEFAULT_OPUS_MODEL`, `ANTHROPIC_DEFAULT_SONNET_MODEL`, and `ANTHROPIC_DEFAULT_HAIKU_MODEL` to specific versions. Without pinning, model aliases resolve to the latest version, which may not be enabled in the cloud account when Anthropic releases an update. [source: [Claude Code third-party integrations](https://code.claude.com/docs/en/third-party-integrations)]

3. **Disable experimental betas.** When routing through a gateway using the Anthropic Messages format with Bedrock or Vertex backends, set `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1`. [source: [Claude Code LLM gateway](https://code.claude.com/docs/en/llm-gateway)]

4. **Use dynamic API key helpers.** For rotating keys or per-user auth, use the `apiKeyHelper` setting to call a script that fetches keys from a vault or generates JWT tokens. [source: [Claude Code LLM gateway](https://code.claude.com/docs/en/llm-gateway)]

**For Path B (Teams/Enterprise Route):**

5. **Deploy organization-wide CLAUDE.md.** Place a managed CLAUDE.md at the OS-level path to set Cornell-specific coding standards, data handling policies, and security requirements. This file cannot be overridden by users. [source: [Claude Code memory](https://code.claude.com/docs/en/memory)]

6. **Configure managed permissions.** Use server-managed settings to enforce:
   - Deny rules for commands that access sensitive data
   - Deny rules for `.env` file access
   - Disable `bypassPermissions` mode
   - Restrict permission rules to managed-only (`allowManagedPermissionRulesOnly`)
   [source: [Claude Code permissions](https://code.claude.com/docs/en/permissions)]

7. **Enable OpenTelemetry.** Configure telemetry via managed settings to track usage, costs, and tool activity across all users. This data feeds budget planning and contract justification. [source: [Claude Code monitoring](https://code.claude.com/docs/en/monitoring-usage)]

   ```json
   {
     "env": {
       "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
       "OTEL_METRICS_EXPORTER": "otlp",
       "OTEL_LOGS_EXPORTER": "otlp",
       "OTEL_EXPORTER_OTLP_ENDPOINT": "http://collector.example.com:4317"
     }
   }
   ```

8. **Invest in shared CLAUDE.md files.** Check project-level CLAUDE.md files into repositories so all team members benefit. This is Anthropic's top recommendation for enterprise adoption. [source: [Claude Code third-party integrations](https://code.claude.com/docs/en/third-party-integrations)]

9. **Create a one-click install.** Anthropic explicitly recommends that organizations create a simplified installation process to drive adoption. A setup script that configures the gateway URL, API key, and managed settings would reduce friction. [source: [Claude Code third-party integrations](https://code.claude.com/docs/en/third-party-integrations)]

10. **Start with guided usage.** Anthropic recommends beginning with codebase Q&A and small bug fixes before letting users run Claude Code agentically. This builds competence and trust. [source: [Claude Code third-party integrations](https://code.claude.com/docs/en/third-party-integrations)]

---

### 6. Cost Management and Governance

**Gateway-Based Cost Tracking (Path A):**

Cornell's LiteLLM gateway already provides:
- Per-team usage tracking (input/output tokens)
- KFS account billing
- Configurable per-key and per-team budgets
- Rate limiting
- $0.002 per-request surcharge (as of January 2026)
- $10/month free tier per team

[source: [Cornell AI API Gateway Confluence](https://confluence.cornell.edu/spaces/citai/pages/541787315/AI+API+Gateway)]

**Claude Code-Specific Cost Considerations:**

Claude Code is a high-volume consumer of API calls. Each interactive session involves many requests as Claude reads files, plans, executes, and verifies. Key factors:

- **Prompt caching** is enabled by default on all providers and significantly reduces costs for repeated context. [source: [Claude Code third-party integrations](https://code.claude.com/docs/en/third-party-integrations)]
- **Per-request charges** ($0.002/request through the Cornell gateway) add meaningful overhead given Claude Code's request volume. Monitor this closely.
- **OpenTelemetry metrics** can export session-level cost data for tracking. Relevant metrics include `input_tokens`, `output_tokens`, `cache_read_tokens`, and `cache_creation_tokens`. [source: [Claude Code monitoring](https://code.claude.com/docs/en/monitoring-usage)]
- **Budget alerts** should be set per-team and per-key in the gateway to prevent surprise costs during pilot.

**Cost Estimation:**

Without public pricing for the models routed through Cornell's gateway, exact costs are difficult to estimate. A reasonable approach for a pilot:
- Set a team-level monthly budget in the gateway (e.g., $500/month for 5 developers)
- Monitor actual usage for 30 days
- Use the data to project annual costs for enterprise contract negotiation

---

### 7. Risk Mitigation

**Risk: Gateway Preview Status**

The Cornell AI Gateway is in preview. The service documentation states "anything about this service may change or be removed while it is in preview." [source: [Cornell AI API Gateway Confluence](https://confluence.cornell.edu/spaces/citai/pages/541787315/AI+API+Gateway)]

Mitigation: Claude Code supports multiple backends. If the gateway changes, developers can switch to direct Anthropic API access, Bedrock, Vertex, or Foundry with configuration changes only (environment variables). No code changes required.

**Risk: No Enterprise Contract**

Without a formal Anthropic contract, Cornell lacks negotiated terms for data processing, SLAs, and liability.

Mitigation: The gateway routes through existing cloud vendor contracts (Azure, AWS, GCP) that Cornell already has in place. Data privacy is governed by those existing agreements, not a direct Anthropic relationship. Cornell's gateway documentation confirms: "your data remains private to you" and "never made available to the model providers for things like model training." [source: [Cornell AI API Gateway Confluence](https://confluence.cornell.edu/spaces/citai/pages/541787315/AI+API+Gateway)]

**Risk: Data Classification**

Claude Code may encounter sensitive data in codebases. Cornell's gateway permits Moderate and Low risk data. FERPA data requires notification. High risk data requires ITSO coordination.

Mitigation: Use managed permission deny rules to block access to `.env` files, secrets directories, and other sensitive paths. Deploy an organization-wide CLAUDE.md with data handling guidelines. Enable sandboxing for additional filesystem isolation. [source: [Claude Code security](https://code.claude.com/docs/en/security), [Claude Code permissions](https://code.claude.com/docs/en/permissions)]

**Risk: Shadow IT**

Individual developers may use personal Claude accounts for work, bypassing institutional controls.

Mitigation: Make the institutional path easier than the personal path (one-click install). Track adoption through gateway usage metrics. Communicate acceptable use policies through the managed CLAUDE.md.

**Risk: Server-Managed Settings Bypass**

When using `ANTHROPIC_BASE_URL` (the gateway route), server-managed settings from Claude.ai are not delivered. This is a documented limitation. [source: [Claude Code server-managed settings](https://code.claude.com/docs/en/server-managed-settings)]

Mitigation: For the gateway route, use endpoint-managed settings deployed via MDM, managed settings files, or the OS-level managed CLAUDE.md path. Alternatively, use a hybrid approach: Claude for Teams subscriptions for enterprise features, with the gateway as a backend through environment variables.

---

## Confidence Assessment

| Finding | Confidence | Basis |
|---|---|---|
| Claude Code enterprise features (managed settings, OTel, permissions) | **High** | Official Anthropic documentation, directly verified |
| Cornell AI Gateway configuration and capabilities | **High** | Internal documentation, Pete's working config |
| LiteLLM gateway compatibility with Claude Code | **High** | Official documentation with detailed config examples |
| Plan pricing (Teams $150/seat) | **High** | Official documentation |
| Cloud provider backend support (Bedrock/Vertex/Foundry) | **High** | Official documentation with detailed guides |
| Organization-wide CLAUDE.md deployment paths | **High** | Official documentation with specific file paths |
| Internet2 contract status | **Low** | No public evidence found; may exist behind member portals |
| University deployment case studies | **Low** | No public case studies of Claude Code at universities found |
| Anthropic education-specific programs | **Low** | Education page exists at claude.com/solutions/education but content could not be retrieved |
| Gateway per-request cost impact on Claude Code | **Medium** | Known rate ($0.002/request) but Claude Code request volume varies significantly |

---

## Open Questions

1. **Internet2 contract timeline.** What is the actual status of any Internet2 + Anthropic negotiation? Contact `ai-support@cornell.edu` or Cornell procurement to get insider status.

2. **Anthropic education pricing.** Anthropic has an education solutions page (`claude.com/solutions/education`) but the content was inaccessible during research. Does Anthropic offer education-specific pricing or terms?

3. **Cloud provider model availability.** Which specific Claude models are available through Cornell's existing Azure/AWS/GCP contracts? Are Opus, Sonnet, and Haiku all available?

4. **FERPA compliance.** Has Anthropic completed a FERPA compliance review? Is a Data Processing Addendum (DPA) available for educational institutions?

5. **Per-request cost at scale.** At $0.002/request through the gateway, what is the realistic monthly cost per developer for Claude Code usage? A 30-day pilot would answer this.

6. **Gateway general availability timeline.** When does CIT plan to move the AI Gateway from preview to GA? This affects long-term planning.

7. **Server-managed settings via gateway.** Is it possible to get server-managed settings while routing through the Cornell gateway, perhaps via a hybrid Team plan + gateway configuration?

---

## Sources

**Anthropic Official Documentation:**
- [Claude Code overview](https://code.claude.com/docs/en/overview) (installation, surfaces, capabilities)
- [Claude Code third-party integrations / enterprise deployment](https://code.claude.com/docs/en/third-party-integrations) (plan comparison, deployment options, best practices)
- [Claude Code LLM gateway configuration](https://code.claude.com/docs/en/llm-gateway) (LiteLLM setup, authentication, endpoint config)
- [Claude Code authentication](https://code.claude.com/docs/en/authentication) (login methods, Teams/Enterprise setup, Console setup, API key management)
- [Claude Code security](https://code.claude.com/docs/en/security) (permission architecture, prompt injection protections, team security)
- [Claude Code permissions](https://code.claude.com/docs/en/permissions) (managed permissions, managed-only settings, permission modes)
- [Claude Code server-managed settings](https://code.claude.com/docs/en/server-managed-settings) (centralized config, delivery, enforcement)
- [Claude Code monitoring](https://code.claude.com/docs/en/monitoring-usage) (OpenTelemetry metrics, events, traces, admin config)
- [Claude Code memory / CLAUDE.md](https://code.claude.com/docs/en/memory) (org-wide deployment, rules, auto memory)

**Cornell Internal Documentation:**
- [Cornell AI API Gateway](https://confluence.cornell.edu/spaces/citai/pages/541787315/AI+API+Gateway) (service details, data security, billing, access)
- Cornell CLAUDE-SETUP.md at `Cornell/Documentation/CLAUDE-SETUP.md` (local config reference)
- Cornell AI-API-GATEWAY.md at `Cornell/Documentation/AI-API-GATEWAY.md` (gateway details)

**Previous Research Reports:**
- `research-reports/claude-code-enterprise-security-features.md` (2026-04-06, security features deep dive)

---

## Update History

| Date | Change |
|---|---|
| 2026-04-24 | Initial report |

---

## How This Report Was Generated

Research agent using Claude Code (Opus 4.6 via Cornell AI Gateway). Primary sources were Anthropic's official Claude Code documentation at `code.claude.com/docs` and Cornell's internal AI API Gateway documentation. Web searches for Internet2 contract information and university case studies were attempted but yielded no results due to API access restrictions and the likely non-public nature of contract negotiations. The report explicitly flags gaps where evidence was insufficient, particularly around Internet2 contract status and university deployment case studies.
