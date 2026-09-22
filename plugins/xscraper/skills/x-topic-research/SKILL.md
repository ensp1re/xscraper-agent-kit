---
name: x-topic-research
description: Find out what people on X (Twitter) are saying about a topic, product, brand, ticker, event or keyword, and summarize the main themes, sentiment and most-engaged posts with links. Use this for "what's the sentiment on X about ...", "what are people saying about our launch", brand or competitor listening, crypto/stock chatter ($TICKER), reactions to news, finding complaints or feature requests, or gathering examples and quotes from X for a report. Uses the xscraper MCP tools or API.
---

# X topic research

Goal: a short, sourced picture of the conversation, not a dump of posts. Search is the most expensive call, so plan the queries.

## Plan the queries (before any call)

1. Turn the topic into 1–3 queries with X operators. Examples:
   - Product feedback: `"ProductName" -from:ProductHandle -filter:retweets lang:en`
   - Complaints: `"ProductName" (broken OR bug OR refund OR worst) -filter:retweets`
   - Ticker: `$TICKER min_faves:20 -filter:replies`
   - A time window: add `since:YYYY-MM-DD until:YYYY-MM-DD`
   - Keep one neutral query (the topic words only). Queries with sentiment words (complaint or praise words) are good for finding example posts, but they bias the sample, so compute shares and sentiment from the neutral query only.
   - Reactions to one announcement: find the source post first with `from:<official account> <keyword>`, `mode: top`, then read its replies and quotes (below). Replies and quotes are reactions to one post, so report them separately from the neutral sample's shares.
2. Pick the mode: `top` to see what shaped opinion, `latest` to see what is happening now. For sentiment, run `top` first; add `latest` only if the user cares about the last hours. The tool defaults to `latest`, so pass `top` explicitly.
3. Dates: "last two weeks" becomes `since:<today minus 14 days>`; `until:` excludes its own date, so leave it off to include today.
4. Tell the user the plan in one line. If the estimate is over about 50 tokens, ask before starting (the xscraper-api spend rule). A count below 20 saves nothing: results come in pages of about 20.

## Fetch

- One `x_search_tweets` call per query at the default count. Read it.
- Page with the cursor only if the themes are not clear yet. Two pages per query is usually enough; stop when new posts repeat old themes.
- For a post that drives the conversation, `x_get_tweet` with `include: replies` and then `include: quotes` shows the reactions to it; complaints usually land in replies.
- Drop duplicates, retweets and obvious spam (same text from many accounts, only hashtags).

## Analyze

- Group posts into 3–6 themes. For each: one sentence, rough share of the sample, 1–2 example post URLs.
- Sentiment: positive / negative / mixed per theme, based on what the posts say, not on emoji.
- Note who drives it: accounts with the most engagement in the sample, and whether they are official, press, or users.
- Be honest about the sample: "based on N posts from the top results for these queries", not "X thinks".

## Output

```
## What X is saying about <topic> (<date range or "as of" date>)
Summary: 2–3 sentences.

### Complaints
1. Theme — ~share of the sample. Example: <url>
### Praise
1. Theme — ~share. Example: <url>
### Mixed or neutral
- ...
### Most engaged posts
- @author: "short quote" (likes, views) <url>
### Notes
Sample: N posts, queries used, what was not covered.
Cost: N tokens
```
