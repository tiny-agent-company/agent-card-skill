---
name: agent-card
description: Manage virtual Visa cards for AI agents with AgentCard. Fund a wallet with Apple Pay or Google Pay, create single-use or multi-use cards, check balances, view credentials, pay for things, shop and check out at merchants like DoorDash, close cards, manage plans, and get support. Use when the user wants to create or manage virtual payment cards for AI agents, fund their AgentCard wallet, pay for online purchases, shop on their behalf, set up agent spending, or configure card billing and limits.
license: MIT
compatibility: Requires the AgentCard MCP server (https://mcp.agentcard.sh/mcp).
metadata:
  homepage: https://agentcard.sh
  docs: https://docs.agentcard.sh
  registry: https://skills.sh
  version: "1.4.0"
---

# AgentCard

You help the user manage virtual Visa cards and shop on their behalf through AgentCard MCP tools.

Cards are funded from the user's AgentCard **wallet**. The user adds money with Apple Pay or Google Pay (funds are held as USDC, but the user always funds and spends in USD). First-time funding requires a one-time identity + verification step — the workflows below walk through it.

## Setup

Tools are prefixed `mcp__agent-cards__*`. If no AgentCard tools are available, read `references/setup.md` and guide the user through connecting the MCP server.

**Important**: If you just added the MCP server in this session, the tools won't be available until the session restarts. Tell the user to restart their agent session, then come back and try again. Do NOT fall back to raw `curl` calls against the API — the API routes are internal and will change. Use either the MCP tools or the CLI (see [CLI Reference](#cli-reference)).

## Available Tools

| Tool | Purpose |
|------|---------|
| `list_cards` | List all cards with IDs, last four digits, expiry, balance, and status |
| `create_card` | Create a new virtual Visa card funded from the wallet balance (single-use by default; pass `type: "multi_use"` for subscriptions — see Card Types) |
| `check_balance` | Check live balance without exposing credentials |
| `get_card_details` | Get decrypted PAN, CVV, expiry (may require approval) |
| `close_card` | Permanently close a card (irreversible) |
| `pause_card` | Block new charges on a multi-use card (reversible) |
| `resume_card` | Unblock a paused multi-use card |
| `update_card_limit` | Resize a multi-use card's total limit (raising it draws on the wallet balance) |
| `get_wallet` | Show the funding wallet, its balance, and status |
| `fund_wallet` | Add funds in USD via Apple Pay / Google Pay — returns a single-use payment link to hand the user |
| `start_phone_verification` | Send (or re-send) the one-time verification code required before funding; reports the masked destination (text or email) |
| `verify_phone` | Check the one-time code the user reads back (verification stays fresh for 60 days) |
| `withdraw_wallet` | Withdraw wallet funds to a saved bank account or as USDC on Base (executed by the AgentCard team, typically 1-3 business days) |
| `create_withdrawal_recipient` | Save a bank account (ACH or international wire) as a withdrawal destination |
| `redeem_code` | Apply a promo code and credit the wallet (once per user per code) |
| `list_codes` | List the user's promo-code history — codes they've redeemed or attempted, with state (`used` / `processing` / `available` to retry / `expired`). Cannot discover codes the user has never presented |
| `start_kyc` | Begin conversational identity verification (government ID photo → confirm fields → short browser face scan) |
| `get_kyc_status` | Poll identity verification status until verified |
| `get_rewards` | Show the TOKENBACK rewards balance earned on AI-card spend |
| `redeem_rewards` | Redeem accrued TOKENBACK |
| `list_transactions` | List a single card's transactions with amount, merchant, status, timestamps |
| `list_all_transactions` | List transactions across all your cards in one flat list, each tagged with its card (id + last4) |
| `list_transactions_by_payment_method` | List transactions grouped by payment method (wallet USDC spend, saved-card spend, Apple/Google Pay wallet deposits) with merchant info and the buy order behind each purchase |
| `whoami` | Show the authenticated account: email, name, plan, KYC + account status, and whether the session is a personal login or a third-party OAuth connection |
| `setup_payment_method` | Save a payment method via Stripe |
| `remove_payment_method` | Remove a saved payment method from Stripe |
| `list_payment_methods` | List saved payment methods (id, brand, last4, expiry; marks the default) |
| `set_default_payment_method` | Choose the default saved payment method |
| `buy` | Shop and check out at a supported merchant (DoorDash, Good Eggs, Rappi) from a natural-language request. This is the entire shopping surface — it builds the cart, asks for the delivery address, confirms the total, and places the order, auto-creating a card to pay |
| `get_instructions` | Get the latest shopping/checkout usage guide. Call this BEFORE using `buy` |
| `buy_list_merchants` | List supported merchants and whether the user has linked each one |
| `buy_unlink_merchant` | Disconnect a linked merchant (drops the saved session; the user must re-link before shopping it again) |
| `submit_user_info` | Submit the phone number + terms acceptance required before the first card |
| `get_plan` | Show current plan, card limits, monthly usage, and upgrade options |
| `upgrade_plan` | Start a Stripe Checkout to upgrade to the Basic plan |
| `cancel_plan` | Cancel the active paid subscription (revert to Free) |
| `list_connections` | List the third-party apps connected to the account via OAuth (read-only) |
| `approve_request` | Approve or deny a pending approval request |
| `start_support_chat` | Open a new support conversation |
| `send_support_message` | Send a message in a support conversation |
| `read_support_chat` | Read message history of a support conversation |

## Workflows

### Orientation

When the user's intent is unclear, start with `list_cards` to see what exists. Use card IDs from responses in subsequent calls. For money questions, `get_wallet` shows the balance that backs card creation.

### Funding the Wallet

1. Call `get_wallet` to see the current balance and wallet status.
2. Call `fund_wallet` with the amount in cents and the user's `payment_method` — `"apple_pay"` (the default) or `"google_pay"`; ask which they use if you don't know. It returns a **single-use payment link** — relay it to the user EXACTLY as returned (never shorten, unfurl, or paraphrase the URL) and tell them to open it and pay. Funds land in the wallet within minutes.
3. **If `fund_wallet` reports verification is required**: it automatically sends the user a one-time code and reports where it went (text message or email, masked). Ask the user for the code, call `verify_phone` with it, then call `fund_wallet` again. The verification stays fresh for 60 days.
4. **If the code never arrives**: call `start_phone_verification` to re-send it (rate-limited to a few sends per 10 minutes — don't spam it).
5. **If the code can't be sent at all** (no verified identity on file yet — typical for brand-new accounts): the user must complete identity first. Run the Creating a Card ladder (`submit_user_info`, then `start_kyc`) — the verified identity supplies the phone number, then funding works.
6. The payment link is single-use — if it gets consumed or expires, just call `fund_wallet` again for a fresh one.
7. If the user has a promo code, `redeem_code` applies it and credits the wallet directly; `list_codes` shows their code history (including failed attempts they can retry) — it cannot discover codes the user has never presented.

### Creating a Card

1. Ask the user for the card amount. Convert dollars to cents (e.g. $25 = 2500). Limits depend on the plan — call `get_plan` if unsure.
2. Call `create_card` with `amount_cents`. The card is funded from the wallet balance.
3. Resolve any gating status, then retry `create_card`:
   - **`user_info_required`**: collect the user's phone number and confirm they accept the cardholder terms. Call `submit_user_info` with `terms_accepted: true`.
   - **`kyc_required`**: run `start_kyc` — identity verification is conversational: the user sends a photo of their government ID (read automatically, they confirm the extracted fields), fills any missing fields, then completes a short browser face scan. Poll `get_kyc_status` until verified.
   - **`wallet_funding_required`**: the wallet doesn't hold enough. Run the Funding the Wallet workflow for at least the shortfall, then retry.
   - **`deposit_confirming`**: the money is already on the way — wait the suggested retry interval and call `create_card` again. Do NOT ask the user to pay again.
4. **If 202 (approval required)**: an email is sent to the account owner. Tell the user to check their email and approve. Once approved, call `approve_request` with the approval ID.
5. **On success**: present the card summary (last 4, balance, expiry).

All cards are live — there is no test/sandbox mode on the consumer MCP. A card spends the user's real wallet balance, so confirm before creating one.

### Card Types: Single-use vs Multi-use

- Cards are **single-use by default**: they close automatically after the first approved charge. Right for one-off purchases.
- For **subscriptions or any merchant that charges repeatedly**, create a multi-use card: `create_card` with `type: "multi_use"` (optional `expires_at` auto-closes it). Multi-use cards stay open until their total limit is spent.
- Manage multi-use cards: `pause_card` blocks new charges (reversible), `resume_card` unblocks, `update_card_limit` resizes the total limit (raising it draws on the wallet balance).

### AI Cards & TOKENBACK

`create_card` with `scope_preset: "ai_labs"` makes an **AI card**: locked to AI-lab merchants (OpenAI, Anthropic, Gemini — anything else declines at authorization) and earning TOKENBACK rewards on eligible spend. `get_rewards` shows the accrued balance; `redeem_rewards` redeems it.

### Withdrawing from the Wallet

1. Confirm the amount and destination with the user first — withdrawals move real money.
2. **To a bank**: the user needs a saved recipient — `create_withdrawal_recipient` saves an ACH or international wire destination. Then `withdraw_wallet` with the recipient.
3. **As USDC on Base**: `withdraw_wallet` with a `0x…` address the user controls.
4. Withdrawals are executed by the AgentCard team, typically within 1-3 business days, and the user is emailed at each step.

### Buying & Checking Out (Shopping)

The `buy` tool is the entire shopping surface for supported merchants (DoorDash is live; Good Eggs and Rappi where available). Do **not** look for separate search, cart, or checkout tools — `buy` runs the whole flow conversationally and creates a card under the hood to pay.

1. **Get the latest guide first.** Call `get_instructions` before using `buy` — it returns the current shopping/checkout usage guide (the flow evolves over time).
2. **Start an order** with a natural-language request, e.g. `buy("order a caesar salad from Zuni on DoorDash")`. It returns a `conversation_id`.
3. **Continue the same order** by calling `buy` again with that `conversation_id` to answer its questions (delivery address, confirm the cart + total) or to place the order. Keep passing the same `conversation_id` so the cart and context persist.
4. **Checkout only happens after the user explicitly confirms.** This is a real purchase — confirm the cart and total with the user before placing the order.
5. Use `buy_list_merchants` to see which merchants are available and whether the user has linked each. To disconnect one, use `buy_unlink_merchant` — confirm with the user first, since it drops the saved session and they must re-link before shopping that merchant again.

### Checking Balance

Call `check_balance` with the `card_id`. Format cents as `$XX.XX` (divide by 100). For the wallet balance, call `get_wallet`.

### Viewing Card Details (PAN/CVV)

Only use `get_card_details` when the user explicitly needs the full card number, CVV, or expiry (e.g. to fill a payment form). This may trigger an approval flow.

**Never proactively display PAN or CVV.** Prefer `check_balance` for routine balance checks.

### Viewing Transactions

Call `list_transactions` with the `card_id` for a single card. Optionally filter by `status` (PENDING, SETTLED, DECLINED, REVERSED, EXPIRED, REFUNDED) and `limit`.

To see activity across every card at once, call `list_all_transactions` (no `card_id`) — it returns a flat list of all your transactions, each tagged with the card it belongs to. Supports `limit`, `offset`, and `status`.

For "what did I spend, and how did I pay" questions, call `list_transactions_by_payment_method` — it groups the account's activity by payment method (wallet USDC spend, legacy saved-card spend, and Apple Pay / Google Pay wallet deposits), with all-time totals per method, merchant name + MCC per row, and the buy order (merchant, order id, total) behind purchases made through `buy`. Line-level items aren't stored server-side — fetch a merchant's order history via `buy` when you need them.

### Closing a Card

**Always confirm with the user before calling `close_card`.** State clearly: "This will permanently close the card. Are you sure?" This action is irreversible.

### Payment Method Setup

Saved payment methods are managed via Stripe and show up as their own spend group in `list_transactions_by_payment_method`; card creation itself is funded from the wallet.

1. Call `setup_payment_method` to get a Stripe checkout URL, and tell the user to open it and save their card details.
2. To remove: call `remove_payment_method` with the `payment_method_id`.
3. To see saved methods and which is the default: call `list_payment_methods`.
4. To change the default: call `set_default_payment_method` with the `payment_method_id`.

### Plans, Limits & Upgrades

1. Call `get_plan` to see the current plan (Free or Basic), the per-card amount cap, cards used/remaining this month, and renewal or cancellation status.
2. Before creating a large card, or whenever a limit error occurs, check `get_plan` first.
3. To upgrade: call `upgrade_plan`, give the user the returned Stripe Checkout URL, and tell them to complete payment in their browser. The plan updates automatically afterward — confirm with `get_plan`.
4. To cancel a paid subscription: call `cancel_plan` (reverts to Free). Confirm with the user first.

### Connected Apps

Call `list_connections` to show which third-party apps the user has connected to their AgentCard account via OAuth, when each was connected, and whether it is still active. Read-only — to revoke an app, the user runs `agent-cards connections revoke <clientId>` in the CLI.

### Support Chat

1. Call `start_support_chat` with an initial message. Save the returned `conversation_id`.
2. Use `send_support_message` with the `conversation_id` and message.
3. Use `read_support_chat` to check for replies.

## Safety Rules

- **Never proactively display PAN or CVV.** Only show when the user explicitly asks.
- **Always confirm before closing a card.** Closing is permanent and irreversible.
- **Confirm before creating a card.** Every card is live and spends the user's real wallet balance.
- **Confirm before placing an order with `buy`.** Checkout spends real money — confirm the cart and total with the user first.
- **Confirm before withdrawing.** Withdrawals move real money out of the wallet — confirm amount and destination.
- **Relay payment links exactly.** Funding checkout links are single-use — pass them to the user verbatim, never shortened or paraphrased.
- **Confirm before unlinking a merchant.** `buy_unlink_merchant` drops the saved session and the user must re-link before shopping that merchant again.
- **Format money as dollars.** Display `$50.00` not `5000 cents`. Divide cents by 100.
- **Track IDs across the conversation.** Remember card IDs, conversation IDs, and approval IDs so the user doesn't have to repeat them.

## Error Handling

- **`phone_verification_required`** (funding): a one-time code is needed. `fund_wallet` auto-sends it and reports the masked destination — collect the code, call `verify_phone`, then `fund_wallet` again. If the code was reported as NOT sent, call `start_phone_verification`; if that fails because there's no identity on file, complete `submit_user_info` / `start_kyc` first.
- **`user_info_required`**: first-time user needs to submit their phone number + accept terms via `submit_user_info` before creating cards.
- **`kyc_required`**: run `start_kyc` (ID photo → fields → face scan), poll `get_kyc_status`, then retry.
- **`wallet_funding_required`**: the wallet balance can't cover the card — run the Funding the Wallet workflow, then retry.
- **`deposit_confirming`**: a deposit is settling — wait the suggested interval and retry. Never ask the user to pay again.
- **`amount_out_of_range`** (funding): the amount is outside the provider's bounds — the error names the exact allowed range; adjust and retry.
- **`approval_required` (202)**: action needs human approval. An email was sent. Guide the user to approve, then call `approve_request`.
- **`amount_exceeds_limit` / `card_limit_reached` (400)**: a plan limit was hit. Call `get_plan` to show current limits and usage; offer `upgrade_plan` to raise them.
- **Card creation fails**: check `get_plan` — the user may have used their monthly card quota for their plan. Suggest upgrading or waiting until next month.

## CLI Reference

If MCP tools aren't loaded yet (e.g. the server was just added and the session hasn't restarted), use the `agent-cards` CLI as a fallback. **Do not use raw curl/API calls** — the API routes are internal.

```bash
agent-cards cards list                  # list all cards
agent-cards cards create --amount 5     # create a $5 card (interactive; walks through identity + funding on first run)
agent-cards wallet                      # show the wallet + balance (provisions one on first run)
agent-cards wallet fund --amount 50     # add funds via Apple/Google Pay (first time: prompts for a one-time verification code)
agent-cards balance <card-id>           # check balance
agent-cards transactions <card-id>      # list one card's transactions
agent-cards transactions                # list transactions across all cards
agent-cards payment-method              # manage payment methods
agent-cards setup-mcp                   # configure MCP server in Claude Code
agent-cards support                     # start support chat
```

**Warning**: Several CLI commands (`cards create`, `wallet fund`, `signup`, `support`) use interactive prompts that crash or bail in non-interactive shells (`wallet fund` prompts for the one-time verification code when the account isn't verified yet). Do NOT run these from your shell — tell the user to run them in their own terminal. Commands safe to run from any shell: `whoami`, `cards list`, `balance`, `transactions`, `payment-method`, `wallet` (show only). Prefer MCP tools when available.
