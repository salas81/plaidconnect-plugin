# PlaidConnect Plugin

Claude Code plugin for interacting with the [PlaidConnect server](https://github.com/salas81/plaidconnect-server) — financial data through Plaid APIs.

## What this plugin provides

Four skills that let Claude Code help you:

| Skill | What it does |
|---|---|
| `plaid-setup` | Configure and manage the PlaidConnect server |
| `plaid-link` | Connect bank accounts via Plaid Link |
| `plaid-query` | Query accounts, transactions, identity, balances |
| `plaid-analyze` | Analyze spending, find subscriptions, cash flow insights |

## Prerequisites

You need the **PlaidConnect server** running somewhere. This plugin is just the Claude Code interface — it sends HTTP requests to the server.

1. Set up the server: https://github.com/salas81/plaidconnect-server
2. Make sure the server is reachable from where Claude Code runs

## Configuration

Set these env vars in `.claude/settings.local.json`:

```json
{
  "env": {
    "PLAIDCONNECT_URL": "http://your-server:3000",
    "API_KEY": "your-api-key-matching-the-server"
  }
}
```

| Variable | Description |
|---|---|
| `PLAIDCONNECT_URL` | Where the PlaidConnect server is reachable |
| `API_KEY` | Must match the server's `API_KEY` |

## Install

```bash
claude plugins install salas81/plaidconnect-plugin
```

Or clone directly:

```bash
git clone https://github.com/salas81/plaidconnect-plugin.git
```
