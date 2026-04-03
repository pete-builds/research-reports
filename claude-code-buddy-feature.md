---
title: "Claude Code Buddy: Anthropic's Terminal Pet Companion"
date: 2026-04-02
updated: 2026-04-03
status: complete
canary: r-7f3a9c12
summary: "Claude Code Buddy is a terminal pet companion (think Tamagotchi) that launched April 1, 2026 in Claude Code v2.1.89+. Type /buddy to deterministically hatch one of 18 ASCII species at one of 5 rarity tiers. It watches your coding session and drops comments in speech bubbles. Initially discovered in a source code leak (v2.1.88 shipped a 59.8 MB .map file), the feature was officially released the next day. Anthropic confirmed on April 1 it is April Fools' only and will be removed after April 7. The community has built independent plugins and progression specs to keep the concept alive."
---

**TL;DR:** `/buddy` hatches a deterministic ASCII pet in your Claude Code terminal. 18 species, 5 rarity tiers (Common through Legendary), 1% shiny chance, stat system, hats. Your pet is permanently tied to your account ID via hashing, so no rerolling. It watches your coding and occasionally comments in speech bubbles. Requires Claude Code 2.1.89+ and a Pro subscription. Leaked a day early via an accidental source map publish, then officially launched April 1, 2026. **Anthropic confirmed on April 1 it will be removed after April 7 and is not a permanent feature.** The community has responded with independent plugins and a full evolution spec.

## Current Status

- **Live** in Claude Code v2.1.89+ as of April 1, 2026 [source: [CHANGELOG.md](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)]
- **Confirmed temporary:** Anthropic staff member @alii posted April 1: "Unfortunately since this is just a feature for April fools that we will be removing in a few days, we probably won't be adding any progression/upgrades/evolution/etc." [source: [Issue #41867 comment](https://github.com/anthropics/claude-code/issues/41867#issuecomment-april-1)]
- **Removal target: after April 7, 2026.** The `isBuddyLive` flag theory suggesting permanent availability after April 8 is contradicted by Anthropic's own statement.
- Requires **Pro subscription** (not available on free tier) [source: [claudefa.st](https://claudefa.st/blog/guide/mechanics/claude-buddy)]
- Issue #41867 (feature request for customization/progression) is now **closed** [source: GitHub]
- Community has built multiple independent plugins: buddy-evolution, ccbuddy.dev, choose-buddy, any-buddy v2.1.3 [source: GitHub issue comments]
- **Security warning:** Threat actors are distributing Vidar infostealer via fake "leaked Claude Code" repos on GitHub [source: [BleepingComputer](https://www.bleepingcomputer.com/news/security/claude-code-leak-used-to-push-infostealer-malware-on-github/)]

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
| April Fools' only, removed after April 7 | High (Anthropic staff @alii confirmed) |
| Hex-encoding rationale (build scanner bypass) | Medium |
| Permanent availability after April 8 (prior report) | **Retracted** — contradicted by @alii's statement |

## Open Questions

1. **Will Anthropic reconsider?** The feature request got 88+ upvotes and generated a full community spec in under 24 hours. @alii did not rule out future consideration, only the current iteration.
2. **Will buddy-evolution or similar community plugins persist?** Multiple independent implementations exist and do not depend on the native `/buddy` command.
3. **Will Buddy expand to non-Pro tiers?** Currently Pro-only. No indication either way for a future version.
4. **What is the capybara/codename connection?** The hex-encoding theory is plausible but unconfirmed by Anthropic.

## Sources

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
| 2026-04-03 | Major correction: Anthropic confirmed April Fools' only, removal after April 7 (retracted permanent availability claim). Added malware warning (Vidar infostealer via fake GitHub repos). Added community ecosystem section (ccbuddy.dev, choose-buddy, buddy-evolution, buddy-evolution-spec). Noted any-buddy updated to v2.1.3. Issue #41867 now closed. |
| 2026-04-02 | Initial report |
