---
name: cope-watchlists
description: Manage wallet watchlists for tracking smart money traders by Fomo handle. Create, update, delete watchlists and add/remove traders to monitor their activity.
metadata:
  author: cope-capital
  version: "1.0"
---

# Instructions

Manage watchlists to organize and track smart money wallets by their Fomo handles. Cope Capital automatically resolves Fomo handles to wallet addresses across Solana and Base chains.

## When to use this skill
- User wants to organize wallet tracking into groups
- User mentions specific Fomo handles like "icemandot", "frankdegods"
- User needs to add/remove traders from their monitoring
- User wants to see their current watchlist configuration

## Create a watchlist

```bash
curl -X POST https://api.cope.capital/v1/watchlists \
  -H "Authorization: Bearer cope_<your_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Smart Money Whales",
    "handles": ["icemandot", "frankdegods", "randomxbt"]
  }'
```

### Response
```json
{
  "id": "wl_abc123",
  "name": "Smart Money Whales",
  "created_at": 1707600000,
  "wallets": [
    {
      "handle": "icemandot",
      "solana": "7xKpG8X1N1YKp5Z2bF3Hm9qP4sT6vW8dC9nE5rY3uL1x",
      "base": "0x4f2b0a1e8d3c5g6h7i8j9k0l1m2n3o4p5q6r7s8t9u0v",
      "status": "active",
      "tracked_since": "2026-01-15",
      "note": null
    }
  ]
}
```

## List all watchlists

```bash
curl -X GET https://api.cope.capital/v1/watchlists \
  -H "Authorization: Bearer cope_<your_api_key>"
```

### Response
```json
[
  {
    "id": "wl_abc123",
    "name": "Smart Money Whales",
    "wallet_count": 3,
    "created_at": 1707600000
  }
]
```

## Get watchlist details

```bash
curl -X GET https://api.cope.capital/v1/watchlists/wl_abc123 \
  -H "Authorization: Bearer cope_<your_api_key>"
```

## Update a watchlist

Add and remove handles, or change the name:

```bash
curl -X PUT https://api.cope.capital/v1/watchlists/wl_abc123 \
  -H "Authorization: Bearer cope_<your_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Updated Whale Watchlist",
    "add_handles": ["lookonchain", "whale_alert"],
    "remove_handles": ["oldhandle"]
  }'
```

All fields are optional - only include what you want to change.

## Delete a watchlist

```bash
curl -X DELETE https://api.cope.capital/v1/watchlists/wl_abc123 \
  -H "Authorization: Bearer cope_<your_api_key>"
```

Returns 204 (no content) on success.

## Limits

- **Free tier**: 1 watchlist, 10 handles per watchlist
- **x402 enabled**: 10 watchlists, 100 handles per watchlist

## Handle status meanings
- `active`: Wallet is being tracked and activity will appear
- `onboarding`: Handle found but wallets are being added to tracking
- `not_found`: Fomo handle doesn't exist

## Notes
- Unknown Fomo handles are automatically added to tracking
- Wallet addresses are resolved automatically from Fomo profiles
- Use `cope-fomo-sync` skill to bulk-add all your Fomo follows
- All watchlist management is free and doesn't count toward daily limits