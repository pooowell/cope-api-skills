---
name: cope-activity
description: Monitor real-time trading activity from tracked smart money wallets. Poll for new activity efficiently, then fetch full trade details when activity is detected.
metadata:
  author: cope-capital
  version: "1.0"
---

# Instructions

Monitor real-time trading activity from your tracked wallets using an efficient polling pattern. Check for new activity with the free polling endpoint, then fetch full trade details when activity is detected.

## When to use this skill
- User wants real-time trading alerts from smart money wallets
- User mentions monitoring, tracking, or watching for trades
- User wants to build trading bots that react to smart money moves
- User needs to filter activity by chain, action, or trade size

## Polling Pattern (Recommended)

**Step 1**: Poll for new activity (free, doesn't count toward daily limit)

```bash
curl -X GET "https://api.cope.capital/v1/activity/poll?since=1707600000" \
  -H "Authorization: Bearer cope_<your_api_key>"
```

### Response
```json
{
  "count": 3,
  "latest_at": 1707603400
}
```

**Step 2**: If count > 0, fetch full activity details (paid endpoint)

```bash
curl -X GET "https://api.cope.capital/v1/activity?since=1707600000&limit=10" \
  -H "Authorization: Bearer cope_<your_api_key>"
```

## Poll for new activity (FREE)

Check if there's new activity without fetching full details:

```bash
curl -X GET "https://api.cope.capital/v1/activity/poll" \
  -H "Authorization: Bearer cope_<your_api_key>"
```

### Filters (all optional)
- `watchlist_id=wl_abc123` - Filter to specific watchlist
- `chain=solana` - Filter by blockchain (`solana`, `base`, or `all`)
- `action=buy` - Filter by trade action (`buy`, `sell`, or `all`)
- `min_usd=1000` - Minimum trade size in USD
- `since=1707600000` - Only include activity after this timestamp

### Rate limit
10 requests per minute for polling endpoint.

## Fetch full activity (PAID)

Get complete trade details with token information:

```bash
curl -X GET "https://api.cope.capital/v1/activity?chain=solana&action=buy&min_usd=5000&limit=20" \
  -H "Authorization: Bearer cope_<your_api_key>"
```

### Response
```json
{
  "activity": [
    {
      "fomo_handle": "icemandot",
      "wallet": "7xKpG8X1N1YKp5Z2bF3Hm9qP4sT6vW8dC9nE5rY3uL1x",
      "chain": "solana",
      "action": "buy",
      "token_mint": "DezX7iJ4W8VqRXPpWLNq6YYr5ky2nrSR1GvSa8L7pump",
      "token_symbol": "BONK",
      "usd_amount": 2400.50,
      "timestamp": 1707603400
    }
  ],
  "cursor": "eyJpZCI6IjQ1NiIsInRzIjoxNzA3NTk5OTAwfQ",
  "has_more": true
}
```

### Filters
All the same filters as polling, plus:
- `handle=icemandot` - Filter to specific Fomo handle
- `limit=50` - Maximum results (1-100, default 50)
- `cursor=xyz` - For pagination

## Example: Real-time monitoring script

```bash
#!/bin/bash
API_KEY="cope_your_api_key"
LAST_CHECK=$(date +%s)

while true; do
  # Poll for new activity
  RESULT=$(curl -s "https://api.cope.capital/v1/activity/poll?since=${LAST_CHECK}" \
    -H "Authorization: Bearer ${API_KEY}")
  
  COUNT=$(echo $RESULT | jq -r '.count')
  
  if [ "$COUNT" != "0" ]; then
    echo "Found $COUNT new trades, fetching details..."
    
    # Fetch full activity
    curl -X GET "https://api.cope.capital/v1/activity?since=${LAST_CHECK}&limit=10" \
      -H "Authorization: Bearer ${API_KEY}" | jq '.'
    
    # Update timestamp
    LAST_CHECK=$(echo $RESULT | jq -r '.latest_at')
  fi
  
  sleep 30
done
```

## Pricing
- **Poll endpoint** (`/v1/activity/poll`): FREE
- **Full activity** (`/v1/activity`): $0.005 per call (after 250 free daily calls)

## Notes
- Activity is updated every 60 seconds from the underlying data source
- Timestamps are Unix milliseconds
- Use the `since` parameter with the `latest_at` value to avoid duplicate data
- Chain validation: only `solana`, `base`, or `all` are accepted
- Action validation: only `buy`, `sell`, or `all` are accepted