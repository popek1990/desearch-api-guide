# Changelog: what changed at Desearch

Newest first. Sources: posts of [@desearch_ai](https://x.com/desearch_ai) (109 posts read, 2026-09-25 … 2026-11-25),
the Desearch docs and the OpenAPI spec read on 2026-11-25. These are announcements by Desearch, not our tests.

## 2026-11

- **11-25** SN22 publishes its service map on-chain: one URL pointing to OpenAPI, MCP, docs and payment guidance
  (https://www.desearch.ai/services.json).
- **11-24** Go SDK `desearch-go` v0.1.0, generated from a pinned OpenAPI snapshot (92 operations).
- **11-21** MCP adds cost and usage metadata to every billed tool result, including list and text outputs.
- **11-20** Semantic Kernel connector (`desearch-semantic-kernel`).
- **11-19** AI Search: pick sources with `tools` (`web`, `twitter`); a `warnings` array reports an empty or failed source.
- **11-18** New `POST /search` price: 0.30 per 1000 results, at least 10 billed (0.003 per default search), empty = free,
  summary free. Monitors pricing published.
- **11-17** Company Search guide: 45 USD / 1000 companies, minimum 1 company per task.
- **11-12** People Search available in the Console.
- **11-11** Company Search available in the Console.
- **11-10** **Monitors** live: saved searches on a schedule over X, Reddit, YouTube, web and more, only new items,
  optional webhook.
- **11-09** Benchmark: on SimpleQA Verified, Desearch put the reference page in the top 10 for 99.1 % of 463 questions
  (retrieval only, not answer correctness). https://www.desearch.ai/benchmarks
- **11-07** MCP supports Meta Muse connector OAuth.
- **11-06** Company Search can stop at a USD budget and continue an earlier task without repeating companies.
- **11-05** API notes: social list endpoints return a next-page cursor header; Instagram, TikTok and Facebook results
  have new fields; **X results include the post card**; Extract/crawl accepts a URL without `https://`. Search modes:
  fast vs balanced.
- **11-03** **Dedicated YouTube, Reddit and Threads APIs**, with Python, JavaScript and MCP tools.
- **11-02** `desearch-txtai` adapter for txtai pipelines and agents.
- **11-01** Live network stats stream once per second; `Last-Event-ID` replays the last 20 minutes.

## 2026-10

- **10-30** **Company Search** live: companies from a description, with public contacts, leaders and source pages.
- **10-29** **People Search** live: describe people, each result shows which criteria it matches and which it cannot
  confirm.
- **10-28** Embedding work on SN22 uses `Desearch-Embedding-4B`, an open model on Hugging Face.
- **10-27** Reddit Search demo: https://reddit-search.demos.desearch.ai
- **10-23** Web Search demo showing cost and cache state: https://web-search.demos.desearch.ai
- **10-21** Embedding tasks are part of SN22 main: GPU owners can embed crawled pages; validators re-check a sample.
- **10-19** Series of small open Python scripts, one per day (repo buzz, arXiv chatter, news digest, VC tracker, RAG
  dataset, complaint radar, Black Friday tracker, X trends by city …).
- **10-16** Crawl/Extract says plainly when a URL is invalid.
- **10-15** Web search can return page text and a publish-date window in one request.
- **10-14** Web search with start and end publish dates.
- **10-13** Guide: following an Instagram hashtag in three calls (accounts, posts, comments).
- **10-12** **Search returns page text** for ordinary queries, not just snippets.
- **10-10** AI Search modes in the Console: Fast, Balanced, Deep.
- **10-09** Desearch in Raycast's MCP Registry.
- **10-08** Plugin bundle for Claude Code, Cursor and Grok on GitHub. Cost headers on every billable response.
- **10-07** `desearch-mcp-server` 0.1.3 on npm and the MCP Registry.
- **10-06** Desearch MCP as a Gemini CLI extension. Crawl mining live on SN22.
- **10-03** Public DesearchBot crawler page (robots.txt, crawl delay, noindex).
- **10-02** Own fine-tuned embedding model for retrieval; search noticeably faster.
- **10-01** **Desearch 2.0**: SN22 miners crawl and embed web pages, rewards go by verified work. Hosted MCP over HTTP.

## 2026-09 (from 09-25)

- **09-30** Desearch 2.0 plan: the index grows with the Bittensor network; later, fine-tuning competitions for
  retrieval and ranking models.
- **09-27** SN22 validator v0.0.229 paused miner requests during the move to 2.0. **+100 USD API credits** for a public
  YouTube walkthrough of the Search API (at least 3 minutes, real workflow, after approval).
- **09-25** Desearch MCP exposes the public JS SDK as tools. Blog: how to earn free API credits
  (https://www.desearch.ai/blog/how-to-earn-free-api-credits-desearch).

---

Found this useful? Sign up to Desearch with the author's referral link: https://console.desearch.ai/signup?ref=542D2C7D
