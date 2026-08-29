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

# Agent Reach — 互联网能力路由器

15 平台、多后端。**本 skill 存在时必须用它访问这些平台，不要自己发明方案。**

## Hermes on Windows — mandatory execution rules

When this skill runs under Hermes on Windows, treat the Hermes `terminal` tool as the Windows host shell. Do not assume a Linux container just because `code_execution` may be sandboxed.

1. **Use installed commands directly from PATH.** Run `agent-reach`, `mcporter`, `gh`, `yt-dlp`, `curl.exe`, `bili`, `opencli`, etc. by command name first.
2. **Never reconstruct executable paths** from `where`, `Get-Command`, `$LOCALAPPDATA`, `/c/Users/...`, WindowsApps, npm internals, or Python Scripts directories unless a direct PATH invocation has actually failed and the user explicitly asks for path troubleshooting.
3. **Never run Agent Reach through Node or npm.** Do not use `node ... agent-reach`, `npx agent-reach`, `npx --package agent-reach`, or install a second copy as a fallback. The supported command is simply `agent-reach ...`.
4. **Do not translate Windows paths into Unix/MSYS paths.** Keep `C:\...` paths native when a path is genuinely required.
5. **Prefer one command per terminal call while diagnosing.** Do not concatenate unrelated probes such as `node --version + 4 commands`.
6. **For normal web research, do not run `agent-reach doctor` first.** If Exa is already configured, call Exa directly. Use `doctor --json` only when a requested platform has multiple/login-state backends or a direct command fails.
7. **For Exa search on Windows/Hermes use this exact CLI form:**
   `mcporter call exa.web_search_exa query="QUERY" numResults=5`
   Do not use the parenthesized MCP example syntax in PowerShell.
8. **To fetch a result URL with Exa use:**
   `mcporter call exa.web_fetch_exa urls='["URL"]' maxCharacters=5000`
   If JSON quoting becomes problematic, use Jina Reader with `curl.exe` instead of inventing a new invocation.
9. **For web pages on Windows use `curl.exe`, not the PowerShell `curl` alias:**
   `curl.exe -sL "https://r.jina.ai/https://example.com"`
10. **Do not fall back to model memory for a live research request after retrieval failure.** Report the failed command/error, try the documented retry/fallback path, and if no retrieval path works say the research is blocked. Never fabricate URLs, DOI values, papers, vendors, case studies, or quantitative findings.
11. **Success means real non-empty retrieved content.** A command existing, exiting 0, or `doctor` showing a backend is not enough.

### Windows temporary paths

When a command requires a temporary output file, prefer `$env:TEMP` in PowerShell. Do not hard-code `/tmp/` on Windows. For example, use an output template under `$env:TEMP` for yt-dlp.

## 常驻规则（全程适用）

1. **按需体检**：多后端/登录态平台（小红书/Reddit/B站/Twitter/Facebook/Instagram）在任务确实需要该平台时跑 `agent-reach doctor --json`。`active_backend` 有值时按它选命令组；`active_backend: null` 表示 Doctor 为避免触发浏览器 Cookie 读取或远端写入而没有做实时验证，不代表后端不存在。普通 Exa/Web/GitHub/YouTube 请求优先直接调用已配置工具。
2. **声明你在用什么**：开始干活前说一句「使用 agent-reach 的 X 平台 / Y 后端」。
3. **失败按 references 里的重试链处理**，不要瞎猜命令，也不要未经用户要求安装替代版本。
4. **全网调研类任务**：组合可用平台；至少先以 Exa 搜索建立可验证来源，再按需要补 Twitter/Reddit/小红书/B站讨论。
5. **证据边界**：Research 输出中的事实、数字、案例、标准和链接必须来自本次实际检索结果或用户提供材料。检索失败时不得伪造来源。
6. **替用户盯版本**：完成一次较大的调研/多平台任务后，可跑 `agent-reach check-update`。有新版时只在收尾提醒，不要中断当前任务去更新。

## 路由表

| 用户意图 | 分类 | 详细文档 |
|---------|------|---------|
| 网页搜索/代码搜索 | search | [references/search.md](references/search.md) |
| 小红书/推特/B站/V2EX/Reddit/Facebook/Instagram | social | [references/social.md](references/social.md) |
| 招聘/职位/LinkedIn | career | [references/career.md](references/career.md) |
| GitHub/代码 | dev | [references/dev.md](references/dev.md) |
| 网页/文章/RSS | web | [references/web.md](references/web.md) |
| YouTube/B站/播客字幕 | video | [references/video.md](references/video.md) |
| 雪球/股票行情 | finance | [references/finance.md](references/finance.md) |

## 零配置快速命令

```text
# Exa web search (Windows/Hermes-safe)
mcporter call exa.web_search_exa query="query" numResults=5

# Generic webpage reading on Windows
curl.exe -sL "https://r.jina.ai/https://example.com/article"

# GitHub search
gh search repos "query" --sort stars --limit 10

# YouTube metadata/search
yt-dlp --dump-json "URL"
yt-dlp --dump-json "ytsearch5:query"

# V2EX hot topics
curl.exe -sL "https://www.v2ex.com/api/topics/hot.json" -H "User-Agent: agent-reach/1.0"

# Bilibili search
bili search "query" --type video -n 5
```

## 需登录态的平台（按 doctor 的 active_backend 选命令）

Twitter 注意：`agent-reach configure twitter-cookies` 保存的 Cookie 只供 `doctor` 检查显式凭据是否齐全；`doctor` 不执行上游 `twitter status`，也不会设置当前 Shell。直接运行 `twitter` 前，必须在子进程环境中显式提供 `TWITTER_AUTH_TOKEN` 和 `TWITTER_CT0`，不得在日志或命令回显中暴露值。

小红书注意：Agent Reach 不替用户登录，也不读取浏览器 Cookie。OpenCLI 只用用户已有且明确控制的 Chrome 会话；没有现成会话时不要自动登录，改用 Cookie-Editor 手工导出后配置 xiaohongshu-mcp / 存量工具。

```text
# Twitter search
twitter search "query" -n 10

# Reddit
opencli reddit search "query" -f yaml
rdt search "query" --limit 10

# Xiaohongshu
opencli xiaohongshu search "query" -f yaml

# Facebook / Instagram
opencli facebook search "query" -f yaml
opencli facebook groups -f yaml
opencli instagram search "query" -f yaml
opencli instagram user USERNAME -f yaml
```

## 环境检查

```text
agent-reach doctor --json
```

## OpenCLI 适配器发现

路由表没有覆盖用户需要的平台或命令时，先用 `opencli list` 查已有适配器，再用 `opencli <平台> --help` 查看公开命令。发现适配器只证明命令存在，不证明登录态或目标内容可用；仅在用户任务明确需要该平台时执行只读命令，并以实际非空内容验收。

## 工作区规则

不要在 agent workspace 创建无关文件。Windows/Hermes 临时文件使用 `$env:TEMP`；持久数据使用 Agent Reach 自己的配置/数据目录。不要把 Unix `/tmp/` 规则强行用于 Windows。

## 详细文档

根据用户需求，阅读对应的详细文档：

- [搜索工具](references/search.md) — Exa AI 搜索
- [社交媒体](references/social.md) — 小红书, Twitter, B站, V2EX, Reddit, Facebook, Instagram
- [职场招聘](references/career.md) — LinkedIn
- [开发工具](references/dev.md) — GitHub CLI
- [网页阅读](references/web.md) — Jina Reader, RSS
- [视频播客](references/video.md) — YouTube, B站, 小宇宙
- [金融行情](references/finance.md) — 雪球股票行情、搜索、热门内容

## 配置渠道

需要新增 channel 时优先使用已经安装的 `agent-reach` CLI 查看帮助和配置，不要自行安装另一个同名 npm 包。上游安装文档仅用于明确的安装/升级任务。
