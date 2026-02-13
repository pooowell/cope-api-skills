---
name: cope-leaderboard
description: Get ranked leaderboard of top performing wallets by PnL, trading volume, and win rates. Discover the most profitable smart money traders across different timeframes.
metadata:
  author: cope-capital
  version: "1.0"
---

# Instructions

Access ranked leaderboards of top performing smart money wallets from the Fomo ecosystem. See which traders are generating the highest PnL across different timeframes.

## When to use this skill
- User wants to discover top performing traders
- User asks about "best wallets", "top traders", or "profitable wallets"
- User wants PnL rankings or performance metrics
- User needs to identify smart money to follow

## Get leaderboard

```bash
curl -X GET "https://api.cope.capital/v1/leaderboard?timeframe=7d&limit=20" \
  -H "Authorization: Bearer cope_<your_api_key>"
```

### Parameters
- `timeframe`: Time period for rankings
  - `24h` - Past 24 hours
  - `7d` - Past 7 days (default)
  - `30d` - Past 30 days  
  - `all` - All-time performance
- `limit`: Number of wallets to return (1-100, default 50)

### Response
```json
{
  "leaderboard": [
    {
      "handle": "frankdegods",
      "display_name": "[PN] frank",
      "solana_address": "ABC123...",
      "base_address": "0xDEF456...",
      "pnl": 291361.03,
      "num_trades": 1398,
      "swap_count": 3676,
      "total_volume": 28334001.76,
      "followers": 71948,
      "total_holdings": 14
    },
    {
      "handle": "icemandot",
      "display_name": "iceman",
      "solana_address": "XYZ789...",
      "base_address": null,
      "pnl": 156420.89,
      "num_trades": 892,
      "swap_count": 2341,
      "total_volume": 15670000.45,
      "followers": 45231,
      "total_holdings": 8
    }
  ],
  "timeframe": "7d",
  "cached": true
}
```

## Field explanations

- `pnl`: Profit/Loss in USD for the selected timeframe
- `num_trades`: Number of completed trades (positions opened and closed)
- `swap_count`: Total number of swaps/transactions
- `total_volume`: Total trading volume in USD
- `followers`: Number of Fomo followers
- `total_holdings`: Current number of tokens held
- `cached`: Whether response was served from cache (updated every 5 minutes)

## Example: Find top performers by timeframe

**Top daily performers**
```bash
curl -X GET "https://api.cope.capital/v1/leaderboard?timeframe=24h&limit=10" \
  -H "Authorization: Bearer cope_<your_api_key>"
```

**All-time legends**
```bash
curl -X GET "https://api.cope.capital/v1/leaderboard?timeframe=all&limit=50" \
  -H "Authorization: Bearer cope_<your_api_key>"
```

**Monthly winners**
```bash
curl -X GET "https://api.cope.capital/v1/leaderboard?timeframe=30d&limit=25" \
  -H "Authorization: Bearer cope_<your_api_key>"
```

## Use cases

**Discover new traders to follow**
```bash
# Get top 20 performers this week
curl -X GET "https://api.cope.capital/v1/leaderboard?timeframe=7d&limit=20" \
  -H "Authorization: Bearer cope_<your_api_key>" | jq -r '.leaderboard[].handle'
```

**Build a watchlist from top performers**
1. Get leaderboard handles
2. Use `cope-watchlists` skill to create a watchlist with the top handles
3. Monitor their activity with `cope-activity` skill

**Track follower growth**
- Compare `followers` counts over time to see which traders are gaining popularity
- High follower count often correlates with consistent performance

## Performance metrics

**Understanding the numbers:**
- High `pnl` = Profitable in the timeframe
- High `num_trades` + positive PnL = Active and successful
- High `total_volume` = Large position sizes or very active
- `swap_count` > `num_trades` = Multiple entries/exits per position
- High `followers` = Community-recognized trader

**Quality indicators:**
- Consistent positive PnL across timeframes
- Growing follower count
- Reasonable trade frequency (not just luck on a few trades)

## Pricing
- $0.005 per call (after 250 free daily calls)
- Data is cached for 5 minutes per timeframe to optimize performance

## Notes
- Leaderboard is sourced directly from Fomo's verified PnL calculations
- Only includes wallets with completed trades in the timeframe
- Rankings update every 5 minutes
- Addresses may be null if the trader hasn't linked wallets to their Fomo profile