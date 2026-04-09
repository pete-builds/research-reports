---
title: "Claude Code Buddy: Anthropic's Terminal Pet Companion"
date: 2026-04-02
updated: 2026-04-09
status: complete
canary: r-7f3a9c12
summary: "Claude Code Buddy launched April 1, 2026 in v2.1.89 as an April Fools' terminal pet companion. Anthropic silently removed it on April 8 in v2.1.97 with no changelog mention. The /buddy command now returns 'Unknown skill: buddy.' Anthropic staff confirmed removal is intentional with no immediate plans to restore. Pinning to v2.1.96 is the only workaround. A community campaign ('Bring Back Buddy,' Issue #45596) has 250+ reactions but no official response. The feature was NOT extended past April 7; it was removed the day after."
---

**TL;DR:** `/buddy` hatched a deterministic ASCII pet in your Claude Code terminal. 18 species, 5 rarity tiers (Common through Legendary), 1% shiny chance, stat system, hats. Your pet was permanently tied to your account ID via hashing, so no rerolling. It watched your coding and occasionally commented in speech bubbles. Required Claude Code 2.1.89-2.1.96 and a Pro subscription. Leaked a day early via an accidental source map publish, then officially launched April 1, 2026. **Removed silently in v2.1.97 (April 8) with no changelog entry.** Anthropic staff confirmed removal is intentional: "maybe one day your buddy will return" but "not planned" for now. If your buddy still works, you are on v2.1.96 or earlier and have not updated.

## Current Status

- **REMOVED** as of Claude Code v2.1.97 (released April 8, 2026). The `/buddy` command returns "Unknown skill: buddy." [source: [Issue #45517](https://github.com/anthropics/claude-code/issues/45517)]
- **Removal was silent:** No changelog entry, no deprecation notice, no farewell. The feature simply disappeared between v2.1.96 and v2.1.97. [source: [Claude Code Changelog](https://code.claude.com/docs/en/changelog)]
- **Anthropic confirmed intentional removal.** Staff member Daniel Hudson (@notitatall) on Issue #45517: "Apologies friends, maybe one day your buddy will return. Closing as not planned for now." Issue closed as NOT_PLANNED. [source: [Issue #45517](https://github.com/anthropics/claude-code/issues/45517)]
- **Workaround:** Pinning to v2.1.96 (`npm install -g @anthropic-ai/claude-code@2.1.96`) restores `/buddy`. Multiple users confirmed this works. If your buddy is still alive, you have not updated past v2.1.96. [source: [Issue #45517](https://github.com/anthropics/claude-code/issues/45517)]
- **"Bring Back Buddy" campaign:** Issue #45596 consolidates 8+ related issues, has 142 thumbs-up and 111 heart reactions (250+ total), 49 comments. No official Anthropic response on this issue. [source: [Issue #45596](https://github.com/anthropics/claude-code/issues/45596)]
- **Companion data persists:** `~/.claude.json` still contains the `"companion"` object (name, species, personality, hatchedAt). The data was not cleaned up. [source: [Issue #45596](https://github.com/anthropics/claude-code/issues/45596)]
- **Community replacements emerging:** "Buddi" (macOS Notch app by @grayashh/talkvalue), "claude-buddy" standalone reimplementation (by @1270011), plus earlier tools (any-buddy, ccbuddy.dev, choose-buddy, buddy-evolution) [source: GitHub]
- **Security warning (ongoing):** Threat actors still distributing Vidar infostealer via fake "leaked Claude Code" repos on GitHub [source: [BleepingComputer](https://www.bleepingcomputer.com/news/security/claude-code-leak-used-to-push-infostealer-malware-on-github/)]

## 1. What It Is

Claude Buddy is a virtual ASCII pet that lives in your terminal alongside Claude Code. It observes your conversations and occasionally drops comments in speech bubbles. You can also talk to it directly by name. Its speech bubbles run on independent logic, so it has zero impact on Claude's response speed or quality. [source: [claudefa.st](https://claudefa.st/blog/guide/mechanics/claude-buddy)]

**Confidence: High** (multiple corroborating sources, confirmed in official changelog)

## 2. How to Activate

| Command | Effect |
|---------|--------|
| `/buddy` | First-time hatch with animation |
| `/buddy pet` | Floating heart animation (2.5 seconds) |
| `/buddy card` | View stat card with sprite and rarity |
| `/buddy mute` | Silence speech bubbles |
| `/buddy unmute` | Restore speech |
| `/buddy off` | Hide buddy entirely |

You can also address your buddy by name for LLM-powered conversation. [source: [claudefa.st](https://claudefa.st/blog/guide/mechanics/claude-buddy)]

**Confidence: High**

## 3. Species and Rarity

### 18 Species

Axolotl, Blob, Cactus, Capybara, Cat, Chonk, Dragon, Duck, Ghost, Goose, Mushroom, Octopus, Owl, Penguin, Rabbit, Robot, Snail, Turtle. Each has unique ASCII art sprites (5 lines, 12 characters, 3 animation frames). [source: [apiyi.com](https://help.apiyi.com/en/claude-code-buddy-terminal-pet-companion-activation-guide-en.html)]

### 5 Rarity Tiers

| Tier | Drop Rate | Notes |
|------|-----------|-------|
| Common | 60% | Basic appearance |
| Uncommon | 25% | Hat accessories unlocked |
| Rare | 10% | Extended customization |
| Epic | 4% | Exclusive accessories |
| Legendary | 1% | Maximum stats, "Tiny Duck" hat exclusive |

Independent 1% chance for a "Shiny" variant with rainbow shimmer effects, making a Shiny Legendary a 0.01% occurrence. [source: [claudefa.st](https://claudefa.st/blog/guide/mechanics/claude-buddy)]

**Confidence: High**

## 4. Stats and Attributes

Each buddy has five stats on a 0-100 scale:

- **Debugging**: code issue detection
- **Patience**: interaction gentleness
- **Chaos**: unpredictability level
- **Wisdom**: technical insight depth
- **Snark**: commentary sharpness

Every pet has one peak and one valley attribute. Higher rarities grant more total stat points. [source: [apiyi.com](https://help.apiyi.com/en/claude-code-buddy-terminal-pet-companion-activation-guide-en.html)]

**Eye styles**: 6 options (centered dot, star, x, bullseye, at-sign, degree). **Hats**: 8 types unlocked by rarity tier. [source: [apiyi.com](https://help.apiyi.com/en/claude-code-buddy-terminal-pet-companion-activation-guide-en.html)]

**Confidence: High**

## 5. Technical Implementation

### Deterministic Generation

Your buddy is generated from your account UUID using FNV-1a hashing with Mulberry32 PRNG and salt `friend-2026-401`. Same account always produces the same species, rarity, and stats. Cannot be manipulated by editing config files. [source: [claudefa.st](https://claudefa.st/blog/guide/mechanics/claude-buddy)]

### "Bones vs Soul" Architecture

The system splits buddy data into two categories:

- **Bones** (species, rarity, stats): Recomputed every session from your user ID, never persisted to disk
- **Soul** (name, personality, hatch date): Generated once via LLM call, stored in global config

This design prevents config file manipulation since the immutable traits are recalculated from the account hash each time. [source: [claudefa.st](https://claudefa.st/blog/guide/mechanics/claude-buddy)]

### Hex-Encoded Species Names

All 18 species names in the source code are hex-encoded rather than stored as plain strings. Evidence suggests "capybara" matches an internal model codename, and engineers hex-encoded all species uniformly to bypass Anthropic's own `excluded-strings.txt` build scanner. [source: [claudefa.st](https://claudefa.st/blog/guide/mechanics/claude-buddy)]

**Confidence: Medium** (hex-encoding confirmed in source; the "why" is inference from the claudefa.st analysis)

## 6. Discovery via Source Code Leak

On March 31, 2026, security researcher Chaofan Shou discovered that Claude Code v2.1.88 shipped with a 59.8 MB `.map` file in the npm package that exposed 512,000+ lines of TypeScript across roughly 1,900 files. The `src/buddy/` directory leaked entirely (approximately 79KB across 5 files). The cause was a missing `.npmignore` entry. [source: [claudefa.st](https://claudefa.st/blog/guide/mechanics/claude-buddy)]

Anthropic's statement: "No sensitive customer data or credentials were involved or exposed. This was a release packaging issue caused by human error." [source: [claudefa.st](https://claudefa.st/blog/guide/mechanics/claude-buddy)]

The feature was officially launched the next day (April 1) in v2.1.89. [source: [CHANGELOG.md](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)]

### Malware Warning (April 2, 2026)

Threat actors are exploiting the leak by creating fake GitHub repositories posing as the leaked source. Searching for "leaked Claude Code" on Google surfaces these repos, which deliver a Rust-based executable (ClaudeCode_x64.exe) containing the Vidar infostealer and GhostSocks proxying tool. The primary fake repo is operated by user "idbzoomh" and advertises "unlocked enterprise features." Do not download files from unofficial sources claiming to offer the leaked code. [source: [BleepingComputer, April 2 2026](https://www.bleepingcomputer.com/news/security/claude-code-leak-used-to-push-infostealer-malware-on-github/)]

**Confidence: High**

## 7. Community Response

Despite Anthropic confirming Buddy is temporary, the community has built extensively around it:

- **ccbuddy.dev**: Browser-based buddy selector/roller tool built by @vancik01 [source: [ccbuddy.dev](https://ccbuddy.dev/), Issue #41867]
- **choose-buddy**: CLI tool to override your assigned buddy before removal; advertises "spend your last days with a Legendary dragon" [source: [GitHub, MahammadNuriyev62](https://github.com/MahammadNuriyev62/choose-buddy), Issue #41867]
- **any-buddy** (v2.1.3, April 3): Interactive CLI with species selection, rarity override, binary patching, and live ASCII preview [source: [GitHub, cpaczek](https://github.com/cpaczek/any-buddy)]
- **buddy-evolution** (v1.2): Plugin by @FrankFMY adding 12 personality types, activity tracking, and session summaries; integrates with Claude's personality layer rather than the native `/buddy` command [source: [GitHub, FrankFMY](https://github.com/FrankFMY/buddy-evolution), Issue #41867]
- **buddy-evolution-spec** (v3): Community specification repo by @Hegemon78 with branching evolution paths, achievement categories, buddy journal, and universal activity model (beyond coding) [source: [GitHub, Hegemon78](https://github.com/Hegemon78/buddy-evolution-spec)]
- **buddy-customizer plugin** (PR #41921): Adds XP, leveling (Hatchling to Legend), 10 achievements, daily streaks; still open and unmerged [source: [GitHub PR #41921](https://github.com/anthropics/claude-code/pull/41921)]

**Confidence: High** (all sourced from GitHub issue comments and repo pages)

## Confidence Assessment

| Claim | Confidence |
|-------|------------|
| Feature exists and is live in v2.1.89+ | High |
| 18 species, 5 rarity tiers, 1% shiny | High |
| Deterministic FNV-1a hash with Mulberry32 PRNG and salt `friend-2026-401` | High |
| Bones vs Soul architecture | High |
| Pro subscription required | High |
| Removed in v2.1.97 (April 8), no changelog entry | High (confirmed by multiple users, Anthropic staff closed as NOT_PLANNED) |
| Pinning to v2.1.96 restores buddy | High (multiple user confirmations) |
| Hex-encoding rationale (build scanner bypass) | Medium |
| Permanent availability after April 8 (prior report) | **Retracted** — removed on April 8, opposite of predicted |
| "Maybe one day" return | Low (vague staff comment, no commitment) |

## Open Questions

1. **Will Anthropic bring buddy back?** Staff said "maybe one day" but closed the bug as NOT_PLANNED. The "Bring Back Buddy" campaign (Issue #45596, 250+ reactions) has received no official response. Community sentiment is strongly in favor, but Anthropic has given no timeline or commitment.
2. **Why was the removal silent?** No changelog entry, no deprecation notice, no blog post. This is unusual for a feature that generated significant community engagement. Was it a deliberate quiet removal to avoid amplifying the conversation?
3. **How long is pinning to v2.1.96 viable?** Users staying on v2.1.96 miss security fixes, new features, and eventual API changes. This is not a sustainable workaround.
4. **Will community replacements fill the gap?** Standalone tools like "Buddi" (Notch app) and "claude-buddy" (reimplementation) exist but lack the native integration of the original.
5. **What is the capybara/codename connection?** The hex-encoding theory is plausible but unconfirmed by Anthropic.

## Sources

- [BUG: /buddy command completely missing in v2.1.97 (Issue #45517)](https://github.com/anthropics/claude-code/issues/45517) - Bug report confirming removal; Anthropic staff response "not planned"
- [Bring Back Buddy: A Consolidated Plea from the Community (Issue #45596)](https://github.com/anthropics/claude-code/issues/45596) - Community campaign with 250+ reactions
- [Claude Code Changelog (official)](https://code.claude.com/docs/en/changelog) - v2.1.97 release notes (no buddy mention)
- [claude-buddy: Keep Your Claude Code Buddy Forever (GitHub, 1270011)](https://github.com/1270011/claude-buddy) - Standalone reimplementation
- [Buddi: Notch app replacement (GitHub, talkvalue)](https://github.com/talkvalue/Buddi) - macOS Notch app with same species/rarity system
- [Claude Buddy: Anthropic April Fools Terminal Tamagotchi (claudefa.st)](https://claudefa.st/blog/guide/mechanics/claude-buddy) - Primary technical analysis
- [Enable Claude Code Buddy terminal pet: complete guide (apiyi.com)](https://help.apiyi.com/en/claude-code-buddy-terminal-pet-companion-activation-guide-en.html) - Species and rarity details
- [Claude Code CHANGELOG.md (GitHub)](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) - Official release notes
- [Claude Code Leaked Source: BUDDY, KAIROS & Every Hidden Feature (WaveSpeed AI)](https://wavespeed.ai/blog/posts/claude-code-leaked-source-hidden-features/) - Leak analysis
- [Always-on agent and AI pet Buddy (The Week)](https://www.theweek.in/news/sci-tech/2026/04/01/always-on-agent-and-ai-pet-buddy-anthropics-claude-source-code-leak-reveals-hidden-features.html) - News coverage
- [PR #41921: buddy-customizer plugin (GitHub)](https://github.com/anthropics/claude-code/pull/41921) - Community gamification extension
- [any-buddy: Hack Claude Code to get any buddy (GitHub)](https://github.com/cpaczek/any-buddy) - Community reroll tool (v2.1.3)
- [Leaked Claude Code Shows Anthropic Building Tamagotchi Feature Into It (Futurism)](https://futurism.com/artificial-intelligence/leaked-claude-code-tamagotchi) - News coverage
- [Issue #41867: Buddy customization & progression (GitHub)](https://github.com/anthropics/claude-code/issues/41867) - Community feature request; @alii confirms April Fools' scope (April 1, 2026)
- [Claude Code leak used to push infostealer malware on GitHub (BleepingComputer)](https://www.bleepingcomputer.com/news/security/claude-code-leak-used-to-push-infostealer-malware-on-github/) - Malware campaign warning (April 2, 2026)
- [buddy-evolution spec (GitHub, Hegemon78)](https://github.com/Hegemon78/buddy-evolution-spec) - Community evolution specification
- [buddy-evolution plugin (GitHub, FrankFMY)](https://github.com/FrankFMY/buddy-evolution) - Community plugin v1.2
- [ccbuddy.dev](https://ccbuddy.dev/) - Community buddy selector tool

## Update History

| Date | Change |
|------|--------|
| 2026-04-09 | Buddy confirmed REMOVED in v2.1.97 (April 8). No changelog entry. Anthropic staff closed bug as NOT_PLANNED: "maybe one day your buddy will return." Pinning to v2.1.96 is the only workaround. "Bring Back Buddy" campaign (Issue #45596) has 250+ reactions, no official response. Added community replacements: Buddi (Notch app), claude-buddy (standalone). Updated status from "live" to "removed." Pete's buddy still works because he hasn't updated past v2.1.96. |
| 2026-04-03 | Major correction: Anthropic confirmed April Fools' only, removal after April 7 (retracted permanent availability claim). Added malware warning (Vidar infostealer via fake GitHub repos). Added community ecosystem section (ccbuddy.dev, choose-buddy, buddy-evolution, buddy-evolution-spec). Noted any-buddy updated to v2.1.3. Issue #41867 now closed. |
| 2026-04-02 | Initial report |
