---
title: "Claude Code + AWS Bedrock + IAM Identity Center (SSO) in VS Code on WSL2"
audience: "Pete (senior engineer, WSL2 + VS Code + Bedrock via IdC)"
report_type: "diagnostic + integration"
date: 2026-05-27
timezone: America/New_York
generated_by: Research agent (Claude Code)
update_history_count: 0
---

# Claude Code + AWS Bedrock + IAM Identity Center (SSO) in VS Code on WSL2

## TL;DR

Pete's "Claude Code tab in VS Code is just spinning" is almost certainly one of three known bugs, ranked by how well they match the WSL2 + Bedrock + IdC profile:

1. **Most likely (WSL2-specific): the extension's `bash -lic` PATH probe is hanging on something in `~/.bashrc`.** Filed and confirmed as a real bug (anthropics/claude-code [#49583](https://github.com/anthropics/claude-code/issues/49583)). Closed by github-actions for inactivity, never fixed. Symptom matches exactly: WSL2, VS Code Claude Code panel hangs on the spinner forever, no error, "/model" never responds, "Open new tab" does nothing. Root cause: extension spawns `bash -lic` to discover PATH, that inherits Pete's interactive `.bashrc`, and any blocking command in `.bashrc` (typically a `sudo` without NOPASSWD, but `ssh` agent prompts, `read`, slow network mounts, even `nvm`/`pyenv` init that hangs on a network call all do it) pins the probe forever, so the `claude` native binary is never spawned. Workaround: gate blocking `.bashrc` lines on `[ -t 0 ] && [ -t 1 ]`, then `Developer: Reload Window`.

2. **The "config probe 60-second timeout" bug ([#60045](https://github.com/anthropics/claude-code/issues/60045), open as of 2026-05-17).** Every first tab opens a *second* full Claude subprocess just to call `getSettings()`. That probe races the real session for resources and reliably times out after 60s, showing a red banner. The real session works after that. Confirmed reproducer is "Bedrock + many MCP servers + first tab after VS Code launch."

3. **Bedrock auth is silently absent because `disableLoginPrompt` isn't set, so the extension is sitting on the Anthropic OAuth sign-in screen instead of using Bedrock at all.** This isn't actually a hang, but it presents as "the panel won't load anything useful." Fix is one VS Code setting: `claudeCode.disableLoginPrompt: true` plus `Developer: Reload Window`. Documented but easy to miss.

A fourth common cause for Pete specifically: **the extension host doesn't see the AWS credentials.** VS Code launched from the Start menu, or VS Code Remote-WSL launched from Windows, doesn't inherit the same env as a shell where `aws sso login --profile` was run. The Codespaces variant of this bug ([#51108](https://github.com/anthropics/claude-code/issues/51108)) shows the exact same `claude` binary works from the integrated terminal and fails from the extension. Pin everything into `~/.claude/settings.json` `env` block to make it deterministic.

**Diagnostic order:**

1. Open `Output` panel in VS Code → channel `Claude Code` (or `Anthropic.claude-code` extension log) → look for `launch_claude` followed by nothing. That confirms #49583.
2. In a WSL terminal: `ps -eo pid,etime,cmd | grep extensionHost | grep -v grep`, then `pstree -p <PID>`. If you see a bash→sudo (or bash→ssh, bash→curl) chain hanging off the extension host with no `claude(...)` child, that's #49583.
3. Run `claude` directly in the WSL integrated terminal of the same VS Code window. If that works and only the GUI panel hangs, the issue is in the extension's launch path (probe or env propagation), not Bedrock auth.
4. `aws sts get-caller-identity` in the same terminal. If that succeeds and #3 also works but the panel still hangs, set `claudeCode.disableLoginPrompt: true` and reload.

Full diagnostics, fixes, and known limitations of Bedrock vs first-party in [Findings](#findings).

## Table of Contents

- [TL;DR](#tldr)
- [Current Status](#current-status)
- [Findings](#findings)
  - [The spinner: most-likely causes ranked](#the-spinner-most-likely-causes-ranked)
  - [How the Claude Code VS Code extension actually launches](#how-the-claude-code-vs-code-extension-actually-launches)
  - [How AWS auth resolves: precedence chain](#how-aws-auth-resolves-precedence-chain)
  - [`awsAuthRefresh` vs `awsCredentialExport`: when to use which](#awsauthrefresh-vs-awscredentialexport-when-to-use-which)
  - [IAM Identity Center setup for Bedrock + Claude Code](#iam-identity-center-setup-for-bedrock--claude-code)
  - [Diagnostic playbook for the spinning tab](#diagnostic-playbook-for-the-spinning-tab)
  - [WSL2-specific gotchas](#wsl2-specific-gotchas)
  - [Known limitations: Bedrock vs first-party Anthropic API](#known-limitations-bedrock-vs-first-party-anthropic-api)
  - [Recommended settings.json for Pete](#recommended-settingsjson-for-pete)
- [Confidence Assessment](#confidence-assessment)
- [Open Questions](#open-questions)
- [Sources](#sources)
  - [Official Anthropic / AWS docs](#official-anthropic--aws-docs)
  - [Most-relevant GitHub issues (anthropics/claude-code)](#most-relevant-github-issues-anthropicsclaude-code)
  - [Community writeups](#community-writeups)
- [Update History](#update-history)
- [How This Report Was Generated](#how-this-report-was-generated)

## Current Status

As of 2026-05-27, Anthropic supports Bedrock as a first-class provider for Claude Code via the `CLAUDE_CODE_USE_BEDROCK=1` environment variable, with documented support for AWS SSO / IAM Identity Center credentials. The VS Code extension (`anthropic.claude-code`) supports third-party providers including Bedrock, but does so by:

- Honoring `~/.claude/settings.json` `env` block for `CLAUDE_CODE_USE_BEDROCK`, `AWS_PROFILE`, `AWS_REGION`, and related variables (this is the shared config with the CLI).
- Requiring a separate VS Code extension setting, `claudeCode.disableLoginPrompt: true`, to skip the Anthropic OAuth sign-in screen.
- Spawning a `bash -lic` PATH probe at activation (on Linux/WSL) before spawning the actual `claude` binary.
- Spawning a second "config probe" subprocess on first tab open per VS Code session, which has its own 60s timeout.
- Inheriting the extension host's environment, which on WSL2 / Remote-WSL is *not* necessarily the same as a logged-in shell.

The most relevant first-party docs are [Claude Code on Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock), [Use Claude Code in VS Code](https://code.claude.com/docs/en/vs-code), and [Environment variables](https://code.claude.com/docs/en/env-vars).

For enterprise rollout, AWS publishes an [official solution guide](https://docs.aws.amazon.com/solutions/claude-code-with-amazon-bedrock/) with sample code at [`aws-solutions-library-samples/guidance-for-claude-code-with-amazon-bedrock`](https://github.com/aws-solutions-library-samples/guidance-for-claude-code-with-amazon-bedrock). As of v2.2 of that solution (released spring 2026), IAM Identity Center is a first-class authentication option alongside OIDC IdPs (Okta, Azure AD, Auth0).

## Findings

### The spinner: most-likely causes ranked

The "Claude Code tab spins forever in VS Code with Bedrock + IdC SSO on WSL2" symptom has four overlapping causes. Ordered by how well each fits Pete's setup (WSL2 + VS Code Remote-WSL + Bedrock + IdC):

#### Cause 1 — Extension `bash -lic` PATH probe hangs on `~/.bashrc` (High confidence, WSL2-specific)

GitHub issue [#49583](https://github.com/anthropics/claude-code/issues/49583), filed 2026-04-16 by a user whose environment is essentially identical to Pete's (Windows 11 Insider, WSL2 Ubuntu 24.04.3, VS Code 1.115.0, Claude Code extension 2.1.112). The reporter grepped the installed extension bundle and found `printenv PATH` invoked via `bash -lic`. That `-l -i` combination forces bash to source `~/.bashrc` as a login + interactive shell. Anything in `.bashrc` that blocks on `/dev/tty` (sudo password prompt, ssh-add for a passphrase-protected key, `read`, `expect`, even `nvm`/`pyenv` init that does a slow network lookup) pins the probe forever. The extension never receives the probe output and never spawns the `claude` native binary. Symptoms match exactly: webview loads, "loading models" spins, `/model` never responds, "Open new tab" does nothing, no error surfaced. Extension Host log for `Anthropic.claude-code` stops at `launch_claude` with no further lines.

Issue closed 2026-05-24 by `github-actions` for inactivity. **Not fixed.** No version note. Confirmed to reproduce across "all versions that contain the `printenv PATH` probe" per the reporter. [source](https://github.com/anthropics/claude-code/issues/49583)

**How to confirm on Pete's machine:** in a fresh WSL terminal (not in the broken VS Code window):

```bash
ps -eo pid,etime,cmd | grep extensionHost | grep -v grep
pstree -p <PID-of-extension-host>
```

Signature: no `claude(...)` child under the extension host, instead a bash→sudo (or bash→ssh, bash→curl, bash→nvm) chain hanging off it for as long as the spinner has been spinning.

**Workaround:** wrap any blocking line in Pete's `~/.bashrc` with a TTY guard:

```bash
# Before:
sudo service mysql start > /dev/null 2>&1

# After:
[ -t 0 ] && [ -t 1 ] && sudo service mysql start > /dev/null 2>&1
```

Then `kill` the stuck bash/sudo processes and `Developer: Reload Window` in VS Code.

#### Cause 2 — Config probe subprocess hits a 60s timeout (Medium-high confidence)

GitHub issue [#60045](https://github.com/anthropics/claude-code/issues/60045), open as of 2026-05-17, environment: macOS, Claude Code extension 2.1.143, Bedrock with `global.anthropic.claude-opus-4-6-v1`, OneDrive-synced working directory, "many MCP servers configured." On every first tab open after VS Code launch, the extension calls `spawnConfigProbe()` which spawns a *full second* `claude` subprocess just to call `getSettings()`. That probe runs through every SessionStart hook, MCP server connect, skills load, etc. In the bug report it completes in ~2s but the extension's `initializationResult()` never resolves, so it waits the full 60-second timeout, shows a red banner ("Subprocess initialization did not complete within 60000ms — check authentication and network connectivity"), and the user has to re-send their prompt. Subsequent tabs work immediately because the config is cached. [source](https://github.com/anthropics/claude-code/issues/60045)

A second commenter on the same issue (ezwep, macOS) reproduces it on every cold boot when MCP Docker containers aren't healthy yet at VS Code launch. Pete runs many MCP servers (per his workspace skills list: searxng, threatintel, github, homelab-arr, portainer, n8n, strava, nfl, unifi, etc.), some of which depend on nix1 over Tailscale. If any of those are slow to connect at extension activation, the probe will hit the 60s deadline.

This is technically a "fails after 60s" not a "spins forever," but if Pete has multiple MCP servers and is opening his first tab of a VS Code session, he'd see the spinner for a full minute before the error appears, which is plausibly being mis-reported as "just spinning."

#### Cause 3 — `disableLoginPrompt` not set; extension stuck on Anthropic OAuth (Medium confidence)

Documented Anthropic behavior. The VS Code extension defaults to its own OAuth login screen even when `~/.claude/settings.json` has `CLAUDE_CODE_USE_BEDROCK=1`. The CLI in the integrated terminal will use Bedrock fine; the GUI panel will not. The fix is documented but easy to miss: in VS Code settings, search "Claude Code login," check **Disable Login Prompt** (`claudeCode.disableLoginPrompt: true`), then `Developer: Reload Window` ([source](https://code.claude.com/docs/en/vs-code#use-third-party-providers), [source](https://dev.to/gunnargrosch/from-terminal-to-ide-using-claude-code-with-bedrock-in-vs-code-and-jetbrains-3gbn)).

The dev.to walkthrough author calls this "the part that trips people up" and warns: "The VS Code extension has its own authentication setting that's separate from your `~/.claude/settings.json` file. Even with Bedrock configured, the extension will show you an Anthropic login screen unless you explicitly tell it not to."

This presents as the panel loading and then showing a sign-in button or being unresponsive, not strictly as a spinner, but if combined with [#8361](https://github.com/anthropics/claude-code/issues/8361) (Bedrock token detection broken in extension v2.0.0, closed/resolved), the user can perceive it as the tab not working.

#### Cause 4 — Extension host doesn't see AWS credentials (Medium confidence)

GitHub issue [#51108](https://github.com/anthropics/claude-code/issues/51108) (closed for inactivity, not fixed): in GitHub Codespaces, the *exact same bundled `claude` binary* fails with `Could not load credentials from any providers` when invoked via the VS Code extension UI (`cc_entrypoint=claude-vscode`) but succeeds when run manually from the integrated terminal (`cc_entrypoint=cli`). The reporter proves identical binary, identical credentials, different outcome based on launch context. Commenter `0xbrainkid` summarized: "the failure is not Bedrock itself, nor the account credentials in general. It is something about the extension launch context, env propagation, credential-provider chain, or process sandboxing on the `claude-vscode` path." [source](https://github.com/anthropics/claude-code/issues/51108)

This applies to Pete because VS Code on Windows, when launched from the Start menu and connected to WSL via Remote-WSL, runs the extension host with the WSL user's *non-interactive* environment. If Pete has been doing `aws sso login --profile foo && export AWS_PROFILE=foo` in his shell, the extension host doesn't see `AWS_PROFILE`. The fix is to put `AWS_PROFILE` and `AWS_REGION` into the `env` block of `~/.claude/settings.json` so they're read directly from the file, not inherited from the shell. See [Recommended settings.json](#recommended-settingsjson-for-pete) below.

### How the Claude Code VS Code extension actually launches

Knowing the launch path is what lets you diagnose the spinner instead of guess. Reconstructed from the extension docs ([VS Code page](https://code.claude.com/docs/en/vs-code)), the bug reports above (which contain extension-bundle disassembly and extension-host log excerpts), and the `[BUG] Extension stuck on "loading models" in WSL` reporter's own grep of the extension `extension.js`:

1. **Activation.** VS Code activates `anthropic.claude-code`. Extension reads its own VS Code settings (`claudeCode.useTerminal`, `claudeCode.disableLoginPrompt`, `claudeCode.environmentVariables`, etc.).
2. **PATH probe (Linux/WSL/macOS).** Extension spawns `bash -lic 'printenv PATH'` to discover the user's PATH as it would be in a real login shell. Inherits `~/.bashrc`. This is where #49583 hangs.
3. **Sign-in resolution.** If `disableLoginPrompt` is `false` (default), and there's no OAuth token in `~/.claude/.credentials.json`, the webview renders the sign-in screen. If `disableLoginPrompt` is `true`, the extension skips the sign-in screen and proceeds to launch.
4. **Config probe.** On the first tab of a VS Code session, the extension calls `spawnConfigProbe()`, which spawns a *separate* full `claude` subprocess just to call `getSettings()` so it can cache the user settings. This probe runs SessionStart hooks, connects MCP servers, loads skills, and is supposed to return an `initializationResult()` back to the extension. Hits a 60s hard timeout. This is where #60045 fires.
5. **Real session subprocess.** The actual chat-backing `claude` subprocess is spawned for the tab. Its env is built from: VS Code extension's `claudeCode.environmentVariables` setting, plus `~/.claude/settings.json` `env` block, plus the inherited extension-host environment (which on WSL Remote is the non-interactive shell env). The auth chain is then resolved per [Authentication precedence](https://code.claude.com/docs/en/iam#authentication-precedence): if `CLAUDE_CODE_USE_BEDROCK=1` is set, the AWS SDK default credential provider chain runs (env vars → AWS_PROFILE → SSO cache in `~/.aws/sso/cache/` → instance metadata).
6. **IDE MCP server (local).** The extension also runs an `ide` MCP server bound to `127.0.0.1` on a random high port, with a fresh per-activation auth token written to `~/.claude/ide/<port>.lock` (mode `0600` in a `0700` dir). This is how the panel and the underlying `claude` process share state for diff viewing, selection capture, etc. Not usually a failure point but worth knowing about ([source](https://code.claude.com/docs/en/vs-code#the-built-in-ide-mcp-server)).

The two probe steps (2 and 4) are the spinner's two main hang points. The credential-provider chain in step 5 is the silent-failure point.

### How AWS auth resolves: precedence chain

When `CLAUDE_CODE_USE_BEDROCK=1` is set, Claude Code uses the AWS SDK default credential provider chain. From the [Bedrock setup docs](https://code.claude.com/docs/en/amazon-bedrock#2-configure-aws-credentials), supported options are:

- **AWS CLI configuration**: standard `~/.aws/credentials` and `~/.aws/config`. Includes SSO profiles.
- **Environment variables (access key)**: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, optionally `AWS_SESSION_TOKEN`.
- **Environment variables (SSO profile)**: `AWS_PROFILE` pointing at a profile in `~/.aws/config` that has `sso_session` / `sso_account_id` / `sso_role_name`.
- **AWS Management Console credentials**: short-term keys generated via `aws login`.
- **Bedrock API keys**: `AWS_BEARER_TOKEN_BEDROCK`. New-ish; sent as a bearer token; bypasses SigV4 ([source](https://aws.amazon.com/blogs/machine-learning/accelerate-ai-development-with-amazon-bedrock-api-keys/)).

Per the docs, `AWS_REGION` is **required** as an environment variable. Claude Code does *not* read region from `~/.aws/config`. This catches people repeatedly ([source](https://code.claude.com/docs/en/amazon-bedrock#3-configure-claude-code)).

The full Claude Code auth precedence (from [iam docs](https://code.claude.com/docs/en/iam#authentication-precedence)):

1. Cloud provider credentials, when `CLAUDE_CODE_USE_BEDROCK`, `CLAUDE_CODE_USE_VERTEX`, or `CLAUDE_CODE_USE_FOUNDRY` is set.
2. `ANTHROPIC_AUTH_TOKEN` env var (bearer).
3. `ANTHROPIC_API_KEY` env var (x-api-key).
4. `apiKeyHelper` script output.
5. `CLAUDE_CODE_OAUTH_TOKEN` env var.
6. Subscription OAuth credentials from `/login`.

If any of (2)-(6) is present and `CLAUDE_CODE_USE_BEDROCK` is *not* set in the extension's environment, Claude Code will try to use Anthropic-direct credentials and you'll never reach Bedrock. This is one more reason to put `CLAUDE_CODE_USE_BEDROCK=1` into the `env` block of `~/.claude/settings.json` rather than relying on shell-inherited env.

### `awsAuthRefresh` vs `awsCredentialExport`: when to use which

Two settings in `~/.claude/settings.json` for auto-refreshing AWS creds. Different trigger conditions, both with a **3-minute hard timeout** (added in v2.1.41, 2026-02-13, but [undocumented](https://github.com/anthropics/claude-code/issues/25457)).

**`awsAuthRefresh`** ([docs](https://code.claude.com/docs/en/amazon-bedrock#advanced-credential-configuration)):
- Runs only when Claude Code detects that AWS creds are expired (locally by timestamp, or after Bedrock returns a cred error).
- Output is displayed to the user. Interactive input is *not* supported, so use it for browser-based flows where the CLI just prints a URL to click.
- Recommended for SSO. Typical value: `"awsAuthRefresh": "aws sso login --profile myprofile"`.
- Bug history: regressions where it loops indefinitely ([#12421](https://github.com/anthropics/claude-code/issues/12421), closed/fixed) and where the auth link doesn't surface in the terminal ([#52978](https://github.com/anthropics/claude-code/issues/52978), open, "broken on versions >0.92" per reporter, status unclear). Per a 2026-02 comment on the related thread, by v2.1.87 it's working again for SSO when set in `~/.claude/settings.json` (not the project settings).
- **Auth-loop pitfall** ([Bedrock docs, "Authentication loop with SSO and corporate proxies"](https://code.claude.com/docs/en/amazon-bedrock#troubleshooting)): if browser tabs spawn repeatedly when using AWS SSO, remove the `awsAuthRefresh` setting. Corporate VPN / TLS-inspection proxies can interrupt the SSO browser flow, which Claude Code interprets as a failure, re-runs `awsAuthRefresh`, and loops indefinitely. If Pete is on a network with TLS inspection, do `aws sso login` manually before launching VS Code and skip `awsAuthRefresh`.

**`awsCredentialExport`** ([docs, same section](https://code.claude.com/docs/en/amazon-bedrock#advanced-credential-configuration)):
- Runs at session start and on every credential reload, regardless of whether creds are still valid.
- Output is captured silently; user doesn't see it.
- Must print JSON in the format Claude Code expects: `{ "Credentials": { "AccessKeyId": "...", "SecretAccessKey": "...", "SessionToken": "..." } }`. Note: this is **not** the AWS CLI `credential_process` shape (which uses `{ "Version": 1, "AccessKeyId": ..., ... }` at the top level). Confirmed against the current [Bedrock docs](https://code.claude.com/docs/en/amazon-bedrock#advanced-credential-configuration).
- Use only if you cannot modify `~/.aws/` and must directly return creds. Typical case: cross-account where the default credential chain would resolve the wrong account, so you need to call out to a vault.

### IAM Identity Center setup for Bedrock + Claude Code

Two paths, depending on whether Pete is using IdC directly or through the AWS solutions guidance.

#### Path A: Direct (`aws sso login` + `AWS_PROFILE`)

This is what Pete is doing. Configure an SSO session and profile in `~/.aws/config`:

```ini
[sso-session pete-corp]
sso_start_url = https://pete-corp.awsapps.com/start
sso_region = us-east-1
sso_registration_scopes = sso:account:access

[profile bedrock-dev]
sso_session = pete-corp
sso_account_id = 123456789012
sso_role_name = BedrockDeveloperAccess
region = us-east-1
output = json
```

Then `aws sso login --profile bedrock-dev` opens a browser, drops the token in `~/.aws/sso/cache/<hash>.json` (default 8h, configurable up to 90 days per session), and the AWS SDK reads from there as long as `AWS_PROFILE=bedrock-dev` is set.

The IAM policy attached to `BedrockDeveloperAccess` permission set (from [bespinian's guide](https://bespinian.io/en/blog/claude-code-using-aws-bedrock/)):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowModelAndInferenceProfileAccess",
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream",
        "bedrock:ListInferenceProfiles"
      ],
      "Resource": [
        "arn:aws:bedrock:*:*:inference-profile/*",
        "arn:aws:bedrock:*:*:application-inference-profile/*",
        "arn:aws:bedrock:*:*:foundation-model/*"
      ]
    },
    {
      "Sid": "AllowMarketplaceSubscription",
      "Effect": "Allow",
      "Action": [
        "aws-marketplace:ViewSubscriptions",
        "aws-marketplace:Subscribe"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:CalledViaLast": "bedrock.amazonaws.com"
        }
      }
    }
  ]
}
```

Add `bedrock:GetInferenceProfile` if Pete uses application inference profiles, to skip a round-trip on each new model lookup (per official Bedrock docs).

**SSO profile gotcha** ([#14086](https://github.com/anthropics/claude-code/issues/14086), closed for inactivity, **still reproduces in v2.1.22+** per commenters): the docs show `aws sso login --profile foo && export AWS_PROFILE=foo` as sufficient, but multiple users report `API Error: Value not present for 'accessToken' in SSO Token`. Workaround that worked for the reporter and confirmers: add `credential_process = aws configure export-credentials --profile <profile>` to the profile in `~/.aws/config`. That forces the SDK to call `aws configure export-credentials` (which handles SSO token exchange) instead of trying to read the SSO cache directly. This is a real bug, not a config error.

#### Path B: AWS Solutions Library `ccwb` tool

AWS publishes [aws-solutions-library-samples/guidance-for-claude-code-with-amazon-bedrock](https://github.com/aws-solutions-library-samples/guidance-for-claude-code-with-amazon-bedrock) — a CloudFormation + Python deployment tool (`ccwb init` / `ccwb deploy`) that wires up Bedrock access via IAM Identity Center as a first-class option (added in v2.2). It generates `~/.aws/config`, the right CloudFormation stack, and platform-specific installer packages.

For Pete: probably overkill since this is his own workspace, not a team deployment. But worth knowing it exists. The same repo's "None (use existing AWS credentials)" deployment mode (v2.1+) deploys just the observability/analytics infrastructure for usage tracking without any IdP. Identity attribution works automatically for IdC users because the `AWSReservedSSO_*` role ARN encodes the email and permission set.

The repo's identity-mode comparison:

| Mode | Identity Source | Session Length | Quota Enforcement | When to use |
|---|---|---|---|---|
| External IdP (OIDC) | Okta / Azure AD / Auth0 / Cognito JWT claims | Refresh token lifetime | Full | Existing enterprise IdP |
| AWS IAM Identity Center | `AWSReservedSSO_*` ARN (email + permission set) | Up to 90 days (recommended 7) | Not available | Native AWS identity, or when corporate policy blocks `localhost:8400` callback |
| None | IAM user ARN or hashed role principal | AWS credential TTL | Not available | Analytics-only / internal tools |

[source](https://github.com/aws-solutions-library-samples/guidance-for-claude-code-with-amazon-bedrock)

### Diagnostic playbook for the spinning tab

Run these in this order. Most cases resolve at step 2 or 3.

1. **Open the extension log.** VS Code → View → Output → dropdown → "Claude Code" (and/or "Anthropic.claude-code"). Look for:
   - `launch_claude` followed by silence: classic #49583. The extension started the probe and never got a response. Go to step 2.
   - `Loading config cache by launching Claude (no channel)...` then 60s later `Subprocess initialization did not complete within 60000ms`: classic #60045. Workaround: dismiss the banner and re-send. Permanent workaround: reduce MCP server count or stop using Bedrock for the first tab.
   - `Could not load credentials from any providers`: extension host doesn't see AWS creds. Go to step 4.
   - Nothing at all: extension didn't activate. Try `Developer: Reload Window` and check for activation errors in the developer console (`Help → Toggle Developer Tools → Console`).

2. **Confirm the bash probe is the culprit.** In a fresh WSL terminal (open a new Windows Terminal tab, not in the broken VS Code window):

   ```bash
   ps -eo pid,etime,cmd | grep extensionHost | grep -v grep
   # Note the PID with the longest etime (it's been hanging since you opened VS Code)
   pstree -p <PID>
   ```

   If the tree shows a bash → sudo or bash → ssh or bash → curl chain with no `claude(...)` child, that's the probe hanging. Identify which `.bashrc` line is responsible:

   ```bash
   bash -lic 'printenv PATH'   # Reproduce the hang in a terminal you can Ctrl-C out of
   # If it hangs, comment out lines in ~/.bashrc one at a time until it returns.
   ```

   Wrap the offending line in a TTY guard (`[ -t 0 ] && [ -t 1 ] && <cmd>`), kill the hung processes, then `Developer: Reload Window`.

3. **Test the CLI in the integrated terminal.** In the same VS Code window's integrated terminal:

   ```bash
   claude --version
   claude /status      # or just: claude, then type /status
   ```

   If the CLI works in the integrated terminal but the GUI panel still hangs, the issue is in the extension's launch path (probe or env propagation), not Bedrock itself.

4. **Verify AWS auth in the integrated terminal.**

   ```bash
   aws sts get-caller-identity   # should print AccountId / Arn / UserId
   echo "$AWS_PROFILE $AWS_REGION CLAUDE_CODE_USE_BEDROCK=$CLAUDE_CODE_USE_BEDROCK"
   aws bedrock list-inference-profiles --region "$AWS_REGION" | head
   ```

   If `get-caller-identity` works but the extension can't auth: the extension host doesn't see the env. Move all of `AWS_PROFILE`, `AWS_REGION`, `CLAUDE_CODE_USE_BEDROCK` into `~/.claude/settings.json` `env` block. See [Recommended settings.json](#recommended-settingsjson-for-pete).

5. **Verify `disableLoginPrompt`.** In VS Code settings (Ctrl+,), search "Claude Code login," confirm "Disable Login Prompt" is checked. If not: check it, then `Developer: Reload Window`.

6. **Last resort: extension debug logs.** Enable verbose extension logging by setting `DEBUG=1` and `CLAUDE_CODE_DEBUG_LOG_LEVEL=verbose` in the `env` block of `~/.claude/settings.json`, then reload. Logs land at `~/.claude/debug/<session-id>.txt` (or wherever `CLAUDE_CODE_DEBUG_LOGS_DIR` points). Attach to a GitHub issue if you need to file a new one.

7. **`/doctor`.** If you can get a CLI session up, run `/doctor` inside Claude Code (or `claude doctor` from the shell if `claude` won't start). It runs an automated check across installation, settings, MCP servers, and context usage ([source](https://code.claude.com/docs/en/troubleshooting)).

### WSL2-specific gotchas

Beyond the `.bashrc` probe issue:

- **`Shift+Enter` doesn't insert newline in WSL on Windows 10** ([#1262](https://github.com/anthropics/claude-code/issues/1262), open). Annoyance, not a hang.
- **Slow search results on WSL across the Windows/Linux boundary** ([troubleshooting docs](https://code.claude.com/docs/en/troubleshooting#slow-or-incomplete-search-results-on-wsl)). Pete's workspace is at `/mnt/c/Users/pster/ai-cli-workspace/`. That's on the Windows filesystem. Reading from `/mnt/c/` in WSL is documented to be much slower than the native Linux filesystem (`/home/`). For Claude Code's search and `@file`-mention tools, this can return fewer results than expected. Solutions per Anthropic: scope searches narrowly, move the project to `/home/`, or run Claude Code natively on Windows. *Note: not a hang either, but it affects perceived responsiveness.*
- **VS Code Remote-WSL env propagation.** As above, the extension host inherits the non-interactive shell env. Anything you set with `export` in `.bashrc` interactive sections won't be visible. Use `~/.profile` for env that needs to be present in non-interactive contexts, or just pin everything into `~/.claude/settings.json` `env` block.

### Known limitations: Bedrock vs first-party Anthropic API

A short list of what Claude Code on Bedrock *cannot* do that direct Anthropic API can, plus the practical asterisks. Source primarily the [official Bedrock docs](https://code.claude.com/docs/en/amazon-bedrock) and various community blog posts.

- **Region-specific model availability.** Modern Claude models (3.7+) require Bedrock cross-region inference profiles (`us.`, `eu.`, `global.` prefixes). `AWS_REGION` must be one of the regions in the profile. Pete needs to pick a region where his target models are available; `aws bedrock list-inference-profiles --region us-east-1` enumerates what's accessible ([source](https://code.claude.com/docs/en/amazon-bedrock#region-issues), [source](https://bespinian.io/en/blog/claude-code-using-aws-bedrock/)).
- **Prompt caching is region-limited.** Cache token counts staying at zero is the symptom. Check [the Bedrock prompt-caching support matrix](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-caching.html#prompt-caching-models) ([source](https://code.claude.com/docs/en/amazon-bedrock#3-configure-claude-code)).
- **Bedrock uses the Invoke API, not Converse.** Has tool-streaming differences. Several closed bugs around eager input streaming on tool definitions ([#26941](https://github.com/anthropics/claude-code/issues/26941), [#33180](https://github.com/anthropics/claude-code/issues/33180)). Generally fixed but worth knowing if tool use seems slow.
- **Per-account rate limits.** Bedrock enforces RPM/TPM per AWS account. Default for Opus 4.6 is 25 RPM, upgradable to 500. One AWS account = one team max, realistically ([source](https://bespinian.io/en/blog/claude-code-using-aws-bedrock/)).
- **Extended thinking on Opus 4.7 in Bedrock previously had a `thinking.type.enabled` parameter validation bug** ([#50100](https://github.com/anthropics/claude-code/issues/50100), closed). Resolved but watch for similar.
- **`/logout` is unavailable on Bedrock.** Authentication is handled through AWS creds, so the CLI logout flow doesn't apply ([source](https://code.claude.com/docs/en/amazon-bedrock#3-configure-claude-code)).
- **`/login` not applicable.** Same reason. If the VS Code extension shows "Please run /login," it means `disableLoginPrompt` isn't set or the extension thinks it doesn't have Bedrock creds (often: missing `CLAUDE_CODE_USE_BEDROCK=1` in the extension-host env).
- **`global.*` inference profile auto-selected when not desired.** [#51483](https://github.com/anthropics/claude-code/issues/51483) (closed) reported `/model` picker selecting `global.*` over regional `eu.*` / `apac.*` on non-US `AWS_REGION`. Resolved but worth pinning the model explicitly via `ANTHROPIC_DEFAULT_SONNET_MODEL` / `ANTHROPIC_DEFAULT_OPUS_MODEL` / `ANTHROPIC_DEFAULT_HAIKU_MODEL` to avoid surprises.
- **`ANTHROPIC_MODEL` environment variable ignored on Bedrock in VS Code extension** ([#19011](https://github.com/anthropics/claude-code/issues/19011), open as of 2026-01-18). Workaround per the dev.to writeup: open `/model` in the prompt box and pick a model (even "Default"), which forces the extension to refresh and use the correct Bedrock model IDs. Or use the documented `ANTHROPIC_DEFAULT_*_MODEL` vars (which *are* honored, per the Bedrock docs' "Pin model versions" section).
- **No MCP-related Bedrock limitations specifically.** MCP servers work on Bedrock the same way as direct API.

What's the same:
- Tool use, hooks, MCP, plugins, skills, slash commands, sessions, checkpointing, IDE diff viewing, `@`-mentions, plan mode, auto-edit mode — all work identically. The VS Code extension is provider-agnostic above the auth layer.
- Mantle endpoint (Anthropic API shape over Bedrock auth) is available via `CLAUDE_CODE_USE_MANTLE=1` for orgs whose AWS account has Mantle access. Different model lineup. Probably not what Pete needs but documented in [Bedrock docs, "Use the Mantle endpoint"](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint).

### Recommended settings.json for Pete

Put this in `~/.claude/settings.json`. It pins everything into the settings file so the VS Code extension host doesn't depend on inheriting shell env, uses the documented IdC pattern, and includes the auth-refresh hook. Replace `pete-bedrock` with Pete's actual SSO profile name, and `us-east-1` with the right region for his target models.

```json
{
  "awsAuthRefresh": "aws sso login --profile pete-bedrock",
  "env": {
    "CLAUDE_CODE_USE_BEDROCK": "1",
    "AWS_PROFILE": "pete-bedrock",
    "AWS_REGION": "us-east-1",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "us.anthropic.claude-sonnet-4-6",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "us.anthropic.claude-opus-4-7-v1",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "us.anthropic.claude-haiku-4-5",
    "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "16384"
  }
}
```

In VS Code settings (Ctrl+,), also:

- `claudeCode.disableLoginPrompt: true`
- (Optional) `claudeCode.useTerminal: true` if Pete prefers CLI-style over the graphical panel. The CLI in the integrated terminal has more features than the panel (see the [docs feature comparison](https://code.claude.com/docs/en/vs-code#vs-code-extension-vs-claude-code-cli)).

If Pete is on a network with TLS inspection (Cornell), **remove the `awsAuthRefresh` line** and run `aws sso login --profile pete-bedrock` manually before launching VS Code. Otherwise the SSO browser flow can get interrupted and Claude Code will loop ([source](https://code.claude.com/docs/en/amazon-bedrock#troubleshooting)).

If `aws sts get-caller-identity` works but Claude Code fails with `Value not present for 'accessToken' in SSO Token`, add this to the profile in `~/.aws/config` ([#14086](https://github.com/anthropics/claude-code/issues/14086) workaround that's still required as of v2.1.22+):

```ini
[profile pete-bedrock]
sso_session = pete-corp
sso_account_id = 123456789012
sso_role_name = BedrockDeveloperAccess
region = us-east-1
credential_process = aws configure export-credentials --profile pete-bedrock
```

## Confidence Assessment

| Claim | Confidence | Why |
|---|---|---|
| The `bash -lic` probe hang (#49583) is the most likely cause of Pete's spinner | **Medium-high** | The bug report's environment (Windows 11, WSL2 Ubuntu, VS Code Remote-WSL, Claude Code extension) matches Pete's exactly. Symptoms verbatim. Reporter grepped the extension bundle and confirmed the `printenv PATH` probe. Bug closed-as-not-planned by github-actions for inactivity, never fixed. Only one GitHub issue documents this specific failure mode; no second-source corroboration outside the report. |
| The config probe 60s timeout (#60045) is a real and common second-place cause | **High** | Issue is open, well-documented with extension log excerpts, second reproducer confirmed. Pete has many MCP servers, which is a documented amplifier. |
| `disableLoginPrompt` is required even when settings.json has `CLAUDE_CODE_USE_BEDROCK=1` | **High** | Official Anthropic docs say so. Multiple community writeups call out this exact trip wire. |
| Extension host env on WSL Remote doesn't inherit interactive shell env | **High** | Standard VS Code Remote-WSL behavior. #51108 confirms exact-same-binary failure across launch contexts. |
| `awsAuthRefresh` 3-minute timeout is the actual timeout value | **Medium** | Documented in #25457 (open docs bug) referencing the v2.1.41 release note. Only one source; not independently corroborated. |
| The `credential_process = aws configure export-credentials --profile ...` workaround is still required in v2.1.22+ | **Medium** | #14086 closed for inactivity but two commenters confirmed it still reproduces in 2.1.9 and 2.1.22. No newer confirmation either way. |
| Bedrock prompt caching is region-limited | **Medium** | Documented but the supported-regions list is on the AWS side and changes frequently. Check before relying on it. |
| Bedrock `/logout` is unavailable | **High** | Explicit in Anthropic Bedrock docs. |
| Pete's Cornell network has TLS inspection that breaks `awsAuthRefresh` browser flow | **Low** | Not verified specifically for Cornell. Documented as a "corporate VPN / TLS-inspection proxies" symptom by Anthropic. If Pete hits this, the symptom is repeated browser tab spawning, not a silent spinner. |
| Pete is using IAM Identity Center (vs. an OIDC IdP federated to IAM) | **Low** | Question framing suggests IdC, but the actual auth target (Cornell IdP? Personal IdC? AWS Organizations IdC?) wasn't specified. The diagnostic playbook is correct regardless. |

## Open Questions

These need Pete's input or a follow-up investigation to nail down:

- **Which AWS account?** Cornell, personal, or some other? Determines what SSO profile structure and region pinning makes sense.
- **What does the Output panel "Claude Code" channel actually say** when Pete reproduces the spinner? That alone usually identifies whether it's #49583 (silent after `launch_claude`), #60045 (60s timeout error), or something else.
- **Is Pete launching VS Code via Remote-WSL** (Windows VS Code connecting to WSL) **or via the Linux build of VS Code inside WSLg**? Different env-inheritance behavior. The diagnostic still works but the fix list is slightly different.
- **Does the CLI work in VS Code's integrated terminal** with the same settings? That's the cleanest A/B for whether the issue is the extension launch path or actually Bedrock.
- **Pete's `.bashrc` content.** If it has `sudo`, `ssh-add`, `nvm use`, network-dependent prompt setup (Starship + slow git probe?), or anything else that might block on no-TTY, that's a likely #49583 trigger.

## Sources

### Official Anthropic / AWS docs

- [Claude Code on Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock) — primary setup guide, credential chain, IAM policy, troubleshooting including auth-loop warning.
- [Use Claude Code in VS Code](https://code.claude.com/docs/en/vs-code) — extension docs, third-party provider section, troubleshooting, internal IDE MCP server description.
- [Environment variables](https://code.claude.com/docs/en/env-vars) — all `CLAUDE_CODE_*`, `ANTHROPIC_*`, `AWS_*`, `DEBUG`, watchdog vars.
- [Authentication (IAM)](https://code.claude.com/docs/en/iam) — auth precedence chain, credential storage locations, `apiKeyHelper`.
- [Troubleshooting](https://code.claude.com/docs/en/troubleshooting) — `/doctor`, WSL search performance, hangs.
- [Settings](https://code.claude.com/docs/en/settings) — scopes (managed / user / project / local), precedence, file locations.
- [AWS Solutions: Guidance for Claude Code with Amazon Bedrock](https://docs.aws.amazon.com/solutions/claude-code-with-amazon-bedrock/) — AWS-published enterprise deployment overview.
- [aws-solutions-library-samples/guidance-for-claude-code-with-amazon-bedrock (GitHub)](https://github.com/aws-solutions-library-samples/guidance-for-claude-code-with-amazon-bedrock) — `ccwb` CLI, IdC as first-class auth option in v2.2+, identity-mode comparison.

### Most-relevant GitHub issues (anthropics/claude-code)

- [#49583](https://github.com/anthropics/claude-code/issues/49583) (CLOSED, not fixed) — WSL2 extension hangs on `bash -lic` PATH probe when `.bashrc` has blocking commands. Pete's most-likely-cause issue.
- [#60045](https://github.com/anthropics/claude-code/issues/60045) (OPEN) — Config probe subprocess 60s timeout on first tab.
- [#51108](https://github.com/anthropics/claude-code/issues/51108) (CLOSED, not fixed) — VS Code extension can't load Bedrock creds while same binary works from shell (Codespaces, generalizes to Remote-WSL).
- [#41064](https://github.com/anthropics/claude-code/issues/41064) (CLOSED) — Claude Code stuck when Bedrock auth expires; valuable comment thread with `UserPromptSubmit` hook workaround.
- [#52978](https://github.com/anthropics/claude-code/issues/52978) (OPEN) — `awsAuthRefresh` broken on versions >0.92, auth link not shown in terminal until 20s timeout.
- [#12421](https://github.com/anthropics/claude-code/issues/12421) (CLOSED) — `awsAuthRefresh` constant loop regression in 2.0.53.
- [#14086](https://github.com/anthropics/claude-code/issues/14086) (CLOSED, still reproduces) — AWS SSO profile fails with "accessToken not present"; `credential_process` workaround.
- [#19011](https://github.com/anthropics/claude-code/issues/19011) (OPEN) — VSCode extension ignores `ANTHROPIC_MODEL` on Bedrock; `/model` picker workaround.
- [#25457](https://github.com/anthropics/claude-code/issues/25457) (OPEN docs) — `awsAuthRefresh` / `awsCredentialExport` 3-minute timeout undocumented.
- [#8361](https://github.com/anthropics/claude-code/issues/8361) — VSCode extension 2.0.0 doesn't allow Bedrock token (the prompt-discovery starting point for this research).
- [#61851](https://github.com/anthropics/claude-code/issues/61851) (OPEN) — Code tab forces enterprise SSO on personal Pro account (different surface but illustrates that the extension has its own auth resolution independent of the rest of the app).

### Community writeups

- [Gunnar Grosch — From Terminal to IDE: Using Claude Code with Bedrock in VS Code and JetBrains](https://dev.to/gunnargrosch/from-terminal-to-ide-using-claude-code-with-bedrock-in-vs-code-and-jetbrains-3gbn) — the clearest `disableLoginPrompt` writeup and troubleshooting matrix.
- [bespinian — Using Claude Code on AWS Bedrock](https://bespinian.io/en/blog/claude-code-using-aws-bedrock/) — full IAM policy, multi-account scaling, model selection per region, pricing.
- [Apply Digital — How to Connect Claude Code for VS Code Extension to AWS Bedrock](https://applydigital.github.io/ai-engineering/reference/claude-setup-vs-code/) — short setup, includes the explicit warning that the extension is "still a little immature and lacks full support for all of Claude's features (e.g. MCP)" and recommends terminal mode.
- [randomwits — Setting up Claude Code with AWS Bedrock and SSO Authentication](https://randomwits.com/blog/setup-claud-code-aws-bedrock-sso-auth) — Okta-flavored SSO walkthrough with the corporate SSL inspection NODE_TLS_REJECT_UNAUTHORIZED note.
- [Steven Ge — Set Up VS Code for Claude Code on WSL](https://gexijin.github.io/vibe/Claude_Code_in_VS_Code_Win) — WSL-native walkthrough, decent reference for the file-system boundary issue.
- [Ali Khallad — Your Missing Guide to Claude Code on Windows & VS Code](https://alikhallad.com/your-missing-guide-to-claude-code-on-windows-vs-code/) — native Windows install walkthrough with three real-world failure modes (PATH, no shell, file system provider).

## Update History

- *(Fresh report; no updates yet.)*

## How This Report Was Generated

- Skill: Research (Claude Code).
- Searches: SearXNG MCP (`search`, `search_deep`, `search_tech`) across Bing, DuckDuckGo, technical engines. Several rounds with progressively more specific queries.
- Documents fetched and read in full: Anthropic docs (amazon-bedrock, vs-code, troubleshooting, env-vars, iam, settings), AWS solutions library README + landing page, five community walkthroughs, two large-output docs read by extracting JSON tool-output files via `jq` + python.
- GitHub issue mining via `gh issue list --repo anthropics/claude-code --search` with three different queries (bedrock vscode, hanging spinner, bedrock SSO) yielding ~70 unique issues, of which ~10 were directly relevant. Full bodies + first 4-6 comments read via `gh issue view --json`.
- All claims cite the source URL inline or in the [Sources](#sources) list. Confidence levels assigned in the [Confidence Assessment](#confidence-assessment) table.
- Two URLs (`medium.com`, `aws.plainenglish.io`) returned 403 to the fetcher. Coverage was not materially affected because their content was either summarized elsewhere or duplicative of dev.to / Apply Digital sources.
- Generated 2026-05-27, America/New_York timezone (system clock UTC was converted before labeling; verified with `TZ='America/New_York' date`).
