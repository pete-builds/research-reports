---
title: "Ways to Distribute Claude Agent Skills to End Users"
date: 2026-05-29
updated: 2026-05-29T14:58:00-04:00
summary: "A landscape reference covering every official and adjacent mechanism for packaging and delivering Anthropic Agent Skills (SKILL.md) to end users: Claude apps, Claude Code, the Agent SDK, the Claude Developer Platform/API, plugins/marketplaces, manual/MDM distribution, and what a model gateway like LiteLLM can and cannot do."
---

## TL;DR

Agent Skills are SKILL.md-based folders of instructions, scripts, and resources that Claude loads on demand. You can get them to end users through five primary channels: upload in the claude.ai/Claude Desktop UI (per-user), drop them on the filesystem for Claude Code and the Agent SDK (`~/.claude/skills/`, `.claude/skills/`), upload them to the Claude API via the `/v1/skills` endpoints (workspace-wide), or package and share them as Claude Code plugins through marketplaces (team or community). Manual methods (git, zip, install scripts, MDM/config-management push of skill folders) underpin most filesystem-based distribution. A model-routing gateway like LiteLLM does not host or distribute skills: it proxies the API traffic and can pass through Anthropic's Skills API call, but the skill content lives client-side or in Anthropic's container, not in the gateway ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)) ([source](https://docs.litellm.ai/docs/providers/anthropic)).

## Current Status

- Agent Skills are available across Claude's agent products: claude.ai, Claude Desktop, Claude Code, the Claude Developer Platform/API, the Claude Agent SDK, and the managed surfaces Claude Platform on AWS and Microsoft Foundry (which inherit API behavior) ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).
- Anthropic ships pre-built skills (PowerPoint/pptx, Excel/xlsx, Word/docx, PDF/pdf) plus the ability to author custom skills; both work identically and Claude invokes them automatically when relevant ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).
- The Claude API Skills feature requires three beta headers (`code-execution-2025-08-25`, `skills-2025-10-02`, `files-api-2025-04-14`) and runs inside the code execution container, indicating the API path is still gated behind beta headers as of this report ([source](https://platform.claude.com/docs/en/build-with-claude/skills-guide)).
- SKILL.md is published as an open standard at agentskills.io (originally developed by Anthropic, released openly) and has been adopted by a large ecosystem of third-party agents (Cursor, GitHub Copilot/VS Code, Gemini CLI, OpenAI Codex, Goose, OpenHands, and many more) ([source](https://agentskills.io)) ([source](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)).
- Custom skills do not sync across surfaces: a skill uploaded to claude.ai is not available via the API, and Claude Code's filesystem skills are separate from both. You manage and upload skills per surface ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).
- On claude.ai there is no centralized admin management or org-wide distribution of custom skills today; custom skills are individual to each user. Centralized/org-wide distribution exists in Claude Code via managed settings and plugins, and in the API via workspace-wide sharing ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)) ([source](https://code.claude.com/docs/en/skills)).

## Table of Contents

- [TL;DR](#tldr)
- [Current Status](#current-status)
- [Findings](#findings)
  - [1. What Agent Skills Are (brief grounding)](#1-what-agent-skills-are-brief-grounding)
  - [2. Distribution via Claude.ai / Claude Desktop App (incl. Team/Enterprise admin-managed)](#2-distribution-via-claudeai--claude-desktop-app-incl-teamenterprise-admin-managed)
  - [3. Distribution via Claude Code (personal/project skills, plugins, marketplaces)](#3-distribution-via-claude-code-personalproject-skills-plugins-marketplaces)
  - [4. Distribution via the Agent SDK](#4-distribution-via-the-agent-sdk)
  - [5. Distribution via the Claude Developer Platform / API](#5-distribution-via-the-claude-developer-platform--api)
  - [6. Skills vs MCP (and whether MCP delivers skills)](#6-skills-vs-mcp-and-whether-mcp-delivers-skills)
  - [7. Gateways (LiteLLM etc.): What They Can and Can't Do for Skills](#7-gateways-litellm-etc-what-they-can-and-cant-do-for-skills)
  - [8. Manual / Script-Based Distribution (git, zip, install scripts, MDM push)](#8-manual--script-based-distribution-git-zip-install-scripts-mdm-push)
  - [9. Comparison Table (method x audience x how it reaches the user x trade-offs)](#9-comparison-table-method-x-audience-x-how-it-reaches-the-user-x-trade-offs)
- [Confidence Assessment](#confidence-assessment)
  - [High Confidence](#high-confidence)
  - [Medium Confidence](#medium-confidence)
- [Open Questions](#open-questions)
- [Sources](#sources)
- [Update History](#update-history)
- [How This Report Was Generated](#how-this-report-was-generated)

## Findings

### 1. What Agent Skills Are (brief grounding)

An Agent Skill is "a folder containing a `SKILL.md` file" with YAML frontmatter (at minimum `name` and `description`) plus markdown instructions, and optionally bundled scripts, references, and assets ([source](https://agentskills.io)). The frontmatter `name` must be lowercase letters/numbers/hyphens, max 64 chars, no XML tags, and cannot use reserved words "anthropic" or "claude"; `description` is non-empty, max 1024 chars ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).

Skills use **progressive disclosure** across three levels: Level 1 metadata (`name`/`description`, ~100 tokens, always loaded into the system prompt); Level 2 the SKILL.md body (loaded when the skill is triggered); Level 3+ bundled resources and scripts (read or executed via bash only when needed, so they cost zero tokens until used) ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)). This architecture lets you install many skills cheaply and is what makes filesystem-based distribution efficient ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).

Anthropic released SKILL.md as an open standard (agentskills.io) for cross-platform portability, with the engineering blog noting skills "complement Model Context Protocol (MCP) servers by teaching agents more complex workflows that involve external tools and software" (post dated October 16, 2025, updated December 18, 2025) ([source](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)). Confidence: High.

### 2. Distribution via Claude.ai / Claude Desktop App (incl. Team/Enterprise admin-managed)

**What it is:** The consumer/web and desktop Claude apps support both pre-built skills (working automatically behind the scenes when you create documents) and custom skills you upload yourself ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).

**How a skill reaches the user:** A user uploads their own skill as a zip file through **Settings > Features** in claude.ai. This is available on Pro, Max, Team, and Enterprise plans with code execution enabled ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)). The anthropics/skills GitHub repo points claude.ai users to the Claude Help Center "Using skills in Claude" article to upload skills ([source](https://github.com/anthropics/skills)).

**Who it's for:** Individual end users on the hosted Claude product who want skills in chat without touching the filesystem or API.

**Trade-offs / limits:**
- Custom skills on claude.ai are **individual to each user**: not shared organization-wide and cannot be centrally managed by admins. Each team member must upload separately ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).
- Skills uploaded to claude.ai do **not** transfer to the API or to Claude Code ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).
- Runtime network access varies by user/admin settings (full, partial, or none) ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).
- The Skills feature is not eligible for Zero Data Retention ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).

Note on "Claude Desktop": the overview groups the hosted Claude products together. The Gemini-grounded summary stated Claude Desktop offers pre-built skills and a skill-creator/upload path; the authoritative Anthropic overview documents the upload flow specifically for claude.ai (Settings > Features). The desktop app shares the claude.ai account/skill model. Confidence: High for claude.ai specifics; Medium that the desktop app exposes the identical Settings > Features upload path (documented under claude.ai, and the desktop app is the same product surface).

### 3. Distribution via Claude Code (personal/project skills, plugins, marketplaces)

Claude Code supports **only custom skills** (no pre-built document skills), and they are filesystem-based and discovered automatically with no API upload ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)). Claude Code skills follow the agentskills.io open standard and extend it with invocation control, subagent execution, and dynamic context injection ([source](https://code.claude.com/docs/en/skills)).

**Where skills live (four scopes), and who each reaches** ([source](https://code.claude.com/docs/en/skills)):
- **Enterprise / Managed**: deployed organization-wide through managed settings. Applies to all users in the org.
- **Personal**: `~/.claude/skills/<skill-name>/SKILL.md`. Applies across all the user's projects.
- **Project**: `.claude/skills/<skill-name>/SKILL.md`. Applies to that project only; commit it to version control to share with collaborators.
- **Plugin**: `<plugin>/skills/<skill-name>/SKILL.md`. Applies wherever the plugin is enabled.

When names collide across levels, enterprise overrides personal, and personal overrides project; plugin skills are namespaced (`plugin-name:skill-name`) so they can't conflict ([source](https://code.claude.com/docs/en/skills)). Project skills also load from `.claude/skills/` in every parent directory up to the repo root and from nested directories on demand (monorepo support), and `.claude/skills/` inside an `--add-dir` directory is loaded automatically ([source](https://code.claude.com/docs/en/skills)).

**Three ways the docs frame sharing in Claude Code:** commit project skills to version control; create a `skills/` directory in a plugin; or deploy org-wide through managed settings ([source](https://code.claude.com/docs/en/skills)).

**Plugins as a distribution vehicle.** A plugin is a directory with a `.claude-plugin/plugin.json` manifest plus a `skills/` directory (each skill a `<name>/SKILL.md` folder) ([source](https://code.claude.com/docs/en/plugins)). Plugins are the recommended packaging unit for sharing skills with a team or community, give versioned releases, and yield namespaced skill names like `/my-plugin:hello` ([source](https://code.claude.com/docs/en/plugins)). You can test a plugin locally with `claude --plugin-dir ./my-plugin` (also accepts a `.zip`), or fetch a hosted zip with `claude --plugin-url https://example.com/my-plugin.zip` ([source](https://code.claude.com/docs/en/plugins)).

**Marketplaces.** A marketplace is a catalog (a repo or hosted file containing `.claude-plugin/marketplace.json`) that users add and then install plugins from ([source](https://code.claude.com/docs/en/discover-plugins)). Mechanisms:
- Official marketplace `claude-plugins-official` is available automatically; install with `/plugin install <name>@claude-plugins-official` ([source](https://code.claude.com/docs/en/discover-plugins)).
- Community marketplace: `/plugin marketplace add anthropics/claude-plugins-community`, then `/plugin install <plugin-name>@claude-community` ([source](https://code.claude.com/docs/en/discover-plugins)).
- Add any source: GitHub `owner/repo`, any git URL (GitLab/Bitbucket/self-hosted, with optional `#ref`), local paths, or a remote `marketplace.json` URL ([source](https://code.claude.com/docs/en/discover-plugins)).
- Private/internal distribution: host the marketplace in a private repository to keep it team-internal ([source](https://code.claude.com/docs/en/plugins)).
- Team auto-install: admins add `extraKnownMarketplaces` (and `enabledPlugins`) to a project's `.claude/settings.json`; on folder trust, members are prompted to install ([source](https://code.claude.com/docs/en/discover-plugins)).
- Install scopes: user, project (`.claude/settings.json`, shared with collaborators), local (this repo, not shared), or managed (admin-installed, not modifiable). After install, run `/reload-plugins` ([source](https://code.claude.com/docs/en/discover-plugins)).
- Anthropic's own skills repo doubles as a marketplace: `/plugin marketplace add anthropics/skills` then `/plugin install document-skills@anthropic-agent-skills` ([source](https://github.com/anthropics/skills)).

**Who it's for:** Developers and engineering teams; the plugin/marketplace path scales to org-wide and community distribution.

**Trade-offs / limits:** Plugin skills are always namespaced (longer invocation names) ([source](https://code.claude.com/docs/en/plugins)). Plugins execute arbitrary code with user privileges, so only trusted sources should be installed; orgs can restrict allowed marketplaces via managed settings ([source](https://code.claude.com/docs/en/discover-plugins)). Claude Code skills have full network access (same as any program on the user's machine), and the docs discourage global package installation ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)). Confidence: High.

### 4. Distribution via the Agent SDK

In the Claude Agent SDK, skills are **filesystem artifacts only**: there is no programmatic API to register a skill (unlike subagents, which can be defined in code) ([source](https://code.claude.com/docs/en/agent-sdk/skills)). You package skills with your SDK app by shipping `.claude/skills/<name>/SKILL.md` directories alongside it ([source](https://code.claude.com/docs/en/agent-sdk/skills)).

**How a skill reaches the runtime:**
- Skills are discovered through `settingSources` (TypeScript) / `setting_sources` (Python). With default options the SDK loads `user` and `project` sources, so `~/.claude/skills/`, `<cwd>/.claude/skills/`, and `.claude/skills/` in parent dirs up to the repo root are available. If you set `settingSources` explicitly you must include `'user'` or `'project'` to keep skill discovery (or load from a path via the `plugins` option) ([source](https://code.claude.com/docs/en/agent-sdk/skills)).
- The `skills` option filters which discovered skills are active: omit it (default on), pass `"all"`, pass a list of names (use `plugin:skill` for plugin skills), or pass `[]` to disable all. Setting `skills` auto-enables the Skill tool ([source](https://code.claude.com/docs/en/agent-sdk/skills)).
- Plugins can supply skills to an SDK app via the `plugins` option/path ([source](https://code.claude.com/docs/en/agent-sdk/skills)).

**Who it's for:** Developers embedding Claude in their own applications/agents and wanting to bundle domain skills with the deployed app.

**Trade-offs / limits:** No programmatic skill registration; you must place files on disk ([source](https://code.claude.com/docs/en/agent-sdk/skills)). The `skills` option is a context filter, not a sandbox: unlisted skills are hidden from the model but their files remain readable via Read/Bash ([source](https://code.claude.com/docs/en/agent-sdk/skills)). The SKILL.md `allowed-tools` frontmatter is ignored under the SDK; control tools via the main `allowedTools` option instead ([source](https://code.claude.com/docs/en/agent-sdk/skills)). Confidence: High.

### 5. Distribution via the Claude Developer Platform / API

The Claude API supports both pre-built and custom skills; both work identically by specifying a `skill_id` in the `container` parameter alongside the code execution tool (up to 8 skills per request) ([source](https://platform.claude.com/docs/en/build-with-claude/skills-guide)) ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).

**Prerequisites / how it reaches users:**
- Three beta headers: `code-execution-2025-08-25`, `skills-2025-10-02`, `files-api-2025-04-14` ([source](https://platform.claude.com/docs/en/build-with-claude/skills-guide)).
- Pre-built skills are referenced by short id (`pptx`, `xlsx`, `docx`, `pdf`) with type `anthropic`; custom skills get a generated id like `skill_01AbCdEf...` with type `custom` ([source](https://platform.claude.com/docs/en/build-with-claude/skills-guide)).
- **Skills Management endpoints** (the `/v1/skills` CRUD surface): `POST /v1/skills` (create, upload SKILL.md + files, total < 30 MB), `GET /v1/skills` (list, with `?source=custom` filter), `GET /v1/skills/{skill_id}` (retrieve), `GET/POST /v1/skills/{skill_id}/versions` and `DELETE /v1/skills/{skill_id}/versions/{version}` (version management), `DELETE /v1/skills/{skill_id}` (delete; must delete all versions first) ([source](https://platform.claude.com/docs/en/build-with-claude/skills-guide)).
- Custom skills uploaded via the API are **shared workspace-wide**: all workspace members can access them ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)) ([source](https://platform.claude.com/docs/en/build-with-claude/skills-guide)).
- Claude Platform on AWS and Microsoft Foundry inherit this behavior; upload custom skills to those through the Skills API ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).

**Who it's for:** Application developers and platform teams who want skills available to every member of an API workspace and versioned via API.

**Trade-offs / limits:** API skills run in Anthropic's code execution container with **no network access**, **no runtime package installation**, and only pre-configured dependencies ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)). API skills are not visible on claude.ai and vice versa (no cross-surface sync) ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)). Feature is gated behind beta headers and not ZDR-eligible ([source](https://platform.claude.com/docs/en/build-with-claude/skills-guide)) ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)). Confidence: High.

### 6. Skills vs MCP (and whether MCP delivers skills)

Skills and MCP solve different problems. MCP is a protocol for connecting agents to external tools/data sources (servers exposing tools, resources, prompts). Skills are filesystem folders of instructions/scripts that teach an agent how to do a task. Anthropic frames them as **complementary**: skills "complement Model Context Protocol (MCP) servers by teaching agents more complex workflows that involve external tools and software" ([source](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)).

Does MCP deliver skills? There is no documented mechanism in the official Anthropic docs by which an MCP server packages and installs a SKILL.md into a client as a skill. The Claude Code docs reference skills "provided by an MCP server" only in the narrow sense that the `skillOverrides` setting can adjust visibility of such skills, implying an MCP server can surface skill-like entries that Claude Code lists ([source](https://code.claude.com/docs/en/skills)). Beyond that, the canonical distribution channels for skills are filesystem (Claude Code / SDK), the claude.ai upload UI, the API `/v1/skills` endpoints, and plugins/marketplaces, not MCP. Plugins can bundle both skills and MCP server configs together as one installable unit, which is the practical way the two coexist in a distribution ([source](https://code.claude.com/docs/en/plugins)). Confidence: High for the complementary framing and plugin co-bundling; Medium on the exact mechanics of an MCP server exposing a skill (docs mention it only via `skillOverrides`).

### 7. Gateways (LiteLLM etc.): What They Can and Can't Do for Skills

This is the point most worth getting right. A model-routing gateway like LiteLLM is a **proxy for model API calls**, not a skill host or distribution registry.

**What a gateway does NOT do:**
- It does not store, version, or distribute SKILL.md content to end users. The official surfaces for that are claude.ai uploads, the filesystem (Claude Code/SDK), the API `/v1/skills` store, and plugins/marketplaces ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).
- LiteLLM's general proxy docs describe it as mapping inputs/outputs to a common format and routing requests across providers; nothing in the core proxy behavior makes it a skill repository ([source](https://docs.litellm.ai/docs/anthropic_unified)).

**What a gateway CAN do:**
- It routes the model traffic that a skill-enabled client generates. When a Claude Code session, SDK app, or API caller invokes a skill, the underlying Messages API request still flows to the model endpoint; a gateway can sit in that path and route/load-balance it ([source](https://docs.litellm.ai/docs/anthropic_unified)).
- LiteLLM specifically documents support for using Agent Skills with the API by passing the `container` object through to Claude; the container and its id appear in `provider_specific_fields` in the response, which confirms LiteLLM **routes the request to Anthropic** rather than hosting the skill. The skill itself resolves in Anthropic's container, not in LiteLLM ([source](https://docs.litellm.ai/docs/providers/anthropic)).
- LiteLLM's docs navigation lists a dedicated "/skills - Anthropic Skills API" page, indicating the proxy offers a passthrough to Anthropic's Skills API so a client can manage Anthropic-hosted skills *through* the proxy ([source](https://docs.litellm.ai/docs/anthropic_unified)). The exact request/response shape and whether it explicitly handles the `anthropic-beta: skills-2025-10-02` header could not be verified: the dedicated page returned a 404 at the guessed URL, and the `providers/anthropic` page documents only the `container` passthrough, not a `/skills` endpoint or that header. Either way, any such passthrough forwards the call to Anthropic's Skills API rather than originating or storing skill content.

**Bottom line:** A gateway is plumbing. It can carry skill-related API calls (including pass-through of the Skills API and beta headers) and apply auth/routing/governance to that traffic, but the skill content lives client-side (Claude Code/SDK filesystem) or in Anthropic's workspace-scoped Skills store (API), never "in" the gateway. Treating LiteLLM as a way to "distribute skills" overstates it. Confidence: High that LiteLLM does not host/distribute skills and routes to Anthropic; Medium on the exact shape of LiteLLM's `/skills` passthrough endpoint because the dedicated doc page returned 404 and the detail came from the providers/anthropic page plus search summaries.

### 8. Manual / Script-Based Distribution (git, zip, install scripts, MDM push)

Because Claude Code and Agent SDK skills are just files on disk, the most common real-world distribution is plain file movement. These are not separate "features" so much as the delivery substrate under sections 3 and 4.

- **Git / version control**: Commit `.claude/skills/` into a project repo so every collaborator who clones gets the project skills ([source](https://code.claude.com/docs/en/skills)).
- **Zip bundles**: Skills upload to claude.ai as zip files; plugins can be tested/loaded as `.zip` via `--plugin-dir ./x.zip` or fetched as a hosted zip via `--plugin-url` ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)) ([source](https://code.claude.com/docs/en/plugins)).
- **Install scripts / dotfiles**: Any script that writes `SKILL.md` directories into `~/.claude/skills/` (personal) or a repo's `.claude/skills/` (project) installs a skill; Claude Code watches these directories and picks up adds/edits/removals within the current session (a brand-new top-level skills directory needs a restart) ([source](https://code.claude.com/docs/en/skills)).
- **MDM / config-management push (Jamf/Intune/SCCM, Ansible, etc.)**: Pushing skill folders to managed endpoints' `~/.claude/skills/` is a valid filesystem distribution path. For Claude Code, the documented org-wide mechanism is **managed settings** (enterprise-level skills and force-enabled/disabled plugins, plus `extraKnownMarketplaces`/`enabledPlugins`), which is the cleanest fit for centrally managed fleets ([source](https://code.claude.com/docs/en/skills)) ([source](https://code.claude.com/docs/en/discover-plugins)).
- **Open-source skill repos**: Anthropic publishes skills in `github.com/anthropics/skills` (most under Apache 2.0; the document skills are source-available). Users clone/install them or add the repo as a Claude Code marketplace ([source](https://github.com/anthropics/skills)).

**Who it's for:** Platform/IT teams and individual power users who want repeatable, fleet-wide, or scripted installation without the claude.ai UI.

**Trade-offs / limits:** Filesystem/MDM distribution only reaches Claude Code and the Agent SDK (filesystem surfaces), not claude.ai or the API, which require their own upload/endpoint flows. Manual zip uploads to claude.ai remain per-user. Security: skills run code, so treat installing one "like installing software" and audit untrusted skills ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)). Confidence: High for the documented mechanisms; Medium that MDM tools specifically are "supported" (they are not named in docs, but the underlying filesystem placement they perform is the documented path).

### 9. Comparison Table (method x audience x how it reaches the user x trade-offs)

| Method | Primary audience | How the skill reaches the user | Sharing scope | Key trade-offs / limits |
|---|---|---|---|---|
| claude.ai upload (Settings > Features) | End users on hosted Claude (Pro/Max/Team/Enterprise) | User uploads a zip in the UI | Per-user only; no admin/org management | No cross-surface sync; no central admin distribution; varying network access ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)) |
| Claude Code personal skills | Individual developers | Files in `~/.claude/skills/` | All the user's projects | Local to that user/machine ([source](https://code.claude.com/docs/en/skills)) |
| Claude Code project skills | Teams on a repo | `.claude/skills/` committed to git | Everyone who clones the repo | Per-project; trust dialog governs `allowed-tools` ([source](https://code.claude.com/docs/en/skills)) |
| Claude Code managed/enterprise | IT/platform admins | Managed settings deploy org-wide | All users in org | Requires managed settings infra; overrides personal/project ([source](https://code.claude.com/docs/en/skills)) |
| Claude Code plugins + marketplaces | Teams and community | `/plugin marketplace add` + `/plugin install` (git/url/local) | Where plugin is enabled; private repo = team-internal | Namespaced skill names; runs arbitrary code, trust required ([source](https://code.claude.com/docs/en/discover-plugins)) ([source](https://code.claude.com/docs/en/plugins)) |
| Agent SDK (filesystem + `skills` option) | App developers embedding Claude | Ship `.claude/skills/` with the app; load via `settingSources`/`skills` | Whatever the app deploys | No programmatic registration; `allowed-tools` frontmatter ignored ([source](https://code.claude.com/docs/en/agent-sdk/skills)) |
| Claude API `/v1/skills` | API/platform teams | Upload via `POST /v1/skills`; reference `skill_id` in `container` | Workspace-wide (all workspace members) | Beta headers required; container has no network/package install; not ZDR-eligible ([source](https://platform.claude.com/docs/en/build-with-claude/skills-guide)) ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)) |
| Manual: git / zip / install script | Power users, IT | Write SKILL.md dirs into skill folders | Depends on target (per-user / per-repo) | Reaches only filesystem surfaces (Claude Code/SDK) ([source](https://code.claude.com/docs/en/skills)) |
| MDM / config management (Jamf/Intune/SCCM) | Managed fleets | Push skill folders to `~/.claude/skills/`; or managed settings | Fleet-wide | Not named in docs; managed settings is the first-class path ([source](https://code.claude.com/docs/en/discover-plugins)) |
| Gateway (LiteLLM) | Platform teams managing model traffic | Routes the API calls; can pass through Skills API + beta headers | N/A (does not store skills) | Does NOT host/distribute skills; skill lives client-side or in Anthropic's container ([source](https://docs.litellm.ai/docs/providers/anthropic)) ([source](https://docs.litellm.ai/docs/anthropic_unified)) |

## Confidence Assessment

### High Confidence
- The five primary distribution channels (claude.ai upload, Claude Code filesystem, Agent SDK filesystem, API `/v1/skills`, plugins/marketplaces) and their sharing scopes, from official Anthropic docs ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)) ([source](https://code.claude.com/docs/en/skills)).
- SKILL.md structure, progressive disclosure, and the open standard with broad third-party adoption ([source](https://agentskills.io)) ([source](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)).
- API beta headers, `/v1/skills` CRUD endpoints, workspace-wide sharing, and container runtime constraints ([source](https://platform.claude.com/docs/en/build-with-claude/skills-guide)).
- Claude Code plugin/marketplace mechanics and managed-settings org distribution ([source](https://code.claude.com/docs/en/plugins)) ([source](https://code.claude.com/docs/en/discover-plugins)).
- LiteLLM does not host or distribute skills; it routes the call to Anthropic, where the skill resolves ([source](https://docs.litellm.ai/docs/providers/anthropic)).

### Medium Confidence
- That the Claude Desktop app exposes the identical Settings > Features upload flow documented for claude.ai (it is the same hosted product/account model, but the doc text says "claude.ai") ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).
- The precise shape of LiteLLM's dedicated `/skills` passthrough endpoint. A "/skills - Anthropic Skills API" page is listed in LiteLLM's docs nav, but the dedicated page 404'd at the guessed URL and the `providers/anthropic` page documents only the `container` passthrough ([source](https://docs.litellm.ai/docs/anthropic_unified)).
- The exact mechanics of an MCP server surfacing a skill (docs reference it only via the `skillOverrides` setting) ([source](https://code.claude.com/docs/en/skills)).
- MDM tools being a "supported" path (not named in docs; the filesystem placement they perform is documented) ([source](https://code.claude.com/docs/en/discover-plugins)).

## Open Questions

- Does the Claude Desktop app have any distribution path distinct from claude.ai (e.g., a desktop-only skill-creator or local skill folder)? The Gemini-grounded summary suggested a skill-creator/upload path, but Anthropic's overview documents uploads under claude.ai specifically. Not verified against a desktop-specific doc page.
- Will claude.ai gain centralized admin/org-wide custom-skill management for Team/Enterprise? Currently documented as unavailable ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).
- Exact request/response schema of LiteLLM's `/skills` passthrough endpoint (dedicated doc page not reachable at the guessed URL).
- Whether any first-class mechanism lets an MCP server install a SKILL.md as a skill into a client, beyond visibility management via `skillOverrides`.

## Sources

- **Official Anthropic Docs**
  - [Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
  - [Use Skills with the Claude API (skills-guide)](https://platform.claude.com/docs/en/build-with-claude/skills-guide)
  - [Use Skills in Claude Code](https://code.claude.com/docs/en/skills)
  - [Agent Skills in the SDK](https://code.claude.com/docs/en/agent-sdk/skills)
  - [Create plugins](https://code.claude.com/docs/en/plugins)
  - [Discover and install plugins (marketplaces)](https://code.claude.com/docs/en/discover-plugins)

- **Anthropic Blog / News**
  - [Engineering: Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

- **Official GitHub Repos**
  - [anthropics/skills (open-source + source-available skills, also usable as a Claude Code marketplace)](https://github.com/anthropics/skills)

- **Open Standard**
  - [Agent Skills open standard / client showcase](https://agentskills.io)

- **Third-Party / Community**
  - [LiteLLM Anthropic provider (Agent Skills via `container`, routes to Anthropic)](https://docs.litellm.ai/docs/providers/anthropic)
  - [LiteLLM Anthropic unified /v1/messages (proxy/routing behavior)](https://docs.litellm.ai/docs/anthropic_unified)

## Update History

- 2026-05-29 — initial report

## How This Report Was Generated

Generated by the Research agent via Claude Code. SearXNG MCP tools were unavailable (the first calls returned `Invalid request parameters`), so research used the Tier 2 fallback (Cornell Gateway Gemini Enterprise Web Search) to surface candidate URLs, followed by WebFetch verification of every official Anthropic documentation page, the engineering blog post, the agentskills.io standard page, the anthropics/skills GitHub repo, and LiteLLM documentation. Every factual claim is grounded in a fetched source and cited inline. Compiled 2026-05-29 2:58 PM ET.
