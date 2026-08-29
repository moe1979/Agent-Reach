# 社交媒体 & 社区

小红书、Twitter/X、B站、V2EX、Reddit、Facebook、Instagram。

## 小红书 / XiaoHongShu（多后端）

小红书有三个后端，**先跑 `agent-reach doctor --json` 看 xiaohongshu 的 `active_backend` 是哪个**，再用对应命令组。

### 后端 A：OpenCLI（桌面首选）

```bash
opencli xiaohongshu search "query" -f yaml
opencli xiaohongshu note "NOTE_URL" -f yaml
opencli xiaohongshu comments NOTE_ID -f yaml
opencli xiaohongshu feed -f yaml
opencli xiaohongshu user USER_ID -f yaml
```

> 要求 Chrome 打开且装了 OpenCLI 扩展。OpenCLI 只使用用户已经存在且明确控制的 Chrome 会话；Agent Reach 不替用户登录，也不读取浏览器 Cookie。

### 后端 B：xiaohongshu-mcp（服务器场景）

```bash
agent-reach configure xhs-cookies
mcporter call xiaohongshu.check_login_status --timeout 120000
mcporter call xiaohongshu.search_feeds keyword="query" --timeout 120000
mcporter call xiaohongshu.get_feed_detail feed_id="..." xsec_token="..." --timeout 120000
```

### 后端 C：xhs-cli（存量备选）

```bash
xhs search "query"
xhs read NOTE_ID_OR_URL
xhs comments NOTE_ID_OR_URL
xhs hot
xhs feed
```

## Twitter/X (twitter-cli)

运行任何 `twitter` 命令前必须在同一 Shell 或子进程环境显式提供 `TWITTER_AUTH_TOKEN` 和 `TWITTER_CT0`，不得在日志或命令回显中暴露值。

```bash
twitter feed -n 20
twitter tweet URL_OR_ID
twitter article URL_OR_ID
twitter user-posts @username -n 20
twitter user @username
twitter search "query" -n 10
```

搜索失败时依次：直接重试一次；升级 twitter-cli；换 OpenCLI；最后改用 feed/user-posts 绕路。

## B站 / Bilibili

> 不要用 yt-dlp 读 B站。

```bash
bili search "query" --type video -n 5
bili hot -n 10
bili video BVxxx
opencli bilibili subtitle BVxxx
```

## V2EX

```bash
curl -s "https://www.v2ex.com/api/topics/hot.json" -H "User-Agent: agent-reach/1.0"
curl -s "https://www.v2ex.com/api/topics/show.json?node_name=python&page=1" -H "User-Agent: agent-reach/1.0"
curl -s "https://www.v2ex.com/api/topics/show.json?id=TOPIC_ID" -H "User-Agent: agent-reach/1.0"
curl -s "https://www.v2ex.com/api/replies/show.json?topic_id=TOPIC_ID&page=1" -H "User-Agent: agent-reach/1.0"
```

## Reddit

Reddit بدون login path ندارد. ابتدا `agent-reach doctor --json` را اجرا کن.

```bash
opencli reddit search "query" -f yaml
opencli reddit read POST_ID -f yaml
opencli reddit subreddit LocalLLaMA -f yaml
opencli reddit hot -f yaml

rdt search "query" --limit 10
rdt read POST_ID
rdt sub python --limit 20
```

## Facebook

```bash
opencli facebook search "query" -f yaml
opencli facebook profile zuck -f yaml
opencli facebook feed --limit 10 -f yaml
opencli facebook groups --limit 20 -f yaml
```

## Instagram

```bash
opencli instagram search "query" -f yaml
opencli instagram profile nasa -f yaml
opencli instagram user nasa --limit 12 -f yaml
opencli instagram explore --limit 20 -f yaml
opencli instagram saved --limit 20 -f yaml
```

برای همه پلتفرم‌های login-state فقط از session موجود و صریحاً تحت کنترل کاربر استفاده کن؛ عملیات نوشتن را انجام نده مگر کاربر صریحاً درخواست کند و ابزار مربوطه ایمن و پشتیبانی‌شده باشد.
