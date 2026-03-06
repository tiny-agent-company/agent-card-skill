# agent-card skill

Give your AI agent a debit card. [Agent Cards](https://agentcard.sh) lets AI agents create and spend prepaid virtual Visa cards — each with a fixed budget, real card credentials, and full MCP integration. This skill teaches your agent how to use them.

Works with Claude Code, Codex, and any agent that supports [skills.sh](https://skills.sh).

## Setup

### 1. Create an Agent Cards account

```bash
npx agent-cards signup
```

### 2. Connect the MCP server

**Option A — Automatic (Claude Code)**

```bash
npx agent-cards setup-mcp
```

This auto-configures Claude Code with the hosted MCP server. Restart Claude Code after running.

**Option B — Manual (any MCP client)**

Add to your MCP client config (Claude Desktop, Cursor, etc.):

```json
{
  "mcpServers": {
    "agent-cards": {
      "url": "https://mcp.agentcard.sh/mcp",
      "headers": {
        "Authorization": "Bearer <your-jwt>"
      }
    }
  }
}
```

Get your JWT from `~/.agent-cards/config.json` after signing up.

See the [MCP server repo](https://github.com/agent-cards/mcp) for more options (stdio, self-hosted, etc.).

### 3. Install the skill

```bash
npx skills add agent-cards/skill
```

## What it does

Once installed, your agent knows how to:

- **Create cards** — set a dollar amount, complete Stripe Checkout, get a card back
- **Check balances** — quick balance lookup, formatted as dollars
- **View card credentials** — full card number, CVV, expiry (with email approval for security)
- **Close cards** — permanent closure with user confirmation
- **Get support** — open and manage support conversations

## Usage

Use the `/agent-card` slash command in Claude Code:

```
/agent-card
```

Or just ask naturally — the skill activates when your agent detects card management intent.

### Example prompts

- "List my cards"
- "Create a $50 test card"
- "What's the balance on my card?"
- "I need the card number to pay for something"
- "Close card xyz"
- "I need help with my card"

## MCP Tools

These are the tools the skill orchestrates. You don't call them directly — the skill guides your agent through the right workflow.

| Tool | What it does |
|------|-------------|
| `list_cards` | List all your virtual cards with balances and status |
| `create_card` | Create a new prepaid Visa with a fixed USD budget |
| `get_funding_status` | Poll until a new card is ready after payment |
| `get_card_details` | Get decrypted card number, CVV, expiry |
| `check_balance` | Quick balance check (no credentials exposed) |
| `close_card` | Permanently close a card |
| `start_support_chat` | Open a support conversation |
| `send_support_message` | Send a message to support |
| `read_support_chat` | Read support replies |

## Links

- [Agent Cards](https://agentcard.sh) — product homepage
- [MCP Server](https://github.com/agent-cards/mcp) — MCP server setup and docs
- [skills.sh](https://skills.sh) — agent skills directory

## License

MIT
