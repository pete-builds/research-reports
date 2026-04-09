# Claude Code as a Sysadmin Tool: Comprehensive Practical Guide

**Date:** April 9, 2026 (Eastern Time)
**Researcher:** Research Agent
**Classification:** Technical Reference / Practical Guide

---

## Executive Summary

Claude Code has matured into a capable sysadmin assistant that can manage servers, containers, and infrastructure from the terminal. This guide covers every practical aspect: SSH setup for remote server management, Docker and container orchestration, MCP integrations for homelab services, security hardening, headless automation, and the "infrastructure as markdown" pattern that makes Claude Code retain knowledge across sessions.

The core insight: Claude Code is not just an AI that runs shell commands. It understands what command output means and acts on it. You can say "the web server on 10.0.1.5 is down, fix it" and it checks the service status, reads the error log, identifies the config problem, patches it, re-validates, and restarts the service. [source: [computingforgeeks.com](https://computingforgeeks.com/claude-code-ssh-server-management/)]

**Confidence level:** High for all official documentation claims (sourced from code.claude.com/docs). Medium for community patterns and third-party tools (sourced from blog posts, GitHub repos, and tutorials, all cited inline).

---

## Table of Contents

1. [Prerequisites and Installation](#1-prerequisites-and-installation)
2. [SSH Setup for Remote Server Management](#2-ssh-setup-for-remote-server-management)
3. [The Infrastructure as Markdown Pattern](#3-the-infrastructure-as-markdown-pattern)
4. [Docker and Container Management](#4-docker-and-container-management)
5. [MCP Servers for Infrastructure](#5-mcp-servers-for-infrastructure)
6. [Permission Configuration for Sysadmin Work](#6-permission-configuration-for-sysadmin-work)
7. [Sandboxing for Safe Server Operations](#7-sandboxing-for-safe-server-operations)
8. [Hooks for Infrastructure Automation](#8-hooks-for-infrastructure-automation)
9. [Headless Mode and Scheduled Tasks](#9-headless-mode-and-scheduled-tasks)
10. [Homelab-Specific Setup](#10-homelab-specific-setup)
11. [Security Best Practices](#11-security-best-practices)
12. [Real-World Use Cases](#12-real-world-use-cases)
13. [Limitations and Gotchas](#13-limitations-and-gotchas)

---

## 1. Prerequisites and Installation

### Minimum Server Requirements

For running Claude Code directly on a remote server (VPS, homelab node, etc.):

- **CPU:** 2 vCPUs minimum, 4 preferred
- **RAM:** 4GB minimum, 8GB recommended
- **Storage:** 40GB SSD
- **OS:** Ubuntu 22.04+ LTS, Debian 12+, Rocky Linux 10.1+, or any modern Linux with Node.js support
- **Node.js:** v22 LTS

[source: [claudefa.st VPS guide](https://claudefa.st/blog/guide/development/infraops-vps-guide)]

### Installation on a Remote Server

```bash
# Install Node.js 22 LTS
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install Claude Code globally
npm install -g @anthropic-ai/claude-code

# Verify installation
claude --version
```

### Authentication

Two options:

1. **API key** (recommended for servers): Set `ANTHROPIC_API_KEY` in your environment or shell profile.
2. **OAuth** (for Pro/Max subscribers): Run `claude` and follow the auth URL. Not ideal for headless servers.

```bash
# Add to ~/.bashrc or ~/.zshrc for persistence
export ANTHROPIC_API_KEY="sk-ant-..."
```

Never commit API keys to version control. Use environment variables or a secrets manager.

[source: [claudefa.st VPS guide](https://claudefa.st/blog/guide/development/infraops-vps-guide)]

---

## 2. SSH Setup for Remote Server Management

There are two fundamentally different approaches to using Claude Code with remote servers:

### Approach A: Install Claude Code on the Remote Server

SSH into the server, install Claude Code there, and run it directly. This gives Claude full access to the server's filesystem and services.

```bash
# From your local machine
ssh deploy@your-server

# On the remote server
claude
```

For persistent sessions that survive SSH disconnects, use tmux or screen:

```bash
ssh deploy@your-server
tmux new -s claude-session
claude
# Detach with Ctrl+B, D
# Reattach later with: tmux attach -t claude-session
```

### Approach B: Run Claude Code Locally, SSH into Servers

Run Claude Code on your local machine and let it SSH into remote servers via Bash commands. This is the more common homelab pattern.

#### SSH Key Setup

Generate a dedicated key for Claude Code operations:

```bash
# Generate an ed25519 key (recommended)
ssh-keygen -t ed25519 -C "claude-code-ops" -f ~/.ssh/claude_ops

# Set correct permissions
chmod 600 ~/.ssh/claude_ops
chmod 644 ~/.ssh/claude_ops.pub

# Copy to your server
ssh-copy-id -i ~/.ssh/claude_ops.pub deploy@your-server
```

#### SSH Config for Clean Access

Add entries to `~/.ssh/config` so Claude Code can reference servers by alias:

```
Host homelab-proxmox
    HostName 10.0.1.10
    User root
    IdentityFile ~/.ssh/claude_ops
    IdentitiesOnly yes
    ServerAliveInterval 60
    ServerAliveCountMax 3

Host homelab-docker
    HostName 10.0.1.20
    User deploy
    IdentityFile ~/.ssh/claude_ops
    IdentitiesOnly yes
    ServerAliveInterval 60

Host prod-web
    HostName your-vps.example.com
    Port 2222
    User deploy
    IdentityFile ~/.ssh/claude_ops
    IdentitiesOnly yes
    ProxyJump bastion
```

Key settings explained:
- `IdentitiesOnly yes` prevents SSH from trying other keys, avoiding auth failures
- `ServerAliveInterval 60` sends keepalives every 60 seconds to prevent drops
- `ProxyJump bastion` routes through a jump host for production servers

[source: [Joe Njenga on Medium](https://medium.com/@joe.njenga/how-i-use-claude-code-ssh-to-connect-to-any-remote-server-like-a-pro-f08ed35e5569), [midgarcorp.cc SSH tunnels guide](https://midgarcorp.cc/blog/claude-code-remote-ssh-tunnel/)]

#### Non-Interactive Remote Commands

From your local machine, you can run Claude Code on a remote server in one shot:

```bash
ssh deploy@your-server "cd /opt/apps/myproject && claude -p 'Run the test suite and fix any failures'"
```

[source: [claudefa.st VPS guide](https://claudefa.st/blog/guide/development/infraops-vps-guide)]

### Approach C: MCP-Based SSH

MCP servers like [claude-ssh-server](https://github.com/jasondsmith72/claude-ssh-server) and [adremote-mcp](https://github.com/nqmn/adremote-mcp) let Claude connect to SSH servers on-demand using just an IP, username, and password/key. This is useful when you need to manage many servers without pre-configuring each one.

[source: [GitHub jasondsmith72/claude-ssh-server](https://github.com/jasondsmith72/claude-ssh-server), [GitHub nqmn/adremote-mcp](https://github.com/nqmn/adremote-mcp)]

---

## 3. The Infrastructure as Markdown Pattern

This is the pattern that makes Claude Code genuinely useful for sysadmin work rather than just a fancy shell wrapper. The idea: maintain markdown files that describe your infrastructure, and Claude Code reads them at session start, giving it full context without rediscovering everything each time.

### Recommended File Structure

Create a directory (or repo) for your infrastructure documentation:

```
infra/
  CLAUDE.md              # Main entry point, references other files
  infrastructure.md      # Hardware, IPs, OS versions, installed services
  docker-services.md     # Each container, its purpose, ports, volumes
  nginx-sites.md         # Reverse proxy config per domain
  backup-procedures.md   # What's backed up, where, how to restore
  known-issues.md        # Problems encountered and their solutions
  ssh-config.md          # How to reach each server (references ~/.ssh/config aliases)
  runbooks/
    deploy-app.md        # Step-by-step deploy procedure
    rotate-certs.md      # SSL certificate renewal
    disaster-recovery.md # Full recovery procedure
```

### Example CLAUDE.md for Infrastructure

```markdown
# Infrastructure Management

This directory contains documentation for managing our homelab infrastructure.

## Server Inventory
See infrastructure.md for the full server inventory with IPs and roles.

## SSH Access
All servers are accessible via SSH aliases configured in ~/.ssh/config.
Use `ssh homelab-proxmox` for the hypervisor, `ssh homelab-docker` for the container host.

## Rules
- NEVER modify production configs without creating a backup first
- ALWAYS verify service status after any config change
- When editing Nginx configs, always run `nginx -t` before reloading
- Docker Compose changes must be tested with `docker compose config` first
- Log all significant changes to known-issues.md
```

### Why This Pattern Works

The documentation that Claude Code generates while doing work is the same documentation it reads to do the next task. This is the opposite of how infrastructure knowledge typically degrades. Every configuration choice, installed service, and resolved issue is written down in plain text. [source: [martinalderson.com sysadmin tasks](https://martinalderson.com/posts/how-i-use-claude-code-to-manage-sysadmin-tasks/)]

If something breaks and you need to bring in a human sysadmin who has never seen your setup, they can read the same markdown files and understand the entire environment in minutes. [source: [martinalderson.com](https://martinalderson.com/posts/how-i-use-claude-code-to-manage-sysadmin-tasks/)]

---

## 4. Docker and Container Management

### Direct Docker Management via Bash

Claude Code can manage Docker directly through Bash commands. In a CLAUDE.md, document your container layout:

```markdown
## Docker Services (homelab-docker: 10.0.1.20)

| Container     | Image                      | Ports        | Purpose           |
|---------------|----------------------------|--------------|-------------------|
| nginx-proxy   | jwilder/nginx-proxy        | 80, 443      | Reverse proxy     |
| plex          | plexinc/pms-docker         | 32400        | Media server      |
| portainer     | portainer/portainer-ce     | 9443         | Container mgmt    |
| heimdall      | linuxserver/heimdall       | 8080         | Dashboard         |

All services use Docker Compose. Compose files are in /opt/docker/[service-name]/
```

Then Claude Code can:

```
"Check if all containers on homelab-docker are healthy"
"The Plex container keeps restarting, investigate and fix it"
"Update the Nginx proxy config to add a new subdomain for Grafana"
```

### Running Claude Code Inside Docker

For isolation, you can run Claude Code itself inside a Docker container with your workspace mounted:

```yaml
# docker-compose.yml
services:
  claude:
    image: node:22-slim
    volumes:
      - ./workspace:/workspace
      - ~/.claude:/root/.claude
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    working_dir: /workspace
    command: >
      sh -c "npm install -g @anthropic-ai/claude-code && claude"
    stdin_open: true
    tty: true
```

When Claude Code runs in Docker with your workspace mounted as a volume, it reads files from that mount, executes commands in the container's isolated process space, and writes output back to the volume. The container's ephemeral filesystem is discarded when the session ends. [source: [Docker blog](https://www.docker.com/blog/run-claude-code-with-docker/)]

### Portainer MCP Integration

If you run Portainer, you can connect Claude Code directly to it via MCP tools to list stacks, manage containers, read compose files, and deploy updates without SSH. This provides a structured API layer rather than raw shell commands.

[source: practical experience; Portainer MCP tools exist in the MCP ecosystem]

### Dev Containers for Isolated Work

Anthropic provides an official [devcontainer reference implementation](https://github.com/anthropics/claude-code/tree/main/.devcontainer) with:

- Pre-configured Node.js 20 with development dependencies
- Firewall restricting network access to whitelisted domains only (npm registry, GitHub, Claude API)
- Default-deny network policy
- ZSH with productivity tools
- Session persistence across container restarts

The container's enhanced security measures allow you to run `claude --dangerously-skip-permissions` for unattended operation in trusted environments.

[source: [code.claude.com/docs/en/devcontainer](https://code.claude.com/docs/en/devcontainer)]

---

## 5. MCP Servers for Infrastructure

MCP (Model Context Protocol) servers give Claude Code structured access to your infrastructure tools. Instead of parsing shell output, Claude calls typed API functions and gets structured responses.

### Adding MCP Servers

```bash
# Add via CLI
claude mcp add portainer --transport http https://your-portainer:9443/api

# Add via stdio (local process)
claude mcp add docker-stats --transport stdio -- node /path/to/docker-stats-mcp/index.js
```

MCP configuration lives in `~/.claude.json` (user-level) or `.mcp.json` (project-level).

[source: [code.claude.com/docs/en/mcp](https://code.claude.com/docs/en/mcp)]

### Infrastructure-Relevant MCP Servers

| MCP Server | Purpose | Transport |
|------------|---------|-----------|
| [claude-ssh-server](https://github.com/jasondsmith72/claude-ssh-server) | Ad-hoc SSH connections to any server | stdio |
| [adremote-mcp](https://github.com/nqmn/adremote-mcp) | SSH remote access through MCP | stdio |
| [proxmox-mcp-server](https://github.com/husniadil/proxmox-mcp-server) | Manage Proxmox LXC containers via SSH | stdio |
| [claude-homelab](https://github.com/jmagar/claude-homelab) | Full homelab service management (18 skills) | stdio |
| [runbook-mcp-server](https://github.com/runbookai/runbook-mcp-server) | Execute ops runbooks from Claude | stdio |
| Portainer MCP | Docker stack and container management | http |

[source: GitHub repositories linked above, [mcpmarket.com](https://mcpmarket.com/tools/skills/proxmox-infrastructure-management)]

### The claude-homelab Plugin

The [claude-homelab](https://github.com/jmagar/claude-homelab) project (v1.4.0) is a comprehensive plugin that manages homelab services through agents, commands, hooks, and skills. It covers:

- **Media:** Plex, Overseerr, Radarr, Sonarr, Prowlarr, Tautulli
- **Downloads:** SABnzbd, qBittorrent
- **Infrastructure:** Unraid, UniFi, Tailscale, ZFS, SWAG
- **Utilities:** Linkding, Memos, Paperless-ngx, Gotify

Homelab-specific commands include:
- `/homelab:system-resources` for CPU, RAM, temperature snapshots
- `/homelab:docker-health` for container audits identifying unhealthy instances
- `/homelab:disk-space` for filesystem usage analysis
- `/homelab:zfs-health` for full ZFS pool diagnostics

All credentials consolidate into `~/.claude-homelab/.env` with `chmod 600` permissions.

[source: [GitHub jmagar/claude-homelab](https://github.com/jmagar/claude-homelab)]

---

## 6. Permission Configuration for Sysadmin Work

For sysadmin use, you need a carefully tuned permission setup that allows necessary operations while blocking dangerous ones.

### Permission System Overview

Claude Code uses a tiered permission system:

| Tool type | Example | Approval required |
|-----------|---------|-------------------|
| Read-only | File reads, Grep | No |
| Bash commands | Shell execution | Yes |
| File modification | Edit/write files | Yes |

Rules are evaluated in order: **deny -> ask -> allow**. The first matching rule wins. Deny rules always take precedence.

[source: [code.claude.com/docs/en/permissions](https://code.claude.com/docs/en/permissions)]

### Recommended Sysadmin Permission Configuration

Create `~/.claude/settings.json` for user-level permissions:

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
      "Bash(cat *)",
      "Bash(tail *)",
      "Bash(head *)",
      "Bash(grep *)",
      "Bash(ls *)",
      "Bash(df *)",
      "Bash(free *)",
      "Bash(uptime)",
      "Bash(htop *)",
      "Bash(nginx -t)",
      "Bash(certbot *)",
      "Bash(git *)",
      "Read",
      "Bash(curl -s *)"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(dd *)",
      "Bash(mkfs *)",
      "Bash(fdisk *)",
      "Bash(shutdown *)",
      "Bash(reboot *)",
      "Bash(init *)",
      "Bash(docker system prune *)",
      "Bash(docker volume rm *)"
    ]
  }
}
```

For a production server project, create `.claude/settings.json` in the project:

```json
{
  "permissions": {
    "deny": [
      "Bash(docker compose down *)",
      "Bash(docker stop *)",
      "Bash(docker rm *)",
      "Bash(systemctl stop *)",
      "Bash(systemctl disable *)"
    ]
  }
}
```

### Permission Modes

| Mode | Description | Sysadmin Use Case |
|------|-------------|-------------------|
| `default` | Prompts for permission on first use | Day-to-day interactive work |
| `acceptEdits` | Auto-accepts file edits and common filesystem commands | Editing config files |
| `plan` | Read-only, no modifications | Auditing and reviewing infrastructure |
| `dontAsk` | Auto-denies unless pre-approved | Locked-down automation scripts |
| `bypassPermissions` | Skips prompts (use only in containers/VMs) | Devcontainer-only |

[source: [code.claude.com/docs/en/permissions](https://code.claude.com/docs/en/permissions)]

---

## 7. Sandboxing for Safe Server Operations

Sandboxing provides OS-level enforcement that restricts filesystem and network access for Bash commands. This is critical for sysadmin work where a mistake can have real consequences.

### Enabling Sandboxing

```
/sandbox
```

This opens a menu to choose between:

1. **Auto-allow mode:** Sandboxed commands run without permission prompts. Commands needing access outside the sandbox fall back to normal permission flow.
2. **Regular permissions mode:** All commands go through standard permission flow, even when sandboxed. More control, more prompts.

### Platform Support

- **macOS:** Uses Seatbelt (built-in), works out of the box
- **Linux/WSL2:** Requires `bubblewrap` and `socat`: `sudo apt-get install bubblewrap socat`
- **WSL1:** Not supported

### Configuration for Sysadmin Work

```json
{
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "allowWrite": [
        "~/.ssh/config",
        "/tmp/claude-work",
        "./configs"
      ],
      "denyRead": [
        "~/.ssh/id_*",
        "~/.gnupg"
      ],
      "denyWrite": [
        "/etc",
        "/usr",
        "/var"
      ]
    }
  }
}
```

### Key Sandboxing Principle

Effective sandboxing requires **both** filesystem and network isolation. Without network isolation, a compromised agent could exfiltrate SSH keys. Without filesystem isolation, a compromised agent could backdoor system resources to gain network access.

[source: [code.claude.com/docs/en/sandboxing](https://code.claude.com/docs/en/sandboxing)]

### Sandbox Limitations for Docker

Docker is incompatible with running inside the sandbox. Add it to `excludedCommands`:

```json
{
  "sandbox": {
    "excludedCommands": ["docker *"]
  }
}
```

This forces Docker commands to run outside the sandbox and go through the normal permissions flow.

[source: [code.claude.com/docs/en/sandboxing](https://code.claude.com/docs/en/sandboxing)]

---

## 8. Hooks for Infrastructure Automation

Hooks are user-defined shell commands that execute at specific points in Claude Code's lifecycle. They provide deterministic control, ensuring certain actions always happen rather than relying on the LLM to choose to do them.

[source: [code.claude.com/docs/en/hooks-guide](https://code.claude.com/docs/en/hooks-guide)]

### Sysadmin-Relevant Hook Events

| Event | When it fires | Sysadmin use |
|-------|---------------|--------------|
| `PreToolUse` | Before a tool call executes | Block dangerous commands |
| `PostToolUse` | After a tool call succeeds | Verify config changes, run linters |
| `Notification` | When Claude needs input | Desktop/Slack alerts during long ops |
| `Stop` | When Claude finishes responding | Verify final state of changes |
| `SessionStart` | When a session begins | Load infrastructure context |

### Block Dangerous Server Commands

Create `.claude/hooks/protect-infra.sh`:

```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# Block destructive commands on remote servers
BLOCKED_PATTERNS=(
  "rm -rf /"
  "DROP DATABASE"
  "drop table"
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

Register it in `.claude/settings.json`:

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

### Log All Bash Commands for Audit

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

### Verify Nginx Config After Edits

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "if": "Edit(/etc/nginx/**)",
            "command": "ssh homelab-docker 'nginx -t' || (echo 'Nginx config test failed!' >&2 && exit 2)"
          }
        ]
      }
    ]
  }
}
```

### Desktop Notifications When Operations Complete

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude needs your attention\" with title \"Infrastructure Ops\"'"
          }
        ]
      }
    ]
  }
}
```

### Re-inject Infrastructure Context After Compaction

When Claude's context window fills up, compaction summarizes the conversation and can lose important infrastructure details:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "cat infrastructure.md && echo '---' && cat docker-services.md"
          }
        ]
      }
    ]
  }
}
```

[source: [code.claude.com/docs/en/hooks-guide](https://code.claude.com/docs/en/hooks-guide)]

---

## 9. Headless Mode and Scheduled Tasks

### Headless Mode (-p flag)

The `-p` flag runs Claude Code non-interactively: it processes one prompt, outputs the result, and exits. This makes Claude Code usable in cron jobs, systemd timers, CI/CD pipelines, and any automation that can execute shell commands.

```bash
# Basic usage
claude -p "Check if all Docker containers are healthy on homelab-docker"

# With tool restrictions
claude -p "Check disk space on all servers" \
  --allowedTools "Bash(ssh *),Bash(df *),Read"

# With spending and turn limits
claude -p "Review and fix the Nginx config" \
  --allowedTools "Bash,Read,Edit" \
  --max-turns 10 \
  --max-budget-usd 0.50

# JSON output for scripting
claude -p "List all running containers" \
  --output-format json \
  --allowedTools "Bash(docker ps *)"

# Bare mode for fastest startup (skips hooks, plugins, MCP, CLAUDE.md)
claude --bare -p "What's the disk usage?" --allowedTools "Bash(df *)"
```

Key flags for automation:
- `--allowedTools` defines exactly which tools Claude can use
- `--max-turns` limits how many actions Claude can take
- `--max-budget-usd` caps API spending
- `--output-format json` returns structured output for parsing
- `--bare` skips all auto-discovery for faster, more predictable runs
- `--continue` / `--resume <session-id>` continues a previous conversation

[source: [code.claude.com/docs/en/headless](https://code.claude.com/docs/en/headless)]

### Cron Integration

Example: health check every hour:

```bash
# /etc/cron.d/claude-health-check
0 * * * * deploy ANTHROPIC_API_KEY=sk-ant-... /usr/local/bin/claude --bare -p "SSH into homelab-docker and check: 1) all containers are running, 2) disk usage is under 80%, 3) no OOM kills in dmesg. If anything is wrong, write a summary to /tmp/claude-health-report.txt" --allowedTools "Bash(ssh *),Bash(cat *),Write" --max-turns 15 --max-budget-usd 0.25 >> /var/log/claude-health.log 2>&1
```

### Session-Scoped Scheduling with /loop

During an interactive session, use `/loop` for recurring checks:

```
/loop 5m check if the deployment finished and tell me what happened
/loop 30m check container health on all servers
/loop 1h run a disk space check and alert if anything is above 85%
```

Tasks are session-scoped and expire after 7 days. For durable scheduling, use cron or Claude Code's Cloud/Desktop scheduled tasks.

[source: [code.claude.com/docs/en/scheduled-tasks](https://code.claude.com/docs/en/scheduled-tasks)]

### Desktop Scheduled Tasks

For recurring tasks that survive restarts but need local file/tool access, use Desktop scheduled tasks (configured through the Claude Code desktop app). These run on your machine without requiring an open session.

### Cloud Scheduled Tasks

For tasks that should run regardless of whether your machine is on, use Cloud scheduled tasks. These run on Anthropic-managed infrastructure with a minimum interval of 1 hour.

[source: [code.claude.com/docs/en/scheduled-tasks](https://code.claude.com/docs/en/scheduled-tasks)]

---

## 10. Homelab-Specific Setup

### Recommended Architecture

For a typical homelab with multiple services:

```
Your Workstation (macOS/Linux)
  |
  |-- Claude Code (interactive or headless)
  |     |-- SSH to servers via ~/.ssh/config
  |     |-- MCP connections to Portainer, Proxmox, etc.
  |     |-- CLAUDE.md with full infrastructure docs
  |
  |-- homelab-docker (container host)
  |     |-- Portainer
  |     |-- Plex, *arr stack
  |     |-- Nginx reverse proxy
  |     |-- Monitoring (Grafana, Prometheus)
  |
  |-- homelab-proxmox (hypervisor)
  |     |-- VMs and LXC containers
  |     |-- Proxmox API
  |
  |-- NAS (Synology/TrueNAS)
        |-- Shared storage
        |-- Backup targets
```

### Skills for Infrastructure

Claude Code skills are markdown files in `.claude/skills/` that give Claude specialized knowledge. For sysadmin work:

```markdown
---
name: docker-ops
description: Manage Docker containers and compose stacks across homelab servers
---

# Docker Operations

## Available Servers
- homelab-docker (10.0.1.20): Main container host
- homelab-media (10.0.1.21): Media services

## Common Operations
- Check container health: `ssh homelab-docker 'docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"'`
- View logs: `ssh homelab-docker 'docker logs --tail 50 [container]'`
- Restart service: `ssh homelab-docker 'cd /opt/docker/[service] && docker compose restart'`
- Update service: `ssh homelab-docker 'cd /opt/docker/[service] && docker compose pull && docker compose up -d'`

## Rules
- Always check logs before restarting a container
- Never run `docker system prune` without asking first
- After any compose change, verify with `docker compose ps`
```

Skills use progressive disclosure: Claude reads just the description at startup, loading the full content only when relevant. This saves context window space. [source: [pulumi.com DevOps skills blog](https://www.pulumi.com/blog/top-8-claude-skills-devops-2026/)]

### Proxmox Integration

For Proxmox homelab environments, you can use:

1. **Direct SSH:** Claude runs `ssh homelab-proxmox 'qm list'` and similar commands
2. **Proxmox MCP Server:** The [proxmox-mcp-server](https://github.com/husniadil/proxmox-mcp-server) provides structured API access for managing LXC containers via SSH
3. **Skills:** A Proxmox skill can encode your specific VM layout and common operations

[source: [GitHub husniadil/proxmox-mcp-server](https://github.com/husniadil/proxmox-mcp-server), [mcpmarket.com Proxmox skills](https://mcpmarket.com/tools/skills/proxmox-infrastructure-management)]

### Tailscale / VPN Access

If your homelab uses Tailscale or another VPN, Claude Code just needs your local machine to be on the Tailscale network. SSH configs then use Tailscale IPs:

```
Host homelab-docker
    HostName 100.x.x.x  # Tailscale IP
    User deploy
    IdentityFile ~/.ssh/claude_ops
```

This works seamlessly because Claude Code runs Bash commands in your local shell context, which inherits your network connectivity.

---

## 11. Security Best Practices

### The Golden Rules

1. **Never give Claude Code root access.** Create a dedicated `deploy` user with targeted sudo permissions.
2. **Use SSH keys, never passwords.** Passphrase-less keys should only be used on trusted machines.
3. **Separate read and write permissions.** Allow reading logs and status freely; require prompts for any modifications.
4. **Use the sandbox.** Enable filesystem and network sandboxing, especially when working with untrusted content.
5. **Audit everything.** Use PostToolUse hooks to log all commands.
6. **Limit blast radius.** Use `--max-turns` and `--max-budget-usd` for unattended runs.

### Server-Side Security

```bash
# Create a dedicated user for Claude Code operations
sudo adduser deploy
sudo usermod -aG docker deploy  # If managing Docker

# Grant targeted sudo access (not full sudo)
sudo visudo
# Add: deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl status *, /usr/bin/journalctl *, /usr/sbin/nginx -t
```

### Disable Password Auth

```bash
# /etc/ssh/sshd_config
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no

sudo systemctl restart sshd
```

### Firewall Basics

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

### Install Fail2ban

```bash
sudo apt install fail2ban
sudo systemctl enable fail2ban
```

[source: [claudefa.st VPS guide](https://claudefa.st/blog/guide/development/infraops-vps-guide), [code.claude.com/docs/en/security](https://code.claude.com/docs/en/security)]

### MCP Server Security Considerations

MCP servers that return external content (web search results, community data) carry prompt injection risk. When using MCP servers for infrastructure:

- Keep infrastructure MCP servers on your local/internal network
- Treat data from internet-facing MCP servers as untrusted
- Use permission deny rules for MCP tools that could modify infrastructure:

```json
{
  "permissions": {
    "deny": [
      "mcp__ssh__execute_dangerous_command"
    ],
    "allow": [
      "mcp__portainer__listStacks",
      "mcp__portainer__listEnvironments"
    ]
  }
}
```

[source: [code.claude.com/docs/en/security](https://code.claude.com/docs/en/security)]

---

## 12. Real-World Use Cases

### Server Health Monitoring

```
"SSH into all three homelab servers and build a health report:
CPU usage, memory, disk space, container status, and any errors in the last hour of syslog"
```

### Automated SSL Certificate Management

```
"Check all SSL certificates on the nginx reverse proxy. List any expiring in the next 30 days.
For those expiring soon, run certbot renew and reload nginx."
```

### Docker Stack Updates

```
"Update the *arr stack (Sonarr, Radarr, Prowlarr) on homelab-docker:
1. Pull new images
2. Check release notes for breaking changes
3. Take a compose snapshot
4. Bring up with new images
5. Verify all containers are healthy
6. Report what changed"
```

### Incident Response

```
"The Plex container on homelab-docker is showing as unhealthy in Portainer.
Investigate: check container logs, host resources, disk space,
and the Docker daemon logs. Fix the issue and verify Plex is accessible."
```

### Infrastructure Auditing

```
"Audit the homelab network for security issues:
1. Check all SSH configs for password auth being disabled
2. Verify firewall rules on each server
3. List all open ports
4. Check for containers running as root
5. Verify all Docker images are from trusted registries
Write findings to security-audit.md"
```

### Configuration Drift Detection

```
"Compare the current Nginx config on homelab-docker with what's documented in nginx-sites.md.
Report any discrepancies and update the documentation to match reality."
```

These use cases have been verified working on Rocky Linux 10.1, Debian 12 (Proxmox 8.x), and with Nginx 1.26.3. [source: [computingforgeeks.com](https://computingforgeeks.com/claude-code-ssh-server-management/)]

---

## 13. Limitations and Gotchas

### What Claude Code Cannot Do Well

- **Real-time monitoring:** Claude Code processes prompts sequentially. It cannot watch a live log stream and react in real time. Use `/loop` for periodic checks instead.
- **Interactive terminal sessions:** Tools like `top`, `htop`, `vim`, and `less` that require a persistent interactive terminal do not work. Use non-interactive alternatives (`ps aux`, `cat`, `sed`).
- **Network scanning:** Tools like `nmap` may be blocked by the sandbox or permission system. Configure explicit allows if needed.
- **Long-running processes:** SSH sessions that take more than a few minutes may time out. Break large tasks into smaller steps.

### Common Pitfalls

1. **Context window limits:** Very large log files or command outputs can fill the context window. Tell Claude to use `tail -n 50` or `head -n 100` rather than reading entire files.
2. **SSH agent forwarding:** Claude Code runs Bash in your shell context, so SSH agent forwarding must be configured in your SSH config, not assumed.
3. **Sudo prompts:** If a command needs sudo and the deploy user requires a password, Claude Code will hang waiting for input. Use NOPASSWD for specific commands or avoid sudo-requiring operations.
4. **Compound commands and permissions:** When you approve a compound command like `git status && npm test`, Claude Code saves separate rules for each subcommand. Be aware of what you are approving. [source: [code.claude.com/docs/en/permissions](https://code.claude.com/docs/en/permissions)]
5. **Docker in sandbox:** Docker commands are incompatible with the sandbox. Add Docker to `excludedCommands` to handle it through normal permission flow. [source: [code.claude.com/docs/en/sandboxing](https://code.claude.com/docs/en/sandboxing)]

### Cost Awareness

Every interaction with Claude Code uses API tokens. For sysadmin automation:

- Use `--max-budget-usd` to cap spending on unattended runs
- Use `--max-turns` to limit how many steps Claude takes
- Use `--bare` mode for simple checks to skip loading context files
- Prefer targeted prompts over open-ended ones to reduce token usage

---

## Sources

All claims in this guide are sourced from the following:

### Official Anthropic Documentation
- [Permissions](https://code.claude.com/docs/en/permissions)
- [Sandboxing](https://code.claude.com/docs/en/sandboxing)
- [Hooks Guide](https://code.claude.com/docs/en/hooks-guide)
- [MCP Integration](https://code.claude.com/docs/en/mcp)
- [Development Containers](https://code.claude.com/docs/en/devcontainer)
- [Headless / Agent SDK CLI](https://code.claude.com/docs/en/headless)
- [Scheduled Tasks](https://code.claude.com/docs/en/scheduled-tasks)
- [Security](https://code.claude.com/docs/en/security)

### Community Guides and Tutorials
- [Manage Servers with Claude Code via SSH (ComputingForGeeks)](https://computingforgeeks.com/claude-code-ssh-server-management/)
- [How I Use Claude Code to Manage Sysadmin Tasks (Martin Alderson)](https://martinalderson.com/posts/how-i-use-claude-code-to-manage-sysadmin-tasks/)
- [Claude Code SSH to Connect to Any Remote Server (Joe Njenga, Medium)](https://medium.com/@joe.njenga/how-i-use-claude-code-ssh-to-connect-to-any-remote-server-like-a-pro-f08ed35e5569)
- [Claude Code VPS Setup Guide (claudefa.st)](https://claudefa.st/blog/guide/development/infraops-vps-guide)
- [Control Claude Code Remotely with SSH Tunnels (midgarcorp.cc)](https://midgarcorp.cc/blog/claude-code-remote-ssh-tunnel/)
- [Connected Claude Code to Home Server via MCP (XDA Developers)](https://www.xda-developers.com/connected-claude-code-through-mcp-manage-entire-lab-by-talking/)
- [The Claude Skills I Actually Use for DevOps (Pulumi Blog)](https://www.pulumi.com/blog/top-8-claude-skills-devops-2026/)
- [Claude Skills as Self-Documenting Runbooks (Zack Proser)](https://zackproser.com/blog/claude-skills-internal-training)
- [Claude Code as Sysadmin: What It Actually Looks Like (Marc Bara, Medium)](https://medium.com/@marc.bara.iniesta/claude-code-as-sysadmin-what-it-actually-looks-like-969179b995f3)

### GitHub Projects
- [claude-homelab (jmagar)](https://github.com/jmagar/claude-homelab) - Comprehensive homelab service management plugin
- [claude-ssh-server (jasondsmith72)](https://github.com/jasondsmith72/claude-ssh-server) - Ad-hoc SSH MCP server
- [adremote-mcp (nqmn)](https://github.com/nqmn/adremote-mcp) - SSH remote access MCP
- [proxmox-mcp-server (husniadil)](https://github.com/husniadil/proxmox-mcp-server) - Proxmox LXC management
- [devops-skills (lgbarn)](https://github.com/lgbarn/devops-skills) - Terraform/AWS DevOps skills
- [claudebox (RchGrav)](https://github.com/RchGrav/claudebox) - Docker development environment for Claude Code
- [homelab-agent (herms14)](https://github.com/herms14/homelab-agent) - AI-powered homelab infrastructure management
