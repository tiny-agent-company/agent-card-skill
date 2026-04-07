---
name: agent-card
description: Manage prepaid virtual Visa cards for AI agents with AgentCard. Create cards, check balances, view credentials, pay for things, close cards, and get support. Use when the user wants to create or manage virtual payment cards for AI agents, pay for online purchases, or set up agent spending.
version: 1.0.1
metadata:
  openclaw:
    emoji: "💳"
    homepage: https://agentcard.sh
    requires:
      anyBins:
        - agent-cards
        - npx
    install:
      - kind: node
        package: agent-cards
        bins: [agent-cards]
---

# AgentCard

You help the user manage prepaid virtual Visa cards through AgentCard MCP tools.

## Scope

This skill operates exclusively against the AgentCard service (`agentcard.sh`) via its official MCP server and CLI. It does not read user files outside AgentCard's own state, does not extract credentials from local config files, and does not install browser extensions or modify other applications' configuration on the user's behalf. Setup steps that require touching system configuration or installing extensions must be completed by the user, not the agent.

## Setup

Tools are prefixed `mcp__agent-cards__*`. If no AgentCard tools are available, read `references/setup.md` and guide the user through connecting the MCP server.

**Important**: If you just added the MCP server in this session, the tools won't be available until the session restarts. Tell the user to restart their agent session, then come back and try again. Do NOT fall back to raw `curl` calls against the API — the API routes are internal and will change. Use either the MCP tools or the CLI.

## Available Tools

| Tool | Purpose |
|------|---------|
| `list_cards` | List all cards with IDs, last four digits, expiry, balance, and status |
| `create_card` | Create a new virtual debit card (requires saved payment method, max $50/card, max 5 active) |
| `check_balance` | Check live balance without exposing credentials |
| `get_card_details` | Get decrypted PAN, CVV, expiry (may require approval) |
| `close_card` | Permanently close a card (irreversible) |
| `list_transactions` | List transactions with amount, merchant, status, timestamps |
| `setup_payment_method` | Save a payment method via Stripe for future card creation |
| `remove_payment_method` | Remove a saved payment method from Stripe |
| `detect_checkout` | Check if current browser tab is a checkout page (requires Chrome extension) |
| `fill_card` | Fill an existing card into a checkout form (requires Chrome extension) |
| `pay_checkout` | Auto-create card and fill checkout form in one step (requires Chrome extension) |
| `submit_user_info` | Submit KYC info (name, DOB, phone) required before first card |
| `approve_request` | Approve or deny a pending approval request |
| `start_support_chat` | Open a new support conversation |
| `send_support_message` | Send a message in a support conversation |
| `read_support_chat` | Read message history of a support conversation |

## Workflows

### Orientation

When the user's intent is unclear, start with `list_cards` to see what exists. Use card IDs from responses in subsequent calls.

### Creating a Card (First Time)

First-time users hit up to 3 prerequisites before a card is actually created. Handle them in order:

1. **Payment method**: Call `create_card`. If it returns `payment_method_required`, call `setup_payment_method` to get a Stripe URL. Tell the user to open it in their browser and save their card. Wait for them to confirm, then retry `create_card`.
2. **KYC (identity verification)**: If `create_card` returns `user_info_required`, collect from the user: first name, last name, date of birth (YYYY-MM-DD), and phone number. Confirm they accept the Stripe Issuing cardholder terms. Call `submit_user_info` with `terms_accepted: true`, then retry `create_card`.
3. **Approval**: If `create_card` returns 202 (approval required), an email is sent to the account owner. Tell the user to check their email. Once approved, call `approve_request` with the returned approval ID.
4. **Beta capacity**: If 403 with `beta_capacity_reached`, the user is waitlisted. Nothing to do.

After clearing prerequisites, `create_card` succeeds. Present: last 4 digits, balance, expiry.

### Creating a Card (Returning User)

1. Ask the user for the funding amount. Convert dollars to cents (e.g. $25 = 2500). Min $1.00, max $50.00.
2. Call `create_card` with `amount_cents`. Optionally `sandbox: true` for testing.
3. Present the card summary.

### Checking Balance

Call `check_balance` with the `card_id`. Format cents as `$XX.XX` (divide by 100).

### Viewing Card Details (PAN/CVV)

Only use `get_card_details` when the user explicitly needs the full card number, CVV, or expiry (e.g. to fill a payment form). This may trigger an approval flow.

**Never proactively display PAN or CVV.** Prefer `check_balance` for routine balance checks.

### Viewing Transactions

Call `list_transactions` with the `card_id`. Optionally filter by `status` (PENDING, SETTLED, DECLINED, REVERSED, EXPIRED, REFUNDED) and `limit`.

### Closing a Card

**Always confirm with the user before calling `close_card`.** State clearly: "This will permanently close the card. Are you sure?" This action is irreversible.

### Paying for Things (Chrome Extension)

For users with the AgentCard Pay Chrome extension:

1. **Detect**: Call `detect_checkout` to check if the current tab is a checkout page. Returns confidence score and detected amount.
2. **Fill**: Call `fill_card` with a `card_id` to fill an existing card into the form. Or use `pay_checkout` to create a new card and fill it in one step.
3. **Verify**: After filling, the user submits the form manually.

If the extension is not installed, the `detect_checkout`, `fill_card`, and `pay_checkout` tools will return an error. Direct the user to install the official AgentCard Pay extension from https://agentcard.sh/extension and follow the instructions there. Do not run extension installation commands on the user's behalf.

### Payment Method Setup

1. Call `setup_payment_method` to get a Stripe checkout URL.
2. Tell the user to open the URL and save their card details.
3. Once saved, the payment method is used automatically for future card creation.
4. To remove: call `remove_payment_method` with the `payment_method_id`.

### Support Chat

1. Call `start_support_chat` with an initial message. Save the returned `conversation_id`.
2. Use `send_support_message` with the `conversation_id` and message.
3. Use `read_support_chat` to check for replies.

## Safety Rules

- **Never proactively display PAN or CVV.** Only show when the user explicitly asks.
- **Always confirm before closing a card.** Closing is permanent and irreversible.
- **Format money as dollars.** Display `$50.00` not `5000 cents`. Divide cents by 100.
- **Track IDs across the conversation.** Remember card IDs, conversation IDs, and approval IDs so the user doesn't have to repeat them.

## Error Handling

- **`beta_capacity_reached` (403)**: User has been waitlisted. Nothing to do but wait.
- **`user_info_required`**: First-time user needs to submit identity info via `submit_user_info` before creating cards.
- **`approval_required` (202)**: Action needs human approval. An email was sent. Guide the user to approve, then call `approve_request`.
- **`payment_method_required`**: No saved payment method. Call `setup_payment_method` first.
- **Card creation fails**: Check if they have 5 active cards (the maximum). Suggest closing unused cards.

## Testing

For testing without real payment, pass `sandbox: true` to `create_card`. This creates a test card immediately.

## CLI Reference

If MCP tools aren't loaded yet (e.g. server was just added, session not restarted), you can use the `agent-cards` CLI as a fallback. **Do not use raw curl/API calls** — the API routes are internal.

```bash
agent-cards cards list                  # list all cards
agent-cards cards create --amount 5     # create a $5 card (interactive prompt)
agent-cards balance <card-id>           # check balance
agent-cards transactions <card-id>      # list transactions
agent-cards payment-method              # manage payment methods
agent-cards setup-mcp                   # configure MCP server in Claude Code
agent-cards support                     # start support chat
```

**Warning**: Several CLI commands (`cards create`, `signup`, `support`) use interactive prompts (inquirer) that crash in non-interactive shells. Do NOT run these from your shell — tell the user to run them in their own terminal. Prefer MCP tools when available.

Commands that are safe to run from any shell: `whoami`, `cards list`, `balance`, `transactions`, `payment-method`.
