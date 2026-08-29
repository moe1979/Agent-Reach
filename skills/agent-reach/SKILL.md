---
name: agent-reach
description: >
  Internet retrieval skill for research/search/lookups across LinkedIn, GitHub, YouTube,
  Reddit, Twitter/X, Facebook, Instagram, Xiaohongshu, Bilibili, V2EX, RSS and web URLs.
  MUST USE for explicit platform requests and live/current internet research. For LinkedIn,
  companies, people, jobs, hiring, or LinkedIn posts, use the configured LinkedIn MCP via
  mcporter before generic web search. Do not fabricate sources or pad requested result counts.
metadata:
  homepage: https://github.com/moe1979/Agent-Reach
---

# Agent Reach — Internet capability router

Use this skill to retrieve real external information. Do not substitute model memory for a retrieval task.

## FIRST ACTION gate — mandatory

Before executing any terminal, browser, Python/code, file-write, GitHub, or generic web action, classify the request:

1. If it explicitly names **LinkedIn**, the FIRST retrieval command must be an appropriate `mcporter call linkedin.*` command. If the exact LinkedIn tool is uncertain, the only allowed preliminary command is `mcporter list linkedin` or loading `references/career.md`.
2. If it explicitly names another supported platform, load/use that platform reference before generic web search.
3. If it asks for generic web research without naming a platform, use Exa.

For an explicit LinkedIn request, **do not issue curl to linkedin.com, do not run `gh auth status`, do not use Python/code execution, do not use Google/Bing, and do not create a helper script before the first LinkedIn MCP call.**

There is **no `scripts/search.js`** in this Hermes skill. Never attempt to run `node .../agent-reach/scripts/search.js` or invent any other missing script. Use the direct commands documented here and in `references/*.md`.

## Platform-first routing

- LinkedIn / companies / profiles / people / jobs / hiring / LinkedIn posts → load `references/career.md`; use `linkedin.*` MCP via `mcporter`.
- GitHub/code → `references/dev.md`.
- YouTube/Bilibili/podcast → `references/video.md`.
- Twitter/Reddit/Facebook/Instagram/Xiaohongshu/V2EX → `references/social.md`.
- Web/RSS/known article URL → `references/web.md`.
- Generic web research with no explicit platform → `references/search.md` and Exa.

### LinkedIn direct commands

```text
mcporter call linkedin.search_companies keywords="QUERY"
mcporter call linkedin.search_people keywords="QUERY"
mcporter call linkedin.search_jobs keywords="QUERY" location="LOCATION" max_pages=2
mcporter call linkedin.search_posts keywords="QUERY" max_pages=2
mcporter call linkedin.get_person_profile linkedin_username="USERNAME"
mcporter call linkedin.get_company_profile company_name="SLUG"
```

Do not add unsupported parameters. If a LinkedIn call rejects a parameter, run `mcporter list linkedin`, read the signature, and retry once with supported parameters.

## Hermes on Windows — mandatory execution rules

When this skill runs under Hermes on Windows, treat Hermes `terminal` as the Windows host shell. `code_execution` may be sandboxed and must not be used as a substitute for host CLI tools.

1. Use installed commands directly from PATH: `agent-reach`, `mcporter`, `gh`, `yt-dlp`, `curl.exe`, `bili`, `opencli`.
2. Never reconstruct executable paths from `where`, `Get-Command`, `$LOCALAPPDATA`, `/c/Users/...`, WindowsApps, npm internals, or Python Scripts directories unless direct PATH invocation failed and the user explicitly asks for path troubleshooting.
3. Never run Agent Reach through Node/npm/npx and never install a second copy as fallback.
4. Do not translate Windows paths into Unix/MSYS paths.
5. Prefer one command per terminal call while diagnosing.
6. Do not run `agent-reach doctor` before ordinary Exa, LinkedIn, GitHub or YouTube requests. Use it for multi-backend/login-state troubleshooting or after direct failure.
7. Exa: `mcporter call exa.web_search_exa query="QUERY" numResults=5`.
8. Exa fetch: `mcporter call exa.web_fetch_exa urls='["URL"]' maxCharacters=5000`.
9. Generic webpage on Windows: `curl.exe -sL "https://r.jina.ai/https://example.com"`.
10. Do not fall back to model memory after live retrieval failure. Follow documented retry/fallback, then report blocked if retrieval cannot be completed.
11. Success requires real non-empty retrieved content.
12. Do not create scripts/files merely to manufacture a research answer.

## Evidence Gate — mandatory for research lists

When the user asks for N entities matching multiple criteria, each entity must be supported by retrieved evidence for **every material criterion** before it counts toward N.

Example: "AI predictive maintenance for medical equipment" requires evidence for both the medical-equipment/imaging domain and predictive-maintenance/equipment-health capability. AI + healthcare alone is insufficient. Predictive maintenance for automotive/industrial equipment alone is insufficient.

- Deduplicate by canonical URL/identifier.
- Never pad a list to reach N.
- If only 2 of 5 requested entities are verified, return 2 verified entities and say the remaining 3 could not be verified from the retrieved evidence.
- Adjacent candidates may be listed separately as "Adjacent / needs verification" but must not count toward the verified total.
- Never invent or infer URLs, market share, installed base, customer counts, accuracy, downtime reduction, rankings, or other quantitative claims.
- "What they do" and "Why relevant" must be grounded in retrieved content from this run.

## Source discipline

For live research, every factual source/entity claim must be traceable to actual tool output from the current task or user-provided material. A failed tool call does not authorize memory-based completion. Do not state that LinkedIn was searched or verified unless a `linkedin.*` MCP call returned content.

## Routing table

| User intent | Reference / primary backend |
|---|---|
| LinkedIn/people/companies/jobs/hiring | `references/career.md` / LinkedIn MCP |
| Generic web search | `references/search.md` / Exa |
| GitHub/code | `references/dev.md` / gh CLI |
| YouTube/Bilibili/podcasts | `references/video.md` |
| Twitter/Reddit/Facebook/Instagram/Xiaohongshu/V2EX | `references/social.md` |
| Web pages/articles/RSS | `references/web.md` |
| Xueqiu/stocks | `references/finance.md` |

## Quick commands

```text
mcporter call exa.web_search_exa query="query" numResults=5
mcporter call linkedin.search_companies keywords="query"
mcporter call linkedin.search_people keywords="query" location="location"
mcporter call linkedin.search_jobs keywords="query" location="location" max_pages=2
curl.exe -sL "https://r.jina.ai/https://example.com/article"
gh search repos "query" --sort stars --limit 10
yt-dlp --dump-json "URL"
yt-dlp --dump-json "ytsearch5:query"
curl.exe -sL "https://www.v2ex.com/api/topics/hot.json" -H "User-Agent: agent-reach/1.0"
bili search "query" --type video -n 5
```

## Login-state platforms

LinkedIn: prefer the explicitly configured persistent `mcp-server-linkedin` session through `mcporter`; do not auto-import another browser session when profile isolation is configured. Default to read/search tools. Connection requests and messages require explicit user intent.

Twitter/Xiaohongshu/Reddit/Facebook/Instagram: follow `references/social.md` and preserve user-controlled authentication boundaries.

## Windows temporary paths

Use `$env:TEMP` for temporary output. Do not hard-code `/tmp/` on Windows.

## Environment check

```text
agent-reach doctor --json
```

Use only when backend state troubleshooting is actually needed.

## Detailed references

- [LinkedIn/career](references/career.md)
- [Search](references/search.md)
- [Development](references/dev.md)
- [Web/RSS](references/web.md)
- [Video/podcast](references/video.md)
- [Social/community](references/social.md)
- [Finance](references/finance.md)

## Configuration

When a channel needs configuration, prefer the already-installed `agent-reach` CLI and the documented reference. Do not install a different same-named package as a workaround.
