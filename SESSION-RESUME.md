# LiteLLM Research Monitor - Session Resume

## What's Running
- **Hourly research loop** monitoring the LiteLLM PyPI supply chain attack (March 24, 2026)
- Cron job was session-only, needs to be re-created with `/loop 1h` or CronCreate
- Uses `/research` agent, pushes to BOTH repos on new findings

## Repos
- **Working copy**: `/mnt/c/Users/pster/ai-cli-workspace/research-reports/litellm-pypi-supply-chain-attack.md`
- **Public repo**: `pete-builds/research-reports` (cloned at `/tmp/research-reports/` - needs re-clone)
- **Workspace repo**: `pete-builds/ai-cli-workspace`

## Current Article State (as of 2:10 PM ET, March 24)
- 26 sources, 260+ lines
- Latest findings: Mandiant engaged, CircleCI confirmed as CI platform, domain takedown stalled
- Line ranges for targeted edits: header ~9, Section 1 ~13-22, Section 2 ~23-57, Timeline ~58-87, Discovery ~89-92, Response ~94-105, TeamPCP ~107-121, XZ comparison ~123-139, Implications ~141-158, IoCs ~160-200, Sources ~202-228, Confidence ~230-255, Methodology ~257-262

## Cron Loop Prompt
Re-create with this (hourly, uses /research, targeted edits, pushes both repos):

```
Use /research to investigate the latest developments on the LiteLLM PyPI supply chain attack (March 24, 2026). Focus on:
1. GitHub issues #24518 and #24512 at https://github.com/BerriAI/litellm
2. New releases or PyPI status changes
3. CVE assignments, CISA advisories
4. New security firm reports
5. Community discussion
6. Law enforcement updates

If NO new info, say "No new findings this cycle." and skip edits.

If new info found, apply TARGETED edits to /mnt/c/Users/pster/ai-cli-workspace/research-reports/litellm-pypi-supply-chain-attack.md:
- Do NOT read the entire file. Use offset/limit.
- Surgical Edit calls only. No full section rewrites.
- NEVER append "Update (XX:XX):" blocks. Integrate naturally.
- Timeline table is the ONLY place for chronological entries.
- Update "Last updated" header with: TZ='America/New_York' date '+%I:%M %p ET'
- Use Eastern Time (ET) for all timestamps.

Push to BOTH repos:
1. git -C /mnt/c/Users/pster/ai-cli-workspace add research-reports/litellm-pypi-supply-chain-attack.md && git -C /mnt/c/Users/pster/ai-cli-workspace commit -m "Update LiteLLM research - [description]" && git -C /mnt/c/Users/pster/ai-cli-workspace push origin main
2. cp /mnt/c/Users/pster/ai-cli-workspace/research-reports/litellm-pypi-supply-chain-attack.md /tmp/research-reports/ && cd /tmp/research-reports && git add . && git commit -m "Update LiteLLM research - [description]" && git push origin main
```

## TODO for Next Session
1. Re-clone `/tmp/research-reports/` (it's a tmpdir): `git clone https://github.com/pete-builds/research-reports.git /tmp/research-reports`
2. Re-create the hourly cron loop using the prompt above
3. Use `mcp__threatintel__lookup_ioc` to check these IoCs against Abuse.ch/OTX:
   - `models.litellm.cloud` / `46.151.182.203`
   - `checkmarx.zone` / `83.142.209.11`
   - `scan.aquasecurtiy.org` / `45.148.10.212`
4. Use `mcp__threatintel__search_threats` for "TeamPCP" and "litellm"
5. Add any threat intel findings to the article
