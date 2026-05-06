---
title: "Canvas (Instructure) Security Breach: May 2026"
date: 2026-05-05
updated: 2026-05-06T12:38:41-04:00
summary: "Instructure disclosed a cybersecurity incident on May 1, 2026 affecting its Canvas LMS. ShinyHunters claims 275M users across ~9,000 schools were impacted; Instructure confirms exposure of names, emails, student IDs, and user-to-user messages, but no passwords or financial data."
---

## Current Status

- Instructure publicly disclosed the incident on May 1, 2026 and stated containment by May 2, 2026 [source: https://www.bitdefender.com/en-us/blog/hotforsecurity/canvas-data-breach-2026]
- Confirmed exposed data: names, email addresses, student ID numbers, and messages exchanged among users; some reporting also references phone numbers [source: https://www.bleepingcomputer.com/news/security/instructure-confirms-data-breach-shinyhunters-claims-attack/] [source: https://www.insidehighered.com/news/tech-innovation/administrative-tech/2026/05/05/pay-or-leak-hackers-target-big-higher-ed-vendor]
- Instructure says no evidence that passwords, dates of birth, government identifiers, or financial information were involved [source: https://www.bitdefender.com/en-us/blog/hotforsecurity/canvas-data-breach-2026]
- ShinyHunters listed Instructure on its leak site on May 3, 2026, claiming 3.65 TB of data covering ~9,000 schools and 275 million individuals; these numbers are unverified by Instructure [source: https://www.bleepingcomputer.com/news/security/instructure-confirms-data-breach-shinyhunters-claims-attack/]
- ShinyHunters' May 6, 2026 ransom deadline has arrived; as of midday ET no public bulk dump has been observed and Instructure has not commented on negotiations [source: https://www.malwarebytes.com/blog/news/2026/05/millions-of-students-personal-data-stolen-in-major-education-cyberattack]
- Canvas Data 2 and Canvas Beta restored to all customers as of Instructure's May 5 update; no fresh status-page entry has been posted since [source: https://status.instructure.com/]
- Instructure rotated application keys / OAuth tokens as a precaution, requiring customers to re-authorize integrations using newly issued, timestamp-named keys [source: https://www.bleepingcomputer.com/news/security/instructure-confirms-data-breach-shinyhunters-claims-attack/]
- Initial attack vector has not been disclosed by Instructure; no CVE has been assigned and no IOCs published as of midday May 6, 2026 [source: https://gbhackers.com/canvas-confirms-data-breach-following-shinyhunters-claim/]
- International disclosures emerging: University of Auckland (NZ) posted its first advisory on May 6, joining U.S. campuses (Rutgers, UT Austin, Boise State, UWM) [source: https://www.auckland.ac.nz/en/news/notices/2026/cybersecurity-incident-involving-canvas.html]

## Table of Contents

- [Current Status](#current-status)
- [Findings](#findings)
  - [1. What Happened](#1-what-happened)
  - [2. Technical Analysis](#2-technical-analysis)
  - [3. Timeline](#3-timeline)
  - [4. Discovery and Disclosure](#4-discovery-and-disclosure)
  - [5. Vendor Response](#5-vendor-response)
  - [6. Higher-Ed Sector Response](#6-higher-ed-sector-response)
  - [7. Threat Actor Context: ShinyHunters](#7-threat-actor-context-shinyhunters)
  - [8. Prior Incident: September 2025 Salesforce Breach](#8-prior-incident-september-2025-salesforce-breach)
- [Indicators of Compromise](#indicators-of-compromise)
- [Recommended Actions for Higher-Ed IT Admins](#recommended-actions-for-higher-ed-it-admins)
- [Confidence Assessment](#confidence-assessment)
- [Open Questions](#open-questions)
- [Sources](#sources)
- [Update History](#update-history)
- [How This Report Was Generated](#how-this-report-was-generated)

## Findings

### 1. What Happened

On May 1, 2026, Instructure (parent of the Canvas LMS) disclosed a cybersecurity incident. CISO Steve Proud said the company "recently experienced a cybersecurity incident perpetrated by a criminal threat actor" and engaged outside forensics experts [source: https://www.bleepingcomputer.com/news/security/edu-tech-firm-instructure-discloses-cyber-incident-probes-impact/].

In a follow-up update the next day, Instructure confirmed that exposed information included "certain identifying information of users at affected institutions, such as names, email addresses, and student ID numbers, as well as messages among users." The company stated: "At this time, we have found no evidence that passwords, dates of birth, government identifiers, or financial information were involved" [source: https://status.instructure.com/].

On May 3, 2026, the extortion group ShinyHunters listed Instructure on its leak site, alleging theft of 3.65 TB of data covering 275 million students, teachers, and staff at nearly 9,000 schools. The group's posting also referenced compromise of an associated Salesforce instance and "several billions of private messages among students and teachers" [source: https://www.bleepingcomputer.com/news/security/instructure-confirms-data-breach-shinyhunters-claims-attack/]. Instructure has not confirmed those numbers [source: https://www.k12dive.com/news/instructure-confirms-cybersecurity-incident/819362/].

Confidence: High that Instructure confirmed a breach with the listed data categories. Medium confidence on the 275M / 9,000 schools figures (attacker-supplied, unverified by Instructure or independent investigators).

### 2. Technical Analysis

The initial access vector has not been publicly disclosed by Instructure. ShinyHunters claimed in their leak-site posting that data was stolen "via a vulnerability in their systems, which has now been patched" [source: https://www.bleepingcomputer.com/news/security/instructure-confirms-data-breach-shinyhunters-claims-attack/]. One analysis attributes the entry to "Salesforce misconfiguration," consistent with the broader 2025-2026 ShinyHunters campaign against Salesforce-tenant customers [source: https://www.securitymagazine.com/articles/102283-instructure-parent-of-canvas-confirms-data-breach]. This attribution is the publication's interpretation, not an Instructure confirmation.

The disruption pattern observed by customers points to credential and token compromise affecting platforms that consume Canvas APIs:

- Canvas Data 2 (analytics export pipeline) and Canvas Beta were placed under maintenance starting May 1, causing downstream tools that depend on API keys to fail [source: https://gbhackers.com/canvas-confirms-data-breach-following-shinyhunters-claim/]
- Instructure "revoked privileged credentials and access tokens related to the affected systems" and "rotated certain keys out of an abundance of caution" [source: https://status.instructure.com/]
- New application keys carry a timestamp in their naming convention so customers can distinguish reissued, valid keys from older ones during re-authorization [source: https://www.bleepingcomputer.com/news/security/instructure-confirms-data-breach-shinyhunters-claims-attack/]

No CVE has been assigned. No IOCs (IPs, hashes, domains) have been published by Instructure or any government CERT as of May 5, 2026 [source: https://gbhackers.com/canvas-confirms-data-breach-following-shinyhunters-claim/].

Confidence: High that key/token rotation occurred and that Canvas Data 2 / Beta were affected. Low confidence on the precise initial access vector (Salesforce vs. direct Instructure systems).

### 3. Timeline

Reverse chronological. All times Eastern unless otherwise noted.

- **May 6, 2026 (midday ET)** - ShinyHunters ransom deadline arrives. No public bulk leak observed as of this update. University of Auckland (NZ) posts its first cybersecurity advisory, becoming the first international institution to publicly comment [source: https://www.auckland.ac.nz/en/news/notices/2026/cybersecurity-incident-involving-canvas.html]
- **May 5, 2026** - Canvas Data 2 and Beta services restored to most customers; Test environment still under maintenance for some. UWM posts advisory; Inside Higher Ed publishes "Pay or Leak" feature with sector-wide commentary [source: https://thecyberexpress.com/canvas-cybersecurity-incident/] [source: https://uwm.edu/information-technology/uwm-monitoring-nationwide-canvas-security-breach/] [source: https://www.insidehighered.com/news/tech-innovation/administrative-tech/2026/05/05/pay-or-leak-hackers-target-big-higher-ed-vendor]
- **May 4, 2026** - Multiple universities (Rutgers, UT Austin, Boise State) issue advisories to their campuses [source: https://it.rutgers.edu/alerts/2026/05/04/nationwide-security-breach-involving-canvas/]
- **May 3, 2026 (Sunday)** - ShinyHunters lists Instructure on its leak site claiming 3.65 TB / 275M users / ~9,000 schools, with a leak deadline of May 6, 2026. Canvas Data 2 access restored [source: https://securityaffairs.com/191686/cyber-crime/educational-tech-firm-instructure-data-breach-may-have-impacted-9000-schools.html]
- **May 2, 2026 (Saturday)** - Instructure issues update confirming exposure of names, emails, student IDs, and messages; says incident is contained [source: https://www.bitdefender.com/en-us/blog/hotforsecurity/canvas-data-breach-2026]
- **May 1, 2026 (Friday, ~4:30 PM Mountain Time)** - Instructure publicly discloses the incident; CISO Steve Proud's statement posted to status page; Canvas Data 2 and Beta placed under maintenance [source: https://www.gblock.app/articles/instructure-canvas-may-2026-breach-student-data]
- **April 30, 2026** - Disruptions detected affecting tools that rely on Canvas API keys; investigation begins with outside forensics experts [source: https://gbhackers.com/canvas-confirms-data-breach-following-shinyhunters-claim/]

Confidence: High on the public-disclosure dates. Medium on April 30 detection date (sourced to gbhackers; not directly stated by Instructure).

### 4. Discovery and Disclosure

Instructure has not publicly stated how the intrusion was first detected. Public reporting indicates that disruptions to API key-dependent tools were observed on April 30, 2026 [source: https://gbhackers.com/canvas-confirms-data-breach-following-shinyhunters-claim/]. The company's initial public statement on May 1 came via its corporate status page (status.instructure.com) and noted only that the incident was "perpetrated by a criminal threat actor" and that outside forensics experts had been engaged [source: https://status.instructure.com/].

Disclosure came one day before ShinyHunters listed the company publicly on May 3, suggesting Instructure may have received an extortion notice prior to public disclosure, though this has not been confirmed by the company [source: https://www.bleepingcomputer.com/news/security/instructure-confirms-data-breach-shinyhunters-claims-attack/].

Confidence: High that Instructure disclosed via its status page on May 1. Low on the prior extortion-notice hypothesis.

### 5. Vendor Response

Per Instructure's own status page and CISO Steve Proud's statement, the company has taken the following actions [source: https://status.instructure.com/]:

- Revoked privileged credentials and access tokens for affected systems
- Deployed security patches
- Rotated certain cryptographic keys "out of an abundance of caution"
- Implemented increased monitoring across platforms
- Engaged outside forensics experts and notified law enforcement
- Reissued application keys, requiring customers to re-authorize affected integrations

Instructure stated in its May 2 update: "While our investigation continues alongside our outside forensics experts, at this stage we believe the incident has been contained" [source: https://www.bitdefender.com/en-us/blog/hotforsecurity/canvas-data-breach-2026].

Direct quote on transparency from Steve Proud: "Maintaining your trust is our highest priority, and we are committed to transparency throughout this process" [source: https://www.bleepingcomputer.com/news/security/edu-tech-firm-instructure-discloses-cyber-incident-probes-impact/].

Instructure declined to answer detailed press questions about scope, timing, extortion demands, or affected institution counts; spokesperson Kate Holmes directed inquiries to the status page [source: https://techcrunch.com/2026/05/05/hackers-steal-students-data-during-breach-at-education-tech-giant-instructure/].

Confidence: High.

### 6. Higher-Ed Sector Response

A sample of public university advisories as of May 4-5, 2026:

- **Rutgers University** (May 4): "A nationwide security incident involving Canvas, the university's learning management system... a vendor-driven, nationwide event affecting multiple institutions. Rutgers has not been notified of any direct impact to our campus." Canvas remains operational. No specific user actions advised beyond contacting the OIT Help Desk for issues [source: https://it.rutgers.edu/alerts/2026/05/04/nationwide-security-breach-involving-canvas/]
- **UT Austin** (May 2026): States the incident "was not directed at UT Austin" and represents "a vendor-level event that may impact multiple institutions." Advises caution with phishing and to access Canvas only via canvas.utexas.edu [source: https://tech.utexas.edu/news/canvas-vendor-security-incident-may-2026]
- **Boise State** (May 5): "We have no indication that Boise State Canvas data has been affected." Notes that university passwords are managed by the university's own systems and not shared with Canvas [source: https://www.boisestate.edu/oit/2026/05/05/widespread-data-breach-involving-canvas/]
- **University of Wisconsin-Milwaukee** (May 5): "This breach will not affect final exams or course grades." Emphasizes that "UWM does not collect student ID numbers, dates of birth, government identifiers or financial information in Canvas." No confirmed UWM data exposure [source: https://uwm.edu/information-technology/uwm-monitoring-nationwide-canvas-security-breach/]
- **University of Auckland (New Zealand)** (May 6): First international institutional advisory. Confirms potential exposure of names, emails, student IDs, and Canvas messaging content; states student assessment data, passwords, and SSO credentials were not affected. Notes "at this stage, no data has been released publicly" [source: https://www.auckland.ac.nz/en/news/notices/2026/cybersecurity-incident-involving-canvas.html]
- **K-12 example - Wayzata Public Schools (MN)**: Activated incident response team, told families to watch for phishing referencing Canvas, and to monitor accounts for unusual activity [source: https://www.fox9.com/news/canvas-data-breach-hackers-claim-info-275-million-users-across-9000-schools]

Inside Higher Ed quoted cybersecurity commentators on May 5 framing the broader pattern: vendors like Canvas represent "armored trucks" of data, more attractive targets than individual institutions. ShinyHunters has also recently been publicly attributed to attacks involving University of Pennsylvania, Princeton, and Harvard environments [source: https://www.insidehighered.com/news/tech-innovation/administrative-tech/2026/05/05/pay-or-leak-hackers-target-big-higher-ed-vendor].

A class-action investigation has been announced (sponsored by Bryson Harris Suciu & DeMay PLLC). Chimicles Schwartz Kriner & Donaldson-Smith LLP has also opened an investigation. As of midday May 6, 2026, no federal-court complaint has yet been filed [source: https://www.classaction.org/data-breach-lawsuits/instructure-may-2026] [source: https://chimicles.com/instructure-canvas-data-breach-investigation/].

Confidence: High that these advisories exist with the cited content.

### 7. Threat Actor Context: ShinyHunters

ShinyHunters is a financially motivated extortion group with a history of data-theft operations targeting cloud-tenant SaaS environments. Recent attributed victims include Google, AT&T, and Air France-KLM, with multiple breaches tied to Salesforce-tenant intrusions [source: https://www.bleepingcomputer.com/news/security/instructure-confirms-data-breach-shinyhunters-claims-attack/]. Other Salesforce-related victims publicly attributed to the group during 2025-2026 include Canada Life, McGraw-Hill, Carnival Cruises, and Adobe [source: https://www.gblock.app/articles/instructure-canvas-may-2026-breach-student-data].

The group typically operates a Tor-hosted leak site, posts evidence samples, sets a ransom deadline, and threatens to publish stolen data if extortion is not paid. In the Instructure case, the group set a leak deadline of May 6, 2026 and posted: "Pay or Leak" [source: https://securityaffairs.com/191686/cyber-crime/educational-tech-firm-instructure-data-breach-may-have-impacted-9000-schools.html].

Note: This is external context useful only insofar as it informs prediction of next steps (data dump if ransom is not paid). Detailed actor TTPs are out of scope for this admin-decision report.

Confidence: High on attribution of the leak-site post to ShinyHunters. Medium on the broader campaign attribution pattern (relies on press reporting, not government attribution).

### 8. Prior Incident: September 2025 Salesforce Breach

This is Instructure's second confirmed cybersecurity incident in approximately eight months. In September 2025, ShinyHunters used social engineering against Instructure's Salesforce environment, accessing business-contact information only (no product or customer data). At that time, Instructure notified law enforcement and implemented additional security measures [source: https://www.rescana.com/post/instructure-canvas-cybersecurity-incidents-analysis-of-2025-salesforce-breach-and-2026-canvas-data-2-beta-security-event/].

Whether the May 2026 incident is connected to that earlier compromise (e.g., persistent access, related credentials, or follow-on Salesforce access) has not been confirmed [source: https://www.gblock.app/articles/instructure-canvas-may-2026-breach-student-data].

Confidence: High on the existence of the September 2025 incident. Low on any technical link to the May 2026 incident.

## Indicators of Compromise

**No IOCs (IP addresses, file hashes, domains, or specific TTPs) have been published by Instructure, CISA, or any CERT as of May 5, 2026** [source: https://gbhackers.com/canvas-confirms-data-breach-following-shinyhunters-claim/]. This is consistent with ShinyHunters operations historically: they tend to focus on data theft and extortion rather than persistent malware footprints.

Detection guidance based on what is known:

- Look for anomalous activity associated with Canvas Developer Keys, OAuth tokens, and integration accounts in the days preceding April 30, 2026
- Audit any custom integrations that consumed Canvas Data 2 / Canvas Beta APIs for unusual export volumes or off-hours access
- Watch for phishing emails targeting students, faculty, and staff that reference Canvas (the stolen names + email + student-ID + message-content combination is high-quality phishing-pretext material) [source: https://www.techradar.com/pro/security/canvas-maker-instructure-reveals-data-breach-confirms-user-personal-information-leaked]

Confidence: High that no formal IOC release exists; medium on detection guidance (interpretive, based on disclosed facts).

## Recommended Actions for Higher-Ed IT Admins

Concrete actions a Cornell-scale IT admin can take based on what Instructure has disclosed:

1. **Re-authorize and audit Canvas integrations.** Instructure's reissued application keys carry a timestamp in the naming convention. Walk every active integration through the Developer Keys panel; revoke any older keys that should no longer be in use [source: https://www.bleepingcomputer.com/news/security/instructure-confirms-data-breach-shinyhunters-claims-attack/].
2. **Rotate any Canvas API keys with broad read permissions** even if not flagged by Instructure, especially keys used for Canvas Data 2 exports [source: https://cipherssecurity.com/instructure-canvas-security-incident-2026/].
3. **Enable login audit logging** (Admin -> Account -> Logging in Canvas) and retain logs for forensic review [source: https://cipherssecurity.com/instructure-canvas-security-incident-2026/].
4. **Contact your Instructure CSM/support channel** for tenant-specific scope. Instructure has not published per-institution impact statements; you must request yours [source: https://www.k12dive.com/news/instructure-confirms-cybersecurity-incident/819362/].
5. **Prepare FERPA notification posture.** Names + student IDs + messages cross the FERPA threshold for "education records" in many institutional interpretations. Get Privacy/Counsel reviewing now whether your institution must notify [source: https://cipherssecurity.com/instructure-canvas-security-incident-2026/].
6. **Coordinate phishing alerts.** The exposed data (name + institutional email + student ID + message threads) is a near-perfect spear-phishing kit. Push proactive comms to students/faculty about Canvas-themed phishing [source: https://www.fox9.com/news/canvas-data-breach-hackers-claim-info-275-million-users-across-9000-schools].
7. **Confirm SSO posture.** Several universities (e.g., Boise State) emphasized that Canvas does not hold their primary credentials because authentication is via institutional SSO. Confirm that's true at your institution and that no fallback local accounts exist [source: https://www.boisestate.edu/oit/2026/05/05/widespread-data-breach-involving-canvas/].
8. **Watch the May 6 leak deadline.** ShinyHunters set a deadline of May 6, 2026. If data drops, monitor for your institution's records appearing in samples and update your scoping accordingly [source: https://securityaffairs.com/191686/cyber-crime/educational-tech-firm-instructure-data-breach-may-have-impacted-9000-schools.html].

Confidence: High on items 1, 3, 4, 6, 7. Medium on item 5 (FERPA interpretation depends on institutional counsel).

## Confidence Assessment

### High Confidence

- An incident occurred and was publicly disclosed by Instructure on May 1, 2026 [source: https://status.instructure.com/]
- Exposed data categories: names, email addresses, student IDs, user-to-user messages [source: https://www.bleepingcomputer.com/news/security/instructure-confirms-data-breach-shinyhunters-claims-attack/]
- Categories Instructure says were not exposed: passwords, dates of birth, government IDs, financial info [source: https://www.bitdefender.com/en-us/blog/hotforsecurity/canvas-data-breach-2026]
- Instructure rotated application keys and required customer re-authorization [source: https://status.instructure.com/]
- ShinyHunters listed Instructure on its leak site on May 3 [source: https://securityaffairs.com/191686/cyber-crime/educational-tech-firm-instructure-data-breach-may-have-impacted-9000-schools.html]
- This is Instructure's second confirmed incident within ~8 months (prior: September 2025 Salesforce social-engineering breach) [source: https://www.rescana.com/post/instructure-canvas-cybersecurity-incidents-analysis-of-2025-salesforce-breach-and-2026-canvas-data-2-beta-security-event/]

### Medium Confidence

- 275 million users / ~9,000 institutions / 3.65 TB scope (attacker-supplied; not confirmed by Instructure) [source: https://www.bleepingcomputer.com/news/security/instructure-confirms-data-breach-shinyhunters-claims-attack/]
- Initial access vector being a Salesforce-related compromise (one publication's interpretation; not confirmed by Instructure) [source: https://www.securitymagazine.com/articles/102283-instructure-parent-of-canvas-confirms-data-breach]
- April 30, 2026 detection date (reported by gbhackers; not directly confirmed by Instructure) [source: https://gbhackers.com/canvas-confirms-data-breach-following-shinyhunters-claim/]

### Low Confidence

- Whether the May 2026 incident is technically linked to the September 2025 Salesforce breach
- Whether any specific Cornell-scale institution's data is in the stolen set; per-tenant scope has not been published

## Open Questions

- What was the actual initial access vector?
- How many institutions had data confirmed exfiltrated, vs. those that were only theoretically in scope?
- ~~Will ShinyHunters release the data (deadline: May 6, 2026)?~~ Deadline arrived May 6; no public bulk dump observed at midday ET. Open: will the deadline slip, will partial samples appear, or did Instructure quietly negotiate?
- Will Instructure publish a post-incident technical report or IOC list?
- Is there a regulatory/notification timeline (state-AG breach notifications) that institutions need to plan around?
- No CVE assigned: will this remain a credential/token compromise narrative or will a vulnerability later be cataloged?
- Will more international institutions (UK Russell Group, Australia Group of Eight, Canadian U15) follow Auckland and post advisories?

## Sources

### Vendor Security Advisories

- [Instructure status page](https://status.instructure.com/) - Primary vendor advisory
- [UT Austin: Canvas Vendor Security Incident - May 2026](https://tech.utexas.edu/news/canvas-vendor-security-incident-may-2026)
- [Rutgers IT: Nationwide security breach involving Canvas](https://it.rutgers.edu/alerts/2026/05/04/nationwide-security-breach-involving-canvas/)
- [Boise State OIT: Widespread data breach involving Canvas](https://www.boisestate.edu/oit/2026/05/05/widespread-data-breach-involving-canvas/)
- [UWM IT: UWM monitoring nationwide Canvas security breach](https://uwm.edu/information-technology/uwm-monitoring-nationwide-canvas-security-breach/)
- [University of Auckland: Cybersecurity incident involving Canvas](https://www.auckland.ac.nz/en/news/notices/2026/cybersecurity-incident-involving-canvas.html)

### Government / CERT Advisories

- None published as of May 5, 2026. No CISA advisory, no CVE, no national CERT bulletin.

### Technical Analyses

- [Rescana: Instructure Canvas Cybersecurity Incidents - 2025 Salesforce + 2026 Canvas Data 2 / Beta](https://www.rescana.com/post/instructure-canvas-cybersecurity-incidents-analysis-of-2025-salesforce-breach-and-2026-canvas-data-2-beta-security-event/)
- [Ciphers Security: Instructure Discloses Cybersecurity Incident Affecting Canvas Platform](https://cipherssecurity.com/instructure-canvas-security-incident-2026/)
- [GBlock: Instructure Canvas May 2026 Breach Student Data](https://www.gblock.app/articles/instructure-canvas-may-2026-breach-student-data)

### News Coverage

- [BleepingComputer: Instructure confirms data breach, ShinyHunters claims attack](https://www.bleepingcomputer.com/news/security/instructure-confirms-data-breach-shinyhunters-claims-attack/)
- [BleepingComputer: Edu tech firm Instructure discloses cyber incident, probes impact](https://www.bleepingcomputer.com/news/security/edu-tech-firm-instructure-discloses-cyber-incident-probes-impact/)
- [SecurityWeek: Edtech Firm Instructure Discloses Data Breach](https://www.securityweek.com/edtech-firm-instructure-discloses-data-breach/)
- [TechCrunch: Hackers steal students' data during breach at education tech giant Instructure](https://techcrunch.com/2026/05/05/hackers-steal-students-data-during-breach-at-education-tech-giant-instructure/)
- [TechRepublic: Canvas Breach May Put 275M Users, 9,000 Schools at Risk](https://www.techrepublic.com/article/news-canvas-instructure-breach-275m-users/)
- [TechRadar: Canvas maker Instructure reveals data breach](https://www.techradar.com/pro/security/canvas-maker-instructure-reveals-data-breach-confirms-user-personal-information-leaked)
- [K-12 Dive: Instructure confirms cybersecurity incident](https://www.k12dive.com/news/instructure-confirms-cybersecurity-incident/819362/)
- [Bitdefender Hot for Security: Instructure Confirms Canvas Breach](https://www.bitdefender.com/en-us/blog/hotforsecurity/canvas-data-breach-2026)
- [SecurityAffairs: Educational tech firm Instructure data breach may have impacted 9,000 schools](https://securityaffairs.com/191686/cyber-crime/educational-tech-firm-instructure-data-breach-may-have-impacted-9000-schools.html)
- [Techzine Global: ShinyHunters claims Instructure breach, data from 275M users stolen](https://www.techzine.eu/news/security/140994/shinyhunters-claims-instructure-breach-data-from-275m-users-stolen/)
- [GBHackers: Canvas Confirms Data Breach Following ShinyHunters Claim](https://gbhackers.com/canvas-confirms-data-breach-following-shinyhunters-claim/)
- [Security Magazine: Instructure, Parent of Canvas, Confirms Data Breach](https://www.securitymagazine.com/articles/102283-instructure-parent-of-canvas-confirms-data-breach)
- [The Cyber Express: Canvas Cybersecurity Incident Exposes User Data](https://thecyberexpress.com/canvas-cybersecurity-incident/)
- [TechJack Solutions: Instructure Canvas Discloses Second Cybersecurity Incident in Eight Months](https://techjacksolutions.com/scc-intel/instructure-canvas-discloses-second-cybersecurity-incident-in-eight-months-amid-ongoing-investigation/)
- [FOX 9: Wayzata Public Schools warns parents of Canvas breach](https://www.fox9.com/news/canvas-data-breach-hackers-claim-info-275-million-users-across-9000-schools)
- [Inside Higher Ed: "PAY OR LEAK": Hackers Target Big Higher Ed Vendor](https://www.insidehighered.com/news/tech-innovation/administrative-tech/2026/05/05/pay-or-leak-hackers-target-big-higher-ed-vendor)
- [Malwarebytes: Millions of students' personal data stolen in major education breach](https://www.malwarebytes.com/blog/news/2026/05/millions-of-students-personal-data-stolen-in-major-education-cyberattack)

### Litigation

- [ClassAction.org: Instructure Data Breach Confirmed, Attorneys Investigating](https://www.classaction.org/data-breach-lawsuits/instructure-may-2026)
- [Chimicles Schwartz Kriner & Donaldson-Smith: Instructure (Canvas LMS) Data Breach Investigation](https://chimicles.com/instructure-canvas-data-breach-investigation/)

### Community Tools & Resources

- None specific to this incident as of May 5, 2026.

## Update History

- 2026-05-06 12:38 PM ET - ShinyHunters ransom deadline (May 6) arrived: no public bulk dump observed at midday ET. Added University of Auckland (first international advisory) and UWM advisories. Added Inside Higher Ed "Pay or Leak" feature and Malwarebytes coverage. Noted second class-action investigator (Chimicles). Updated Open Questions. Status page and bleepingcomputer/gbhackers/bitdefender priority sources have not changed since initial report.
- 2026-05-05 11:06 PM ET - Initial report

## How This Report Was Generated

Generated by the Research agent (Claude Opus 4.7) via /research, anti-hallucination guardrails on. Every claim is grounded in a fetched source. Confidence levels are flagged. The searxng and threatintel MCP servers were not available in this session; WebSearch and WebFetch were used in their place. Scheduled for 4-hour update loop.
