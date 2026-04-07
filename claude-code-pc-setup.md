# Claude Code Setup: Windows PC

This guide sets up Claude Code on a Windows PC using WSL2 (the Linux subsystem built into Windows). When you're done, you'll have a `claude-pro` command that launches Claude Code with your personal Claude subscription.

An optional `claude-gateway` command for connecting through an API gateway is included at the bottom.

---

## Contents

- [Before You Start](#before-you-start)
- [Step 1: Install WSL2](#step-1-install-wsl2)
- [Opening Your Ubuntu Terminal](#opening-your-ubuntu-terminal)
- [Step 2: Install Node.js](#step-2-install-nodejs)
- [Step 3: Install Claude Code](#step-3-install-claude-code)
- [Step 4: Add the claude-pro Command](#step-4-add-the-claude-pro-command)
- [Step 5: Launch Claude Code](#step-5-launch-claude-code)
- [Troubleshooting](#troubleshooting)
- [Optional: claude-gateway (API Gateway Connection)](#optional-claude-gateway-api-gateway-connection)
- [Quick Reference](#quick-reference)
- [Next Steps](#next-steps)

---

## Before You Start

### What You'll Need

- Windows 10 (version 2004 or later) or Windows 11
- A Windows account with **admin rights** (needed only for Step 1, one time)
- A Claude Pro or Max subscription from Anthropic

---

## Step 1: Install WSL2

### Why WSL2?

WSL2 (Windows Subsystem for Linux) is a full Linux environment built into Windows. Claude Code requires it on Windows because it relies on a Linux-compatible shell for running commands. Everything after this step runs in user space, no admin rights required beyond this point.

### Installation

1. Click **Start**, search for **PowerShell**, right-click the result, and choose **Run as administrator**
2. Paste this command and press **Enter**:

```powershell
wsl --install
```

3. When it finishes, **restart your computer**
4. After restarting, an **Ubuntu** window may open automatically and prompt you to create a username and password

> **Note:** Your WSL username does not need to match your Windows username. Pick anything you'll remember. This is just for your local Linux environment.

### No Ubuntu App After Restarting?

On some machines, `wsl --install` enables WSL but does not install Ubuntu automatically. If you don't see an Ubuntu app in your Start menu after restarting:

1. Open the **Microsoft Store** (search for it in the Start menu)
2. Search for **Ubuntu** and install it (the one published by Canonical)
3. Once installed, open **Ubuntu** from the Start menu
4. Ubuntu will finish setting up and ask you to create a username and password

### Already Have WSL Installed?

Run these commands in PowerShell to make sure everything is up to date:

```powershell
wsl --update
```

If you have WSL but no Ubuntu distro, run:

```powershell
wsl --install -d Ubuntu
```

> **No admin rights?** Contact your IT support team to install WSL2 for you. Once it's installed, you won't need admin access for any remaining steps.

---

## Opening Your Ubuntu Terminal

Every step from here on happens inside Ubuntu, not PowerShell or Command Prompt. Here's how to open it:

1. Click **Start** and search for **Ubuntu**
2. Click the **Ubuntu** app to open a terminal window

This is your Linux terminal. You'll use it for installation, configuration, and launching Claude Code. You can pin it to your taskbar for quick access.

> **Tip:** If you ever close the window, just open Ubuntu from the Start menu again. Your files and configuration are preserved between sessions.

---

## Step 2: Install Node.js

### Install Node Version Manager (nvm)

Run these commands **one at a time**, pressing Enter after each:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

```bash
source ~/.bashrc
```

```bash
nvm install --lts
```

### Verify the Installation

```bash
node --version
```

You should see something like `v22.x.x`. If you do, Node.js is installed.

> **No sudo needed:** None of these commands require admin or `sudo` access.

---

## Step 3: Install Claude Code

### Install via npm

In your Ubuntu terminal, run:

```bash
npm install -g @anthropic-ai/claude-code
```

### Verify the Installation

```bash
claude --version
```

You should see a version number. If you see `command not found`, close and reopen Ubuntu and try again.

---

## Step 4: Add the claude-pro Command

You'll add a shell function to a config file that Ubuntu reads every time a terminal opens. This lets you launch Claude Code with a single command.

### Open Your Shell Config File

```bash
nano ~/.bashrc
```

This opens a text editor in your terminal. Don't worry about what's already in the file.

### Paste the Function

Use the **arrow keys** to scroll to the very bottom of the file. Paste this block:

```bash
# Claude Code - Personal Subscription
claude-pro() {
  unset CLAUDE_CONFIG_DIR
  unset ANTHROPIC_BASE_URL
  unset ANTHROPIC_AUTH_TOKEN
  unset ANTHROPIC_MODEL
  unset ANTHROPIC_DEFAULT_OPUS_MODEL
  unset ANTHROPIC_DEFAULT_HAIKU_MODEL
  unset ANTHROPIC_DEFAULT_SONNET_MODEL
  unset ANTHROPIC_SMALL_FAST_MODEL
  unset CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS
  unset DISABLE_PROMPT_CACHING
  unset DISABLE_TELEMETRY
  claude
}
```

### Save and Exit nano

- Press **Ctrl+O** (the letter O, not zero) to save, then press **Enter** to confirm
- Press **Ctrl+X** to exit

### Reload Your Shell

```bash
source ~/.bashrc
```

This makes the new `claude-pro` command available without restarting your terminal.

---

## Step 5: Launch Claude Code

### First Login

```bash
cd ~
claude-pro
```

Claude Code will open a browser window and ask you to log in with your Anthropic account. Complete the login and return to the terminal. Subsequent launches skip this step.

You can replace `cd ~` with any folder you want to work in. Claude Code reads files from wherever you launch it.

### Accept Terms of Service

The first time you launch, Claude Code will ask you to accept its terms of service. Type `yes` and press **Enter**.

### Verify It's Working

Once Claude Code opens, run these commands inside it:

| Command | What to expect |
|---|---|
| `/model` | Should show `Sonnet 4.6` (or your subscription's default model) |
| `/cost` | Shows your token usage for the current session |

To exit Claude Code, press **Ctrl+C** or type `/exit`.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| "Authentication failed" | Run `claude-pro` again to re-authenticate through the browser |
| `claude-pro: command not found` | Run `source ~/.bashrc` or close and reopen Ubuntu |
| WSL won't install | Virtualization may be disabled in your BIOS. Contact IT support |
| Ubuntu doesn't appear after restart | Install **Ubuntu** from the Microsoft Store (published by Canonical) |
| Browser login doesn't complete | Make sure your default Windows browser is open and try again |

---

## Optional: claude-gateway (API Gateway Connection)

If your organization runs a self-hosted API gateway (like LiteLLM or a similar proxy), you can add a second command to connect through it. This routes all traffic through the gateway instead of directly to Anthropic.

### Get Your API Key

Your gateway administrator will provide:
- The gateway URL (e.g., `https://api-gateway.yourorg.com/`)
- An API key
- The model names available on the gateway

### Add the Function

```bash
nano ~/.bashrc
```

Scroll to the bottom and paste this block **after** the `claude-pro` function:

```bash
# Claude Code - API Gateway
claude-gateway() {
  export CLAUDE_CONFIG_DIR=~/.claude-gateway
  export ANTHROPIC_BASE_URL="https://your-gateway-url-here/"
  export ANTHROPIC_AUTH_TOKEN="sk-YOURKEYHERE"
  export ANTHROPIC_MODEL="anthropic.claude-4.6-sonnet"
  export ANTHROPIC_SMALL_FAST_MODEL="anthropic.claude-4.5-haiku"
  export ANTHROPIC_DEFAULT_OPUS_MODEL="anthropic.claude-4.6-opus"
  export ANTHROPIC_DEFAULT_HAIKU_MODEL="anthropic.claude-4.5-haiku"
  export ANTHROPIC_DEFAULT_SONNET_MODEL="anthropic.claude-4.6-sonnet"
  export CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1
  export DISABLE_PROMPT_CACHING=1
  export DISABLE_TELEMETRY=1
  claude
}
```

### Configure the Function

Replace these values with what your gateway administrator provided:
- `https://your-gateway-url-here/` with your gateway's URL
- `sk-YOURKEYHERE` with your API key
- Model names if your gateway uses different identifiers

Save with **Ctrl+O**, exit with **Ctrl+X**, then reload:

```bash
source ~/.bashrc
```

### Verify the Gateway Connection

Run `claude-gateway`. Once Claude Code opens, check `/model`. It should show the full gateway model name (e.g., `anthropic.claude-4.6-sonnet`). If it shows a short name like "Sonnet 4.6", Claude Code is connecting directly to Anthropic instead of through the gateway.

### Key Rotation

API keys typically expire on a schedule set by your gateway administrator. When yours expires, get a new key and update the `ANTHROPIC_AUTH_TOKEN` line in `~/.bashrc`, then run `source ~/.bashrc`.

---

## Quick Reference

| Command | What it uses | Billed to |
|---|---|---|
| `claude-pro` | Personal Claude subscription | Your Anthropic account |
| `claude-gateway` | Organization API Gateway | Your organization's account |

---

## Next Steps

Claude Code works best when launched from a dedicated workspace folder containing a `CLAUDE.md` file with your project context. This file tells Claude about your codebase, preferences, and conventions.

Create a workspace folder and add a `CLAUDE.md` to get started:

```bash
mkdir -p ~/workspace
cd ~/workspace
nano CLAUDE.md
```

Add context about your project, then launch Claude Code from that directory.

---

*Last updated: 2026-04-07*
