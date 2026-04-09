# Claude Code as a Sysadmin Tool: Comprehensive Practical Guide

**Date:** April 9, 2026 (Eastern Time)
**Updated:** April 9, 2026 2:00 PM ET
**Researcher:** Research Agent
**Classification:** Technical Reference / Practical Guide

---

## Disclaimer

This guide is intended for **non-production homelab and experimental environments**. Giving an AI agent SSH access to servers carries real risk. Commands can be destructive, context can be misunderstood, and mistakes on production infrastructure can cause data loss or downtime. Use this guide to learn and experiment on systems you can afford to break. If you apply these patterns to production, do so incrementally, with strict permissions, audit logging, and human approval gates in place.

---

## Executive Summary

Claude Code has matured into a capable sysadmin assistant that can manage servers, containers, and infrastructure from the terminal. This guide focuses on the most practical workflow: **running Claude Code on your local machine and giving it SSH access to remote servers**. It also covers how to build a dedicated sysadmin agent skill that knows your infrastructure, follows safety rules, and produces structured output.

The core insight: Claude Code is not just an AI that runs shell commands. It understands what command output means and acts on it. You can say "the web server on 10.0.1.5 is down, fix it" and it checks the service status, reads the error log, identifies the config problem, patches it, re-validates, and restarts the service. [source: [computingforgeeks.com](https://computingforgeeks.com/claude-code-ssh-server-management/)]

**Confidence level:** High for all official documentation claims (sourced from code.claude.com/docs). Medium for community patterns and third-party tools (sourced from blog posts, GitHub repos, and tutorials, all cited inline).

---

## Table of Contents

1. [The Architecture: Local PC → SSH → Server](#1-the-architecture-local-pc--ssh--server)
2. [SSH Setup: Keys, Config, and Access](#2-ssh-setup-keys-config-and-access)
3. [The Infrastructure as Markdown Pattern](#3-the-infrastructure-as-markdown-pattern)
4. [Building a Sysadmin Agent Skill](#4-building-a-sysadmin-agent-skill)
5. [Docker and Container Management](#5-docker-and-container-management)
6. [Permission Configuration for Sysadmin Work](#6-permission-configuration-for-sysadmin-work)
7. [Hooks for Infrastructure Automation](#7-hooks-for-infrastructure-automation)
8. [Headless Mode and Scheduled Tasks](#8-headless-mode-and-scheduled-tasks)
9. [Homelab-Specific Setup](#9-homelab-specific-setup)
10. [Security Best Practices](#10-security-best-practices)
11. [Real-World Use Cases](#11-real-world-use-cases)
12. [Limitations and Gotchas](#12-limitations-and-gotchas)

---

## 1. The Architecture: Local PC → SSH → Server

The recommended setup for homelab and server management is straightforward: Claude Code runs on your local machine (Mac, Windows/WSL2, or Linux), and manages remote servers by running SSH commands through Bash.

```
Your Local Machine (Mac / Windows+WSL2 / Linux)
  └── Claude Code (installed via npm)
        ├── ~/.ssh/config (defines server aliases and keys)
        ├── CLAUDE.md (infrastructure docs Claude reads at start)
        ├── .claude/commands/sysadmin.md (agent skill)
        └── Bash tool → ssh server1 "docker ps"
                       → ssh server2 "systemctl status nginx"
                       → ssh nas "cat /var/log/syslog | tail -50"
```

**Why this approach over installing Claude Code on the server:**

- **One install, many servers.** Your local machine manages all servers through SSH. No need to install Claude Code on every box.
- **Context stays local.** Your CLAUDE.md, agent skills, and infrastructure docs live in your workspace. They don't need to exist on the server.
- **Network access.** Your local machine already has SSH keys, VPN/Tailscale connectivity, and browser access for verifying changes.
- **Safety.** If Claude Code runs amok, it's running SSH commands remotely, not executing directly as root on the server.

### Prerequisites

- **Claude Code installed locally**: `npm install -g @anthropic-ai/claude-code`
- **Node.js 22 LTS** on your local machine
- **SSH key pair** (ed25519 recommended)
- **SSH access** to your servers with key-based auth

[source: [claudefa.st VPS guide](https://claudefa.st/blog/guide/development/infraops-vps-guide)]

---

## 2. SSH Setup: Keys, Config, and Access

### Generate a Dedicated Key

Use a dedicated key for Claude Code operations so you can revoke it independently:

```bash
# Generate an ed25519 key (strongest, shortest)
ssh-keygen -t ed25519 -C "claude-code-ops" -f ~/.ssh/claude_ops

# Set correct permissions
chmod 600 ~/.ssh/claude_ops
chmod 644 ~/.ssh/claude_ops.pub

# Copy to each server
ssh-copy-id -i ~/.ssh/claude_ops.pub deploy@your-server
```

**No passphrase** on this key (Claude Code can't enter passphrases interactively). Compensate with restrictive file permissions and limiting which servers accept it.

### SSH Config for Clean Access

Add entries to `~/.ssh/config` so Claude Code references servers by alias instead of IPs:

```
# Homelab Docker host
Host docker-host
    HostName 192.168.86.20
    User pete
    IdentityFile ~/.ssh/claude_ops
    IdentitiesOnly yes
    ServerAliveInterval 60
    ServerAliveCountMax 3

# Homelab NAS
Host nas
    HostName 192.168.86.40
    User pete
    IdentityFile ~/.ssh/claude_ops
    IdentitiesOnly yes

# Production VPS (non-standard port)
Host prod-web
    HostName your-vps.example.com
    Port 2222
    User pete
    IdentityFile ~/.ssh/claude_ops
    IdentitiesOnly yes

# Tailscale access (when off-LAN)
Host docker-host-ts
    HostName 100.67.119.80
    User pete
    IdentityFile ~/.ssh/claude_ops
    IdentitiesOnly yes
    ServerAliveInterval 60
```

Key settings explained:
- `IdentitiesOnly yes` prevents SSH from trying other keys, avoiding auth failures
- `ServerAliveInterval 60` sends keepalives every 60 seconds to prevent drops during long operations
- Non-standard ports go in the config, not in every command

[source: [Joe Njenga on Medium](https://medium.com/@joe.njenga/how-i-use-claude-code-ssh-to-connect-to-any-remote-server-like-a-pro-f08ed35e5569)]

### Server-Side: Create a Dedicated User

Don't give Claude Code root access. Create a dedicated user with targeted sudo:

```bash
# On the server
sudo adduser deploy
sudo usermod -aG docker deploy  # Docker access without sudo

# Grant targeted sudo (only specific commands, no password)
sudo visudo
# Add this line:
# deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl status *, /usr/bin/systemctl restart *, /usr/bin/journalctl *, /usr/sbin/nginx -t, /usr/sbin/nginx -s reload
```

### Tailscale / VPN Access

If your homelab uses Tailscale, Claude Code just needs your local machine to be on the Tailscale network. SSH configs then use Tailscale IPs. This works seamlessly because Claude Code runs Bash commands in your local shell context, which inherits your network connectivity.

### Test It

Before asking Claude Code to manage anything, verify SSH works:

```bash
ssh docker-host "hostname && uptime && docker ps --format 'table {{.Names}}\t{{.Status}}'"
```

If that works from your terminal, Claude Code can do it too.

[source: [midgarcorp.cc SSH tunnels guide](https://midgarcorp.cc/blog/claude-code-remote-ssh-tunnel/)]

---

## 3. The Infrastructure as Markdown Pattern

This is the pattern that makes Claude Code genuinely useful for sysadmin work rather than just a fancy shell wrapper. Maintain markdown files that describe your infrastructure, and Claude Code reads them at session start.

### Why This Works

The documentation that Claude Code generates while doing work is the same documentation it reads to do the next task. Every configuration choice, installed service, and resolved issue is written down in plain text. If something breaks and you need to bring in a human sysadmin who has never seen your setup, they can read the same markdown files. [source: [martinalderson.com](https://martinalderson.com/posts/how-i-use-claude-code-to-manage-sysadmin-tasks/)]

### Recommended File Structure

The key insight: split your infrastructure docs into a **main context file** (always loaded) and **reference files** (loaded on demand). This saves context window space while keeping deep documentation available.

```
infra/
  CLAUDE.md              # Main entry point, auto-loaded at session start
  connections.md         # SSH aliases, IPs, ports, users
  infrastructure.md      # Full server inventory: hardware, OS, specs, roles
  containers.md          # Every container: name, image, ports, volumes, purpose
  known-issues.md        # Problems encountered and their solutions
  reference/
    common-tasks.md      # SOPs: container management, updates, restarts
    troubleshooting.md   # Diagnostic playbooks: OOM, disk full, network issues
    secrets-monitoring.md # Where secrets live, monitoring thresholds, health checks
  .claude/
    commands/
      sysadmin.md        # Your sysadmin agent skill (see Section 4)
```

### Why Split Into Reference Files?

Your agent reads the main CLAUDE.md every session. But a 500-line troubleshooting guide wastes context window if you're just checking container health. Split docs by task type and load them on demand:

```markdown
## Reference Files

Load these as needed to save context window:

| File | Load When... |
|------|-------------|
| reference/common-tasks.md | Performing operational tasks: container management, updates, config changes |
| reference/troubleshooting.md | Diagnosing issues: container failures, resource exhaustion, network problems |
| reference/secrets-monitoring.md | Auditing secrets, reviewing monitoring thresholds, health checks |
```

The agent instruction says "Load reference files only when the task requires them." Claude reads the table, understands the categories, and only loads the relevant file for the current task.

### Go Deep on Server Documentation

Don't just list hostnames and IPs. Document everything the agent might need to make decisions:

```markdown
# Infrastructure Overview

## Docker Host (docker-host)
| Property | Value |
|----------|-------|
| OS | Ubuntu 24.04 LTS |
| CPU | Intel i7-13700F (24 threads) |
| RAM | 32GB DDR5 |
| Storage | 256GB NVMe (LVM) |
| Role | Primary container host |

### Containers
| Container | Image | Ports | Purpose |
|-----------|-------|-------|---------|
| caddy | caddy:2-alpine | 80, 443 | Reverse proxy |
| portainer | portainer/portainer-ce | 9443 | Container management |
| uptime-kuma | louislam/uptime-kuma | 3001 | Service monitoring |
| plex | linuxserver/plex | 32400 | Media server |
| n8n | n8nio/n8n | 5678 | Workflow automation |

Compose files: /home/deploy/docker/[service-name]/

### Access URLs
- Portainer: https://docker-host:9443
- Uptime Kuma: http://docker-host:3001
- Plex: http://docker-host:32400/web
```

This level of detail means the agent never has to discover your infrastructure by probing. It knows what's running, where the compose files live, and how to reach every service. The documentation is the agent's memory between sessions.

### Example CLAUDE.md

```markdown
# Infrastructure Management

## Server Inventory
See infrastructure.md for full hardware specs and roles.

## Container Inventory
See containers.md for the full container grid with ports and purposes.

## SSH Access
All servers are accessible via SSH aliases in ~/.ssh/config.
Use `ssh docker-host` for the container host, `ssh nas` for the NAS.

## Agent Routing
| Trigger Topics | Skill |
|---|---|
| Homelab, containers, Docker, servers | `/sysadmin` |
| Production servers, deploys, websites | `/keeper` |

## Rules
- NEVER modify production configs without creating a backup first
- ALWAYS verify service status after any config change
- Docker Compose changes must be tested with `docker compose config` first
- After any Nginx edit, run `nginx -t` before reloading
- Log significant changes to known-issues.md
```

---

## 4. Building a Sysadmin Agent Skill

A sysadmin agent is a Claude Code skill (a markdown file in `.claude/commands/`) that gives Claude a specialized persona, infrastructure knowledge, safety rules, and structured output format. When invoked, it spawns a subagent focused entirely on infrastructure work.

This is the most important section of the guide. A well-built agent turns Claude Code from "an AI that can run SSH commands" into a reliable operations tool.

### Anatomy of a Sysadmin Agent

The skill file lives at `.claude/commands/sysadmin.md` and contains:

1. **Trigger and description** (YAML frontmatter)
2. **Persona and style** (how the agent communicates)
3. **Infrastructure context** (what servers exist, how to reach them)
4. **Reference file loading** (progressive disclosure of detailed docs)
5. **Safety rules and restrictions**
6. **Output format** (structured, scannable reports)
7. **Error handling contract**
8. **The `$ARGUMENTS` variable** (passes the user's task to the agent)

### Complete Example: A Homelab Sysadmin Agent

```markdown
---
description: Sysadmin - homelab infrastructure operator. Manages containers, servers, Docker, and networking.
version: "1.0"
---

You are the Sysadmin agent, a dedicated infrastructure operator.

Say "Sysadmin online. Checking systems..." then spawn a Task agent
(subagent_type="general-purpose") with the following instructions:

You are the Sysadmin agent, a dedicated homelab infrastructure operator.

First, read `infra/connections.md` for SSH access details and server inventory.
Load reference files only when the task requires them:
- Container inventory: `infra/containers.md`
- Known issues: `infra/known-issues.md`
- Backup procedures: `infra/backups.md`

Your style: Direct, efficient, technical. Start responses with "Sysadmin here."

## Infrastructure

- docker-host (Ubuntu, Docker host): SSH alias `docker-host`
- nas (Synology DS220+): SSH alias `nas`
- prod-web (Hetzner VPS): SSH alias `prod-web`

For full connection details, see `infra/connections.md`.

## Safety Rules

- NEVER run destructive commands without showing the user first
- NEVER delete volumes, images, or data without explicit approval
- ALWAYS verify changes after making them (restart → check status)
- ALWAYS create backups before modifying production configs
- If a command requires sudo and hangs, report it instead of retrying

## Output Format

Structure every response using these sections. Omit sections that don't apply:

## Status
SERVICE/HOST     | STATUS | DETAIL
-----------------|--------|---------------------------
docker-host      | UP     | load 0.2, 8GB free
container-name   | DOWN   | exited 137, OOM killed
...

## Action Taken
[What was done, with exact commands run. Or "No action needed"]

## Issues Found
[Bulleted list with severity (Critical/Warning/Info), or "None"]

## Error Handling

- If a tool call fails, retry once. If it fails again, report the exact error.
- Never claim a task is complete if any step produced an error.
- If blocked by access or permissions, report what succeeded and what failed.

Task from user: $ARGUMENTS
```

### How It Works

When the user types `/sysadmin check if all containers are healthy`, Claude Code:

1. Reads the skill file
2. Spawns a subagent with the full instructions
3. The subagent reads `connections.md` to learn SSH details
4. Runs `ssh docker-host "docker ps -a --format '{{.Names}}\t{{.Status}}'"` 
5. Parses the output and reports in the structured format

### Key Design Decisions

**Why a subagent?** The sysadmin agent runs as a spawned subagent (`Task agent`), not inline. This protects the main conversation context from being filled with SSH output and container logs. The subagent does its work and returns a clean summary.

**Why progressive disclosure?** The agent reads `connections.md` first (small, always needed), and only loads troubleshooting or secrets docs when relevant. This saves context window space. Skills use progressive disclosure: Claude reads just the description at startup, loading the full content only when relevant. [source: [pulumi.com DevOps skills blog](https://www.pulumi.com/blog/top-8-claude-skills-devops-2026/)]

**Why structured output?** The Status/Action/Issues format makes reports scannable. You see the grid first, then what happened, then what needs attention. This is better than a wall of text.

**Why `$ARGUMENTS`?** This is the magic variable. Whatever the user types after `/sysadmin` gets injected here. The agent receives the task directly without needing to parse conversation history.

### Advanced Pattern: State Tracking with a Scratchpad

For multi-step operations (rolling updates, migrations, incident response), the agent can lose track of where it is if the context window compacts or the session restarts. Solve this with a scratchpad file:

```markdown
## State Tracking

Before multi-step work, write to `infra/execution/.scratchpad.md`:
- Task description
- Planned steps
- Current step and result

After each significant action, update with the result.
On startup, check `.scratchpad.md` for incomplete tasks and offer to resume.
```

Add this to your agent skill, and the agent writes its plan before starting, updates progress as it goes, and can pick up where it left off if interrupted. This is particularly valuable for operations that touch multiple servers or containers in sequence.

### Advanced Pattern: Raw Data First, Commentary Second

A common failure mode with AI agents is getting summaries instead of data. Your agent should show the actual command output first, then add analysis:

```markdown
## Output Rules

1. Start with "Sysadmin here." + brief intro
2. Show RAW DATA first (actual command output, tables, metrics)
3. Add commentary after the data
4. Never summarize instead of showing data
```

This matters because you need to verify what the agent saw. If it says "all containers healthy" but you can't see the actual `docker ps` output, you're trusting the AI's interpretation. Show the data, then the interpretation.

### Advanced Pattern: Persona for Consistency

Giving the agent a distinct persona (name, speaking style, response opener) sounds cosmetic but serves a practical purpose: it makes output instantly recognizable and keeps the response format consistent across sessions. When every response starts with "Sysadmin here." followed by a status grid, you can scan it in seconds. Without a persona, the same agent produces different output structures depending on the prompt.

### Agent Routing (Multiple Agents)

If you have multiple servers or domains (homelab, production, NAS), you can create separate agents for each, then route automatically from CLAUDE.md:

```markdown
## Agent Routing

| Trigger Topics | Skill |
|---|---|
| Homelab, containers, Docker, servers | `/sysadmin` |
| Production servers, deploys, WordPress | `/keeper` |
| NAS, backups, storage, Plex | `/media` |
| Infrastructure health, SSL, security | `/sentinel` |
```

This lets you say "check the Docker containers" and Claude routes to the right agent without you specifying which server. Each agent knows its domain, its servers, and its safety rules. The routing table in CLAUDE.md is the dispatch layer.

For a mature setup, you might have 5-10 agents covering different infrastructure domains: one for the homelab Docker host, one for production servers, one for media/NAS, one for security auditing, one for network/DNS. Each agent is a focused expert with its own context docs and safety constraints.

### What Makes a Sysadmin Agent Effective (Lessons from Production Use)

After running a sysadmin agent against 35+ containers across multiple servers, these are the patterns that matter most:

**Deep infrastructure context beats clever prompting.** An agent that knows every container name, port, compose file location, and access URL will outperform a generic agent with a brilliant system prompt. Document everything. The agent's value comes from knowing what healthy looks like, not from the SSH commands themselves.

**Reference file splitting by task type.** One file for common operations (restarts, updates, health checks), another for troubleshooting (OOM kills, disk full, network issues), another for secrets and monitoring. The agent loads only what the current task requires.

**Scratchpad persistence across interruptions.** Multi-step operations (rolling updates, disk cleanup across servers) need a written plan the agent can resume from. Without this, a context compaction or session restart means starting over.

**Structured output is non-negotiable.** Status grid, action taken, issues found. Every time, same format. This is what makes the agent's output trustworthy and scannable at a glance.

**Show raw data, not summaries.** The human needs to see what the agent saw. "All healthy" is useless without the `docker ps` output that proves it.

---

## 5. Docker and Container Management

### Direct Docker Management via SSH

With the local PC → SSH architecture, Docker management looks like:

```bash
# Claude runs these through the Bash tool
ssh docker-host "docker ps -a --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'"
ssh docker-host "docker logs --tail 50 plex"
ssh docker-host "cd /home/pete/docker/plex && docker compose restart"
ssh docker-host "docker system df"
```

Document your container layout in `containers.md` so the agent knows what's running:

```markdown
## Docker Services (docker-host)

| Container     | Image                      | Ports    | Purpose           |
|---------------|----------------------------|----------|-------------------|
| caddy         | caddy:2-alpine             | 80, 443  | Reverse proxy     |
| portainer     | portainer/portainer-ce     | 9443     | Container mgmt    |
| plex          | linuxserver/plex           | host     | Media server      |
| uptime-kuma   | louislam/uptime-kuma       | 3001     | Monitoring        |

Compose files are in /home/pete/docker/[service-name]/
```

### Docker Best Practices for Container Definitions

When your agent creates or modifies Docker configurations, enforce these standards:

1. **Multi-stage Dockerfiles**: Builder stage installs deps, runtime stage copies only installed packages
2. **No secrets in images**: Config via env_file or runtime env vars only
3. **Named volumes**: Declared in top-level `volumes:` block, not bind mounts
4. **User-defined bridge networks**: Every compose file declares a named network
5. **Declarative compose**: Full stack defined in docker-compose.yml with restart policy

### Optional: MCP for Structured API Access

If you run services with REST APIs (e.g., Portainer for container management), you can build or connect MCP servers that give Claude typed tool calls instead of parsing shell output. MCP configuration lives in `~/.claude.json` (user-level) or `.mcp.json` (project-level).

Most homelab setups use SSH as the primary method. MCP is a nice-to-have for services where a structured API is cleaner than parsing command output.

[source: [code.claude.com/docs/en/mcp](https://code.claude.com/docs/en/mcp)]

---

## 6. Permission Configuration for Sysadmin Work

### Permission System Overview

Claude Code uses a tiered permission system. Rules are evaluated in order: **deny → ask → allow**. The first matching rule wins.

[source: [code.claude.com/docs/en/permissions](https://code.claude.com/docs/en/permissions)]

### Recommended Sysadmin Permissions

Create `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(ssh *)",
      "Bash(scp *)",
      "Bash(rsync *)",
      "Bash(docker ps *)",
      "Bash(docker logs *)",
      "Bash(docker inspect *)",
      "Bash(docker stats *)",
      "Bash(docker compose ps *)",
      "Bash(docker compose logs *)",
      "Bash(systemctl status *)",
      "Bash(journalctl *)",
      "Bash(df *)",
      "Bash(free *)",
      "Bash(uptime)",
      "Bash(ping *)",
      "Bash(curl -s *)",
      "Bash(git *)",
      "Read"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(dd *)",
      "Bash(mkfs *)",
      "Bash(fdisk *)",
      "Bash(shutdown *)",
      "Bash(reboot *)"
    ]
  }
}
```

This allows all read/status operations freely, blocks obviously dangerous commands, and prompts for everything else (docker compose up, service restarts, file edits).

### Permission Modes

| Mode | Description | Sysadmin Use Case |
|------|-------------|-------------------|
| `default` | Prompts for permission on first use | Day-to-day interactive work |
| `plan` | Read-only, no modifications | Auditing and reviewing infrastructure |
| `bypassPermissions` | Skips prompts (containers/VMs only) | Devcontainer automation |

[source: [code.claude.com/docs/en/permissions](https://code.claude.com/docs/en/permissions)]

---

## 7. Hooks for Infrastructure Automation

Hooks are user-defined shell commands that execute at specific points in Claude Code's lifecycle. They provide deterministic control, ensuring certain actions always happen.

[source: [code.claude.com/docs/en/hooks-guide](https://code.claude.com/docs/en/hooks-guide)]

### Block Dangerous Commands

Create `.claude/hooks/protect-infra.sh`:

```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

BLOCKED_PATTERNS=(
  "rm -rf /"
  "DROP DATABASE"
  "> /dev/sda"
  "chmod -R 777"
  ":(){ :|:& };:"
)

for pattern in "${BLOCKED_PATTERNS[@]}"; do
  if echo "$COMMAND" | grep -qi "$pattern"; then
    echo "BLOCKED: Command matches dangerous pattern '$pattern'" >&2
    exit 2
  fi
done

exit 0
```

Register in `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/protect-infra.sh"
          }
        ]
      }
    ]
  }
}
```

### Log All Commands for Audit

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.command' >> ~/.claude/infra-command-log.txt"
          }
        ]
      }
    ]
  }
}
```

### Load Infrastructure Context at Session Start

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"Git: $(git branch --show-current 2>/dev/null) | Modified: $(git status --porcelain 2>/dev/null | wc -l | tr -d ' ')\""
          }
        ]
      }
    ]
  }
}
```

[source: [code.claude.com/docs/en/hooks-guide](https://code.claude.com/docs/en/hooks-guide)]

---

## 8. Headless Mode and Scheduled Tasks

### Headless Mode (-p flag)

The `-p` flag runs Claude Code non-interactively for automation:

```bash
# Basic health check
claude -p "SSH into docker-host and check all container health" \
  --allowedTools "Bash(ssh *)" \
  --max-turns 10 \
  --max-budget-usd 0.25

# JSON output for scripting
claude -p "List all running containers" \
  --output-format json \
  --allowedTools "Bash(ssh *)"

# Bare mode for fastest startup (skips hooks, plugins, MCP, CLAUDE.md)
claude --bare -p "What's the disk usage on docker-host?" \
  --allowedTools "Bash(ssh *)"
```

Key flags: `--allowedTools` (whitelist), `--max-turns` (step limit), `--max-budget-usd` (cost cap), `--bare` (skip auto-discovery).

[source: [code.claude.com/docs/en/headless](https://code.claude.com/docs/en/headless)]

### Cron Integration

```bash
# Health check every hour
0 * * * * deploy ANTHROPIC_API_KEY=sk-ant-... /usr/local/bin/claude --bare -p \
  "SSH into docker-host. Check: 1) all containers running, 2) disk under 80%, 3) no OOM kills in dmesg. Write issues to /tmp/health-report.txt" \
  --allowedTools "Bash(ssh *),Write" --max-turns 15 --max-budget-usd 0.25 \
  >> /var/log/claude-health.log 2>&1
```

### Session-Scoped Scheduling with /loop

```
/loop 5m check if the deployment finished
/loop 30m check container health on all servers
/loop 1h disk space check, alert if anything above 85%
```

[source: [code.claude.com/docs/en/scheduled-tasks](https://code.claude.com/docs/en/scheduled-tasks)]

---

## 9. Homelab-Specific Setup

### Recommended Architecture

```
Your Workstation (Mac / WSL2)
  └── Claude Code + CLAUDE.md + Agent Skills
        ├── SSH → Docker Host (containers, compose stacks)
        ├── SSH → NAS (storage, media services)
        └── SSH → Production VPS (websites, deploys)
```

---

## 10. Security Best Practices

### The Golden Rules

1. **Never give Claude Code root access.** Create a dedicated user with targeted sudo.
2. **Use SSH keys, never passwords.** Passphrase-less keys only on trusted machines.
3. **Separate read and write permissions.** Allow reading freely; require prompts for modifications.
4. **Audit everything.** Use PostToolUse hooks to log all commands.
5. **Limit blast radius.** Use `--max-turns` and `--max-budget-usd` for unattended runs.

### Disable Password Auth on Servers

```bash
# /etc/ssh/sshd_config
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no

sudo systemctl restart sshd
```

### Firewall and Fail2ban

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable

sudo apt install fail2ban
sudo systemctl enable fail2ban
```

### MCP Server Security

MCP servers that return external content carry prompt injection risk. Keep infrastructure MCP servers on your internal network and treat data from internet-facing MCP servers as untrusted.

[source: [code.claude.com/docs/en/security](https://code.claude.com/docs/en/security)]

---

## 11. Real-World Use Cases

### Server Health Monitoring

```
"SSH into all three servers and build a health report:
CPU, memory, disk space, container status, errors in the last hour of syslog"
```

### Docker Stack Updates

```
"Update the media stack on docker-host:
1. Pull new images
2. Take a compose snapshot
3. Bring up with new images
4. Verify all containers are healthy
5. Report what changed"
```

### Incident Response

```
"The Plex container is unhealthy. Investigate: check container logs,
host resources, disk space, Docker daemon logs. Fix the issue and
verify Plex is accessible."
```

### Configuration Drift Detection

```
"Compare the current Nginx config on prod-web with what's documented
in nginx-sites.md. Report discrepancies and update the docs."
```

### Disk Cleanup

```
"Check Docker disk usage on docker-host. Clean up build cache older
than 7 days, remove dangling images, and report how much space was freed."
```

These use cases have been verified working on Ubuntu, Debian, and Rocky Linux servers. [source: [computingforgeeks.com](https://computingforgeeks.com/claude-code-ssh-server-management/)]

---

## 12. Limitations and Gotchas

### What Claude Code Cannot Do

- **Interactive terminals:** `top`, `htop`, `vim`, `less` don't work. Use non-interactive alternatives.
- **Real-time monitoring:** Can't watch a live log stream. Use `/loop` for periodic checks.
- **Long-running processes:** SSH sessions that take more than a few minutes may time out. Break large tasks into steps.

### Common Pitfalls

1. **Sudo prompts:** If a command needs sudo and requires a password, Claude Code hangs. Use NOPASSWD for specific commands.
2. **Context overflow:** Large log dumps fill the context window. Tell Claude to use `tail -n 50` or `head -n 100`.
3. **SSH agent forwarding:** Must be configured in `~/.ssh/config`, not assumed.
4. **Docker in sandbox:** Docker commands are incompatible with the sandbox. Add to `excludedCommands`. [source: [code.claude.com/docs/en/sandboxing](https://code.claude.com/docs/en/sandboxing)]

### Cost Awareness

Every interaction uses API tokens. For automation:
- Use `--max-budget-usd` to cap spending
- Use `--max-turns` to limit steps
- Use `--bare` mode for simple checks
- Prefer targeted prompts over open-ended ones

---

## Sources

### Official Anthropic Documentation
- [Permissions](https://code.claude.com/docs/en/permissions)
- [Sandboxing](https://code.claude.com/docs/en/sandboxing)
- [Hooks Guide](https://code.claude.com/docs/en/hooks-guide)
- [MCP Integration](https://code.claude.com/docs/en/mcp)
- [Headless / Agent SDK CLI](https://code.claude.com/docs/en/headless)
- [Scheduled Tasks](https://code.claude.com/docs/en/scheduled-tasks)
- [Security](https://code.claude.com/docs/en/security)

### Community Guides and Tutorials
- [Manage Servers with Claude Code via SSH (ComputingForGeeks)](https://computingforgeeks.com/claude-code-ssh-server-management/)
- [How I Use Claude Code to Manage Sysadmin Tasks (Martin Alderson)](https://martinalderson.com/posts/how-i-use-claude-code-to-manage-sysadmin-tasks/)
- [Claude Code SSH to Connect to Any Remote Server (Joe Njenga, Medium)](https://medium.com/@joe.njenga/how-i-use-claude-code-ssh-to-connect-to-any-remote-server-like-a-pro-f08ed35e5569)
- [Claude Code VPS Setup Guide (claudefa.st)](https://claudefa.st/blog/guide/development/infraops-vps-guide)
- [Control Claude Code Remotely with SSH Tunnels (midgarcorp.cc)](https://midgarcorp.cc/blog/claude-code-remote-ssh-tunnel/)
- [The Claude Skills I Actually Use for DevOps (Pulumi Blog)](https://www.pulumi.com/blog/top-8-claude-skills-devops-2026/)
- [Claude Code as Sysadmin: What It Actually Looks Like (Marc Bara, Medium)](https://medium.com/@marc.bara.iniesta/claude-code-as-sysadmin-what-it-actually-looks-like-969179b995f3)

## Update History

- **2026-04-09 2:00 PM ET**: Enhanced Sections 3 and 4 with production-tested patterns: reference file splitting by task type, deep server documentation, scratchpad state tracking for multi-step ops, raw-data-first output rules, persona for consistency, and lessons from running a sysadmin agent against 35+ containers. All patterns kept generic and environment-agnostic.
- **2026-04-09 1:15 PM ET**: Restructured around local PC → SSH workflow. Added Section 4: Building a Sysadmin Agent Skill with complete skill file template, design decisions, agent routing, and real-world example. Trimmed sections on installing Claude Code on the server. Removed sandboxing as a standalone section (folded into gotchas). Added Docker best practices reference to container management section.
- **2026-04-09 12:30 PM ET**: Initial report.

## How This Report Was Generated

This report was researched and written by the Research agent using SearXNG deep search across multiple engines, verified by reading primary sources via WebFetch, and cross-referenced with Anthropic's official documentation at code.claude.com. Community claims were verified against multiple sources before inclusion.
