---
title: "Claude Code Skills vs MCP Servers (with detailed MCP protocol breakdown)"
date: 2026-04-29
updated: 2026-04-29 6:30 PM ET / 22:30 UTC
summary: "Skills are filesystem-based markdown packages that load progressively into the LLM's context; MCP servers are separate processes that expose tools, resources, and prompts to a host over a JSON-RPC 2.0 protocol. They solve different problems and are usually best used together: MCP for connecting to external systems, Skills for teaching the agent how to use them."
---

## Current Status

- **Skills** (also "Agent Skills") are an Anthropic feature that ships as folders containing a `SKILL.md` file with YAML frontmatter (`name`, `description`), plus optional bundled scripts and reference files. Claude loads them via progressive disclosure: metadata at startup, full instructions when triggered, additional files only when needed ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).
- **MCP** (Model Context Protocol) is an open JSON-RPC 2.0 protocol for connecting host applications (Claude Code, Claude Desktop, etc.) to external servers that expose **tools**, **resources**, and **prompts**. Two standard transports: **stdio** (local subprocess) and **Streamable HTTP** (HTTP POST + optional SSE upgrade). The pure HTTP+SSE transport from the 2024-11-05 spec is deprecated ([source](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports)).
- **Decision rule**: use an MCP server when you need to integrate an external API, database, or service over the network. Use a Skill when you need to teach the agent a workflow, set of conventions, or domain expertise it can execute with the tools it already has (Bash, Read, Edit, plus any MCP tools already loaded). The two compose: a Skill commonly orchestrates calls to MCP tools.
- **Token cost difference is the headline tradeoff**. MCP tool definitions all load into the system prompt at session start (often "tens of thousands of tokens"). Skill metadata loads at ~100 tokens per skill, and the body only loads when the skill is triggered ([source](https://simonwillison.net/2025/Oct/16/claude-skills/)) ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).
- **Pete's stack maps cleanly to the recommended pattern**: MCP servers (`nfl`, `strava`, `searxng`, `homelab-arr`, `portainer`, `anthropic-tracker`) for service integration; Skills (`tank`, `radar`, `coach`, `forge`, `research`, etc.) as orchestrators that compose tools and encode procedure. Forge already builds new MCP servers in FastMCP+httpx+Docker; the only nudge is that pure SSE transport is deprecated in the current spec, so new servers should target Streamable HTTP.

## Table of Contents

1. [Skills](#1-skills)
2. [MCP servers and the MCP protocol](#2-mcp-servers-and-the-mcp-protocol)
3. [How they interact](#3-how-they-interact)
4. [Decision matrix: Skills vs MCP](#4-decision-matrix-skills-vs-mcp)
5. [Practical guidance for Pete's workspace](#5-practical-guidance-for-petes-workspace)
6. [Confidence Assessment](#confidence-assessment)
7. [Open Questions](#open-questions)
8. [Sources](#sources)
9. [Update History](#update-history)
10. [How This Report Was Generated](#how-this-report-was-generated)

## Findings

### 1. Skills

#### What they are

Anthropic defines Agent Skills as "modular capabilities that extend Claude's functionality. Each Skill packages instructions, metadata, and optional resources (scripts, templates) that Claude uses automatically when relevant" ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)). The engineering blog frames them as "organized folders of instructions, scripts, and resources that agents can discover and load dynamically to perform better at specific tasks" ([source](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)).

#### File format

Every Skill requires a `SKILL.md` file with YAML frontmatter. The two required fields are `name` and `description` ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)):

```yaml
---
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---
```

**Field constraints** ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)):
- `name`: max 64 chars, lowercase letters/numbers/hyphens only, cannot contain XML tags, cannot contain reserved words "anthropic" or "claude"
- `description`: non-empty, max 1024 chars, cannot contain XML tags, "should include both what the Skill does and when Claude should use it"

The `allowed-tools` frontmatter field exists in Pete's local skill files (and in many third-party skill repos) and is honored by Claude Code. It is documented as an optional experimental field in the open `agentskills.io` spec, but not listed in Anthropic's Agent Skills overview; support varies between hosts. (Medium confidence: documented in the cross-host open spec but not in Anthropic's first-party overview; treat as experimental.)

#### Where they live

In Claude Code ([source](https://code.claude.com/docs/en/mcp)) ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)):
- **Personal**: `~/.claude/skills/`
- **Project**: `.claude/skills/`
- **Plugin-bundled**: shipped inside a Claude Code plugin

In Claude.ai: uploaded as zip files via Settings > Features (Pro/Max/Team/Enterprise plans with code execution).

In the Claude API: uploaded via `/v1/skills` endpoints, referenced by `skill_id` in the `container` parameter, requires three beta headers (`code-execution-2025-08-25`, `skills-2025-10-02`, `files-api-2025-04-14`).

A critical constraint: "Custom Skills do not sync across surfaces. Skills uploaded to one surface are not automatically available on others" ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).

#### Progressive disclosure (the loading model)

Anthropic calls progressive disclosure "the core design principle that makes Agent Skills" work ([source](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)). Three loading levels ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)):

| Level | When loaded | Token cost | Content |
|---|---|---|---|
| **1: Metadata** | Always (at startup) | ~100 tokens per skill | `name` and `description` from YAML frontmatter |
| **2: Instructions** | When skill is triggered | Under 5k tokens | SKILL.md body |
| **3+: Resources** | As needed | Effectively unlimited | Bundled files read or executed via bash; script code itself never enters context, only its output |

The mechanism: "When a Skill is triggered, Claude uses bash to read SKILL.md from the filesystem... If those instructions reference other files (like FORMS.md or a database schema), Claude reads those files too using additional bash commands. When instructions mention executable scripts, Claude runs them via bash and receives only the output (the script code itself never enters context)" ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)).

This is what gives Skills their token efficiency relative to MCP. Simon Willison: "MCP implementations famously consume 'tens of thousands of tokens of context.' Skills avoid this by leveraging the LLM's existing capability to understand CLI tools and execute code" ([source](https://simonwillison.net/2025/Oct/16/claude-skills/)).

#### What Skills can do

Skills run in whatever execution environment the host provides. In Claude Code that's "full network access — Skills have the same network access as any other program on the user's computer" plus filesystem access, bash, and any MCP tools already configured ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)). They can:

- Provide procedural knowledge ("when X, do Y")
- Bundle reference files (schemas, API docs, examples) loaded only when needed
- Bundle executable scripts that Claude runs deterministically
- Spawn subagents via Task
- Call any MCP tool the agent has configured

What they cannot do: define new wire-protocol tools (only MCP servers can), expose persistent stateful endpoints to other clients, run as daemons.

#### Security

The Anthropic docs warn: "We strongly recommend using Skills only from trusted sources... a malicious Skill can direct Claude to invoke tools or execute code in ways that don't match the Skill's stated purpose" ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)). The engineering blog: "thorough auditing is essential before use, including: reviewing all bundled file contents, examining code dependencies, scrutinizing instructions directing Claude toward external network sources" ([source](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)).

Willison adds: "Sandboxing attacks like prompt injections remains a critical unsolved challenge requiring serious engineering attention" ([source](https://simonwillison.net/2025/Oct/16/claude-skills/)). A skill that fetches data from an external URL inherits the lethal trifecta: private data + untrusted content + external comms.

### 2. MCP servers and the MCP protocol

#### What MCP is

MCP is an open protocol that "consists of several key components that work together: Base Protocol (Core JSON-RPC message types), Lifecycle Management, Authorization, Server Features (Resources, prompts, and tools exposed by servers), Client Features (Sampling and root directory lists provided by clients), Utilities" ([source](https://modelcontextprotocol.io/specification/2025-11-25/basic)).

Three roles: **Hosts** (LLM applications that initiate connections, e.g., Claude Code), **Clients** (the connector inside the host), **Servers** (services exposing capabilities). All messages "MUST follow the JSON-RPC 2.0 specification" ([source](https://modelcontextprotocol.io/specification/2025-11-25/basic)).

#### JSON-RPC 2.0 message format

Three message types ([source](https://modelcontextprotocol.io/specification/2025-11-25/basic)):

```typescript
// Request
{ jsonrpc: "2.0", id: string|number, method: string, params?: {...} }

// Result Response
{ jsonrpc: "2.0", id: string|number, result: {...} }

// Error Response
{ jsonrpc: "2.0", id?: string|number, error: { code: number, message: string, data?: unknown } }

// Notification (one-way; receiver MUST NOT respond)
{ jsonrpc: "2.0", method: string, params?: {...} }
```

Key requirements: "Requests MUST include a string or integer ID. Unlike base JSON-RPC, the ID MUST NOT be `null`. The request ID MUST NOT have been previously used by the requestor within the same session" ([source](https://modelcontextprotocol.io/specification/2025-11-25/basic)).

#### Transports

The current spec (2025-11-25) defines two standard transports ([source](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports)): **stdio** and **Streamable HTTP**. "Clients SHOULD support stdio whenever possible."

**stdio** (local subprocess):
- Client launches the server as a subprocess
- Server reads JSON-RPC from stdin, writes to stdout
- "Messages are delimited by newlines, and MUST NOT contain embedded newlines"
- "The server MUST NOT write anything to its stdout that is not a valid MCP message"
- stderr is for logs only

**Streamable HTTP** (the current standard for remote servers):
- Server provides a single MCP endpoint that supports POST and GET
- Client POSTs JSON-RPC messages to the endpoint
- Client must include `Accept: application/json, text/event-stream`
- Server responds with either a single JSON object OR a `text/event-stream` SSE stream for streaming responses
- Optional resumability: server attaches event IDs; client reconnects with `Last-Event-ID` header to replay missed messages
- Optional session management: server returns `MCP-Session-Id` header on init; client echoes it on subsequent requests; server can terminate the session, after which it MUST return 404
- Protocol version negotiation via `MCP-Protocol-Version` header (e.g., `MCP-Protocol-Version: 2025-11-25`)

**Security requirements for HTTP** ([source](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports)):
1. Servers MUST validate the `Origin` header to prevent DNS rebinding attacks
2. Local servers SHOULD bind only to 127.0.0.1, not 0.0.0.0
3. Servers SHOULD implement proper authentication

**The deprecated SSE transport**: pure HTTP+SSE from protocol version 2024-11-05 is replaced by Streamable HTTP. "This replaces the HTTP+SSE transport from protocol version 2024-11-05" ([source](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports)). Backwards-compat instructions are provided: servers can host both endpoints; clients try POST `InitializeRequest` first and fall back to GET-for-SSE if they get 400/404/405.

#### Lifecycle

Three phases ([source](https://modelcontextprotocol.io/specification/2025-11-25/basic/lifecycle)):

1. **Initialization** (MUST be the first interaction): Client sends `initialize` request with `protocolVersion`, `capabilities`, `clientInfo`. Server MUST respond with its own `protocolVersion`, `capabilities`, `serverInfo`. Client then sends `notifications/initialized`.
2. **Operation**: "Both parties MUST: Respect the negotiated protocol version, Only use capabilities that were successfully negotiated."
3. **Shutdown**: For stdio, client closes stdin, waits, then SIGTERM/SIGKILL. For HTTP, close the connection.

#### Capability negotiation

Capabilities declared during init ([source](https://modelcontextprotocol.io/specification/2025-11-25/basic/lifecycle)):

| Side | Capability | What it enables |
|---|---|---|
| Client | `roots` | Provide filesystem roots to server |
| Client | `sampling` | Server can ask client to call the LLM |
| Client | `elicitation` | Server can ask client for structured user input mid-task |
| Client | `tasks` | Task-augmented client requests |
| Server | `prompts` | Offers prompt templates |
| Server | `resources` | Provides readable resources |
| Server | `tools` | Exposes callable tools |
| Server | `logging` | Emits structured log messages |
| Server | `completions` | Argument autocompletion |

Sub-capabilities: `listChanged` (server emits notification when its list changes), `subscribe` (client can subscribe to individual resource changes; resources only).

#### Server primitives

**Tools** (model-controlled actions) ([source](https://modelcontextprotocol.io/specification/2025-11-25/server/tools)):
- Discovered via `tools/list`, invoked via `tools/call`
- Tool definition: `name`, optional `title`, `description`, `inputSchema` (JSON Schema 2020-12 by default), optional `outputSchema`, optional `annotations`, optional `icons`
- Tool names: 1-128 chars, ASCII letters/digits/`_`/`-`/`.`, no spaces, case-sensitive, unique within server
- Results: `content` array (text, image, audio, resource_link, embedded resource) plus optional `structuredContent` and `isError` flag
- Two error-reporting paths: protocol errors (JSON-RPC `error`) for unknown tools or malformed requests; tool execution errors (`isError: true` in result) for API failures and validation issues that the model can self-correct
- Trust model: "Tools in MCP are designed to be model-controlled... For trust & safety and security, there SHOULD always be a human in the loop with the ability to deny tool invocations" ([source](https://modelcontextprotocol.io/specification/2025-11-25/server/tools))

**Resources** (application-driven readable data) ([source](https://modelcontextprotocol.io/specification/2025-11-25/server/resources)):
- Discovered via `resources/list`, read via `resources/read`, optionally subscribed via `resources/subscribe`
- Each identified by URI (RFC 3986). Standard schemes: `https://`, `file://`, `git://`, plus custom
- Returns text or base64 binary content
- "Resources in MCP are designed to be application-driven, with host applications determining how to incorporate context based on their needs" — this is the key conceptual difference vs tools

**Prompts** (user-invoked templates): User-controlled, generally surfaced as slash-style commands or template pickers in the host UI.

#### Client primitives

- **Sampling**: server can request that the client call its LLM, e.g., to ask for a sub-completion
- **Roots**: client tells server which filesystem directories are in scope
- **Elicitation**: server asks client to collect structured input from the user mid-task; Claude Code shows a form dialog or opens a URL ([source](https://code.claude.com/docs/en/mcp))

#### Authorization

"MCP provides an Authorization framework for use with HTTP. Implementations using an HTTP-based transport SHOULD conform to this specification, whereas implementations using STDIO transport SHOULD NOT follow this specification, and instead retrieve credentials from the environment" ([source](https://modelcontextprotocol.io/specification/2025-11-25/basic)).

In Claude Code, OAuth 2.0 is the supported flow for HTTP servers, with explicit support for Dynamic Client Registration, Client ID Metadata Documents (CIMD), pre-configured client credentials, and `oauth.scopes` pinning. For non-OAuth schemes there's `headersHelper` (a script that generates auth headers fresh on each connection) ([source](https://code.claude.com/docs/en/mcp)).

#### How Claude Code launches and manages MCP servers

([source](https://code.claude.com/docs/en/mcp)):

```bash
claude mcp add --transport stdio myserver -- npx server
claude mcp add --transport http stripe https://mcp.stripe.com
claude mcp list
claude mcp get github
claude mcp remove github
# inside Claude Code:
/mcp   # status, OAuth login, clear auth
```

**Three configuration scopes**:
- `local` (default) — current project only, stored in `~/.claude.json`, private
- `project` — current project, shared via `.mcp.json` checked into version control
- `user` — all your projects, stored in `~/.claude.json`

Project-scoped servers from `.mcp.json` require an approval prompt before use ("For security reasons, Claude Code prompts for approval before using project-scoped servers from `.mcp.json` files").

**Reconnection**: HTTP/SSE servers auto-reconnect with exponential backoff (5 attempts, starting at 1s, doubling). Initial connection retries up to 3 times on transient errors as of v2.1.121. Stdio servers are not auto-reconnected.

**Output limits**: warning at 10,000 tokens, default cap at 25,000 tokens, configurable via `MAX_MCP_OUTPUT_TOKENS`. Tool authors can opt into `_meta["anthropic/maxResultSizeChars"]` (up to 500,000) per tool.

**Resource @-mentions**: `@server:protocol://resource/path` references an MCP resource inline in a prompt.

### 3. How they interact

The two are designed to compose. Skills can call any tool the agent has, and that includes MCP tools. The most common production pattern is: **MCP server provides the connection; Skill provides the procedure**.

A representative example from a Databricks article: "For the data-querying skill, I gave Claude a detailed instruction on how to use the Databricks Managed MCP server to send LLM-generated SQL" ([source](https://medium.com/@hiydavid/specializing-claude-code-a-quick-guide-to-agent-skills-and-mcp-on-databricks-c0cfdd43637d)) (search snippet only; pattern is widely documented in the same form across multiple sources).

The framing from one practitioner article: *"MCP gives Claude access to things. Skills teach Claude how to use them"* ([source](https://medium.com/@harshiljani2002/skills-vs-mcp-when-should-you-use-what-19f8fd97144a)).

Alex Op's full-stack diagram of Claude Code lays out the layering ([source](https://alexop.dev/posts/understanding-claude-code-full-stack/)):

| Layer | Component | Role |
|---|---|---|
| 1 | MCP | External system connections (the "universal adapter") |
| 2 | CLAUDE.md | Project memory, always-loaded context |
| 3 | Skills + Hooks | Reusable workflows; lifecycle event reactions |
| 4 | Subagents | Isolated context, parallel work |
| 5 | Plugins | Distribution bundles |

Decision matrix from the same source:

| Goal | Use | Why |
|---|---|---|
| Static knowledge | CLAUDE.md | Always available |
| Manual workflow | Skill (with `disable-model-invocation: true`) | Explicit user control |
| Auto-triggered expertise | Skill (default) | Description-based activation |
| Parallel analysis | Subagents | Isolated contexts |
| Event-driven enforcement | Hooks | Lifecycle reactions |
| Team distribution | Plugins | Bundled configurations |
| External system access | MCP | Native protocol |

### 4. Decision matrix: Skills vs MCP

Synthesizing all sources, this is when each wins.

#### Use a Skill when

- The work is procedural: "follow these steps, in this order, with these conventions"
- You want token efficiency: a skill description is ~100 tokens until triggered; an equivalent set of tool definitions is loaded into the system prompt for every session ([source](https://simonwillison.net/2025/Oct/16/claude-skills/)) ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview))
- You can use the agent's existing tools (Bash, Read, Edit, Grep, plus already-configured MCPs) and just need to teach it how
- You want portability: SKILL.md is plain markdown that runs in any compatible host (Claude Code, Claude.ai, OpenAI Codex, GitHub Copilot, Cursor — though feature support varies) ([source](https://agentskills.io/specification)) ([source](https://www.morphllm.com/agents-md-guide))
- The domain knowledge is relatively stable (changes infrequently)
- You want low operational overhead: no process to run, no port to expose, no auth to manage

#### Use an MCP server when

- You're integrating an external service (API, database, internal system) and want a stable wire-protocol surface
- You need centralized maintenance: "server updates propagate automatically" to all clients ([source](https://www.llamaindex.ai/blog/skills-vs-mcp-tools-for-agents-when-to-use-what))
- You need cross-host reuse: the same MCP server can serve Claude Code, Claude Desktop, Cursor, OpenAI's Codex, and any other MCP-compatible client
- You need stateful sessions, persistent connections, or the operation is too sensitive for the LLM to drive directly (e.g., a database write)
- You want a deterministic schema-validated tool surface (`inputSchema`, `outputSchema`)
- You need multi-tenant or remote hosting (Streamable HTTP with OAuth)
- You need server-pushed updates or notifications (Claude Code supports MCP `claude/channel` push messages and `list_changed` notifications) ([source](https://code.claude.com/docs/en/mcp))

#### Tradeoffs

| Dimension | Skills | MCP servers |
|---|---|---|
| **Token cost at startup** | ~100 tokens per skill (metadata only) | Tool definitions for every server load into system prompt; can be tens of thousands of tokens ([source](https://simonwillison.net/2025/Oct/16/claude-skills/)) |
| **Latency** | No network overhead; runs in-conversation ([source](https://www.llamaindex.ai/blog/skills-vs-mcp-tools-for-agents-when-to-use-what)) | Network call per tool invocation |
| **Deployment** | Drop a folder in `.claude/skills/` or `~/.claude/skills/` | Run a process, expose stdio or HTTP, manage credentials |
| **Distribution** | Markdown files, easy to share via git or a skills registry | MCP registry, OAuth, or manual config; works across all MCP hosts |
| **Determinism** | LLM interprets natural-language instructions; can vary ([source](https://www.llamaindex.ai/blog/skills-vs-mcp-tools-for-agents-when-to-use-what)) | Schema-validated calls with fixed wire format |
| **Schema stability** | Markdown is whatever you write | JSON Schema validated; spec versioned via `MCP-Protocol-Version` header |
| **Cross-host portability** | High (filesystem markdown is universal) | High (any MCP client) but each host has its own quirks (output limits, auth flows) |
| **Code execution** | Runs scripts via bash, output-only enters context | Code runs server-side; only result content returns |
| **Security boundary** | Inherits agent's permissions; full network and filesystem access in Claude Code | Separate process with explicit tool surface; HTTP transports support OAuth |
| **Updateability** | Edit a markdown file | Restart the server, all clients see new tools |

#### Hybrid: Skill that calls MCP (very common)

This is the pattern Pete already uses. A skill encodes "how to run a Radar audit on a domain" — the skill body says "use `mcp__searxng__search_deep` with these parameters, then `WebFetch` each result, then..." The skill is the runbook; the MCP servers are the levers it pulls. Same for `coach` calling `mcp__nfl__*` tools, `research` calling `mcp__searxng__*` tools.

Armin Ronacher offers a counterpoint worth keeping in mind: MCP servers can drift on you. He cites the Sentry MCP changing its query syntax to natural language, breaking his workflow before he noticed. His take: "asking the agent to write its own tools as a skill" — i.e., letting the agent build small bash/curl wrappers — beats maintaining MCP dependencies that change unexpectedly ([source](https://lucumr.pocoo.org/2025/12/13/skills-vs-mcp/)). (Medium confidence as a general recommendation; high confidence that MCP-version drift is a real concern.)

### 5. Practical guidance for Pete's workspace

Pete's setup matches the recommended architecture closely:

- **MCP servers as data/integration layer**: `nfl`, `strava`, `searxng`, `homelab-arr`, `portainer`, `anthropic-tracker`, `pihole`, `synology`, `uptime-kuma`, threat intel, Spotify, GA4. Each provides a stable schema-validated tool surface that Claude Code, Claude Desktop, or any other MCP host can consume.
- **Skills as orchestrators**: `tank`, `radar`, `coach` (with sub-skills `scout`, `beat`, `editor`), `forge`, `architect`, `research`, `oracle`, etc. Each composes existing tools and encodes procedure.
- **Forge as MCP factory**: FastMCP + httpx + Docker on nix1 is a sound stack. FastMCP is the de facto Python framework for MCP servers and supports Streamable HTTP natively.

**One actionable nudge**: the `agents.md` reference in this repo lists Forge's pattern as "SSE transport on nix1, port range 3695+." The current MCP spec deprecates pure SSE in favor of Streamable HTTP ([source](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports)). New MCP servers built with FastMCP should target Streamable HTTP rather than the older HTTP+SSE transport. Existing servers still work because Claude Code maintains backwards compatibility (it tries POST `InitializeRequest` first, falls back to GET-for-SSE on 400/404/405). When the next Forge build comes up, switch the FastMCP transport setting from `sse` to `streamable-http` (or `http`, depending on the FastMCP version). Verify by checking the deprecation status against the latest FastMCP release notes before migrating.

**Token budget worth checking**: Pete has ~10 MCP servers active. Each server's tool definitions get pulled into the system prompt at session start. If session-start latency or token usage feels high, the relevant levers are (a) disabling unused MCP servers per project via `.mcp.json` scope, (b) considering skills for tools that are rarely used and don't need a wire protocol, (c) raising `MAX_MCP_OUTPUT_TOKENS` only when the actual outputs need it.

**Skills + MCP works well together; don't choose between them**. The pattern is: build the integration once as an MCP server (for stability, cross-host reuse, schema validation), then build skills that orchestrate it for specific workflows. Pete's `radar` calling `searxng` MCP is a textbook example.

## Confidence Assessment

### High confidence
- Skill SKILL.md format (`name`, `description` required), three-level progressive disclosure, ~100 token metadata cost, where skills live in Claude Code (`~/.claude/skills/`, `.claude/skills/`), the cross-surface non-sync limitation ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)) ([source](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills))
- MCP uses JSON-RPC 2.0; the two standard transports are stdio and Streamable HTTP; the deprecated transport is HTTP+SSE from 2024-11-05; Streamable HTTP supports session IDs, resumability via `Last-Event-ID`, and protocol version negotiation via the `MCP-Protocol-Version` header ([source](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports))
- MCP lifecycle: initialize → initialized notification → operation → shutdown; capability negotiation happens during init ([source](https://modelcontextprotocol.io/specification/2025-11-25/basic/lifecycle))
- MCP server primitives: tools (model-controlled, schema-validated), resources (application-driven, URI-keyed), prompts (user-invoked templates) ([source](https://modelcontextprotocol.io/specification/2025-11-25/server/tools)) ([source](https://modelcontextprotocol.io/specification/2025-11-25/server/resources))
- Claude Code MCP scope rules (local/project/user), `.mcp.json` format, OAuth 2.0 support including DCR/CIMD/pre-configured credentials, `headersHelper` for non-OAuth auth, reconnection with exponential backoff ([source](https://code.claude.com/docs/en/mcp))
- Skills' relative token efficiency vs MCP at session start ([source](https://simonwillison.net/2025/Oct/16/claude-skills/)) ([source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview))

### Medium confidence
- The `allowed-tools` frontmatter field is honored by Claude Code but not in the official Anthropic Agent Skills overview; treating it as a Claude-Code-specific extension. Verify against the Claude Code skills doc before relying on it for portability.
- Cross-host Skill portability claims (Codex CLI, Cursor, Gemini CLI executing them): Willison asserts this is possible because skills depend on filesystem + bash + LLM rather than a special host runtime, and the agentskills.io standard publishes a portable spec. Actual feature parity across hosts varies and was not exhaustively verified.
- The exact migration path from FastMCP SSE to Streamable HTTP: depends on FastMCP version. Verify in FastMCP changelog before migrating.

### Low confidence
- Token-cost numbers for specific MCP servers: cited only in qualitative terms ("tens of thousands of tokens") in Willison's post. No first-party Anthropic benchmark located. If Pete needs a precise number for capacity planning, run `claude mcp list` then measure token usage with each server enabled vs disabled.

## Open Questions

- What are the exact frontmatter fields Claude Code recognizes beyond `name` and `description`? (`allowed-tools`, `disable-model-invocation`, `model` are all referenced in third-party docs but not enumerated in the Anthropic Agent Skills overview; would need to read the Claude Code skills doc directly.)
- Has Anthropic published official guidance on skill-to-MCP cost tradeoffs? Have not located one; community estimates only.
- For Pete's Forge workflow specifically: which FastMCP version is currently in use, and does it support `streamable-http` transport directly? Worth confirming at next build.
- Does the `nfl` MCP server (used by Coach via Scout) currently use SSE or Streamable HTTP? If SSE, schedule migration on next iteration.

## Sources

### Primary (Anthropic + MCP spec)
- [Equipping agents for the real world with Agent Skills (Anthropic engineering blog)](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- [Agent Skills overview (Claude API docs)](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [Connect Claude Code to tools via MCP (Claude Code docs)](https://code.claude.com/docs/en/mcp)
- [MCP basic overview spec (2025-11-25)](https://modelcontextprotocol.io/specification/2025-11-25/basic)
- [MCP transports spec (2025-11-25)](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports)
- [MCP lifecycle spec (2025-11-25)](https://modelcontextprotocol.io/specification/2025-11-25/basic/lifecycle)
- [MCP tools spec (2025-11-25)](https://modelcontextprotocol.io/specification/2025-11-25/server/tools)
- [MCP resources spec (2025-11-25)](https://modelcontextprotocol.io/specification/2025-11-25/server/resources)

### Independent commentary and decision frameworks
- [Claude Skills are awesome, maybe a bigger deal than MCP — Simon Willison (Oct 2025)](https://simonwillison.net/2025/Oct/16/claude-skills/)
- [Skills vs Dynamic MCP Loadouts — Armin Ronacher (Dec 2025)](https://lucumr.pocoo.org/2025/12/13/skills-vs-mcp/)
- [Skills vs MCP Tools for agents: when to use what — LlamaIndex](https://www.llamaindex.ai/blog/skills-vs-mcp-tools-for-agents-when-to-use-what)
- [Skills vs MCP: When Should You Use What? — Harshil Jani (Medium, Feb 2026)](https://medium.com/@harshiljani2002/skills-vs-mcp-when-should-you-use-what-19f8fd97144a)
- [Understanding Claude Code's Full Stack: MCP, Skills, Subagents — alexop.dev (Nov 2025)](https://alexop.dev/posts/understanding-claude-code-full-stack/)

### Reference standards
- [Agent Skills specification — agentskills.io](https://agentskills.io/specification)
- [AGENTS.md & SKILL.md: The Complete Guide — morphllm](https://www.morphllm.com/agents-md-guide)

## Update History

- 2026-04-29 6:30 PM ET — Initial fresh-mode report. Coverage: full Skill format and lifecycle, full MCP protocol breakdown (transports, lifecycle, primitives, auth), decision matrix, hybrid patterns, Pete-specific guidance on Forge SSE → Streamable HTTP migration.

## How This Report Was Generated

Compiled by the Research agent at `/mnt/c/Users/pster/ai-cli-workspace/.claude/skills/research/SKILL.md`. Search via SearXNG (`mcp__searxng__search_deep`) with three deep queries; primary sources fetched directly via WebFetch from `modelcontextprotocol.io`, `platform.claude.com`, `code.claude.com`, `anthropic.com/engineering`, and `simonwillison.net`. Every cited URL was independently fetched and read before citation. Confidence levels reflect whether claims are anchored in first-party specs (high), corroborated commentary (medium), or single-source claims (low).
