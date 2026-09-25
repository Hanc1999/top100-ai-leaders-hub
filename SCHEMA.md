# Schema

Every field, what it means, and where it is uncertain.

All human-readable text is bilingual: a field shown as `{en, zh}` is an object
with those two keys. In the CSVs they are flattened to `_en` / `_zh` columns.

---

## `data/items.csv` — one row per item

The flat table. Both panels in one file, because the common question is
"everything about this person, in order", which should be a filter and not a
join.

| Column | Meaning |
|---|---|
| `person_id` | Stable slug, e.g. `yann-lecun`. Joins to `index.csv` and `people/<id>.json`. |
| `person_name_en` | Convenience only — `person_id` is the key. |
| `panel` | `posts` (they produced it) or `news` (it happened to them). |
| `date` | `YYYY-MM-DD`, the item's own date, not when we collected it. |
| `kind` | For `posts`: `x`, `paper`, `podcast`, `blog`, `report`, `talk`, `interview`. For `news`: the severity level (`major`, `notable`, `minor`). |
| `source` | Where it came from — a publisher (`Bloomberg`), an X handle (`@ylecun`), a venue (`arXiv`). |
| `title_en` / `title_zh` | The item's title. For a tweet, the tweet text. |
| `summary_en` / `summary_zh` | **Written by us at judgement time**, not scraped. For `news` this is the event body; for `posts`, a one-line note. Often empty for short items that are their own summary. |
| `url` | The original. Always present. |

## `data/index.csv` — one row per leader

| Column | Meaning |
|---|---|
| `rank` | By `score_total`, recomputed every build. **Can exceed 100** — see below. |
| `id` | The stable slug. |
| `name_en` / `name_zh` | For many Western names the two are identical; that is intentional, not a missing translation. |
| `role_en` / `role_zh` | Current role, hand-verified. |
| `org` | Primary affiliation. |
| `region_en` | Country. |
| `category` | `lab`, `bigtech`, `academia`, `startup`, `infra`, `investor` — a filter, not part of the ranking. |
| `tier` | `t1`–`t6`, a coarse banding of rank. Empty past rank 100. |
| `score_total` | 0–100 composite. See below. |
| `posts` / `news` | Item counts currently on the board for this person. |
| `tags` | `\|`-separated topic tags. |

## `data/people/<id>.json` — the full record

Everything `index.csv` has, plus:

| Field | Meaning |
|---|---|
| `why` | `{en, zh}` — why this person is on the board. Hand-written. |
| `links` | The channels we watch: `[{kind, label, url, t}]`. `kind` is `x`, `blog`, `github`, `hf`, `site`, `scholar`… This is our source registry, published so you can see exactly what we monitor — and tell us what is missing. |
| `posts` | `[{date, kind, title{en,zh}, source, url, note{en,zh}}]`, newest first. |
| `news` | `[{date, level, title{en,zh}, body{en,zh}, source, url}]`, newest first. |
| `score` | The ranking breakdown — see below. |
| `changed`, `was` | Whether this person's role changed at the last revision, and what it was. |
| `seedRank` | Their rank in the original seed list, before our re-ranking. |

## `score` — how the ranking is computed

```
total = 42% attention + 26% centrality + 22% momentum + 10% stature
```

| Sub-score | What it is |
|---|---|
| `attention` | Public attention, driven by Wikipedia pageviews (`pageviews`). |
| `centrality` | How close to the frontier the work is. |
| `momentum` | Recent activity — `events` and `posts` counts feed this. |
| `stature` | Research standing: `citations`, `hIndex`. **`null` for non-researchers**, which is not a zero — the weight is redistributed. |

`wikiEn` / `wikiZh` are the Wikipedia article titles used for pageview lookup.

**Read the ranking as one view, not a fact.** The weights are a choice. They are
published in `meta.json` → `meta.ranking` precisely so you can disagree with
them and recompute.

## People ranked past the board

Every person carries a `rank`, recomputed from `score_total` on **every build**.
The leaderboard is the top 100 of that ordering — `stats.json` gives the cut as
`board` — and **a rank above it is real but not displayed on the site**.

That makes the hundred a daily measurement rather than a membership list:
someone at 101 is inside it tomorrow if their score says so, and someone at 100
can fall out. They keep being collected and published either way; only the
display of the number changes.

For analysis: filter to `rank <= board` to reproduce the leaderboard, and keep
everyone to study what this set of people publishes. `tier` is empty past the
board, because the tiers are bands within it.

## Caveats worth knowing before you analyse

- **Counts are of what is on the board, not of what the person published.** Each
  person's page holds at most 50 items per kind. Someone with 200 tweets shows
  the newest 50. Do not read `posts` as a productivity measure.
- **Absence is ambiguous.** A missing item may never have been collected (a
  channel we do not know about) or may have been judged not substantive. The
  data cannot tell you which; please open an issue if you spot either.
- **`date` is the item's date.** Where a source gave only a day and no time,
  the time is unknown rather than midnight.
- **X coverage is uneven by account**, for historical reasons — some accounts
  were backfilled years deep, others only recently.
- **`source` for news is the publisher**, extracted from the headline where the
  aggregator appended it. A handful may still carry an aggregator name.
