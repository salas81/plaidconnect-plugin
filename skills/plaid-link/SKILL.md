---
name: plaid-link
description: Connect bank accounts via Plaid Link — create link tokens, exchange public tokens, and manage connected financial institutions. Use when the user asks to "connect a bank account", "link bank", "add bank", "create link token", "exchange token", "Plaid Link", "connect financial institution", or "verify bank login".
version: 1.0.0
author: Lorenzo
license: MIT
metadata:
  hermes:
    tags: [plaid, finance, banking, link, authentication]
    related_skills: [plaid-setup, plaid-query, plaid-analyze]
prerequisites:
  env_vars: [PLAIDCONNECT_URL, API_KEY]
---

# Plaid Link

Managing the Plaid Link flow — creating link tokens, exchanging public tokens, and connecting financial institutions.

## API Base

All requests require the `Authorization: Bearer <API_KEY>` header. Use `$PLAIDCONNECT_URL` as the base URL (defaults to `http://localhost:3000` if not set).

## Create a Link Token

Initiates the Plaid Link flow. The returned `linkToken` is passed to the Plaid Link frontend widget.

```bash
curl -s -X POST ${PLAIDCONNECT_URL:-http://localhost:3000}/api/link/token/create \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "user-123",
    "products": ["auth", "transactions"],
    "countryCodes": ["US"],
    "language": "en"
  }'
```

Response:
```json
{
  "linkToken": "link-sandbox-abc123...",
  "expiration": "2026-05-05T20:00:00Z"
}
```

The `linkToken` expires in 4 hours. It's single-use.

## Exchange a Public Token

After the user completes Plaid Link, the frontend receives a `publicToken`. Exchange it server-side:

```bash
curl -s -X POST ${PLAIDCONNECT_URL:-http://localhost:3000}/api/link/token/exchange \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "publicToken": "public-sandbox-abc123...",
    "userId": "user-123"
  }'
```

Response:
```json
{
  "itemId": "abc123",
  "userId": "user-123",
  "institutionName": "Chase",
  "availableProducts": ["auth", "transactions"]
}
```

After exchange, the item is persisted in the token store and you can query accounts, transactions, and identity using the returned `itemId`.

## Sandbox: Create Test Item

In sandbox mode, create a test item without going through Link:

```bash
curl -s -X POST ${PLAIDCONNECT_URL:-http://localhost:3000}/api/sandbox/token/create \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "institutionId": "ins_109508",
    "userId": "user-123",
    "initialProducts": ["auth", "transactions"]
  }'
```

This creates and exchanges everything in one call — returns an `itemId` ready to query.

## Login via Plaid

A specialized Link flow for identity verification (uses Plaid Identity product):

**Step 1:** Create a login-scoped link token:
```bash
curl -s -X POST ${PLAIDCONNECT_URL:-http://localhost:3000}/api/login/link-token \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"userId": "user-123"}'
```

**Step 2:** After user completes Link, verify identity:
```bash
curl -s -X POST ${PLAIDCONNECT_URL:-http://localhost:3000}/api/login/verify \
  -H "Authorization: Bearer ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"publicToken": "public-sandbox-...", "userId": "user-123"}'
```

Returns bank-verified PII: names, emails, phone numbers, addresses.

## List Connected Items

```bash
curl -s ${PLAIDCONNECT_URL:-http://localhost:3000}/api/items?userId=user-123 \
  -H "Authorization: Bearer ${API_KEY}"
```

Access tokens are stripped from the response for security.
