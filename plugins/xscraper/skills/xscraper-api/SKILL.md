---
name: xscraper-api
description: Read public X (Twitter) data through the xscraper API - posts, search, profiles, followers, likes, lists, trends - and spend its tokens carefully. Use this whenever a task needs live data from X or Twitter (a tweet, an account, what people post about something, followers, trending topics), whenever the xscraper MCP tools (x_search_tweets, x_get_user, x_get_tweet, ...) are available, or when the user mentions xscraper, XSCRAPER_API_KEY, or the xscraper.online API. Also use it to write code that calls the xscraper REST API.
---

# xscraper API

xscraper returns public X (Twitter) data as JSON. Every call costs tokens from the user's prepaid balance, so the job is to answer the question with as few calls as the answer needs.

## Pick the access path

1. **MCP tools present** (names end in `x_search_tweets`, `x_get_user`, ...): use them. Each tool description states its price, and every result ends with `— cost: N tokens · balance: M · next cursor: …`.
2. **No MCP tools, shell available**: call the REST API with curl and `$XSCRAPER_API_KEY`. Read `references/endpoints.md` for paths, parameters, prices and errors.
3. **Neither, or no key**: tell the user they need a key from https://xscraper.online/dashboard and either the MCP server (`claude mcp add --transport http xscraper https://mcp.xscraper.online/mcp --header "x-api-key: $XSCRAPER_API_KEY"`) or the env var.

## Which tool for which question

| Question | Tool | Notes |
|---|---|---|
| What are people saying about X? | `x_search_tweets` | Use operators (below). `mode: top` for the most engaged, `latest` for newest. |
| Who is this account? | `x_get_user` | `recent: tweets` adds their latest posts in the same call. |
| What did they post recently? | `x_get_user` with `recent: tweets` | Older posts: `x_search_tweets` with `from:user until:DATE`. For several accounts at once, one search `(from:a OR from:b) since:DATE` is cheaper than one call per account. |
| What does this tweet say / how did people react? | `x_get_tweet` | `include: replies` or `quotes` adds one page of reactions. Accepts a tweet URL. |
| Who follows them / who do they follow? | `x_get_followers` | Pass `user_id` when you have it; a username costs one extra lookup. |
| What do they like? | `x_get_user_likes` | Often private or empty. |
| Posts from a curated list | `x_get_list_tweets` | List ID is the number in x.com/i/lists/ID. |
| Find accounts in a niche | `x_search_users` | Matches names and bios. |
| What is trending? | `x_get_trends` | |
| How many tokens are left? | `x_get_balance` | Free. |

## Search operators

`from:user` · `to:user` · `@user` (mentions) · `"exact phrase"` · `word1 OR word2` · `-word` · `lang:en` · `since:2026-01-01` · `until:2026-02-01` · `min_faves:100` · `min_retweets:20` · `min_replies:10` · `filter:links` · `filter:media` · `-filter:replies` · `-filter:retweets` · `conversation_id:ID` · `url:example.com` · `$TICKER` · `#hashtag`

Narrow the query before paging: `min_faves:` and `-filter:replies` cut noise more cheaply than reading three more pages.

## Spend rules

- Start with one call at the default count. Read the result, then decide whether you need more.
- Page with the returned cursor only when the answer is not there yet. Stop when a page adds nothing new.
- Before a task that needs more than about 50 tokens (for example, several searches plus many pages), give the user the estimate and ask. Use the prices in the tool descriptions or `references/endpoints.md`.
- If a result shows a low balance, or a call returns 402, stop and tell the user the balance and what the rest would cost. Do not retry a 402.
- A 404 is charged: check the username (no @) or ID before retrying. 429 and 503 are not charged; wait the given seconds and retry once.
- Say what the task cost at the end (sum the `cost:` lines).

## Reading results

Concise output is one line per item: `ID · @author · date · text · likes rts replies views · URL` for posts, `ID · @handle · name · followers following · bio` for accounts. Keep IDs; later calls take them. Ask for `response_format: detailed` only when you need fields the line lacks (media, links, full author, quoted post fields), because detailed output is several times longer.

When you report to the user, cite posts by URL, give numbers as of the fetch time, and say when a sample is small (one page is about 20 posts, not all of X).

## Writing code against the API

Use `references/endpoints.md` for paths and parameters. Send `x-api-key`, read `data` from the envelope, follow `data.next` as `cursor`, and read `x-tokens-remaining` to stop before the balance runs out. Keep the key in an environment variable, never in source.
