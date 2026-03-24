---
title: "LiteLLM PyPI Supply Chain Attack: Analysis and Comparison to XZ Utils Backdoor"
slug: litellm-pypi-supply-chain-attack
date: 2026-03-24
updated: 2026-03-24T19:00:00Z
summary: "On March 24, 2026, the LiteLLM Python package (versions 1.82.7 and 1.82.8) was compromised by threat actor TeamPCP via hijacked PyPI credentials, deploying a three-stage credential stealer that harvested SSH keys, cloud credentials, and crypto wallets from systems with ~95 million monthly downloads. This report analyzes the attack and compares it to the 2024 XZ Utils backdoor."
---

> **Last updated: March 24, 2026, 2:10 PM ET.** Mandiant engaged for incident response. CircleCI confirmed as the CI platform where credentials leaked. All releases blocked pending security scans. Exfiltration domain takedown efforts stalled.

## Findings

### 1. What Happened

On March 24, 2026, a threat actor operating under the handle TeamPCP published two malicious versions of the LiteLLM Python package to PyPI. LiteLLM is a widely used AI/ML library providing a unified interface to 100+ LLM providers, with approximately 95 million monthly downloads. The compromised versions (1.82.7 and 1.82.8) existed solely on PyPI. GitHub releases only extended to v1.82.6.dev1, meaning the attacker published directly to the package registry without touching the source repository. [source: https://www.endorlabs.com/learn/teampcp-isnt-done, https://github.com/BerriAI/litellm/issues/24518]

The compromise chain traces back to an earlier breach of Aqua Security's Trivy project. LiteLLM's CI/CD ran Trivy via `ci_cd/security_scans.sh`, which installed Trivy from the apt repo without version pinning. The compromised Trivy binary (v0.69.4+) contained credential-harvesting code that dumped environment variables via Cloudflare Tunnels. The compromised Trivy leaked all CircleCI credentials (not GitHub Actions as initially assumed), which included both the PyPI publish token and the GitHub PAT. The maintainer confirmed on Hacker News: "this originated from the trivy used in our ci/cd." The maintainer also confirmed the accounts had 2FA enabled, indicating the compromise was via a stolen CI/CD token rather than weak credentials. [source: https://github.com/BerriAI/litellm/issues/24518, https://news.ycombinator.com/item?id=47501729]

The attacker obtained not only the PyPI publishing token but also a GitHub Personal Access Token (PAT). Community members on issue #24518 identified suspicious activity on an unrelated repo owned by the maintainer (`krrishdholakia/blockchain`), confirming the attacker had broader access than initially reported. The attacker also created a `tpcp-docs` repository in the victim's GitHub account as an alternative exfiltration channel. [source: https://github.com/BerriAI/litellm/issues/24518]

The attacker's access extended beyond the main litellm repo. Malicious code was pushed into a separate BerriAI repository called `litellm-skills` on March 23 (commit `81c851c`), modifying a test workflow file with what was described as "a modification to secrets attack payload from @AdnaneKhan gato-x tool." This confirms the attacker was active in BerriAI infrastructure a full day before the PyPI publish. The code has since been deleted. [source: https://github.com/BerriAI/litellm/issues/24518, comment by seljak00vac at 12:21 PM ET]

### 2. Technical Analysis: The Three-Stage Payload

**Version 1.82.7** embedded an obfuscated payload in `litellm/proxy/proxy_server.py`, triggered on import via a base64-decoded payload. The injected 12 lines sat between legitimate code blocks to avoid suspicious clustering. The attack used `subprocess.run()` instead of `exec()`, bypassing static analysis. The attacker rebuilt the wheel with a regenerated RECORD file, so the RECORD entry for `proxy_server.py` contains the SHA-256 of the backdoored file. Standard integrity checks against the wheel's own metadata pass cleanly, which is why automated tooling did not catch the tampering. [source: https://www.endorlabs.com/learn/teampcp-isnt-done]

**Version 1.82.8** (published 13 minutes later at 6:52 AM ET) escalated the attack by adding a `.pth` file named `litellm_init.pth` (34,628 bytes). Python's site module executes `.pth` files on every interpreter startup. Any line beginning with `import` is passed to `exec()`, meaning the malware ran on every Python process, even without importing LiteLLM. [source: https://safedep.io/malicious-litellm-1-82-8-analysis/]

The payload operated in three stages:

**Stage 1 (Orchestrator):** Decoded embedded base64 scripts, piped credential harvester output to a temporary file, encrypted with AES-256-CBC + RSA-4096 OAEP using a hardcoded 4096-bit RSA public key, and exfiltrated as `tpcp.tar.gz` to `https://models.litellm.cloud/` (a spoofed domain registered March 23, 2026). [source: https://safedep.io/malicious-litellm-1-82-8-analysis/]

**Stage 2 (Credential Harvester):** A 17,281-byte collector targeting:
- SSH keys (RSA, Ed25519, ECDSA, DSA formats) across all users
- Environment variables, specifically targeting LLM API keys: `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `AZURE_API_KEY`, `COHERE_API_KEY`, `GEMINI_API_KEY`, `REPLICATE_API_KEY`, plus `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `GOOGLE_APPLICATION_CREDENTIALS`, `VERTEX_PROJECT`, database connection strings, JWT tokens, webhook credentials, and CI/CD secrets [source: https://mrcloudbook.com/litellm-1828-malicious-pypi-package-credential-stealer/]
- AWS credentials, including a full AWS SigV4 signing routine implemented in pure Python to dump Secrets Manager and SSM Parameter Store without requiring boto3
- GCP application default credentials
- Azure directory trees
- Kubernetes service account tokens and secrets across all namespaces
- Cryptocurrency wallets (Bitcoin, Ethereum, Solana, Monero, and others)
- Database passwords, SSL private keys, shell history, CI/CD configs
- `.env` files, `.gitconfig`
[source: https://safedep.io/malicious-litellm-1-82-8-analysis/, https://www.endorlabs.com/learn/teampcp-isnt-done]

**Stage 3 (Persistence + Lateral Movement):**
- Installed a systemd user service named `sysmon.service` (impersonating legitimate telemetry) running `~/.config/sysmon/sysmon.py` with `StartLimitIntervalSec=0`, forcing systemd to restart it indefinitely
- Polled `https://checkmarx.zone/raw` every 50 minutes for arbitrary binaries to execute
- Featured a kill switch checking for "youtube.com" in responses
- On Kubernetes clusters: created privileged pods named `node-setup-*` in `kube-system` namespace on every node, with `hostPID: True` and `hostNetwork: True`, mounting the host filesystem and using `chroot` to install persistence directly on each node's root filesystem
[source: https://safedep.io/malicious-litellm-1-82-8-analysis/, https://www.endorlabs.com/learn/teampcp-isnt-done]

The published package contained three commented-out earlier payload iterations, revealing the attacker's development progression: Iteration 1 used `StringIO` + `exec()` with RC4 cipher obfuscation and named output files. Iteration 2 was a transitional version carrying both old and new harvester implementations (the largest blob at 69,316 base64 characters). Iteration 3 (the active payload) rewrote the orchestrator to use subprocess piping, removed `exec()` calls, and stripped the RC4 layer. Leaving these iterations in the published package was an operational security failure. [source: https://www.endorlabs.com/learn/teampcp-isnt-done]

An ironic flaw: the `.pth` launcher spawned a child Python process via `subprocess.Popen`, but because `.pth` files trigger on every interpreter startup, the child re-triggered the same `.pth`, creating an exponential fork bomb that crashed affected machines, which actually helped surface the attack. [source: https://futuresearch.ai/blog/litellm-pypi-supply-chain-attack/]

### 3. Affected Versions and Timeline

| Time (ET) | Event |
|---|---|
| March 22, 2026 | Last clean version (1.82.6) published |
| March 23, 2026 | Spoofed domain `models.litellm.cloud` registered |
| March 23, 2026 | Malicious code pushed to `litellm-skills` repo (commit `81c851c`) |
| March 24, 6:39 AM | litellm 1.82.7 published to PyPI (payload in proxy_server.py) |
| March 24, 6:52 AM | litellm 1.82.8 published to PyPI (added .pth file) |
| March 24, ~8:30 AM | Version 1.82.7 confirmed also compromised |
| March 24, ~9:03 AM | GitHub issue #24512 closed as "not planned" and flooded with bot spam |
| March 24, ~9:12 AM | MLflow merges emergency PR pinning `litellm<=1.82.6` [source: https://github.com/mlflow/mlflow/pull/21971] |
| March 24, post-discovery | PyPI quarantined the entire litellm package (all versions) |
| March 24, ~9:48 AM | Issue #24518 opened as clean tracking issue (by `isfinne`) |
| March 24, ~9:52 AM | `krrishdholakia` confirms PyPI quarantine on GitHub |
| March 24, ~10:11 AM | Community discovers maintainer's GitHub PAT also compromised |
| March 24, ~11:09 AM | `krrish-berri-2` announces all maintainer accounts deleted and recreated |
| March 24, ~11:27 AM | Compromised versions deleted, package unquarantined |
| March 24, ~11:27 AM | BerriAI promises official advisory and incident report |
| March 24, post-11:27 AM | Issue #24512 reopened (now has 360+ comments) |
| March 24, ~12:06 PM | PyPI maintainer Mike Fiedler assigns **PYSEC-2026-2** advisory [source: https://github.com/pypa/advisory-database/blob/main/vulns/litellm/PYSEC-2026-2.yaml] |
| March 24, ~12:17 PM | Trivy dependency pinned to last known safe version (PR #24525) |
| March 24, ~12:21 PM | Community identifies malicious code in `litellm-skills` repo from March 23 |
| March 24, ~12:28 PM | Maintainer identity partially verified via HN account cross-reference |
| March 24, ~12:33 PM | Maintainer promises further updates: "I will be updating this thread, as we have more to share" |
| March 24, ~12:57 PM | `krrish-berri-2` confirms all 30 BerriAI repos validated: no deploy keys or env vars remaining |
| March 24, ~1:03 PM | BerriAI confirms engagement of Google's Mandiant for incident response |
| March 24, ~1:16 PM | Maintainer confirms CircleCI (not GitHub Actions) as the CI platform where credentials leaked; Trivy pinned to v0.69.3 (last safe version) |
| March 24, ~1:16 PM | All new releases blocked pending completion of security scans |
| March 24, ~1:52 PM | Domain takedown efforts for `litellm.cloud` stalled: Spaceship registrar returning empty automated replies |

[source: https://futuresearch.ai/blog/litellm-pypi-supply-chain-attack/, https://github.com/BerriAI/litellm/issues/24518]

### 4. Discovery

FutureSearch discovered the compromise when LiteLLM was pulled as a transitive MCP dependency in Cursor. The fork bomb behavior (machines crashing on any Python startup) likely accelerated detection. Independently, MDR firm Daylight AI identified the compromise through a Wiz Defend alert ("Python script executed base64 encoded code") in an actual customer production environment, confirming at least one real-world production compromise during the attack window. [source: https://futuresearch.ai/blog/litellm-pypi-supply-chain-attack/, https://daylight.ai/blog/litellm-library-and-an-expanding-supply-chain-campaign]

### 5. Response

PyPI quarantined the entire litellm package immediately after discovery, preventing downloads of all versions. Users reported that "All versions currently return 'No matching distribution found.'" The initial GitHub issue (#24512) was closed and flooded with attacker-generated bot spam, suggesting the attacker attempted to suppress disclosure. A second tracking issue (#24518) was opened to maintain a clean timeline. [source: https://github.com/BerriAI/litellm/issues/24518]

The maintainer (`krrishdholakia`) confirmed the quarantine on GitHub, then deleted all compromised maintainer accounts and migrated to new ones. Operating from a new GitHub account (`krrish-berri-2`), he coordinated with the PyPI team to remove the malicious versions and unquarantine the package. All GitHub, Docker, and PyPI keys were rotated. The PyPI package now shows version 1.82.6 as the latest available. The Trivy dependency was pinned to the last known safe version via PR #24525, closing the original attack vector. BerriAI has engaged Google's Mandiant for incident response and has blocked all new releases pending completion of security scans. The last 30 BerriAI repositories were audited and confirmed to have no remaining deploy keys or environment variables. An official advisory and incident report is actively being prepared. [source: https://github.com/BerriAI/litellm/issues/24518, https://github.com/BerriAI/litellm/pull/24525]

The new account created immediate community concern about identity verification. User `lattwood` noted: "uhh you joined github an hour ago," highlighting the difficulty of verifying maintainer identity after a compromise. The maintainer later linked his GitHub profile to his Hacker News account (`detente18`), which was cross-referenced by community members. However, user `pwilkin` warned that photo-based verification is insufficient in the LLM era, posting a deepfake proof-of-concept and pushing for real-world verification through official channels. No GPG-signed or official channel verification has been completed yet. Issue #24512 was subsequently reopened and has accumulated 360+ comments. [source: https://github.com/BerriAI/litellm/issues/24518]

PyPI maintainer Mike Fiedler assigned formal advisory **PYSEC-2026-2**, confirming the malicious releases and their removal. The advisory credits Callum McMahon (FutureSearch) as the reporter and describes the incident as: "After an API Token exposure from an exploited trivy dependency, two new releases of litellm were uploaded to PyPI containing automatically activated malware, harvesting sensitive credentials and files, and exfiltrating to a remote API." No CVE number has been assigned yet. [source: https://github.com/pypa/advisory-database/blob/main/vulns/litellm/PYSEC-2026-2.yaml]

Downstream projects moved quickly. MLflow merged an emergency PR (#21971) pinning `litellm<=1.82.6` at 9:12 AM ET, marked as "critical and needs to be in the next patch release." One enterprise user on issue #24512 reported running LiteLLM v1.81.14 as a centralized gateway for ~200 developers via AWS Bedrock, noting that their version pinning strategy protected them. Docker image users were generally safe because the image version was pinned prior to v1.82.7. [source: https://github.com/mlflow/mlflow/pull/21971, https://github.com/BerriAI/litellm/issues/24512]

### 6. TeamPCP: The Broader Campaign

This was not an isolated incident. TeamPCP executed a coordinated supply chain campaign across five ecosystems in under a month:

| Date | Target | Ecosystem | Method |
|---|---|---|---|
| Feb 28 | Trivy | GitHub Actions | Pwn Request workflow vulnerability |
| Mar 19 | Trivy (2nd wave) | GitHub/Docker Hub/Containers | Residual access from incomplete remediation |
| Mar 20 | npm (45+ packages) | npm | Self-propagating worm via stolen tokens |
| Mar 22 | Docker Hub | Container images | Aqua credential reuse |
| Mar 23 | KICS/OpenVSX | GitHub Actions/IDE extensions | Service account compromise |
| Mar 24 | LiteLLM | PyPI | Publishing credentials compromised via Trivy |

Attribution relies on matching tradecraft: identical C2 domains, persistence paths (`~/.config/sysmon/`), encryption schemes, and the `tpcp.tar.gz` filename directly referencing TeamPCP. The group also operates under aliases including DeadCatx3, PCPcat, ShellForce, and CanisterWorm. Two commented-out earlier payload iterations remained in the published package, an operational security failure revealing development progression from RC4-obfuscated shells to plaintext harvester code. [source: https://www.endorlabs.com/learn/teampcp-isnt-done]

Sysdig's analysis revealed additional C2 infrastructure: `checkmarx[.]zone` resolving to `83.142.209.11`, and Trivy exfiltration domain `scan.aquasecurtiy[.]org` (typosquat of "aquasecurity") at `45.148.10.212`. TeamPCP used vendor-specific typosquat domains for each compromised action to evade pattern-based detection. An additional exfiltration method involved creating a `tpcp-docs` repository in compromised GitHub accounts as a fallback data channel. Efforts to take down the `litellm.cloud` exfiltration domain (registered on Spaceship) have stalled, with abuse complaints to Spaceship and the registrar backend (demenin) receiving only empty automated replies. ICANN escalation is being considered, and the domain may still be live. [source: https://www.sysdig.com/blog/teampcp-expands-supply-chain-compromise-spreads-from-trivy-to-checkmarx-github-actions, https://github.com/BerriAI/litellm/issues/24518]

### 7. Comparison to XZ Utils Backdoor (CVE-2024-3094)

The XZ Utils backdoor, discovered March 29, 2024, is the most significant prior open-source supply chain attack. Here is how the two compare:

| Dimension | XZ Utils (CVE-2024-3094) | LiteLLM (March 2026) |
|---|---|---|
| **Attack vector** | Social engineering: 2+ years building trust as maintainer "Jia Tan" to gain commit and release manager rights [source: https://gist.github.com/thesamesam/223949d5a074ebc3dce9ee78baad9e27] | Credential theft: PyPI publishing token stolen via compromised Trivy in CI/CD pipeline [source: https://github.com/BerriAI/litellm/issues/24518] |
| **Time to execute** | ~2.5 years of patient social engineering | Hours (part of a 25-day multi-ecosystem blitz) |
| **Sophistication** | Extremely high: custom binary backdoor injected via build system (`build-to-host.m4`), exploiting glibc's IFUNC resolver to hook OpenSSH's `RSA_public_decrypt`. Conditional activation only on x86_64, glibc, Debian/Red Hat with systemd-integrated sshd. [source: https://gist.github.com/thesamesam/223949d5a074ebc3dce9ee78baad9e27] | High but more operational: Python-based three-stage payload with custom AWS SigV4 implementation, Kubernetes lateral movement, and persistent C2. Less technically novel but broader in scope. [source: https://safedep.io/malicious-litellm-1-82-8-analysis/] |
| **Target** | Remote code execution via SSH on Linux servers | Credential harvesting, data exfiltration, persistent backdoor access |
| **Scope of impact** | Narrow: only reached Fedora 40 beta, Debian unstable/testing/experimental, Kali, Arch. Never hit stable releases. [source: https://securitylabs.datadoghq.com/articles/xz-backdoor-cve-2024-3094/] | Broad: ~95M monthly downloads, transitive dependency for many AI agent frameworks, MCP servers, and LLM orchestration tools. Anyone who ran `pip install` during the window was immediately compromised. [source: https://www.endorlabs.com/learn/teampcp-isnt-done] |
| **Discovery** | Andres Freund noticed 500ms SSH latency anomaly while benchmarking. Serendipitous. [source: https://gist.github.com/thesamesam/223949d5a074ebc3dce9ee78baad9e27] | FutureSearch hit it as transitive MCP dependency. Fork bomb behavior accelerated detection. [source: https://futuresearch.ai/blog/litellm-pypi-supply-chain-attack/] |
| **Attribution** | Likely state-sponsored (consensus but unconfirmed). Single pseudonymous actor with possible support network creating social pressure on original maintainer. [source: https://securitylabs.datadoghq.com/articles/xz-backdoor-cve-2024-3094/] | Criminal group "TeamPCP" running multi-ecosystem campaign. Operational security failures (commented-out payloads, self-referencing filenames) suggest a skilled but not state-level actor. [source: https://www.endorlabs.com/learn/teampcp-isnt-done] |
| **Stealth** | Extremely stealthy: backdoor hidden in test files and build macros, not in source code visible on GitHub. Release tarballs differed from repository. | Moderately stealthy: code between legitimate blocks, but fork bomb bug and 13-minute version gap left obvious traces. |
| **Ecosystem response** | Industry-wide reckoning about maintainer burnout, single-maintainer dependencies, and tarball vs. git source divergence. | Highlights AI/ML supply chain as a new high-value target. Questions about PyPI account security and transitive dependency risk. |

**Key insight:** XZ Utils was a precision weapon: a likely state actor spent years cultivating trust to plant a surgical backdoor in critical infrastructure. LiteLLM was a smash-and-grab: a criminal group exploited a chain of CI/CD compromises to rapidly deploy a broad credential harvester across the AI ecosystem. XZ was more technically sophisticated; LiteLLM was more operationally impactful in the short term because it targeted credentials directly and hit a package with massive download volume in the fast-moving AI space.

### 8. Broader Implications for AI/ML Supply Chain Security

**High confidence:**

1. **AI/ML packages are high-value targets.** LiteLLM sits at the intersection of every major cloud provider's credentials. A library that proxies API calls to OpenAI, Anthropic, AWS Bedrock, and others naturally has access to the most sensitive keys in any organization. The attacker knew this, targeting cloud credentials, Kubernetes secrets, and crypto wallets specifically.

2. **Transitive dependency risk is amplified in AI tooling.** LiteLLM is pulled in by AI agent frameworks, MCP servers, and LLM orchestration tools. Many developers who were compromised never directly installed it. Specific downstream frameworks identified as affected include **DSPy** (uses LiteLLM as its primary upstream provider interface) and **CrewAI** (depends on it as a fallback mechanism). The AI ecosystem's rapid growth and deep dependency chains create an expanding attack surface. [source: https://news.ycombinator.com/item?id=47501729]

3. **CI/CD pipeline security is the new perimeter.** The attack chain started with Trivy (a security scanner, ironically), moved through npm, Docker Hub, and IDE extensions, and ended at PyPI. Unpinned dependencies in CI/CD pipelines were the entry point. Organizations that pinned their security tooling versions were protected.

4. **PyPI account security remains a systemic risk.** A single compromised publishing token gave the attacker the ability to push malware to 95 million monthly downloaders. This echoes ongoing debates about mandatory 2FA, trusted publishers, and package signing on PyPI.

**Medium confidence:**

5. **The 13-minute gap between 1.82.7 and 1.82.8 suggests the attacker iterated in real-time**, possibly testing whether the proxy_server.py injection was sufficient before escalating to the .pth file approach. This implies active monitoring and adaptation during the attack window.

6. **The fork bomb bug may have been the saving grace.** Without the accidental recursive .pth triggering that crashed machines, the attack could have remained undetected for much longer, silently harvesting credentials across the AI ecosystem.

### 9. Indicators of Compromise

For anyone running LiteLLM in their environment:

| IoC | Type |
|---|---|
| `8a2a05fd8bdc329c8a86d2d08229d167500c01ecad06e40477c49fb0096efdea` | SHA-256 (1.82.7 tarball) |
| `8395c3268d5c5dbae1c7c6d4bb3c318c752ba4608cfcd90eb97ffb94a910eac2` | SHA-256 (1.82.7 wheel) |
| `d39f4e7a218053cce976c91eacf184cf09a6960c731cc9d66d8e1a53406593a5` | SHA-256 (1.82.8 tarball) |
| `d2a0d5f564628773b6af7b9c11f6b86531a875bd2d186d7081ab62748a800ebb` | SHA-256 (1.82.8 wheel) |
| `a0d229be8efcb2f9135e2ad55ba275b76ddcfeb55fa4370e0a522a5bdee0120b` | SHA-256 (compromised proxy_server.py) |
| `71e35aef03099cd1f2d6446734273025a163597de93912df321ef118bf135238` | SHA-256 (litellm_init.pth) |
| `models.litellm.cloud` (`46.151.182.203`) | Exfiltration endpoint |
| `checkmarx.zone` (`83.142.209.11`) | C2 polling domain |
| `scan.aquasecurtiy.org` (`45.148.10.212`) | Trivy exfiltration domain (typosquat) |
| `~/.config/sysmon/sysmon.py` | Persistence script |
| `~/.config/systemd/user/sysmon.service` | Persistence service |
| `/tmp/pglog` or `/tmp/.pg_state` | State files |
| `node-setup-*` pods in `kube-system` | Kubernetes lateral movement |
| `tpcp-docs` repository in victim GitHub accounts | Fallback exfiltration channel |

Detection commands:
```bash
pip show litellm | grep Version
find $(python -c "import site; print(site.getsitepackages()[0])") -name "litellm_init.pth"
ls -la ~/.config/sysmon/
kubectl get pods -n kube-system | grep node-setup
```

Remediation:
```bash
# Purge pip cache to prevent reinstalling cached malicious wheels
pip cache purge
# Remove systemd persistence artifacts
systemctl --user stop sysmon.service
systemctl --user disable sysmon.service
rm -f ~/.config/sysmon/sysmon.py ~/.config/systemd/user/sysmon.service
rm -f /tmp/pglog /tmp/.pg_state
# Verify Docker containers if running LiteLLM
docker exec <container> pip show litellm
```

[source: https://www.endorlabs.com/learn/teampcp-isnt-done, https://safedep.io/malicious-litellm-1-82-8-analysis/, https://github.com/BerriAI/litellm/issues/24512, https://www.sysdig.com/blog/teampcp-expands-supply-chain-compromise-spreads-from-trivy-to-checkmarx-github-actions]

## Sources

1. [FutureSearch: Supply Chain Attack in litellm 1.82.8 on PyPI](https://futuresearch.ai/blog/litellm-pypi-supply-chain-attack/)
2. [Endor Labs: TeamPCP Isn't Done - Threat Actor Behind Trivy and KICS Compromises Now Hits LiteLLM](https://www.endorlabs.com/learn/teampcp-isnt-done)
3. [SafeDep: Malicious litellm 1.82.8 - Credential Theft and Persistent Backdoor Analysis](https://safedep.io/malicious-litellm-1-82-8-analysis/)
4. [GitHub Issue #24518: litellm PyPI package (v1.82.7 + v1.82.8) compromised - full timeline and status](https://github.com/BerriAI/litellm/issues/24518)
5. [GitHub Issue #24512: CRITICAL: Malicious litellm_init.pth in litellm 1.82.8 - credential stealer](https://github.com/BerriAI/litellm/issues/24512)
6. [LWN.net: LiteLLM on PyPI is compromised](https://lwn.net/Articles/1064479/)
7. [Datadog Security Labs: The XZ Utils backdoor (CVE-2024-3094)](https://securitylabs.datadoghq.com/articles/xz-backdoor-cve-2024-3094/)
8. [Sam James (Gentoo): xz-utils backdoor situation (CVE-2024-3094)](https://gist.github.com/thesamesam/223949d5a074ebc3dce9ee78baad9e27)
9. [XDA Developers: A popular Python library just became a backdoor to your entire machine](https://www.xda-developers.com/popular-python-library-backdoor-machine/)
10. [Hacker News Discussion: LiteLLM Python package compromised by supply-chain attack](https://news.ycombinator.com/item?id=47501729)
11. [GitGuardian: Trivy's March Supply Chain Attack Shows Where Secret Exposure Hurts Most](https://blog.gitguardian.com/trivys-march-supply-chain-attack-shows-where-secret-exposure-hurts-most/)
12. [CrowdStrike: From Scanner to Stealer - Inside the trivy-action Supply Chain Compromise](https://www.crowdstrike.com/en-us/blog/from-scanner-to-stealer-inside-the-trivy-action-supply-chain-compromise/)
13. [Docker Blog: Trivy supply chain compromise - what Docker Hub users should know](https://www.docker.com/blog/trivy-supply-chain-compromise-what-docker-hub-users-should-know/)
14. [Wiz: KICS GitHub Action Compromised - TeamPCP Supply Chain Attack](https://www.wiz.io/blog/teampcp-attack-kics-github-action)
15. [Phoenix Security: Trivy Supply Chain Compromise - Again](https://phoenix.security/trivy-supply-chain-compromise-teampcp-weaponised-scanner-ongoing-attack/)
16. [The Hacker News: Trivy Security Scanner GitHub Actions Breached](https://thehackernews.com/2026/03/trivy-security-scanner-github-actions.html)
17. [Security Affairs: 44 Aqua Security repositories defaced after Trivy supply chain breach](https://securityaffairs.com/189856/hacking/44-aqua-security-repositories-defaced-after-trivy-supply-chain-breach.html)
18. [PyPA Advisory Database: PYSEC-2026-2](https://github.com/pypa/advisory-database/blob/main/vulns/litellm/PYSEC-2026-2.yaml)
19. [MLflow Emergency Pin PR #21971](https://github.com/mlflow/mlflow/pull/21971)
20. [Sysdig: TeamPCP Expands Supply Chain Compromise](https://www.sysdig.com/blog/teampcp-expands-supply-chain-compromise-spreads-from-trivy-to-checkmarx-github-actions)
21. [MrCloudBook: litellm 1.82.8 Malicious PyPI Package - Credential Stealer](https://mrcloudbook.com/litellm-1828-malicious-pypi-package-credential-stealer/)
22. [ByteIota: LiteLLM PyPI Attack - 95M Downloads Hit by Malware](https://byteiota.com/litellm-pypi-attack-95m-downloads-hit-by-malware-today/)
23. [Daylight AI: litellm Library and an Expanding Supply Chain Campaign](https://daylight.ai/blog/litellm-library-and-an-expanding-supply-chain-campaign)
24. [Simon Willison: Malicious litellm](https://simonwillison.net/2026/Mar/24/malicious-litellm/)
25. [Awesome Agents: LiteLLM Compromised - Credential Stealer in PyPI Package](https://awesomeagents.ai/news/litellm-supply-chain-compromise-credential-theft/)
26. [CyberInsider: New supply chain attack hits LiteLLM with 95M monthly downloads](https://cyberinsider.com/new-supply-chain-attack-hits-litellm-with-95m-monthly-downloads/)

## Confidence & Gaps

### High Confidence
- The attack mechanism (both .pth file and proxy_server.py injection) is well-documented across multiple independent sources
- TeamPCP attribution is supported by matching tradecraft across five ecosystems
- The credential harvesting scope and three-stage payload architecture are confirmed by independent code analysis (SafeDep, Endor Labs)
- The Trivy CI/CD vector is confirmed by the maintainer on Hacker News: "this originated from the trivy used in our ci/cd"
- Timeline of events is consistent across all sources
- IoCs are verified and consistent

### Medium Confidence
- The "13-minute iteration" interpretation (attacker testing and adapting in real-time) is reasonable inference but not confirmed by the attacker
- Download impact estimates (95M monthly, ~3.6M daily) come from PyPI stats. At least one production environment confirmed compromised via Wiz alert, but total victim count during the attack window remains unknown
- TeamPCP aliases (DeadCatx3, PCPcat, ShellForce, CanisterWorm) reported by Endor Labs, but alias attribution across groups can be uncertain
- DSPy, CrewAI, and MLflow identified as downstream dependents (MLflow confirmed via emergency PR; DSPy and CrewAI from HN community reports)

### Gaps
- **Exact number of affected users/organizations**: No source provides confirmed victim counts
- **BerriAI's detailed post-mortem**: Mandiant engaged, but official incident report not yet published
- **Exfiltration domain still live**: `litellm.cloud` takedown stalled at registrar level. Domain may still be collecting data from compromised systems with active persistence
- **Law enforcement response**: No information found on whether TeamPCP is being actively investigated. User `pwilkin` on #24518 commented "This should be reported to the authorities as well," suggesting no known government involvement
- **CVE assignment**: PYSEC-2026-2 has been assigned (PyPI-specific advisory), but no CVE number has been issued yet. No CISA or NVD advisory found.
- **Identity verification of new maintainer**: Partially resolved via HN cross-reference, but community raised deepfake concerns about photo-based verification. No GPG-signed or official channel verification yet
- **Full scope of GitHub PAT compromise**: The attacker had access to the maintainer's GitHub account (confirmed via activity on `krrishdholakia/blockchain` and creation of `tpcp-docs` repo), but the full extent of actions taken with the PAT is not yet documented
- **No new clean version published**: Latest on PyPI is still 1.82.6; latest GitHub release is v1.82.6.rc.2

## How This Report Was Generated

- Researched using Claude Code (Opus 4.6) with strict anti-hallucination guardrails
- Sources gathered via SearXNG (self-hosted metasearch aggregating Bing, DuckDuckGo, Brave, Startpage, Reddit), then verified by reading each source directly
- Every factual claim is cited inline; confidence levels flagged where evidence varies
- No claims made without a verifiable source; gaps explicitly acknowledged
- This report is actively monitored and updated every 30 minutes via an automated research loop that checks GitHub issues, PyPI, security advisories, and news sources for new developments
