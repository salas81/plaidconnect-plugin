---
name: plaid-query
description: This skill should be used when the user asks to "get", "fetch", "query", "show", "pull", or "retrieve" financial data via Plaid. Trigger phrases include "get my transactions", "show my accounts", "fetch balances", "get account numbers", "show identity", "pull transactions", "check balances", "get routing number", "what are my bank accounts", "get spending history".
---

# Plaid Query

Querying financial data from connected Plaid items through the PlaidConnect REST API.

## API Base

All requests require `Authorization: Bearer <API_KEY>`. Use `$PLAIDCONNECT_URL` as base URL (defaults to `http://localhost:3000`).

## Accounts

### List Accounts

```bash
curl -s "${PLAIDCONNECT_URL:-http://localhost:3000}/api/accounts?itemId=abc123" \
  -H "Authorization: Bearer ${API_KEY}"
```

Returns all accounts (checking, savings, credit) with names, types, and mask/partial numbers.

### Get Real-Time Balances

```bash
curl -s "${PLAIDCONNECT_URL:-http://localhost:3000}/api/accounts/balance?itemId=abc123" \
  -H "Authorization: Bearer ${API_KEY}"
```

Returns current and available balances for each account. This makes a live call to the institution — use sparingly.

## Transactions

### Incremental Sync (Recommended)

Use cursor-based sync to get only new transactions:

```bash
curl -s "${PLAIDCONNECT_URL:-http://localhost:3000}/api/transactions/sync?itemId=abc123" \
  -H "Authorization: Bearer ${API_KEY}"
```

First call (no cursor): returns all transactions + `nextCursor`. Subsequent calls with the cursor return only new/modified/removed.

```bash
# Subsequent call with cursor
curl -s "http://localhost:3000/api/transactions/sync?itemId=abc123&cursor=cursor_value" \
  -H "Authorization: Bearer ${API_KEY}"
```

### Get Transactions (Date Range)

```bash
curl -s "${PLAIDCONNECT_URL:-http://localhost:3000}/api/transactions?itemId=abc123&startDate=2026-04-01&endDate=2026-05-01&count=50" \
  -H "Authorization: Bearer ${API_KEY}"
```

Parameters:
- `startDate` — YYYY-MM-DD (default: 30 days ago)
- `endDate` — YYYY-MM-DD (default: today)
- `count` — max transactions (default: 100)
- `offset` — pagination offset (default: 0)

### Force Refresh

Request a fresh pull from the institution:

```bash
curl -s -X POST ${PLAIDCONNECT_URL:-http://localhost:3000}/api/transactions/refresh \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"itemId": "abc123"}'
```

This is async — Plaid returns immediately, then fires a `SYNC_UPDATES_AVAILABLE` webhook when done. After receiving the webhook, call `/api/transactions/sync` to get the new data.

## Identity

Get bank-verified PII from the connected account:

```bash
curl -s "${PLAIDCONNECT_URL:-http://localhost:3000}/api/identity?itemId=abc123" \
  -H "Authorization: Bearer ${API_KEY}"
```

Returns names, emails, phone numbers, and addresses on file at the bank.

## Auth (ACH)

Get account and routing numbers for ACH transfers:

```bash
curl -s "${PLAIDCONNECT_URL:-http://localhost:3000}/api/auth?itemId=abc123" \
  -H "Authorization: Bearer ${API_KEY}"
```

Returns full account numbers, routing numbers, and wire routing numbers.

## Item Status

Get the status of a connected item:

```bash
curl -s "${PLAIDCONNECT_URL:-http://localhost:3000}/api/items/abc123" \
  -H "Authorization: Bearer ${API_KEY}"
```

Shows available products, error status, consent expiration, and institution health.

## Remove an Item

Disconnect and remove a financial institution:

```bash
curl -s -X DELETE "${PLAIDCONNECT_URL:-http://localhost:3000}/api/items/abc123" \
  -H "Authorization: Bearer ${API_KEY}"
```

## Best Practices

- Use `/transactions/sync` (cursor-based) for ongoing updates; use `/transactions` (date range) only for one-off queries
- Call `/accounts/balance` only when you need live balances — it hits the institution
- After a `/transactions/refresh`, wait for the webhook before syncing
- Identity and Auth are separate Plaid products — they may not be available on all items
- All amounts are in the account's native currency
- Transaction amounts: positive = credit/deposit, negative = debit/withdrawal
