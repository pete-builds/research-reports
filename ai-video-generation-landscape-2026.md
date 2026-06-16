---
title: "AI Video Generation Landscape and Platforms in 2026"
date: 2026-06-16
updated: 2026-06-16T15:30:00-04:00
summary: "A mid-2026 market overview of AI video generation: the major models (Sora 2, Veo 3.1, Runway Gen-4.5, Kling 3.0, Seedance 2.0 and others), their capabilities and pricing, recent releases, US-vs-China competitive dynamics, and where the field is heading."
---

## TL;DR

As of June 2026, AI video generation has matured into a crowded, fast-moving market where native audio, multi-shot sequencing, and character consistency are now table stakes rather than differentiators. US labs (OpenAI's Sora 2, Google's Veo 3.1, Runway's Gen-4.5) compete on quality and ecosystem integration, but Chinese labs have surged: blind-vote quality leaderboards in mid-2026 are topped by ByteDance's Seedance 2.0, Alibaba's HappyHorse-1.0, and Kuaishou's Kling 3.0 (an early native-4K consumer model). Pricing has fallen sharply and consolidated into multi-model aggregator subscriptions, while regulation (US TAKE IT DOWN Act, state deepfake laws, C2PA provenance) races to catch up.

## Current Status

- **Quality leaders (mid-2026, by blind human vote on Artificial Analysis):** Chinese models lead the text-to-video arena. With audio, the top tier is Dreamina Seedance 2.0 (ByteDance), HappyHorse-1.0 (Alibaba), SkyReels V4 (Skywork), and Kling 3.0 (Kuaishou); Google's Veo 3.1 is the highest-ranked US model in that slice ([source](https://artificialanalysis.ai/video/leaderboard/text-to-video)).
- **Native audio is now standard.** Sora 2, Veo 3/3.1, Kling 2.6+, Runway Gen-4.5 (added May 2026), and Seedance 2.0 all generate synchronized dialogue, sound effects, and ambient audio in one pass ([source](https://developers.googleblog.com/introducing-veo-3-1-and-new-creative-capabilities-in-the-gemini-api/)) ([source](https://www.prnewswire.com/news-releases/kling-ai-launches-video-2-6-model-with-simultaneous-audio-visual-generation-capability-redefining-ai-video-creation-workflow-302634067.html)) ([source](https://aivideopicks.com/posts/runway-review-2026.html)).
- **Native 4K arrived.** Kling 3.0 (Feb 5, 2026) generates true 3840x2160 video, up to 15s, an early major consumer model to do so natively rather than via upscaling. (CineD reports 30fps; some accounts claim 60fps, unconfirmed.) ([source](https://www.cined.com/kling-3-0-ai-video-model-introduced-native-4k-enhanced-photorealism-multi-shot-sequencing-and-integrated-audio/)).
- **Clip lengths remain short.** Most flagship single generations cap at roughly 8 to 20 seconds; longer videos are built via scene-extension and multi-shot "director" features rather than one long generation ([source](https://en.wikipedia.org/wiki/Veo_(text-to-video_model))) ([source](https://developers.openai.com/api/docs/guides/video-generation)).
- **Pricing collapsed and bundled.** Google cut Veo 3 from $0.75 to $0.40/sec; aggregators now bundle multiple models into single subscriptions, with Adobe Firefly aggregating 30+ third-party models ([source](https://developers.googleblog.com/en/veo-3-and-veo-3-fast-new-pricing-new-configurations-and-better-resolution/)) ([source](https://blog.adobe.com/en/publish/2026/03/19/adobe-firefly-expands-video-image-creation-with-new-ai-capabilities-custom-models)).
- **Regulation tightening.** The federal TAKE IT DOWN Act (2025) mandates platform removal of non-consensual sexual deepfakes; states are moving to hold AI platforms (not just users) liable ([source](https://www.multistate.us/insider/2026/2/12/how-ai-generated-content-laws-are-changing-across-the-country)). C2PA provenance metadata and watermarking are the leading authentication tools ([source](https://magiclight.ai/news/c2pa-and-global-watermarking-mandates-for-ai-video-in-2026/)).

## Table of Contents

- [TL;DR](#tldr)
- [Current Status](#current-status)
- [Findings](#findings)
  - [1. The Platform Landscape](#1-the-platform-landscape)
  - [2. Capabilities Comparison](#2-capabilities-comparison)
  - [3. Access & Pricing](#3-access--pricing)
  - [4. Recent Releases (2026 Timeline)](#4-recent-releases-2026-timeline)
  - [5. Competitive Positioning](#5-competitive-positioning)
  - [6. Use Cases & Adoption](#6-use-cases--adoption)
  - [7. Concerns: Deepfakes, Copyright, Regulation](#7-concerns-deepfakes-copyright-regulation)
  - [8. Where It's Heading](#8-where-its-heading)
- [Confidence Assessment](#confidence-assessment)
  - [High Confidence](#high-confidence)
  - [Medium Confidence](#medium-confidence)
  - [Low Confidence (reported but not asserted as fact)](#low-confidence-reported-but-not-asserted-as-fact)
- [Open Questions](#open-questions)
- [Sources](#sources)
  - [Vendor / Official](#vendor--official)
  - [Technical Analyses / Reference](#technical-analyses--reference)
  - [News Coverage](#news-coverage)
  - [Industry / Market / Legal](#industry--market--legal)
- [Update History](#update-history)
- [How This Report Was Generated](#how-this-report-was-generated)

## Findings

### 1. The Platform Landscape

The field splits into US frontier labs, Chinese frontier labs, creative-suite incumbents (Adobe), and aggregators that resell multiple models. Below is a per-platform snapshot with the latest verified versions as of June 2026.

| Platform / Model | Vendor | Latest version (date) | Notes |
|---|---|---|---|
| Sora 2 / Sora 2 Pro | OpenAI | Sora 2 (Sept 30, 2025) | Social-app + API. Cameo/"characters" feature, native audio, visible watermark ([source](https://en.wikipedia.org/wiki/Sora_(text-to-video_model))). |
| Veo | Google DeepMind | Veo 3.1 (Oct 15, 2025) | Native audio, scene extension, up to 3 reference images. Accessed via Gemini app, Flow, Vertex/Gemini API ([source](https://developers.googleblog.com/introducing-veo-3-1-and-new-creative-capabilities-in-the-gemini-api/)) ([source](https://en.wikipedia.org/wiki/Veo_(text-to-video_model))). |
| Gen | Runway | Gen-4.5 (Feb 2026) | Native audio added May 2026, character consistency, extend feature. Gen-4.5 (25 cr/sec) and Gen-3 Turbo (5 cr/sec) ([source](https://aivideopicks.com/posts/runway-review-2026.html)). |
| Kling | Kuaishou | Kling 3.0 (Feb 5, 2026) | Native 4K, AI Director (6 shots in one 15s clip), multilingual audio. CineD reports 30fps (60fps unconfirmed) ([source](https://www.cined.com/kling-3-0-ai-video-model-introduced-native-4k-enhanced-photorealism-multi-shot-sequencing-and-integrated-audio/)). |
| Seedance / Dreamina | ByteDance | Seedance 2.0 (Feb 12, 2026) | Tops blind-vote leaderboards mid-2026; strong human motion ([source](https://artificialanalysis.ai/video/leaderboard/text-to-video)) ([source](https://www.cnbc.com/2026/02/14/new-china-ai-models-alibaba-bytedance-seedance-kuaishou-kling.html)). |
| HappyHorse | Alibaba (ATH) | HappyHorse-1.0 (April 2026) | High-ranking on Artificial Analysis arena ([source](https://artificialanalysis.ai/video/leaderboard/text-to-video)). |
| Dream Machine / Ray3 | Luma Labs | Ray3 (Sept 18, 2025); Ray3 Modify (Dec 18, 2025); Ray3.14 (Jan 26, 2026) | Native 1080p; Ray3.14 is 4x faster / 3x cheaper ([source](https://lumalabs.ai/press/luma-ai-announces-ray3-modify)). |
| Pika | Pika Labs | Pika 2.2 (Feb 2025) | PikaFrames keyframe transitions, up to 10s at 1080p. Older flagship vs 2026 leaders ([source](https://www.aibase.com/news/15808)). |
| Firefly Video | Adobe | Expanded March 19, 2026 | Aggregator: 30+ models (Veo 3.1, Runway Gen-4.5, Kling 2.5 Turbo, Nano Banana 2, Adobe's own), custom models ([source](https://blog.adobe.com/en/publish/2026/03/19/adobe-firefly-expands-video-image-creation-with-new-ai-capabilities-custom-models)). |
| HunyuanVideo | Tencent | HunyuanVideo 1.5 (Nov 2025) | Open-source, 8.3B params ([source](https://wavespeed.ai/blog/posts/ai-video-generation-models-2026/)). |
| Wan | Alibaba | Wan 2.5 / 2.6 | Open-source, widely used in 2026 ([source](https://wavespeed.ai/blog/posts/ai-video-generation-models-2026/)). |

Notable additional entrants surfaced in 2026 arena data but not independently verified in depth here: Skywork's SkyReels V4, xAI's grok-imagine-video, and PixVerse V6 ([source](https://artificialanalysis.ai/video/leaderboard/text-to-video)).

### 2. Capabilities Comparison

Capabilities are version-specific and dated to mid-2026. Where a vendor offers multiple tiers, the flagship figure is shown.

| Model | Max clip (single gen) | Resolution | Native audio | Multi-shot / extend | Character consistency |
|---|---|---|---|---|---|
| Sora 2 / Pro | 16-20s (per OpenAI API docs) | up to 1920x1080 (Pro for 1080p) | Yes (dialogue + SFX) | Remix existing videos | "Characters"/cameo feature |
| Veo 3.1 | ~8s per clip | 720p / 1080p | Yes (dialogue, SFX, ambient) | Scene extension to >1 min | Up to 3 reference images |
| Runway Gen-4.5 | 10s, extendable | 720p to 4K (upscale) | Yes (added May 2026) | Multi-shot sequencing, extend | Image-to-video + custom training |
| Kling 3.0 | 15s | Native 4K (3840x2160) | Yes (multilingual) | AI Director: 6 shots / 15s clip | Elements (up to 4 reference images, from 2.6) |
| Seedance 2.0 | not verified here | up to 720p+ tiers | Yes (top audio-arena model) | not verified here | Strong human/dance motion |
| Luma Ray3.14 | not verified here | Native 1080p | not verified here | Modify/performance editing | not verified here |
| Pika 2.2 | 10s | 1080p | Not verified | PikaFrames transitions | Pikascene |

Sources: ([source](https://developers.openai.com/api/docs/guides/video-generation)) ([source](https://en.wikipedia.org/wiki/Sora_(text-to-video_model))) ([source](https://en.wikipedia.org/wiki/Veo_(text-to-video_model))) ([source](https://aivideopicks.com/posts/runway-review-2026.html)) ([source](https://www.cined.com/kling-3-0-ai-video-model-introduced-native-4k-enhanced-photorealism-multi-shot-sequencing-and-integrated-audio/)) ([source](https://lumalabs.ai/press/luma-ai-announces-ray3-modify)) ([source](https://www.aibase.com/news/15808))

Caveat: clip-length and resolution figures move with every version bump. Treat the table as a June-2026 snapshot, not durable specs.

### 3. Access & Pricing

The dominant 2026 pattern is the multi-model aggregator subscription plus per-second API pricing for developers.

**Google Veo (API, verified):**
- Veo 3: $0.40/sec (cut from $0.75/sec) ([source](https://developers.googleblog.com/en/veo-3-and-veo-3-fast-new-pricing-new-configurations-and-better-resolution/)).
- Veo 3 Fast: $0.15/sec (cut from $0.40/sec) ([source](https://developers.googleblog.com/en/veo-3-and-veo-3-fast-new-pricing-new-configurations-and-better-resolution/)).
- Both support 720p and 1080p and 9:16 vertical. Veo 3.1 is "the same price as Veo 3" ([source](https://developers.googleblog.com/introducing-veo-3-1-and-new-creative-capabilities-in-the-gemini-api/)).
- Consumer access via Gemini app subscription tiers + AI credits, and via Flow for longer edited projects ([source](https://en.wikipedia.org/wiki/Veo_(text-to-video_model))).

**OpenAI Sora 2 (verified models, pricing partial):**
- Two API models: Sora 2 (speed/social) and Sora 2 Pro (production, 1080p). Durations of 16 and 20 seconds; resolutions include 1280x720, 1080x1920, 1920x1080 ([source](https://developers.openai.com/api/docs/guides/video-generation)).
- Per-second API pricing is not stated on the official docs page reviewed; Sora 2 Pro is noted as slower and more expensive than Sora 2. Specific per-second figures circulating on third-party sites are unverified ([source](https://developers.openai.com/api/docs/guides/video-generation)).

**Runway (verified, third-party review):**
- Free: $0 (125 one-time credits, 720p). Standard: $12/mo (625 credits). Pro: $28/mo (2,250 credits). Unlimited: $76/mo ([source](https://aivideopicks.com/posts/runway-review-2026.html)).
- Gen-4.5 costs 25 credits/sec; Gen-3 Turbo 5 credits/sec ([source](https://aivideopicks.com/posts/runway-review-2026.html)).

**Adobe Firefly:** Aggregates 30+ third-party models inside Creative Cloud, with unlimited generations offered on certain tiers to remove "credit anxiety" ([source](https://blog.adobe.com/en/publish/2026/03/19/adobe-firefly-expands-video-image-creation-with-new-ai-capabilities-custom-models)).

### 4. Recent Releases (2026 Timeline)

Reverse chronological, 2026 events first:

- **April 2026:** Adobe transitions Firefly toward an agentic "Creative Agent" / Project Moonlight model orchestrating production workflows; Alibaba's HappyHorse-1.0 reaches the top of arena rankings ([source](https://blog.adobe.com/en/publish/2026/03/19/adobe-firefly-expands-video-image-creation-with-new-ai-capabilities-custom-models)) ([source](https://artificialanalysis.ai/video/leaderboard/text-to-video)).
- **March 19, 2026:** Adobe Firefly expands video/image creation, adds 30+ third-party models (Veo 3.1, Runway Gen-4.5, Kling 2.5 Turbo, Google Nano Banana 2) and custom models ([source](https://blog.adobe.com/en/publish/2026/03/19/adobe-firefly-expands-video-image-creation-with-new-ai-capabilities-custom-models)).
- **Feb 12, 2026:** ByteDance ships Seedance 2.0 ([source](https://www.cnbc.com/2026/02/14/new-china-ai-models-alibaba-bytedance-seedance-kuaishou-kling.html)).
- **Feb 2026:** Runway releases Gen-4.5 (current flagship), with image-to-video earlier in the year and native audio following in May 2026 ([source](https://aivideopicks.com/posts/runway-review-2026.html)).
- **Feb 5, 2026:** Kuaishou launches Kling 3.0 (Video 3.0, 3.0 Omni, Image 3.0): native 4K, up to 15s, AI Director multi-shot (CineD reports 30fps; 60fps unconfirmed) ([source](https://www.cined.com/kling-3-0-ai-video-model-introduced-native-4k-enhanced-photorealism-multi-shot-sequencing-and-integrated-audio/)).
- **Jan 26, 2026:** Luma releases Ray3.14 (native 1080p, 4x faster, 3x cheaper) ([source](https://lumalabs.ai/press/luma-ai-announces-ray3-modify)).

Key late-2025 context (immediately preceding the 2026 wave):

- **Dec 18, 2025:** Luma announces Ray3 Modify for hybrid-AI performance/editing workflows ([source](https://lumalabs.ai/press/luma-ai-announces-ray3-modify)).
- **Dec 3, 2025:** Kling Video 2.6 introduces simultaneous audio-visual generation ([source](https://www.prnewswire.com/news-releases/kling-ai-launches-video-2-6-model-with-simultaneous-audio-visual-generation-capability-redefining-ai-video-creation-workflow-302634067.html)).
- **Nov 2025:** Tencent releases open-source HunyuanVideo 1.5 ([source](https://wavespeed.ai/blog/posts/ai-video-generation-models-2026/)).
- **Oct 15, 2025:** Google releases Veo 3.1 ([source](https://developers.googleblog.com/introducing-veo-3-1-and-new-creative-capabilities-in-the-gemini-api/)).
- **Sept 30, 2025:** OpenAI launches Sora 2 with an iOS app ([source](https://en.wikipedia.org/wiki/Sora_(text-to-video_model))).
- **Sept 18, 2025:** Luma releases Ray3 ([source](https://lumalabs.ai/press/luma-ai-announces-ray3-modify)).
- **May 2025:** Google releases Veo 3 (first with native audio) ([source](https://en.wikipedia.org/wiki/Veo_(text-to-video_model))).

### 5. Competitive Positioning

**Quality (blind human vote):** Chinese models lead the Artificial Analysis text-to-video arena in mid-2026. In the with-audio slice the top tier is Dreamina Seedance 2.0 (ByteDance, ~1217 Elo), HappyHorse-1.0 (Alibaba, ~1123), SkyReels V4 (Skywork, ~1105), and Kling 3.0 (Kuaishou, ~1104), with Veo 3.1 the highest-ranked US model at ~1094 ([source](https://artificialanalysis.ai/video/leaderboard/text-to-video)).

**US vs China:** A CNBC report on the early-2026 Chinese model wave (Alibaba, ByteDance Seedance, Kuaishou Kling) frames it as evidence that Chinese labs are competitive at or near the frontier. The page itself returned a 403 and could not be fetched, so this positioning is reported at lower confidence from the search summary ([source](https://www.cnbc.com/2026/02/14/new-china-ai-models-alibaba-bytedance-seedance-kuaishou-kling.html)).

**Strategic differentiation:**
- **OpenAI Sora 2:** distribution and social. The Sora app, cameo feature, and a reported $1B Disney character-licensing deal differentiate on consumer reach and licensed IP rather than raw arena score ([source](https://en.wikipedia.org/wiki/Sora_(text-to-video_model))).
- **Google Veo:** ecosystem integration (Gemini app, Flow editor, Vertex AI) and aggressive API price cuts ([source](https://developers.googleblog.com/en/veo-3-and-veo-3-fast-new-pricing-new-configurations-and-better-resolution/)).
- **Runway / Adobe:** the aggregator play. Rather than win on a single model, Adobe bundles 30+ competitors' models into one subscription and competes on workflow, editing, and pipeline integration; Runway competes on its own Gen models plus editing tooling ([source](https://aivideopicks.com/posts/runway-review-2026.html)) ([source](https://blog.adobe.com/en/publish/2026/03/19/adobe-firefly-expands-video-image-creation-with-new-ai-capabilities-custom-models)).
- **Kling / Kuaishou:** technical headline features, an early native-4K consumer model and the AI Director multi-shot system ([source](https://www.cined.com/kling-3-0-ai-video-model-introduced-native-4k-enhanced-photorealism-multi-shot-sequencing-and-integrated-audio/)).
- **Open-source (Tencent Hunyuan, Alibaba Wan):** the two open-weight families in serious 2026 use, enabling self-hosting and fine-tuning ([source](https://wavespeed.ai/blog/posts/ai-video-generation-models-2026/)).

### 6. Use Cases & Adoption

Adoption clusters around four areas, supported by product positioning in the sources:

- **Marketing / advertising / social content:** vertical 9:16 output, short clips, and rapid iteration target social and ad workflows directly (Veo's vertical support; Sora 2's social app) ([source](https://developers.googleblog.com/en/veo-3-and-veo-3-fast-new-pricing-new-configurations-and-better-resolution/)) ([source](https://en.wikipedia.org/wiki/Sora_(text-to-video_model))).
- **Film / VFX / professional editing:** Adobe's Firefly expansion and "Quick Cut" first-draft assembly, plus Luma's Ray3 Modify hybrid-acting workflows, push into professional pipelines ([source](https://blog.adobe.com/en/publish/2026/03/19/adobe-firefly-expands-video-image-creation-with-new-ai-capabilities-custom-models)) ([source](https://lumalabs.ai/press/luma-ai-announces-ray3-modify)).
- **Narrative / multi-shot storytelling:** Kling's AI Director (6 shots in one clip) and Runway's multi-shot sequencing explicitly target short narrative content ([source](https://www.cined.com/kling-3-0-ai-video-model-introduced-native-4k-enhanced-photorealism-multi-shot-sequencing-and-integrated-audio/)) ([source](https://aivideopicks.com/posts/runway-review-2026.html)).
- **Self-hosted / enterprise custom:** open-weight Hunyuan and Wan, plus Adobe custom models trained on a brand's own assets ([source](https://wavespeed.ai/blog/posts/ai-video-generation-models-2026/)) ([source](https://blog.adobe.com/en/publish/2026/03/19/adobe-firefly-expands-video-image-creation-with-new-ai-capabilities-custom-models)).

Quantified adoption metrics (revenue, MAU, share) were not found in verified sources and are intentionally omitted.

### 7. Concerns: Deepfakes, Copyright, Regulation

**Deepfakes and regulation:**
- The federal **TAKE IT DOWN Act** (passed 2025) requires platforms to remove non-consensual sexual deepfakes ([source](https://www.multistate.us/insider/2026/2/12/how-ai-generated-content-laws-are-changing-across-the-country)).
- Every US state has introduced legislation targeting non-consensual sexual deepfakes and AI-CSAM; states also require disclaimers on manipulated political/campaign content. Some laws (e.g., California's political-deepfake statute) have been struck down on First Amendment grounds ([source](https://www.multistate.us/insider/2026/2/12/how-ai-generated-content-laws-are-changing-across-the-country)).
- 2026 trend: lawmakers expanding liability beyond individual creators to the **platforms, payment processors, hosting, and cloud providers** that enable production and distribution ([source](https://www.multistate.us/insider/2026/2/12/how-ai-generated-content-laws-are-changing-across-the-country)).

**Provenance and watermarking:**
- **C2PA** embeds provenance metadata into generated content as a leading authentication approach in 2026, alongside model-level watermarking ([source](https://magiclight.ai/news/c2pa-and-global-watermarking-mandates-for-ai-video-in-2026/)). (Specific C2PA membership and standardization status, and SynthID volume/robustness figures, were not verified against primary sources for this report; treated as open questions.)
- Sora 2 outputs carry a visible moving watermark, though third-party removal tools appeared quickly after launch, illustrating the cat-and-mouse problem ([source](https://en.wikipedia.org/wiki/Sora_(text-to-video_model))).

**Copyright / training data:**
- Sora 2's early generations reproduced copyrighted characters; OpenAI subsequently signed a reported $1B Disney deal to license 200+ characters, a notable shift toward paid IP licensing rather than unlicensed training ([source](https://en.wikipedia.org/wiki/Sora_(text-to-video_model))).
- Broader training-data litigation against video model makers was referenced in search results but not independently verified in primary sources for this report; treated as an open question.

### 8. Where It's Heading

Directional signals, framed as inference from the verified 2026 releases (Medium confidence unless noted):

- **Native 4K becomes the new baseline** following Kling 3.0; expect competitors to match native-4K (and likely higher frame rates) through 2026 ([source](https://www.cined.com/kling-3-0-ai-video-model-introduced-native-4k-enhanced-photorealism-multi-shot-sequencing-and-integrated-audio/)).
- **Multi-shot "director" generation** (one prompt producing a sequenced scene with continuity) replaces single-clip generation as the headline feature ([source](https://www.cined.com/kling-3-0-ai-video-model-introduced-native-4k-enhanced-photorealism-multi-shot-sequencing-and-integrated-audio/)) ([source](https://aivideopicks.com/posts/runway-review-2026.html)).
- **Aggregation and agentic workflows:** the value shifts from owning one model to orchestrating many (Runway's bundle, Adobe's 30+ models and Project Moonlight agent) ([source](https://blog.adobe.com/en/publish/2026/03/19/adobe-firefly-expands-video-image-creation-with-new-ai-capabilities-custom-models)).
- **Continued price compression** as Veo cuts and cheaper "fast/light" tiers (Veo 3 Fast, Ray3.14) proliferate ([source](https://developers.googleblog.com/en/veo-3-and-veo-3-fast-new-pricing-new-configurations-and-better-resolution/)).
- **Licensing over scraping:** the Disney-OpenAI deal hints at a paid-IP model for character generation as litigation and provenance pressure mount ([source](https://en.wikipedia.org/wiki/Sora_(text-to-video_model))).
- **Mandatory provenance/watermarking** likely expands via C2PA and state/federal action ([source](https://magiclight.ai/news/c2pa-and-global-watermarking-mandates-for-ai-video-in-2026/)) ([source](https://www.multistate.us/insider/2026/2/12/how-ai-generated-content-laws-are-changing-across-the-country)).

## Confidence Assessment

### High Confidence
- Release dates and core capabilities of Veo 3 / 3.1, Sora 2, Kling 2.6 / 3.0, Runway Gen-4.5, and Luma Ray3 line (verified against official vendor pages, OpenAI/Google developer docs, Kuaishou IR/press, Luma press, Wikipedia, and Adobe's own blog).
- Veo API pricing ($0.40/sec Veo 3, $0.15/sec Veo 3 Fast) and Sora 2 / Sora 2 Pro model split and durations (verified on official developer docs).
- TAKE IT DOWN Act and the state-law direction (verified on MultiState legal analysis).

### Medium Confidence
- Mid-2026 arena leadership by Chinese models (verified directly on the Artificial Analysis leaderboard, a single source; the corroborating CNBC page returned 403 and could not be fetched).
- Runway pricing tiers (from a single third-party 2026 review, not Runway's own page).
- Kling 3.0 native-4K and AI Director multi-shot (single secondary source, CineD).
- Seedance 2.0 / HappyHorse-1.0 release dates from aggregated search summaries; arena presence is confirmed but exact ship dates rest on secondary sources.
- Adobe's agentic "Creative Agent" / Project Moonlight direction (vendor blog plus secondary reviews).
- C2PA as a leading provenance approach in 2026 (single secondary source; membership/standardization specifics unverified).

### Low Confidence (reported but not asserted as fact)
- Kling 3.0 60fps (CineD reports 30fps; 60fps appears only as unconfirmed secondhand claims).
- A Runway "six-model bundle," Adobe aggregating "Kling 3.0" (its source says Kling 2.5 Turbo), deep Adobe Premiere/After Effects integration, Ray3.14 dropping audio/character reference, a "Wan 2.7" version, a "Seedance 2.0 Mini," and HunyuanVideo "running on a single RTX 4090": each appeared in earlier drafting but was NOT supported by the cited source on re-verification, and has been removed or downgraded.
- Specific C2PA membership/ISO-standardization status and Google SynthID volume (10B+) / compression-robustness figures (single secondary source did not substantiate them).
- Claims that consumer ChatGPT tiers "lost Sora access on April 26, 2026" and that the "Sora 2 API sunsets Sept 24, 2026" come from low-quality affiliate/SEO pages and could not be corroborated on any official OpenAI source. Not treated as fact in this report.
- Specific Sora 2 per-second API pricing figures circulating on third-party sites (official docs reviewed did not state them).
- CNBC US-vs-China framing (page returned 403; relied on search summary only).

## Open Questions

- What is OpenAI's official, current per-second Sora 2 / Sora 2 Pro API pricing? (Official docs page reviewed did not list it.)
- Is there active, named training-data litigation against video model makers in 2026, and what is its status?
- Exact technical specs (clip length, max resolution) for Seedance 2.0 and HappyHorse-1.0 from primary vendor documentation.
- Real adoption/revenue/market-share numbers, none were found in verified sources.
- Status of Meta Movie Gen and Stability AI video efforts in 2026 (not surfaced as shipped products in this search; absence is itself a finding worth a dedicated follow-up).
- Did Veo advance past 3.1 (e.g., a Veo 4) by mid-2026? Not found; 3.1 appears current as of the search date.
- Confirmed C2PA membership and ISO-standardization status, and verified Google SynthID adoption/robustness figures, from primary sources.
- Whether Kling 3.0 outputs native 60fps or 30fps (sources conflict).
- Whether a Runway multi-model bundle, a "Wan 2.7" release, a "Seedance 2.0 Mini," and deep Adobe Premiere/After Effects Firefly integration actually exist (each was unverified on re-check).

## Sources

### Vendor / Official
- [OpenAI Sora 2 video generation API docs](https://developers.openai.com/api/docs/guides/video-generation)
- [Google Developers Blog, Veo 3.1](https://developers.googleblog.com/introducing-veo-3-1-and-new-creative-capabilities-in-the-gemini-api/)
- [Google Developers Blog, Veo 3 / Veo 3 Fast pricing](https://developers.googleblog.com/en/veo-3-and-veo-3-fast-new-pricing-new-configurations-and-better-resolution/)
- [Kuaishou / Kling Video 2.6 (PRNewswire release)](https://www.prnewswire.com/news-releases/kling-ai-launches-video-2-6-model-with-simultaneous-audio-visual-generation-capability-redefining-ai-video-creation-workflow-302634067.html)
- [Luma AI press, Ray3 Modify](https://lumalabs.ai/press/luma-ai-announces-ray3-modify)
- [Adobe Blog, Firefly expansion (March 19, 2026)](https://blog.adobe.com/en/publish/2026/03/19/adobe-firefly-expands-video-image-creation-with-new-ai-capabilities-custom-models)

### Technical Analyses / Reference
- [Wikipedia, Sora (text-to-video model)](https://en.wikipedia.org/wiki/Sora_(text-to-video_model))
- [Wikipedia, Veo (text-to-video model)](https://en.wikipedia.org/wiki/Veo_(text-to-video_model))
- [Artificial Analysis, Text-to-Video Leaderboard](https://artificialanalysis.ai/video/leaderboard/text-to-video)
- [CineD, Kling 3.0 introduced](https://www.cined.com/kling-3-0-ai-video-model-introduced-native-4k-enhanced-photorealism-multi-shot-sequencing-and-integrated-audio/)
- [WaveSpeed Blog, AI video generation models 2026](https://wavespeed.ai/blog/posts/ai-video-generation-models-2026/)
- [AIVideoPicks, Runway review 2026](https://aivideopicks.com/posts/runway-review-2026.html)
- [AIBase, Pika 2.2 release](https://www.aibase.com/news/15808)

### News Coverage
- [CNBC, new China AI models (Alibaba, ByteDance Seedance, Kuaishou)](https://www.cnbc.com/2026/02/14/new-china-ai-models-alibaba-bytedance-seedance-kuaishou-kling.html)

### Industry / Market / Legal
- [MultiState, state deepfake laws 2026](https://www.multistate.us/insider/2026/2/12/how-ai-generated-content-laws-are-changing-across-the-country)
- [MagicLight, C2PA and watermarking mandates 2026](https://magiclight.ai/news/c2pa-and-global-watermarking-mandates-for-ai-video-in-2026/)

## Update History

- 2026-06-16T15:30:00-04:00: Post-verification pass. An independent verifier re-fetched every cited URL and flagged claims unsupported by their sources. Removed or downgraded: Runway six-model bundle, Adobe aggregating "Kling 3.0" (corrected to 2.5 Turbo), Adobe Premiere/AE integration, Ray3.14 "drops audio + character reference," "Wan 2.7" (corrected to 2.5/2.6), "Seedance 2.0 Mini," HunyuanVideo "single RTX 4090," and the C2PA-backers/ISO + SynthID "10B pieces/compression-surviving" claims. Kling 3.0 "60fps" downgraded to unconfirmed (CineD reports 30fps). Arena leadership and C2PA moved from High to Medium confidence (single-source).
- 2026-06-16T14:41:55-04:00: Initial draft (Fresh Mode). Built from 12 WebFetch-verified sources plus WebSearch discovery. Covers the platform landscape, capabilities, pricing, 2026 timeline, competitive positioning, use cases, and concerns.

## How This Report Was Generated

- Date: 2026-06-16 (Eastern Time confirmed via `TZ='America/New_York' date`).
- Mode: Fresh (no prior AI video report existed in research-reports/).
- Search tiers: Tier 1 (SearXNG MCP) was unavailable this session. Tier 2 (Cornell Gateway Gemini Search) failed: no gateway token present in this environment. Fell back to Tier 3, built-in WebSearch for discovery + WebFetch for source verification.
- Verification: ~12 URLs fetched and read via WebFetch before citing (OpenAI docs, Google dev blog x2, Kuaishou/Kling press, Luma press, Adobe blog, Wikipedia x2, Artificial Analysis leaderboard, CineD-via-search, MultiState). Two pages (OpenAI Sora 2 marketing page, CNBC) returned HTTP 403 and were handled at reduced confidence via search summaries; the Kuaishou IR page timed out and was substituted with the PRNewswire mirror.
- Anti-hallucination: low-quality affiliate claims (Sora access/API sunset dates, unverified Sora per-second pricing) were explicitly downgraded to Low confidence and not asserted as fact.
- Tools: WebSearch, WebFetch, Bash (timestamp).
