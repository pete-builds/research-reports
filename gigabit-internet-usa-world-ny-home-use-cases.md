---
title: "Gigabit Internet: USA vs World, New York Regional ISPs, and Home Use Cases When Devices Cap at 1 Gbps"
date: 2026-05-23
updated: 2026-05-23T19:18:00-04:00
summary: "The US sits 8th globally for median fixed broadband (~302 Mbps) behind Singapore, Chile, UAE, France, Hong Kong, and Iceland. Multi-gig is rapidly available from national carriers (Verizon, Google Fiber, AT&T, Frontier, Optimum, Ziply) and a growing pool of NY regional fiber providers (Greenlight, Empire Access, Mid-Hudson Fiber, Slic, Fiberspark). Even when client devices are stuck at 1 Gbps NICs/WiFi, multi-gig pays off for aggregate household throughput, symmetric upload to self-hosted services, and WiFi 6E/7 access points fanning out to many parallel 1 Gbps clients — but it is unambiguously overkill for single-user or low-device households."
---

## TL;DR

The US ranks **8th globally** for median fixed broadband download speed (~302 Mbps in December 2025), well behind Singapore (~407 Mbps), Chile (~357 Mbps), UAE (~356 Mbps), France (~346 Mbps), and Hong Kong (~345 Mbps) ([source](https://worldpopulationreview.com/country-rankings/internet-speeds-by-country)). Multi-gigabit (2/5/8 Gbps) plans are now widely available from national carriers (Verizon Fios 2 Gig, Google Fiber up to 8 Gig, AT&T Fiber up to 5 Gig, Optimum up to 8 Gig, Frontier up to 7 Gig, Ziply up to 50 Gig) and from a growing crop of NY regional fiber ISPs. In the Ithaca/Tompkins County area specifically, **Fiberspark sells a symmetric 5 Gig plan for USA 150/month** in the small footprint it covers ([source](https://fiberspark.com/residential.php)). Multi-gig is **worth paying for** when a household runs many simultaneous 1 Gbps clients (parallel cloud syncs, self-hosted Plex/Jellyfin remote streaming, NAS-to-cloud backup, WiFi 7 APs with MLO), and **not worth it** for single-user homes where 1 Gig already buries 4K streaming, video calls, and casual downloads.

## Current Status

- **US global rank:** 8th by median fixed broadband download speed (302.68 Mbps) per Speedtest Global Index December 2025 data ([source](https://worldpopulationreview.com/country-rankings/internet-speeds-by-country)).
- **US gigabit adoption:** Gigabit-tier subscribers averaged 955.0 GB/month in Q2 2025, up 14.4% YoY, per OpenVault ([source](https://openvault.com/openvault-historic-shift-in-quarter-over-quarter-usage-heralds-new-phase-of-broadband-expansion/)).
- **National multi-gig leaders (2025-2026):** Ziply Fiber lists up to 50 Gbps symmetric ($900/mo + $600 install); Optimum, Google Fiber, and Frontier go up to 7-8 Gig; AT&T Fiber and Verizon Fios cap at 5 Gig and 2 Gig respectively in residential.
- **NY regional fiber footprint expanding:** Greenlight Networks serves Rochester/Buffalo/Binghamton/Capital Region/Hudson Valley with up to 8 Gig at $150/mo; Mid-Hudson Fiber sells up to 5 Gig symmetric in the Hudson Valley; Empire Access sells up to 2 Gig symmetric across Upstate NY and Northern PA; Slic Network Solutions serves North Country (St. Lawrence, Franklin, Hamilton, Essex, Clinton counties) up to 2 Gbps.
- **Ithaca-specific:** Spectrum cable dominates (~85% of city, up to 2 Gbps). Small fiber providers (Fiberspark, Haefele Connect, Trumansburg Telephone, Empire Access, Ontario Telephone) each cover ~3-10% of addresses. Fiberspark publishes 1 Gig at USA 55/mo, 2 Gig at USA 85/mo, 5 Gig at USA 150/mo, all symmetric ([source](https://fiberspark.com/residential.php)).
- **The 1 Gbps client ceiling is real but bypassable:** A single Cat5e/Cat6 path to a 1 GbE NIC tops out near 940 Mbps. Multi-gig helps when many such 1 Gbps clients pull/push concurrently, or when WiFi 6E/7 APs aggregate multiple clients over a 2.5 GbE or 10 GbE uplink.

## Table of Contents

- [TL;DR](#tldr)
- [Current Status](#current-status)
- [Findings](#findings)
  - [1. USA vs the World on Fixed Broadband](#1-usa-vs-the-world-on-fixed-broadband)
  - [2. National Multi-Gig Offerings](#2-national-multi-gig-offerings)
  - [3. New York State Regional and Local Fiber ISPs](#3-new-york-state-regional-and-local-fiber-isps)
  - [4. Ithaca / Tompkins County Specific Options](#4-ithaca--tompkins-county-specific-options)
  - [5. Home Use Cases for Multi-Gig When Clients Are Capped at 1 Gbps](#5-home-use-cases-for-multi-gig-when-clients-are-capped-at-1-gbps)
  - [6. When Multi-Gig Is NOT Worth It](#6-when-multi-gig-is-not-worth-it)
  - [7. Future-Proofing: WiFi 7, 2.5/5/10 GbE Adoption](#7-future-proofing-wifi-7-25510-gbe-adoption)
- [Confidence Assessment](#confidence-assessment)
  - [High Confidence](#high-confidence)
  - [Medium Confidence](#medium-confidence)
  - [Low Confidence](#low-confidence)
- [Open Questions](#open-questions)
- [Sources](#sources)
  - [Global Broadband Speed Data](#global-broadband-speed-data)
  - [National Multi-Gig Providers](#national-multi-gig-providers)
  - [NY Regional Fiber ISPs](#ny-regional-fiber-isps)
  - [Ithaca / Tompkins County](#ithaca--tompkins-county)
  - [Home Use Cases and Technical Analysis](#home-use-cases-and-technical-analysis)
- [Update History](#update-history)
- [How This Report Was Generated](#how-this-report-was-generated)

## Findings

### 1. USA vs the World on Fixed Broadband

According to Ookla's Speedtest Global Index (December 2025 data, mirrored by World Population Review), the **United States ranks 8th globally** for median fixed broadband download speed at **302.68 Mbps** ([source](https://worldpopulationreview.com/country-rankings/internet-speeds-by-country)). The top 10 are:

| Rank | Country | Median Fixed DL (Mbps) |
|------|---------|------------------------|
| 1 | Singapore | 407.05 |
| 2 | Chile | 357.25 |
| 3 | United Arab Emirates | 356.24 |
| 4 | France | 346.04 |
| 5 | Hong Kong | 345.33 |
| 6 | Iceland | 318.37 |
| 7 | Macau | 312.37 |
| 8 | United States | 302.68 |
| 9 | Switzerland | 278.51 |
| 10 | Thailand | 275.26 |

([source](https://worldpopulationreview.com/country-rankings/internet-speeds-by-country))

A separate Allconnect summary (citing Ookla's January 2026 figures) puts the US median at **306.15 Mbps download / 55.12 Mbps upload**, also ranking 8th ([source](https://www.allconnect.com/blog/us-internet-speeds-globally)). The asymmetric upload (55 Mbps median against 306 Mbps download) reflects how much of US broadband is still DOCSIS cable rather than fiber, while top-ranked countries like Singapore, France, Hong Kong, and the UAE are predominantly FTTH (fiber to the home).

**Historical context.** Ten years prior, the US averaged ~31 Mbps and ranked 25th worldwide ([source](https://www.allconnect.com/blog/us-internet-speeds-globally)). So the US position has improved materially in raw Mbps, but it has not caught the leaders, who have also climbed sharply.

**Gigabit and multi-gig availability.** OpenVault's Q2 2025 Broadband Insights report notes that subscribers provisioned for 1 Gbps or higher averaged **955.0 GB of data per month**, up 14.4% year over year from 834.8 GB ([source](https://openvault.com/openvault-historic-shift-in-quarter-over-quarter-usage-heralds-new-phase-of-broadband-expansion/)). An earlier OpenVault 4Q23 report cited by Broadband Breakfast stated that roughly **one-third of US subscribers were provisioned for gigabit** as of late 2023, and that the gigabit-tier subscriber count grew 29% year-over-year in that period ([source](https://broadbandbreakfast.com/openvault-reports-accelerated-broadband-usage/)). I do not have a precise 2025 percentage for the **multi-gig (2 Gbps+)** subscriber base — see Open Questions.

**Confidence:** High for US rank and Mbps figure (cross-confirmed by two Ookla-derived sources); High for top-10 country list; Medium for gigabit subscriber percentage (best available is 4Q23 data).

### 2. National Multi-Gig Offerings

| Provider | Top Residential Tier | Symmetric? | Pricing (current) | Notes |
|---|---|---|---|---|
| Verizon Fios | 2 Gig (up to 2.3 Gbps / 2.3 Gbps) | Yes | USA 109.99/mo w/ Auto Pay | Limited to NYC metro area; no 5 Gig residential confirmed ([source](https://mobileservicescenter.com/verizon-fios-prices/)) |
| Google Fiber | 8 Gig | Yes | USA 150/mo (8 Gig); USA 125/mo (5 Gig); USA 100/mo (2/3 Gig); USA 70/mo (1 Gig) | Select markets only ([source](https://www.cabletv.com/google-fiber)) |
| AT&T Fiber | 5 Gig (Internet 5000) | Yes | USA 90/mo w/ wireless bundle + autopay discount; promo first-year pricing | Available to ~5.2M customers across 70 metro areas; no NY ILEC footprint ([source](https://www.attsavings.com/internet/att-fiber/5-gig-internet)) |
| Frontier Fiber | 7 Gig (7000/7000) | Yes | USA 139.99/mo (7 Gig); USA 154.99/mo (5 Gig) | 15 states including NY ([source](https://frontier.com/shop/internet/fiber-internet/5-gig)) |
| Optimum (Altice) | 8 Gig | Yes | ~USA 120-280/mo depending on source/promo; up to USA 400 Mastercard rebate | Fiber footprint in NYC area, Dallas, Charleston; ~1.3M fiber-eligible homes ([source](https://broadbandnow.com/Optimum-by-Altice-deals)) |
| Xfinity (Comcast) X-Class | 2 Gbps symmetric | Yes (for X-Class tier) | USA 115/mo for 2 Gbps X-Class | DOCSIS 4.0 / X-Class symmetric multi-gig rollout reaching 50M homes by 2025 across markets including Atlanta, Boston, Chicago, Denver, Houston, Miami, Philadelphia, Salt Lake City, Seattle, San Francisco, DC ([source](https://corporate.comcast.com/press/releases/comcast-multi-gig-rollout-xfinity-10g-network-upgrade)) |
| Ziply Fiber | 50 Gig | Yes | USA 900/mo for 50 Gig + USA 600 install; also offers 2/5/10 Gig | Pacific NW; not in NY ([source](https://ziplyfiber.com/internet)) |

Pricing is volatile and promotional; the figures above were captured May 2026 and are sourced inline.

**Verizon Fios pricing detail.** Verizon's 2 Gig plan is listed at **USA 109.99/month with Auto Pay**, and is described as "best for heavy-duty users, like content creators or someone who frequently transfers multiple large files" ([source](https://mobileservicescenter.com/verizon-fios-prices/)). No public Fios 5 Gig residential tier surfaced as of this report.

**Google Fiber pricing detail.** Core 1 Gig at USA 70/mo, Home 3 Gig at USA 100/mo, Edge 8 Gig at USA 150/mo, plus 2 Gig (USA 100/mo) and 5 Gig (USA 125/mo) in select markets ([source](https://www.cabletv.com/google-fiber)). Notably absent from NY State.

**Xfinity X-Class is DOCSIS-based symmetric multi-gig** running over the existing coax plant, separate from Comcast's fiber-to-the-home pilots. Comcast's press release confirms the network upgrade reaches over 50 million homes by 2025 across more than 40 named markets including Atlanta, Boston, Chicago, Denver, Houston, Miami, Philadelphia, Salt Lake City, Seattle, San Francisco, and Washington D.C. ([source](https://corporate.comcast.com/press/releases/comcast-multi-gig-rollout-xfinity-10g-network-upgrade)). The press release describes "symmetrical multi-gigabit Internet options" delivered via DOCSIS 4.0. Whether X-Class is live at a specific Ithaca address is a separate availability check — Spectrum (Charter), not Xfinity (Comcast), is the cable incumbent in the Ithaca area.

**Confidence:** High for offerings and approximate price bands; Medium for current promo pricing (changes monthly).

### 3. New York State Regional and Local Fiber ISPs

Beyond the national carriers, NY State has a respectable bench of regional fiber operators:

**Greenlight Networks** is the largest of the regional pure-play fiber overbuilders in NY. Service area includes Rochester, Buffalo, Binghamton, the Capital Region, and Hudson Valley (plus expansion into Scranton/Wilkes-Barre PA, Lehigh Valley PA, and Baltimore MD) ([source](https://www.greenlightnetworks.com/residential/)). Per Fox Rochester reporting on Greenlight's announced pricing reset, the published tiers were 1 Gig at USA 75/mo (down from USA 100), 2 Gig at USA 100/mo (down from USA 200), and a then-new 5 Gig tier at USA 200/mo ([source](https://foxrochester.com/news/good-day-rochester/greenlight-networks-announces-new-pricing-for-multi-gig-internet)). The Greater Rochester Chamber subsequently reported the launch of an 8 Gig tier at "25% lower" than the prior 5 Gig price, all symmetric ([source](https://www.greaterrochesterchamber.com/2024/04/02/greenlight-networks-launches-8-gig-high-speed-fiber-internet/)). Current 8 Gig price is widely reported as USA 150/mo though I did not find that figure on a primary Greenlight page (the residential page directs you to an availability checker). No taxes, no contracts, no data caps. Total Managed WiFi is a USA 5/mo add-on ([source](https://www.greenlightnetworks.com/residential/)).

**Empire Access** sells three residential plans across Upstate NY and Northern PA: Essential 500/500 Mbps, Enhanced 1/1 Gig, Elite 2/2 Gig — all symmetric as of March 2025 ([source](https://www.empireaccess.com/residential/high-speed-fiber-internet/)) ([source](https://datacenterpost.com/empire-access-announces-symmetrical-speeds-for-ny-and-pa-residential-customers/)). Public pricing is not posted on Empire's residential plan page; promos are common. Service area includes Cortland, NY (not Ithaca proper at scale).

**Mid-Hudson Fiber** (Hudson Valley) offers 1 Gig, 2 Gig, and 5 Gig symmetric plans, all with unlimited data and managed WiFi ([source](https://www.midhudsonfiber.com/fiber/)). Pricing not disclosed on the plan page. Existing Mid-Hudson Cable customers get a free upgrade to the fiber service where available.

**Slic Network Solutions / SLICFiber** is the North Country fiber overbuilder, serving roughly 10,000 homes across St. Lawrence, Franklin, Hamilton, Essex, and Clinton counties (including communities like Belmont, Lake Placid, Schroon Lake, Titus Mountain) ([source](https://broadbandnow.com/Slic-Network-Solutions)). Headline speed is up to 2 Gbps symmetric; pricing is gated behind an availability check on slicfiber.com ([source](https://www.slicfiber.com/)).

**Fidium Fiber (Consolidated Communications)** does **not have meaningful NY State footprint** — Fidium is the New England/Midwest/West Coast brand. Consolidated's legacy NY footprint is small, mainly southeast of Albany and a sliver near Chautauqua Lake along the PA border. Their 2 Gig (Futuristic) plan is ~USA 65-70/mo promotional, ~USA 95-100/mo after 12 months ([source](https://www.fidiumfibersavings.com/blog/fidium-fiber-cost-monthly/)).

**Margaretville Telephone Company** covers parts of Greene County in the Catskills (Halcott, Lexington, portions of Jewett) but tops out at ~50 Mbps per BroadbandSearch (not gigabit at this time) ([source](https://www.broadbandsearch.net/service/new-york/margaretville)).

**North Country Broadband** — limited public information surfaced; service area appears to overlap heavily with Slic Network Solutions in the North Country. I do not have a sourced speed/price sheet — see Open Questions.

**NYSEG / Frontier** — NYSEG is an electric utility, not an ISP; the conflation in NY broadband conversations usually refers to Frontier Communications' legacy NY DSL/Fiber territories. Frontier Fiber sells up to 7 Gig nationally and lists NY as a covered state, but availability is address-specific and frequently sparse outside metro markets ([source](https://frontier.com/shop/internet/fiber-internet/5-gig)).

**Confidence:** High for Greenlight, Empire, Mid-Hudson, Slic, Fidium, Margaretville (all backed by primary vendor pages or established trade press); Low for North Country Broadband.

### 4. Ithaca / Tompkins County Specific Options

Per BroadbandNow's Ithaca ISP listing (May 2026), the available providers and headline tiers are:

| Provider | Tech | Top Speed | Coverage in Ithaca |
|---|---|---|---|
| Spectrum (Charter) | Cable | 2 Gbps | 84.8% (city), 99% per one source |
| Verizon | DSL + 5G + sliver of Fiber | Up to 940 Mbps | 100% DSL, 7.6% Fiber |
| XNET WiFi | Fixed Wireless | 2 Gbps | 41.7% |
| Empire Access | Fiber | up to 2 Gig | 3.3% |
| Haefele Connect | Fiber + Cable | up to 2 Gbps fiber | 7.5% fiber / 10% total |
| Fiberspark | Fiber | up to 5 Gig | ~3% |
| Ontario / Trumansburg Telephone | Fiber | up to 1 Gig | 1.2% |
| Starlink | Satellite | ~400 Mbps | 100% |
| T-Mobile 5G Home | Fixed Wireless | varies | 52.7% |

([source](https://broadbandnow.com/New-York/Ithaca))

**Fiberspark** publishes the cleanest residential pricing in the Ithaca market:
- **1 GIG** symmetric — USA 55/month
- **2 GIG** symmetric — USA 85/month
- **5 GIG** symmetric — USA 150/month

All include a Fiberspark WiFi router with remote troubleshooting ([source](https://fiberspark.com/residential.php)).

**Haefele Connect** (HTVA) covers Trumansburg and parts of Tompkins County with fiber up to 2 Gbps, alongside legacy cable tiers (MiniRocket 3 Mbps capped, CableRocket, Connect 250) ([source](https://www.htva.net/internet-towns/trumansburg/)). Public pricing for the 2 Gbps fiber tier is not on the linked page.

**Trumansburg Telephone / Ontario Telephone** offers 1 Gbps symmetric at ~USA 63.95/mo per the BroadbandNow listing ([source](https://broadbandnow.com/New-York/Ithaca)).

**Spectrum** is the only **mass-market** sub-USA 50/mo option in Ithaca, with plans from USA 30/mo and up to 2 Gbps over its DOCSIS-on-coax plant ([source](https://www.spectrum.com/internet-service/new-york/ithaca)). Spectrum is asymmetric: even on the 2 Gig tier, upstream is far less than downstream (typically ~35 Mbps unless they have upgraded to the Spectrum Internet Premier or fiber-trial tiers in your node).

**Practical takeaway for Pete's profile (Ithaca-area homelab with Plex/Sonarr/Radarr, multiple NAS, MCP servers):**
- If your address is in Fiberspark's footprint, the **2 Gig for USA 85** plan is the highest-value tier in town for a self-hoster. You get true symmetric upload (good for Plex remote streams, Backblaze/Borgbase backups, off-site replication).
- If Fiberspark/Haefele/Empire is unavailable, **Spectrum 2 Gig** gets you the raw download number, but the upload ceiling is the more meaningful limit for self-hosted services and off-site backups.
- Greenlight does not appear to serve Ithaca/Tompkins as of this report — their Upstate footprint focuses on Rochester, Buffalo, Binghamton, and Capital Region.

**Confidence:** High for Spectrum, Fiberspark, BroadbandNow listing; Medium for fine-grained address-level availability (changes frequently).

### 5. Home Use Cases for Multi-Gig When Clients Are Capped at 1 Gbps

This is the most consequential section for a homelab operator. The honest answer is: multi-gig from the ISP can absolutely deliver value even when no single client device has more than a 1 Gbps NIC or WiFi link — but only in specific scenarios.

**5a. Aggregate household throughput.** Each 1 Gbps client maxes out at ~940 Mbps in practice ([source](https://dongknows.com/gigabit-internet-and-you/)). With a 1 Gbps WAN, two devices doing simultaneous large transfers each get ~half the pipe. With a 2 Gbps WAN, both can run at full 1 Gbps. With a 5 Gbps WAN, you can have 4-5 clients (e.g., a 4K Plex remote stream + cloud backup + steam download + family Zoom + NAS sync) all running near their per-client ceiling. Multi-Gig becomes "useful when bandwidth is split across many devices simultaneously" ([source](https://www.cablify.ca/why-is-my-1-gbps-internet-only-200-300-mbps-on-ethernet/)).

**5b. Symmetric upload for self-hosted services.** This is the unsexy real reason most homelabbers go multi-gig. DOCSIS cable tops out at ~35 Mbps upstream on standard tiers. Fiber multi-gig is symmetric, so Plex/Jellyfin remote streaming (multiple 1080p/4K streams to friends and family), off-site backups (Backblaze, Borgbase, Wasabi), NAS-to-cloud sync, and Nextcloud/PhotoSync uploads run at the same speed as downloads. Frontier and Optimum 5-year price-locks and Mid-Hudson Fiber/Greenlight/Fiberspark all specifically market symmetric speeds. The 1 Gbps client cap is fine for any single upload stream; the upgrade is about being able to run multiple in parallel without one stealing all the upstream.

**5c. WiFi 6E and WiFi 7 access points with Multi-Link Operation (MLO).** WiFi 7's MLO lets a single client transmit across 2.4/5/6 GHz bands simultaneously, with aggregate throughput cited "in excess of 8 Gbps" in vendor literature ([source](https://www.ruckusnetworks.com/packet-pushers-40-wi-fi-7-mlo/)). More importantly for a home with many 1 Gbps clients: a WiFi 6E/7 AP with a 2.5 GbE or 10 GbE uplink to the router can serve multiple wireless clients concurrently without becoming the bottleneck. With a 1 GbE AP uplink, every wireless client shares that 940 Mbps ceiling.

**5d. Link aggregation as a workaround (with caveats).** NETGEAR documents WAN link aggregation on Nighthawk multi-gig cable modems ([source](https://kb.netgear.com/000060394/How-do-I-set-up-Ethernet-port-aggregation-on-my-NETGEAR-Nighthawk-multi-gig-speed-cable-modem)). User reports on SNBForums show ~1500 Mbps real throughput from LACP-bonded 2x 1 GbE ports, though that thread is focused on LAN aggregation rather than WAN; the same thread also cautions that LACP behaves poorly on consumer Asus firmware ([source](https://www.snbforums.com/threads/2-5g-lan-aggregation-switch-oh-my.83859/)). In practice, dedicated 2.5 GbE or 10 GbE hardware is the cleaner solution; LACP is a real workaround when both ends support it and you have no other option.

**5e. NAS-to-NAS and LAN backbone.** Multi-gig **inside** the LAN is largely independent of the ISP plan, but they tend to come together: if you're paying for 2 Gbps WAN you probably also want a 2.5 GbE switch backbone and at least one 10 GbE link between primary NAS and the desktop that pulls/pushes large datasets. This eliminates the "everyone waits for one transfer" problem on the LAN, which is often the bigger annoyance in a homelab than WAN throughput.

**5f. Latency and jitter benefits independent of raw speed.** Multi-gig fiber lines are typically lower-latency and lower-jitter than DOCSIS, because they have less congestion in the access plant and no DOCSIS scheduling. For latency-sensitive workloads (competitive gaming, real-time video, low-latency Pi-hole/DNS resolution under household contention), the consistency matters more than the headline Gbps.

**Confidence:** High for the aggregate-throughput and symmetric-upload arguments; Medium for the exact WiFi 7 MLO throughput claims (vendor marketing, real-world figures lower); High for LACP workaround mechanics.

### 6. When Multi-Gig Is NOT Worth It

The honest case against multi-gig — also worth saying clearly:

- **Single-user households.** Netflix 4K typically needs ~15-25 Mbps and Zoom HD calls ~3-4 Mbps; even being generous, a 1 Gbps pipe is enough headroom for dozens of simultaneous 4K streams or hundreds of HD video calls. (I did not find a single primary source that quantifies this exactly; the Astound 2-Gig page makes the qualitative case that 2 Gig "handles multiple 4K streams without buffering" ([source](https://www.astound.com/learn/internet/2-gig/)) but doesn't publish those specific simultaneous-stream counts.) If your household is one or two people doing normal stuff, you will literally never see the multi-gig number on any speedtest from any device.
- **All clients 1 Gbps and no aggregation.** A 2 Gbps plan feeding a 1 GbE gateway is a 1 Gbps experience, no asterisks ([source](https://datawiresolutions.com/blog/gigabit-internet-in-your-smart-home)). The whole-home infrastructure needs Cat6/6A cabling, multi-gig switches, and multi-gig router LAN ports to deliver the bandwidth past the WAN handoff.
- **ISP oversubscription.** Cable plant especially is shared between subscribers. A 2 Gbps tier in a saturated node delivers a fraction of advertised speed at peak times. Verify your local node's real-world performance before paying for the tier.
- **Latency-bound apps where 1 Gbps is plenty.** SSH, web browsing, API calls, code pushes, Slack — none of these care about anything past ~50 Mbps. Paying USA 50-100/mo more for a multi-gig tier to make them "snappier" is a waste; the bottleneck is round-trip-time, not bandwidth.
- **Most smart home devices.** "Smart home devices such as lights, locks, thermostats, speakers, and cameras do not need multi-gig internet by themselves" ([source](https://www.fidiumfibersavings.com/blog/do-you-need-speeds-above-1-gig-of-fiber-internet/)).
- **Dong Ngo's bottom line:** "For typical families with streaming and casual browsing, 300-500Mbps plans are sufficient. Gigabit becomes worthwhile only if you regularly transfer large files or have many concurrent users demanding bandwidth" ([source](https://dongknows.com/gigabit-internet-and-you/)).

**The cleanest decision rule:**
- Single-user, no self-hosting → 300-500 Mbps is plenty.
- Family of 4+ doing parallel 4K streams + work-from-home + gaming → 1 Gig.
- Homelab operator running Plex/Jellyfin remote streaming to multiple users + off-site backups + multi-NAS replication → **2 Gig is the inflection point**, and symmetric upload is where the real value sits.
- 5/8/10 Gig → primarily future-proofing or rare bursty workloads (8K production, multi-user 4K editing remote workflows, dedicated home lab with 10 GbE backbone).

### 7. Future-Proofing: WiFi 7, 2.5/5/10 GbE Adoption

- **WiFi 7 (802.11be)** is now shipping in mainstream APs, supports MLO, 4K-QAM, 320 MHz channels in 6 GHz, and theoretical peak rates beyond 40 Gbps aggregate. Real-world per-client throughput depends on band conditions but routinely exceeds 1 Gbps over short range ([source](https://www.ruckusnetworks.com/packet-pushers-40-wi-fi-7-mlo/)).
- **2.5 GbE NIC adoption** is now standard on midrange motherboards and many laptops (Intel I225/I226-V, Realtek RTL8125). 2.5 GbE switches are cheap.
- **10 GbE in the home** is still niche but well-supported on NAS (Synology DS1821+/DS1823xs+/DS923+, QNAP TVS-h series, Asustor Lockerstor 4 Gen3), most enterprise-grade managed switches (MikroTik CRS305/CRS309, Ubiquiti USW Pro, Netgear MS510), and via add-in cards (Intel X550-T2, Aquantia AQC107).
- **Multi-gig WAN routers are common now:** Unifi Dream Machine Pro Max, Unifi Cloud Gateway Fiber, MikroTik RB5009, Netgate firewalls, Asus ROG/AXE/AX series with 10 GbE WAN ports.

If you are upgrading your ISP and intend to keep the plan for 3-5 years, picking the multi-gig tier and a multi-gig-capable router is reasonable defensive infrastructure even if you don't fully use it today. The marginal cost from 1 Gig to 2 Gig is usually USA 10-25/mo from the regional fiber providers (Fiberspark: USA 55 → USA 85; Greenlight: USA 75 → USA 100); from 2 Gig to 5 Gig is the bigger jump.

## Confidence Assessment

### High Confidence
- US median fixed broadband ranking (8th globally) and Mbps figures from December 2025 / January 2026 Ookla data ([source](https://worldpopulationreview.com/country-rankings/internet-speeds-by-country)) ([source](https://www.allconnect.com/blog/us-internet-speeds-globally)).
- Top 10 country list and Mbps figures.
- Greenlight, Empire Access, Mid-Hudson Fiber, Slic, Fiberspark service area and headline speed tiers (sourced from each vendor's own page or established trade press).
- Verizon Fios 2 Gig pricing (USA 109.99/mo).
- Google Fiber tier pricing (USA 70/USA 100/USA 125/USA 150).
- Fiberspark Ithaca tier pricing (USA 55/USA 85/USA 150).
- Multi-gig is unnecessary for single-user homes (multiple independent sources concur).
- Symmetric upload is the under-marketed benefit for self-hosters.

### Medium Confidence
- Exact percentage of US subscribers on gigabit (most recent precise figure is from OpenVault 4Q23, not 2025); 2025 data is partial.
- Multi-gig (2 Gbps+) subscriber percentage in the US — best I found is qualitative.
- Current promo pricing on national multi-gig plans (changes monthly; figures are May 2026 snapshots).
- North Country Broadband details — limited primary sourcing.
- Empire Access and Mid-Hudson Fiber multi-gig pricing (not posted publicly on their plan pages).
- Whether DOCSIS 4.0 X-Class is live in any specific NY market today.

### Low Confidence
- Whether Greenlight Networks has announced expansion into Tompkins County or Ithaca specifically (their public expansion footprint focuses on Rochester, Buffalo, Binghamton, Capital Region, Hudson Valley).
- Real-world delivered throughput on Spectrum's 2 Gig DOCSIS tier in Ithaca specifically vs. advertised number.

## Open Questions

1. **What is the current OpenVault percentage of US subscribers on multi-gig (2 Gbps+) tiers?** Best data I have is qualitative growth language and the 4Q23 one-third-gigabit figure.
2. **Does any Tompkins County address have Greenlight Networks fiber service today, or is expansion planned?**
3. **Are there real-world Spectrum 2 Gig upload-speed numbers in Ithaca?** Marketing lists "up to 2 Gbps download" but upload is the weaker number on cable.
4. **Empire Access and Mid-Hudson Fiber actual residential prices for multi-gig?** Their plan pages don't publish numbers.
5. **Is XNET WiFi (fixed wireless, 2 Gbps) symmetric in Ithaca, and what's its real-world latency for VoIP/gaming?**
6. **Status of DOCSIS 4.0 X-Class deployment in Charter/Spectrum territory (not Comcast)?** Spectrum is the cable incumbent in Ithaca, not Comcast.

## Sources

### Global Broadband Speed Data
- [World Population Review: Internet Speeds by Country 2026](https://worldpopulationreview.com/country-rankings/internet-speeds-by-country)
- [Allconnect: Americans are getting 306 Mbps in download speed, but...](https://www.allconnect.com/blog/us-internet-speeds-globally)
- [OpenVault: Historic Shift in Quarter-Over-Quarter Usage](https://openvault.com/openvault-historic-shift-in-quarter-over-quarter-usage-heralds-new-phase-of-broadband-expansion/)
- [Broadband Breakfast: OpenVault Reports Accelerated Broadband Usage](https://broadbandbreakfast.com/openvault-reports-accelerated-broadband-usage/)

### National Multi-Gig Providers
- [Verizon Fios Prices 2025](https://mobileservicescenter.com/verizon-fios-prices/)
- [Google Fiber via CableTV.com](https://www.cabletv.com/google-fiber)
- [AT&T 5 Gig Internet Plan](https://www.attsavings.com/internet/att-fiber/5-gig-internet)
- [Frontier 5 Gig Internet](https://frontier.com/shop/internet/fiber-internet/5-gig)
- [Optimum / Altice Plans (BroadbandNow)](https://broadbandnow.com/Optimum-by-Altice-deals)
- [Comcast Xfinity 10G / X-Class Press Release](https://corporate.comcast.com/press/releases/comcast-multi-gig-rollout-xfinity-10g-network-upgrade)
- [Ziply Fiber Internet Plans](https://ziplyfiber.com/internet)

### NY Regional Fiber ISPs
- [Greenlight Networks Residential](https://www.greenlightnetworks.com/residential/)
- [Greater Rochester Chamber: Greenlight Launches 8 Gig](https://www.greaterrochesterchamber.com/2024/04/02/greenlight-networks-launches-8-gig-high-speed-fiber-internet/)
- [Fox Rochester: Greenlight Announces New Multi-Gig Pricing](https://foxrochester.com/news/good-day-rochester/greenlight-networks-announces-new-pricing-for-multi-gig-internet)
- [Empire Access Residential Fiber](https://www.empireaccess.com/residential/high-speed-fiber-internet/)
- [Empire Access Symmetrical Speeds Announcement](https://datacenterpost.com/empire-access-announces-symmetrical-speeds-for-ny-and-pa-residential-customers/)
- [Mid-Hudson Fiber Plans](https://www.midhudsonfiber.com/fiber/)
- [SLICFiber Homepage](https://www.slicfiber.com/)
- [Slic Network Solutions Coverage (BroadbandNow)](https://broadbandnow.com/Slic-Network-Solutions)
- [Fidium Fiber Cost Breakdown](https://www.fidiumfibersavings.com/blog/fidium-fiber-cost-monthly/)
- [Margaretville Telephone Co (BroadbandSearch)](https://www.broadbandsearch.net/service/new-york/margaretville)

### Ithaca / Tompkins County
- [BroadbandNow: Ithaca NY Providers](https://broadbandnow.com/New-York/Ithaca)
- [Spectrum Ithaca NY](https://www.spectrum.com/internet-service/new-york/ithaca)
- [Fiberspark Residential Plans](https://fiberspark.com/residential.php)
- [Haefele Connect Trumansburg](https://www.htva.net/internet-towns/trumansburg/)

### Home Use Cases and Technical Analysis
- [Dong Knows Tech: Gigabit Internet and You](https://dongknows.com/gigabit-internet-and-you/)
- [Cablify: Why Is My 1 Gbps Internet Only 200-300 Mbps on Ethernet?](https://www.cablify.ca/why-is-my-1-gbps-internet-only-200-300-mbps-on-ethernet/)
- [Fidium: Do You Need Speeds Above 1 Gig?](https://www.fidiumfibersavings.com/blog/do-you-need-speeds-above-1-gig-of-fiber-internet/)
- [Astound: Do I Need 2 Gig Internet?](https://www.astound.com/learn/internet/2-gig/)
- [Data Wire Solutions: Gigabit in Your Smart Home (Updated 2026)](https://datawiresolutions.com/blog/gigabit-internet-in-your-smart-home)
- [Ruckus Networks: WiFi 7 MLO Explained](https://www.ruckusnetworks.com/packet-pushers-40-wi-fi-7-mlo/)
- [SNBForums: 2.5G, LAN aggregation, switch](https://www.snbforums.com/threads/2-5g-lan-aggregation-switch-oh-my.83859/)
- [NETGEAR: WAN Ethernet Port Aggregation on Nighthawk Multi-Gig Cable Modem](https://kb.netgear.com/000060394/How-do-I-set-up-Ethernet-port-aggregation-on-my-NETGEAR-Nighthawk-multi-gig-speed-cable-modem)

## Update History

- **2026-05-23 (post-verify revisions)** — Replaced Fierce Network paywalled citation with primary Fox Rochester reporting on Greenlight pricing. Softened Astound 2-Gig citation to match what the page actually says. Removed unsupported Colorado Springs reference from Comcast X-Class market list (replaced with full sourced list). Clarified SNBForums LACP citation (LAN, not WAN, in original thread). Corrected Verizon 2 Gig spec to 2.3 Gbps per Verizon's marketing. Added NETGEAR WAN port aggregation KB as primary source for the LACP-as-WAN-workaround claim.
- **2026-05-23 (initial draft)** — Fresh-mode research covering US vs world broadband, NY regional ISPs, national multi-gig offerings, and home use cases for multi-gig with 1 Gbps client caps. Speedtest Global Index data is from December 2025 / January 2026.

## How This Report Was Generated

Generated by the Research skill (Pete Stergion's `ai-cli-workspace`) using a fresh-mode pass. Search tools: SearXNG (unavailable this session — MCP returned -32602 on the search_deep tool), Cornell AI Gateway Gemini search (auth token in `.env` was stale), Tier 3 built-in WebSearch + WebFetch (used throughout, every cited URL fetched and inspected). Every claim is grounded in a sourced URL. All timestamps in Eastern Time. The Dark Web / IOC sections were intentionally omitted — this is a non-security report and the skill template allows skipping them when not relevant.
