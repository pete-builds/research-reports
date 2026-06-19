---
title: "Setlist Predictor Games: Umphreys McGee, Phish, and the Jam Band Community"
date: 2026-06-18
updated: 2026-06-18 11:08 PM EDT
summary: "Setlist prediction games are a thriving micro-genre of fan engagement in the jam band world, ranging from official band contests to community-built apps, open-source self-hosted platforms, and real-time interactive bingo formats. Phish and Goose have the richest ecosystems; Umphreys McGee occupies a unique position through both official UMBowl interaction and an emerging open-source game engine purpose-built for the community."
---

## TL;DR

Setlist prediction games let fans guess which songs a band will play before (or during) a show, scoring points for correct calls. The jam band world has developed this into a real genre: from fan-run spreadsheets to polished iOS apps, official band contests, and self-hosted open-source platforms. Phish and Goose have the most app infrastructure; Umphreys McGee has a history of official fan interaction through UMBowl events and an emerging open-source game engine.

## Current Status

- Multiple dedicated mobile apps (Pick 5, Pick Phive, Flodown, Zabriskie, DMB Hub) serve specific jam band fanbases with setlist prediction, live scoring, and leaderboards.
- Open-source platform `open-setlist-stash` (GitHub: pete-builds/open-setlist-stash) offers a production-grade, forkable setlist prediction game for any band, with Phish as the reference deployment ("Tweezer Picks").
- Umphreys McGee lacks a dedicated prediction app but has a long history of official fan interaction through UMBowl events, where audience votes and text-message submissions directly shape the setlist in real time.
- Goose took the most theatrical approach: a 2020 "Bingo Tour" where a live stage bingo draw determined the setlist, with fans playing along on digital bingo cards for prizes.
- Phantasy Tour remains the broadest web-based community for jam band setlist prediction discussion, though its tooling is informal compared to dedicated apps.
- Scoring systems vary widely: some apps use rarity-weighted point formulas (favoring bust-outs and long-gap songs), others use flat points-per-correct-pick, and some (Zabriskie) explicitly reject global competitive ranking in favor of per-show community celebration.

## Table of Contents

- [TL;DR](#tldr)
- [Current Status](#current-status)
- [Findings](#findings)
  - [1. What Are Setlist Predictor Games?](#1-what-are-setlist-predictor-games)
  - [2. Umphreys McGee and Setlist Prediction](#2-umphreys-mcgee-and-setlist-prediction)
  - [3. Other Bands with Setlist Games](#3-other-bands-with-setlist-games)
  - [4. Platforms and Distribution Methods](#4-platforms-and-distribution-methods)
  - [5. Community and Scoring](#5-community-and-scoring)
  - [6. Tools and Apps](#6-tools-and-apps)
- [Confidence Assessment](#confidence-assessment)
  - [High Confidence](#high-confidence)
  - [Medium Confidence](#medium-confidence)
- [Open Questions](#open-questions)
- [Sources](#sources)
- [How This Report Was Generated](#how-this-report-was-generated)

## Findings

### 1. What Are Setlist Predictor Games?

Setlist predictor games are contests in which fans attempt to guess which songs a band will perform at an upcoming show, before the setlist is revealed. The genre emerged organically from the jam band community, where no two shows are identical, setlists are constructed (not repeated), and "bust-out" songs (rarely played tracks) carry social cachet.

**Core mechanics:**

- **Pre-show prediction:** Players submit picks before showtime. A "lock window" closes submissions at or before doors/start time to prevent live leaking.
- **Slot predictions:** More sophisticated games allow predicting not just songs but positions: opener, set closer, encore. Correct slot placement earns bonus points.
- **Rarity weighting:** The most interesting scoring systems reward harder predictions. A "bust-out" song not played in 80 shows is worth more than a song played three times this run.
- **Auto-resolve:** After the show, the actual setlist (from authoritative sources like phish.net or allthings.umphreys.com) is compared to predictions and scores are computed automatically.
- **Leaderboards:** Weekly, tour, and all-time standings let fans track their prediction skills over time.

The genre draws on similar psychology to fantasy sports: domain knowledge, statistical research, and timing create competitive advantage. Fan databases like phish.net, setlist.fm, and All Things Umphreys are essential infrastructure for these games, supplying song history, gap counts, and venue data.

**Formats that exist:**

1. **Prediction contests** - Pick songs before showtime, score after. (Most apps)
2. **Live song calling** - Guess the next song during set break or mid-show. (Zabriskie)
3. **Bingo formats** - Cards pre-filled with potential songs; mark off as played. (Goose Bingo Tour)
4. **Real-time audience choice** - The band actually plays what the audience votes for. (UMBowl)
5. **Fantasy setlist** - Build your dream setlist, not a prediction. (Casual/party format)

[source: gemini-search, open-setlist-stash PHASE-4-PLAN.md]

---

### 2. Umphreys McGee and Setlist Prediction

Umphreys McGee sits in a distinct position in this space. The band does not have a single official ongoing setlist prediction game with a dedicated platform. However, they have the deepest tradition of *audience-driven setlist determination* through their UMBowl events, which go further than prediction: the audience actually controls the show.

**UMBowl (2010-present):**

UMBowl is an annual concert event structured like a football game, with four "quarters" each featuring a different interactive theme. Fan interaction is central to the format:

- **Raw Stewage:** Fans vote ahead of the show on their favorite previously improvised passages. The band develops these into new songs, sometimes debuted at the event. This fan-driven process has influenced album compositions (e.g., "Blueprints"). [source: gemini-search]
- **S2 Events:** Intimate sessions (~50 fans) where the band improvises entirely on text submissions from the audience.
- **Choose Your Own Adventure Sets:** The audience votes live on which songs to play from a pre-selected list.
- **Real-time text voting:** Fans send text messages during the show to influence the setlist and improv themes in real time.

**"Get Lucky with Raw Stewage" prediction contest:**

Separately from UMBowl, Umphreys has run prediction contests around specific segments. In one documented example, fans submitted predictions for which "Raw Stewage" improvised pieces the band would perform, with correct guesses earning show download credits. The contest required 100% accuracy to win; ties were broken by odds of the selected segments. Submission was via tweet to `@McLSucks` or email. [source: gemini-search]

**Open-source game engine for Umphreys:**

The `open-setlist-stash` project (see Section 6) was built with band-agnostic forking in mind. A companion project, `mcp-umphreys`, provides an MCP server with tool output shapes byte-for-byte compatible with the `mcp-phish` contract that `open-setlist-stash` reads. This means an Umphreys-branded instance of the game ("Umphreys Picks" or similar) could be deployed by any community operator by forking the platform and pointing it at `mcp-umphreys` instead of `mcp-phish`.

The `umphreys-vault` project supplies the historical data layer: a full Postgres corpus of Umphreys setlists sourced from the All Things Umphreys (ATU) REST API v2, with computed gap counts, debut dates, and times-played per song across ~1,128 songs in the catalog. [source: /Users/ps959/ai-cli-workspace/Mine/Self-Hosted/umphreys-vault/README.md, /Users/ps959/ai-cli-workspace/Mine/Self-Hosted/mcp-umphreys/README.md]

**All Things Umphreys (allthings.umphreys.com):**

The primary fan-run Umphreys data resource. It functions as a comprehensive archive with its own REST API (v2, no auth required), covering all setlist entries, song catalog, venues, jam chart entries, and guest appearances. It is the authoritative upstream source for any Umphreys setlist game. [source: /Users/ps959/ai-cli-workspace/Mine/Self-Hosted/umphreys-vault/README.md]

---

### 3. Other Bands with Setlist Games

**Phish**

Phish has the richest prediction game ecosystem. The band's highly variable setlists, massive song catalog (~300+ active songs), and dedicated data infrastructure (phish.net, phish.in) make it ideal for structured prediction games.

- **Pick Phive** (iOS, App Store ID 1454153920): Phish-specific prediction app. Pre-show song picks, live scoring as songs are called, competition among fans.
- **Pick 5 - Predict The Setlist App** (iOS, App Store ID 1611794276): Multi-band app that covers Phish prominently. Rarity-weighted scoring using a 30-show sliding window with linear decay (recent plays worth less; stale songs worth more). New songs start at 100 points.
- **open-setlist-stash / Tweezer Picks**: Open-source self-hosted platform with Phish as the reference deployment. See Section 6 for full detail.
- **Phish.net**: The authoritative Phish data archive; not itself a prediction game, but the data backbone all other tools rely on. [source: gemini-search]

**Goose**

Goose occupies a unique position by having run an actual bingo-format event that directly determined the setlist:

- **Bingo Tour (June 2020):** A two-week live-streamed concert series during the COVID-19 pandemic. Fans purchased tickets that came with digital bingo boards filled with song names, covers, and musical directives (e.g., "20-minute jam," specific cover songs). Stage manager "Coach" Lombardi drew bingo balls live, and the result directly determined what the band played next. Winning patterns earned prizes (VIP experiences, merch). Generated over $100,000 for the band. Streamed on LivexLive. [source: gemini-search]
- **Flodown** (iOS, App Store ID 1554907406): Goose-specific fan app with setlist prediction game, show archive, and community forums. Leaderboards and prediction stats tracking.

**Dave Matthews Band**

- **DMB Hub** (official app): Features a full setlist prediction game ("Setlist Game") with drag-and-drop pick submission, live tracking, season standings, all-time stats, and league competition. Recently added "Ants Intelligence Predictions" showing AI-generated next-song predictions. Also includes live trivia and ShowFlow per-song voting. [source: gemini-search]

**Grateful Dead / Dead & Company**

No official prediction game is documented. Fan-created party bundles (printable, sold on Etsy) include "Dream Setlist" cards. The GD catalog's size and the culture of unexpected song choices make prediction games appealing in theory. Dead & Company is supported in Zabriskie. [source: gemini-search]

**Widespread Panic**

Supported in Pick 5. No dedicated platform found. [source: gemini-search]

**Billy Strings**

Supported in Pick 5 and Zabriskie. No dedicated platform found. [source: gemini-search]

**String Cheese Incident**

No dedicated prediction game or platform documented. The band's varied setlists make them a natural fit; fan prediction likely happens informally on community forums. [source: gemini-search, confidence: medium]

**Pearl Jam**

Fan-created "Dark Matter Bingo" (named after their 2024 album) documented in concert reviews; entirely informal. [source: gemini-search]

**King Gizzard and the Lizard Wizard, My Morning Jacket, Spafford**

Supported in Zabriskie for live song calling and community features, but no dedicated prediction games found. [source: gemini-search]

---

### 4. Platforms and Distribution Methods

Setlist prediction games are distributed across four main channels:

**Native mobile apps (iOS)**

The dominant distribution model for polished setlist prediction experiences. All major prediction apps are iOS-first:

| App | Primary Band | Distribution |
|-----|-------------|--------------|
| Pick Phive | Phish | iOS App Store (ID: 1454153920) |
| Pick 5 - Predict The Setlist | Multi-band | iOS App Store (ID: 1611794276) |
| Flodown | Goose | iOS App Store (ID: 1554907406) |
| Zabriskie | Multi-band jam | iOS App Store (ID: 1643905500) |
| DMB Hub | Dave Matthews Band | iOS App Store (official) |

No Android equivalents were identified for these apps, though the market may have them. [source: gemini-search]

**Web-based community platforms**

- **Phantasy Tour** (phantasytour.com): Long-running jam band community forum with informal setlist prediction features. Users post predictions in forum threads; no automated scoring. The "five-dollar March Madness pool" model is common. [source: gemini-search]
- **Phish.net**: Data backbone; no prediction game, but the API is the engine powering most Phish-specific tools.
- **All Things Umphreys** (allthings.umphreys.com): Data archive; no prediction game directly, but public REST API enables community tools.
- **setlist.fm**: Global multi-artist setlist archive; no prediction game found as of research date. [source: gemini-search]

**Self-hosted open-source**

- **open-setlist-stash** (github.com/pete-builds/open-setlist-stash): Docker-based, forkable, band-agnostic setlist prediction platform. Deploy on your own homelab or VPS. Phish reference deployment at "Tweezer Picks." See Section 6. [source: /Users/ps959/ai-cli-workspace/Mine/Self-Hosted/open-setlist-stash/README.md]

**Official band contests**

- **Umphreys McGee** has run at-show contests via tweet/email with prize downloads.
- **Goose Bingo Tour** ran as an official ticketed event with digital bingo boards delivered via email.
- **DMB Hub** is an official app distributed and maintained with band involvement.

**Informal / social media**

Fan communities on Reddit (r/phish, r/UmphreysMcGee, r/goosetheband), Discord servers, and Twitter/X run informal prediction threads before major shows, particularly for anticipated bust-outs and run openers. No automated scoring; social proof only.

---

### 5. Community and Scoring

**Scoring philosophies**

Different platforms reflect genuinely different philosophies about what prediction games are for:

1. **Rarity-weighted competitive** (Pick 5, open-setlist-stash): The game rewards statistical knowledge. Predicting a song with a 95-show gap is worth far more than predicting a staple. This philosophy treats the contest as a skill game where deep catalog knowledge confers advantage. The open-setlist-stash formula uses `round(10 * log2(1 + gap_current) * (200 / max(20, times_played)))` per pick, plus flat slot bonuses (+25 opener, +25 closer, +30 encore). [source: /Users/ps959/ai-cli-workspace/Mine/Self-Hosted/open-setlist-stash/PHASE-4-PLAN.md]

2. **Community celebration** (Zabriskie): The app explicitly rejects global all-time leaderboards. Correct song calls earn per-show recognition and a celebratory animation. The goal is to share the moment of calling a song, not to build a competitive ranking. [source: gemini-search]

3. **Social competition** (DMB Hub): Season standings, league play, friend comparisons. Positioned more like fantasy sports with social/group features as the engagement hook.

4. **Event-based prizes** (Goose Bingo Tour, Umphreys Raw Stewage): Prizes like VIP access, merch, or show downloads. No ongoing leaderboard; the game is the event.

**Anti-cheating measures**

The most sophisticated self-hosted platforms include technical safeguards to ensure fair play:

- **Lock windows:** Predictions close at showtime (or a configurable pre-show cutoff). open-setlist-stash uses both application-layer rejection (409 Conflict) and a database-level trigger that blocks any INSERT past lock time, regardless of how the data arrives.
- **AI assist gating:** open-setlist-stash disables gap statistics, venue history, and statistical hints during the prediction window. These "assist" features unlock post-lock as a retro/review tool. The song search autocomplete returns only title and slug (no gap counts) before lock. [source: /Users/ps959/ai-cli-workspace/Mine/Self-Hosted/open-setlist-stash/PHASE-4-PLAN.md]
- **Lockout windows in Zabriskie:** Pre-show opener calls enter a lockout window before the show begins to prevent transcribers from seeing leaked setlists and entering early. [source: gemini-search]

**Community data dependencies**

All setlist prediction games depend on authoritative setlist databases as infrastructure:

- **phish.net**: Phish setlists since 1983, with a public API.
- **phish.in**: Phish audio, used by mcp-phish for the Phish data platform.
- **allthings.umphreys.com**: Umphreys setlist archive, public REST API v2.
- **setlist.fm**: Multi-artist setlist database, used as a fallback or secondary source for non-jam bands.

These databases publish setlists post-show, sometimes within 30 minutes, sometimes overnight. Auto-resolve systems poll on a schedule (typically every 30 minutes) and score predictions when setlist data appears.

---

### 6. Tools and Apps

**open-setlist-stash** [source: github.com/pete-builds/open-setlist-stash, /Users/ps959/ai-cli-workspace/Mine/Self-Hosted/open-setlist-stash/]

The only known open-source, self-hostable setlist prediction game engine. Key characteristics:

- **Stack:** Python 3.13, FastAPI, Jinja2, HTMX, PostgreSQL, Docker
- **Band-agnostic design:** The `SITE_NAME` environment variable controls all branding. Theme is swappable via CSS override (private theme mount or bundled). Any band community can fork and rebrand.
- **Reference deployment:** "Tweezer Picks" — a Phish-specific instance running on a private homelab (LAN/Tailscale only through Phase 5; public exposure planned for Phase 6).
- **Data source:** All vault data is read via the `mcp-phish` MCP server, never directly from the Postgres vault. The game only owns its own game-state Postgres (users, predictions, leaderboards, scoring audit).
- **Umphreys extension:** `mcp-umphreys` is a drop-in compatible MCP server for Umphreys data, allowing an Umphreys-branded instance of the same game to be deployed by any operator.
- **Scoring formula:** Rarity-weighted (gap + times played), plus slot bonuses for opener/closer/encore prediction. Scores stored with a full JSONB breakdown per pick.
- **Auth:** Anonymous handles via signed session cookie (Phase 4). Magic-link email auth planned (Phase 4b).
- **Fairness:** Pre-lock assist disable, DB lock guard trigger, configurable lockout policy.
- **CI:** ruff, mypy --strict, pytest, Trivy image scan, Dependabot; 167 tests.
- **License:** MIT

**Pick 5 - Predict The Setlist** [source: gemini-search, App Store ID 1611794276]

Multi-band iOS app. Covers Phish, Widespread Panic, Billy Strings, Goose, and others. Scoring uses a 30-show sliding window with linear decay. Song predictions, opener calls, deeper prop bets (bust-outs, Type II jams). Community leaderboards, band-separated friend lists.

**Pick Phive** [source: gemini-search, App Store ID 1454153920]

Phish-specific iOS app. Live scoring during shows. Fan competition. iPhone only (not iPad).

**Flodown** [source: gemini-search, App Store ID 1554907406]

Goose-specific iOS app. Setlist prediction game, show archive, community forums. Leaderboard and prediction stat tracking.

**Zabriskie** [source: gemini-search, App Store ID 1643905500]

Multi-band iOS app. Bands include: Goose, Phish, Trey Anastasio Band, Umphrey's McGee, Dead & Company, Billy Strings, My Morning Jacket, Spafford, King Gizzard, Dogs in a Pile, and others. Key features: real-time song calls (tap to call, fuzzy-matched against catalog), lockout window before shows, per-show leaderboard, post-tour Jam Bracket Tournaments (March Madness-style, seeded by live-chat sentiment analysis of heaters). Intentionally avoids global all-time ranking.

**DMB Hub** [source: gemini-search]

Official Dave Matthews Band fan app. Setlist Game with drag-and-drop picks, season standings, league play, AI-assisted next-song predictions ("Ants Intelligence"), live ShowFlow per-song voting, personal show history and achievement Stubs.

**Goose Bingo Tour** (historical) [source: gemini-search]

June 2020 live-streamed event. Digital bingo boards distributed via email to ticket holders. Stage manager drew bingo balls live, directly determining the setlist. Winners emailed goosebingotour@gmail.com. Streamed on LivexLive. Raised over $100,000.

**Phantasy Tour** [source: gemini-search]

Web forum (phantasytour.com). Long-running jam band community with informal prediction threads. No automated scoring; peer-verified. Common format is a pool-style contest with a small buy-in.

---

## Confidence Assessment

### High Confidence

- The `open-setlist-stash` project details, scoring formula, data model, and architecture are directly verified from source code in this workspace. [source: /Users/ps959/ai-cli-workspace/Mine/Self-Hosted/open-setlist-stash/]
- `umphreys-vault` and `mcp-umphreys` READMEs confirm the Umphreys data infrastructure and MCP compatibility with the game engine. [source: /Users/ps959/ai-cli-workspace/Mine/Self-Hosted/umphreys-vault/README.md, /Users/ps959/ai-cli-workspace/Mine/Self-Hosted/mcp-umphreys/README.md]
- Goose Bingo Tour mechanics and LivexLive platform verified via search results.

### Medium Confidence

- App Store IDs and feature descriptions for Pick 5, Pick Phive, Flodown, Zabriskie, and DMB Hub are sourced from Gemini search results only. App Store URLs return 404 via automated fetch (App Store actively blocks web scrapers), so existence and feature accuracy could not be independently confirmed. Claims are consistent with community discussion but should be verified manually by visiting the App Store.
- UMBowl structure (Raw Stewage, S2, Choose Your Own Adventure, text voting) is well-established community knowledge but specific details (e.g., the "Blueprints" album attribution) are sourced from a single Gemini result with no independently fetched primary source.
- Goose Bingo Tour $100K+ fundraising claim is sourced from Gemini search only; no independently fetched source confirmed the dollar figure.
- Phish active-song-count estimate (~300+) is sourced from Gemini search; phish.net was unreachable for direct verification.
- Phantasy Tour's prediction game mechanics are described via community reports rather than official documentation. The "five-dollar pool" framing comes from community descriptions, not official rules.
- String Cheese Incident, Pearl Jam, and Grateful Dead fan prediction activity is described from secondary sources; no dedicated platforms were identified and this absence may not be complete.
- DMB Hub scoring specifics (exact point system) are not publicly documented in detail; described as "point-based" from app store descriptions.
- App availability and feature sets change over time; iOS App Store URLs and feature descriptions current as of June 2026 but may drift.

---

## Open Questions

1. Is there a dedicated Umphreys community game beyond the UMBowl-adjacent contests? The Umphreys subreddit and UMbase community may run informal prediction threads not captured here.
2. Does Phantasy Tour have a formal scoring system, or is it entirely informal poll/thread-based?
3. Are any of the prediction apps available on Android? All identified apps are iOS-only, which would be a notable gap if accurate.
4. Has open-setlist-stash been publicly deployed anywhere beyond a private homelab, and has any Umphreys-branded fork been deployed?
5. Does phish.net have any plans to add gamification features, or has this been explicitly declined by the maintainers?

---

## Sources

- [open-setlist-stash README](https://github.com/pete-builds/open-setlist-stash) — project description, stack, scoring, branding
- [open-setlist-stash PHASE-4-PLAN.md](/Users/ps959/ai-cli-workspace/Mine/Self-Hosted/open-setlist-stash/PHASE-4-PLAN.md) — scoring formula, data model, lock policy, auth, anti-cheat design
- [umphreys-vault README](/Users/ps959/ai-cli-workspace/Mine/Self-Hosted/umphreys-vault/README.md) — ATU API, schema, gap computation
- [mcp-umphreys README](/Users/ps959/ai-cli-workspace/Mine/Self-Hosted/mcp-umphreys/README.md) — tool contract, hot window, mcp-phish compatibility
- [Pick 5 - Predict The Setlist App](https://apps.apple.com/us/app/pick-5-predict-the-setlist-app/id1611794276) — multi-band prediction app
- [Pick Phive](https://apps.apple.com/us/app/pick-phive/id1454153920) — Phish-specific prediction app
- [Flodown](https://apps.apple.com/us/app/flodown/id1554907406) — Goose-specific prediction app
- [Zabriskie](https://apps.apple.com/us/app/zabriskie/id1643905500) — multi-band jam community app
- [Phantasy Tour](https://www.phantasytour.com/) — jam band community forum
- [All Things Umphreys](https://allthings.umphreys.com) — Umphreys setlist archive and API
- [Phish.net](https://phish.net/) — authoritative Phish setlist archive
- [setlist.fm](https://setlist.fm/) — multi-artist setlist community
- Gemini enterprise web search (Cornell AI Gateway, 2026-06-18) — app details, UMBowl mechanics, Goose Bingo Tour, DMB Hub features, Billy Strings/Billy Strings/Dead & Company community data

---

## How This Report Was Generated

Researched on 2026-06-18 using the Cornell AI Gateway (Gemini 2.5 Flash with enterprise web search) and direct inspection of source code in the `open-setlist-stash`, `umphreys-vault`, and `mcp-umphreys` projects in the ai-cli-workspace. WebFetch was attempted on several App Store and fan-site URLs but was blocked (403/404); those details were sourced from Gemini search results. Claims are grounded in at least one verified source; confidence levels note where direct verification was not possible.
