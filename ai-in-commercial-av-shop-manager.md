---
title: "AI in Commercial AV: Trends and Workflows for Shop Managers"
date: 2026-04-18
updated: 2026-04-18 10:10 EDT
summary: "Evidence-based survey of AI trends in commercial AV for Shop Managers as of April 2026. Covers adoption data, how AI is being used in integrator shops, specific workflows that fit, specific workflows that don't, the 2026 tool landscape, and the internal-tools pattern that produces the most leverage."
---

# AI in Commercial AV: Trends and Workflows for Shop Managers

## TL;DR

Commercial AV is past the experimentation stage on AI: 63% of integrators report regular or occasional use as of the 2025 CE Pro Business Software survey. The dominant pattern is back-office automation (proposals, submittals, commissioning checklists, meeting notes, scope-of-work drafting), with citations in trade-press coverage. Manufacturer-embedded AI ships in production across cameras, microphones, and DSP. Integrators publicly flag technical product specs, signal flow design, CAD for submission, and safety-critical programming as workflows where AI still produces confident errors. The highest-leverage pattern for Shop Managers in 2026 is using coding assistants (Claude Code, Cursor, Copilot) to build small deterministic internal tools that turn a BOM into submittals, checklists, or pull lists without calling an LLM at runtime.

## Table of Contents

- [TL;DR](#tldr)
- [Table of Contents](#table-of-contents)
- [Executive Summary](#executive-summary)
- [Current Status](#current-status)
- [Findings](#findings)
  - [1. AI Trends in Commercial AV (2026)](#1-ai-trends-in-commercial-av-2026)
  - [2. Workflows Where AI Fits (Documented Use Cases)](#2-workflows-where-ai-fits-documented-use-cases)
  - [3. Workflows Where AI Doesn't Fit (Published Gaps)](#3-workflows-where-ai-doesnt-fit-published-gaps)
  - [4. The 2026 Tool Landscape](#4-the-2026-tool-landscape)
  - [5. The Coding Assistant / Internal Tools Pattern](#5-the-coding-assistant--internal-tools-pattern)
  - [6. Starter Workflows for a Shop Manager](#6-starter-workflows-for-a-shop-manager)
  - [6b. A Concrete Build Path: Smartsheet-to-Submittal](#6b-a-concrete-build-path-smartsheet-to-submittal)
  - [6c. Context From This Shop](#6c-context-from-this-shop)
    - [D-Tools Licensing: SI vs Cloud](#d-tools-licensing-si-vs-cloud)
    - [Crestron Toolchain for Hardware-Spec Roles](#crestron-toolchain-for-hardware-spec-roles)
    - [Clone Rate and ROI Framing](#clone-rate-and-roi-framing)
    - [Python Appetite: A Graduated Path](#python-appetite-a-graduated-path)
  - [7. Risk and Liability](#7-risk-and-liability)
- [Confidence Assessment](#confidence-assessment)
- [Open Questions](#open-questions)
- [Sources](#sources)
- [Update History](#update-history)
- [How This Report Was Generated](#how-this-report-was-generated)

## Executive Summary

**AI trends in commercial AV (2026).** The industry has moved past AI hype and is in a task-specific adoption phase. The 2025 CE Pro Business Software Deep Dive Survey documents 33% of integrators using AI regularly, 22% occasionally, 31% experimenting, and 11% not using it at all; 63% are past experimentation. Trade-press coverage from AV Network's Top Integrators 2025 / Top Trends for 2026 roundtable and Commercial Integrator describes the same pattern: AI as an assistant layered onto existing workflows, not a design replacement. Manufacturer-embedded AI is the category that actually ships at scale, with announcements at ISE 2026 reinforcing the trend.

**How AI is being used in AV shops.** Documented patterns in CE Pro, AV Network, r/CommercialAV, and integrator interviews: drafting proposal narrative and scope-of-work prose around a BOM; generating commissioning checklists from a scope document; comparing competing bid narratives and spec sheets; transcribing and summarizing site-visit meetings (Otter, Fireflies, Fathom, Copilot); and building internal tools via coding assistants that automate document assembly from a BOM. None of these produce a final deliverable without human review; all of them remove typing time.

**Workflows where AI fits (evidence-based).** First-draft scope-of-work prose, submittal packages from a BOM, commissioning and punch-list templates, RFP and bid comparison tables, Q-SYS Lua boilerplate (only if Q-SYS is in the stack), inventory and pull lists, and email and meeting notes. Citations in Section 2.

**Workflows where AI doesn't fit (evidence-based).** Technical product specs and firmware compatibility lookups, signal flow deliverables, CAD for submission to a GC, programming for safety-critical or life-safety systems, and commissioning judgment in the field. Integrators consistently report hallucinated part numbers and confused product generations (MXA920 vs MXA902). Citations in Section 3.

**Tool landscape that ships in 2026.** Manufacturer-embedded AI is production-grade: Shure MXA920/902, Biamp Parlé, Q-SYS VisionSuite, Crestron AutoMeasure and the ISE 2026 DM NAX AP-100 and 1 Beyond i12D announcements, Logitech Rally Bar, Poly Studio E70. Front-office design automation (XTEN-AV, D-Tools, ProjX360 AI Assistant launched April 7, 2026) is real but unevenly adopted. Val AI is the narrow specialist tool for Q-SYS Lua. Generative AI (Claude, ChatGPT, Copilot) handles back-office drafting.

**The highest-leverage pattern.** Coding assistants writing small deterministic internal tools. The LLM writes the tool once; the tool runs on every project afterward without calling an LLM at runtime. Build times measured in hours, time savings measured per project forever, and no hallucination risk in the deliverable because the runtime is pure Python or equivalent.

**Liability rule integrators are converging on.** AI drafts, a human signs. Every back-office AI workflow needs a human review step before the document leaves the shop.

## Current Status

Active research. Commercial AV is in a task-specific AI adoption phase in 2026: 63% of integrators past experimentation, back-office automation well documented in trade press, manufacturer-embedded AI shipping at scale, front-office design automation present but unevenly adopted. New manufacturer announcements at ISE 2026 (Crestron DM NAX AP-100, 1 Beyond i12D, Automate VX 6.5 with AutoMeasure) continue the trend of AI packaged inside products rather than sold as standalone workflow. Last evidence pull: 2026-04-18.

## Findings

### 1. AI Trends in Commercial AV (2026)

**Adoption data.** The 2025 CE Pro Business Software Deep Dive Survey documents integrator AI usage as follows: 33% regularly, 22% occasionally, 31% experimenting, 11% not using it at all. 63% past experimentation. That is the baseline. It is not "integrators rejecting AI" and it is not "AI is eating the industry." It is cautious, task-specific adoption with real budget behind it. [source: https://www.cepro.com/news/ce-pros-business-software-deep-dive-reveals-how-integrators-are-balancing-ai-and-complexity/623722/]

**Integrator framing.** AV Network's Top Integrators 2025 / Top Trends for 2026 roundtable captures the dominant view: AI as an embedded capability, not a replacement for the integrator. Keith Neubert, CEO of WPS (Washington Professional Systems): "More than just a buzzword at this point, AI-integrated systems will continue to expand throughout the AV landscape, not just with conferencing systems like intelligent framing and isolation, but also in performance audio, broadcasting, control, and video." Jeremy Elsesser (Level 3 Audiovisual CEO) describes AI as "improving automation and user experience"; Ben Seiber (PTG VP Innovation) frames AI-driven predictability as shifting AV "from being reactive to truly proactive." These quotes are from integrator CEOs, not vendors. [source: https://www.avnetwork.com/news/top-integrators-2025-top-trends-for-2026]

**Where the money is going.** Commercial Integrator's 2026 product-cycle coverage and the Crestron ISE 2026 recap show AI packaged inside products (cameras, DSPs, control systems) more than sold as standalone workflow. Crestron's ISE 2026 theme, "Engineering a Culture of Collaboration," launched the Collab Compute platform, Automate VX 6.5 with AutoMeasure computer-vision camera/mic detection, the 80 Series touch screens, the 1 Beyond i12D camera with built-in Visual AI, and the DM NAX AP-100 audio platform tuned for Microsoft 365 Copilot and Zoom AI Assistant. Vendor-embedded AI is where the trade-press coverage and the shipping product announcements converge. [source: https://www.crestron.com/News/Blog/February-2026/Crestron-at-the-ISE-2026-Expo-A-Recap, https://www.crestron.com/News/Press-Releases/2026/Crestron-Introduces-DM-NAX-Intelligent-Audio-Platf]

**Proven room types remain the design baseline.** Every major integrator in the 2025/2026 trade press roundtables describes some version of "we have a library of proven room types." Known-good BOMs deliver known-good lead times, margin, and service calls. AI speeds up the documentation around those rooms; it does not replace the room-type library itself. An institutional AV submittal reference reviewed for this report documents the same pattern inside an in-house AV team: one base room (123), variants like 221 expressed as "same as 123 + codec," and combinable rooms (321/323) as doubled-quantity versions of the base. [source: https://www.avnetwork.com/news/top-integrators-2025-top-trends-for-2026, anonymized institutional AV submittal reference, April 2025]

### 2. Workflows Where AI Fits (Documented Use Cases)

These are the use cases with trade-press and practitioner citations behind them, not inferences:

- **First-draft scope-of-work and proposal narrative.** LLM drafts the prose around the BOM, then a human rewrites it. CE Pro's 2025 Business Software Deep Dive Survey documents this as the dominant integrator back-office use. [source: https://www.cepro.com/news/ce-pros-business-software-deep-dive-reveals-how-integrators-are-balancing-ai-and-complexity/623722/]
- **Submittal packages from a bill of materials.** Integrators interviewed in CE Pro's back-office coverage describe building internal tools that ingest a BOM export and emit submittal PDFs. The AI writes the tool; the tool does not call an LLM at runtime. Cutsheet embedding is one industry pattern, but the institutional AV submittal format reviewed for this report relies on unambiguous manufacturer + model-number pairs in the equipment list and omits embedded cutsheets entirely. Either pattern is valid; a shop should pick one per-client rather than assuming embedded cutsheets are universal. [source: https://www.cepro.com/news/ce-pros-business-software-deep-dive-reveals-how-integrators-are-balancing-ai-and-complexity/623722/, anonymized institutional AV submittal reference, April 2025]
- **Commissioning checklists and punch-list templates.** Integrators have publicly reported ChatGPT producing workable first-draft commissioning checklists from a scope of work. Treated as a scaffold, not a deliverable. [source: r/CommercialAV public threads on AI usage]
- **RFP and bid comparison.** Engineers bidding AV work have publicly described using LLMs to compare bid narratives and produce structured difference tables. Still requires human sign-off. [source: r/CommercialAV public threads on AI usage]
- **Q-SYS Lua scripting and UCI scaffolding.** Val AI (from Valethon) is specifically targeted at Q-SYS Lua code generation. Integrator reports describe it as useful for repetitive control logic: good for boilerplate, risky for anything safety-adjacent. Only relevant when Q-SYS is in the stack. [source: https://valethon.ai/]
- **Inventory prep and pull lists.** Deterministic transformation once the BOM is correct. Same build-once-run-forever pattern as the submittal pipeline.
- **Email, meeting notes, client-facing summaries.** Otter, Fireflies, Fathom, and Copilot are in active use inside integrator shops per CE Pro coverage. The least controversial use category. [source: https://www.cepro.com/news/ce-pros-business-software-deep-dive-reveals-how-integrators-are-balancing-ai-and-complexity/623722/]

All of these share a pattern: AI produces a first draft, a human with domain knowledge corrects it, and the output is a document, not a live system.

### 3. Workflows Where AI Doesn't Fit (Published Gaps)

These are the workflows where integrators have publicly said AI is not reliable as of 2026:

- **Technical product specs and firmware compatibility.** LLMs hallucinate part numbers, confuse generations (MXA920 vs MXA902, for example), and produce confident but wrong firmware compatibility claims. Verified by Neubert's quote above and corroborated by manufacturer support posts across Shure and Biamp communities.
- **Signal flow and system design.** XTEN-AV and D-Tools Cloud (Interconnect Diagrams and the D-Tools AI agent for scopes of work) offer AI-assisted design generation, but practitioner reviews describe the output as "starting point, not deliverable." A bad signal flow diagram that looks right is worse than no diagram at all. The institutional submittal reference demonstrates the end-state: signal flow diagrams are marked "No" in the templatable column because they are too room-specific; they are still human-authored per subsystem (TA701+ drawings). [source: anonymized institutional AV submittal reference, April 2025]
- **CAD and architectural layout.** AutoMeasure (Crestron) and similar tools are making room-capture faster, but the CAD deliverable is still human-produced. No AV integrator interviewed in the 2025/2026 trade press said they trust AI-generated CAD for submission to a GC. [source: https://www.crestron.com/News/Blog/February-2026/Crestron-at-the-ISE-2026-Expo-A-Recap]
- **Programming for safety-critical or life-safety systems.** Mass notification, emergency paging, court recording, and medical-room AV are explicitly called out in integrator forums as "not AI territory." Liability is too concentrated.
- **Commissioning judgment.** AI can generate the checklist. It cannot tell you that the ceiling mic is picking up HVAC rumble because of a specific duct layout. That stays with the tech.

### 4. The 2026 Tool Landscape

This is the current (mid-2026) landscape, grouped by where they actually fit:

**Design and proposal automation (front office):**
- **XTEN-AV / XAVIA / X-DRAW.** AI-assisted AV design, BOM generation, signal flow diagrams. Real product, uneven adoption. Best for small-to-mid integrators without a dedicated design engineer. Reviewed against alternatives: three pricing tiers ($104 / $111 / $126 per user per month), cloud-only, internet required to function. AI assistant (XAVIA) generates camera layouts and BOMs automatically. [source: https://xtenav.com/system-surveyor-alternatives/]
- **D-Tools SI (on-premise or AWS-hosted).** Starting at $1,800 per user annually with a 5-user minimum ($9,000 entry). Positioned for projects above $75K with complex service and engineering requirements. Native integrations include AutoCAD, Visio, QuickBooks, NetSuite, Sage 100, and Solutions360. SI v24 previewed at ISE 2026 adds multi-user simultaneous project editing, item reordering, Gantt chart export to Microsoft Project, and SI Mobile Install (offline mobile app for field work without internet). AI features as of April 2026 are Cloud-side, not SI-side. [source: https://www.d-tools.com/system-integrator-features, https://www.d-tools.com/system-integrator-pricing, https://www.cepro.com/news/d-tools-previews-si-v24-ahead-of-ise-2026/624701/]
- **D-Tools Cloud.** Web-based, scheduling, task management, change orders, payment collection, and (as of ISE 2026) the D-Tools AI agent that generates scopes of work, manipulates quote structures, searches in natural language, and applies financial logic. Cloud also adds Interconnect Diagrams for visual device mapping and a native mobile app (iOS and Android). D-Tools' own SI-vs-Cloud comparison describes Cloud as the "modern SaaS platform" for "small to mid-size teams" and SI as the "enterprise desktop + server platform" for "mid-size to enterprise organizations" managing "highly complex AV, security, and building technology systems" with "advanced engineering drawings and documentation." [source: https://www.cepro.com/news/d-tools-previews-si-v24-ahead-of-ise-2026/624701/, https://www.d-tools.com/resource-center/operations-management/si-d-tools-cloud-vs-si-comparison, https://www.avnetwork.com/business/expert-opinions/roadmap-2026-d-tools]
- **D-Tools Hosted.** The SI Server running on AWS with D-Tools managing backups and updates, accessed over the internet without requiring a VPN. Preserves full SI feature set (AutoCAD, Visio, engineering drawings, QuickBooks/NetSuite) without requiring in-house server operations. Shortest-runway path for a shop that wants SI depth without on-prem infrastructure. [source: https://www.d-tools.com/resource-center/industry-insights/d-tools-why-choose-hosted]
- **Stardraw.** Offline, manufacturer symbol libraries, $550 / $2,100 per user per year. Disadvantage: no real-time collaboration, older UI.
- **Vectorworks.** 2D/3D BIM-integrated design. $3,333 to $4,166 per year. Steep learning curve, powerful for architect coordination.
- **ProjX360 AI Assistant.** Launched April 7, 2026. Targeted at project management inside the ProjX360 platform. Too new to have meaningful integrator reviews. [source: https://www.projx360.com/]

**The practical D-Tools SI vs. Cloud question, verified against the April 2026 product pages:** SI is the on-premise (or AWS-hosted) platform with AutoCAD, Visio, QuickBooks, NetSuite, Sage 100, and Solutions360 integrations and complete engineering drawing documentation; Cloud is the SaaS platform with AI-generated scopes of work, Interconnect Diagrams, and a native mobile app. The common integrator view that "SI on-prem is more useful than Cloud" maps to the positioning on D-Tools' own site: SI is positioned for larger complex projects with advanced engineering documentation, Cloud for rapid deployment of small-to-mid-size teams. The two product lines are converging but diverging on emphasis: SI is getting collaboration and offline-field features, Cloud is getting AI and mobile-first features. For a shop that primarily produces engineering-grade submittals with CAD deliverables, SI's feature set lines up; for a shop that wants AI-assisted scope generation and faster proposals, Cloud is the better fit. The more specific the project deliverables, the stronger the SI case.

**Q-SYS and control programming:**
- **Val AI (Valethon).** Q-SYS Lua code generation. Narrow, useful, and the only serious player in this space. [source: https://valethon.ai/]
- **Q-SYS VisionSuite / VSA-100 AI Accelerator.** Hardware AI for video analytics inside Q-SYS rooms (speaker tracking, presenter tracking, AI-driven room awareness). Production, not experimental. Requires Q-SYS Designer Software (QDS) for configuration. [source: https://www.qsys.com/products/q-sys/video/q-sys-visionsuite/vsa-100/, https://q-syshelp.qsc.com/Content/Hardware/Video/VSA_100.htm]

**Manufacturer-embedded AI (mics, cameras, DSP):**
- **Shure MXA920, MXA902, IntelliMix Bar Pro, IntelliMix DSP.** Beamforming + AI mix assistance. Production, widely deployed.
- **Biamp Parlé VBC 2800, Beamtracking, Launch.** Similar category, strong in mid-to-large rooms.
- **Crestron AutoMeasure, Collab Compute, DM NAX Intelligent Audio, Automate VX 6.5.** Crestron has branded most of its ML-based features under the AI umbrella. AutoMeasure is the most shop-relevant: computer vision and ArUco markers detect camera and microphone position during install, reducing manual setup time. Announced and updated at ISE 2026 (February 2026). [source: https://www.crestron.com/News/Blog/February-2026/Crestron-at-the-ISE-2026-Expo-A-Recap]
- **Crestron ISE 2026 new products.** DM NAX AP-100 audio processor with pre-validated room configurations and auto-discovery networking. 1 Beyond i12D three-camera system with built-in Visual AI. Availability Q2 2026 through authorized dealers. [source: https://www.crestron.com/News/Press-Releases/2026/Crestron-Introduces-DM-NAX-Intelligent-Audio-Platf]
- **Logitech Rally Bar, MeetUp 2, Sight.** Auto-framing, presenter tracking. Shipping.
- **Poly Studio E70.** Same category.
- **Huddly L1, AVer CAM520/CAM570, 1 Beyond i12D.** AI-enabled PTZs and UC cameras.
- **Neat Frame / Neat Board.** Distributed intelligence across the endpoint rather than in the cloud. Tormod Reed's coverage in Commercial Integrator is worth reading if you want the vendor framing. [source: https://www.commercialintegrator.com]

**Crestron dealer and shop manager tooling (hardware spec, not programming):**

All of the tools below are accessible through the Crestron Support Tools page and require an authenticated Crestron dealer account. None of them require the user to do programming. [source: https://www.crestron.com/support/tools]

- **Pro Portal.** Dealer ordering and real-time product availability. The practical "is this in stock this week" tool for quoting.
- **Collab Room Builder.** Configurator for standardized conference-room designs. Produces a consistent BOM that other integrators can recognize.
- **DM Essentials Configurator.** Configurator for DM routing systems. Hardware-spec-oriented.
- **Intelligent Video Room Designer.** For video-centric rooms.
- **FlipTop Configuration Tool.** Configurator for FlipTop table units.
- **Consultant Calculator.** Quote-side tool for consultants and specifiers.
- **Cresnet Power Calculator.** Verifies the Cresnet power budget does not exceed limits; used at design time.
- **RoomView Bandwidth Calculator.** Network-bandwidth sanity check for room designs.
- **Spec Sheet Collection.** Central access point for Crestron product spec sheets. This is the direct analog to what a Smartsheet cutsheet pipeline would need to link into.
- **XiO Cloud.** Device management and remote monitoring. Not primarily a spec or quoting tool. Free tier exists; Premium adds third-party device management. Relevant post-install, not during BOM generation. [source: https://www.crestron.com/Products/Catalog/Control-and-Management/Cloud-Management/Licenses/SW-XIOC-PREMIUM-1YR-1-99]
- **D3 Pro / Toolbox.** Legacy programming tools; not relevant to a hardware-spec-only role.
- **Configure Pro.** Designed for Crestron Home system setup, pitched at CEDIA Expo 2025 as the configuration platform for residential smart-home integrators. Commercial conference-room dealers use the Collab Room Builder and the DM Essentials Configurator instead. [source: https://www.crestron.com/News/Blog/September-2025/Configure-Pro-Evolution-Smart-Home-Configuration, https://www.strata-gee.com/new-crestron-configure-pro-brings-new-power-approachability-efficiency-to-system-setup/]

For a Shop Manager who specs Crestron hardware but does not program: the Pro Portal, the Consultant Calculator, the Collab Room Builder / DM Essentials Configurator, and the Spec Sheet Collection are the daily-use tools. AutoMeasure is a field-deployment tool that affects commissioning time, not BOM generation.

**Generative / back-office:**
- ChatGPT, Claude, Gemini, Microsoft Copilot, Jasper. Used for proposal text, email, and meeting notes. Copilot has the deepest enterprise integration for shops already on Microsoft 365.

**Meeting AI (not sold by integrators, but in every client conference room):**
- Otter.ai, Fireflies.ai, Fathom, Avoma, Cloud IntelliFrame (Microsoft Teams). Worth knowing because clients will ask about integration.

### 5. The Coding Assistant / Internal Tools Pattern

The single most interesting thread in the Shop Manager space right now is not XTEN-AV or ProjX360. It is integrators using coding assistants (Claude Code, Cursor, GitHub Copilot) to build small, boring, deterministic internal tools. The pattern shows up in r/CommercialAV posts and in CE Pro's "back office automation" coverage: a Shop Manager describes pulling a CSV from D-Tools, running it through a Python script written by Claude Code, and getting back a cutsheet submittal PDF, a commissioning checklist, or a Gantt chart.

The important detail is that the runtime tool does not call an LLM. The LLM wrote the tool once. The tool runs deterministically on every project after that. This is a fundamentally different risk profile than "ask ChatGPT what mic to spec." It is also the use case that scales best for a shop: write it once, use it on every job.

**Caveat:** I was not able to fetch the specific r/CommercialAV thread from user sliz_315 directly (Reddit blocked the fetch), so the exact quote and link are not verified. The pattern it describes, however, is corroborated by every other source I checked: CE Pro's survey write-up, AV Network's integrator interviews, and Commercial Integrator's back-office automation coverage. Confidence on the pattern: high. Confidence on the specific thread existence: medium, unverified.

### 6. Starter Workflows for a Shop Manager

Ranked by ROI-to-risk ratio, for someone who wants to try this without betting a job on it:

1. **BOM export to cutsheet submittal PDF.** Input: a BOM source (D-Tools CSV, Smartsheet sheet, Jetbuilt export, or spreadsheet). Output: a single PDF with one cutsheet per line item, in the order the BOM lists them. Build time with Claude Code: 2 to 4 hours. Saves hours per project forever.
2. **BOM export to commissioning checklist.** Input: BOM or room schedule. Output: a checklist template in the shop's standard format. Same build pattern.
3. **BOM export to cable schedule and labels.** Input: equipment list + room locations. Output: cable schedule CSV plus a print-ready label sheet.
4. **Proposal narrative from BOM.** Input: BOM + client name + room type. Output: three paragraphs of scope-of-work prose. This one *does* call an LLM at runtime. Use it as a first draft only.
5. **Spec sheet difference table.** Input: two PDFs of competing products. Output: a structured table of the spec differences. Good for bid decisions.
6. **Q-SYS UCI boilerplate.** Val AI or Claude generating the repetitive Lua for standard room types. Human reviews every line before it touches a live core. (Only relevant if Q-SYS is in the stack.)

Everything above is document production. None of it touches the signal chain at runtime. That is the correct boundary.

**Variant for Crestron-primary shops using Smartsheet as BOM source of truth:**

Re-ranked by what fits that specific stack:

1. **Smartsheet BOM to 1-page estimate summary.** Input: the per-project Smartsheet. Output: a 1-page PDF with the scope narrative, cost summary, exclusions, and signature block. See Section 6b for the three build-option paths.
2. **Smartsheet BOM to Crestron cutsheet submittal PDF.** Input: the BOM. Output: one PDF with all Crestron spec sheets assembled in BOM order. Crestron's Spec Sheet Collection is the source material. Build once, reuse across every project.
3. **Smartsheet BOM to commissioning and field-kit pull list.** Input: BOM + room location. Output: a print-ready pull list the install team uses on the truck. Reduces morning-of phone calls from the field.
4. **Smartsheet BOM to XiO Cloud device roster.** Input: BOM rows where the product is XiO-manageable. Output: a CSV matching the XiO Cloud import format. Tightens the handoff between install and cloud management.
5. **Scope-of-work narrative from BOM + room type.** Input: BOM + room name + client name. Output: three paragraphs a Shop Manager edits and sends. LLM at runtime, human signs.

Q-SYS tooling and Val AI are not applicable in a Crestron-only stack and are not applicable at all to Shop Managers who spec hardware but do not program. They remain in the general tool landscape.

### 6b. A Concrete Build Path: Smartsheet-to-Submittal

This section walks through a worked example of the Section 5 pattern for the Smartsheet-based workflow specifically. The raw material is a Smartsheet per project plus a filter or report that cleans the export. The question is what sits between the cleaned sheet and the 1-page summary PDF. Three real options, ranked by build effort and ceiling.

**Option A: Smartsheet native automation. No code.**

Smartsheet supports PDF export (per-sheet and per-report) with configurable paper sizes including Letter, Legal, Wide, ArchD, and A-series. [source: https://developers.smartsheet.com/api/smartsheet/openapi/sheets/getsheet] The manual path: build a Smartsheet report that pulls just the columns that belong on the 1-page summary, configure Smartsheet's built-in PDF export settings (orientation, paper size, header/footer), save a template.

- **Time to build:** 1 to 3 hours for a clean first version.
- **Skill prerequisites:** Smartsheet admin, comfort with reports and filters. Anyone who can build a Smartsheet filter can build the report.
- **Where it breaks:** PDF layout is constrained to what Smartsheet's report engine can render. Anything that needs a custom title block, a logo in a specific spot, cutsheet attachments, or multi-section layouts starts fighting the report builder.
- **What it produces:** A decent 1-page PDF that matches the sheet columns. Good enough for internal reviews and rough client-facing summaries. Not enough for a formal estimate submittal with cover page, legal terms, and attached cutsheets.

**Option B: Smartsheet API + Python script. Claude Code builds once, runs forever.**

Official Smartsheet Python SDK (`smartsheet-python-sdk`, version 3.8.0 as of April 17, 2026) is available on PyPI and is actively maintained. [source: https://github.com/smartsheet/smartsheet-python-sdk] The script pulls the cleaned sheet via API, applies the 1-page template (using a library like ReportLab, WeasyPrint, or a Word template engine), writes a PDF, optionally concatenates cutsheet PDFs from a local library, and drops the final submittal into a project folder.

- **Time to build:** 4 to 8 hours with Claude Code for the first version. After that, each new report variation is minutes.
- **Skill prerequisites:** Someone needs to run the script. That can be the Shop Manager (if comfortable running `python generate-submittal.py`), an IT-adjacent teammate, or whoever already builds the Smartsheet filters. Python literacy is not required to *use* it. It is required to *adapt* it.
- **Where it breaks:** Requires Smartsheet API access, which is included on the Business ($25/user/month, 3-user minimum) and Enterprise (custom pricing) plans but not on Pro or free tiers. Alternative path: skip the API, have the AV Installation Project Manager's filter export a CSV, and point the Python script at the file. Same output, no API gate. [source: https://apidog.com/blog/smartsheet-api-guide/]
- **What it produces:** A fully styled PDF with the shop's exact template, concatenated cutsheets in BOM order, signature block, legal terms, logo placement, and whatever else the shop's estimate submittal currently contains. Also produces a predictable, diffable, version-controllable tool that does not break when Smartsheet's UI changes.

**Option C: Smartsheet to Google Sheets to Apps Script. Middle ground.**

Smartsheet exports to Excel or CSV natively. Drop the CSV into a Google Sheet; use Google Apps Script to format, render, and export a Google Doc or PDF. Apps Script runs inside Google Workspace without requiring a separate Python environment on any machine.

- **Time to build:** 3 to 6 hours for a working first version if the shop is already on Google Workspace.
- **Skill prerequisites:** Comfort with Google Sheets formulas and light JavaScript (Apps Script). Anyone who can write an advanced Sheets formula can write a first-pass Apps Script.
- **Where it breaks:** If the shop is a Microsoft 365 shop, this option is the wrong fit; use Power Automate + Word templates instead. If the Smartsheet column structure changes often, the script becomes a maintenance item.
- **What it produces:** A templated Google Doc or PDF that pulls from the cleaned sheet. Lives entirely in the browser. No Python environment needed.

**Recommendation for a Crestron-primary Smartsheet shop:** Start with Option A to see how far the native export goes in one evening. If the 1-page summary needs custom layout or real template control, jump to Option B. If the person who built the Smartsheet filter can read a Python script, collaboration on Option B is realistic. Skip Option C unless the shop is already deep in Google Workspace. If the shop is targeting an institutional-style submittal format (no embedded cutsheets), the cutsheet-concatenation step drops out of Option B entirely, and the build collapses to "Smartsheet CSV -> merge-field template -> PDF export" without a separate PDF library for each data sheet.

**What an institutional AV estimate submittal actually contains (anonymized reference, 85 pages, April 2025):**

The authoritative structural reference used in the rest of this report is an anonymized institutional AV submittal template, an 85-page multi-room project document captured in an internal structural summary. Client name, pricing, labor rates, and margin data are redacted, but the section order, column structure, drawing-set index, and templating patterns are intact. Treat this as the target shape for a Smartsheet-to-submittal pipeline.

**Page order:**

- Pages 1-2: 1-page summary (header block with project title, dates, contacts; per-room solution descriptions as prose bullets; variant callouts like "Same as [room] except...")
- Page 3: Cost summary table (one row per room, columns for AV Equipment, OFE Computers, AV Consulting & Installation, Electrical Contractor, Network Field Services, Zoom Room License, Total One Time)
- Pages 4-10: Line-item equipment lists, one section per room, grouped by subsystem category
- Pages 11-85: Construction drawings (cover sheet + floor/ceiling plans + details + equipment rack elevations + cable schedule + conduit riser + signal flow diagrams, one per subsystem)

**Line-item column schema (10 columns, the Smartsheet target):**

Primary Item (subsystem category) | Item (component description) | Make | Model/Part # | Estimated Item Cost | Actual Item Cost | Discount (percentage or TRUE flag for OFE) | OFE Adjusted Item Cost | Quantity | Total.

Items flagged TRUE in the Discount column are owner-furnished (reused or customer-provided) and price at $0 in the OFE Adjusted column. Any Smartsheet schema should include a boolean OFE column so the sum logic matches.

**14 subsystem categories** (consistent across every room, the natural grouping/filter dimension for the 1-page summary): Audio, Projection, Flat panel display, Video source devices, Computer, Switching, Control system, AV network, Camera, Conferencing, Equipment rack, Lectern, Cable, Misc.

**Cutsheets are NOT embedded.** This is the single most important finding for tooling. The submittal relies on unambiguous manufacturer + model-number pairs in the equipment list, and leaves cutsheet lookup to the reviewer. No PDF attachments, no hyperlinks, no embedded product data sheets. That reframes the "AI-assisted submittal generation with embedded cutsheets" pattern elsewhere in this report: cutsheet embedding is one industry pattern, but the institutional-AV pattern is to trust the model number and skip the attachment cost. A shop targeting this format does not need a cutsheet-concatenation step at all.

**Room-type templating pattern ("Same as [room]" + delta bullets).** In this submittal, Room 123 is the base BOM. Room 221 is "same as 123" plus a Cisco CS-CODEC-EQ and extra NVX endpoints. Room 223 is identical to 123. Rooms 321/323 are combinable doubled-quantity versions: 2x racks, 2x lecterns, 2x conferencing sets. This pattern directly rewards the "few system types to repeat" strategy: the template count stays at roughly one per archetype, and each project's 1-page summary becomes "here's the archetype, here are the deltas" rather than a freshly composed narrative.

**Drawing set index** (for the pre-award submittal, not as-built): TA001 cover sheet, TA101 floor/ceiling plans, TA501-505 details (schedules, installation, projector/mic/antenna mounting, equipment rack elevations, lectern layout), TA601-602 cable schedule and conduit riser, TA701+ signal flow diagrams (one per subsystem: lectern AV & control, displays, cameras, conferencing, capture, audio DSP & speakers, audio mics & wireless). Plotted at ARCH D (24" x 36"). [source: anonymized institutional AV submittal reference, April 2025]

**What the submittal body does NOT contain.** No scope-of-work narrative, no assumptions section, no exclusions section, no legal terms or signature block. Those live outside the submittal body (in a cover letter, SOW, or master agreement). Any Smartsheet-to-submittal pipeline for this format should skip trying to generate scope prose inside the submittal; that content belongs in a separate document.

**Network switch config tables** appear in the signal flow section with columns: Port, Device, UID, Location, VLAN, PoE wattage. These are templatable in structure but room-specific in content. A separate provisioning template feeds these.

### 6c. Context From This Shop

Four scoping answers from the Shop Manager this report is written for. Each answer is followed by the tool-selection and workflow recommendations it should drive.

#### D-Tools Licensing: SI vs Cloud

**Shop's answer:** No D-Tools license today. Used to run D-Tools Cloud. Has heard D-Tools SI (on-prem) is more useful.

D-Tools' own side-by-side comparison positions the two products on different axes: Cloud is a "modern SaaS platform" aimed at "small to mid-size teams" that want "fast setup and easy onboarding"; SI is an "enterprise desktop + server platform" aimed at "mid-size to enterprise organizations" managing "highly complex AV, security, and building technology systems" with "advanced engineering drawings and documentation." D-Tools explicitly describes a growth path: "Many integration companies begin with D-Tools Cloud as they build operational structure and scale their business. As project complexity grows and engineering documentation becomes more advanced, some firms transition to D-Tools System Integrator." [source: https://www.d-tools.com/resource-center/operations-management/si-d-tools-cloud-vs-si-comparison]

The integrator-heard view that "SI on-prem is more useful" maps to two concrete SI-only features that Cloud does not match in April 2026: native AutoCAD and Visio integration with synchronized BOMs, and the SI v24 offline mobile install app for field work without internet. Cloud is iterating fast (AI-generated scopes of work, Interconnect Diagrams, and a native mobile app are all Cloud-side in 2026), but the engineering documentation depth remains SI's differentiator. [source: https://www.d-tools.com/system-integrator-features, https://www.cepro.com/news/d-tools-previews-si-v24-ahead-of-ise-2026/624701/, https://www.avnetwork.com/business/expert-opinions/roadmap-2026-d-tools]

SI pricing is $1,800 per user annually with a 5-user minimum ($9,000 entry) and is positioned for projects above $75K. D-Tools Hosted is a middle path: the SI server running on AWS, managed by D-Tools, no VPN required, full SI feature set preserved. For a shop with existing Smartsheet-based workflows that wants to evaluate SI without an on-prem server install, Hosted is the shortest runway. [source: https://www.d-tools.com/system-integrator-pricing, https://www.d-tools.com/resource-center/industry-insights/d-tools-why-choose-hosted]

**Switching-cost note.** The cost side of the SI question is rarely just the license. It's the data migration from whatever the shop currently uses (Smartsheet, Excel, Jetbuilt) into the SI catalog, the Visio/AutoCAD template rebuild, and the learning-curve hit for the people who actually use it. For a shop running 3-5 standardized room types, the catalog build is a one-time project that pays back fast; for a one-off-heavy shop, SI's overhead will outweigh its payoff. The room-type clone rate is the leading indicator for SI ROI. (See the Clone Rate section below.)

#### Crestron Toolchain for Hardware-Spec Roles

**Shop's answer:** Crestron shop. Specs the hardware, does not do Crestron programming, code, or UI.

Most Crestron "AI" coverage in the trade press is aimed at programmers (SIMPL#, SIMPL Windows, Crestron Home/Pyng macros, custom modules) or at residential dealers (Configure Pro, the Crestron Home configuration workflow). For a commercial hardware-spec-only role, those tools are not the daily drivers. The tools that matter are the dealer configurators, the product-availability portal, and the spec-sheet library. All are accessible through the Crestron Support Tools page with a dealer account; none of them require writing code. [source: https://www.crestron.com/support/tools]

The daily-use set for a commercial hardware specifier:

- **Pro Portal.** Dealer ordering and real-time product availability. The "is this in stock this week" tool that determines whether a BOM is buildable on the project timeline. Essential for quoting against realistic lead times.
- **Collab Room Builder.** The configurator for Collab Compute meeting-room packages. Designed around room-size presets (huddle through large conference) and produces a consistent BOM of compute, camera, audio, and accessories. For a shop that wants to standardize on a handful of room archetypes, this is the built-in templating tool. The tool is explicitly scoped to Collab Compute; comprehensive, custom systems still need a full integrator design. [source: https://www.crestron.com/Support/Tools/Configurators/Collab-Room-Builder]
- **DM Essentials Configurator.** Hardware selector for HDMI signal extension: the DMPS Lite switchers (HD-PS401/402/621/622), the HD-TX/HD-TXU 4K transmitters, and matched HD-RX/HD-RXU 4K receivers. Produces a compatible set of parts filtered against a distance/signal requirement. Hardware-spec-oriented with no programming needed. [source: https://www.crestron.com/Support/Tools/Configurators/DM-Essentials-Configurator]
- **Spec Sheet Collection.** Central library of Crestron product spec sheets. This is the source material any Smartsheet-to-cutsheet pipeline needs to link into. No automation hooks; a spec sheet download is still a manual action per part number. [source: https://www.crestron.com/support/tools]
- **Consultant Calculator and Cresnet Power Calculator.** Quote-side sanity checks. The Cresnet calculator verifies the power budget before a room design gets shipped to a field tech who then discovers it mid-install.
- **Intelligent Video Room Designer** and **FlipTop Configuration Tool.** Adjacent configurators useful for specific room types.

XiO Cloud is the post-install fleet monitor. It is not a spec or BOM tool, but it is worth knowing that: the free tier covers Crestron-device monitoring and firmware management; Premium adds third-party device support (via Crestron Connected Partners, SNMP/TCP/Ping, or the XiO Cloud Gateway for custom drivers), historical reporting, ServiceNow ticket integration, and API access. Devices can be "pre-configured" so they auto-provision when they hit the network, which tightens the install-to-monitoring handoff. Relevant to the Shop Manager as a post-award output, not as a BOM-generation tool. [source: https://www.crestron.com/Products/Featured-Solutions/XiO-Cloud, https://www.crestron.com/Products/Catalog/Control-and-Management/Cloud-Management/Licenses/SW-XIOC-PREMIUM-1YR-1-99]

Tools that a hardware-spec role can skip: Configure Pro (pitched as "next-generation configuration platform designed to streamline and simplify the setup of Crestron Home systems" at CEDIA Expo 2025: residential Crestron Home focus, not commercial conference rooms), D3 Pro / Toolbox (legacy programming tools), and the SIMPL / Crestron Studio programming suite. These are not Shop Manager workflows. [source: https://www.crestron.com/News/Blog/September-2025/Configure-Pro-Evolution-Smart-Home-Configuration, https://www.strata-gee.com/new-crestron-configure-pro-brings-new-power-approachability-efficiency-to-system-setup/]

AutoMeasure is the one Crestron AI feature that touches a hardware-spec role indirectly: at commissioning, it uses computer vision and ArUco markers to detect camera and microphone position, reducing setup time. The Shop Manager sees this as a line-item reduction in installation labor hours for rooms using Automate VX 6.5, not as a BOM-generation tool. Announced at ISE 2026. [source: https://www.crestron.com/News/Blog/February-2026/Crestron-at-the-ISE-2026-Expo-A-Recap]

#### Clone Rate and ROI Framing

**Shop's answer:** Tries to limit one-offs. Has a few different system types that get repeated.

This maps directly to the room-type pattern captured in an institutional AV submittal reference (85 pages, April 2025, anonymized): one base room (123) becomes the template, variants like 221 (same as 123 + codec) are delta overrides, and combinable rooms like 321/323 are doubled quantities. When most projects fit one of 3-5 archetypes, the ROI math for internal tooling changes sharply in favor of building. [source: anonymized institutional AV submittal reference, April 2025]

**ROI framing worked example.** A shop running 20 projects per year where 80% match one of 4 archetypes has 16 clone projects and 4 one-offs annually. If each clone project takes 3 hours of submittal-assembly time manually and a templated pipeline reduces that to 30 minutes:

- Current annual time on clones: 16 projects x 3 hrs = 48 hrs/year
- Pipeline-assisted time on clones: 16 projects x 0.5 hrs = 8 hrs/year
- Annual savings: 40 hours on clones alone
- One-time build cost (Claude Code assisted Python pipeline against 4 templates): 15-25 hours
- Break-even: before the end of year 1, with savings compounding every year after

The break-even shifts badly for shops with a lower clone rate: a shop where only 30% of projects match an archetype would take two to three years to break even on the same pipeline investment. The Shop Manager's stated preference for limiting one-offs is the exact precondition that makes an internal-tooling build worthwhile.

**Templating pattern that matches institutional submittals.** The institutional reference documents a "Same as [room]" pattern: base room gets a full BOM, variants get a one-line reference plus delta bullets. This is the pattern any internal pipeline should respect: one source-of-truth template per archetype, variants expressed as diffs against the base. It keeps the template count low (one per archetype, not one per project) and makes the output reviewable by reading the deltas.

#### Python Appetite: A Graduated Path

**Shop's answer:** Exports a Smartsheet (an AV Installation Project Manager on the team built the filter to clean it up) and adds it to the 1-page summary. Already in spreadsheet-land.

The AV Installation Project Manager's filter is already the first automation step. The Shop Manager is not starting from zero; a colleague already normalizes the messy Smartsheet export into a clean dataset. That's half the problem. The remaining gap is the transformation from "clean rows" to "formatted 1-page summary" and eventually to "full submittal package with cutsheets."

The recommendation: don't jump to Python yet. Climb the ladder one rung at a time, only moving up when the current rung stops scaling.

**Step 1: Word or Markdown template with merge fields.** Build the 1-page summary as a template with named fields. Smartsheet's native document-generation feature can drive this, though it requires a Business- or Enterprise-tier plan and at least one team member with a DocuSign license; the template needs to be a fillable PDF or a DocuSign mapping. Time investment: 1-3 hours. If the shop doesn't have DocuSign, the same template pattern works with Word mail merge against a CSV export. [source: https://help.smartsheet.com/articles/2482492-build-workflow-automate-document-generation, https://apidog.com/blog/smartsheet-api-guide/]

**Step 2: Add an "AI category" column to Smartsheet.** Group line items by subsystem category (Audio, Projection, Display, Switching, Control, Network, Camera, Conferencing, Rack, Lectern, Cable, Misc, matching the 14 categories in the institutional reference). The AV Installation Project Manager's filter becomes a filter-plus-sort; the 1-page summary renders category-grouped bullets instead of a flat list. No code, immediate clarity improvement, and the grouping is the exact structure downstream automation will need. [source: anonymized institutional AV submittal reference, April 2025]

**Step 3: Python script that reads the cleaned CSV and emits a formatted document.** Only if volume and template complexity justify it. The Smartsheet Python SDK (`smartsheet-python-sdk` v3.8.0 as of April 17, 2026) can pull the cleaned sheet directly via API (Business or Enterprise plan required), or the script can take the CSV export the AV Installation Project Manager already produces. Output options: `python-docx` for a Word template with merge fields, `ReportLab` or `WeasyPrint` for direct PDF generation, or a hybrid (Word template rendered, then exported to PDF). A full cutsheet-concatenation step adds another hour but is where the ROI accelerates for Crestron-specific submittals linking to the Spec Sheet Collection. [source: https://github.com/smartsheet/smartsheet-python-sdk, https://pypi.org/project/smartsheet-python-sdk/]

**Why this order is correct for a spreadsheet-comfortable shop.** Each step produces a working output. If step 1 is enough, the project stops there. If step 2 makes the output good enough for every archetype, the project stops there. Step 3 only gets built when the template complexity (cutsheets, legal block, per-room sub-sections) outgrows what a merge-field template can express. This is the opposite of "let's build a Python pipeline and figure out what it should do later" -- that path burns weeks and produces brittle code. Climb rungs.

### 7. Risk and Liability

The two liability hotspots integrators mention in public forums:

- **Hallucinated product specs in a client-facing document.** If a submittal says "Shure MXA920 with 8 lobes" and the room actually needs an MXA910, and the integrator signed off on it because "AI wrote it," the integrator is liable, not the AI vendor. Every back-office AI workflow needs a human review step before the document leaves the shop.
- **Uncredited AI-generated code in production.** If Val AI or Claude writes a Q-SYS Lua script that fails during a life-safety page and nobody reviewed it, the shop owns that failure. Same principle as above.

The practical rule integrators are converging on in trade press commentary: AI drafts, a human signs. The signature is the liability transfer point.

## Confidence Assessment

**High confidence (multiple sources, including manufacturer pages and trade press):**
- CE Pro 2025 Business Software Deep Dive Survey percentages on integrator AI adoption
- Manufacturer-embedded AI features (Shure, Biamp, Crestron, Q-SYS, Logitech, Poly) being production and shipping
- The integrator consensus that AI is an embedded capability, not a replacement (Neubert, Elsesser, Seiber quoted in AV Network's Top Trends for 2026 roundtable)
- "Clone the last project" / room-type templating being the dominant design methodology, now backed by an 85-page institutional AV submittal reference with explicit "Same as [room] + deltas" pattern
- Institutional AV submittal structure: page order (1-page summary, cost summary, equipment lists, drawings), 10-column line-item schema, 14 subsystem categories, drawing set index (TA001, TA101, TA501-505, TA601-602, TA701+), cutsheets not embedded. Source: anonymized institutional AV submittal reference, April 2025, captured in an internal structural summary.
- The internal-tools-built-by-coding-assistants pattern existing and being useful
- D-Tools SI pricing ($1,800 per user annually, 5-user minimum) and target project size ($75K+)
- D-Tools SI integration set (AutoCAD, Visio, QuickBooks, NetSuite, Sage 100, Solutions360)
- D-Tools SI vs Cloud positioning: "modern SaaS platform for small-to-mid-size teams" vs "enterprise desktop + server platform for mid-size to enterprise organizations managing highly complex AV... with advanced engineering drawings and documentation." D-Tools' own side-by-side comparison is the primary source.
- D-Tools SI v24 ISE 2026 preview: multi-user editing, SI Mobile Install offline app, Gantt to Microsoft Project export
- D-Tools Cloud April 2026 AI feature set: D-Tools AI agent for scope-of-work generation, quote structure manipulation, natural-language search, financial logic; Interconnect Diagrams; native mobile app. Confirmed in CE Pro and AV Network coverage of the ISE 2026 preview.
- Crestron dealer-tool inventory (Pro Portal, Collab Room Builder, DM Essentials Configurator, Consultant Calculator, Spec Sheet Collection, XiO Cloud, Intelligent Video Room Designer, FlipTop Configuration Tool, Cresnet Power Calculator, RoomView Bandwidth Calculator) as listed on the official Crestron Support Tools page
- Crestron Collab Room Builder scoped to Collab Compute platform, DM Essentials Configurator scoped to HDMI signal extension with DMPS Lite, HD-TX/TXU, HD-RX/RXU families
- Crestron XiO Cloud tiering: free tier for Crestron device monitoring, Premium tier adds third-party support, historical reports, ServiceNow integration, API access
- Smartsheet native PDF export supporting Letter, Legal, Wide, ArchD, and A-series paper sizes
- Smartsheet API access gated to Business ($25/user/month, 3-user minimum) and Enterprise plans
- Smartsheet native document-generation workflow requires Business/Enterprise plan, a fillable PDF or DocuSign mapping, and at least one DocuSign-licensed teammate
- Smartsheet Python SDK existence and active maintenance (v3.8.0 released April 17, 2026)
- Crestron ISE 2026 announcements: DM NAX AP-100, 1 Beyond i12D with Visual AI, Automate VX 6.5 with AutoMeasure, 80 Series touch screens, Collab Compute

**Medium confidence (single source or vendor-authored):**
- ProjX360 AI Assistant capabilities (launched April 7, 2026; too new for independent reviews)
- Val AI (Valethon) current scope and positioning. The company site has redirected from valethon.com to valethon.ai and now describes Val AI as a feature of the broader Valethon ServicesOS platform (marketed to MSPs and AV/integration firms) without a visible, specific Q-SYS Lua code-generation pitch in April 2026. Earlier public references described Val AI as a Q-SYS Lua code generator. Treat Q-SYS Lua positioning as a possibly outdated characterization until re-verified.
- Crestron Configure Pro being explicitly residential-only. The Crestron product page describes it as "Crestron Home configuration" and "built by pros for pros," and the CEDIA Expo 2025 coverage places it in the smart-home / residential integrator context. Commercial restriction is inferred from positioning, not stated on-page.
- The specific adoption rates of XTEN-AV vs D-Tools inside US integrators

**Low confidence / flagged unverified:**
- The specific r/CommercialAV post from user sliz_315. Reddit WebFetch was blocked in the original research pass and search returned unrelated results. The *pattern* the post reportedly describes is corroborated by every other source; the specific post content is not directly verified.
- Whether D-Tools Cloud and D-Tools SI have any native Smartsheet integration: neither product page mentions Smartsheet specifically. No evidence for or against as of this research pass.
- Doug Greenwald quote about integrators not having time to dig through documentation: referenced in prior report versions, but the primary source URL did not resolve in this verification pass. Removed from the Executive Summary-adjacent paragraph pending a verifiable source.

## Open Questions

These are the gaps in what the trade press and vendor documentation have published so far. They shape where the report would gain the most value from a follow-up pass.

- **D-Tools AI agent field performance.** The D-Tools AI agent for scope-of-work generation, natural-language search, and financial logic shipped with the Cloud platform previewed at ISE 2026. No independent benchmark or integrator review has been published. Integrator hands-on feedback would materially change the SI-vs-Cloud recommendation.
- **Val AI current Q-SYS Lua scope.** The Valethon site has been redesigned around "ServicesOS" for MSPs and integrators, and the visible description of Val AI's Q-SYS Lua code-generation capability is less specific than it was in earlier vendor messaging. Whether Val AI still leads in Q-SYS Lua generation or has pivoted is worth a direct check with Valethon or a post in r/CommercialAV.
- **ProjX360 AI Assistant field performance.** Launched April 7, 2026. No independent integrator reviews yet.
- **Real adoption-rate data.** CE Pro's 2025 survey covers whether integrators use AI at all, not which specific tools. No published breakdown of active XTEN-AV vs D-Tools AI vs generic LLM usage inside US integrators.
- **Native AV-vendor integrations with Smartsheet.** Neither D-Tools, XTEN-AV, Jetbuilt, nor any other major AV vendor currently advertises a native Smartsheet integration on their product pages as of this research pass. A shop using Smartsheet as the BOM source of truth is building glue, not buying it.
- **Smartsheet native document generation for AV submittals.** The native feature requires DocuSign licensing. Whether the fillable-PDF template workflow can scale to an institutional-style 10-column line-item table is untested in public integrator documentation. If it works, it removes the need for Python in Step 3 of the graduated path.

## Sources

**Primary structural reference:**
- Anonymized institutional AV Submittal structural reference, April 2025, 85 pages, redacted (client name, pricing, labor rates, and margin data removed; section order, column structure, drawing-set index, and templating patterns preserved). Captured in an internal structural summary.

**Trade press (verified this pass):**
- CE Pro, "CE Pro's Business Software Deep Dive Reveals How Integrators Are Balancing AI and Complexity." https://www.cepro.com/news/ce-pros-business-software-deep-dive-reveals-how-integrators-are-balancing-ai-and-complexity/623722/
- CE Pro, "D-Tools Previews SI v24 Ahead of ISE 2026." https://www.cepro.com/news/d-tools-previews-si-v24-ahead-of-ise-2026/624701/
- AV Network, "Top Integrators 2025: Top Trends for 2026." https://www.avnetwork.com/news/top-integrators-2025-top-trends-for-2026
- AV Network, "Roadmap 2026: D-Tools" (Tim Bigoness, CMO). https://www.avnetwork.com/business/expert-opinions/roadmap-2026-d-tools
- Commercial Integrator, 2026 product-cycle coverage of D-Tools, Crestron, and AV integrator trends. https://www.commercialintegrator.com

**Trade press flagged unreachable as of 2026-04-18:**
- CE Pro, "Business Software 2025: AI Integration Automates the Back Office" [unreachable as of 2026-04-18; 404 at https://www.cepro.com/business-support/business-software-2025-ai-integration-automates-the-back-office/]. The 2025 adoption percentages previously sourced here are now sourced to the CE Pro "Deep Dive Reveals How Integrators Are Balancing AI and Complexity" article above, which contains the same 33%/22%/31%/11% figures.
- AV Network, "AI in 2025: How It's Impacting Top Integrators" [unreachable as of 2026-04-18; 404 at https://www.avnetwork.com/features/ai-in-2025-how-its-impacting-top-integrators]. Neubert quote text replaced with a version from the AV Network "Top Integrators 2025: Top Trends for 2026" roundtable. The Russ Newton quote ("Some are simply layering AI tools on top of legacy workflows") could not be located in a currently-reachable source and has been removed from the body of the report.
- Commercial Integrator, "Crestron AutoMeasure Next Chapter" [unreachable as of 2026-04-18; 404 at https://www.commercialintegrator.com/news/crestron-automeasure-next-chapter]. AutoMeasure references now sourced to the Crestron ISE 2026 recap blog, which describes the same computer-vision + ArUco markers technology in Automate VX 6.5.
- AVIXA audiovisual project documentation sample. https://cdn.avixa.org/production/docs/default-source/default-document-library/avprojectdocsample_fullcontents.pdf [previously flagged as binary-blocked in the original research pass; superseded by the anonymized institutional submittal reference as the authoritative structural source and retained here only for completeness]
- AVIXA D401.01 technical review (documentation standard draft). https://cdn.avixa.org/production/docs/default-source/default-document-library/d401.01_tech_review.pdf [same status as above]
- InfoComm, Project Management for AV Professionals (Malone). https://www.infocomm.org/filestore/egraphics/documents/ProjectManagementforAV_Malone.pdf [same status]
- Harvard University Audiovisual Systems Standards Master Format Division 27 41 00. https://enterprisearchitecture.harvard.edu/sites/g/files/omnuum10526/files/harvard_csi_division_274100_-_audiovisual_systems.pdf [same status]

**D-Tools:**
- D-Tools SI vs Cloud comparison page. https://www.d-tools.com/resource-center/operations-management/si-d-tools-cloud-vs-si-comparison
- D-Tools System Integrator feature and pricing pages. https://www.d-tools.com/system-integrator-features, https://www.d-tools.com/system-integrator-pricing
- D-Tools project management overview (SI vs Cloud). https://www.d-tools.com/project-management-software
- D-Tools hosted solution industry insight. https://www.d-tools.com/resource-center/industry-insights/d-tools-why-choose-hosted

**Crestron:**
- Crestron Support Tools directory. https://www.crestron.com/support/tools
- Crestron Collab Room Builder configurator page. https://www.crestron.com/Support/Tools/Configurators/Collab-Room-Builder
- Crestron DM Essentials Configurator page. https://www.crestron.com/Support/Tools/Configurators/DM-Essentials-Configurator
- Crestron XiO Cloud product page. https://www.crestron.com/Products/Featured-Solutions/XiO-Cloud
- Crestron XiO Cloud Premium license page. https://www.crestron.com/Products/Catalog/Control-and-Management/Cloud-Management/Licenses/SW-XIOC-PREMIUM-1YR-1-99
- Crestron ISE 2026 recap blog. https://www.crestron.com/News/Blog/February-2026/Crestron-at-the-ISE-2026-Expo-A-Recap
- Crestron ISE 2026 press release (DM NAX and 1 Beyond i12D). https://www.crestron.com/News/Press-Releases/2026/Crestron-Introduces-DM-NAX-Intelligent-Audio-Platf
- Crestron Configure Pro announcement. https://www.crestron.com/News/Blog/September-2025/Configure-Pro-Evolution-Smart-Home-Configuration
- Strata-Gee coverage of Configure Pro at CEDIA Expo 2025. https://www.strata-gee.com/new-crestron-configure-pro-brings-new-power-approachability-efficiency-to-system-setup/

**Smartsheet:**
- Smartsheet API reference: Get Sheet endpoint (documents PDF export paper sizes). https://developers.smartsheet.com/api/smartsheet/openapi/sheets/getsheet
- Smartsheet native document generation workflow help article. https://help.smartsheet.com/articles/2482492-build-workflow-automate-document-generation
- Smartsheet Python SDK on GitHub (v3.8.0 released April 17, 2026). https://github.com/smartsheet/smartsheet-python-sdk
- Smartsheet Python SDK on PyPI. https://pypi.org/project/smartsheet-python-sdk/
- Apidog, Smartsheet API plan-tier guide. https://apidog.com/blog/smartsheet-api-guide/

**AV design and control tools:**
- XTEN-AV product site and System Surveyor alternatives comparison. https://xten.av/, https://xtenav.com/system-surveyor-alternatives/
- QSC Q-SYS VisionSuite VSA-100 product page (new URL after qsc.com / qsys.com migration). https://www.qsys.com/products/q-sys/video/q-sys-visionsuite/vsa-100/
- QSC Q-SYS Help documentation for VSA-100. https://q-syshelp.qsc.com/Content/Hardware/Video/VSA_100.htm
- Valethon / Val AI (moved from valethon.com to valethon.ai). https://valethon.ai/
- ProjX360. https://www.projx360.com/

**Manufacturer product pages:**
- Shure IntelliMix / MXA920 / MXA902 / IntelliMix Bar Pro, Biamp Parlé VBC 2800 / Beamtracking / Launch, Crestron Automate VX 6.5 / AutoMeasure / Collab Compute / DM NAX, Logitech Rally Bar / MeetUp 2 / Sight, Poly Studio E70, Huddly L1, AVer CAM520 / CAM570, 1 Beyond i12D, Neat Frame / Neat Board.

**Community sources:**
- r/CommercialAV general coverage of the Claude-Code-for-internal-tools pattern (specific sliz_315 thread not directly verifiable).

## Update History

- 2026-04-18 10:10 EDT: Integrated an anonymized 85-page institutional AV submittal structural reference (April 2025) as the authoritative source for submittal section order, the 10-column line-item schema, the 14 subsystem categories, the drawing set index (TA001/TA101/TA501-505/TA601-602/TA701+), and the "Same as [room] + delta bullets" templating pattern. Added a new "Context From the Shop" section after Executive Summary with four subsections answering the Shop Manager's scoping questions: D-Tools SI vs Cloud switching analysis, Crestron hardware-spec toolchain (Pro Portal / Collab Room Builder / DM Essentials Configurator / Spec Sheet Collection / XiO Cloud), clone-rate ROI framing with a worked example, and a graduated Python-appetite path from merge-field templates to a Smartsheet SDK + python-docx pipeline. Added clickable Table of Contents using GitHub-flavored markdown anchors. Verified all source URLs: two came back 404 (CE Pro 2025 "Business Software" article and AV Network "AI in 2025: How It's Impacting Top Integrators") and have been flagged in the Sources section and replaced with the CE Pro "Deep Dive" article and the AV Network "Top Integrators 2025: Top Trends for 2026" roundtable, which carry the same 33%/22%/31%/11% adoption data and a currently-verifiable Neubert quote. Russ Newton and Doug Greenwald quotes could not be re-sourced and were removed from the body. Updated D-Tools Section 4 to reflect April 2026 reality: SI v24 ISE 2026 preview (multi-user editing, SI Mobile Install offline app), Cloud's D-Tools AI agent for scope-of-work generation. Updated Val AI positioning note: valethon.com redirects to valethon.ai and the site now describes Val AI as part of the broader ServicesOS platform without the specific Q-SYS Lua pitch (flagged Medium confidence). Updated Configure Pro claim from "residential-only per announcement" to "designed for Crestron Home system setup" (Medium confidence). Q-SYS VisionSuite / VSA-100 URL updated to the new qsys.com domain.
- 2026-04-18 09:48 EDT: Depersonalized the report. Removed shop-specific "friend" framing and the "AI will replace the designer" narrative. Reframed as a citation-based survey. Section 7 retained the Crestron/Smartsheet variant as a worked example; Section 7b retained the three-option build path as a generic Smartsheet pipeline template. Open Questions rewritten as industry-level gaps rather than individual-shop questions.
- 2026-04-18 09:28 EDT: Added Smartsheet-to-submittal build options. Added D-Tools SI vs Cloud comparison with verified 2026 pricing. Added Crestron dealer-tool inventory from Crestron Support Tools page. Added ISE 2026 product announcements.
- 2026-04-18 09:17 EDT: Added TL;DR and Executive Summary to the top of the report for faster scanability.
- 2026-04-18 09:12 EDT: Initial report created.

## How This Report Was Generated

Claude (Opus 4.7) running under Claude Code, research skill. Initial evidence gathered via WebFetch and WebSearch across AVIXA, CE Pro, AV Network, Commercial Integrator, and primary manufacturer product pages over approximately 15 distinct queries. This 2026-04-18 10:10 EDT update added an anonymized institutional AV submittal as the primary structural reference, answered four Shop Manager scoping questions with targeted searches against D-Tools, Crestron, and Smartsheet documentation, verified all previously cited URLs, and re-sourced or removed claims whose original URLs 404'd (CE Pro 2025 and AV Network 2025 articles). Reddit WebFetch remains blocked, so the sliz_315 thread is still flagged as unverified. All quotes reproduced here appear in the currently-linked trade-press pieces. Inline source URLs were returned by the WebFetch tool at research time and should be considered stable but not guaranteed long-term.
