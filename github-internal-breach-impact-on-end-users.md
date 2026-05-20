---
title: "GitHub Internal Breach: Impact on End Users"
date: 2026-05-20
updated: 2026-05-20 8:30 AM ET
summary: "GitHub confirmed on May 20, 2026 that attackers exfiltrated roughly 3,800 of its own internal repositories after a GitHub employee installed a poisoned VS Code extension. As of this writing GitHub reports no evidence of impact to customer repositories, organizations, or user data, so most GitHub users do not need to act, but credential hygiene is strongly advised."
---

## Current Status

- A real, confirmed incident. GitHub publicly confirmed on May 20, 2026 that a third party gained unauthorized access to GitHub-internal repositories. This is not a rumor or a fabricated story ([source](https://www.securityweek.com/github-confirms-hack-impacting-3800-internal-repositories/)).
- Scope as currently understood: approximately 3,800 GitHub-internal repositories were exfiltrated. The threat actor (TeamPCP) claims roughly 4,000; GitHub says the ~3,800 figure is "directionally consistent" with its investigation ([source](https://thehackernews.com/2026/05/github-investigating-teampcp-claimed.html)).
- Attack vector: a poisoned Visual Studio Code extension installed on a single GitHub employee's device. GitHub detected and contained the compromised endpoint on May 19, 2026 ([source](https://www.bleepingcomputer.com/news/security/github-confirms-breach-of-3-800-repos-via-malicious-vscode-extension/)).
- Customer impact, per GitHub: as of May 20, 2026, "no evidence of impact to customer information stored outside of GitHub's internal repositories" (customer enterprises, organizations, and repositories). The investigation is ongoing and GitHub is monitoring for follow-on activity ([source](https://www.bleepingcomputer.com/news/security/github-investigates-internal-repositories-breach-claimed-by-teampcp/)).
- What is unresolved: GitHub has not published a full post-incident report, has not named the malicious extension, and has not detailed exactly what secrets or credentials lived on the compromised device. The stolen internal source code could still be mined by attackers for secrets or vulnerabilities ([source](https://www.securityweek.com/github-confirms-hack-impacting-3800-internal-repositories/)).
- Bottom line for the reader: this is a breach of GitHub the company, not of github.com customer data. Most developers and org admins do not need to take emergency action, but rotating any credentials hardcoded in code and auditing IDE extensions are prudent precautions.

## Table of Contents

1. What Happened
2. Technical Analysis
3. Timeline
4. Discovery
5. Response
6. Impact on End Users
7. Related Incidents (Context)
8. Indicators of Compromise
9. Confidence Assessment
10. Open Questions
11. Sources
12. Update History
13. How This Report Was Generated

## Findings

### 1. What Happened

On May 19, 2026, a threat actor group operating as **TeamPCP** posted on the "Breached" cybercrime forum claiming it had accessed GitHub's internal systems and was offering roughly 4,000 private code repositories for sale, with a minimum asking price of $50,000. The group framed the listing as a sale rather than extortion and indicated it would leak the data publicly if no buyer emerged ([source](https://www.bleepingcomputer.com/news/security/github-investigates-internal-repositories-breach-claimed-by-teampcp/)).

GitHub confirmed the incident publicly on May 20, 2026. According to GitHub's statements, attackers compromised an employee's device through a **poisoned Visual Studio Code extension**, and used the access gained on that device to exfiltrate **approximately 3,800 GitHub-internal repositories**. GitHub stated: "Our current assessment is that the activity involved exfiltration of GitHub-internal repositories only. The attacker's current claims of ~3,800 repositories are directionally consistent with our investigation so far" ([source](https://www.securityweek.com/github-confirms-hack-impacting-3800-internal-repositories/)).

The critical distinction for readers: what was stolen is **GitHub's own corporate/internal source code**, not the repositories, organizations, or account data of GitHub's customers. GitHub stated: "While we currently have no evidence of impact to customer information stored outside of GitHub's internal repositories (such as our customers' enterprises, organizations, and repositories), we are closely monitoring our infrastructure for follow-on activity" ([source](https://www.bleepingcomputer.com/news/security/github-investigates-internal-repositories-breach-claimed-by-teampcp/)).

GitHub is owned by Microsoft, and the VS Code extension used as the entry point ran in Microsoft's own Visual Studio Code editor, which makes this a developer-tooling supply-chain attack against the platform vendor itself ([source](https://www.infosecurity-magazine.com/news/github-confirms-breach-vs-code/)).

### 2. Technical Analysis

**Entry point: a poisoned VS Code extension.** GitHub's security team identified a malicious version of a VS Code extension running on an employee device. GitHub did not publicly name the specific extension. VS Code extensions run with broad access to the developer's machine: security researchers quoted in coverage noted that an extension can read credentials, SSH keys, cloud keys, and configuration files present on the device. That access is what let the attacker pivot from the employee's laptop into GitHub-internal repositories ([source](https://www.securityweek.com/github-confirms-hack-impacting-3800-internal-repositories/)).

**Why the access reached internal repos.** A GitHub engineer's machine would typically hold credentials, tokens, or session material that authenticate to GitHub-internal source control. Once the poisoned extension harvested that material, the attacker could authenticate as the employee and clone internal repositories at scale. GitHub confirmed that "critical secrets" were present and required rotation, but did not enumerate the secret types ([source](https://www.it-connect.tech/github-breach-3800-internal-repositories-stolen-after-an-employees-pc-was-hacked/)).

**Threat actor profile.** TeamPCP is a group repeatedly linked to software supply-chain attacks across GitHub, PyPI, npm, and Docker ecosystems. Independent technical analysis from Phoenix Security places this GitHub breach as the latest in a series of TeamPCP "waves" running since March 2026, using a self-propagating worm that steals cloud and developer credentials and abuses them to publish malicious package versions. Phoenix Security frames the GitHub employee compromise as the same playbook: "compromised developer tooling as the path to upstream infrastructure" ([source](https://phoenix.security/teampcp-github-breach-durabletask-pypi-supply-chain-wave-four-2026/)).

Caveat on attribution: the TeamPCP-to-worm linkage and the campaign timeline come from Phoenix Security's analysis and the group's own forum and Telegram claims. GitHub itself has not publicly named or attributed the attacker as of this writing. Treat the TeamPCP attribution as Medium confidence ([source](https://phoenix.security/teampcp-github-breach-durabletask-pypi-supply-chain-wave-four-2026/)).

**No CVE involved.** This was not the exploitation of a software vulnerability in github.com. There is no CVE to patch. The compromise was credential theft via a malicious extension, so the defensive response is credential rotation and endpoint hygiene rather than patching ([source](https://phoenix.security/teampcp-github-breach-durabletask-pypi-supply-chain-wave-four-2026/)).

### 3. Timeline

Reverse chronological. All times converted to Eastern Time where a precise time was published.

- **May 20, 2026, morning ET:** GitHub publicly confirms the breach, attributing it to a poisoned VS Code extension on an employee device and stating ~3,800 internal repositories were exfiltrated. Multiple outlets (BleepingComputer, SecurityWeek, Infosecurity Magazine, The Hacker News) publish confirmations ([source](https://www.securityweek.com/github-confirms-hack-impacting-3800-internal-repositories/)).
- **May 19, 2026:** GitHub detects and contains the compromised employee device. GitHub stated: "Yesterday we detected and contained a compromise of an employee device involving a poisoned VS Code extension." GitHub removed the malicious extension version, isolated the endpoint, and began incident response ([source](https://www.bleepingcomputer.com/news/security/github-confirms-breach-of-3-800-repos-via-malicious-vscode-extension/)).
- **May 19, 2026:** TeamPCP posts on the "Breached" cybercrime forum claiming access to ~4,000 GitHub private repositories, asking a $50,000 minimum ([source](https://www.bleepingcomputer.com/news/security/github-investigates-internal-repositories-breach-claimed-by-teampcp/)).
- **Night of May 19 into May 20, 2026:** GitHub rotates critical secrets, prioritizing the highest-impact credentials first. GitHub stated: "Critical secrets were rotated yesterday and overnight with the highest-impact credentials prioritized first" ([source](https://www.infosecurity-magazine.com/news/github-confirms-breach-vs-code/)).

Note: there is minor inconsistency across outlets on whether the forum post and the detection occurred on May 19 or May 20. The weight of coverage places initial detection and the forum claim on May 19, with GitHub's public confirmation on May 20. Treat the exact hour-level sequence as Medium confidence ([source](https://www.it-connect.tech/github-breach-3800-internal-repositories-stolen-after-an-employees-pc-was-hacked/)).

### 4. Discovery

GitHub's own security team discovered the incident. According to GitHub, the team detected the compromised employee device and identified the poisoned VS Code extension, then removed the malicious extension version and isolated the endpoint. Infosecurity Magazine reports the breach was detected on May 19, 2026. The public claim by TeamPCP on the Breached forum occurred in the same window, so the disclosure was effectively forced into the open by the attacker's sale listing rather than discovered solely through GitHub's internal monitoring ([source](https://www.infosecurity-magazine.com/news/github-confirms-breach-vs-code/)).

GitHub disclosed the incident through public statements (reported as a thread on X and statements provided to press) rather than through a formal post on the GitHub Blog. As of this report, the GitHub Blog homepage carried no post about the breach, the poisoned extension, or TeamPCP ([source](https://github.blog/)).

### 5. Response

GitHub's stated response actions:

- **Removed** the malicious extension version and isolated the compromised endpoint ([source](https://www.bleepingcomputer.com/news/security/github-confirms-breach-of-3-800-repos-via-malicious-vscode-extension/)).
- **Rotated critical secrets**, prioritizing the highest-impact credentials first, overnight from May 19 into May 20 ([source](https://www.infosecurity-magazine.com/news/github-confirms-breach-vs-code/)).
- **Began incident response** immediately and is continuing to "analyze logs, validate secret rotation, and monitor for any follow-on activity" ([source](https://www.securityweek.com/github-confirms-hack-impacting-3800-internal-repositories/)).
- **Committed to notifying customers** through established incident response channels if any customer impact is discovered, and to publishing a fuller report once the investigation completes ([source](https://coinedition.com/cz-warns-developers-to-rotate-api-keys-as-github-confirms-internal-breach/)).

GitHub's posture is reactive on customer notification: it will notify customers only if customer impact is found. As of this writing, GitHub says no such impact has been found ([source](https://coinedition.com/cz-warns-developers-to-rotate-api-keys-as-github-confirms-internal-breach/)).

### 6. Impact on End Users

This is the central question. Based on current evidence:

**For the typical GitHub user (individual developers, most organizations): no confirmed impact, no emergency action required.** GitHub states the exfiltration involved GitHub-internal repositories only, with no evidence of impact to customer repositories, enterprise organizations, or user data stored outside internal systems. github.com itself was not reported down or compromised as a service. Passwords, customer private repos, and customer access tokens are not reported as exposed ([source](https://coinedition.com/cz-warns-developers-to-rotate-api-keys-as-github-confirms-internal-breach/)).

**The residual risk: what attackers can learn from GitHub's own source code.** The stolen material is GitHub's internal source code. Attackers can mine that code for hardcoded secrets, undisclosed vulnerabilities, or architectural weaknesses in the platform. If a usable secret or an exploitable flaw is found in the stolen code, that could later translate into a follow-on attack that does affect customers. This is a real but currently unrealized risk, and it is why GitHub is rotating secrets and monitoring ([source](https://www.securityweek.com/github-confirms-hack-impacting-3800-internal-repositories/)).

**Prudent precautions developers and org admins can take now (not mandated by GitHub, but low-cost and sensible):**

1. **Rotate any credentials hardcoded in code, even in private repositories.** Changpeng Zhao (CZ), founder of Binance, publicly advised: "If you have API keys in your code, even private repos, now is the time to double-check and change them." This is general good hygiene that the incident makes timely; it is not GitHub instructing customers to rotate ([source](https://coinedition.com/cz-warns-developers-to-rotate-api-keys-as-github-confirms-internal-breach/)).
2. **Audit VS Code (and other IDE) extensions.** The breach's root cause was a malicious extension on a developer machine. Review installed extensions on developer and CI machines, remove anything unrecognized, and consider disabling extension auto-update so new versions are verified before they run ([source](https://phoenix.security/teampcp-github-breach-durabletask-pypi-supply-chain-wave-four-2026/)).
3. **Watch for GitHub's follow-up communications.** GitHub has committed to notifying customers through incident response channels if impact is found, and to publishing a fuller report. Monitor official GitHub channels rather than relying on forum claims ([source](https://coinedition.com/cz-warns-developers-to-rotate-api-keys-as-github-confirms-internal-breach/)).
4. **Treat the broader TeamPCP package campaign as the more direct end-user threat.** If your projects depend on PyPI/npm packages, the TeamPCP package-poisoning waves (see Related Incidents) are a more concrete supply-chain risk than the GitHub corporate breach itself ([source](https://phoenix.security/teampcp-github-breach-durabletask-pypi-supply-chain-wave-four-2026/)).

**What is NOT required:** There is no evidence that GitHub users need to change their GitHub account passwords, revoke GitHub personal access tokens, or assume their private repositories were copied. GitHub has not asked customers to do any of this. If that changes, GitHub says it will notify affected customers directly ([source](https://www.bleepingcomputer.com/news/security/github-investigates-internal-repositories-breach-claimed-by-teampcp/)).

### 7. Related Incidents (Context)

Two adjacent incidents are circulating alongside this story. They are distinct from the GitHub corporate breach and should not be conflated with it, but they share the GitHub-as-attack-surface theme.

**Grafana Labs GitHub token breach (May 16-17, 2026).** Grafana Labs disclosed that an unauthorized party obtained a token granting access to Grafana's GitHub environment and downloaded portions of Grafana's codebase. The attacker attempted extortion; Grafana refused to pay, citing FBI guidance. Grafana stated: "no customer data or personal information was accessed during this incident." Coverage attributes the Grafana incident to a group identified as "CoinbaseCartel," not TeamPCP. This is a separate company and a separate threat actor; it is relevant only as evidence that GitHub-hosted code environments are an active target in May 2026 ([source](https://thehackernews.com/2026/05/grafana-github-token-breach-led-to.html)).

**TeamPCP package-poisoning campaign (PyPI durabletask and others).** Independent analysis from Phoenix Security ties the GitHub employee compromise to a wider TeamPCP campaign that, on May 19, 2026, also pushed malicious versions (1.4.1, 1.4.2, 1.4.3) of the PyPI package `durabletask`, which has roughly 417,000 monthly downloads. That package campaign steals AWS, Azure, GCP, Kubernetes, and developer-tool credentials from machines that import the poisoned package. For developers, this package campaign is the more direct supply-chain hazard. Indicators for it are included below. Note: the durabletask details rest on a single analyst source (Phoenix Security) and the actor's own claims; treat as Medium confidence ([source](https://phoenix.security/teampcp-github-breach-durabletask-pypi-supply-chain-wave-four-2026/)).

## Indicators of Compromise

The GitHub corporate breach itself has no published IOCs for end users: the compromised device was a GitHub employee laptop, and GitHub has not named the malicious VS Code extension. The threat intelligence feeds checked for this report (urlhaus, malwarebazaar, threatfox, feodo, CISA KEV) returned no GitHub-breach-specific indicators, which is expected for a corporate-internal incident [source: mcp__threatintel__search_threats].

The IOCs below are for the **related TeamPCP `durabletask` PyPI package campaign**, sourced from Phoenix Security. They are actionable for developers who use PyPI packages. They are NOT indicators of the GitHub corporate breach. Treat as Medium confidence (single analyst source).

**Malicious packages (PyPI):**

| Package / Version | SHA-256 |
|---|---|
| durabletask 1.4.1 | 3de04fe2a76262743ed089efa7115f4508619838e77d60b9a1aab8b20d2cc8bf |
| durabletask 1.4.2 | 85f54c089d78ebfb101454ec934c767065a342a43c9ee1beac8430cdd3b2086f |
| durabletask 1.4.3 | c0b094e46842260936d4b97ce63e4539b99a3eae48b736798c700217c52569dc |
| rope.pyz second-stage payload | 069ac1dc7f7649b76bc72a11ac700f373804bfd81dab7e561157b703999f44ce |

**C2 infrastructure:**

| Indicator | Type |
|---|---|
| check.git-service[.]com | domain |
| t.m-kosche[.]com | domain |
| 83.142.209.0/24 | IP range |

**File artifacts (Linux):**

| Path | Meaning |
|---|---|
| /tmp/managed.pyz | dropped second-stage payload |
| ~/.cache/.sys-update-check | AWS SSM propagation marker |
| ~/.cache/.sys-update-check-k8s | Kubernetes propagation marker |
| /usr/bin/pgmonitor.py or ~/.local/bin/pgmonitor.py | persistence binary |
| pgsql-monitor.service | persistence systemd service (disguised) |

([source](https://phoenix.security/teampcp-github-breach-durabletask-pypi-supply-chain-wave-four-2026/))

**Detection commands (for the durabletask campaign, Linux):**

```bash
# Check the installed durabletask version
pip show durabletask

# Scan for the dropped payload
ls -la /tmp/managed.pyz

# Check for propagation markers
ls ~/.cache/.sys-update-check ~/.cache/.sys-update-check-k8s

# Check for the persistence service and binary
systemctl status pgsql-monitor.service
ls /usr/bin/pgmonitor.py ~/.local/bin/pgmonitor.py
```

([source](https://phoenix.security/teampcp-github-breach-durabletask-pypi-supply-chain-wave-four-2026/))

**Remediation for the durabletask campaign:** pin to `durabletask==1.4.0`, remove versions 1.4.1 through 1.4.3, rotate cloud and developer credentials if a compromised version ran, add the C2 domains and the 83.142.209.0/24 range to egress deny lists, and remove the `pgsql-monitor.service` persistence ([source](https://phoenix.security/teampcp-github-breach-durabletask-pypi-supply-chain-wave-four-2026/)).

## Confidence Assessment

### High Confidence

- A real security incident occurred and GitHub publicly confirmed it on May 20, 2026. This is corroborated by at least five independent outlets (BleepingComputer, SecurityWeek, Infosecurity Magazine, The Hacker News, IT-Connect) all quoting GitHub statements ([source](https://www.securityweek.com/github-confirms-hack-impacting-3800-internal-repositories/)).
- The attack vector was a poisoned VS Code extension on a GitHub employee's device. Consistently reported across all sources, with GitHub quoted directly ([source](https://www.bleepingcomputer.com/news/security/github-confirms-breach-of-3-800-repos-via-malicious-vscode-extension/)).
- Roughly 3,800 GitHub-internal repositories were exfiltrated; GitHub calls this figure "directionally consistent" with its investigation ([source](https://thehackernews.com/2026/05/github-investigating-teampcp-claimed.html)).
- As of May 20, 2026, GitHub reports no evidence of impact to customer repositories, organizations, or user data outside its internal systems ([source](https://www.bleepingcomputer.com/news/security/github-investigates-internal-repositories-breach-claimed-by-teampcp/)).
- GitHub rotated critical secrets and is continuing its investigation ([source](https://www.infosecurity-magazine.com/news/github-confirms-breach-vs-code/)).

### Medium Confidence

- Attribution to "TeamPCP." This rests on the group's own forum/Telegram claims and on Phoenix Security's analysis; GitHub itself has not publicly named the attacker ([source](https://phoenix.security/teampcp-github-breach-durabletask-pypi-supply-chain-wave-four-2026/)).
- The hour-level timeline (exact sequence of forum post vs. detection on May 19) varies slightly between outlets ([source](https://www.it-connect.tech/github-breach-3800-internal-repositories-stolen-after-an-employees-pc-was-hacked/)).
- The link between the GitHub corporate breach and the broader TeamPCP `durabletask`/worm campaign, and the durabletask IOCs themselves, rest substantially on a single analyst source (Phoenix Security) ([source](https://phoenix.security/teampcp-github-breach-durabletask-pypi-supply-chain-wave-four-2026/)).
- The $50,000 asking price and the "~4,000 repositories" figure are the attacker's own claims, repeated in reporting but not independently verified ([source](https://www.bleepingcomputer.com/news/security/github-investigates-internal-repositories-breach-claimed-by-teampcp/)).

## Open Questions

- Which VS Code extension was the malicious one? GitHub has not named it, so other developers cannot check whether they installed the same extension.
- What specific secrets or credentials were on the compromised employee device, and were all of them successfully rotated before the attacker could use them?
- Does any of the stolen GitHub-internal source code contain usable secrets or exploitable vulnerabilities that could later be turned against customers?
- Will GitHub publish a formal post-incident report, and when?
- Is the GitHub corporate breach genuinely the same TeamPCP operation as the PyPI `durabletask` campaign, or are reporters and analysts linking two separate events because they happened in the same week?
- Has any buyer actually acquired the data, or has TeamPCP leaked it publicly?

## Sources

### Vendor Security Advisories
- [GitHub Blog](https://github.blog/) (homepage checked; no breach post present as of report date)

### Technical Analyses
- [Phoenix Security, "TeamPCP Wave Four: GitHub Breach via Poisoned VS Code Extension"](https://phoenix.security/teampcp-github-breach-durabletask-pypi-supply-chain-wave-four-2026/)

### News Coverage
- [BleepingComputer, "GitHub confirms breach of 3,800 repos via malicious VSCode extension"](https://www.bleepingcomputer.com/news/security/github-confirms-breach-of-3-800-repos-via-malicious-vscode-extension/)
- [BleepingComputer, "GitHub investigates internal repositories breach claimed by TeamPCP"](https://www.bleepingcomputer.com/news/security/github-investigates-internal-repositories-breach-claimed-by-teampcp/)
- [SecurityWeek, "GitHub Confirms Hack Impacting 3,800 Internal Repositories"](https://www.securityweek.com/github-confirms-hack-impacting-3800-internal-repositories/)
- [Infosecurity Magazine, "GitHub Confirms Breach of Internal Repositories Via Malicious VS Code Extension"](https://www.infosecurity-magazine.com/news/github-confirms-breach-vs-code/)
- [The Hacker News, "GitHub Breached - Employee Device Hack Led to Exfiltration of 3,800 Internal Repositories"](https://thehackernews.com/2026/05/github-investigating-teampcp-claimed.html)
- [IT-Connect, "GitHub Breach: 3,800 Internal Repositories Stolen After an Employee's PC Was Hacked"](https://www.it-connect.tech/github-breach-3800-internal-repositories-stolen-after-an-employees-pc-was-hacked/)
- [CoinEdition, "CZ warns developers to rotate API keys as GitHub confirms internal breach"](https://coinedition.com/cz-warns-developers-to-rotate-api-keys-as-github-confirms-internal-breach/)
- [The Hacker News, "Grafana GitHub Token Breach Led to Codebase Download and Extortion Attempt"](https://thehackernews.com/2026/05/grafana-github-token-breach-led-to.html) (related, separate incident)

### Threat Intelligence Tooling
- Threat intelligence feed query (urlhaus, malwarebazaar, threatfox, feodo, CISA KEV) returned no GitHub-breach-specific indicators; feeds confirmed healthy except OTX, which was unavailable (OTX_API_KEY not set).

## Update History

- 2026-05-20 8:30 AM ET: Initial report. Fresh research. Documented the confirmed May 19-20, 2026 GitHub internal repositories breach, attack vector, GitHub's stated customer-impact assessment, end-user guidance, and the related Grafana and TeamPCP durabletask incidents as context.

## How This Report Was Generated

Research agent run on 2026-05-20. Web research via the SearXNG metasearch MCP (search_deep, search_news, search). Every cited URL was retrieved with WebFetch before citation; one source (cybernews.com) returned HTTP 403 and was excluded. Threat intelligence cross-check via the threatintel MCP (search_threats, get_recent_threats, get_feed_status): no GitHub-breach-specific indicators were present in the malware/IOC feeds, which is expected for a corporate-internal incident. The OTX feed was unavailable during the run. No formal GitHub Blog post or CVE exists for this incident; GitHub disclosed via public statements. Claims are graded High or Medium confidence; attacker-sourced figures and single-analyst attribution are flagged as Medium. Timestamps converted from UTC to Eastern Time (EDT, UTC-4).
