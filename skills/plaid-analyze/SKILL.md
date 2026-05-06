---
name: plaid-analyze
description: Analyze financial data from PlaidConnect — spending patterns, subscriptions, cash flow, and merchant breakdowns. Use when the user asks to "analyze spending", "find spending patterns", "categorize transactions", "monthly summary", "income vs expenses", "financial insights", "spending trends", "top merchants", "recurring payments", "budget analysis", "cash flow analysis", "identify subscriptions", or "audit my spending".
version: 1.0.0
author: Lorenzo
license: MIT
metadata:
  hermes:
    tags: [plaid, finance, banking, analysis, spending, budgeting]
    related_skills: [plaid-setup, plaid-link, plaid-query]
prerequisites:
  env_vars: [PLAIDCONNECT_URL, API_KEY]
---

# Plaid Analyze

Analyze financial data from PlaidConnect — spending patterns, categorization, income tracking, and financial insights.

## Analysis Workflow

### Step 1: Fetch Transaction Data

Start with a transaction sync for the relevant item:

```bash
curl -s "${PLAIDCONNECT_URL:-http://localhost:3000}/api/transactions/sync?itemId=ITEM_ID" \
  -H "Authorization: Bearer ${API_KEY}"
```

For historical analysis, use a date range query:

```bash
curl -s "${PLAIDCONNECT_URL:-http://localhost:3000}/api/transactions?itemId=ITEM_ID&startDate=2026-01-01&endDate=2026-05-01&count=500" \
  -H "Authorization: Bearer ${API_KEY}"
```

### Step 2: Parse and Structure

Plaid transactions include these key fields:

- `amount` — transaction amount (negative = outflow, positive = inflow)
- `date` — posting date
- `name` — merchant/description
- `category[]` — hierarchical category array (e.g. `["Food and Drink", "Restaurants"]`)
- `merchant_name` — cleaned merchant name (may differ from `name`)
- `pending` — whether the transaction is still pending
- `payment_channel` — "online", "in store", "other"
- `transaction_type` — "debit", "credit", "transfer", etc.

Filter out pending transactions for analysis (amounts may change).

## Spending by Category

Group transactions by the primary category (first element of `category[]`) and sum amounts:

```
Category            Amount      % of Total
───────────────────────────────────────────
Food and Drink      -$847.23    18.2%
Transportation      -$623.50    13.4%
Shopping            -$512.10    11.0%
Travel              -$498.00    10.7%
Entertainment       -$342.80     7.4%
...
```

When presenting results:
- Show absolute amounts and percentages
- Flag categories that increased >20% month-over-month
- Compare against typical spending benchmarks if known

## Recurring Transactions Detection

Identify likely subscriptions and recurring bills:

1. Group by `name` (or `merchant_name` if available)
2. Flag transactions with the same amount appearing at regular intervals
3. Check for monthly cadence (same merchant, similar amount, ~30 days apart)

Present findings as:
```
Likely Subscriptions:
  Netflix           -$15.49/month
  Spotify           -$10.99/month
  Amazon Prime      -$139.00/year (approx $11.58/mo)
  Gym Membership    -$49.99/month

Total monthly recurring: ~$88.05
```

## Income Analysis

Income transactions are typically positive amounts (`amount > 0`):

1. Filter for positive amounts
2. Group by `name` to identify income sources
3. Compute monthly totals and averages

```
Income Sources (last 3 months):
  Direct Deposit - Employer   +$5,200.00/month (avg)
  Freelance Payment           +$850.00/month (avg, varies)
  
Average monthly net: +$6,050.00
```

## Monthly Cash Flow

Compute net cash flow per month:

```
Month         Inflow      Outflow     Net
────────────────────────────────────────────
Jan 2026      $6,200      $4,850      +$1,350
Feb 2026      $5,800      $5,120      +$680
Mar 2026      $6,400      $5,300      +$1,100
Apr 2026      $6,100      $4,980      +$1,120
```

## Top Merchants

Rank by total spend:

```
Top 10 Merchants (last 90 days):
  1. Amazon             -$742.50
  2. Walmart            -$423.18
  3. Uber               -$318.40
  4. DoorDash           -$287.95
  5. Shell              -$245.00
  ...
```

## Unusual Activity Detection

Flag anomalies:

- Transactions >2 standard deviations above the 90-day average for the merchant
- Transactions at unusual times (weekends for normally weekday-only merchants)
- Duplicate transactions (same amount, same merchant, same day)
- Large round-number transactions (potential fraud indicator)
- New merchants with high amounts (first-time large purchase)

## Presenting Insights

Structure findings as:

1. **Headline** — one-sentence takeaway
2. **Data** — the numbers
3. **Context** — is this normal, good, or concerning?
4. **Recommendation** — actionable suggestion

Example:
```
Headline: Restaurant spending doubled this month.
Data: $847 vs $420 average over the past 3 months.
Context: This is well above your typical range ($380-450/mo).
Recommendation: Review recent restaurant transactions to confirm they're valid
and consider a budget cap if this wasn't intentional.
```

## Data Quality Notes

- Pending transactions may have incorrect amounts — exclude from serious analysis
- Category assignments from Plaid are automated and may be wrong — note when categories look miscategorized
- Transfers between your own accounts appear as paired credit/debit — filter these out for spending analysis
- Some merchants use cryptic names — cross-reference with `merchant_name` field
- Plaid syncs are eventually consistent — recent transactions may take 1-2 days to appear
