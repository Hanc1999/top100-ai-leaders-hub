# Top 100 AI Leaders — News MCP & Open Data

[![Connect the MCP server](https://img.shields.io/badge/MCP-Connect_now-087f5b?style=for-the-badge)](https://top100-ai-leaders.vercel.app/developers)
[![Public service, no API key](https://img.shields.io/badge/Public-No_API_key-222222?style=for-the-badge)](https://top100-ai-leaders.vercel.app/developers)

**Give your AI assistant the latest published AI news and first-hand X posts —
by person, time window, topic or source.**

[Connect in minutes](#connect-your-ai-assistant) · [Browse the news](https://top100-ai-leaders.vercel.app/#/news) · [中文说明](#chinese)

## Connect your AI assistant

**MCP endpoint · Streamable HTTP · public, read-only · no API key**

```text
https://top100-ai-leaders.vercel.app/mcp
```

**Codex — one command:**

```bash
codex mcp add ai100 --url https://top100-ai-leaders.vercel.app/mcp
```

**Cursor — add this to your MCP configuration** ([official setup guide](https://cursor.com/docs/mcp)):

```json
{
  "mcpServers": {
    "ai100": {
      "url": "https://top100-ai-leaders.vercel.app/mcp"
    }
  }
}
```

For other clients, add a remote MCP server using the endpoint above and select
**Streamable HTTP**. No account, token or local server is needed.

Then ask your assistant:

> “What happened in AI today? Include first-hand X posts and link the sources.”
>
> “Find Jensen Huang's news from the last seven days about chips or compute.”
>
> “Show recent funding news, exclude items containing “rumor” or “传闻”, and group them by person.”

| What you want | MCP tools |
|---|---|
| Today's news or a recent time window | `get_today_news`, `get_recent_news` |
| News about one person | `get_person_news` |
| Combine people, topics, kinds, keywords and source filters | `search_news` |
| Find a person or read one item | `list_people`, `get_person`, `get_news_item` |
| Check how fresh the data is | `get_feed_status` |

**[Connection guide, filter reference and live query explorer →](https://top100-ai-leaders.vercel.app/developers)**

The website collects, reviews and publishes about **hourly**. MCP queries use
the latest **published website data**; they do not trigger a fresh crawl.
The files in this GitHub repository are a separate **daily** snapshot.
Check `get_feed_status` for collection and publication timestamps. “Today”
defaults to `Asia/Singapore`; you can choose another timezone.

---

## Download the open dataset

An open, continuously-updated dataset of what the 100 people moving the AI
frontier are actually publishing and what is being written about them.

**Live site:** https://top100-ai-leaders.vercel.app
**Updated:** hourly upstream, synced here daily. Data through 2026-11-16.

Every item is bilingual (English / 简体中文), carries its source URL, and was
individually judged before it entered the set — none of it is raw crawler
output. See [How the data is made](#how-the-data-is-made).

```
100 leaders · 5,204 first-hand outputs · 9,020 dated events
x 2,030 · paper 1,740 · podcast 986 · blog 353 · report/talk/interview 95
```

Exact current counts: [`data/stats.json`](data/stats.json).

---

## What is in here

| Path | What it is |
|---|---|
| [`data/items.csv`](data/items.csv) | **Start here.** One row per item — all 14,224. Opens in Excel or pandas. |
| [`data/index.csv`](data/index.csv) | One row per leader: rank, role, org, region, score, item counts. |
| [`data/people/<id>.json`](data/people) | One file per leader — full record, browsable on GitHub. |
| [`data/meta.json`](data/meta.json) | Ranking weights, tier bands, category definitions. |
| [`SCHEMA.md`](SCHEMA.md) | Every field, what it means, and what it does *not* mean. |

Two panels per person, because they answer different questions:

- **`posts`** — what the person *produced*: tweets, papers, blog posts, podcast
  appearances, talks. Their own output.
- **`news`** — what *happened to or around* them: funding, departures, launches,
  controversies. Dated events with a severity level.

## Using it

```bash
# the flat table
curl -LO https://raw.githubusercontent.com/Hanc1999/top100-ai-leaders-hub/main/data/items.csv
```

```python
import pandas as pd
df = pd.read_csv("data/items.csv", parse_dates=["date"])

# everything Yann LeCun published, newest first
df[(df.person_id == "yann-lecun") & (df.panel == "posts")].sort_values("date", ascending=False)

# which leaders published the most papers this quarter
(df[(df.panel == "posts") & (df.kind == "paper") & (df.date >= "2026-07-01")]
   .person_name_en.value_counts().head(10))
```

```bash
# one person, full record
jq '.posts[] | select(.kind == "paper") | .title.en' data/people/fei-fei-li.json
```

## How the data is made

Collection runs hourly against every channel we have registered for each person
— X, Google News (EN/ZH), Bing News, arXiv, GitHub, Hugging Face, personal
sites and blogs, Apple Podcasts and whole podcast shows, 知乎, 小宇宙, Weibo.

Nothing reaches the dataset automatically. Each candidate item is deduplicated
against everything seen before, filtered by rules that can only *reject*, and
then **judged individually by an LLM agent** against a written guide: is this
really this person's, is it substantive, which panel does it belong in, and what
is the one-line summary in both languages. Rules are never allowed to approve —
only a judgement puts an item in.

The summaries in `summary_en` / `summary_zh` are therefore ours, written at
judgement time. They are not scraped blurbs, and they are not the source's own
description.

**What this means for you:** the set is curated, not exhaustive. An item absent
from here was either never collected (a channel we do not know about) or judged
not substantive. Both are mistakes we want reported — see below.

## Known limits

- **Coverage is per-registered-channel.** If we never found someone's Substack,
  nothing from it is here. Our channel discovery runs weekly and is imperfect.
- **X is the most complete surface, and the most uneven.** Some accounts are
  covered back years; some only recently.
- **Ranking is computed, and opinionated.** `score.total` blends public
  attention, frontier centrality, momentum and research stature. Weights are in
  `meta.json` → `meta.ranking`. Treat it as one view, not a fact.
- **The 100 is a fixed board**, revised deliberately rather than continuously.

## Tell us what is wrong

This is the most useful thing you can do with this repo, and the issue templates
are there to make it quick:

- **[Wrong or misattributed item]** — an item on the wrong person, a bad
  summary, a dead link.
- **[Missing source]** — a channel we should be watching for someone.
- **[Suggest a person]** — someone who belongs on the board, with the case.

If the data is useful to you, a ⭐ genuinely helps — it is how we tell whether
this is worth keeping open.

## License

Data and documentation: **CC BY 4.0** — use it, redistribute it, build on it,
including commercially. Just credit *Top 100 AI Leaders* with a link back.

Items link to their original sources; the linked content belongs to whoever
published it. Titles and URLs are facts about those publications, and the
summaries are our own.

The collection pipeline itself is not open source, so this repo carries no code
— just the data it produces.

---

<a name="chinese"></a>

# Top 100 AI Leaders — 新闻 MCP 与开放数据

**让你的 AI 助手直接查询最新已发布的 AI 新闻与 X 一手消息。**
支持按人物、今天、最近几小时、主题、类型、关键词和来源组合筛选。

**接入地址：** `https://top100-ai-leaders.vercel.app/mcp`

**协议：** Streamable HTTP；公开只读，无需账号或 API key。

Codex 一条命令接入：

```bash
codex mcp add ai100 --url https://top100-ai-leaders.vercel.app/mcp
```

Cursor 配置见[上方快速接入](#connect-your-ai-assistant)；其他支持远程 MCP 的客户端，
添加上述地址并选择 Streamable HTTP 即可。

接入后可以直接问：

> “今天 AI 圈有哪些新消息？包含 X 一手消息，附上原始来源。”
>
> “查黄仁勋最近七天关于芯片和算力的新闻。”
>
> “找最近 24 小时的融资消息，排除含“rumor”或“传闻”的内容，按人物整理。”

**[完整接入教程、筛选参数与在线查询 →](https://top100-ai-leaders.vercel.app/developers)**

网站约每小时采集、审核并发布；MCP 查询的是**最新已发布的网站数据**，
每次调用不会重新抓取上游。此 GitHub 仓库的数据文件每日同步。
可用 `get_feed_status` 检查采集与发布时间；“今天”默认按 `Asia/Singapore` 计算，可指定其他时区。

## 下载开放数据

推动 AI 前沿的 100 个人**实际在发表什么**、以及**外界在报道他们什么**的持续更新数据集。

**站点:** https://top100-ai-leaders.vercel.app　**更新:** 上游每小时,此处每日同步。

所有条目中英双语、附原始链接,并且**每一条都经过单独判定**才会进入数据集 —— 没有任何一条是爬虫的原始输出。

## 里面有什么

| 路径 | 内容 |
|---|---|
| `data/items.csv` | **从这里开始。**一行一条,共 14,224 条,Excel / pandas 直接打开 |
| `data/index.csv` | 一行一个人:排名、职位、机构、地区、评分、条目数 |
| `data/people/<id>.json` | 每人一个文件,完整记录,可在 GitHub 上直接浏览 |
| `data/meta.json` | 排名权重、分层与分类定义 |
| `SCHEMA.md` | 每个字段的含义,以及它**不**代表什么 |

每个人分两栏,回答的是两个不同的问题:

- **`posts`** —— 这个人**产出**了什么:推文、论文、博客、播客、演讲
- **`news`** —— 这个人**身上发生**了什么:融资、离职、发布、争议,带日期和重要性分级

## 数据是怎么来的

每小时对每个人已登记的全部渠道做一次采集 —— X、Google News(中英)、Bing News、
arXiv、GitHub、Hugging Face、个人站与博客、Apple Podcasts 与整档播客、知乎、
小宇宙、微博。

**没有任何条目是自动进来的。**每条候选先与历史全量去重,再经过只能「否决」的规则过滤,
最后由 LLM agent 依据一份成文的判定指南**逐条判定**:这真的是本人的吗、是否有实质内容、
该归入哪一栏、中英文各一句话的摘要是什么。规则永远无权批准,只有判定能让一条进来。

所以 `summary_en` / `summary_zh` 是**我们写的**,在判定时生成,不是抓来的简介,
也不是来源方自己的描述。

**这对你意味着什么:**这是一个策展过的集合,不是穷举。这里没有的条目,要么从未被采集到
(我们不知道那个渠道),要么被判定为无实质内容。**这两种都是我们想知道的错误。**

## 已知局限

- **覆盖以「已登记渠道」为界** —— 没找到某人的 Substack,它的内容就一条都不在这里。
  渠道发现每周跑一次,并不完美。
- **X 是覆盖最全、也最不均匀的面** —— 有的账号回溯到数年前,有的只有近期。
- **排名是算出来的,且带有主观权重** —— `score.total` 由公众关注度、前沿中心性、
  动能、研究声望加权而成,权重见 `meta.json` → `meta.ranking`。它是一种视角,不是事实。
- **这 100 人是一个固定榜单**,按周期审慎修订,而非持续增删。

## 告诉我们哪里错了

这是你能用这个仓库做的最有价值的事,issue 模板就是为了让这件事变快:

- **条目错误** —— 张冠李戴、摘要不准、链接失效
- **缺失渠道** —— 某个人我们应该监控但没有的渠道
- **推荐人选** —— 该上榜但不在榜上的人,附理由

如果这份数据对你有用,**点个 ⭐** 对我们很重要 —— 这是我们判断它是否值得继续开放的唯一信号。

## 许可

数据与文档采用 **CC BY 4.0**:可自由使用、再分发、二次创作,含商业用途,
只需注明 *Top 100 AI Leaders* 并附链接。

条目指向各自的原始来源,被链接内容的版权属于其发布者。标题与 URL 是关于这些发布物的事实,
摘要是我们自己撰写的。

采集管线本身未开源,因此本仓库不含代码,只有它产出的数据。
