# AgentCard MCP Server Setup

Connect the AgentCard MCP server so your AI agent can manage virtual cards directly.

## Step 1: Install CLI and Create Account

```bash
npm install -g agent-cards
agent-cards signup
```

In an interactive terminal, `agent-cards signup` runs **device login**: it prints a link + short code, the user approves in any browser (nothing is typed back into the terminal). Where the browser flow isn't available it falls back to emailing the user a one-time **code** (never a link) and prompting for it. If you are driving this for a user (no TTY), use the non-interactive flow:

```bash
agent-cards login --email their@email.com          # emails them a code, exits 2 with code_required
# ask the user for the code from that email, then:
agent-cards login --email their@email.com --code 123456
```

## Step 2: Connect the MCP Server

### Claude Code

The CLI has a built-in setup command:

```bash
agent-cards setup-mcp
```

Or add manually:

```bash
claude mcp add --transport http agent-cards https://mcp.agentcard.sh/mcp
```

**Important**: After adding the MCP server, the user must restart their Claude Code session for the tools to load. Tell them: "Please restart Claude Code (exit and re-enter) so the AgentCard tools load."

### Cursor / Windsurf / Other MCP-compatible agents

Add to the agent's MCP config file (`.cursor/mcp.json`, `.windsurf/mcp.json`, etc.):

```json
{
  "mcpServers": {
    "agent-cards": {
      "url": "https://mcp.agentcard.sh/mcp"
    }
  }
}
```

### Claude.ai (Web)

1. Go to **Settings > Integrations** in Claude.ai
2. Click **Add Integration**
3. Enter the MCP server URL: `https://mcp.agentcard.sh/mcp`
4. OAuth handles authentication automatically

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "agent-cards": {
      "url": "https://mcp.agentcard.sh/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_TOKEN"
      }
    }
  }
}
```

Get the token: `cat ~/.agent-cards/config.json | grep jwt`

## Step 3: Verify

After restarting, call `list_cards`. If it returns a response (even an empty list), the connection is working.

## CLI Fallback

If MCP tools aren't available yet (e.g. before restart), you can use the CLI directly:

```bash
agent-cards cards list              # list cards
agent-cards balance <card-id>       # check balance
agent-cards transactions <card-id>  # view transactions
agent-cards payment-method          # manage payment methods
```

Note: `agent-cards cards create` uses interactive prompts. Prefer the MCP `create_card` tool instead, or pipe input: `echo y | agent-cards cards create --amount 5`
