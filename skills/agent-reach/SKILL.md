---
name: agent-reach
description: >
  MUST USE when user wants to 调研/research/搜索/search/查/找/look up anything
  on the internet — e.g. 全网调研 X / 帮我调研一下 X / 查一下 X / 搜搜 X /
  看看大家怎么评价 X / X 上有什么讨论 / research this topic。

  Also MUST USE when user mentions any platform or shares any URL/链接:
  小红书/xiaohongshu/xhs, Twitter/推特/X, B站/bilibili, Reddit, Facebook,
  Instagram, V2EX, LinkedIn/领英/招聘/求职/jobs, YouTube, GitHub code search, 小宇宙播客,
  雪球/股票行情, RSS feeds, or any web URL.

  15 platforms, multi-backend routing (OpenCLI / per-platform CLIs / APIs).
  Zero config for 6 channels. Run `agent-reach doctor --json` to see which
  backend serves each platform right now.

  NOT for: 写报告/数据分析/翻译等内容加工（本 skill 只负责从互联网获取内容）；
  发帖/评论/点赞等写操作；已有专门 skill 的平台（先用专门 skill）。

  【路由方式】SKILL.md 包含路由表和常用命令，复杂场景需按需阅读对应分类的 references/*.md。
  分类：search / social (小红书/推特/B站/V2EX/Reddit/Facebook/Instagram) / career(LinkedIn) / dev(github) / web(网页/文章/RSS) / video(YouTube/B站/播客) / finance(雪球/股票)。
metadata:
  homepage: https://github.com/moe1979/Agent-Reach
---

# Agent Reach — Internet capability router

Use this skill to retrieve real external information. Do not substitute model memory for a retrieval task.

## Platform-first routing — mandatory

When the user explicitly names a platform, route to that platform before generic web search.

- **LinkedIn / 领英 / LinkedIn company/profile/jobs/people/posts** → immediately load `references/career.md` and use the configured `linkedin.*` MCP tools through `mcporter`. Do **not** start with `gh`, Exa, Google/Bing scraping, Python, or generic curl.
- GitHub/code → `references/dev.md`.
- YouTube/Bilibili/podcast → `references/video.md`.
- Twitter/Reddit/Facebook/Instagram/Xiaohongshu/V2EX → `references/social.md`.
- Web/RSS/known article URL → `references/web.md`.
- Generic web research with no explicit platform → `references/search.md` and Exa.

For explicit LinkedIn requests, the required first research action is normally one of:

```text
mcporter call linkedin.search_companies keywords="QUERY"
mcporter call linkedin.search_people keywords="QUERY"
mcporter call linkedin.search_jobs keywords="QUERY" location="LOCATION" max_pages=2
mcporter call linkedin.search_posts keywords="QUERY" max_pages=2
mcporter call linkedin.get_person_profile linkedin_username="USERNAME"
mcporter call linkedin.get_company_profile company_name="SLUG"
```

If the exact LinkedIn tool is uncertain, load `references/career.md` or run `mcporter list linkedin`; do not improvise another search method.

## Hermes on Windows — mandatory execution rules

When this skill runs under Hermes on Windows, treat the Hermes `terminal` tool as the Windows host shell. Do not assume a Linux container just because `code_execution` may be sandboxed.

1. Use installed commands directly from PATH: `agent-reach`, `mcporter`, `gh`, `yt-dlp`, `curl.exe`, `bili`, `opencli`, etc.
2. Never reconstruct executable paths from `where`, `Get-Command`, `$LOCALAPPDATA`, `/c/Users/...`, WindowsApps, npm internals, or Python Scripts directories unless direct PATH invocation has actually failed and the user explicitly asks for path troubleshooting.
3. Never run Agent Reach through Node or npm. Do not use `node ... agent-reach`, `npx agent-reach`, `npx --package agent-reach`, or install a second copy as fallback.
4. Do not translate Windows paths into Unix/MSYS paths.
5. Prefer one command per terminal call while diagnosing; do not concatenate unrelated probes.
6. For normal web research, do not run `agent-reach doctor` first. Call the intended backend directly. Use `doctor --json` only for multi-backend/login-state troubleshooting or after a direct command fails.
7. Exa search syntax on Windows/Hermes: `mcporter call exa.web_search_exa query="QUERY" numResults=5`.
8. Exa fetch syntax: `mcporter call exa.web_fetch_exa urls='["URL"]' maxCharacters=5000`. If quoting is problematic, use Jina Reader rather than inventing a new invocation.
9. For web pages on Windows use `curl.exe`, not the PowerShell `curl` alias: `curl.exe -sL "https://r.jina.ai/https://example.com"`.
10. Do not fall back to model memory for a live research request after retrieval failure. Report the failed command/error, follow the documented retry/fallback path, and if no retrieval path works say the research is blocked. Never fabricate URLs, DOI values, papers, vendors, case studies, LinkedIn entities, or quantitative findings.
11. Success means real non-empty retrieved content. A command existing, exiting 0, or `doctor` showing a backend is not enough.
12. Do not create scripts or files merely to manufacture a research answer. Scripts are acceptable only when the user's task genuinely requires programmatic processing of retrieved data.

### Windows temporary paths

When a command requires a temporary output file, prefer `$env:TEMP` in PowerShell. Do not hard-code `/tmp/` on Windows.

## Global operating rules

1. For multi-backend/login-state platforms, run `agent-reach doctor --json` only when the task needs that platform and backend state is uncertain. Generic Exa/Web/GitHub/YouTube requests should call the configured tool directly.
2. State which Agent Reach platform/backend is being used when useful.
3. Follow the retry/fallback chain in the relevant reference. Do not guess commands or install alternatives without need.
4. For broad cross-web research, establish verifiable sources with Exa first, then supplement with social/community platforms when relevant.
5. Facts, numbers, cases, standards, entity URLs, and claims in research output must come from actual retrieval in this run or user-provided material. Retrieval failure does not authorize fabrication.
6. Deduplicate sources/entities before counting them toward a requested number of results.
7. After a substantial multi-platform research task, `agent-reach check-update` may be run; do not interrupt the task to update.

## Routing table

| User intent | Category | Reference |
|---|---|---|
| Generic web search | search | [references/search.md](references/search.md) |
| Xiaohongshu/Twitter/Bilibili/V2EX/Reddit/Facebook/Instagram | social | [references/social.md](references/social.md) |
| LinkedIn/people/companies/jobs/hiring | career | [references/career.md](references/career.md) |
| GitHub/code | dev | [references/dev.md](references/dev.md) |
| Web pages/articles/RSS | web | [references/web.md](references/web.md) |
| YouTube/Bilibili/podcasts | video | [references/video.md](references/video.md) |
| Xueqiu/stocks | finance | [references/finance.md](references/finance.md) |

## Quick commands

```text
# Exa
mcporter call exa.web_search_exa query="query" numResults=5

# LinkedIn company search
mcporter call linkedin.search_companies keywords="query"

# LinkedIn person search
mcporter call linkedin.search_people keywords="query" location="location"

# LinkedIn jobs
mcporter call linkedin.search_jobs keywords="query" location="location" max_pages=2

# Generic webpage
curl.exe -sL "https://r.jina.ai/https://example.com/article"

# GitHub
gh search repos "query" --sort stars --limit 10

# YouTube
yt-dlp --dump-json "URL"
yt-dlp --dump-json "ytsearch5:query"

# V2EX
curl.exe -sL "https://www.v2ex.com/api/topics/hot.json" -H "User-Agent: agent-reach/1.0"

# Bilibili
bili search "query" --type video -n 5
```

## Login-state platforms

Twitter: credentials configured through Agent Reach do not automatically populate every shell. Follow `references/social.md`; never expose credential values in logs or command echo.

Xiaohongshu: Agent Reach must not log the user in or silently read browser cookies. Follow the explicit user-controlled session/Cookie-Editor paths in `references/social.md`.

LinkedIn: prefer the explicitly configured persistent `mcp-server-linkedin` session through `mcporter`; do not auto-import another browser session when profile isolation is configured. Default to read/search tools. Connection requests and messages are write operations and require explicit user intent.

## Environment check

```text
agent-reach doctor --json
```

## Workspace rule

Do not create unrelated files in the agent workspace. On Windows/Hermes use `$env:TEMP` for temporary output. Use Agent Reach's own persistent configuration/data directories for persistent state.

## Detailed references

- [Search](references/search.md) — Exa
- [Social/community](references/social.md) — Xiaohongshu, Twitter, Bilibili, V2EX, Reddit, Facebook, Instagram
- [LinkedIn/career](references/career.md) — LinkedIn MCP
- [Development](references/dev.md) — GitHub CLI
- [Web/RSS](references/web.md) — Jina Reader, RSS
- [Video/podcast](references/video.md) — YouTube, Bilibili, Xiaoyuzhou
- [Finance](references/finance.md) — Xueqiu

## Configuration

When a channel needs configuration, prefer the already-installed `agent-reach` CLI and the documented reference for that platform. Do not install a different same-named npm package as a workaround.
