# Use cases for crypto

How we use Desearch in our crypto tools today, and which other parts of the API are worth trying. Endpoints and prices:
[API.md](API.md). X search pitfalls: [X-SEARCH.md](X-SEARCH.md).

As of: 2026-11-25.

## What we use today

| Use case | Endpoints | How |
|---|---|---|
| **Due-diligence reports on crypto projects** | `GET /twitter`, `/twitter/user/posts`, `/web`, `/web/extract` | who writes about the project on X and when, cashtag history of those authors, forum threads (`site:bitcointalk.org`), clean text of docs and blogs. Our main, permanent source of X data. A normal report costs cents. |
| **Alerts on new X posts for watched coins** | `GET /twitter` | every few minutes one query per coin (`$TICKER`, `"name"`, `from:<official account>`) with `sort=Latest`, `count=20` and `since_id:<last seen>`. Empty answers were free in our tests, so you paid only for new posts (the 2026-11-19 docs say an empty search is billed as 10 posts — re-test before relying on it, see [X-SEARCH.md](X-SEARCH.md)). A free source (FxTwitter) is the fallback when Desearch is down or over budget. |
| **Track record of crypto influencers ("callers")** | `/twitter/user/posts`, `GET /twitter` with `from:` and date windows | collect an account's history, detect cashtags locally, then check prices after 24 h / 7 / 30 / 90 days. Neither endpoint alone gives the full history. |
| **Exchange listing announcements** | `/web/extract`, `/twitter/user/posts` | read an exchange's announcements page as markdown (`js=false`) where plain `curl` gets a Cloudflare 403, and the newest posts of the exchange's official X account. |

## Worth trying next

| Idea | Endpoints | Why | Watch out |
|---|---|---|---|
| **Listing and announcement watch without your own poller** | Monitors: `options.x.accounts` (up to 10 exchange or project accounts) + `web` query like `"listed on <exchange>"` | runs every hour, keeps only new items, AI drops noise, signed webhook pings your bot | nothing faster than 1 h, so not for minute-level trading alerts; you still pay each run's searches |
| **New calls from watched influencers** | Monitors with `options.x.accounts`, `min_score: 0` | every new post of up to 10 accounts per monitor, delivered once | hourly at best; for exact call timestamps keep `since_id` polling |
| **Find new influencers to track** | People Search: `POST /desearch/people/search/x` (25 USD / 1000), YouTube creator search (10 USD / 1000) | describe them ("accounts that post buy calls on small-cap tokens"); each profile comes with the posts that matched | billed per profile delivered; set `count` and `max_cost_usd` |
| **Narrative radar** | `GET /twitter/trends` by place (WOEID) | see which crypto topics trend where, then pull top posts behind them | trends are a sample per moment; dedupe repeated pulls |
| **Scam ads promoting a token** | Monitors `meta_ads`, Facebook Ad Library search | paid ads pushing a new token are a red flag; one read (0.003) returns up to 30 ads | public ad data only |
| **News about a token in a date window, with full text** | `POST /search` with a publish-date window and `summary: true` | one call returns pages, their text and a cited summary; 0.003 per default search | the summary is the vendor's AI; keep the source links as evidence |
| **Web + X in one call for a quick brief** | `POST /desearch/ai/search` with `tools: ["web","twitter"]` | one request, `warnings` tells you if a source failed | for evidence-grade research prefer raw results over summaries |
| **Repo buzz for open-source crypto projects** | `GET /twitter` by repo URL and name (Latest + Top), `POST /search` | day-by-day view of who mentions the project's GitHub | Desearch has an open example: https://www.desearch.ai/use-cases/github-repo-mentions-tracker |
| **What investors say about a sector** | `GET /twitter` with `from:` for 10–15 VC / fund accounts and a topic | read their public stance before a pitch or before investing | open example: https://www.desearch.ai/use-cases/vc-twitter-tracker |
| **Reaction to a launch or announcement** | `/twitter/replies/post`, `/twitter/post/retweeters`, quote posts | who amplified it and what replies say | replies endpoints bill a minimum of 10 posts when anything returns |
| **Desearch from your AI agent** | hosted MCP `https://mcp.desearch.ai/mcp` | call search tools straight from Claude Code, Cursor or Gemini CLI; every result carries its cost | give the agent a budget |
| **SN22 network watch** | `GET /network/stats`, `/network/stats/stream` | free live counters of the Bittensor Subnet 22 network behind Desearch | the history endpoint is marked as an estimate |

## What we decided not to use (and why)

- **Reddit Search API:** we already read Reddit threads with comments through our own scraper for free. Desearch's
  Reddit API stays a backup only.
- **YouTube transcripts:** we already have this in our tools.
- **Company Search / Company Enrich and LinkedIn profile search for team checks:** small crypto projects often have
  anonymous teams and no company or CEO to look up, so the value is low for our research.

## Official examples worth reading

Desearch publishes Python examples, many of them useful for market research:
https://www.desearch.ai/use-cases and https://github.com/Desearch-ai/desearch-py-examples — for crypto especially
"stock Twitter ticker pulse", "X keyword monitor", "social listening report", "deep research report with citations",
"daily news digest" (can post to Telegram) and "x-account-analytics" (excludes reposts from engagement rates).

---

Found this useful? Sign up to Desearch with the author's referral link: https://console.desearch.ai/signup?ref=542D2C7D
