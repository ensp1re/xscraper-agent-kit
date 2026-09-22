---
name: x-thread-digest
description: Summarize an X (Twitter) post and the conversation around it - the post itself, its replies and quote posts - into the main arguments, the notable voices and the overall reaction. Use this when the user pastes an x.com or twitter.com status link, asks "what's this thread about", "how are people reacting to this tweet", "summarize the replies", wants the ratio / pushback on a post, or needs the key points of a long thread. Uses the xscraper MCP tools or API.
---

# X thread digest

Goal: tell the user what the post says and how people reacted, citing the replies that matter.

## Fetch

1. `x_get_tweet` with the URL or ID and `include: replies`. This returns the post and the first page of replies (about 20).
2. If the post is part of an author's thread (the post replies to the same author, or the text says 1/ or 🧵), get the author's other parts with `x_search_tweets`, query `conversation_id:<ID> from:<author>`.
3. Reactions beyond replies: `x_get_tweet` with `include: quotes`. Quote posts often carry the strongest reactions; fetch them when the user asks about reception or pushback.
4. Page further with the cursor only for large conversations where the first page is not representative. Two or three pages are usually enough. Tell the user if you stopped early.

## Digest

- **The post**: what it claims or announces, in 1–3 sentences. Quote the key line.
- **Reaction**: overall tone (supportive, split, critical, joking), with rough shares from the sample.
- **Main arguments**: 3–5 points people make, each with one example reply or quote URL.
- **Notable voices**: replies from accounts with many followers, verified or official accounts, and the author's own replies (corrections, clarifications).
- **Facts to check**: claims in the thread that are disputed or corrected in replies.

Keep engagement numbers (likes, views) next to the posts you cite, so the user can see what carried weight.

End with the sample size ("post + 40 replies + 20 quotes") and the cost.
