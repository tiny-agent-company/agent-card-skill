# AgentCard Skill

Give any AI agent the ability to create and manage prepaid virtual Visa cards.

This skill teaches AI agents how to use [AgentCard](https://agentcard.sh) MCP tools — creating cards, checking balances, paying for things, and more. It works with Claude Code, Cursor, Cline, Windsurf, and [40+ other agents](https://skills.sh).

## Quick Install

```bash
npx skills add tiny-agent-company/agent-card-skill
```

Then connect the MCP server to give your agent access to the tools:

```bash
# Claude Code (option A — built-in CLI command)
agent-cards setup-mcp

# Claude Code (option B — manual)
claude mcp add --transport http agent-cards https://mcp.agentcard.sh/mcp
```

For Cursor, Windsurf, and other MCP-compatible agents, add to the MCP config file (`.cursor/mcp.json`, `.windsurf/mcp.json`, etc.):

```json
{
  "mcpServers": {
    "agent-cards": {
      "url": "https://mcp.agentcard.sh/mcp"
    }
  }
}
```

**Restart your agent session after adding the MCP server** — tools don't load until next session.

Authentication is handled via OAuth — your agent will prompt you to sign in on first use.

## Setup Prompt

Copy-paste this into your AI agent to have it set everything up:

```
Set up AgentCard so I can create and manage virtual Visa cards from this agent.

Do these steps in order. Some steps require me to do something — wait for my confirmation before moving on.

1. Install the AgentCard skill:
   npx skills add tiny-agent-company/agent-card-skill

2. Install the CLI:
   npm install -g agent-cards

3. Check if I'm already logged in:
   agent-cards whoami
   If not logged in, run: agent-cards signup
   (I need to click the magic link in my email — wait for me to confirm)

4. Connect the MCP server:
   agent-cards setup-mcp
   If that doesn't work, run: claude mcp add --transport http agent-cards https://mcp.agentcard.sh/mcp

5. Tell me to restart this session so the MCP tools load.
   Do NOT try to use the tools in this session — they won't work until restart.
   Do NOT fall back to curl or raw API calls.

After I restart the session, I'll ask you to list my cards to verify it works.
```

## Don't Have an Account?

```bash
npm install -g agent-cards
agent-cards signup
```

## What's Included

**Skill** (`SKILL.md`) — Procedural knowledge that teaches the agent:
- When and how to use each of the 27 AgentCard tools
- Workflows: card creation, balance checks, shopping & checkout (`buy`), payments, checkout autofill, plans, support
- Safety rules: never expose PAN/CVV unprompted, confirm before closing cards or placing an order
- Error handling: waitlists, approval flows, KYC requirements

**Setup guide** (`references/setup.md`) — Connection instructions the agent reads if tools aren't available yet.

## What Can Your Agent Do?

| Capability | Tools |
|-----------|-------|
| Issue virtual cards | `create_card`, `submit_user_info` |
| Manage cards | `list_cards`, `check_balance`, `get_card_details`, `close_card` |
| View spending | `list_transactions`, `list_all_transactions`, `list_transactions_by_payment_method` |
| Shop & check out | `buy`, `get_instructions`, `buy_list_merchants`, `buy_unlink_merchant` |
| Pay for things | `detect_checkout`, `fill_card`, `pay_checkout` |
| Payment methods | `setup_payment_method`, `remove_payment_method`, `list_payment_methods`, `set_default_payment_method` |
| Plans & limits | `get_plan`, `upgrade_plan`, `cancel_plan` |
| Connected apps | `list_connections` |
| Account identity | `whoami` |
| Approvals | `approve_request` |
| Support | `start_support_chat`, `send_support_message`, `read_support_chat` |

## How It Works

1. **Skill** = procedural knowledge (a markdown file that gets loaded into your agent's context)
2. **MCP server** = tools (the actual API endpoints your agent calls)

You need both. The skill teaches the agent *how* to use the tools effectively — workflows, safety rules, error handling. The MCP server provides the tools themselves.

```
User: "Create me a $25 virtual card"
  ↓
Agent reads SKILL.md → knows the workflow
  ↓
Agent calls create_card(amount_cents: 2500) via MCP
  ↓
Card issued → agent presents summary
```

## Links

- [AgentCard](https://agentcard.sh) — Product website
- [skills.sh](https://skills.sh) — Skill registry
- [MCP Server](https://mcp.agentcard.sh/mcp) — Remote MCP endpoint (OAuth 2.1)
