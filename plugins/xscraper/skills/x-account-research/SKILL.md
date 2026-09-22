---
name: x-account-research
description: Research who an X (Twitter) account is - profile, what they post about, how they engage, who they follow, how influential they are - and write a short sourced brief. Use this when the user asks "who is @someone", wants background on a person, founder, company, KOL, journalist or candidate from X, wants to vet an account before a deal, partnership, hire or outreach, or asks whether an account is legit, a bot, or worth following. Uses the xscraper MCP tools or API.
---

# X account research

Goal: a brief the user can act on, built from a few cheap calls. Every call costs tokens; see the xscraper-api skill for tool details, or `x_*` tool descriptions for prices.

## Steps

1. **Profile and recent posts in one call**: `x_get_user` with `username`, `recent: tweets`, `count: 20`. This gives bio, counts, verification, join date and what they post now.
2. **Read before fetching more.** Most questions are answered here. Fetch more only for what is still unknown:
   - How they talk to others → `x_get_user` with `recent: replies`.
   - Their interests or circle → `x_get_followers` with `direction: following` (use the `user_id` from step 1).
   - Older posts or a specific topic → `x_search_tweets` with `from:username` plus keywords or `since:`/`until:`.
   - What others say about them → `x_search_tweets` with `@username -from:username`, `mode: top`.
3. Keep the whole brief under about 30 tokens unless the user asked for depth. If they want a deep dive, estimate first.

## Signals worth reporting

- Account age, follower/following ratio, posting rate (posts per day over the fetched window).
- Verification type and affiliation (a business badge or an organization label).
- Topics they post about most, and their tone.
- Engagement relative to followers: views and likes per post vs follower count. Very low engagement on a large account, or a burst of new followers, can mean bought followers; say "possible", not "certain".
- Links in bio and posts that show employer, projects or products.

## Brief format

```
## @handle — Name
One-line summary: who they are and why it matters for the user's question.

- Profile: followers / following, joined YYYY-MM, verification, location, website
- Posts about: 3–5 topics, with one example post URL each
- Activity: ~N posts/day over the last N days; typical engagement
- Network: notable accounts they follow or who interact with them (if fetched)
- Flags: anything that looks off, stated with the evidence
- Sources: post URLs used
Cost: N tokens
```

State what you did not check (for example "likes not fetched") so the user can ask for it.
