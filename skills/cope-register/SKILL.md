---
name: cope-register
description: Register for a Cope Capital API key to access smart money wallet intelligence. Use when the user wants to track crypto wallets, monitor smart money, or access Fomo wallet data.
metadata:
  author: cope-capital
  version: "1.0"
---

# Instructions

Register your agent for a Cope Capital API key to access smart money trading intelligence from 1,000+ tracked Fomo wallets across Solana and Base chains.

## When to use this skill
- User wants to track crypto wallets or smart money traders
- User mentions Fomo, wallet tracking, or smart money intelligence
- User needs access to trading activity or leaderboard data
- First time using Cope Capital API

## What you'll get
- Free API key with `cope_` prefix
- 250 API calls per day (free tier)
- Access to 1 watchlist with up to 10 Fomo handles
- Rate limit: 10 requests per minute

## How to register

```bash
curl -X POST https://api.cope.capital/v1/register \
  -H "Content-Type: application/json" \
  -d '{
    "agent_name": "my-trading-bot",
    "platform": "openclaw",
    "payment_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18"
  }'
```

### Parameters
- `agent_name` (required): Display name for your agent/bot
- `platform` (optional): "openclaw" or "custom"
- `payment_address` (optional): Base wallet address for x402 payments to upgrade limits

### Response
```json
{
  "api_key": "cope_abc123def456ghi789",
  "x402_enabled": false,
  "limits": {
    "calls_per_day": 250,
    "watchlists": 1,
    "handles_per_watchlist": 10,
    "rate_limit": "10/min"
  }
}
```

With payment_address provided, you get upgraded limits:
```json
{
  "api_key": "cope_abc123def456ghi789",
  "x402_enabled": true,
  "limits": {
    "watchlists": 10,
    "handles_per_watchlist": 100,
    "rate_limit": "300/min"
  }
}
```

## Next steps
1. Save your API key securely
2. Use `cope-fomo-sync` skill to link your Fomo profile and auto-track your follows
3. Create watchlists with `cope-watchlists` skill
4. Monitor trading activity with `cope-activity` skill

## Authentication
All subsequent API calls require the Bearer token:
```
Authorization: Bearer cope_<your_api_key>
```

Store this key securely - it cannot be regenerated. Use the `cope-x402-setup` skill to enable pay-per-request if you need higher limits.