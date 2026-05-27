---
title: "Claude Desktop app (3P API → AWS Bedrock via IdC SSO): the Code tab spins"
audience: "Pete (Cornell Mac, Jamf-managed, Cornell AI Gateway in front of Bedrock)"
report_type: "diagnostic + product-limitation analysis"
date: 2026-05-27
updated: 2026-05-27 5:54 PM ET
timezone: America/New_York
generated_by: Research agent (Claude Code)
summary: "Claude Desktop's Code tab is not designed to run against Bedrock. Anthropic's own docs say the Desktop app's Code tab defaults to api.anthropic.com and that 'For Bedrock or Foundry, use the CLI.' The spinning Code tab is consistent with three known, documented failure modes that share a root cause: the Code tab's auth path is independent of, and incompatible with, the Chat tab's 3P-gateway / Bedrock configuration. Follow-up section ([Section 8](#8-adding-skills-and-web-search-on-bedrock-the-cli-recipe)) covers how to add Agent Skills and web search to the CLI-on-Bedrock fallback: custom skills work natively, the marketplace and Anthropic's server-side web_search are blocked, and the recommended bridge is Pete's existing searxng MCP."
---

# Claude Desktop app (3P API → AWS Bedrock via IdC SSO): the Code tab spins

## TL;DR

Pete's spinning "Code tab" inside Claude Desktop is not a transient bug, it is the documented product limitation. **Anthropic's own Desktop docs say the Code tab connects to Anthropic's API by default and that "For Bedrock or Foundry, use the CLI"** ([source](https://code.claude.com/docs/en/desktop)). The Desktop app's enterprise gateway mode (the configuration Pete is using behind the Cornell AI Gateway) is real and works for the Cowork tab, but the Code tab's auth path is independent of the chat/gateway path and, per multiple confirmed reports including one from Pete himself, breaks in this configuration in three different documented ways:

1. **The Code tab does not honor `CLAUDE_CODE_USE_BEDROCK=1` from `~/.claude/settings.json` when a Desktop OAuth token is present in `config.json`** ([#35070](https://github.com/anthropics/claude-code/issues/35070), closed as not planned). The OAuth token silently wins, requests go to `api.anthropic.com` with invalid short-form model IDs, and the user sees `API Error (claude-sonnet-4-6): 400 The provided model identifier is invalid.` — or a silent hang depending on Desktop version.

2. **The Code tab independently scans `~/.aws/config`, detects the corporate SSO endpoint, and forces an enterprise AWS SSO browser flow** that ignores the Desktop app session ([#61851](https://github.com/anthropics/claude-code/issues/61851), open as of 2026-05-23, labels `api:bedrock`, `area:auth`, `area:desktop`). If the SSO flow doesn't complete cleanly (TLS-inspection proxy, blocked callback, expired session), the Code tab just sits at a spinner with no error.

3. **The Code tab in Desktop 3P / gateway mode has documented feature-parity gaps with the Cowork tab.** The Cowork-on-3P configuration flow ("Developer → Configure third-party inference") works for Cowork, but model-picker, environment-variable, and policy settings that work in Cowork are confirmed to not propagate to the Code tab ([#52526](https://github.com/anthropics/claude-code/issues/52526)). Pete's own filed bug [#59407](https://github.com/anthropics/claude-code/issues/59407) ("Chat tab disappears when switching from Anthropic login to third-party API gateway") is a sibling of this class of bug: the Desktop app's tab-level features assume Anthropic-direct auth, and the team-mode-gated 3P path is wired tab-by-tab, not app-wide.

The "Chat tab disappears in gateway mode" issue Pete already filed and the spinning Code tab are the same class of problem: **Anthropic ships per-tab feature gating that's tied to authentication mode, and the docs are incomplete about which tabs work in which 3P configuration**. The fix is not a settings change. The fix is: **use Claude Code CLI on Bedrock for coding (`claude` from the integrated terminal), keep Claude Desktop in 3P mode for Cowork only, and stop expecting the Code tab to work against Bedrock.**

For the AWS-blessed alternative ("Claude Cowork on Amazon Bedrock") that Anthropic + AWS jointly support and document, see the [Recommended path forward](#5-recommended-path-forward-for-petes-setup) section.

## Current Status

As of 2026-05-27:

- **Anthropic's Desktop docs explicitly route Bedrock users to the CLI**, not to the Desktop Code tab ([source](https://code.claude.com/docs/en/desktop)). This is product-level, not a bug.
- The Desktop app's official third-party-provider support, even in enterprise managed-settings deployments, is **Vertex AI and LLM "gateway providers"** ([source](https://code.claude.com/docs/en/desktop)). Bedrock-direct is named separately as available only on the CLI.
- AWS + Anthropic jointly shipped **"Claude Cowork on Amazon Bedrock"** in spring 2026 ([AWS ML blog](https://aws.amazon.com/blogs/machine-learning/from-developer-desks-to-the-whole-organization-running-claude-cowork-in-amazon-bedrock/)). It explicitly states that **"Features that require Anthropic-hosted inference, including the Chat tab, Computer Use, and the Skills Marketplace, are not included because Claude Cowork routes model inference exclusively through Amazon Bedrock."** The Code tab is included under Cowork, but with caveats (see [Findings #4](#4-the-cowork-on-bedrock-product-and-its-gaps)).
- The Cornell AI Gateway scenario (a non-AWS-blessed LLM gateway fronting Bedrock) is the same general class as the AWS Cowork-on-Bedrock pattern, but Anthropic does not officially support gateway-mode for the Code tab with the same fidelity. The "Developer → Configure third-party inference" panel exists and supports `inferenceProvider: gateway` with a base URL and bearer token, but multiple bugs confirm Code-tab parity is incomplete ([#52526](https://github.com/anthropics/claude-code/issues/52526), [#35070](https://github.com/anthropics/claude-code/issues/35070), [#59407](https://github.com/anthropics/claude-code/issues/59407)).
- IAM Identity Center (`aws sso login`) credentials are usable with Bedrock via the standard AWS SDK credential chain, but only via the CLI and the AWS Cowork-on-Bedrock product; the Desktop Code tab does not have a documented IdC SSO path.
- The "spinning forever" symptom is consistent with all three failure modes in the TL;DR. Without the Console app log lines from a reproduction we can't pick one definitively, but the prescription is the same: the Code tab is not supported in this configuration.

## Table of Contents

- [TL;DR](#tldr)
- [Current Status](#current-status)
- [Findings](#findings)
  - [1. What Claude Desktop's Code tab actually is (and what authenticates it)](#1-what-claude-desktops-code-tab-actually-is-and-what-authenticates-it)
  - [2. Why the Code tab can't talk to Bedrock the way the CLI can: the wire-format gap](#2-why-the-code-tab-cant-talk-to-bedrock-the-way-the-cli-can-the-wire-format-gap)
  - [3. The four known failure modes that all present as "spinning Code tab"](#3-the-four-known-failure-modes-that-all-present-as-spinning-code-tab)
  - [4. The Cowork-on-Bedrock product and its gaps](#4-the-cowork-on-bedrock-product-and-its-gaps)
  - [5. Recommended path forward for Pete's setup](#5-recommended-path-forward-for-petes-setup)
  - [6. Diagnostic playbook](#6-diagnostic-playbook)
  - [7. Limitations of Claude Desktop in 3P mode (full inventory)](#7-limitations-of-claude-desktop-in-3p-mode-full-inventory)
  - [8. Adding Skills and web search on Bedrock (the CLI recipe)](#8-adding-skills-and-web-search-on-bedrock-the-cli-recipe)
- [Confidence Assessment](#confidence-assessment)
- [Open Questions](#open-questions)
- [Sources](#sources)
  - [Official Anthropic / AWS docs](#official-anthropic--aws-docs)
  - [Most-relevant GitHub issues (anthropics/claude-code)](#most-relevant-github-issues-anthropicsclaude-code)
  - [Community walkthroughs and operator guides](#community-walkthroughs-and-operator-guides)
  - [Skills, web search, and Bedrock parity (added 2026-05-27 5:38 PM ET)](#skills-web-search-and-bedrock-parity-added-2026-05-27-538-pm-et)
- [Update History](#update-history)
- [How This Report Was Generated](#how-this-report-was-generated)

## Findings

### 1. What Claude Desktop's Code tab actually is (and what authenticates it)

Claude Desktop is the standalone Mac/Windows app (`Claude.app` / `Claude.exe`) shipped by Anthropic. The app has three tabs: **Chat** for conversations, **Cowork** for "Dispatch and longer agentic work," and **Code** for software development ([source](https://code.claude.com/docs/en/desktop)). The Code tab is the in-app graphical wrapper around the same Claude Code engine that powers the CLI, with a pane-based UI for chat, diffs, preview, terminal, file editor, sessions sidebar, and PR monitoring.

The Code tab's auth model is distinct from the Chat tab's in two important ways:

1. **It does its own credential resolution.** Even though the Desktop app already holds the user's logged-in session for Chat, the Code tab independently consults `~/.aws/config`, the AWS SDK default credential provider chain, and Claude Code settings files like `~/.claude/settings.json`. Confirmed by [#61851](https://github.com/anthropics/claude-code/issues/61851): "The Chat tab and Code tab in the Claude desktop app use completely separate authentication flows. The Chat tab correctly uses my personal Gmail session. The Code tab ignores that session entirely and triggers a new auth flow that detects ~/.aws/config, finds the corporate AWS SSO endpoint, and forces enterprise authentication with no option to choose a different account or go back."

2. **The cached Desktop OAuth token in `config.json` wins over `~/.claude/settings.json` env vars.** Documented in [#35070](https://github.com/anthropics/claude-code/issues/35070): on Windows, `%APPDATA%\Roaming\Claude\config.json` (on macOS, the equivalent under `~/Library/Application Support/Claude/`) contains cached OAuth tokens that take precedence over `CLAUDE_CODE_USE_BEDROCK=1` in `~/.claude/settings.json`. Result: OAuth token silently overrides Bedrock configuration, forcing all requests to `https://api.anthropic.com` with whatever model ID the Code tab thinks is active, which on a Bedrock account produces `400 The provided model identifier is invalid.` That issue was closed as not planned; no fix shipped.

The architectural takeaway: the Code tab inside Desktop is a separate auth boundary from the rest of the app, and its supported auth backends are Anthropic-direct OAuth (claude.ai Pro/Max/Team/Enterprise) first, with third-party providers a distant second supported only in narrow configurations (see [#3](#3-the-four-known-failure-modes-that-all-present-as-spinning-code-tab) below).

The official feature-comparison table is explicit:

| Surface | Third-party providers |
|---|---|
| CLI | Bedrock, Vertex, Foundry |
| Desktop | **Anthropic's API by default. Enterprise deployments can configure Vertex AI and gateway providers.** |

— ([source](https://code.claude.com/docs/en/desktop))

And in the "What's not available in Desktop" section: **"Third-party providers: Desktop connects to Anthropic's API by default. Enterprise deployments can configure Vertex AI and gateway providers via managed settings. For Bedrock or Foundry, use the CLI."** ([source](https://code.claude.com/docs/en/desktop))

That single sentence is the cleanest answer to "why does the Code tab spin when I'm on Bedrock + IdC": Anthropic does not support that configuration. The Desktop UI doesn't show a useful error because the configuration was never expected to be attempted.

### 2. Why the Code tab can't talk to Bedrock the way the CLI can: the wire-format gap

Pete's intuition in the briefing is correct: **Claude Desktop's chat/code surfaces speak the Anthropic Messages API with a Bearer token. They do not natively perform AWS SigV4 signing.** Pointing the Desktop app directly at `bedrock-runtime.<region>.amazonaws.com` will fail because Bedrock's `InvokeModel` endpoint expects SigV4-signed requests in Bedrock's own request body shape, not the Anthropic Messages shape with a Bearer token.

The official LLM-gateway docs confirm both wire formats Claude Code surfaces can target, and what a gateway must do:

> "For an LLM gateway to work with Claude Code, it must meet the following requirements. **API format**: The gateway must expose to clients at least one of the following API formats: 1. **Anthropic Messages**: `/v1/messages`, `/v1/messages/count_tokens` ... 2. **Bedrock InvokeModel**: `/invoke`, `/invoke-with-response-stream` ... 3. **Vertex rawPredict**: `:rawPredict`, `:streamRawPredict`, `/count-tokens:rawPredict`" ([source](https://code.claude.com/docs/en/llm-gateway)).

For the CLI, the way `CLAUDE_CODE_USE_BEDROCK=1` works is: Claude Code includes a Bedrock-SigV4-signing path that uses the AWS SDK default credential provider chain (env vars, then `AWS_PROFILE` → `~/.aws/config` SSO cache, then instance metadata) to obtain temp creds, then signs Bedrock InvokeModel requests with SigV4 ([source](https://code.claude.com/docs/en/amazon-bedrock)). That is the entire reason `aws sso login` works for the CLI: the temp creds land in `~/.aws/sso/cache/<hash>.json`, the SDK picks them up under `AWS_PROFILE`, and SigV4-signed Bedrock calls proceed.

There are two ways to bridge the wire-format gap without the Desktop app learning SigV4:

- **Bedrock API keys (`AWS_BEARER_TOKEN_BEDROCK`).** Announced as a 2025-2026 simplification, these are bearer tokens accepted by Bedrock's runtime endpoint in an `Authorization: Bearer ...` header, bypassing SigV4. Long-term keys are "associated with an IAM user" auto-created by Bedrock; short-term keys still require SigV4 ([source](https://aws.amazon.com/blogs/machine-learning/accelerate-ai-development-with-amazon-bedrock-api-keys/)). From the Bedrock docs: "Bedrock API keys provide a simpler authentication method without needing full AWS credentials" ([source](https://code.claude.com/docs/en/amazon-bedrock)).

- **An LLM gateway in front of Bedrock.** LiteLLM, the AWS Bedrock Access Gateway, the Cornell AI Gateway, or similar accept an Anthropic-format (`/v1/messages`) request with a Bearer token from the client, translate to Bedrock InvokeModel (or Converse), SigV4-sign with the gateway's own AWS credentials, and forward. From the client's point of view it speaks plain Anthropic Messages. LiteLLM with Claude Code is documented ([source](https://code.claude.com/docs/en/llm-gateway)):
  ```bash
  # Unified Anthropic-Messages endpoint
  export ANTHROPIC_BASE_URL=https://litellm-server:4000

  # Or pass-through Bedrock endpoint
  export ANTHROPIC_BEDROCK_BASE_URL=https://litellm-server:4000/bedrock
  export CLAUDE_CODE_SKIP_BEDROCK_AUTH=1
  export CLAUDE_CODE_USE_BEDROCK=1
  ```

For the CLI, all three approaches work. For the Desktop app's Code tab, both approaches are documented to fail in different ways: the `CLAUDE_CODE_USE_BEDROCK=1` env vars are ignored when an OAuth token is cached ([#35070](https://github.com/anthropics/claude-code/issues/35070)), and the gateway-mode "Cowork on 3P" panel works for Cowork but has Code-tab parity gaps ([#52526](https://github.com/anthropics/claude-code/issues/52526)).

In Pete's stack, the **Cornell AI Gateway** (`api.ai.it.cornell.edu`) plays the second role: it fronts Bedrock (and direct Anthropic) and accepts Bearer-token requests in Anthropic Messages format. The CLI talks to it cleanly via `ANTHROPIC_BASE_URL` + `ANTHROPIC_AUTH_TOKEN`. The Desktop Code tab does not.

### 3. The four known failure modes that all present as "spinning Code tab"

These are the documented patterns where the symptom is "Code tab spins forever / loads forever / never returns." Ordered by how well each fits Pete's setup (Cornell Mac, Jamf-managed, Cornell AI Gateway → Bedrock, `aws sso login` for AWS-direct fallback).

#### 3.1 The OAuth-token-wins-over-Bedrock-env bug ([#35070](https://github.com/anthropics/claude-code/issues/35070), closed not planned)

Pete's setup hits this if he has ever signed into Claude Desktop with an Anthropic account at any point. The cached OAuth token in `~/Library/Application Support/Claude/config.json` (macOS) overrides `~/.claude/settings.json` env vars. The Code tab calls `api.anthropic.com` with a model ID it thinks is current (`claude-sonnet-4-6`), gets `400 The provided model identifier is invalid.`, and depending on Desktop version either surfaces the error or sits on a spinner waiting for retry logic that never completes.

**Reporter's exact diagnosis:** "Settings file respects Bedrock config: `~/.claude/settings.json` with `CLAUDE_CODE_USE_BEDROCK=1` is read. But `config.json` overrides it: `%APPDATA%\Roaming\Claude\config.json` contains cached OAuth tokens that take precedence. Result: OAuth token silently overrides Bedrock configuration, forcing all requests to `https://api.anthropic.com` with invalid short-form models." Workaround was to delete `oauth:tokenCache` from `config.json`, but that "breaks Chat tab authentication and forces re-login," which makes it unusable for enterprise. Closed as "not planned" ([source](https://github.com/anthropics/claude-code/issues/35070)).

#### 3.2 The `~/.aws/config` SSO scan that forces enterprise auth ([#61851](https://github.com/anthropics/claude-code/issues/61851), open as of 2026-05-23)

Reporter's environment: macOS, Claude Code version 2.1.89, Platform: AWS Bedrock. Symptom: open the Code tab → "immediate redirect to enterprise AWS SSO with no account selection step." The Code tab scans `~/.aws/config`, finds the user's corporate SSO profile (the user in this report had configured one for a different purpose), and launches the SSO flow regardless of whether the user wanted Bedrock auth. Tried `launchctl setenv CLAUDE_CONFIG_DIR`, `~/.config/claude/env`, and direct launch injection. All confirmed that "the Code tab ignores `CLAUDE_CONFIG_DIR` entirely." Labeled `api:bedrock`, `area:auth`, `area:desktop`. Open, no Anthropic response visible ([source](https://github.com/anthropics/claude-code/issues/61851)).

For Pete: Cornell Mac has an `~/.aws/config` with the Cornell SSO endpoint. Opening the Code tab triggers this scan. If the IdC SSO browser flow gets interrupted by Cornell's TLS-inspection proxy (CrowdStrike Falcon / Zscaler / similar), the SSO completes from the user's perspective but Claude doesn't receive the callback, and the tab sits on the spinner.

This is the failure mode the **Bedrock-on-Claude-Code docs explicitly warn about** (in the CLI context): "If browser tabs spawn repeatedly when using AWS SSO, remove the `awsAuthRefresh` setting from your settings file. This can occur when corporate VPNs or TLS inspection proxies interrupt the SSO browser flow. Claude Code treats the interrupted connection as an authentication failure, re-runs `awsAuthRefresh`, and loops indefinitely" ([source](https://code.claude.com/docs/en/amazon-bedrock)). In the Desktop Code tab, there's no `awsAuthRefresh` toggle to remove, so the user can't apply the documented workaround.

#### 3.3 The "Code mode hangs silently on all models" Desktop bug ([#56182](https://github.com/anthropics/claude-code/issues/56182), open, stale)

Reporter's environment: Windows desktop app, Anthropic-direct auth (Max plan). Symptom: "no response, no error, just a spinning indicator (✻)." Began May 4, 2026 around the Opus 4.7 incidents that day and "persisted after both incidents marked resolved." Chat mode worked fine in the same app. All models (Opus 4.7, Sonnet 4.6, Haiku 4.5) hung. App restart and sign-out/in didn't fix it ([source](https://github.com/anthropics/claude-code/issues/56182)).

This one's relevant to Pete because the symptom is identical even on a clean Anthropic-direct setup. There's a class of "Code mode auth-state corruption" bugs that don't surface a user-facing error and that the user has no real way to clear from inside the app. The reporter eventually got partial recovery from a full sign-out/in cycle, but it reverted. If Pete's Code tab spinner shows the ✻ indicator, this is the matching pattern even without Bedrock in the picture.

#### 3.4 The Cowork-on-3P-Gateway settings not propagating to the Code tab ([#52526](https://github.com/anthropics/claude-code/issues/52526), open, stale)

Reporter's environment: macOS, Claude Desktop 1.3883.0, Cowork on 3P (Gateway mode), GitHub Copilot as the gateway. Reporter notes: "The `modelOverrides` and `ANTHROPIC_DEFAULT_*_MODEL_NAME` env vars work for the Code tab but have no effect on the Cowork tab picker." More importantly, the same configuration flow (`Developer → Configure third-party inference`) is shown to work for Cowork but with limitations that prove the Code tab is on a different code path ([source](https://github.com/anthropics/claude-code/issues/52526)).

The relevant repro for Pete's research:

> 1. Install Claude Desktop on macOS
> 2. Enable Developer Mode: Help → Troubleshooting → Enable Developer Mode
> 3. Open Developer → Configure third-party inference
> 4. Configure Gateway provider:
>    - Inference provider: gateway
>    - Gateway base URL: https://api.githubcopilot.com/
>    - Gateway API key: <valid GitHub Copilot OAuth token>
>    - Gateway auth scheme: bearer
> 5. Set model list (`inferenceModels`)
> 6. Quit and relaunch
> 7. Select "Cowork on 3P" at the sign-in screen

That flow exists. It is reachable for Cornell-style 3P-gateway use. But it routes through the Cowork-on-3P sign-in path, not a Code-on-3P path, and the docs do not describe an equivalent for the Code tab. Anthropic's docs page for Desktop continues to say "For Bedrock or Foundry, use the CLI" ([source](https://code.claude.com/docs/en/desktop)).

#### Pete's own sibling bug ([#59407](https://github.com/anthropics/claude-code/issues/59407), open as of 2026-05-15)

For context on the broader Desktop-3P-mode brittleness: Pete filed a related bug May 15, 2026: "Chat tab disappears when switching from Anthropic login to third-party API gateway." Repro: "1. Log into Claude Desktop with an Anthropic account — Chat tab is visible. 2. Go to Settings → switch to third-party API (3P) gateway (LiteLLM endpoint). 3. Chat tab is gone. Cowork and Code tabs remain." Pete also asked "whether there is a managed policy key that controls Chat tab visibility in 3P mode, similar to `isClaudeCodeForDesktopEnabled` for the Code tab." No Anthropic response in-thread. Labels `area:desktop`, `bug`, `platform:macos`, `platform:windows` ([source](https://github.com/anthropics/claude-code/issues/59407)).

This is the same family of bugs: **the Desktop app's per-tab feature gating is intertwined with the auth mode, and the gating logic for 3P-mode is incomplete and underdocumented**. The Chat tab disappears when 3P is on; the Code tab spins when 3P points at Bedrock. Different surface, same root cause.

### 4. The Cowork-on-Bedrock product and its gaps

This is the supported pattern that's nearest to what Pete is trying to do.

In April 2026, AWS and Anthropic jointly announced **"Claude Cowork on Amazon Bedrock"** ([source](https://aws.amazon.com/blogs/machine-learning/from-developer-desks-to-the-whole-organization-running-claude-cowork-in-amazon-bedrock/)). It is a deployment mode for Claude Desktop in which model inference goes through Bedrock in the organization's AWS account, configuration is pushed via MDM (Jamf / Intune / Group Policy), and identity is anchored in AWS IAM or Bedrock API keys.

Key documented properties:

- **Features included**: Cowork tab, Code tab, projects, artifacts, memory, file upload/export, remote connectors, skills, plugins, MCP servers.
- **Features NOT included** (exact quote): "Features that require Anthropic-hosted inference, including the **Chat tab, Computer Use, and the Skills Marketplace**, are not included because Claude Cowork routes model inference exclusively through Amazon Bedrock in your AWS account" ([source](https://aws.amazon.com/blogs/machine-learning/from-developer-desks-to-the-whole-organization-running-claude-cowork-in-amazon-bedrock/)).
- **Authentication**: "AWS IAM or Amazon Bedrock API keys" ([source](https://aws.amazon.com/blogs/machine-learning/from-developer-desks-to-the-whole-organization-running-claude-cowork-in-amazon-bedrock/)). IAM Identity Center is the AWS-native identity service that produces IAM-compatible session credentials, so it is implicitly supported through that path, but the blog post does not name IdC explicitly. AWS's solutions library `ccwb` deployment tool ([source](https://github.com/aws-solutions-library-samples/guidance-for-claude-code-with-amazon-bedrock)) (covering the CLI path, not Cowork directly) lists IdC as a first-class identity option in its v2.2 release.
- **Deployment**: "Device management systems (Jamf, Microsoft Intune, Group Policy) push configuration specifying model ID, Amazon Bedrock Inference Profile, authentication method, and organizational policies" ([source](https://aws.amazon.com/blogs/machine-learning/from-developer-desks-to-the-whole-organization-running-claude-cowork-in-amazon-bedrock/)).
- **Gateway support**: "If your organization centralizes model access through an LLM gateway, you point Claude Desktop at the gateway URL through the same managed configuration" ([source](https://aws.amazon.com/blogs/machine-learning/from-developer-desks-to-the-whole-organization-running-claude-cowork-in-amazon-bedrock/)). This is what Cornell does with `api.ai.it.cornell.edu`. The blog doesn't go into the wire-format requirement, but in practice the gateway must speak Anthropic Messages or a translation-compatible Bedrock/Vertex format per [Anthropic's LLM gateway docs](https://code.claude.com/docs/en/llm-gateway).
- **Regions**: "in AWS Regions where Claude models are available on Amazon Bedrock" with support for in-Region, geo cross-Region, and global cross-Region inference profiles.

**Code-tab caveat (Medium confidence):** A third-party walkthrough by Elevata (an enterprise-deployment consultancy whose article reads like operator notes from a real pilot) states: "some Cowork on 3P keys do not yet propagate to Code-tab sessions exactly the way they apply to the Cowork tab" and recommends "treat the Code tab as a separate validation path and also distribute Claude Code managed-settings.json when you need to pin policies, models, or sandboxing directly for coding sessions" ([source](https://elevata.io/en/claude-code-on-aws-complete-guide-bedrock-setup-self-hosted-models)). This corroborates the GitHub-issue evidence in [#52526](https://github.com/anthropics/claude-code/issues/52526) that the Code tab is on a different code path and the same configuration may not fully reach it.

This is the path Cornell would need to formally adopt — Jamf-pushed managed settings for `inferenceProvider: gateway` (or `bedrock`), `inferenceModels`, gateway URL, etc. — for the Code tab to even have a chance of working. Pete's current configuration likely sets some of this manually via the Developer Mode panel ([#52526](https://github.com/anthropics/claude-code/issues/52526) walkthrough), which gets Cowork working but leaves the Code tab on a different code path.

### 5. Recommended path forward for Pete's setup

Given what Anthropic supports today, in priority order:

**Option A (immediate, supported, recommended): Use Claude Code CLI on Bedrock from the integrated terminal.**

In any terminal (including Claude Desktop's Cowork integrated terminal, an iTerm window, or the macOS Terminal):

```bash
# In ~/.claude/settings.json (CLI reads this, distinct from Desktop's config.json):
{
  "awsAuthRefresh": "aws sso login --profile <pete-cornell-bedrock-profile>",
  "env": {
    "CLAUDE_CODE_USE_BEDROCK": "1",
    "AWS_PROFILE": "<pete-cornell-bedrock-profile>",
    "AWS_REGION": "<region-that-hosts-pete's-target-models>",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "us.anthropic.claude-sonnet-4-6",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "us.anthropic.claude-opus-4-7"
  }
}
```

Run `claude` from the terminal. This is the path Anthropic explicitly documents and that **does work today with IdC SSO** ([source: Bedrock docs, Option C: Environment variables (SSO profile)](https://code.claude.com/docs/en/amazon-bedrock)). The browser flow uses Pete's existing IdC session cache in `~/.aws/sso/cache/`.

If Cornell's TLS-inspection proxy interrupts the SSO browser flow during `awsAuthRefresh`, **drop the `awsAuthRefresh` line and run `aws sso login --profile <pete-cornell-bedrock-profile>` manually before launching `claude`** ([source: Bedrock docs, "Authentication loop with SSO and corporate proxies"](https://code.claude.com/docs/en/amazon-bedrock)).

**Option B (better long-term, if Cornell's gateway supports it): Route the CLI through the Cornell AI Gateway in Anthropic-Messages mode.**

If `api.ai.it.cornell.edu` accepts Anthropic-format `/v1/messages` and translates to Bedrock server-side (which it does, based on Pete's memory file and the Cornell AI Hub project notes), use the LiteLLM-style pattern that Anthropic explicitly supports:

```bash
# In ~/.claude/settings.json:
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.ai.it.cornell.edu",
    "ANTHROPIC_AUTH_TOKEN": "<bearer-token-from-Unit-Cornell-API-Gateway-Admin>"
  }
}
```

No `CLAUDE_CODE_USE_BEDROCK=1` needed because the gateway makes Bedrock invisible to the client. No `aws sso login` needed because the gateway handles AWS auth. This is the cleanest setup for Pete: he avoids the SigV4 dance entirely and just speaks Anthropic Messages to a Cornell-owned bearer-token endpoint.

This pattern is documented in [Anthropic's third-party integrations overview (LLM gateway tab)](https://code.claude.com/docs/en/third-party-integrations) and [Anthropic's LLM gateway configuration docs](https://code.claude.com/docs/en/llm-gateway). The `ANTHROPIC_AUTH_TOKEN` is sent as the `Authorization: Bearer ...` header.

**Option C (only if Cornell formally deploys Claude Cowork on Bedrock): use the Cowork tab in Desktop, accept that Chat is gone.**

This requires Cornell to push the AWS Cowork-on-Bedrock managed configuration via Jamf. The deal: Pete gets Cowork (Dispatch, longer agentic work), the Code tab inside Cowork mode "should" work but with the parity caveats from [Findings #4](#4-the-cowork-on-bedrock-product-and-its-gaps), and the Chat tab will be gone (by design, per the AWS blog: it requires Anthropic-hosted inference). This is the per-tab gating logic Pete is already seeing.

This is a Cornell IT-side change, not a Pete-side change. It requires the Cornell AI Hub team to evaluate the Cowork-on-Bedrock product and decide whether to ship it via Jamf 590 (the existing Cornell Mac Config Profile per Pete's memory file). The current Cornell deployment is a generic 3P gateway pointing at `api.ai.it.cornell.edu`, which is not exactly the same thing.

**Option D (not recommended, listed for completeness): Bedrock API keys + Desktop env vars.**

Setting `AWS_BEARER_TOKEN_BEDROCK=<key>` and `CLAUDE_CODE_USE_BEDROCK=1` in `~/.claude/settings.json` will work for the CLI, but per [#35070](https://github.com/anthropics/claude-code/issues/35070) the Desktop Code tab still ignores it when a cached OAuth token exists, so this doesn't unblock the Desktop Code tab. It does unblock the CLI without the SSO browser flow, which makes Option A simpler if Pete prefers not to deal with `aws sso login` at all. From the Bedrock docs: "Bedrock API keys provide a simpler authentication method without needing full AWS credentials" ([source](https://code.claude.com/docs/en/amazon-bedrock)).

### 6. Diagnostic playbook

Run these in order. The point is not to fix the Code tab; it is to confirm which of the four failure modes Pete is hitting so the report-back to Anthropic (and the Cornell AI Hub team) is precise.

1. **Confirm the CLI works.** In the macOS Terminal or Claude Desktop's Cowork integrated terminal, with `CLAUDE_CODE_USE_BEDROCK=1`, `AWS_PROFILE`, `AWS_REGION` set in `~/.claude/settings.json` env block and `aws sso login --profile <profile>` completed:
   ```bash
   aws sts get-caller-identity   # confirm SSO creds are live
   aws bedrock list-inference-profiles --region "$AWS_REGION" | head
   claude --version
   claude   # type "/status", confirm provider line shows "Amazon Bedrock"
   ```
   If `claude` works and the Desktop Code tab doesn't, that confirms the issue is in Desktop, not in Bedrock or in IdC.

2. **Read the Desktop console log.** macOS: `Console.app` → filter by `Claude`. Look for log lines from the Code tab activation. If you see references to OAuth token reads from `~/Library/Application Support/Claude/config.json` followed by 400 errors against `api.anthropic.com`, that's [#35070](https://github.com/anthropics/claude-code/issues/35070). If you see SSO browser-launch attempts that don't complete, that's [#61851](https://github.com/anthropics/claude-code/issues/61851).

3. **Check what's in `config.json`.** macOS: `cat "$HOME/Library/Application Support/Claude/config.json" | jq 'keys'`. If there's an `oauth:tokenCache` or similar key, you're set up for [#35070](https://github.com/anthropics/claude-code/issues/35070) to bite. Don't delete it (that breaks Chat tab); just note that it's there.

4. **Check `~/.aws/config`.** If there's a corporate SSO profile listed, opening the Code tab will trigger [#61851](https://github.com/anthropics/claude-code/issues/61851)'s scan. If the SSO browser flow is interrupted by Cornell's TLS proxy, the tab will spin.

5. **Verify Cornell AI Gateway connectivity from the Mac.**
   ```bash
   curl -sS --max-time 10 -X POST "https://api.ai.it.cornell.edu/chat/completions" \
     -H "Authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"model":"anthropic.claude-haiku-4-5","messages":[{"role":"user","content":"ping"}]}'
   ```
   If this returns a 200 with content, the gateway is reachable and the bearer token is valid. The fact that the Desktop Code tab still spins means the tab isn't using the gateway path. (See [Findings #1](#1-what-claude-desktops-code-tab-actually-is-and-what-authenticates-it).)

6. **Open Developer Mode and inspect 3P config.** Per [#52526](https://github.com/anthropics/claude-code/issues/52526)'s repro: Help → Troubleshooting → Enable Developer Mode → Developer → Configure third-party inference. Note which fields are populated. If `inferenceProvider: gateway` is set with the Cornell URL but the Code tab still spins, that's the Cowork-vs-Code parity gap from [Findings #4](#4-the-cowork-on-bedrock-product-and-its-gaps).

7. **File a Desktop-specific bug.** Use the precise model: "Code tab spins forever with 3P API mode pointing at <Cornell AI Gateway / direct Bedrock>. CLI works fine with the same settings file. Per docs the Code tab in Desktop is not supported for Bedrock, so either (a) document that the Code tab is also unsupported for arbitrary 3P gateways behind which Bedrock sits, or (b) honor the same env vars the CLI does." Reference [#35070](https://github.com/anthropics/claude-code/issues/35070), [#61851](https://github.com/anthropics/claude-code/issues/61851), and Pete's own [#59407](https://github.com/anthropics/claude-code/issues/59407).

### 7. Limitations of Claude Desktop in 3P mode (full inventory)

What works and what doesn't in Claude Desktop when configured for third-party API mode (specifically a gateway in front of Bedrock):

**Documented to work (Cowork on Bedrock per AWS ML blog and Anthropic enterprise docs):**
- Cowork tab (Dispatch, longer agentic work)
- Projects
- Artifacts
- Memory
- File upload and export
- Remote connectors
- Skills (your own; the marketplace is gated)
- Plugins
- MCP servers
- MDM-managed configuration (Jamf, Intune, GPO)
- In-region, cross-region, and global cross-region Bedrock inference profiles

**Documented to NOT work (per AWS ML blog quoting Anthropic):**
- Chat tab (requires Anthropic-hosted inference)
- Computer Use (requires Anthropic-hosted inference)
- Skills Marketplace (requires Anthropic-hosted inference)
- `/desktop` migration from CLI (explicitly: "not available with API key authentication or on Bedrock, Vertex, or Foundry" ([source](https://code.claude.com/docs/en/desktop)))

**Documented as Anthropic-direct only on Desktop, no 3P support:**
- Bedrock as a Desktop-native provider with `CLAUDE_CODE_USE_BEDROCK=1`. Quote: "For Bedrock or Foundry, use the CLI" ([source](https://code.claude.com/docs/en/desktop)).

**Documented as enterprise-only 3P providers for Desktop:**
- Vertex AI
- "Gateway providers" (LLM gateways speaking Anthropic Messages, Bedrock InvokeModel, or Vertex rawPredict — see [LLM gateway docs](https://code.claude.com/docs/en/llm-gateway))

**Observed-broken or under-supported on Desktop in 3P/gateway mode (from GitHub issues):**
- Code tab honoring `CLAUDE_CODE_USE_BEDROCK=1` env when a cached Desktop OAuth token exists ([#35070](https://github.com/anthropics/claude-code/issues/35070), closed not planned)
- Code tab respecting `CLAUDE_CONFIG_DIR` ([#61851](https://github.com/anthropics/claude-code/issues/61851), open)
- Code tab using the same account session as Chat tab when an enterprise SSO is detected in `~/.aws/config` ([#61851](https://github.com/anthropics/claude-code/issues/61851), open)
- Chat tab visibility when 3P gateway mode is enabled ([#59407](https://github.com/anthropics/claude-code/issues/59407), open, Pete's own report)
- Cowork-on-3P model picker honoring full version IDs ([#52526](https://github.com/anthropics/claude-code/issues/52526), stale)
- Organization-deployed plugins appearing in Desktop Code tab ([#53498](https://github.com/anthropics/claude-code/issues/53498), open, stale)

**Region / model considerations (any Bedrock path, including CLI):**
- `AWS_REGION` is a required environment variable. "Claude Code does not read from the `.aws` config file for this setting" ([source](https://code.claude.com/docs/en/amazon-bedrock)).
- Modern Claude models require cross-region inference profiles with `us.`, `eu.`, or `global.` prefixes
- Prompt caching may not be available in all Bedrock regions
- The Bedrock Invoke API is used, not the Converse API
- `/logout` is unavailable on Bedrock; auth is handled through AWS credentials

### 8. Adding Skills and web search on Bedrock (the CLI recipe)

Pete asked the follow-up question that Section 5 implicitly raises: once you've fallen back to the **Claude Code CLI on Bedrock** (Option A), how do you get **Agent Skills** and **web search** working there? Section 4 already noted that AWS's Cowork-on-Bedrock blurb names "the Chat tab, Computer Use, and the Skills Marketplace" as the exclusions; this section makes that precise for the CLI scenario Pete will actually be operating in.

The short answer: **custom skills work natively on Bedrock, the marketplace install path also works, but Anthropic's pre-built skills that depend on the hosted code-execution container do not. Anthropic's server-side `web_search` and `web_fetch` tools are blocked by Bedrock; Pete should use his own `searxng` MCP (already in his stack) and the built-in `WebFetch` tool to fill the gap.**

#### 8.1 Agent Skills: what works on Bedrock and what doesn't

**Skills are a client-side feature in Claude Code.** Anthropic's docs are explicit:

> "Skills are reusable, filesystem-based resources that provide Claude with domain-specific expertise... Custom Skills in Claude Code are filesystem-based and don't require API uploads" ([source: Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).

A skill is a directory with a `SKILL.md` file containing YAML frontmatter (`name`, `description`) and markdown instructions. Claude Code loads skill metadata at startup into the system prompt; full skill content loads on-demand when triggered ([source: Claude Code skills docs](https://code.claude.com/docs/en/skills)). All of this happens **client-side, in the user's filesystem and process**. The Bedrock back-end never sees "a skill" — it sees normal Claude Code messages with the skill's text inlined when triggered. This means model-provider choice is irrelevant to whether skills work.

The Anthropic docs page on Agent Skills makes the surface support explicit:

> "[Claude Code](https://code.claude.com/docs/en/overview) supports only Custom Skills. **Custom Skills**: Create Skills as directories with SKILL.md files. Claude discovers and uses them automatically. Custom Skills in Claude Code are filesystem-based and don't require API uploads." ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview))

And from the Claude Code docs:

> "Where you store a skill determines who can use it: `~/.claude/skills/<skill-name>/SKILL.md`, `.claude/skills/<skill-name>/SKILL.md`, `<plugin>/skills/<skill-name>/SKILL.md`" ([source](https://code.claude.com/docs/en/skills)).

Pete's skills under `~/.claude/skills/` (Tank, Apoc, Coach, Beat, Editor, etc., all enumerated in his routing table) **are loaded directly from disk and work identically against Bedrock, Vertex, Foundry, or Anthropic-direct**. The skill's body becomes part of the system prompt; the LLM sees text, not "a skill object."

**What's blocked: pre-built Anthropic Skills that need the hosted code-execution container.** Anthropic ships first-party skills like `pdf`, `xlsx`, `docx`, `pptx` (the document-creation ones). Per the Skills overview:

> "Pre-built Agent Skills are available on claude.ai, the Claude API, [Claude Platform on AWS](/docs/en/build-with-claude/claude-platform-on-aws), and [Microsoft Foundry](/docs/en/build-with-claude/claude-in-microsoft-foundry)." ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview))

Bedrock is **conspicuously absent** from that list. The reason is in the prerequisites: pre-built skills require three beta headers (`code-execution-2025-08-25`, `skills-2025-10-02`, `files-api-2025-04-14`) and run "in the code execution container" alongside the Files API ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)). Bedrock doesn't expose Anthropic's hosted code-execution / Files API surface, so the API-uploaded pre-built skills can't run there. ([Note: "Claude Platform on AWS" is a separate Anthropic-operated AWS Marketplace product that does support pre-built skills; that is not the same as "Claude in Amazon Bedrock," which is what Pete is using. The Claude Platform on AWS distinction is documented in the Bedrock page header: it "serves Claude through the Messages API at /anthropic/v1/messages... For an Anthropic-operated alternative on AWS with AWS Marketplace billing and typically same-day feature access, see Claude Platform on AWS" ([source](https://platform.claude.com/docs/en/build-with-claude/claude-in-amazon-bedrock)).])

The closely-related "Skills Marketplace" feature in Claude Desktop's Cowork tab is the one the AWS blog explicitly excludes from Cowork-on-Bedrock ([source](https://aws.amazon.com/blogs/machine-learning/from-developer-desks-to-the-whole-organization-running-claude-cowork-in-amazon-bedrock/) and see [Section 4](#4-the-cowork-on-bedrock-product-and-its-gaps) above). That's a separate, Anthropic-hosted UI for browsing and installing community skills into the Desktop app's Chat/Cowork surfaces. It's distinct from the Claude Code CLI plugin marketplace (next paragraph), which works fine on Bedrock.

**What does work on Bedrock CLI: the Claude Code plugin marketplace.** Anthropic ships a plugin-marketplace mechanism in the CLI itself, and it's filesystem-based:

> "You can register this repository as a Claude Code Plugin marketplace by running the following command in Claude Code: `/plugin marketplace add anthropics/skills`. Then, to install a specific set of skills: Select `Browse and install plugins` → Select `anthropic-agent-skills` → Select `document-skills` or `example-skills` → Select `Install now`." ([source: anthropics/skills repo](https://github.com/anthropics/skills))

This installs skill folders into the CLI's local plugin directory. The skills then run client-side like any other custom skill. So Pete can use `/plugin marketplace add anthropics/skills` (or any third-party marketplace) on his Bedrock-backed CLI and the install itself succeeds. Whether each individual installed skill actually works depends on what it needs at runtime: a skill that calls `git`, edits files, or runs bash is fine; a skill that requires the Anthropic Files API or hosted code-execution container will fail when triggered (the bash commands in `SKILL.md` will run, but anything that calls Anthropic-hosted infrastructure won't).

**Bundled skills shipped with the CLI all work on Bedrock.** Claude Code ships built-in bundled skills like `/code-review`, `/batch`, `/debug`, `/loop`, `/claude-api`, `/run`, `/verify` ([source](https://code.claude.com/docs/en/skills)). These are "prompt-based: they give Claude detailed instructions and let it orchestrate the work using its tools" ([source: same](https://code.claude.com/docs/en/skills)), so they're 100% client-side and work identically on any model provider. Pete already uses several of these (e.g., `/code-review`, `/verify`, `/run`) per his skill list.

**Reconciling an apparent contradiction in the official docs.** The Anthropic Bedrock page lists "Agent infrastructure (Agent Skills, MCP connector, programmatic tool calling)" under **Features not supported** ([source](https://platform.claude.com/docs/en/build-with-claude/claude-in-amazon-bedrock)). Read in isolation, that looks like a flat "no Agent Skills on Bedrock." It is not. That bullet refers specifically to **API-uploaded Skills via the Skills API** (the server-side path that requires the `skills-2025-10-02` + `code-execution-2025-08-25` + `files-api-2025-04-14` beta headers and runs inside Anthropic's hosted code-execution container). That whole pipeline is what Bedrock doesn't expose. **Claude Code CLI's filesystem-based custom skills are a different mechanism entirely**: the skill content is loaded by the CLI on Pete's machine and inlined into the system prompt as plain text; the Bedrock back-end only ever sees Messages-API text, no "skill object," no beta headers, no Files API. That is why filesystem skills work and API-uploaded pre-built skills don't — they are two different things sharing a name.

**Confidence: High** that custom and bundled skills work on Bedrock; **High** that the marketplace install command (`/plugin marketplace add`) works on Bedrock; **High** that pre-built Anthropic skills requiring the hosted code-execution container do not work on Bedrock.

Independent confirmation in the wild: Gunnar Grosch's series on running Claude Code with Bedrock spends the entire fifth post on "Extending Claude Code with Plugins and Skills for AWS Development" with no Bedrock-specific caveats on skills, only on the bundled marketplace concept ([source](https://dev.to/gunnargrosch/extending-claude-code-with-plugins-and-skills-for-aws-development-4p9o)). Haowen Huang's dev.to walkthrough on Claude Agent SDK with Skills on Bedrock concludes: "Everything from the original SDK works: Skills, Subagents with parallel execution, MCP servers (Notion, etc.), All built-in tools (WebSearch, Bash, Write, etc.)" ([source](https://dev.to/haowen_huang/running-claude-agent-sdk-with-skills-on-amazon-bedrock-el6)) — though see Section 8.2 below on the WebSearch caveat in that quote.

#### 8.2 Web search: what works on Bedrock and what doesn't

This is the more constrained side of the answer.

**Anthropic's `web_search` server tool is NOT available on Amazon Bedrock.** Three independent confirmations:

1. Anthropic's own web-search-tool docs: "The web search tool (with and without dynamic filtering) is available on the Claude API, [Claude Platform on AWS](/docs/en/build-with-claude/claude-platform-on-aws), and [Microsoft Foundry](/docs/en/build-with-claude/claude-in-microsoft-foundry). **Web search is not available on Amazon Bedrock.**" ([source](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool))

2. AWS re:Post answer (AWS Official tag): "As of the latest available information, Amazon Bedrock does not directly support the `web_search` tool for Anthropic models through the Converse API in the way you've described." Reproducer's error: `An error occurred (ValidationException) when calling the Converse operation: The model returned the following errors: tools.0: Input tag 'web_search_20250305' found using 'type' does not match any of the expected tags: 'bash_20250124', 'custom', 'text_editor_20250124'` ([source](https://repost.aws/questions/QUSd3wAByQTtyzUPzgqss3TQ/web-search-for-anthropic-models-in-bedrock)).

3. Operator post: "Anthropic's Claude 3 models recently introduced new tools such as `web_search_20250305` and `code_execution_20250305`. These are available via the Anthropic API, but Bedrock hasn't yet added support for these tool types. As a result, any tool request using these identifiers is blocked before it ever reaches the Claude model" ([source](https://www.tech-reader.blog/2025/05/solve-validationexception-websearch-not.html)).

The same is true for **Anthropic's `web_fetch` server tool**: "The web fetch tool (with and without dynamic filtering) is available on the Claude API, Claude Platform on AWS, and Microsoft Foundry. **It is not currently available on Amazon Bedrock or Vertex AI.**" ([source](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-fetch-tool)).

**Claude Code's built-in `WebSearch` tool is hidden on Bedrock-backed sessions.** Reverse-engineering of Claude Code's runtime by Mikhail Shilkov:

> "When it doesn't know the URL, Claude Code issues search requests to Anthropic's server-side `WebSearch` tool, the same one that Claude chat uses... **The server-side search tool is available on Anthropic's first-party API but it isn't supported on Bedrock/Vertex. If Claude Code is configured to use those platforms, Claude Code hides the `WebSearch` tool entirely.**" ([source](https://mikhail.io/2025/10/claude-code-web-tools/))

This is exactly the behavior reported in the closed-as-duplicate bug [#32385](https://github.com/anthropics/claude-code/issues/32385) ("WebSearch, Token Counting API unavailable on bedrock-connected billing accounts"), and the upstream [#11369](https://github.com/anthropics/claude-code/issues/11369) ("web search not working (Did 0 searches in 7s)") that has comments from multiple users confirming the same behavior on Bedrock, LiteLLM-routed-to-Bedrock, and Vertex deployments. The bug was closed inactive without a fix.

**Claude Code's built-in `WebFetch` tool: likely works on Bedrock.** This is the important asymmetry, and one Pete should plan around. Per Mikhail Shilkov's runtime analysis, `WebFetch` runs through an Anthropic-operated pipeline: a backend `domain_info` endpoint resolves and fetches the URL, the HTML is converted to Markdown, and a fast model (Haiku 3.5) on Anthropic's API summarizes the result ([source](https://mikhail.io/2025/10/claude-code-web-tools/)). The fetch + summarization pipeline is operated by Anthropic, but the inference does **not** depend on the configured backend model — Haiku 3.5 is the summarizer regardless of whether Pete is on Bedrock, Vertex, or Anthropic-direct. That's the architectural reason `WebFetch` should not be gated by the Bedrock setting in the same way `WebSearch` is. **(Medium confidence: I did not find an explicit Anthropic doc statement that `WebFetch` works on Bedrock-backed Claude Code sessions, and Shilkov's source describes the Anthropic-backend pipeline rather than a purely-local fetch. The conclusion is inferred from the architecture, the existence of a separate Haiku-summarization call, and the absence of `WebFetch` from the bug reports about WebSearch on Bedrock. Pete should validate with a `WebFetch` invocation against a known URL in his first Bedrock CLI session.)** If `WebFetch` is also blocked, the searxng MCP recommendation below covers fetch as well as search.

**Third-party MCP servers fill the gap.** Because Anthropic's `web_search` server tool is blocked on Bedrock and Claude Code's built-in `WebSearch` is therefore hidden, the only architecturally-clean way to get search on Bedrock-backed Claude Code is an MCP server. MCP servers run client-side and pass results back to Claude as ordinary tool-use blocks, so they're completely model-provider-agnostic. Pete's options, ranked:

1. **Pete's own `pete-builds/mcp-searxng`** ([source](https://github.com/pete-builds/mcp-searxng)). His memory file flags this as his preferred default ([[feedback_searxng_primary]]). It's a FastMCP server, wraps a self-hosted SearXNG, requires no API keys, and ships 9 tools including `search`, `search_deep`, `search_tech`, `read_url`, and a person-vetting fan-out. This is the obvious recommendation and the one he should put in his `~/.claude/settings.json` MCP block first.

2. **Brave Search MCP server** (`@brave/brave-search-mcp-server`) — official Brave-maintained, web/local/places/image/video/news/AI summarization tools, STDIO + HTTP transports ([source](https://github.com/brave/brave-search-mcp-server)). Free tier of Brave Search API is fine for moderate use.

3. **Tavily MCP** — AI-focused search and extraction, sophisticated filtering ([source](https://docs.tavily.com/documentation/mcp)). Pay-per-call.

4. **Exa MCP** — semantic/AI-tuned search optimized for LLM workflows ([source](https://exa.ai/docs/reference/exa-mcp)). Best for research-style queries.

5. **Gemini-search skill** — Pete already has this in his skills list (uses Cornell Gateway Gemini Enterprise Web Search). Works on Bedrock because it makes its own outbound HTTP call from Pete's shell.

For Pete, recommendation #1 is the answer. The other options exist for completeness.

#### 8.3 Recommended `~/.claude/settings.json` for Pete

Combining the Section 5 Bedrock auth recipe with the Section 8.1/8.2 skills + web-search recipe, the consolidated `~/.claude/settings.json` for the **Claude Code CLI on Bedrock** path looks like this. (Skills are auto-discovered from `~/.claude/skills/` and `~/.claude/plugins/`; nothing to configure in settings.json for skills themselves.)

```json
{
  "awsAuthRefresh": "aws sso login --profile <pete-cornell-bedrock-profile>",
  "env": {
    "CLAUDE_CODE_USE_BEDROCK": "1",
    "AWS_PROFILE": "<pete-cornell-bedrock-profile>",
    "AWS_REGION": "<region-that-hosts-pete's-target-models>",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "us.anthropic.claude-sonnet-4-6",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "us.anthropic.claude-opus-4-7"
  },
  "mcpServers": {
    "searxng": {
      "type": "http",
      "url": "http://<pete's-searxng-mcp-host>:<port>"
    }
  }
}
```

If Cornell's TLS-inspection proxy interrupts the SSO browser flow during `awsAuthRefresh`, **drop the `awsAuthRefresh` line and run `aws sso login --profile <pete-cornell-bedrock-profile>` manually** before launching `claude` ([source: Bedrock docs, "Authentication loop with SSO and corporate proxies"](https://code.claude.com/docs/en/amazon-bedrock)).

For MCP server transport format, Pete's existing mcp-searxng server is FastMCP / Streamable HTTP per his workspace conventions; the `http` transport above is the right one for that. For STDIO-based MCP servers (Brave, Tavily, etc., when run via `npx` locally), use:

```json
"mcpServers": {
  "brave-search": {
    "command": "npx",
    "args": ["-y", "@brave/brave-search-mcp-server"],
    "env": { "BRAVE_API_KEY": "<key>" }
  }
}
```

Per Claude Code's MCP docs: "When configuring MCP servers via JSON in `.mcp.json`, `~/.claude.json`, or `claude mcp add-json`, the type field accepts `streamable-http` as an alias for `http`. The MCP specification uses the name `streamable-http` for this transport, so configurations copied from server documentation work without modification." ([source](https://code.claude.com/docs/en/mcp))

**Custom skills** require no configuration. Drop a `SKILL.md` (with `name` and `description` YAML frontmatter) under `~/.claude/skills/<name>/SKILL.md` and it's available immediately. To install the official Anthropic example skills:

```
/plugin marketplace add anthropics/skills
/plugin install example-skills@anthropic-agent-skills
```

Per the anthropics/skills repo ([source](https://github.com/anthropics/skills)). Note: skills that depend on Anthropic-hosted features (Files API, hosted code execution) may fail when triggered on Bedrock; skills that are pure bash + filesystem operations work fine.

#### 8.4 Desktop Code tab vs CLI: per-feature matrix

A single table to settle the surface question for each feature, for Pete's stack (Bedrock backend, IdC SSO, Cornell AI Gateway optional):

| Feature | Desktop Code tab (3P/Bedrock mode) | Claude Code CLI on Bedrock |
|---|---|---|
| Custom skills (`~/.claude/skills/<n>/SKILL.md`) | Unverifiable (tab broken, see Section 3) | **Yes** ([source](https://code.claude.com/docs/en/skills)) |
| Bundled skills (`/code-review`, `/debug`, `/loop`, `/run`, `/verify`) | Unverifiable | **Yes** ([source](https://code.claude.com/docs/en/skills)) |
| Anthropic pre-built skills (pdf, xlsx, docx, pptx) | Not via Skills Marketplace ([source](https://aws.amazon.com/blogs/machine-learning/from-developer-desks-to-the-whole-organization-running-claude-cowork-in-amazon-bedrock/)) | Skills install but runtime requires hosted code-execution which Bedrock lacks; **No** for the document-skills as designed ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)) |
| `/plugin marketplace add anthropics/skills` | Unverifiable | **Yes** (install succeeds; individual skill runtime success varies, see above) ([source](https://github.com/anthropics/skills)) |
| MCP servers (searxng, Brave, Tavily, Exa) | Unverifiable | **Yes** ([source](https://code.claude.com/docs/en/mcp)) |
| Anthropic `web_search` server tool (the one used by Claude Code's built-in WebSearch) | Not on Bedrock ([source](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool)) | **No**; Claude Code hides WebSearch when configured for Bedrock ([source](https://mikhail.io/2025/10/claude-code-web-tools/)) |
| Anthropic `web_fetch` server tool | Not on Bedrock ([source](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-fetch-tool)) | **No** as a server tool |
| Claude Code built-in `WebFetch` (client-side, calls Haiku to summarize) | Unverifiable | Likely **Yes** (Medium confidence; client-side fetch + Haiku summarization is independent of the configured backend model, though the Haiku call still goes to Anthropic's API) |
| Hooks, subagents, custom commands | Unverifiable | **Yes** (all client-side) |
| Skills Marketplace (Desktop UI for browsing/installing community skills) | **No** on Cowork-on-Bedrock; "not included because Claude Cowork routes model inference exclusively through Amazon Bedrock" ([source](https://aws.amazon.com/blogs/machine-learning/from-developer-desks-to-the-whole-organization-running-claude-cowork-in-amazon-bedrock/)) | N/A (CLI uses the plugin marketplace, which is different) |
| Computer Use | **No** on Cowork-on-Bedrock ([source: same](https://aws.amazon.com/blogs/machine-learning/from-developer-desks-to-the-whole-organization-running-claude-cowork-in-amazon-bedrock/)) | **No** on Bedrock |

"Unverifiable" above means: the Code tab is broken in this configuration ([Section 3](#3-the-four-known-failure-modes-that-all-present-as-spinning-code-tab)), so feature availability inside the tab can't be tested, and the official docs don't enumerate it.

#### 8.5 The "third option": Mantle and full-bridge gateways

Two architectural patterns recently shipped that bridge the Bedrock feature gap. Pete doesn't need either today, but they're worth knowing about as the "what if I want Anthropic-direct features without giving up AWS billing" path.

**Mantle (Anthropic-shape over Bedrock auth).** Anthropic and AWS jointly shipped a Bedrock endpoint that serves Claude models through the native Anthropic API shape rather than the Bedrock Invoke API. From the Claude Code Bedrock docs:

> "Mantle is an Amazon Bedrock endpoint that serves Claude models through the native Anthropic API shape rather than the Bedrock Invoke API. It uses the same AWS credentials, IAM permissions, and `awsAuthRefresh` configuration described earlier on this page." ([source](https://code.claude.com/docs/en/amazon-bedrock))

Set `CLAUDE_CODE_USE_MANTLE=1` and the CLI talks to `bedrock-mantle.{region}.api.aws/anthropic/v1/messages` ([source](https://platform.claude.com/docs/en/build-with-claude/claude-in-amazon-bedrock)). The key thing for Pete's question: **Mantle uses the Anthropic Messages shape, which means it could in principle support server-side tools that the Bedrock Invoke API doesn't accept**. As of the date of this report, I did not find an explicit statement that Mantle has shipped `web_search` support, but the architecture removes the wire-format objection. The Bedrock docs mention Mantle without enumerating per-tool support; per-model availability also differs from the standard Bedrock catalog ([source](https://code.claude.com/docs/en/amazon-bedrock)). **(Confidence: Low on Mantle web_search support specifically; this would need a direct probe or a future Anthropic doc update.)** Mantle requires Claude Code v2.1.94 or later.

**Full-bridge gateways (LiteLLM, CCAG, Bedrock Access Gateway).** Three different community/Anthropic-adjacent projects translate between Anthropic Messages format and Bedrock InvokeModel, often adding `web_search` interception by routing through a third-party search provider:

- **LiteLLM** has a `websearch_interception` callback that detects Claude Code's `web_search` tool requests, executes them via Perplexity/Tavily/SearXNG/etc., and returns results in Anthropic format before the request ever reaches Bedrock ([source](https://docs.litellm.ai/docs/tutorials/claude_code_websearch)). The implementation has been buggy: an open issue ([#26252](https://github.com/BerriAI/litellm/issues/26252)) confirms that as of v1.83.7-stable the interception doesn't fire for Claude Code's exact `web_search_20250305` request shape when routed via Bedrock pass-through, and a pending PR has a "native provider skip" that would re-break it. The intent is right; verify the current state if Pete adopts this path.

- **CCAG (Claude Code AWS Gateway)** is purpose-built for this scenario: "When Claude Code connects to Bedrock directly (`CLAUDE_CODE_USE_BEDROCK=1`), it operates in a reduced-capability mode — extended thinking, web search, and some tool use features are disabled on the client side. CCAG presents as the Anthropic Messages API, so Claude Code enables its full feature set while inference still runs through your AWS account... Web search interception with per-user configurable providers (DuckDuckGo, Tavily, Serper, or custom)" ([source](https://github.com/antkawam/claude-code-aws-gateway)). Self-hosted Rust gateway, Docker-deployable, OIDC SSO, team budgets — overkill for Pete personally but exactly what Cornell IT would want if they were standardizing Cornell-wide.

- The **bedrock-access-gateway** (AWS Samples) is the simplest of the three but does not implement Anthropic Messages API (only OpenAI Chat Completions), so it's wrong for Claude Code ([source](https://dev.to/aws-builders/bedrock-for-ai-coding-tools-mantle-vs-gateway-vs-litellm-a-decision-guide-for-aws-credit-burners-1h01)).

For Pete's personal stack, none of this is necessary. The **searxng MCP** is the minimal, supported, already-in-his-stack answer to "I need web search on Claude Code over Bedrock." Mantle and CCAG are escalation paths if Cornell ever decides to formalize Anthropic-feature-parity for Bedrock-backed developers.

## Confidence Assessment

| Claim | Confidence | Why |
|---|---|---|
| Claude Desktop's Code tab does not natively support Bedrock | **High** | Anthropic's own Desktop docs say so verbatim: "For Bedrock or Foundry, use the CLI" ([source](https://code.claude.com/docs/en/desktop)). |
| The Code tab spins because the configuration is not supported, not because of a transient bug | **High** | Documented product limitation. Closed-as-not-planned status of [#35070](https://github.com/anthropics/claude-code/issues/35070) reinforces this. |
| Pete's likely failure mode is the OAuth-token-wins bug ([#35070](https://github.com/anthropics/claude-code/issues/35070)) | **Medium** | Fits the symptom and matches Pete's "tried to point Desktop at Bedrock via 3P API" scenario, but [#61851](https://github.com/anthropics/claude-code/issues/61851) (the `~/.aws/config` scan) is equally plausible given Pete is on a Cornell Mac with an SSO profile configured. Without log lines, can't pick definitively. |
| Pete's "Chat tab disappears in gateway mode" bug ([#59407](https://github.com/anthropics/claude-code/issues/59407)) and the spinning Code tab are the same class of issue | **Medium-high** | Both are per-tab feature gating tied to 3P-mode logic; both have been filed by users in similar enterprise-gateway configurations; both are open without Anthropic response. Inference, not direct confirmation. |
| Cowork on Bedrock (AWS-blessed) includes the Code tab with caveats | **High** | AWS ML blog explicitly lists what's in and out; "Chat tab, Computer Use, Skills Marketplace" are the exclusions ([source](https://aws.amazon.com/blogs/machine-learning/from-developer-desks-to-the-whole-organization-running-claude-cowork-in-amazon-bedrock/)). |
| Cowork-on-3P-Gateway managed-settings don't fully propagate to the Code tab | **Medium** | Two corroborating sources: [#52526](https://github.com/anthropics/claude-code/issues/52526) (the reporter explicitly says model overrides work in Code tab but not Cowork picker, implying separate code paths) and the Elevata operator guide. Neither is from Anthropic. |
| The CLI `aws sso login` path is the supported workaround | **High** | Direct from Anthropic's Bedrock docs as Option C. The TLS-inspection-proxy auth-loop and the workaround (manual `aws sso login` instead of `awsAuthRefresh`) are also explicitly documented ([source](https://code.claude.com/docs/en/amazon-bedrock)). |
| Claude Desktop's surfaces speak Anthropic Messages with a Bearer token, not native SigV4 | **High** | Implied by the gateway requirements page (lists Anthropic Messages format as the primary supported gateway protocol, with Bedrock InvokeModel as a separate alternative that gateways may also expose) and by the fact that `CLAUDE_CODE_SKIP_BEDROCK_AUTH=1` exists as a documented setting for "If gateway handles AWS auth" scenarios ([source: LLM gateway docs](https://code.claude.com/docs/en/llm-gateway), [source: third-party integrations](https://code.claude.com/docs/en/third-party-integrations)). |
| Bedrock API keys (`AWS_BEARER_TOKEN_BEDROCK`) bypass SigV4 for long-term keys | **High** | Per AWS ML blog on Bedrock API keys: "Short-term API keys use AWS Signature Version 4 for authentication" implies long-term don't; the Anthropic Bedrock docs list it as an alternative auth method with simpler bearer-token semantics ([source: AWS ML blog](https://aws.amazon.com/blogs/machine-learning/accelerate-ai-development-with-amazon-bedrock-api-keys/), [source: Anthropic Bedrock docs](https://code.claude.com/docs/en/amazon-bedrock)). |
| Anthropic has no documented managed-policy key to hide/show the Code tab in 3P mode that would resolve Pete's issue | **Low** | Could not find one in the documented enterprise-configuration policy list. `isClaudeCodeForDesktopEnabled` exists as an enable/disable for the whole Code tab feature, but no per-mode gating. The Desktop docs list `permissions.disableBypassPermissionsMode`, `disableAutoMode`, `autoMode`, `sshConfigs`, `sshHostAllowlist`, `managedMcpServers` as the supported keys ([source](https://code.claude.com/docs/en/desktop)). None of those address 3P-mode tab visibility. |
| Custom skills (filesystem `SKILL.md`) work on Claude Code CLI when Bedrock is the model provider | **High** | Anthropic docs are explicit: "Custom Skills in Claude Code are filesystem-based and don't require API uploads" ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)). Architecture: skill content is inlined into the system prompt client-side; the model provider is transparent. Independent corroboration in Gunnar Grosch's AWS-development series ([source](https://dev.to/gunnargrosch/extending-claude-code-with-plugins-and-skills-for-aws-development-4p9o)) and Haowen Huang's Bedrock walkthrough ([source](https://dev.to/haowen_huang/running-claude-agent-sdk-with-skills-on-amazon-bedrock-el6)). |
| Anthropic pre-built skills requiring hosted code-execution (`pdf`, `xlsx`, `docx`, `pptx`) do NOT work on Bedrock | **High** | "Pre-built Agent Skills are available on claude.ai, the Claude API, Claude Platform on AWS, and Microsoft Foundry" ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)) — Bedrock omitted. Mechanism: pre-built skills require `code-execution-2025-08-25` and `files-api-2025-04-14` beta headers, which Bedrock doesn't expose. |
| Anthropic's `web_search` server tool is NOT available on Amazon Bedrock | **High** | Direct from Anthropic docs: "Web search is not available on Amazon Bedrock" ([source](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool)). AWS re:Post official answer confirms the same ([source](https://repost.aws/questions/QUSd3wAByQTtyzUPzgqss3TQ/web-search-for-anthropic-models-in-bedrock)) including the exact ValidationException error from Bedrock Converse. Multiple operator posts reproduce ([source](https://www.tech-reader.blog/2025/05/solve-validationexception-websearch-not.html)). |
| Anthropic's `web_fetch` server tool is NOT available on Amazon Bedrock | **High** | "It is not currently available on Amazon Bedrock or Vertex AI" ([source](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-fetch-tool)). |
| Claude Code's built-in `WebSearch` tool is silently hidden when configured for Bedrock | **High** | Runtime reverse-engineering: "If Claude Code is configured to use those platforms [Bedrock/Vertex], Claude Code hides the `WebSearch` tool entirely" ([source](https://mikhail.io/2025/10/claude-code-web-tools/)). Corroborated by bug reports [#11369](https://github.com/anthropics/claude-code/issues/11369) (closed as inactive without fix) and the duplicate [#32385](https://github.com/anthropics/claude-code/issues/32385). Multiple users in #11369 confirm the same on Bedrock and LiteLLM-to-Bedrock paths. |
| Claude Code's built-in `WebFetch` tool works on Bedrock (client-side HTTP + Haiku summarization) | **Medium** | Architectural inference from runtime analysis ([source](https://mikhail.io/2025/10/claude-code-web-tools/)): the fetch is client-side and the summarization is a separate Haiku call. Not contradicted by any source. Not explicitly confirmed in Anthropic docs. Pete should verify with a single test invocation in his first Bedrock CLI session. |
| MCP servers (including searxng) are model-provider-agnostic and work on Bedrock | **High** | MCP servers run client-side, expose tools to Claude as ordinary tool-use blocks. No provider-side dependency. Confirmed in many community walkthroughs ([source](https://dev.to/gunnargrosch/extending-claude-code-with-plugins-and-skills-for-aws-development-4p9o)), in Anthropic's MCP docs ([source](https://code.claude.com/docs/en/mcp)), and in the Haowen Huang Bedrock-SDK confirmation that "MCP servers (Notion, etc.)" work unchanged ([source](https://dev.to/haowen_huang/running-claude-agent-sdk-with-skills-on-amazon-bedrock-el6)). |
| `/plugin marketplace add anthropics/skills` install path works on Bedrock CLI | **Medium-High** | Architecture: the marketplace install is a filesystem operation that places skill folders under the CLI's local plugin directory; runtime success depends on each skill's specific dependencies, but the install command itself doesn't touch the model provider. No source directly states this for Bedrock, but no source contradicts it; the anthropics/skills repo documents the install command without provider caveats ([source](https://github.com/anthropics/skills)). |
| Mantle endpoint may unlock some Anthropic-direct tools on Bedrock | **Low** | Mantle uses Anthropic Messages shape over Bedrock auth ([source](https://platform.claude.com/docs/en/build-with-claude/claude-in-amazon-bedrock)), which removes the wire-format objection that blocks `web_search` on Bedrock Invoke. But Anthropic has not explicitly listed `web_search` as supported on Mantle. Model lineup also differs from standard Bedrock catalog. Not a current recommendation; Pete should not pursue this unless Cornell formally adopts Mantle. |

## Open Questions

These remain unresolved after this research pass and would need either Pete to provide more info, or a follow-up with Anthropic enterprise support / the Cornell AI Hub team:

- **What does the Claude Desktop Console.app log actually say** when Pete reproduces the spinning Code tab? Would let us pick between [#35070](https://github.com/anthropics/claude-code/issues/35070), [#61851](https://github.com/anthropics/claude-code/issues/61851), and #56182 definitively.
- **What's in Pete's `~/Library/Application Support/Claude/config.json`?** Specifically: is there an `oauth:tokenCache` from a previous Anthropic-direct login? If yes, this almost certainly triggers [#35070](https://github.com/anthropics/claude-code/issues/35070).
- **Is the Cornell AI Gateway configured in Pete's Desktop app via Settings → 3P API, or via the Developer Mode "Configure third-party inference" panel?** Different code paths, different bug classes.
- **Did Cornell push the AWS Cowork-on-Bedrock managed config via Jamf, or is the 3P deployment a Cornell-specific gateway config that doesn't match the AWS-blessed pattern?** Cornell using the AWS pattern would unlock the documented Code-tab-in-Cowork-mode path. Cornell using their own gateway pattern leaves Pete in the un-officially-supported zone.
- **Has Anthropic privately documented `isCodeTabEnabledFor3PMode` or similar policy keys for enterprise admins?** Pete's [#59407](https://github.com/anthropics/claude-code/issues/59407) asks the public question; the public docs don't list one. Possibly available via Anthropic's enterprise support channel only.
- **Does Claude Code's built-in `WebFetch` tool fire on a Bedrock-backed session?** Architectural inference says yes (it's a client-side fetch + a separate Haiku-API summarization call); not explicitly documented. Pete should run a single test invocation against a known URL on his first Bedrock CLI session and report back.
- **Does the Mantle endpoint expose Anthropic's `web_search` server tool?** Mantle removes the wire-format objection (it speaks Anthropic Messages), but Anthropic hasn't published a per-tool support matrix for Mantle. A direct probe with `CLAUDE_CODE_USE_MANTLE=1` and a `web_search` request would settle this.
- **Which exact Anthropic pre-built skills in the `anthropics/skills` repo are pure-bash (and therefore work via marketplace install on Bedrock) vs require hosted code execution (and therefore fail at runtime)?** The repo doesn't tag them. The `pdf`/`xlsx`/`docx`/`pptx` document-skills almost certainly need hosted code execution; the example-skills and creative skills probably don't. Pete could test by installing `example-skills@anthropic-agent-skills` and probing.

## Sources

### Official Anthropic / AWS docs

- [Claude Code: Desktop application](https://code.claude.com/docs/en/desktop) — primary source for Code tab feature comparison, "What's not available in Desktop", enterprise configuration, MDM policy list. Contains the verbatim "For Bedrock or Foundry, use the CLI" sentence.
- [Claude Code on Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock) — CLI Bedrock setup, IdC SSO via `Option C: Environment variables (SSO profile)`, TLS-inspection-proxy auth-loop workaround, `awsAuthRefresh` / `awsCredentialExport`, Mantle endpoint (Anthropic-shape over Bedrock auth).
- [Claude Code: Third-party integrations overview](https://code.claude.com/docs/en/third-party-integrations) — feature comparison across Bedrock / Vertex / Foundry / Claude Platform on AWS / Teams/Enterprise; LLM gateway env vars (`ANTHROPIC_BEDROCK_BASE_URL`, `CLAUDE_CODE_SKIP_BEDROCK_AUTH`).
- [Claude Code: LLM gateway configuration](https://code.claude.com/docs/en/llm-gateway) — gateway API-format requirements (Anthropic Messages / Bedrock InvokeModel / Vertex rawPredict), LiteLLM setup, `ANTHROPIC_AUTH_TOKEN`, gateway model discovery.
- [Claude Code: Enterprise network configuration](https://code.claude.com/docs/en/network-config) — proxy, custom CA, mTLS, allowlists.
- [AWS Machine Learning Blog: From developer desks to the whole organization: Running Claude Cowork in Amazon Bedrock](https://aws.amazon.com/blogs/machine-learning/from-developer-desks-to-the-whole-organization-running-claude-cowork-in-amazon-bedrock/) — primary source for what's in/out of the Cowork-on-Bedrock product, MDM deployment, gateway support.
- [AWS Machine Learning Blog: Accelerate AI development with Amazon Bedrock API keys](https://aws.amazon.com/blogs/machine-learning/accelerate-ai-development-with-amazon-bedrock-api-keys/) — bearer-token vs SigV4 distinction, `AWS_BEARER_TOKEN_BEDROCK`.
- [aws-solutions-library-samples/guidance-for-claude-code-with-amazon-bedrock (GitHub)](https://github.com/aws-solutions-library-samples/guidance-for-claude-code-with-amazon-bedrock) — `ccwb` CLI, IdC as first-class auth in v2.2+.

### Most-relevant GitHub issues (anthropics/claude-code)

- [#35070](https://github.com/anthropics/claude-code/issues/35070) (CLOSED, not planned) — "Code tab in Claude desktop app should support independent Bedrock authentication (like VS Code extension)." Root cause: cached OAuth token in `config.json` overrides `~/.claude/settings.json` `CLAUDE_CODE_USE_BEDROCK=1`. **Most-likely match for Pete's symptom**.
- [#61851](https://github.com/anthropics/claude-code/issues/61851) (OPEN, 2026-05-23) — "Code tab forces enterprise AWS SSO on personal Pro account, ignores existing app session." Root cause: Code tab independently scans `~/.aws/config` and triggers SSO, ignoring `CLAUDE_CONFIG_DIR` and the active app session. **Second-most-likely match**.
- [#59407](https://github.com/anthropics/claude-code/issues/59407) (OPEN, 2026-05-15) — **Filed by Pete (@pete-builds)**. "Chat tab disappears when switching from Anthropic login to third-party API gateway." Sibling bug to the spinning Code tab. Cited as evidence of the pattern.
- [#56182](https://github.com/anthropics/claude-code/issues/56182) (OPEN, stale, 2026-05-05) — "Code mode hangs silently on all models and sessions on Windows desktop app after May 4 Opus 4.7 incidents." Even on Anthropic-direct, the Code mode can hang silently. Demonstrates that "spinning Code tab" is a known class of Desktop bug independent of 3P mode.
- [#52526](https://github.com/anthropics/claude-code/issues/52526) (OPEN, stale, 2026-04-23) — "Cowork 3P Gateway — Model picker ignores version and display name." Confirms that the Developer Mode "Configure third-party inference" flow exists, and that 3P settings don't propagate uniformly across Cowork and Code tabs.
- [#48310](https://github.com/anthropics/claude-code/issues/48310), [#48676](https://github.com/anthropics/claude-code/issues/48676), [#48407](https://github.com/anthropics/claude-code/issues/48407) — Class of "tabs disappear after auto-update" Desktop bugs around v1.2581.0; not directly Pete's issue but illustrates that the Desktop app's per-tab visibility logic is fragile and bug-prone.
- [#33322](https://github.com/anthropics/claude-code/issues/33322), [#32668](https://github.com/anthropics/claude-code/issues/32668) (both CLOSED as `invalid`) — Two separate feature requests asking for Claude Desktop to natively support Bedrock. Both closed as not in scope for the claude-code repo, with no real explanation. Useful as evidence that Anthropic does not consider Desktop-on-Bedrock-direct a roadmap item.
- [#53498](https://github.com/anthropics/claude-code/issues/53498) (OPEN, stale) — "Organization plugins not available in Desktop Code tab." Another per-tab feature-gap issue.
- [#36362](https://github.com/anthropics/claude-code/issues/36362) (CLOSED, not planned) — "Code mode in Claude Desktop app does not respond to messages. No loading indicator appears after sending." Demonstrates the broader class of unresolved Code-tab unresponsiveness bugs, though that specific report was a git auth issue not Bedrock.
- [#58666](https://github.com/anthropics/claude-code/issues/58666) (OPEN) — "Claude Desktop Windows 11 Cowork and Code tabs give this error always now: Claude Code process exited with code 3." User on Anthropic-direct, not Bedrock, but illustrates that Desktop's Code-tab subprocess pipeline has its own stability issues.

### Community walkthroughs and operator guides

- [Elevata: Claude AI Model Deployment on Amazon Bedrock: Desktop & Code Requirements](https://elevata.io/en/claude-code-on-aws-complete-guide-bedrock-setup-self-hosted-models) — clearest published distinction between "Claude Code on Bedrock via CLI" and "Claude Desktop 3P with Bedrock." Confirms the Code-tab-parity gap.
- [Elevata: What requirements need to be ready for Claude Code on Bedrock?](https://elevata.io/en/claude-code-on-bedrock) — Research-Preview / 3P / Code-tab validation list. Source of "validate availability, MDM, credentials, egress, local files, plugins, MCP, observability, and the Code tab separately for any team pilot."
- [DEV.to (Haowen Huang): Running Claude Code and Claude Desktop on Amazon Bedrock](https://dev.to/haowen_huang/running-claude-code-and-claude-desktop-on-amazon-bedrock-1ed2) — community walkthrough; useful for CLI setup pattern, less authoritative on Desktop-side.

### Skills, web search, and Bedrock parity (added 2026-05-27 5:38 PM ET)

Sources for [Section 8](#8-adding-skills-and-web-search-on-bedrock-the-cli-recipe).

- [Anthropic Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) — primary source on Custom vs pre-built skills, surface support matrix (claude.ai / Claude API / Claude Platform on AWS / Microsoft Foundry — note Bedrock-direct is NOT on this list for pre-built), Skills API beta headers, and progressive disclosure architecture.
- [Anthropic Skills in Claude Code](https://code.claude.com/docs/en/skills) — primary source on filesystem-based custom skills, bundled skills (`/code-review`, `/debug`, `/loop`, `/run`, `/verify`), SKILL.md frontmatter reference, and skill discovery rules.
- [Anthropic web search tool docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool) — verbatim "Web search is not available on Amazon Bedrock."
- [Anthropic web fetch tool docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-fetch-tool) — verbatim "It is not currently available on Amazon Bedrock or Vertex AI."
- [Claude in Amazon Bedrock (Anthropic docs)](https://platform.claude.com/docs/en/build-with-claude/claude-in-amazon-bedrock) — current Bedrock + Mantle endpoint architecture, auth paths (Bedrock service role / IAM role / Bedrock API key), model IDs, regions.
- [Claude Code on Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock) — `CLAUDE_CODE_USE_MANTLE` env var, Mantle endpoint behavior, SSO loop workaround, region/model considerations.
- [Mikhail Shilkov: Inside Claude Code's Web Tools: WebFetch vs WebSearch](https://mikhail.io/2025/10/claude-code-web-tools/) — runtime reverse-engineering. Source of "Claude Code hides the `WebSearch` tool entirely" on Bedrock/Vertex; also documents WebFetch's domain-allow-list + Haiku-summarize architecture.
- [AWS re:Post: Web Search for Anthropic Models in Bedrock](https://repost.aws/questions/QUSd3wAByQTtyzUPzgqss3TQ/web-search-for-anthropic-models-in-bedrock) — AWS Official answer confirming `web_search` is not on Bedrock; exact ValidationException error from reproducer.
- [Tech-Reader: Solve ValidationException web_search Not Supported in Bedrock Claude Tool Calls](https://www.tech-reader.blog/2025/05/solve-validationexception-websearch-not.html) — operator-level confirmation of Bedrock blocking `web_search_20250305` and `code_execution_20250305`.
- [anthropics/skills GitHub repo](https://github.com/anthropics/skills) — Anthropic-hosted Agent Skills marketplace; `/plugin marketplace add anthropics/skills` install path; document-skills (`docx`, `pdf`, `pptx`, `xlsx`) and example-skills.
- [Gunnar Grosch (DEV.to): Extending Claude Code with Plugins and Skills for AWS Development](https://dev.to/gunnargrosch/extending-claude-code-with-plugins-and-skills-for-aws-development-4p9o) — primary operator-side confirmation that custom skills, plugins, MCP servers, hooks, agents all work on Bedrock-backed Claude Code, with worked examples (deploy, security-review, public-api-standards) and frontmatter reference.
- [Haowen Huang (DEV.to): Running Claude Agent SDK with Skills on Amazon Bedrock](https://dev.to/haowen_huang/running-claude-agent-sdk-with-skills-on-amazon-bedrock-el6) — independent confirmation: "Everything from the original SDK works: Skills, Subagents with parallel execution, MCP servers (Notion, etc.), All built-in tools (WebSearch, Bash, Write, etc.)."
- [LiteLLM: Claude Code WebSearch Across All Providers](https://docs.litellm.ai/docs/tutorials/claude_code_websearch) — LiteLLM's `websearch_interception` feature for translating Claude Code's `web_search` requests to Perplexity/Tavily/SearXNG/etc. before they hit Bedrock.
- [LiteLLM Issue #26252: websearch_interception does not fire for Claude Code + Bedrock pass-through](https://github.com/BerriAI/litellm/issues/26252) — open bug confirming the LiteLLM bridge has shipped buggy and is mid-fix. Independent confirmation that Bedrock cannot handle `web_search_20250305` on any API path: "tools.0: Input tag 'web_search_20250305' found using 'type' does not match any of the expected tags: 'bash_20250124', 'custom', 'text_editor_20250124'".
- [antkawam/claude-code-aws-gateway (CCAG)](https://github.com/antkawam/claude-code-aws-gateway) — purpose-built self-hosted gateway. Source of the explicit quote: "When Claude Code connects to Bedrock directly (`CLAUDE_CODE_USE_BEDROCK=1`), it operates in a reduced-capability mode — extended thinking, web search, and some tool use features are disabled on the client side."
- [DEV.to (Gabriel Koo): Bedrock for AI Coding Tools — Mantle vs Gateway vs LiteLLM](https://dev.to/aws-builders/bedrock-for-ai-coding-tools-mantle-vs-gateway-vs-litellm-a-decision-guide-for-aws-credit-burners-1h01) — best decision guide for the three full-bridge architectures.
- [Claude Code MCP docs](https://code.claude.com/docs/en/mcp) — MCP server configuration; `streamable-http` as alias for `http` transport in `.mcp.json` / `~/.claude.json`.
- [pete-builds/mcp-searxng](https://github.com/pete-builds/mcp-searxng) — Pete's own SearXNG MCP. The recommended web-search bridge for his Bedrock CLI setup.
- [brave/brave-search-mcp-server](https://github.com/brave/brave-search-mcp-server) — official Brave Search MCP server. Alternative MCP option.
- [Tavily MCP Server docs](https://docs.tavily.com/documentation/mcp) — Tavily's MCP integration. Alternative MCP option.
- [Exa MCP](https://exa.ai/docs/reference/exa-mcp) — Exa's MCP integration. Alternative MCP option.
- [Claude Code Issue #11369: web search not working (Did 0 searches in 7s)](https://github.com/anthropics/claude-code/issues/11369) — closed-as-inactive, but the comment thread documents the same WebSearch-on-Bedrock failure across multiple users (Bedrock direct, LiteLLM-to-Bedrock, Vertex, MiniMax) and a community-found explanation that the model vendor needs to provide an MCP replacement.
- [Claude Code Issue #32385: WebSearch, Token Counting API unavailable on bedrock-connected billing accounts](https://github.com/anthropics/claude-code/issues/32385) — closed as duplicate of #11369 but useful as an explicit "WebSearch and Token Counting are Anthropic-direct-only" framing.

## Update History

- **2026-05-27 5:54 PM ET**: Verify-pass cleanup before public publish. (a) Section 8.1 now directly addresses the apparent contradiction with the official Anthropic Bedrock page's "Features not supported: Agent infrastructure (Agent Skills, MCP connector, programmatic tool calling)" line — added a paragraph explaining that bullet refers to API-uploaded Skills via the Skills API + Files API + hosted code-execution container, not Claude Code CLI filesystem skills, which is why the two mechanisms have opposite Bedrock-compatibility outcomes despite sharing a name. (b) Section 8.2's `WebFetch` paragraph reworded to accurately reflect Mikhail Shilkov's source: the pipeline is Anthropic-backend-operated (`domain_info` endpoint + HTML-to-Markdown + Haiku 3.5 summarization), not "purely client-side" — the architectural reason WebFetch is not gated by the Bedrock setting is that its inference doesn't depend on the configured backend model, not that the fetch is local.
- **2026-05-27 5:38 PM ET**: Appended [Section 8: Adding Skills and web search on Bedrock (the CLI recipe)](#8-adding-skills-and-web-search-on-bedrock-the-cli-recipe) in response to Pete's follow-up question on how to enable Agent Skills and web search once he falls back to the Claude Code CLI on Bedrock per Section 5 Option A. Findings: (1) **custom skills work natively** on Bedrock-backed Claude Code because they're filesystem + system-prompt; (2) the **plugin marketplace install path works**, but Anthropic's pre-built skills that depend on hosted code-execution / Files API do not run on Bedrock; (3) Anthropic's **`web_search` and `web_fetch` server tools are blocked** on Bedrock; (4) Claude Code's built-in `WebSearch` is therefore **silently hidden** on Bedrock-backed sessions (per Mikhail Shilkov's runtime analysis); (5) the recommended bridge is Pete's existing **`pete-builds/mcp-searxng`** server in his `~/.claude/settings.json` `mcpServers` block; (6) the per-feature matrix in Section 8.4 settles each capability for Desktop Code tab vs CLI on Bedrock; (7) the Mantle endpoint and full-bridge gateways (LiteLLM, CCAG) exist as escalation paths if Cornell ever wants Anthropic-feature-parity for Bedrock-backed developers, but are not needed for Pete personally. Confidence Assessment extended with nine new rows. Sources section gained a dedicated subsection ("Skills, web search, and Bedrock parity") with 19 new sources.
- **2026-05-27 4:50 PM ET**: Fresh report after scrapping a prior wrong-scope research run that focused on the VS Code extension. The earlier draft is deleted. This report is scoped specifically to the **Claude Desktop app's Code tab** (not the VS Code extension, not the CLI) in **3P API mode pointing at AWS Bedrock** with **IdC SSO**. Primary new findings: (1) Anthropic explicitly documents that the Desktop Code tab does not support Bedrock; (2) Pete's own filed bug [#59407](https://github.com/anthropics/claude-code/issues/59407) is part of a documented class of Desktop-3P-mode tab-gating bugs; (3) the recommended path is Claude Code CLI on Bedrock, or routing the CLI through the Cornell AI Gateway in Anthropic-Messages mode.

## How This Report Was Generated

- **Skill**: Research (Claude Code), Opus 4.7 model.
- **Searches**: SearXNG MCP (`search_deep` and `search_tech`) across Bing, Brave, DuckDuckGo, and technical engines. Specific search terms emphasized **"Claude Desktop"** (the standalone app), the **Code tab inside it**, **third-party API / 3P / gateway** mode, and **AWS Bedrock + IdC SSO**. Search results were heavily filtered to exclude wrong-scope material about the Claude Code CLI or the VS Code extension.
- **Documents fetched and read in full**: Anthropic Desktop docs page; Anthropic Bedrock docs page; LLM gateway docs page; third-party integrations overview; network configuration page; AWS ML blog on Claude Cowork on Bedrock; AWS ML blog on Bedrock API keys; Elevata operator guides on Desktop 3P + Bedrock.
- **GitHub issue mining**: `gh issue list --repo anthropics/claude-code --search` with three different searches (Code tab + Bedrock, Desktop + third-party + Code tab, "Code tab" + spinning/hanging). Full bodies + key comments read via `gh issue view --json` and WebFetch. ~15 issues directly relevant, of which 9 are cited.
- **Cross-validation**: Multiple sources independently confirm the central finding (Code tab doesn't support Bedrock-direct). Anthropic's own docs are the primary source. GitHub issues #35070, #61851, #59407, #56182, and #52526 each provide an independent angle on why the configuration breaks. AWS ML blog confirms what the AWS-blessed alternative includes and excludes. Elevata's operator guides corroborate the Cowork-vs-Code parity gap.
- **What was deliberately excluded**: All findings from the previous (wrong-scope) report that focused on the VS Code extension's `bash -lic` PATH probe, the VS Code extension's config-probe 60s timeout, and any VS Code-extension-specific advice. Those were valid for that scope but irrelevant to Pete's actual question.
- **Sources NOT fetched**: Reddit (Claude Code's WebFetch blocks www.reddit.com). Two Reddit threads (`r/ClaudeAI` "Can Claude Desktop chat/cowork/code be configured to route through a custom gateway to AWS Bedrock?" and "Claude Desktop App BedRock Integration") showed in search results but couldn't be read. Did not materially affect findings because the underlying questions are resolved by official Anthropic docs and GitHub-issue evidence.
- **Timestamps**: System clock is UTC. Current ET verified via `TZ='America/New_York' date` → 2026-05-27 4:50 PM EDT (UTC-4) for the original report, and 2026-05-27 5:38 PM EDT for the Section 8 append. All timestamps in this report converted before labeling.
- **Section 8 research pass** (added 2026-05-27 5:38 PM ET): Same Research skill, Opus 4.7. Six SearXNG `search_deep` calls (Skills + Bedrock, web_search + Bedrock, WebSearch/WebFetch built-in tools, MCP server search, Mantle endpoint, bundled skills). Twelve `read_url` fetches of primary sources (Anthropic Agent Skills overview, Claude Code skills docs, web_search/web_fetch tool docs, Claude in Amazon Bedrock, Claude Code on Amazon Bedrock, Mikhail Shilkov runtime analysis, AWS re:Post, tech-reader operator post, anthropics/skills GitHub README, antkawam/claude-code-aws-gateway README, Gunnar Grosch DEV.to plugins/skills post, Haowen Huang DEV.to Bedrock-SDK post, LiteLLM Claude Code WebSearch tutorial, LiteLLM Issue #26252, Bedrock decision-guide post). Two GitHub issues fetched via `gh issue view`: #32385 and #11369. Anchored every Section 8 claim to a fetched-and-quoted source; flagged the WebFetch-on-Bedrock claim as Medium confidence because it's architectural inference, not explicit doc statement.
- **Generated**: 2026-05-27, America/New_York timezone.
