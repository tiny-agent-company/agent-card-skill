# AgentCard MCP Server Setup

Connect the AgentCard MCP server so your AI agent can manage virtual cards directly.

## Step 1: Install CLI and Create Account

```bash
npm install -g agent-cards
```

**The signup command requires interactive input (email prompt) and will crash in non-interactive shells.** Tell the user to run this in their own terminal:

```bash
agent-cards signup
```

They enter their email, receive a magic link, click it, and they're in. Wait for them to confirm they're logged in before proceeding. You can verify with `agent-cards whoami`.

## Step 2: Connect the MCP Server

### Claude Code

The CLI has a built-in setup command that handles everything:

```bash
agent-cards setup-mcp
```

**Important**: `setup-mcp` calls signup internally if not logged in, which uses interactive prompts. Make sure the user is already logged in (`agent-cards whoami` returns an email) before running `setup-mcp`. If they're already logged in, `setup-mcp` runs non-interactively and works fine from any shell.

If `setup-mcp` doesn't work, add manually:

```bash
claude mcp add --transport http agent-cards https://mcp.agentcard.sh/mcp
```

**After adding the MCP server, the user must restart their agent session for the tools to load.** Tell them: "Please restart your session (exit and re-enter) so the AgentCard tools load." Do NOT try to use the tools in the same session — they will not be available.

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

Direct the user to follow the integration instructions at https://agentcard.sh/setup/claude-desktop. Do not read tokens from local files or modify Claude Desktop's config on the user's behalf — the user should complete this setup themselves through the documented flow.

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

Note: Several CLI commands (`cards create`, `signup`, `support`) use interactive prompts via inquirer that crash in non-interactive shells. These commands must be run in the user's own terminal. Prefer MCP tools when available.
