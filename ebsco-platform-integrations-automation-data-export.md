---
title: "EBSCO Platform Integrations, Automation & Data Export: A Technical Reference"
date: 2026-04-13
updated: 2026-04-13
status: complete
canary: r-eb5c0a72
summary: "EBSCO offers a layered ecosystem of APIs (EDS, HoldingsIQ, LinkIQ, PublicationIQ, Entitlement, SUSHI/COUNTER), admin reporting tools (EBSCOadmin), and analytics products (Usage Consolidation, EBSCONET Analytics, Panorama). Data can be exported in JSON, XML, TSV, Excel, and KBART formats. Automation is possible via SUSHI harvesting, scheduled EBSCOadmin reports, and direct API calls. Integration with BI tools like Power BI or Tableau requires an intermediary step: pulling data via API or export, then loading into the BI layer. FOLIO is the deepest integration point, with native eHoldings support via HoldingsIQ. Most APIs require subscription-level access and credentials obtained through EBSCO sales."
---

**TL;DR:** EBSCO provides multiple API families for search/discovery, knowledge base management, and usage statistics. The most automation-friendly paths are: (1) COUNTER_SUSHI API for automated usage harvesting in JSON, (2) HoldingsIQ API for programmatic access to the EBSCO Knowledge Base, and (3) EBSCOadmin scheduled reports for recurring Excel/TSV delivery via email. There is no direct BI connector (no native Tableau or Power BI plugin), so institutions typically land data in a database or data warehouse first, then connect BI tools to that layer. All APIs require EBSCO subscriptions and credentials obtained through your sales rep.

## Table of Contents

1. [Available APIs](#1-available-apis)
2. [Data Export Formats](#2-data-export-formats)
3. [Automation Options](#3-automation-options)
4. [Authentication Methods](#4-authentication-methods)
5. [BI and Analytics Compatibility](#5-bi-and-analytics-compatibility)
6. [FOLIO Integration](#6-folio-integration)
7. [Known Limitations](#7-known-limitations)
8. [Common Use Cases in Academic Libraries](#8-common-use-cases-in-academic-libraries)
9. [Recommended Architecture](#9-recommended-architecture)

## 1. Available APIs

EBSCO organizes its APIs into three families, all documented at the EBSCO Developer Network. [source: [developer.ebsco.com/home/docs/available-apis-and-requirements](https://developer.ebsco.com/home/docs/available-apis-and-requirements)]

### Search and Discovery APIs

| API | Purpose | Auth Method | Developer Account Required |
|-----|---------|-------------|---------------------------|
| **EDS API** | Integrate EBSCO Discovery Service search into websites/portals. Access an institution's entire collection through a single search box. | Username/password + session token (`x-authenticationToken`, `x-sessionToken`) | No |
| **EBSCOhost Entitlement API** | Customize the EBSCO open link resolver; access full-text content from EBSCOhost databases. | OAuth2 (Consumer Key/Secret) | Yes |
| **BiblioGraph API** | Linked data and bibliographic graph access. | OAuth2 | Yes |

[source: [developer.ebsco.com/home/page/search-discovery](https://developer.ebsco.com/home/page/search-discovery)]

The EDS API is RESTful and returns JSON or XML. It supports standard HTTP methods (GET, POST, PUT, DELETE) and requires an active EDS subscription. [source: [developer.ebsco.com/eds-api/docs/introduction](https://developer.ebsco.com/eds-api/docs/introduction)]

### EBSCO Knowledge Services APIs

| API | Purpose | Auth Method | Developer Account Required |
|-----|---------|-------------|---------------------------|
| **HoldingsIQ** | Access EBSCO Knowledge Base data, holdings data, and HLM services. Integrates with FOLIO and other library systems. | API Key (`x-api-key` header) | No |
| **LinkIQ** | Customize the EBSCO open link resolver for patron access. | Customer profile/password | No |
| **PublicationIQ** | Search publications designated as part of a customer's collection. | Customer profile/password | No |

[source: [developer.ebsco.com/knowledge-services/holdingsiq](https://developer.ebsco.com/knowledge-services/holdingsiq)]

HoldingsIQ is the most important API for electronic resource management. It exposes vendor data, package data, and title-level information from the EBSCO Knowledge Base. Documentation includes sections on environment/rate limits, authentication, search types, and holdings data downloading. A subscription is required. [source: [developer.ebsco.com/knowledge-services/docs/holdingsiq-overview](https://developer.ebsco.com/knowledge-services/docs/holdingsiq-overview)]

### Medical Point of Care APIs

Six MedsAPI endpoints (DynaMed, Dynamic Health, DynaMed Decisions, Consumer Health, NAH Reference Center, Product Content) provide clinical decision support content. All require a developer account and OAuth2 authentication. [source: [developer.ebsco.com/home/docs/available-apis-and-requirements](https://developer.ebsco.com/home/docs/available-apis-and-requirements)]

### COUNTER_SUSHI API (Usage Statistics)

EBSCO provides a RESTful COUNTER_SUSHI API for automated harvesting of COUNTER Release 5 usage reports in JSON format. This is distinct from the APIs above and is accessed through a SUSHI endpoint. Key details:

- Returns COUNTER R5 reports in JSON format
- Designed for automatic harvesting by ERM systems (Alma, FOLIO, etc.)
- Uses the NISO SUSHI protocol (Standardized Usage Statistics Harvesting Initiative)
- Supports all standard COUNTER R5 report types (TR, DR, PR, IR)
- SUSHI servers must respond within 120 seconds per the COUNTER specification

[source: [connect.ebsco.com/s/article/EBSCOhost-SUSHI-Web-Service-FAQs](https://connect.ebsco.com/s/article/EBSCOhost-SUSHI-Web-Service-FAQs)]

**Confidence: High.** All API details sourced directly from EBSCO's developer documentation and support articles.

## 2. Data Export Formats

EBSCO supports multiple export formats across different tools and APIs:

| Format | Where Available | Use Case |
|--------|----------------|----------|
| **JSON** | EDS API, COUNTER_SUSHI API, HoldingsIQ API | Primary API response format; machine-readable usage data |
| **XML** | EDS API (legacy), older SUSHI implementations | Legacy integrations |
| **TSV (Tab-Separated Values)** | Usage Consolidation, COUNTER R5 reports, EBSCOadmin | Preferred format for loading COUNTER R5 data into Usage Consolidation |
| **Excel (.xlsx)** | EBSCOadmin reports, Usage Consolidation | Manual analysis, ad-hoc reporting |
| **KBART** | Holdings Management, Knowledge Base exports | Title list exchange between systems; NISO standard format |
| **EDIFACT** | FOLIO acquisitions integration | Purchase order and invoice exchange with financial systems |
| **Tab-delimited text** | EBSCOadmin scheduled reports | Database-loadable format for recurring delivery |

**COUNTER R5 Specifics:** Per the COUNTER Code of Practice, reports must be delivered in both JSON (via SUSHI API) and tabular form (TSV or Excel). EBSCO's free COUNTER R5 tool saves reports "either in COUNTER R5 tabular form or as a delimited text file suitable for loading into a database." [source: [about.ebsco.com/news-center/press-releases/ebsco-develops-free-tool-assist-librarians-counter-r5](https://about.ebsco.com/news-center/press-releases/ebsco-develops-free-tool-assist-librarians-counter-r5)]

**KBART Exports:** Holdings can be downloaded from EBSCO Holdings Management in KBART-compatible tab-delimited format. EBSCO also publishes KBART-formatted title lists monthly through the NISO KBART Registry. [source: [connect.ebsco.com/s/article/Holdings-Management-Downloading-Your-Holdings](https://connect.ebsco.com/s/article/Holdings-Management-Downloading-Your-Holdings)] [source: [sites.google.com/niso.org/kbart-registry](https://sites.google.com/niso.org/kbart-registry)]

**Confidence: High.** Format details confirmed across multiple EBSCO documentation pages and the COUNTER specification.

## 3. Automation Options

### SUSHI Automated Harvesting

The most robust automation path for usage data. EBSCO's COUNTER_SUSHI API enables:

- **Scheduled harvesting** by ERM systems (Alma, FOLIO, Koha, etc.) that have built-in SUSHI clients
- **Programmatic retrieval** of COUNTER R5 reports (TR, DR, PR, IR) in JSON
- No manual download required once credentials are configured
- Library systems like Alma support scheduled SUSHI harvesting jobs via configuration

[source: [connect.ebsco.com/s/article/EBSCOhost-SUSHI-Web-Service-FAQs](https://connect.ebsco.com/s/article/EBSCOhost-SUSHI-Web-Service-FAQs)]

### EBSCOadmin Scheduled Reports

EBSCOadmin provides built-in report scheduling:

- **Delivery options:** Download from the Download Reports tab, or email delivery
- **Email scheduling:** One-time or scheduled monthly delivery
- **Formats:** MS Excel or tab-delimited
- **Report types:** Standard usage reports (sessions, searches, full-text requests), COUNTER-compliant reports, database-level statistics

Reports can be scheduled to arrive automatically at the beginning of each month for the prior reporting period. [source: [connect.ebsco.com/s/article/EBSCOadmin-Retrieving-Statistics](https://connect.ebsco.com/s/article/EBSCOadmin-Retrieving-Statistics)]

### HoldingsIQ API for Holdings Automation

HoldingsIQ supports programmatic access to holdings data:

- Query vendor, package, and title information
- Download holdings data in bulk
- Integrate with automated workflows in FOLIO or custom scripts

[source: [developer.ebsco.com/knowledge-services/docs/holdingsiq-overview](https://developer.ebsco.com/knowledge-services/docs/holdingsiq-overview)]

### EDS API for Search Integration

The EDS API can be called programmatically to:

- Run automated searches and retrieve results
- Build custom search interfaces
- Integrate discovery into institutional portals or intranets

[source: [developer.ebsco.com/eds-api/docs/introduction](https://developer.ebsco.com/eds-api/docs/introduction)]

### What EBSCO Does NOT Offer (as of April 2026)

- **No SFTP delivery** of reports or data exports (email and web download only from EBSCOadmin)
- **No webhook/event-driven notifications** for data changes
- **No direct database connector** for BI tools

**Confidence: High** for what exists. **Medium confidence** on the "does not offer" items, as EBSCO may have features not publicly documented. If SFTP delivery exists, it would likely be through a custom arrangement with EBSCO support.

## 4. Authentication Methods

EBSCO uses different authentication mechanisms depending on the API:

| Method | Used By | Details |
|--------|---------|---------|
| **API Key** (`x-api-key` header) | HoldingsIQ | Obtained through EBSCO sales/support; tied to subscription |
| **Session-based auth** (`x-authenticationToken` + `x-sessionToken`) | EDS API | Authenticate first, receive tokens, include in subsequent requests |
| **OAuth2** (Consumer Key/Secret) | Entitlement API, BiblioGraph, MedsAPI family | Register at developer.ebsco.com; receive client credentials |
| **Customer profile/password** | LinkIQ, PublicationIQ | Use existing EBSCO account credentials |
| **SUSHI credentials** (Customer ID, Requestor ID, API Key) | COUNTER_SUSHI API | Configured per institution in EBSCOadmin |
| **IP-based authentication** | EBSCOhost web access (not API) | Standard for patron access to databases; works with EZproxy, OpenAthens |

**API Key Request Process:**
1. Register at the EBSCO Developer portal
2. Complete the application form (organization details, ILS/ERM info)
3. An EBSCO representative contacts you within 5 business days with credentials

[source: [developer.ebsco.com/request-api-key-information](https://developer.ebsco.com/request-api-key-information)]

**Patron Authentication:** EBSCO supports IP-based authentication, EZproxy, OpenAthens, and SAML/Shibboleth for end-user access to databases. The FOLIO integration supports "seamless integration with OpenAthens." These are for database access, not API access. [source: [about.ebsco.com/products/ebsco-folio-integrations](https://about.ebsco.com/products/ebsco-folio-integrations)]

**Confidence: High.** Authentication details sourced directly from EBSCO developer documentation.

## 5. BI and Analytics Compatibility

### Direct EBSCO Analytics Products

EBSCO offers several built-in analytics tools:

- **EBSCO Usage Consolidation:** Centralized usage data from EBSCO and non-EBSCO vendors. Supports cost-per-use analysis, reports by title/publisher/platform/database. Can export data for external analysis. [source: [about.ebsco.com/products/ebsco-usage-consolidation](https://about.ebsco.com/products/ebsco-usage-consolidation)]
- **EBSCONET Analytics:** Dashboard with usage statistics for e-journals and e-packages ordered through EBSCO. [source: [about.ebsco.com/corporations/products/journal-subscription-services/usage-analytics-tools](https://about.ebsco.com/corporations/products/journal-subscription-services/usage-analytics-tools)]
- **Panorama (FOLIO):** Reporting module within FOLIO covering acquisitions, circulation, inventory, and COUNTER usage data. [source: [about.ebsco.com/products/ebsco-folio-integrations](https://about.ebsco.com/products/ebsco-folio-integrations)]

### Connecting to Tableau, Power BI, or Excel

There is no native EBSCO connector for Tableau or Power BI. The typical architecture involves:

1. **Extract:** Pull data via COUNTER_SUSHI API (JSON), HoldingsIQ API (JSON), or EBSCOadmin exports (Excel/TSV)
2. **Load:** Land data into a database (PostgreSQL, SQL Server, etc.) or data warehouse
3. **Connect:** Point Power BI or Tableau at the database layer

For simpler needs, Excel exports from EBSCOadmin or Usage Consolidation can be opened directly in Power BI or Tableau.

**Practical pattern for recurring dashboards:**
- Schedule monthly EBSCOadmin reports via email (Excel or tab-delimited)
- Write a script to parse incoming files and load into a staging database
- Alternatively, write a SUSHI client script that pulls COUNTER data monthly and inserts into a database
- Connect Power BI/Tableau to the database for visualization

**Confidence: Medium.** The absence of a native BI connector is inferred from the lack of any mention in EBSCO documentation. EBSCO may offer custom integration services not publicly documented. The ETL pattern described is standard practice in academic libraries based on search results.

## 6. FOLIO Integration

FOLIO is EBSCO's deepest integration point, with native support for the EBSCO Knowledge Base.

### eHoldings App

The FOLIO eHoldings app connects directly to EBSCO's Knowledge Base via the HoldingsIQ API. Configuration requires:

- **EBSCO KB API credentials** (API key entered in FOLIO settings)
- **Root proxy server** selection
- **Usage Consolidation credentials** (Usage Consolidation ID, Client ID, API Key) for institutions subscribing to that service

[source: [docs.folio.org/docs/settings/settings_eholdings/settings_eholdings/](https://docs.folio.org/docs/settings/settings_eholdings/settings_eholdings/)]

### Broader FOLIO Integrations

FOLIO connects with institutional systems including:

- **Student Information Systems (SIS):** Patron data import
- **Financial/ERP Systems:** Fees, fines, purchase orders, invoices (EDIFACT standard)
- **Authentication:** OpenAthens, SAML, institutional SSO
- **ILL/Resource Sharing:** Standard interlibrary loan platforms
- **Discovery:** EBSCO Discovery Service, Full Text Finder OpenURL link resolver
- **Acquisitions:** GOBI and EBSCONET ordering platforms

[source: [about.ebsco.com/products/ebsco-folio-integrations](https://about.ebsco.com/products/ebsco-folio-integrations)]

### Knowledge Base Options

FOLIO supports two knowledge base sources:
- **EBSCO Knowledge Base** (via HoldingsIQ API, the most common)
- **GOKb** (Global Open Knowledgebase, open-source alternative via Local KB Admin app)

[source: [folio-org.atlassian.net/wiki/spaces/SYSOPS/pages/2097208](https://folio-org.atlassian.net/wiki/spaces/SYSOPS/pages/2097208)]

**Confidence: High.** FOLIO integration details confirmed across FOLIO documentation and EBSCO product pages.

## 7. Known Limitations

### Rate Limits

- HoldingsIQ documentation includes a section on "Environment and rate limits" but specific numbers are not publicly documented. Rate limits are likely communicated with API credentials. [source: [developer.ebsco.com/knowledge-services/docs/holdingsiq-overview](https://developer.ebsco.com/knowledge-services/docs/holdingsiq-overview)]
- COUNTER_SUSHI servers must respond within 120 seconds per the COUNTER specification. Servers that cannot meet this must adopt background processing architecture. [source: [projectcounter.org SUSHI specification](https://www.projectcounter.org/wp-content/themes/project-counter-2016/pdfs/COUNTER-code-of-practice-sushi.pdf)]
- No publicly documented rate limits for EDS API or Entitlement API.

### Licensing Constraints

- **All APIs require active subscriptions** to the corresponding EBSCO product. You cannot access EDS API without an EDS subscription, HoldingsIQ without a KB subscription, etc.
- API credentials are tied to institutional subscriptions, not individual developers.
- EBSCO representative approval is required for production API access (5 business day turnaround).
- Medical APIs (MedsAPI family) require both a developer account and product subscription.

### Technical Limitations

- **No real-time streaming.** All APIs are request/response; no WebSocket or event-driven options.
- **No bulk export API** for search results (EDS API is designed for interactive search, not data mining).
- **Usage Consolidation** requires COUNTER R5 formatted data as TSV (preferred) for upload. Non-COUNTER data requires manual formatting.
- **EBSCOadmin reports** are limited to email delivery or web download. No SFTP, S3, or webhook delivery.
- **Holdings downloads** from HLM are manual via the EBSCO website unless using HoldingsIQ API.

**Confidence: Medium.** Limitations are inferred from documented capabilities. Some limitations may have been addressed in features not publicly documented.

## 8. Common Use Cases in Academic Libraries

### Electronic Resource Management (ERM)

- Use HoldingsIQ to sync holdings data between EBSCO KB and the local ILS/LSP (FOLIO, Alma, etc.)
- Manage activation/deactivation of packages and titles programmatically
- Download KBART title lists for catalog overlay or link resolver maintenance

### Usage Reporting and Collections Analysis

- Harvest COUNTER R5 data via SUSHI for all EBSCO databases
- Load data into Usage Consolidation alongside non-EBSCO vendor stats
- Generate cost-per-use reports to inform renewal decisions
- Schedule monthly reports from EBSCOadmin for ongoing monitoring

### Discovery Integration

- Embed EDS API search into library website, LMS (Canvas, Blackboard), or institutional portal
- Build custom search interfaces with EDS API for specialized user populations
- Use Entitlement API to customize link resolver behavior

### Dashboard and Reporting Pipelines

Institutions typically operationalize EBSCO data for dashboards by:

1. **Monthly SUSHI harvest** of COUNTER data into a local database (often automated by the ILS)
2. **Quarterly HoldingsIQ pulls** to refresh collection metadata
3. **Annual cost-per-use analysis** combining Usage Consolidation exports with financial data
4. **BI layer** (Power BI, Tableau, or even Google Sheets) connected to the local database

Chalmers University of Technology was the first institution to go live with FOLIO hosted by EBSCO, demonstrating the full integration stack. [source: [librarytechnology.org/pr/24658](https://librarytechnology.org/pr/24658)]

## 9. Recommended Architecture

For an institution wanting to maximize automation and analytics from EBSCO data:

```
+-------------------+     +------------------+     +------------------+
| EBSCO APIs        |     | Staging Layer    |     | Analytics Layer  |
|                   |     |                  |     |                  |
| SUSHI/COUNTER ----+---->| Local Database   +---->| Power BI         |
| HoldingsIQ -------+---->| (PostgreSQL,     |     | Tableau          |
| EDS API ----------+---->|  SQL Server)     |     | Excel            |
| EBSCOadmin -------+---->|                  |     | Custom Dashboards|
| (email exports)   |     +------------------+     +------------------+
+-------------------+
```

**Key components:**

1. **SUSHI client** (built into FOLIO/Alma, or a custom script using Python + requests) pulls COUNTER data monthly
2. **HoldingsIQ integration** via FOLIO eHoldings or custom scripts for collections metadata
3. **ETL process** (cron job, n8n workflow, or similar) that processes EBSCOadmin email exports
4. **Database** as the single source of truth for all EBSCO data
5. **BI tool** connecting to the database for visualization

For institutions using FOLIO, much of this is built in. Panorama provides native analytics, and the eHoldings app handles KB integration. For institutions on other systems, the intermediary database pattern is standard.

---

## Confidence Assessment

| Section | Confidence | Notes |
|---------|-----------|-------|
| Available APIs | High | Sourced directly from developer.ebsco.com |
| Data Export Formats | High | Confirmed across EBSCO docs and COUNTER spec |
| Automation Options | High | SUSHI and EBSCOadmin scheduling well-documented |
| Authentication Methods | High | Developer portal documentation |
| BI Compatibility | Medium | No native connector confirmed; ETL pattern is common practice |
| FOLIO Integration | High | FOLIO docs and EBSCO product pages |
| Known Limitations | Medium | Inferred from documented capabilities |
| Use Cases | Medium | Based on library technology literature and EBSCO marketing |

## Open Questions

1. **Specific rate limits for HoldingsIQ and EDS API.** The documentation references rate limits but does not publish numbers. These are likely communicated with API credentials.
2. **SFTP delivery options.** EBSCO may offer SFTP or other automated file delivery for enterprise customers through custom arrangements not publicly documented.
3. **EBSCONET Analytics API access.** It is unclear whether EBSCONET Analytics data can be accessed programmatically or only through the web dashboard.
4. **Sandbox/testing environments.** Developer documentation references sandbox access but details on availability and limitations are gated behind the registration process.
5. **COUNTER R5.1 support timeline.** EBSCO's adoption timeline for COUNTER R5.1 (which uses OpenAPI 3.1 for the SUSHI specification) is not publicly documented.

## Sources

1. EBSCO Developer Network, "Available APIs and Requirements" - [developer.ebsco.com/home/docs/available-apis-and-requirements](https://developer.ebsco.com/home/docs/available-apis-and-requirements)
2. EBSCO Developer Network, "Introduction - Getting Started" - [developer.ebsco.com/home/docs/introduction](https://developer.ebsco.com/home/docs/introduction)
3. EBSCO Developer Network, "HoldingsIQ Overview" - [developer.ebsco.com/knowledge-services/docs/holdingsiq-overview](https://developer.ebsco.com/knowledge-services/docs/holdingsiq-overview)
4. EBSCO Developer Network, "Search & Discovery" - [developer.ebsco.com/home/page/search-discovery](https://developer.ebsco.com/home/page/search-discovery)
5. EBSCO Developer Network, "Request API Key Information" - [developer.ebsco.com/request-api-key-information](https://developer.ebsco.com/request-api-key-information)
6. EBSCO Developer Network, "EDS API Introduction" - [developer.ebsco.com/eds-api/docs/introduction](https://developer.ebsco.com/eds-api/docs/introduction)
7. EBSCO Connect, "EBSCOhost SUSHI Web Service FAQs" - [connect.ebsco.com/s/article/EBSCOhost-SUSHI-Web-Service-FAQs](https://connect.ebsco.com/s/article/EBSCOhost-SUSHI-Web-Service-FAQs)
8. EBSCO Connect, "EBSCOadmin Retrieving Statistics" - [connect.ebsco.com/s/article/EBSCOadmin-Retrieving-Statistics](https://connect.ebsco.com/s/article/EBSCOadmin-Retrieving-Statistics)
9. EBSCO Connect, "EBSCOadmin Reports & Statistics Best Practices" - [connect.ebsco.com/s/article/EBSCOadmin-Reports-Statistics-Best-Practices](https://connect.ebsco.com/s/article/EBSCOadmin-Reports-Statistics-Best-Practices)
10. EBSCO Connect, "Usage Consolidation COUNTER Release 5 FAQ" - [connect.ebsco.com/s/article/Usage-Consolidation-COUNTER-Release-5-FAQ](https://connect.ebsco.com/s/article/Usage-Consolidation-COUNTER-Release-5-FAQ)
11. EBSCO Connect, "Holdings Management - Downloading Your Holdings" - [connect.ebsco.com/s/article/Holdings-Management-Downloading-Your-Holdings](https://connect.ebsco.com/s/article/Holdings-Management-Downloading-Your-Holdings)
12. EBSCO Products, "EBSCO Usage Consolidation" - [about.ebsco.com/products/ebsco-usage-consolidation](https://about.ebsco.com/products/ebsco-usage-consolidation)
13. EBSCO Products, "EBSCO FOLIO Integrations" - [about.ebsco.com/products/ebsco-folio-integrations](https://about.ebsco.com/products/ebsco-folio-integrations)
14. EBSCO Products, "EBSCONET Analytics & Usage Tools" - [about.ebsco.com/corporations/products/journal-subscription-services/usage-analytics-tools](https://about.ebsco.com/corporations/products/journal-subscription-services/usage-analytics-tools)
15. FOLIO Documentation, "Settings > eHoldings" - [docs.folio.org/docs/settings/settings_eholdings/settings_eholdings/](https://docs.folio.org/docs/settings/settings_eholdings/settings_eholdings/)
16. FOLIO Wiki, "List of Integrations" - [folio-org.atlassian.net/wiki/spaces/SYSOPS/pages/2097208](https://folio-org.atlassian.net/wiki/spaces/SYSOPS/pages/2097208)
17. COUNTER Code of Practice R5, "Formats for COUNTER Reports" - [cop5.projectcounter.org/en/5.1/03-specifications/02-formats-for-counter-reports.html](https://cop5.projectcounter.org/en/5.1/03-specifications/02-formats-for-counter-reports.html)
18. COUNTER Code of Practice R5, "SUSHI for Automated Report Harvesting" - [cop5.projectcounter.org/en/5.1/08-sushi/index.html](https://cop5.projectcounter.org/en/5.1/08-sushi/index.html)
19. EBSCO Press Release, "EBSCO Develops Free Tool to Assist Librarians with COUNTER R5" - [about.ebsco.com/news-center/press-releases/ebsco-develops-free-tool-assist-librarians-counter-r5](https://about.ebsco.com/news-center/press-releases/ebsco-develops-free-tool-assist-librarians-counter-r5)
20. Library Technology Guides, "Chalmers University of Technology goes live with FOLIO" - [librarytechnology.org/pr/24658](https://librarytechnology.org/pr/24658)
21. NISO KBART Registry - [sites.google.com/niso.org/kbart-registry](https://sites.google.com/niso.org/kbart-registry)
22. EBSCOhost Integration Toolkit Support Center - [support.ebsco.com/eit/api.php](https://support.ebsco.com/eit/api.php)

## Update History

| Date | Change |
|------|--------|
| 2026-04-13 | Initial research and publication |

## How This Report Was Generated

Researched on April 13, 2026 (ET) using SearXNG deep search across multiple engines, with direct page fetches of EBSCO developer documentation, EBSCO Connect support articles, FOLIO documentation, and COUNTER specification pages. All factual claims are grounded in source material. Areas of uncertainty are flagged with confidence levels. No EBSCO API calls were made during research; all information comes from publicly accessible documentation.
