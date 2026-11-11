# DailyNews bot

Scheduled pipeline that posts curated science/tech/culture news notes to the Rote account whose OpenKey sits in `~/.rote-toolkit/config.json`. Confirm the rebuild plan with the user before creating cron jobs or posting.

## Schedule

- Three notes per day at fixed times 09:04 / 13:05 / 19:05 Asia/Shanghai.
- Runs as three Hermes cron jobs (one per time), timezone Asia/Shanghai, each loading this skill plus `rote`. Cron prompts must be self-contained: read state files, browse the web, compose, post, update state, and never repeat an already-covered URL.

## Note format (exact)

```
<中文标题>

<中文摘要：150–250 字，有信息量、带观点的重写，不是机翻>

<原文 URL>
```

- Tags: `DailyNews` plus one topic tag (生物 / 历史 / AI / 科学 / 太空 / 物理 / 数学 / 安全 / 健康 / 科技 / 编程 / 文化 …).
- Create with `isPublic: true` and no separate title field — the whole format lives in `content`.
- After posting, add a 🤖 author reaction as the bot marker: `addReaction({ roteid, type: "🤖", metadata: { source: "dailynews-bot" } })`.

## State — `~/.hermes/dailynews/`

- `covered.json`: log of every posted URL (dedupe source). Check it AND search Rote content for the URL before posting; skip duplicates.
- `sources.json`: source pool with categories. The pool is NOT fixed — it grows as the bot browses: each run may add 1–2 newly discovered sources after verifying content quality, and drops nothing without user request.
- Core sources by historical frequency: nautil.us, arstechnica.com, smithsonianmag.com, quantamagazine.org, theguardian.com; frequent long-tail: sciencealert.com, wired.com, bbc.com, spectrum.ieee.org, news.mit.edu, nature.com, theconversation.com, discovermagazine.com.

## Topic selection

Weight topics by historical tag frequency: 生物 highest, then 历史, then AI / 科学 / 太空, then 安全 / 物理 / 数学 / 健康. Prefer fresh items not yet covered; never reuse a URL.

## Gates

- The posting account is whichever OpenKey is in `~/.rote-toolkit/config.json` at run time; do not swap accounts silently.
- Schedule, posting account, 🤖 reaction, and summary style are user-confirmation points — reconfirm before changing any of them.