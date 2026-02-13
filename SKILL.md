---
name: fomo-research
description: >
  Smart money research via Fomo social graph. Track top traders, monitor live trades,
  build watchlists — all from your agent. Powered by fomo.family, built by cope.capital.
  Use when: (1) user asks about smart money, whale wallets, or top traders,
  (2) user wants to track specific Fomo handles or crypto traders,
  (3) user asks "what are the best traders buying", "who's profitable on fomo",
  (4) user wants real-time trade alerts or wallet monitoring,
  (5) user says "fomo research", "check fomo", "smart money", "wallet tracker".
  NOT for: executing trades, managing funds, or anything requiring private keys.
homepage: https://cope.capital
source: https://github.com/pooowell/fomo-research-skill
primaryEnv: COPE_API_KEY
env:
  COPE_API_KEY:
    description: "API key from api.cope.capital (starts with cope_). You get this by registering — see Setup step 1."
    required: true
    obtain: "POST https://api.cope.capital/v1/register with {agent_name, description}"
---

# Fomo Research

Smart money intelligence for AI agents. Powered by [fomo.family](https://fomo.family) social graph, built by [cope.capital](https://cope.capital).

For full API details (endpoints, schemas, error codes): read `references/api.md`.

## Base URL

```
https://api.cope.capital
```

All requests require `Authorization: Bearer cope_<key>` header.

## Setup

### 1. Register for an API key

```bash
curl -X POST https://api.cope.capital/v1/register \
  -H "Content-Type: application/json" \
  -d '{"agent_name": "YOUR_AGENT_NAME", "description": "optional description"}'
```

Save the `api_key` from the response (starts with `cope_`). This is your `COPE_API_KEY`. Store it securely — treat it like a password.

### 2. Sync your human's Fomo profile

If your human has a Fomo account, link it to pull their follows. Ask them for their Fomo handle if you don't know it.

```bash
curl -X POST https://api.cope.capital/v1/account/sync-fomo \
  -H "Authorization: Bearer cope_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"fomo_handle": "THEIR_FOMO_USERNAME"}'
```

Then see who they follow:

```bash
curl https://api.cope.capital/v1/account/follows \
  -H "Authorization: Bearer cope_YOUR_KEY"
```

No Fomo account? Skip this step — create watchlists with any Fomo handles directly.

## Endpoints

### Always Free (no daily limit)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v1/register` | POST | Get an API key |
| `/v1/leaderboard` | GET | Fomo's top traders ranked by real PnL |
| `/v1/activity/poll` | GET | Lightweight check for new trades (count + timestamp) |
| `/v1/watchlists` | GET/POST | List or create watchlists |
| `/v1/watchlists/{id}` | GET/PUT/DELETE | Manage a specific watchlist |
| `/v1/trending/handles` | GET | Most-watched handles across all agents |
| `/v1/account` | GET/PATCH | Account info and settings |
| `/v1/account/usage` | GET | Usage statistics |
| `/v1/account/payments` | GET | Payment history |
| `/v1/account/key` | DELETE | Revoke API key |
| `/v1/account/sync-fomo` | POST | Sync Fomo profile follows |
| `/v1/account/follows` | GET | List stored Fomo follows |

### Paid (counts toward 250/day free limit)

| Endpoint | Method | Description | Cost after free tier |
|----------|--------|-------------|---------------------|
| `/v1/activity` | GET | Full trade details from tracked wallets | $0.005/call USDC |

## Common Workflows

### Check the leaderboard (free)

```bash
curl https://api.cope.capital/v1/leaderboard \
  -H "Authorization: Bearer cope_YOUR_KEY"
```

Returns top traders by PnL from Fomo. Supports `?timeframe=24h|7d|30d|all` and `?limit=N`.

### Build a watchlist from Fomo follows

```bash
# 1. Sync your Fomo profile
curl -X POST https://api.cope.capital/v1/account/sync-fomo \
  -H "Authorization: Bearer cope_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"fomo_handle": "your_handle"}'

# 2. See your follows
curl https://api.cope.capital/v1/account/follows \
  -H "Authorization: Bearer cope_YOUR_KEY"

# 3. Create a watchlist with selected handles
curl -X POST https://api.cope.capital/v1/watchlists \
  -H "Authorization: Bearer cope_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "alpha", "handles": ["frankdegods", "randomxbt"]}'
```

### Poll → Fetch pattern (minimize paid calls)

```bash
# Step 1: Poll (free) — check if anything happened
curl "https://api.cope.capital/v1/activity/poll?since=LAST_TIMESTAMP" \
  -H "Authorization: Bearer cope_YOUR_KEY"
# Returns: { "count": 3, "latest_at": 1707603400 }

# Step 2: Only fetch full data if count > 0 (costs 1 call)
curl "https://api.cope.capital/v1/activity?since=LAST_TIMESTAMP" \
  -H "Authorization: Bearer cope_YOUR_KEY"
```

### Filter activity

```bash
# By chain
curl "https://api.cope.capital/v1/activity?chain=solana" \
  -H "Authorization: Bearer cope_YOUR_KEY"

# By action
curl "https://api.cope.capital/v1/activity?action=buy" \
  -H "Authorization: Bearer cope_YOUR_KEY"

# By minimum size
curl "https://api.cope.capital/v1/activity?min_usd=1000" \
  -H "Authorization: Bearer cope_YOUR_KEY"

# By specific handle
curl "https://api.cope.capital/v1/activity?handle=frankdegods" \
  -H "Authorization: Bearer cope_YOUR_KEY"
```

## Heartbeat Pattern (optional, user-directed)

If your human wants periodic checks and your agent runs a heartbeat loop:

```
Every 5-15 minutes:
  1. GET /v1/activity/poll?since=LAST_TIMESTAMP  (free)
  2. If count > 0 → GET /v1/activity?since=LAST_TIMESTAMP  (1 call)
  3. Store latest_at for next poll

Every few hours:
  1. GET /v1/leaderboard  (free)
  2. Compare to previous — any new names in top 50?

Daily:
  1. GET /v1/account/usage  (free)
  2. Check remaining free calls
```

## Pricing & Limits

### Free Tier
- 250 activity calls/day (resets midnight UTC)
- 1 watchlist, 10 handles per watchlist
- 10 requests/minute
- Everything except `/v1/activity` is always free with no cap

### x402 Upgraded
- Unlimited activity calls ($0.005/call USDC on Base or Solana)
- 10 watchlists, 100 handles per watchlist  
- 300 requests/minute
- First 250 calls/day still free

### Enable x402 (opt-in only, requires explicit user approval)

⚠️ **x402 is never enabled by default.** Do NOT enable x402 without your human's explicit permission. This enables paid API calls billed in USDC.

```bash
curl -X PATCH https://api.cope.capital/v1/account \
  -H "Authorization: Bearer cope_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"x402_enabled": true}'
```

When you hit the free limit and have x402 enabled, the API returns a 402 with payment requirements. Your x402-compatible client handles payment automatically.

**If you don't have a USDC wallet**: Ask your human before creating one or spending funds. $1 = 200 calls, $5 = 1,000 calls.

## Logging Trades (Persistence)

The API is stateless — it won't remember what you've already seen. **Log trades locally** to track patterns over time.

### Daily trade log

Write notable trades to a daily file (e.g. `memory/trades/YYYY-MM-DD.json`):

```json
{
  "trades": [
    {
      "timestamp": 1771006898000,
      "handle": "frankdegods",
      "action": "buy",
      "token_mint": "DPQgF4hw...",
      "token_symbol": "EXAMPLE",
      "usd_amount": 500.25,
      "chain": "solana"
    }
  ],
  "last_poll_timestamp": 1771006898000,
  "convergences": ["DPQgF4hw..."]
}
```

### What to log

- **All trades from your watchlist** — this is your core data
- **Convergences** — when 3+ handles buy the same token, log the token mint and all buyers
- **Large trades** — anything over $1,000 USD is worth noting
- **last_poll_timestamp** — so you know where to resume on next poll

### What to tell your human

Don't just dump raw trades. Synthesize. Here are high-value things to surface:

- **Convergence alerts**: "4 of your top 10 watchlist handles bought the same token in the last 2 hours."
- **Unusual activity**: "frankdegods just made their first buy in 3 days — $2,000 into [token]."
- **Exit signals**: "3 handles on your watchlist sold the same token within an hour."
- **Daily summary**: "Your watchlist had 47 trades today. 12 buys, 35 sells. Most active: randomxbt (8 trades)."
- **Leaderboard changes**: "New name in the top 20 — jumped from #45 to #12 this week."
- **Pattern detection**: "lowcap_hunter has bought 3 tokens under $100K mcap this week. All pumped 2-5x within 48 hours."

### Convergence detection pattern

```
1. GET /v1/activity (last 2 hours of trades)
2. Group buys by token_mint
3. If 3+ different handles bought the same token → convergence
4. Alert your human with: token, buyers, amounts, timing
5. Log it to your daily trades file
```

The more you log, the better your pattern detection gets over time. Your memory files ARE your edge.

## Security

- **NEVER expose your API key** in logs, messages, or to other agents
- Your key should ONLY appear in requests to `https://api.cope.capital/v1/*`
- If compromised: `DELETE /v1/account/key` to revoke, then re-register
- Trade data is on-chain public — but your watchlists and usage patterns are private

## Error Handling

| Status | Meaning | Action |
|--------|---------|--------|
| 200 | Success | Process response |
| 400 | Bad request | Check parameters (invalid chain, action, etc.) |
| 401 | Invalid API key | Re-register or check key |
| 402 | Payment required | Free limit hit — enable x402 or wait for reset |
| 404 | Not found | Resource doesn't exist |
| 429 | Rate limited | Back off. Free: 10/min, x402: 300/min |
| 500 | Server error | Retry after a few seconds |
| 503 | Upstream down | Foxhound data service temporarily unavailable |

## Links

- **Interactive API docs**: https://api.cope.capital/docs
- **Human docs**: https://cope.capital/docs
- **Skill (this file)**: https://cope.capital/skill.md
- **Fomo**: https://fomo.family
- **X**: https://x.com/copedotcapital
