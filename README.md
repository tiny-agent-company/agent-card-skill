# agent-card skill

An agent skill for managing prepaid virtual Visa cards through [Agent Cards](https://github.com/keyserfaty/agent-cards) MCP tools. Works with Claude Code, Codex, and any agent that supports [skills.sh](https://skills.sh).

## Install

```bash
npx skills add agent-cards/skill
```

## What it does

This skill teaches your AI agent how to:

- **Create cards** — fund via Stripe Checkout, handle approval flows, poll until ready
- **Check balances** — quick balance lookup formatted as dollars
- **View card details** — PAN/CVV/expiry retrieval with email approval when required
- **Close cards** — with mandatory user confirmation (irreversible)
- **Support chat** — open conversations, send messages, read replies

## Prerequisites

You need the Agent Cards MCP server connected to your agent. See the [MCP setup guide](https://github.com/keyserfaty/agent-cards#mcp-server) for stdio or HTTP configuration.

## Usage

After installing, use the `/agent-card` slash command in Claude Code:

```
/agent-card
```

Or just ask your agent to manage your cards — the skill activates when card management intent is detected.

### Example prompts

- "List my cards"
- "Create a $50 test card"
- "What's the balance on my card?"
- "I need the card number to pay for something"
- "Close card xyz"

## Tools covered

| Tool | Purpose |
|------|---------|
| `list_cards` | List all cards for the authenticated user |
| `create_card` | Create a new prepaid virtual Visa card |
| `get_funding_status` | Poll checkout session status after payment |
| `get_card_details` | Get decrypted PAN, CVV, expiry (requires approval) |
| `check_balance` | Get card balance without revealing credentials |
| `close_card` | Permanently close a card (irreversible) |
| `start_support_chat` | Open a new support conversation |
| `send_support_message` | Send a message in a support conversation |
| `read_support_chat` | Read messages from a support conversation |

## License

MIT
