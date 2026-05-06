---
name: plaid-setup
description: This skill should be used when the user asks to "set up plaid", "configure plaid", "start plaid server", "plaid credentials", "connect to plaid", or "initialize plaid". Also use when the PlaidConnect server isn't running and needs to be started.
---

# PlaidConnect Setup

Guide for configuring and running the PlaidConnect REST API server.

## Architecture

```
┌─────────────────────┐         HTTP (Bearer token)        ┌──────────────────────┐
│  Claude Code        │ ──────────────────────────────────>│  PlaidConnect Server  │
│  (any machine)      │ <──────────────────────────────────│  (runs anywhere)      │
│  curl to API        │         JSON responses             │  Express + Plaid SDK  │
└─────────────────────┘                                    └──────────────────────┘
```

The server is a standalone process — Claude Code talks to it over HTTP. They can be on the **same machine**, **different machines on a tailnet**, or **separated by the internet** (with TLS). The only requirement: the Claude Code machine can reach the server's port.

## Deployment Options

### Option A: Same machine (dev)
Server runs on `localhost:3000`. Set `PLAIDCONNECT_URL=http://localhost:3000`.

### Option B: Tailscale (home lab)
Run the server on a Raspberry Pi or home server, reach it via Tailscale:

```bash
# On the server machine
tailscale serve --bg 3000   # exposes port 3000 on your tailnet

# On the client machine
export PLAIDCONNECT_URL=http://your-pi.tailnet-name.ts.net:3000
```

### Option C: VPS / Cloud
Deploy to a VPS with a reverse proxy (nginx + Let's Encrypt). Set `PLAIDCONNECT_URL=https://plaid.yourdomain.com`.

## Prerequisites

1. A Plaid account at https://dashboard.plaid.com
2. API credentials (Client ID and Secret) from the Plaid Dashboard
3. Node.js 20+ installed on the server machine

## Configuration

The server reads configuration from `.env`:

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PLAID_CLIENT_ID` | Yes | — | Plaid Dashboard client ID |
| `PLAID_SECRET` | Yes | — | Plaid Dashboard secret (sandbox or production) |
| `PLAID_ENV` | Yes | sandbox | sandbox, development, or production |
| `API_KEY` | Yes | — | Shared secret to secure the API (sent as Bearer token) |
| `CORS_ORIGIN` | No | http://localhost:5173 | Allowed CORS origin |
| `PORT` | No | 3000 | Server port |

The client (Claude Code) needs this env var:

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PLAIDCONNECT_URL` | No | http://localhost:3000 | URL where the server is reachable |

All skills use `$PLAIDCONNECT_URL` in curl commands so they work regardless of where the server lives.

## Starting the Server

On the server machine:

```bash
cd PlaidConnect
npm run dev
```

Verify it's reachable:

```bash
curl ${PLAIDCONNECT_URL:-http://localhost:3000}/health
# {"status":"ok","environment":"sandbox","timestamp":"..."}
```

## Sandbox Mode

When `PLAID_ENV=sandbox`, additional endpoints are available:

- `POST /api/sandbox/token/create` — create test bank items instantly
- `POST /api/sandbox/webhook/fire` — simulate webhook events
- `POST /api/sandbox/transactions/create` — generate test transactions

Use sandbox institution `ins_109508` (First Platypus Bank) for testing.

## Stopping the Server

```bash
lsof -ti:3000 | xargs kill
```

## Token Storage

Access tokens are stored in-memory (a `Map`). They are lost when the server restarts. In production, replace `token-store.ts` with a database-backed implementation.

## Webhooks from Plaid

When the server runs behind Tailscale or on a home network, Plaid can't reach it directly for webhooks. Options:

1. **Polling**: Call `/transactions/sync` periodically instead of relying on webhooks
2. **Tunnel**: Use `tailscale funnel` or Cloudflare Tunnel to expose the webhook endpoint publicly
3. **VPS relay**: Forward webhooks to a lightweight cloud relay that pushes to your server
