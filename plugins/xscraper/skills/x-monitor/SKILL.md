---
name: x-monitor
description: Watch X (Twitter) accounts, keywords, brands, competitors or tickers for new posts and report only what is new since the last check - a watchlist digest. Use this when the user wants to track or monitor accounts or mentions on X, get alerts or a daily/hourly digest of new tweets, follow what competitors or KOLs post, watch a launch or incident, or set up a recurring X check with /loop, /schedule or cron. Uses the xscraper MCP tools or API.
---

# X monitor

Goal: a repeatable check that costs little each run and reports only new posts.

## Set up the watchlist (first run)

Agree with the user on, and save to a file in the working directory (default `x-watchlist.json`):

```json
{
  "accounts": ["XDevelopers", "OpenAI"],
  "queries": ["\"our product\" -from:ourhandle -filter:retweets"],
  "includeReplies": false,
  "seen": [],
  "lastRun": null
}
```

`seen` holds the IDs of posts already reported (keep the newest 500).

## One run costs one search per watch

Combine all accounts into one search instead of checking them one by one:

- Accounts: `x_search_tweets` with `(from:XDevelopers OR from:OpenAI OR from:AnthropicAI) -filter:replies -filter:retweets since:<date of lastRun minus 1 day>`, `mode: latest`, default count. One call covers up to about 25 accounts (the query limit is 500 characters); split into more queries beyond that. Drop `-filter:replies` if `includeReplies` is true.
- Each keyword query: the same call with the user's query plus `since:`.

This costs the same whether 1 or 25 accounts are watched. `-filter:replies` also hides the rest of an account's own threads (only the first post shows); drop it if the user wants threads, and skip replies to other accounts yourself. If the user already has an X list with these accounts, `x_get_list_tweets` on that list is cheaper than a search. Checking accounts one by one with `x_get_user` costs a profile plus posts per account, so use it only for one or two accounts, or when the user needs every reply an account makes.

Before starting, tell the user the cost of one run (number of searches × the search price in the tool description) and per day at the chosen interval, and offer a longer interval if they asked for "cheap".

## Each run

1. Read the watchlist file.
2. Run the searches above.
3. New posts are those whose ID is not in `seen`. Use the set of seen IDs, not "newest ID so far": search can index a post a few minutes late, and a newest-ID check would skip it.
4. If every post on the page is new and the result gives a next cursor, there may be more: fetch the next page (at most two extra pages, then report that the run was capped).
5. Add the new IDs to `seen`, set `lastRun`, save the file. Save even when nothing is new.
6. Report only new posts. If nothing is new, say so in one line.

The first run has no `seen` list: report posts from the last 24 hours and mark everything returned as seen.

## Digest format

```
## X watch — <date time>
@account (2 new)
- <one-line summary> (likes, views) <url>
"query" (5 new)
- ...
Nothing new: @other, "other query"
Cost this run: N tokens · balance: M
```

Flag posts that need attention: engagement several times the account's usual level in earlier digests, a complaint from a large account, or a launch, pricing or outage announcement.

## Running it on a schedule

- `/loop 1h <run the X watch>` in Claude Code repeats the check in this session and keeps `x-watchlist.json` on disk between runs.
- `/schedule` runs in a fresh cloud checkout each time: the watchlist file only survives if the run commits it to the repository.
- cron or CI with `claude -p "<run the X watch>"` works like `/loop` if the working directory stays the same.

Every result ends with the balance: stop and tell the user when it will not cover one more day of checks.

`/loop` runs only while the Claude Code session is open, and each run also uses the user's Claude usage. For the cheapest unattended check, suggest a cron job that calls the REST API with `curl` (see the xscraper-api skill's reference) and only alerts on new IDs, with no model in the loop.
