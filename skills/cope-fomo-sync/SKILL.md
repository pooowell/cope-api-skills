---
name: cope-fomo-sync
description: Link your Fomo profile to automatically sync your follows and track all the wallets you follow on Fomo. Streamlines watchlist creation from your existing follows.
metadata:
  author: cope-capital
  version: "1.0"
---

# Instructions

Connect your Fomo profile to automatically track all the wallets you follow. This is the fastest way to start monitoring smart money without manually entering Fomo handles.

## When to use this skill
- User has a Fomo account and wants to track their follows
- User mentions "sync my follows", "import from Fomo", or similar
- User wants to automatically track traders they already follow
- User needs to bulk-import wallet tracking

## Workflow: Sync → Browse → Create Watchlists

1. **Sync your Fomo profile** to store all your follows
2. **Browse your follows** to see who you're tracking
3. **Create watchlists** from subsets of your follows

## Step 1: Link and sync your Fomo profile

```bash
curl -X POST https://api.cope.capital/v1/account/sync-fomo \
  -H "Authorization: Bearer cope_<your_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "fomo_handle": "icemandot"
  }'
```

### Response
```json
{
  "fomo_handle": "icemandot",
  "follows_synced": 47,
  "synced_at": 1707782400000
}
```

## Step 2: Browse your synced follows

```bash
curl -X GET https://api.cope.capital/v1/account/follows \
  -H "Authorization: Bearer cope_<your_api_key>"
```

### Response
```json
{
  "fomo_handle": "icemandot",
  "synced_at": 1707782400000,
  "follows": [
    {
      "handle": "frankdegods",
      "solana_address": "ABC123...",
      "base_address": "0xDEF456...",
      "active": true
    },
    {
      "handle": "whale_trader",
      "solana_address": "XYZ789...",
      "base_address": null,
      "active": true
    },
    {
      "handle": "old_account",
      "solana_address": null,
      "base_address": null,
      "active": false
    }
  ],
  "total": 47
}
```

## Step 3: Create watchlists from your follows

Use the standard watchlist creation with handles from your follows:

```bash
curl -X POST https://api.cope.capital/v1/watchlists \
  -H "Authorization: Bearer cope_<your_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Fomo Follows",
    "handles": ["frankdegods", "whale_trader", "top_performer"]
  }'
```

## Re-syncing your follows

Call the sync endpoint again anytime to refresh your follows:

```bash
curl -X POST https://api.cope.capital/v1/account/sync-fomo \
  -H "Authorization: Bearer cope_<your_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "fomo_handle": "icemandot"
  }'
```

This will:
- Update your stored follows list
- Add any new follows to tracking
- Mark inactive/unfollowed accounts appropriately

## Example: Complete setup workflow

```bash
#!/bin/bash
API_KEY="cope_your_api_key"

echo "1. Syncing Fomo profile..."
curl -X POST https://api.cope.capital/v1/account/sync-fomo \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"fomo_handle": "your_handle"}'

echo "2. Getting your follows..."
FOLLOWS=$(curl -s https://api.cope.capital/v1/account/follows \
  -H "Authorization: Bearer $API_KEY")

echo "Found $(echo $FOLLOWS | jq -r '.total') follows"

echo "3. Creating watchlist from top 10 active follows..."
TOP_HANDLES=$(echo $FOLLOWS | jq -r '.follows[] | select(.active == true) | .handle' | head -10 | jq -R . | jq -s .)

curl -X POST https://api.cope.capital/v1/watchlists \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"name\": \"Top Follows\", \"handles\": $TOP_HANDLES}"
```

## Benefits of syncing

- **Automatic tracking**: All your follows are added to Cope Capital's tracking system
- **Address resolution**: Wallet addresses are automatically resolved from Fomo profiles
- **Bulk operations**: Create watchlists with multiple follows at once
- **Stay updated**: Re-sync anytime to catch new follows or changes

## Field explanations

- `active`: Whether this follow is still active on your Fomo profile
- `solana_address` / `base_address`: Resolved wallet addresses (may be null)
- `synced_at`: When your follows were last synced (Unix timestamp)

## What happens during sync

1. Cope Capital fetches your complete follows list from Fomo
2. All followed wallets are added to the tracking system (if not already tracked)
3. Wallet addresses are resolved from Fomo profiles
4. Your follows are stored for easy watchlist creation

## Notes

- Syncing is **FREE** and doesn't count toward daily limits
- Browsing follows is **FREE**
- Unknown follows are automatically added to the tracking system
- You can sync multiple times - it updates your stored follows
- Only you can access your synced follows list
- Wallet addresses may be null if traders haven't linked wallets to Fomo profiles