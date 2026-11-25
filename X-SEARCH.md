# X (Twitter) search in practice

How Desearch X search really behaves: our paid test calls, query pitfalls, what you are really billed, and the risks.
For the list of X endpoints and the general pricing see [API.md](API.md).

As of: 2026-11-25 (measurements 2026-10-28 … 2026-11-12, each with its date).

Labels used below:
- **Influencer-tracker test (2026-11-10):** 115 calls with a separate key on 2 active X accounts, cost read from the
  `X-Desearch-Cost-Usd` header.
- **Server test:** calls with `curl` from our own server, with a direct key.
- `untested`: needs a key and was not tested yet.

## Docs (2026-11-19) vs our measurements — needs a re-test

The current X docs (`llms-full.txt` reviewed 2026-11-19 and the pricing FAQ, read 2026-11-25) describe billing and
queries differently from our measurements of 2026-11-10 and 2026-11-12. We do not know yet whether billing changed or
only the description. Until a new test: read the real cost from `X-Desearch-Cost-Usd` and budget for the worse case.

| Topic | Our measurement | Docs 2026-11-19 | If the docs are right |
|---|---|---|---|
| `GET /twitter`, minimum | no minimum, you pay only for posts returned (2026-11-10, 2026-11-12) | minimum 10 posts per successful search | a query with 1–9 posts costs like 10 (0.0015 USD) |
| `GET /twitter`, empty result | 0 USD (2026-10-30, 2026-11-01, 2026-11-12) | the 10-post minimum applies to an empty result too; only failed searches are free | every empty poll costs 0.0015 USD; frequent `since_id` polling is no longer free |
| `/twitter/user/posts` page | 15–21 posts; empty page 0.0015 USD (2026-11-10) | 10-post minimum, up to 60 posts per page; an empty page is free | the last page of a timeline is free, fewer pages per account |
| `OR` without brackets plus dates | dates applied only to the last term (2026-11-12) | filters apply to every `OR` term; the query is wrapped in brackets | bug fixed; keep the brackets anyway, they cost nothing |
| `start_date` / `end_date` | day precision, day borders not clean UTC (2026-11-10, 2026-11-12) | `YYYY-MM-DD` (whole UTC day) or ISO 8601 date-time with a time zone, both inclusive; any other value or a `cursor` = 422, not charged | narrow hour windows by parameter instead of `since:` with hours |
| `/twitter/replies` | for an active account only posts from 1–3 years ago (2026-11-02) | replies newest first, `count` 1–100 (default 10), `query`, next page via the `x-desearch-next-cursor` header | may now work for an account's current replies |

Planned test (direct key, `curl`, under 0.05 USD): empty `GET /twitter` and a query returning 1–3 posts with
`count=10`; the last page of `/twitter/user/posts`; `"A" OR $T` with dates and no brackets; `start_date` with an ISO
8601 time; `/twitter/replies` for an active account.

## Endpoints we use and what we learned

| Endpoint | What it gives | Notes from use |
|---|---|---|
| `GET /twitter` | post search with operators: `from:`, `$cashtag`, `since:`/`until:`, `min_faves:`, `lang:`; `count` 1–100, default 20; sort Top (default) or Latest; no cursor. Parameters (OpenAPI 2026-11-25): `query`, `sort`, `user`, `start_date`, `end_date`, `lang`, `verified`, `blue_verified`, `is_quote`, `is_video`, `is_image`, `min_retweets`, `min_replies`, `min_likes`, `count`. Posts with none of the query's words are left out, so you may get fewer than `count` (docs) | mentions of a project, who writes, authors' cashtag history |
| `GET /twitter/user/posts` | an account's posts with paging: only `username` and `cursor`, no `count`, no sort; a page has 15–21 posts at 0.00015 USD each (influencer-tracker test: 42–44 pages per account); the timeline ends after ~780–790 newest posts (empty page); has posts and reposts (`retweet` object), few replies | author history; reposts that search does not return |
| `GET /twitter/post/retweeters`, `/twitter/replies/post` | who reposted, replies to a post | reposts and reactions |
| `GET /twitter/replies` | an account's replies; needs `user`, not `query`. Server test 2026-11-02: for an active account it returned only old posts from 1–3 years ago | useless for current replies; use `GET /twitter` with `from:<account>` and dates instead (worked in the same test) |
| `GET /twitter/urls`, `/twitter/post` | fetch specific posts | proof behind a link in a report |

## Query rules

- **Time operators** (list from the docs; server test 2026-11-01: `since_id` and `since:` with seconds work, the given second is included; `since_time` only a negative test; the rest `untested`): `since_id:<id>` (after this post, excluding it), `max_id:<id>`, `since_time:<unix>`, `until_time:<unix>`, `since:YYYY-MM-DD_HH:MM:SS_UTC`, `within_time:5m`. `since:YYYY-MM-DD` and the `start_date`/`end_date` parameters work to the day. A time operator must go with another operator. Max ~22–23 operators per query.
- **`max_id:`** works and is inclusive: the post with that id is in the result (influencer-tracker test).
- **`end_date` is inclusive, and day borders are not clean UTC:** `start_date` = `end_date` = D also returns the early hours of D+1 (influencer-tracker test). The end of the window also depends on the query (server test 2026-11-12, window 5–11 Nov, one sample): a cashtag alone reached ~05:46 UTC of D+1, while a name alone stopped ~23:00 UTC of D and missed posts from 23:06–23:44 that another query returned. Set `end_date` one day later, filter by `created_at` yourself and do not assume the last hours of a day are covered.
- **Always put `OR` in brackets when you use dates: `("name" OR $TICKER)`.** (Docs 2026-11-19 say the API now wraps the query in brackets itself — see the table at the top; keep the brackets anyway.) Without brackets the `start_date`/`end_date` window applies only to the last term; the first comes without any date limit, with no warning (server test 2026-11-12, 8 calls, 0.012 USD): `"A" OR $T` with dates gave the same IDs as `"A"` without dates, `$T OR "A"` with dates the same as `$T` without dates; only brackets gave posts from the window. Most likely the API appends dates as an AND condition that binds tighter than `OR` (probable; not documented).
- **`from:<account>` vs `/twitter/user/posts`:** `from:` search returned mostly replies (69–93 % of results) and no reposts; `user/posts` returned mostly posts and reposts, ~780 newest. Neither alone gives the full history of an account (influencer-tracker test).
- **`entities.symbols` (cashtags recognised by X) is incomplete:** filled in 73 % and 98 % of posts with a cashtag in the text (2 accounts, influencer-tracker test). Detect cashtags from the text yourself.
- **Reposts:** search skips reposts by default; `include:nativeretweets` works ~7–10 days back. The `retweet` object exists only in `/twitter/user/posts`.
- **Index delay** is not documented; the docs only say "real-time" and recommend checking `created_at`. `untested`.
- **Complex queries gave false zeros.** Server test 2026-10-31: queries with `OR`, a cashtag and `start_date` returned 0 results while posts existed that simpler queries found. Zero does not prove there are no posts. Rule: one simple condition per query; add dates only after a simple query worked. Server test 2026-11-12 (active project, `count=10`, `Latest`): name, `$TICKER` and `"name" OR $TICKER`, each with and without dates, gave 10 posts each, no zero; the cause of the 2026-10-31 zeros was not found (a small project might behave differently). That test found the unbracketed `OR` bug instead (point above). Server test 2026-11-01: `(from:a OR from:b) since_id:<id>` and the cashtag `$BTC` alone returned posts, so `OR` and cashtags by themselves do not break results. Server test 2026-11-02: `from:<account>` with dates returned posts, so dates with `from:` work. **A cashtag with `since:`/`until:` with hours gives a false zero** (server test 2026-11-02, `/twitter`): `$TICKER since:YYYY-MM-DD_HH:MM:SS_UTC until:…` gave 0 results while posts existed, and the same cashtag with `start_date`/`end_date` (whole days) worked. For a narrow hour window with a cashtag, use whole days and filter hours yourself by `created_at`.
- **No "any cashtag" operator.** Only a specific `$TICKER` works (operator list: https://desearch.ai/docs/guide/apis/x-queries, read 2026-11-09). To get all posts with cashtags from one account: `from:<account>` in short date windows, pick cashtags locally.
- **`sort=Latest` and `count=100`:** can return the full 100 posts (influencer-tracker test: `from:<account>`, 1-month window, both accounts). `count` takes 1–100; 20 is the default, not a hard limit. Do not assume a fixed number of results and do not cut them to 20. `GET /twitter` returns no cursor; history needs shorter date windows and checking the range of each answer.
- **Date coverage:** `Latest` starts with the newest posts; `Top` is a ranking with no guarantee of spreading results over the whole window. Check the number of posts, unique IDs and the `created_at` range; hitting `count` or a cut-off range means shorter windows (within budget). After merging, filter dates and remove duplicates by ID. Fewer than `count` results, or posts from both ends of the window, do not prove completeness; report the gaps.
- **A cashtag is probably a sample, not the full stream.** Test 2026-11-01: `$BTC` with `sort=Latest`, 20 posts: 17 within 216 seconds, then a ~19.5-hour gap. Conclusion `untested`, but do not compute volume or completeness from such a sample.

## What you are really billed

- `GET /twitter` (measured; docs 2026-11-19 say otherwise — see the table at the top): you pay for unique posts returned, **no minimum of 10**, at most `count` (influencer-tracker test: `count=10` and 1 post returned = 0.00015 USD, 9 posts = 0.00135 USD; `count=100` and 100 posts = 0.015 USD; confirmed by server test 2026-11-12: `count=10` and 2 posts = 0.0003 USD, 20 posts = 0.003 USD, empty result 0 USD). The X API docs (read 2026-11-12) confirm there is no minimum; the general pricing page still says minimum 10. When they disagree, trust the endpoint's own docs and the cost header.
- `GET /twitter/user/posts` (measured; docs 2026-11-19: an empty page is free — see the table at the top): **an empty page costs 0.0015 USD** (like 10 posts; influencer-tracker test, last page of the timeline); a page with 11 posts 0.00165 USD. An empty `GET /twitter` costs 0 USD.
- `/twitter/replies`, `/twitter/replies/post`: minimum 10 posts when anything is returned, never more than `count`. Empty result free (X API docs, read 2026-11-12; not tested). Do not apply this minimum to other X endpoints.
- Free, among others: 503 errors on X endpoints.
- Errors and cost (influencer-tracker test): 422 (e.g. a wrong `sort` value) comes **without a cost header**; 404 from `/twitter/post` (post not found) has `X-Desearch-Cost-Usd: 0`.
- **Aborted connection** (client closed it after 0.5 s, `/twitter/user/posts`): the Console does not list it among errors, while a deliberate 404 from the same second is listed — the server most likely finished and billed the call. Count aborted calls as paid.
- **The Console does not show spend per key** (read 2026-11-10), so one program's counter cannot be reconciled with the Console; "Latest Errors" shows failed calls of the whole account.
- Empty `GET /twitter` result: the docs say 503 `upstream_empty`, OpenAPI says 200 with an empty list. Test 2026-11-01: 200 and 0 USD, but `upstream_empty` was also seen earlier. A client must accept both.

## Cost in practice

- Search that returned 10 posts: 0.0015 USD (2026-10-28). A client may ask for more posts than it shows; you pay for what the API returns, not what you display.
- First full research run: 13 calls, 452 unique posts, 0.0726 USD. Every post found earlier through FxTwitter was also in the Desearch results.
- Forum test 2026-10-28: `/web` 0.001 USD per query, `/web/extract` 0.0005 USD per page.
- Server test 2026-11-01: `/twitter/user/posts` returns 20 posts for 0.003 USD. Polling `GET /twitter` with `since_id` every minute for 30 minutes with no new posts: 0 USD.
- Server test 2026-10-30: `GET /twitter` with no results costs 0 USD (`X-Desearch-Cost-Usd` header), a query with 12 posts 0.0018 USD — empty queries do not eat the budget.
- Influencer-tracker test: the whole `/twitter/user/posts` timeline of one active account (~785 posts, 42–44 pages) ~0.12 USD; `GET /twitter` in date windows 0.15 USD per 1000 returned posts.

A normal research run costs cents. The real cost is X history of authors in deep research: 5 USD is ~33,000 posts for
the whole run. Decide the number of posts per author before you start.

## Risks

- On 2026-01-15 X cut off apps that pay users for posts (theblock.co, single source). How Desearch accesses X is not described, so a cut-off could come without warning.
- X archive depth is not described. In one test `from:<official account>` did not cover the first ~5 months of the account (answer `upstream_empty`, free); unknown whether there were no posts or the index has a gap. Influencer-tracker test: for an account created in 2019 (~7,100 posts per its profile), year-by-year search returned zero posts for 2019–2023 and the first ones in 2024; the second account had posts in every year since 2021. Do not claim full history.
- Availability varies (503 `upstream_*` codes). Request limits are not published.
- **Terms and data** (read 2026-10-31). Terms of Service from 2025-01-15 (may be outdated) say nothing about X data or passing it to AI models; they forbid, among others, reselling without consent and bypassing limits. The legal basis for reading X is not described. The X Developer Agreement from 2026-04-27 forbids training foundation models on X Content; whether that binds a Desearch customer: `untested`. Privacy: see [API.md](API.md#privacy-and-compliance).

## Measured pitfalls of Extract

- `/web/extract` on a Bitcointalk thread returns only the first page, without post dates (test 2026-10-28).
- `/web/extract` with `js=true` can return an almost empty page: on a large exchange's listings page it returned 756 B with no list, while raw HTML from `curl` (1.2 MB) had the list in embedded JSON (server test 2026-10-31). A short extract is a reason to check the raw HTML, not proof the page is empty.

## Other ways to read X (comparison, read 2026-11-09)

| Option | Price | Notes |
|---|---|---|
| Desearch | 0.15 USD / 1000 posts | our main one; archive depth not described |
| Official X API (pay-per-use) | 0.005 USD / post, 0.010 USD / user | ~33× more expensive; full archive since 2006, `has:cashtags` operator, up to 3 M reads/month (https://docs.x.com/x-api/getting-started/pricing) |
| twitterapi.io | 0.15 USD / 1000, min. 0.00015 USD / call | `untested`; backup if Desearch fails (https://twitterapi.io/pricing) |
| FxTwitter | 0 USD | whether a post exists, and the newest posts; batches of ~20, random 404s, no guarantees (https://docs.fxembed.com/api/introduction). Influencer-tracker test: timeline `/2/profile/<account>/statuses` gives ~780 newest posts (the same ceiling as `user/posts`), ~40 % of answers are random 404s, misses 0–0.7 % of posts compared with Desearch |

For web search only, Brave Search API is an alternative: 5 USD / 1000 queries, 5 USD free credit every month, no X
(https://brave.com/search/api/, as of 2026-10-28; not tested by us).

## Sources

- X search, sorting and billing: https://www.desearch.ai/docs/guide/apis/x-api
- X operators: https://desearch.ai/docs/guide/apis/x-queries
- Limits and errors: https://desearch.ai/docs/guide/introduction/rate-limit

---

Found this useful? Sign up to Desearch with the author's referral link: https://console.desearch.ai/signup?ref=542D2C7D
