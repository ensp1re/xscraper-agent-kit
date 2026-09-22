<!-- Generated from packages/shared by `pnpm --filter @xscraper/mcp gen:reference`. Do not edit. -->
# xscraper REST endpoints

Base URL: `https://api.xscraper.online`. Every endpoint is a GET. Send the key in the `x-api-key` header:

```bash
curl -s -H "x-api-key: $XSCRAPER_API_KEY" "https://api.xscraper.online/api/v1/twitter/users/profile_by_username/XDevelopers"
```

Every response is `{ success, message, data, errors }`. Paged endpoints return `data.next`; pass it back as `?cursor=`.
Every billed response has `x-tokens-cost` and `x-tokens-remaining` headers (use `curl -i` to see them).
`*` marks a required parameter. Path parameters go in the URL, the rest in the query string.

| id | path | params | count default / max | price | MCP tool |
|---|---|---|---|---|---|
| `profile_by_username` | `GET /api/v1/twitter/users/profile_by_username/:username` | username* | - | 1 token | x_get_user |
| `profile_by_userid` | `GET /api/v1/twitter/users/profile_by_userid/:userId` | userId* | - | 3 tokens | x_get_user |
| `user_id` | `GET /api/v1/twitter/users/user_id/:username` | username* | - | 1 token | x_get_followers (resolves a username) |
| `search_profiles` | `GET /api/v1/twitter/users/search_profiles` | query*, count, cursor | 20 / 50 | 3 + 1 per 20 items (4–6 tokens) | x_search_users |
| `user_tweets` | `GET /api/v1/twitter/users/tweets/:username` | username*, count | 5 / 100 | 2 + 1 per 20 items (3–7 tokens) | x_get_user recent=tweets |
| `latest_tweet` | `GET /api/v1/twitter/users/latest_tweet/:username` | username* | - | 1 token | x_get_user recent=tweets count=1 |
| `user_replies` | `GET /api/v1/twitter/users/replies/:username` | username*, count | 5 / 100 | 2 + 1 per 20 items (3–7 tokens) | x_get_user recent=replies |
| `tweets_by_user_id` | `GET /api/v1/twitter/users/tweets_by_user_id/:userId` | userId*, count | 5 / 100 | 2 + 1 per 20 items (3–7 tokens) | x_get_user user_id recent=tweets |
| `replies_by_user_id` | `GET /api/v1/twitter/users/replies_by_user_id/:userId` | userId*, count | 5 / 100 | 2 + 1 per 20 items (3–7 tokens) | x_get_user user_id recent=replies |
| `user_likes` | `GET /api/v1/twitter/users/likes/:username` | username*, count, cursor | 20 / 100 | 3 + 1 per 20 items (4–8 tokens) | x_get_user_likes |
| `tweet` | `GET /api/v1/twitter/tweets/:tweetId` | tweetId* | - | 1 token | x_get_tweet |
| `tweet_replies` | `GET /api/v1/twitter/tweets/:tweetId/replies` | tweetId*, cursor | - | 4 tokens | x_get_tweet include=replies |
| `tweet_quotes` | `GET /api/v1/twitter/tweets/:tweetId/quotes` | tweetId*, cursor | - | 4 tokens | x_get_tweet include=quotes |
| `search` | `GET /api/v1/twitter/tweets/search` | query*, count, mode, cursor | 20 / 100 | 4 + 2 per 20 items (6–14 tokens) | x_search_tweets |
| `advanced_search` | `GET /api/v1/twitter/tweets/advanced_search` | query*, count, mode, cursor | 20 / 50 | 6 + 2 per 20 items (8–12 tokens) | - |
| `followers` | `GET /api/v1/twitter/users/followers/:userId` | userId*, count, cursor | 50 / 50 | 4 tokens | x_get_followers |
| `following` | `GET /api/v1/twitter/users/following/:userId` | userId*, count, cursor | 50 / 50 | 4 tokens | x_get_followers direction=following |
| `list_tweets` | `GET /api/v1/twitter/lists/:listId/tweets` | listId*, count, cursor | 20 / 100 | 2 + 1 per 20 items (3–7 tokens) | x_get_list_tweets |
| `trends` | `GET /api/v1/twitter/trends` | - | - | 2 tokens | x_get_trends |
| `balance` | `GET /api/v1/twitter/balance` | - | - | 0 tokens | x_get_balance |

## Errors

| status | meaning | charged |
|---|---|---|
| 400 | a parameter is missing or invalid; the message names it | no |
| 401 | missing, unknown or revoked key | no |
| 402 | balance lower than the call's price; the message gives both | no |
| 404 | user or list does not exist or is not visible | yes |
| 429 | more than the per-minute limit, or too many calls at once; wait `Retry-After` seconds | no |
| 503 | scraper pool busy; wait `Retry-After` seconds | no |
| other 5xx | upstream failure; safe to retry once | no (refunded) |

A tweet that does not exist or is not visible is not a 404: `tweet` answers 200 with `data: null` and is charged.
