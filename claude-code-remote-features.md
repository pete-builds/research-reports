# Claude Code Remote Features: Complete Landscape

**Date:** April 2, 2026 (Eastern Time)
**Researcher:** Research Agent
**Classification:** Technical Reference

---

## Executive Summary

Claude Code has evolved from a local-only CLI tool into a multi-surface development platform with six distinct remote access mechanisms. The core insight: "remote" in Claude Code's world means different things depending on whether you want to (a) control a local session from another device, (b) run sessions entirely in the cloud without a local machine, (c) react to external events pushed into a session, or (d) schedule autonomous recurring work. All six approaches are documented below with their tradeoffs.

**Confidence level:** High. All claims are sourced from Anthropic's official documentation at code.claude.com/docs, fetched and verified on 2026-04-02.

---

## The Six Remote Access Mechanisms

### 1. Remote Control (local session, remote UI)

**What it does:** Connects claude.ai/code or the Claude mobile app (iOS/Android) to a Claude Code session running on your local machine. Your terminal stays the execution environment; the web/mobile interface is just a window into it.

**How to start:**
- **Server mode:** `claude remote-control` (dedicated server, waits for connections)
- **Interactive mode:** `claude --remote-control` or `claude --rc` (normal session + remote access)
- **From existing session:** `/remote-control` or `/rc` slash command
- **Auto-enable for all sessions:** `/config` > "Enable Remote Control for all sessions" > `true`

**Key flags (server mode):**
| Flag | Purpose |
|------|---------|
| `--name "My Project"` | Custom session title visible at claude.ai/code |
| `--spawn <mode>` | `same-dir` (default) or `worktree` for git worktree isolation |
| `--capacity <N>` | Max concurrent sessions (default 32) |
| `--verbose` | Detailed connection logs |
| `--sandbox` / `--no-sandbox` | Filesystem/network isolation |

**Connection methods:**
- Open the session URL displayed in terminal
- Scan QR code (press spacebar in server mode to toggle)
- Find the session by name in claude.ai/code sidebar (green dot = online)

**Security model:** Outbound HTTPS only, no inbound ports opened. All traffic routes through Anthropic's API over TLS. Multiple short-lived credentials, each scoped to a single purpose and independently expiring.

**Availability:** All plans (Pro, Max, Team, Enterprise). API keys NOT supported. Team/Enterprise: off by default, admin must enable "Remote Control" toggle at claude.ai/admin-settings/claude-code. Requires Claude Code v2.1.51+.

**Limitations:**
- One remote session per interactive process (use server mode + `--spawn` for concurrency)
- Terminal must stay open; closing the terminal ends the session
- Network outage > ~10 minutes causes session timeout
- If laptop sleeps and wakes, it reconnects automatically

**Source:** [Remote Control docs](https://code.claude.com/docs/en/remote-control)

---

### 2. Claude Code on the Web (cloud-native, no local machine needed)

**What it does:** Runs Claude Code sessions entirely on Anthropic-managed cloud infrastructure. Your repo is cloned from GitHub into an isolated VM. No local machine required at all.

**How to start:**
- **From browser:** Visit claude.ai/code, connect GitHub, select environment, submit task
- **From terminal:** `claude --remote "Fix the auth bug in login.ts"` (creates a new cloud session)
- **From CLI setup:** `/web-setup` syncs gh credentials and opens claude.ai/code
- **From Slack:** `@Claude` mention with a coding task (auto-routes to cloud session)

**Key capabilities:**
- Diff view for reviewing changes before PR creation
- Auto-fix: Claude watches PRs and automatically responds to CI failures and review comments
- Multiple repos per session
- Setup scripts (bash, runs before Claude launches) for custom environment config
- Environment variables for secrets/API keys
- Network access: Limited (allowlisted domains, default), Full, or None
- Cloud image includes Python, Node.js, Ruby, PHP, Java, Go, Rust, C++, PostgreSQL 16, Redis 7.0

**Session handoff:**
- **Terminal to web:** `claude --remote "task description"` (one-way: creates NEW cloud session)
- **Web to terminal:** `/teleport` or `claude --teleport` pulls a cloud session back to your terminal
- **From /tasks:** press `t` to teleport into a background session

**Availability:** Pro, Max, Team, Enterprise (Enterprise needs premium or Chat+Code seats). Research preview.

**Limitations:**
- GitHub only (no GitLab, Bitbucket). GitHub Enterprise Server supported for Team/Enterprise.
- Cannot push an existing terminal session to the web (only create new ones with `--remote`)
- No custom VM images/snapshots yet; use setup scripts instead

**Source:** [Claude Code on the web docs](https://code.claude.com/docs/en/claude-code-on-the-web)

---

### 3. Dispatch (phone to Desktop app)

**What it does:** A persistent conversation in the Claude Desktop app's Cowork tab. You message Dispatch a task from your phone, and it spawns a Claude Code session on your Desktop machine.

**How it works:**
- Message Dispatch with a task (e.g., "fix the login bug")
- Dispatch decides if it's development work and spawns a Code session
- Session appears in Desktop sidebar with a "Dispatch" badge
- Push notification on phone when session finishes or needs approval
- If computer use is enabled, Dispatch-spawned sessions can use it too (app approvals expire after 30 minutes)

**Availability:** Pro and Max plans only. NOT available on Team or Enterprise.

**Setup:** Pair mobile app with Desktop per [Dispatch help article](https://support.claude.com/en/articles/13947068).

**Source:** [Desktop docs, Dispatch section](https://code.claude.com/docs/en/desktop)

---

### 4. Channels (external events pushed into running sessions)

**What it does:** MCP servers that push messages, alerts, and webhooks into your running Claude Code session. Claude reacts to events while you're away from the terminal. Two-way: Claude can reply back through the same channel.

**Supported platforms:**
- **Telegram:** Bot-based, BotFather setup, pairing codes
- **Discord:** Bot-based, Developer Portal setup, pairing codes
- **iMessage:** macOS only, reads Messages database directly, AppleScript replies
- **Custom:** Build your own channel via the Channels reference

**How to start:**
```bash
# Install plugin
/plugin install telegram@claude-plugins-official

# Restart with channel enabled
claude --channels plugin:telegram@claude-plugins-official
```

**Security:** Sender allowlist per channel. Only paired/approved IDs can push messages. Permission relay capability lets you approve/deny tool use remotely through the channel.

**Key distinction from Remote Control:** Channels push external events INTO a session. Remote Control lets YOU drive the session from another device. They solve different problems.

**Availability:** Research preview, requires Claude Code v2.1.80+, claude.ai login required. Team/Enterprise: off by default, admin must enable `channelsEnabled` in managed settings.

**Limitations:** Events only arrive while session is open. For always-on setup, run Claude in a background process.

**Source:** [Channels docs](https://code.claude.com/docs/en/channels)

---

### 5. Scheduled Tasks (autonomous recurring work)

Three tiers of scheduling, each with different tradeoffs:

#### 5a. Cloud Scheduled Tasks
**What it does:** Runs a prompt on a recurring cadence on Anthropic cloud infrastructure. Works even when your computer is off.

**How to create:**
- **Web:** claude.ai/code/scheduled > "New scheduled task"
- **Desktop:** Schedule page > "New task" > "New remote task"
- **CLI:** `/schedule` or `/schedule daily PR review at 9am`

**Schedule options:** Hourly, Daily (default 9 AM local), Weekdays, Weekly. Custom cron via `/schedule update`. Minimum interval: 1 hour.

**Key features:**
- One or more GitHub repos per task
- Cloud environment with network access, env vars, setup scripts
- MCP connectors (Slack, Linear, Google Drive, etc.)
- Branch restrictions: pushes to `claude/`-prefixed branches by default
- Runs autonomously with no permission prompts
- Each run creates a session you can review, continue, or PR from

**Source:** [Web scheduled tasks docs](https://code.claude.com/docs/en/web-scheduled-tasks)

#### 5b. Desktop Scheduled Tasks
**What it does:** Runs Claude on a recurring schedule on your local machine. Persistent across restarts, but requires machine to be on.

**Minimum interval:** 1 minute. Access to local files and MCP servers. Permission prompts configurable per task.

**Source:** [Desktop docs, scheduled tasks section](https://code.claude.com/docs/en/desktop)

#### 5c. /loop (session-scoped)
**What it does:** Quick in-session polling. Dies when you exit.

```
/loop 5m check if the deployment finished
/loop 20m /review-pr 1234
```

**Interval syntax:** `5m`, `2h`, `30s` (rounded to nearest minute). Default: 10 minutes. 7-day auto-expiry on recurring tasks.

**Source:** [Scheduled tasks docs](https://code.claude.com/docs/en/scheduled-tasks)

---

### 6. Claude Code in Slack (team cloud sessions)

**What it does:** `@Claude` mentions in Slack channels auto-detect coding tasks and spawn cloud sessions on claude.ai/code. Progress updates posted to Slack threads.

**Routing modes:**
- **Code only:** All mentions route to Claude Code sessions
- **Code + Chat:** Intelligent routing between Code and Chat based on message content

**Session flow:** Mention > detection > cloud session created > progress updates in Slack > completion notification with "View Session" and "Create PR" buttons.

**Availability:** Pro, Max, Team, Enterprise. Requires Claude Code on the web access + connected GitHub repos. Channels only (no DMs).

**Source:** [Slack docs](https://code.claude.com/docs/en/slack)

---

## Headless / Programmatic Mode (the `-p` flag)

Not a "remote" feature per se, but essential context for automation. The `-p` flag runs Claude Code non-interactively:

```bash
claude -p "Fix the bug in auth.py" --allowedTools "Read,Edit,Bash"
```

**Key options:**
- `--output-format json` for structured output with session metadata
- `--output-format stream-json` for real-time streaming
- `--json-schema` for schema-constrained output
- `--bare` for fast startup (skips hooks, MCP, CLAUDE.md discovery)
- `--continue` / `--resume <session-id>` for multi-turn conversations
- `--allowedTools` for auto-approving specific tools

**Use cases:** CI/CD pipelines, GitHub Actions, GitLab CI, scripts, the Agent SDK (Python/TypeScript).

**Source:** [Headless mode docs](https://code.claude.com/docs/en/headless)

---

## Feature Comparison Matrix

| Feature | Execution Location | Requires Local Machine | Mobile Access | Requires GitHub | Plan Availability |
|---------|-------------------|----------------------|---------------|-----------------|-------------------|
| Remote Control | Your machine | Yes (must stay on) | Yes (iOS/Android/web) | No | All plans |
| Cloud Sessions | Anthropic cloud | No | Yes | Yes | Pro, Max, Team, Enterprise |
| Dispatch | Your machine (Desktop) | Yes (Desktop app running) | Yes (phone trigger) | No | Pro, Max only |
| Channels | Your machine | Yes (session running) | Yes (Telegram/Discord/iMessage) | No | All plans (research preview) |
| Cloud Scheduled Tasks | Anthropic cloud | No | Monitor via web/app | Yes | All cloud-eligible plans |
| Slack | Anthropic cloud | No | Via Slack mobile | Yes | Pro, Max, Team, Enterprise |
| Headless (-p) | Your machine | Yes | No (CLI only) | No | All plans + API keys |

---

## Key Gaps and Open Requests

1. **Headless + Remote Control daemon:** There is an open feature request ([GitHub #30447](https://github.com/anthropics/claude-code/issues/30447)) for `claude remote-control --headless` to run as a daemonizable service without TTY dependency. This would make the iOS app effectively a permission controller for an always-on headless instance. As of April 2026, this is not yet implemented.

2. **No terminal-to-web session push:** You can create NEW cloud sessions with `--remote`, but you cannot push an existing in-progress terminal session to the cloud. The handoff is one-way (web-to-terminal via teleport).

3. **GitHub-only cloud sessions:** Cloud sessions, Slack, and scheduled tasks all require GitHub. GitLab, Bitbucket, and other providers are not supported for cloud execution.

4. **No custom VM images:** Cloud environments use Anthropic's universal image. No support for custom Docker images or snapshots yet; setup scripts are the workaround.

---

## Practical Recommendations

**"I want to check on my running task from my phone":** Use Remote Control (`claude --rc`). Scan the QR code to connect from the Claude mobile app.

**"I want to kick off work without being at my desk":** Use `claude --remote "task"` from terminal before leaving, or use Dispatch from the Claude mobile app to spawn a Desktop session.

**"I want Claude to do something every morning automatically":** Use Cloud Scheduled Tasks via claude.ai/code/scheduled or `/schedule` in the CLI.

**"I want CI failures to trigger Claude automatically":** Use Channels with a webhook receiver, or enable Auto-fix on a PR from the web interface.

**"I want to text Claude a task from iMessage":** Set up the iMessage channel plugin on macOS.

---

## Sources

All sources verified via direct WebFetch on 2026-04-02:

- [Remote Control](https://code.claude.com/docs/en/remote-control) - Official Anthropic docs
- [Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web) - Official Anthropic docs
- [Headless / Programmatic mode](https://code.claude.com/docs/en/headless) - Official Anthropic docs
- [Channels](https://code.claude.com/docs/en/channels) - Official Anthropic docs
- [Web Scheduled Tasks](https://code.claude.com/docs/en/web-scheduled-tasks) - Official Anthropic docs
- [CLI Scheduled Tasks (/loop)](https://code.claude.com/docs/en/scheduled-tasks) - Official Anthropic docs
- [Desktop (Dispatch, Desktop scheduling)](https://code.claude.com/docs/en/desktop) - Official Anthropic docs
- [Slack integration](https://code.claude.com/docs/en/slack) - Official Anthropic docs
- [Headless + Remote Control feature request](https://github.com/anthropics/claude-code/issues/30447) - GitHub issue
