---
title: "Flare.io Dark Web Monitoring: Pricing, API, and Alternatives"
slug: flare-io-dark-web-monitoring-pricing
date: 2026-03-24
summary: "Flare.io starts at $18K/year for enterprise dark web monitoring with API access at the $28K+ tier. Lower-cost alternatives with API access include LeakCheck ($9.99/mo), Intelligence X ($2,500/yr), and Dehashed (credit-based). No individual/small team pricing exists for Flare."
---

## Findings

### 1. Flare.io Pricing Tiers

Flare's pricing was confirmed via their AWS Marketplace listing, which shows concrete dollar amounts on 12-month contracts [source: https://aws.amazon.com/marketplace/pp/prodview-rhfxxvfkojmms]:

| Tier | Annual Cost | Identifiers | Key Differentiators |
|------|------------|-------------|---------------------|
| **Starter** | $18,000/yr | 75 | Base package, all data sources, unlimited users, basic alert integrations |
| **Essentials** | $28,000/yr | 400 | Adds Platform API, basic alert integrations |
| **Core** | $57,000/yr | 800 | Adds Global Search, Threat Flow Explorer |
| **Enterprise** | $118,000/yr | 4,000 | Adds 500 takedowns, 10K/month Global Search API |

All tiers include: unlimited users, all data sources, multilingual support (EN/FR), in-app reporting, bi-annual intelligence review, Threat Flow generated intelligence, and customer success calls [source: https://aws.amazon.com/marketplace/pp/prodview-rhfxxvfkojmms].

Pricing is based on "identifiers" (the assets monitored: employee credentials, vendor accounts, domains, IP addresses, keywords), not per-seat [source: https://flare.io/flare-plans]. Additional identifiers and takedowns can be purchased a la carte.

**Verdict: Enterprise-only.** The lowest tier is $18,000/year. There is no individual, small team, or self-serve pricing. You must book a demo or go through AWS Marketplace.

### 2. Flare.io API Access

Flare offers a REST API, but it is **not included in the Starter tier**. API access begins at the Essentials tier ($28,000/yr) [source: https://aws.amazon.com/marketplace/pp/prodview-rhfxxvfkojmms].

The API supports:
- Integration with SIEM/SOAR (Splunk, Azure Sentinel)
- Messaging integrations (Slack, Microsoft Teams)
- Ticketing integrations (Jira, ServiceNow)
- Custom automations via REST API
- SDKs and documentation provided
[source: https://flare.io/platform]

The Enterprise tier ($118K) adds a "Global Search API" with 10K queries/month, which appears to be a more powerful programmatic search capability beyond basic alert/event integrations [source: https://aws.amazon.com/marketplace/pp/prodview-rhfxxvfkojmms].

**MCP server feasibility:** Yes, the REST API at the Essentials tier and above could theoretically be wrapped in an MCP server. However, at $28K/yr minimum for API access, this is cost-prohibitive for personal/small team use.

### 3. Alternatives with API Access at Lower Price Points

#### Intelligence X (intelx.io)
**High confidence** - pricing confirmed from product page.

| Tier | Annual Cost | Searches/Day | Notes |
|------|------------|-------------|-------|
| Free (registered) | $0 | 50 | Limited results |
| Researcher | ~$2,700/yr (EUR 2,500) | 200 | Individual use, Search API with fair-use limits |
| API | ~$7,560/yr (EUR 7,000) | 500 | Company integration/OEM, no Leaks API |
| Identity Portal | ~$10,800/yr (EUR 10,000) | 500 | Leaks API, reverse lookup, stealer log export |
| Enterprise | ~$21,600/yr (EUR 20,000) | 5,000+ | 5 users, full API, bulk operations |

[source: https://intelx.io/product]

Covers: dark web (Tor), paste sites, leaked databases, domain/WHOIS, public documents. Has a well-documented API with a public SDK on GitHub [source: https://github.com/IntelligenceX/SDK]. Free tier available for evaluation.

**MCP server feasibility:** Strong candidate. Public SDK, documented API, free tier for testing, Researcher tier at $2,500/yr for real use.

#### LeakCheck (leakcheck.io)
**High confidence** - pricing confirmed from their website.

| Tier | Cost | Lookups | Notes |
|------|------|---------|-------|
| Basic | $2.99/day | 15 emails/day | Day pass |
| Monthly | $9.99/mo | 200 emails+usernames/day, 15 keywords/day | Recommended |
| Lifetime | $69.99 one-time | 400 emails+usernames/day, 30 keywords/day | Best value |
| Enterprise | from $179/quarter | Unlimited lookups | Reverse search, bulk check, domain search |

[source: https://leakcheck.io]

All tiers include API access. Public API is free (1 req/sec). Pro API v2 starts at $9.99/mo (3 req/sec, upgradeable). 7+ billion records. Python wrapper and CLI tool available on GitHub [source: https://wiki.leakcheck.io/en/api, https://github.com/LeakCheck/leakcheck-api].

**MCP server feasibility:** Excellent candidate. Cheapest option with API, Python wrapper already exists, lifetime deal at $70 is remarkable.

#### Dehashed (dehashed.com)
**Medium confidence** - pricing is opaque, credit-based system.

- Starts at $0.02 per query [source: https://dehashed.com]
- Uses a credit-based system (500/5,000/10,000 credit packages available) [source: https://dehashed.com/api]
- Requires an active Search subscription to use API
- Free access for government agencies, non-profits, and NGOs
- 100 API credits cost approximately $3 (unverified, from third-party source)

The actual per-credit cost is not clearly published. Their API page shows $0.00 for credit packages, which appears to be a display issue [source: https://dehashed.com/api].

**MCP server feasibility:** Possible but frustrating. Credit-based pricing is hard to predict costs for. API requires separate subscription. Documentation is sparse.

#### SpiderFoot (Open Source) / SpiderFoot HX (Commercial)
**Medium confidence** - HX pricing not publicly available.

- **SpiderFoot Open Source**: Free, self-hosted, 100+ data source modules, includes TOR/dark web scanning, paste site checking, breach database lookups [source: https://github.com/smicallef/spiderfoot]
- **SpiderFoot HX**: Commercial cloud version with multi-user, API access, dashboards, reports. Pricing not publicly listed; requires contacting sales [source: https://spiderfoot.org]

SpiderFoot is more of an OSINT automation platform than a dark web monitoring service. Its dark web reach is "limited to what's publicly accessible" [source: https://deepstrike.io/blog/best-dark-web-monitoring-tools].

**MCP server feasibility:** The open-source version could be self-hosted and wrapped, but it's a full OSINT scanner, not a focused dark web API. Overkill for credential/leak monitoring.

#### Breachsense (breachsense.com)
**Low confidence** - pricing not publicly displayed.

- Four tiers based on monitored domains and API query volume
- All plans include: real-time dark web monitoring, credential detection, infostealer log monitoring, ransomware leak tracking, webhook/email alerts, full API access
- 7-day trial available after demo call
- Specific pricing requires contacting sales
[source: https://www.breachsense.com/pricing/]

**MCP server feasibility:** Looks promising feature-wise (full API on all plans), but unknown cost. Likely enterprise-oriented given the demo-call requirement.

### 4. Comparison Summary

| Service | Minimum Cost for API | Dark Web Depth | API Quality | Best For |
|---------|---------------------|----------------|-------------|----------|
| **Flare.io** | $28,000/yr | Deep (Tor, I2P, Telegram, forums) | REST API, SDKs | Enterprise SOC teams |
| **Intelligence X** | $2,700/yr (Researcher) | Moderate (Tor, paste, leaks) | Well-documented, SDK on GitHub | Researchers, small security teams |
| **LeakCheck** | $9.99/mo or $70 lifetime | Breach databases (7B+ records) | Pro API v2, Python wrapper | Credential monitoring, personal use |
| **Dehashed** | ~$0.02/query + subscription | Breach databases | Credit-based REST API | Ad-hoc lookups, OSINT |
| **SpiderFoot** | Free (self-hosted) | Limited (public dark web only) | Local API when self-hosted | OSINT automation, DIY |
| **Breachsense** | Unknown (sales call) | Deep (dark web, infostealers, ransomware) | Full API all plans | SMB/enterprise monitoring |

## Sources

1. [Flare AWS Marketplace Listing](https://aws.amazon.com/marketplace/pp/prodview-rhfxxvfkojmms) - Confirmed pricing tiers
2. [Flare Plans Page](https://flare.io/flare-plans) - Identifier-based pricing model
3. [Flare Platform Overview](https://flare.io/platform) - API and integration details
4. [Intelligence X Product Page](https://intelx.io/product) - Full pricing tiers confirmed
5. [Intelligence X SDK on GitHub](https://github.com/IntelligenceX/SDK) - Public API SDK
6. [LeakCheck Homepage](https://leakcheck.io) - Pricing plans confirmed
7. [LeakCheck API Documentation](https://wiki.leakcheck.io/en/api) - API details and rate limits
8. [LeakCheck API GitHub](https://github.com/LeakCheck/leakcheck-api) - Python wrapper
9. [Dehashed Homepage](https://dehashed.com) - Per-query pricing
10. [Dehashed API Page](https://dehashed.com/api) - Credit system details
11. [SpiderFoot GitHub](https://github.com/smicallef/spiderfoot) - Open source OSINT tool
12. [SpiderFoot Website](https://spiderfoot.org) - Commercial HX version
13. [Breachsense Pricing](https://www.breachsense.com/pricing/) - Tier structure (no prices)
14. [DeepStrike Dark Web Monitoring Tools](https://deepstrike.io/blog/best-dark-web-monitoring-tools) - Market comparison
15. [Datastreamer Dark Web APIs](https://datastreamer.io/2024s-best-dark-web-apis-feed-your-own-systems-with-darknet-data/) - API provider comparison

## Confidence & Gaps

### High Confidence
- Flare.io pricing tiers ($18K-$118K/yr) confirmed via AWS Marketplace listing
- Flare API requires Essentials tier ($28K/yr) minimum
- Flare is enterprise-only, no small team/individual pricing
- Intelligence X pricing tiers confirmed from their product page
- LeakCheck pricing confirmed from their website
- LeakCheck has the lowest barrier to entry for API access

### Medium Confidence
- Dehashed pricing at ~$0.02/query (stated on their site, but credit packages show $0.00 which may be a rendering issue)
- SpiderFoot HX exists as a commercial product but pricing is gated behind sales

### Gaps / Could Not Confirm
- Exact Dehashed credit package costs (their API page appears broken/dynamic)
- SpiderFoot HX specific pricing
- Breachsense specific pricing (requires demo call)
- Whether Flare offers any discounted pricing outside AWS Marketplace (partner/reseller deals mention up to 30% off list)
- Flare API documentation quality and endpoints (would need an account to evaluate)
