# Desearch API — all API families

What Desearch offers, endpoint by endpoint, with prices. Source: Desearch docs (`llms.txt`, `llms-full.txt`, reviewed
by Desearch on 2026-11-19), the runtime OpenAPI spec (`https://api.desearch.ai/openapi.json`, 92 operations) and the
pricing page, all read on 2026-11-25. Everything here is the vendor's description unless marked **measured**; our
measurements of X search are in [X-SEARCH.md](X-SEARCH.md).

## Basics

- **What it is:** live AI, web, X and social search for AI agents. One API key for every source. The web index is built
  by the Bittensor network, Subnet 22 (SN22); Desearch runs the API on top of it.
- **Base URL:** `https://api.desearch.ai`
- **Auth:** `Authorization: <your API key>` (no `Bearer` prefix). Keys: https://console.desearch.ai/
- **User-Agent:** send one that names your app. Python `urllib`'s default (`Python-urllib/3.x`) is blocked at the edge:
  `403` with the body `error code: 1010`. That is not a key error and is not billed. `curl`, `requests`, `httpx` and Node
  `fetch` are fine.
- **Cost of every call:** billable responses carry `X-Desearch-Cost-Usd`, `X-Desearch-Usage-Count`,
  `X-Desearch-Service` and `X-Desearch-Currency`. JSON object bodies may also have `cost_usd`, `usage_count`,
  `service`, `currency`. Arrays, text and streams have the cost only in headers. Log the header and your program can
  track its own spend.
- **Paging on social lists:** many list endpoints return one page plus an `x-desearch-next-cursor` response header;
  send it back as `cursor`.
- **Start credit:** 5 USD after you attach a card in the Console, then pay as you go. No subscriptions.

## Pricing (USD)

Reference rates from 2026-05-07 unless dated otherwise. The Console shows your account's rates.

| Product | Endpoint | Price | Notes |
|---|---|---|---|
| Search | `POST /search` | 0.30 / 1000 results (since 2026-11-18) | at least 10 results billed, so a default search costs 0.003; no results = free; summary free |
| AI Search | `POST /desearch/ai/search` | 0.40 / 1000 items | |
| Web Search (SERP) | `GET /web` | 1.00 / 1000 searches | metered as 0.10 / 100 |
| Extract | `GET /web/extract` | 0.50 / 1000 pages | **measured** 0.0005 per page with `js=false` |
| X Search | `GET /twitter` and other `/twitter/*` | 0.15 / 1000 posts | docs 2026-11-19: minimum 10 posts per search, empty result included; our earlier measurements found no minimum — see [X-SEARCH.md](X-SEARCH.md) |
| X profiles | | 0.15 / 1000 users | |
| X Profile Search | `POST /desearch/people/search/x` | 25.00 / 1000 profiles | per qualified profile delivered |
| YouTube Creator Search | `POST /desearch/people/search/youtube` | 10.00 / 1000 creators | per result delivered |
| LinkedIn Profile Search | `POST /desearch/people/search/linkedin` | 15.00 / 1000 profiles | per result delivered |
| TikTok Creator Search | `POST /desearch/people/search/tiktok` | 15.00 / 1000 creators | per result delivered |
| Company Search | `POST /desearch/companies/search` | 45.00 / 1000 companies | since 2026-10-28; per company delivered |
| Company Enrich | `POST /desearch/companies/enrich` | 50.00 / 1000 companies | per company delivered |
| Monitors | `/desearch/monitors` | searches at list price + AI screening 0.05 / 1000 items | TikTok, Instagram, Facebook, Meta ads reads 3.00 / 1000 (0.003 per account, hashtag or page per run) |
| Reddit, YouTube, Threads, TikTok, Instagram, Facebook endpoints | `/desearch/<platform>/*` | billed per request, empty pages included | the rate is not on the public pricing page; check the Console. 404, 422 and 5xx are not charged |
| Network stats | `/network/stats*` | free | no API key needed |

## API families and endpoints

### 1. Search (web index)

| Endpoint | What it does |
|---|---|
| `POST /search` | ranked web pages from the Desearch index, each with the passage that best matches. Body: `query`, `mode` (`fast` / `balanced`, default balanced), `count` (default 10, max 20), `start_date` / `end_date` (publish date), `highlights` (default true), `page_text` (full page text, **default false**), `summary` (short answer with numbered `citations` — `index`, `url`, `title` — at no extra charge, default false), `include_domains`, `exclude_domains` |

Quick start:

```bash
curl -X POST 'https://api.desearch.ai/search' \
  -H "Authorization: $DESEARCH_API_KEY" -H 'Content-Type: application/json' \
  -d '{"query": "latest Bittensor subnet 22 updates", "count": 10, "summary": true}'
```

### 2. AI Search

| Endpoint | What it does |
|---|---|
| `POST /desearch/ai/search` | X posts and web results together, optional cited summary. Pick sources with `tools` (`web`, `twitter`); web results land in `search`, X posts in `tweets`; a `warnings` array says if one source came back empty or did not answer. Console offers Fast, Balanced and Deep modes |
| `POST /desearch/ai/search/links/twitter` | X post links for a prompt |
| `POST /desearch/ai/search/links/web` | deprecated, use `POST /search` |

### 3. Web Search

| Endpoint | What it does |
|---|---|
| `GET /web` | SERP-style results (`title`, `snippet`, `link`, `highlights`, `text`); `query` and `start` (0–100), up to 20 results per call |
| `GET /web/extract` | clean page content of a URL. Params: `format` (`text` default, `markdown`, `html`), `js` (default `false`), `wait`, `full_page`; a URL without `https://` is accepted; an unfetchable URL returns a plain "invalid URL" answer |
| `GET /web/crawl` | deprecated alias of Extract |

### 4. X (Twitter) Search

| Endpoint | What it does |
|---|---|
| `GET /twitter` | post search with X operators (`from:`, `$cashtag`, `since_id:`, `min_faves:`, `lang:` …) and parameters `user`, `start_date` / `end_date` (day or ISO 8601 date-time), `lang`, `verified`, `blue_verified`, `is_quote`, `is_video`, `is_image`, `min_retweets`, `min_replies`, `min_likes`; `count` 1–100 (default 20); sort Top or Latest; no cursor |
| `GET /twitter/user/posts` | an account's timeline with `cursor` paging (~780 newest posts, **measured**); docs: up to 60 posts per page |
| `GET /twitter/post/user` | search one account's posts, ranked by relevance, not time |
| `GET /twitter/post` | one post by ID |
| `GET /twitter/urls` | posts by URLs; each post billed once, unreadable URLs left out and free |
| `GET /twitter/post/retweeters` | who reposted a post (`cursor` paging) |
| `GET /twitter/replies` | an account's tweets and replies, newest first; `count` 1–100, `query`, `cursor` |
| `GET /twitter/replies/post` | replies to a post, newest first; `count` 1–100, `query`, `cursor` |
| `GET /twitter/trends` | trending topics for a place by WOEID (`woeid`, `count` default 30) |

X results include the post card (since 2026-11). Behaviour, pitfalls and real costs: [X-SEARCH.md](X-SEARCH.md).

### 5. Reddit Search

| Endpoint | What it does |
|---|---|
| `POST /desearch/reddit/search` | public posts across all subreddits by keyword |
| `GET /desearch/reddit/subreddit/{name}` | subreddit info: description, rules, weekly active users (name is case-sensitive) |
| `GET /desearch/reddit/subreddit/{name}/posts` | subreddit posts with Reddit's sort and time filters |
| `GET /desearch/reddit/post/{id}` | one post by ID or URL, without comments |
| `GET /desearch/reddit/comments/{post_id}` | the comment tree of a post |

The old `reddit_search` field of AI Search is gone; this family replaces it.

### 6. YouTube Search

| Endpoint | What it does |
|---|---|
| `POST /desearch/youtube/search` | videos, Shorts, live streams, channels and playlists by keyword |
| `GET /desearch/youtube/video/{id}` | video metadata: views, likes, comments, chapters, caption languages |
| `GET /desearch/youtube/transcript/{video_id}` | captions as plain text and timestamped segments (no captions = 404, free) |
| `GET /desearch/youtube/comments/{video_id}` | top-level comments |
| `GET /desearch/youtube/channel/{handle}` | channel: subscribers, description, country, links |
| `GET /desearch/youtube/channel/{handle}/videos` | channel uploads, latest or most popular |

### 7. Threads Search

| Endpoint | What it does |
|---|---|
| `POST /desearch/threads/search` | posts by keyword (one page, usually 10–20 posts) |
| `POST /desearch/threads/search/users` | accounts by username or name |
| `GET /desearch/threads/profile/{username}` | profile: bio, followers, verification, links |
| `GET /desearch/threads/profile/{username}/posts` | latest 20–30 public posts (no further pages) |
| `GET /desearch/threads/post/{id}` | a post with its replies |

### 8. TikTok, Instagram, Facebook

Public data only: no login, posting, messaging or private surfaces; no watermark-free downloads.

| Platform | Endpoints |
|---|---|
| TikTok | `POST /desearch/tiktok/search`; `GET /desearch/tiktok/post/{id}`, `/profile/{username}`, `/profile/{username}/posts`, `/comments/{video_id}`, `/hashtag/{tag}`, `/trending` |
| Instagram | `POST /desearch/instagram/search`; `GET /desearch/instagram/media/{id}`, `/profile/{username}`, `/profile/{username}/posts`, `/comments/{shortcode}`, `/hashtag/{tag}` |
| Facebook | `POST /desearch/facebook/search`; `GET /desearch/facebook/post/{id}`, `/profile/{profile_id}`, `/profile/{profile_id}/posts`, `/comments/{post_id}`, `/hashtag/{tag}`. Focus: public pages and the Meta Ad Library |

A trending feed is a sample: Desearch's own test found back-to-back TikTok trending pulls for one region shared only
0–2 posts. Deduplicate repeated pulls before comparing.

### 9. Social post engagement and creator stats

| Endpoint | What it does |
|---|---|
| `GET /desearch/social/post` | one TikTok, Instagram, Facebook or YouTube post by link: views, likes, reactions, comments, shares, saves, engagement rate, in one shape for every platform |
| `GET /desearch/social/creator` | one creator by link or handle: followers, posts, category, location, public contacts, engagement over latest posts |

### 10. People Search

Describe people in plain words; get scored **profiles**, not posts.

| Endpoint | What it does |
|---|---|
| `POST /desearch/people/search/{x,youtube,linkedin,tiktok,instagram,facebook}` | starts a background task (usually 1–5 min), returns `task_id` |
| `GET /desearch/people/tasks`, `/tasks/{task_id}`, `/tasks/{task_id}/results`, `/tasks/{task_id}/export` | status, results (page with `next_cursor`), CSV / JSON Lines export |
| `POST /desearch/people/tasks/{task_id}/pause`, `/resume`, `/cancel` | control a task |
| `GET /desearch/people/profiles`, `/profiles/export` | library of everyone your searches found |

- Each result has `score.criteria_checks`: per criterion `met`, `not_met` or `unknown`, with the quote and source URL
  behind a `met`; `score.fully_verified` when all are met.
- Billing: reserves `count` people, bills the qualified ones delivered (min. 1 if anyone is found), returns the rest.
  Without `max_cost_usd` a search that finds nobody is free. Tasks, results, exports and the library are free.
- Send an `Idempotency-Key` header: a repeat returns the same task and is not charged again. Optional `metadata`
  labels (up to 10 keys) let you filter your tasks later. Results stay 30 days.
- Instagram and Facebook creator search leave out business accounts and pages; filters: `min_followers`,
  `max_followers`, `verified_only`, `require_contact`, `countries`, `min_engagement_rate`, `keywords`, `exclude_keywords`.

### 11. Company Search

| Endpoint | What it does |
|---|---|
| `POST /desearch/companies/search` | companies from a plain-language description, each with a fit score |
| `POST /desearch/companies/enrich` | companies for the domains you send |
| `GET /desearch/companies/tasks`, `/tasks/{task_id}`, `/results`, `/export`; `POST …/pause`, `/resume`, `/cancel` | same task model as People Search |

- Public data only: the company's own site, public web results, public LinkedIn profiles of leaders. Every contact
  keeps its `source_url`; an inferred e-mail is marked `source: inferred` with a `confidence`; e-mails carry
  `verification.status`.
- `filters.require` (`email`, `phone`, `linkedin`, `leader`, `leader_email`) keeps only companies that have them; the
  rest are not delivered and not billed. `roles` picks leaders (default CEO, founder, CTO, CFO).
- `max_cost_usd` (up to 100) bounds a task; a task can continue an earlier one without returning the same companies.

### 12. Monitors

A saved search that reruns on a schedule and delivers **only new** items.

| Endpoint | What it does |
|---|---|
| `POST /desearch/monitors` | create: `query`, up to 5 `criteria`, `sources`, `interval`; first run starts at once (`run_now`) |
| `POST /desearch/monitors/estimate` | cost estimate before you create one |
| `GET /desearch/monitors`, `GET/PATCH/DELETE /desearch/monitors/{id}` | list, read, update / pause / resume, delete |
| `POST /desearch/monitors/{id}/trigger` | run now |
| `GET /desearch/monitors/{id}/runs`, `/runs/{run_id}`, `/items` | runs and new items (newest first, `next_cursor`) |
| `POST /desearch/monitors/{id}/webhook/test`, `/webhook/rotate-secret` | test the webhook, rotate its secret |

- **Sources:** `web`, `x`, `reddit`, `youtube`, `threads`, `tiktok`, `instagram`, `facebook`, `meta_ads`.
- **Interval:** `1h`, `3h`, `6h`, `12h`, `1d`, `7d`. Nothing faster than one hour.
- **Watch accounts instead of searching:** up to 10 per source — `options.x.accounts`, `options.reddit.subreddits`,
  `options.tiktok.accounts`, `options.threads.accounts`, `options.instagram.accounts`, `options.facebook.pages`; plus up
  to 10 TikTok / Instagram hashtags. `min_score: 0` keeps every new post of a watched account.
- **Screening:** every new item is scored by AI (`relevant`, `new_info`, your criteria); kept at `min_score`
  (default 60).
- **Meta ads:** `meta_ads` searches the Meta Ad Library by keyword, 10/20/30 ads per run for one read (0.003).
- **Delivery:** `GET …/items` and webhooks `monitor.items.found`, `monitor.run.completed`, `monitor.run.failed`. The
  webhook must be public HTTPS (no redirects) and carries a `Desearch-Signature` header; verify it with the
  `webhook_secret` (`whsec_…`, shown once).
- **Money:** a run holds `max_run_cost_usd` and is billed what it used. Creating, reading, changing, deleting,
  estimating and webhook tests are free. When the balance runs out the monitor pauses
  (`paused_reason: insufficient_credits`). `409` = too many monitors, or a trigger while a run is queued.

### 13. Network stats (SN22)

| Endpoint | What it does |
|---|---|
| `GET /network/stats` | latest public counters of the Desearch network and their per-second growth |
| `GET /network/stats/history` | one-second timeline of the last 20 minutes (marked as an estimate) |
| `GET /network/stats/stream` | Server-Sent Events, one snapshot per second; `Last-Event-ID` replays missed snapshots from the last 20 minutes |

No API key needed. SN22 also publishes its service map on-chain, pointing to https://www.desearch.ai/services.json.

## Errors

- `401`: missing, invalid or revoked key.
- `402`: no funds (card auto top-up exists).
- `403` + `error code: 1010`: blocked User-Agent, not billed (see Basics). Other `403`: token or source rejected.
- `404`: not found; on most social endpoints not charged (YouTube channel 404 is billed as one request).
- `422`: bad parameter, enum or body; not charged.
- `429`: rate limit; back off exponentially. Limits are per key in minute, day and month windows, visible only in the
  Console.
- `5xx`: retry once, then check https://status.desearch.ai. `503` on X endpoints is not charged.

## SDKs, MCP and integrations

- **Python:** `pip install desearch-py` → `from desearch_py import Desearch`.
- **JavaScript:** `npm install desearch-js` (Node 20.18.1+).
- **Go:** `desearch-go` v0.1.0, generated from a pinned OpenAPI snapshot: https://github.com/Desearch-ai/desearch-go
- **MCP server:** hosted at `https://mcp.desearch.ai/mcp`; package `desearch-mcp-server` on npm and the MCP Registry.
  Works with Claude Code, Cursor, VS Code, Gemini CLI (extension), Raycast, Meta Muse (OAuth) and many MCP gateways.
  Every billed tool result carries cost and usage metadata. Source: https://github.com/Desearch-ai/mcp-desearch
- **Plugin bundle** for Claude Code, Cursor and Grok, **txtai adapter** (`desearch-txtai`) and **Semantic Kernel
  connector** (`desearch-semantic-kernel`) on https://github.com/Desearch-ai
- **Examples:** https://github.com/Desearch-ai/desearch-py-examples and https://www.desearch.ai/use-cases
- **Demos:** https://www.desearch.ai/demos (e.g. web search, Reddit search)

## Privacy and compliance

- SOC 2 Type II (under NDA), GDPR, DPA, zero data retention on request, controls aligned with the HIPAA Security Rule:
  https://www.desearch.ai/security
- Queries are usage data kept 12 months (Privacy Policy 2026-10-19), so `from:<account>` queries tell the provider
  which accounts you study. Zero retention: ask support@desearch.ai.
- Website owners: the crawler is DesearchBot and obeys `robots.txt`, crawl delay and `noindex`:
  https://www.desearch.ai/crawler

## Sources

- Docs index for LLMs: https://www.desearch.ai/llms.txt and https://www.desearch.ai/llms-full.txt
- API reference: https://www.desearch.ai/docs/api-reference
- OpenAPI: https://api.desearch.ai/openapi.json
- Pricing: https://www.desearch.ai/pricing
- Status: https://status.desearch.ai/

---

Found this useful? Sign up to Desearch with the author's referral link: https://console.desearch.ai/signup?ref=542D2C7D
