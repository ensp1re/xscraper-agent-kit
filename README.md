# xscraper agent kit

X (Twitter) data for AI agents: the xscraper MCP server and Agent Skills that teach an agent when and how to use it without wasting tokens.

You need an xscraper API key from https://xscraper.online/dashboard (new accounts get 1,000 free tokens).

## What is inside

- **MCP server** at `https://mcp.xscraper.online/mcp` with 9 read-only tools: `x_search_tweets`, `x_search_users`, `x_get_user`, `x_get_tweet`, `x_get_followers`, `x_get_user_likes`, `x_get_list_tweets`, `x_get_trends`, `x_get_balance`. Each tool call costs the tokens of the REST call behind it, and every result ends with the cost and your balance.
- **Skills** in `plugins/xscraper/skills/`:

| Skill | Use it for |
|---|---|
| `xscraper-api` | Core: which tool answers which question, search operators, spend rules, REST reference |
| `x-account-research` | Who is this account, is their engagement real, background before a deal |
| `x-topic-research` | What X says about a topic, brand, launch or ticker; themes and sentiment with links |
| `x-thread-digest` | Summarize a post with its replies and quote posts |
| `x-monitor` | Watch accounts or keywords and report only new posts, on a schedule |

## Install

### Claude Code (MCP server and skills)

```bash
export XSCRAPER_API_KEY=xs_your_key
```

Then in Claude Code:

```
/plugin marketplace add ensp1re/xscraper-agent-kit
/plugin install xscraper@xscraper
```

MCP server only:

```bash
claude mcp add --transport http xscraper https://mcp.xscraper.online/mcp --header "x-api-key: $XSCRAPER_API_KEY"
```

### Other agents that read Agent Skills (Codex, Cursor, VS Code Copilot, Gemini CLI, ...)

```bash
npx skills add ensp1re/xscraper-agent-kit
```

or copy the folders in `plugins/xscraper/skills/` into the agent's skills directory. Without the MCP server the skills fall back to `curl` with `$XSCRAPER_API_KEY`.

### Any client that runs local servers (stdio)

`npx -y xscraper-mcp` with `XSCRAPER_API_KEY` set runs the same tools locally.

### Claude Desktop

`claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "xscraper": { "command": "npx", "args": ["-y", "xscraper-mcp"], "env": { "XSCRAPER_API_KEY": "xs_your_key" } }
  }
}
```

For the skills, zip a skill folder and upload it in Settings → Capabilities → Skills.

### Cursor

`~/.cursor/mcp.json`:

```json
{ "mcpServers": { "xscraper": { "url": "https://mcp.xscraper.online/mcp", "headers": { "x-api-key": "xs_your_key" } } } }
```

### VS Code

`.vscode/mcp.json`:

```json
{
  "servers": { "xscraper": { "type": "http", "url": "https://mcp.xscraper.online/mcp", "headers": { "x-api-key": "${input:xscraper-key}" } } },
  "inputs": [{ "type": "promptString", "id": "xscraper-key", "description": "xscraper API key", "password": true }]
}
```

## Costs

Prices per call are in `plugins/xscraper/skills/xscraper-api/references/endpoints.md` and at https://xscraper.online/docs. Errors are refunded except 404 (not found). The skills tell the agent to start with one page, estimate before larger tasks, and ask before spending more than about 50 tokens.

## Source

This repository is generated from the xscraper monorepo, where the skills are tested against the live price catalog. Report problems in the issues here or at support@xscraper.online.
