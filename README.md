# Cope Capital API Skills

Agent Skills package for the Cope Capital API - programmatic access to smart money wallet intelligence from 1,000+ tracked Fomo wallets across Solana and Base chains. Build custom watchlists, get real-time trading alerts, and access enriched token data to power your trading bots and analytics.

## Installation

```bash
npx skills add cope-capital/cope-api-skills
```

## Available Skills

| Skill | Description | Use Case |
|-------|-------------|----------|
| **cope-register** | Register for API key and get started | First-time setup, onboarding |
| **cope-watchlists** | Manage wallet watchlists by Fomo handle | Organize tracking, add/remove traders |
| **cope-activity** | Monitor real-time trading activity | Trading alerts, bot triggers |
| **cope-leaderboard** | Get top performing wallets by PnL | Discover profitable traders |
| **cope-fomo-sync** | Link Fomo profile and auto-track follows | Bulk import, streamlined setup |
| **cope-x402-setup** | Enable USDC payments for unlimited access | Scale beyond free limits |

## Quick Start

1. **Register** for an API key:
   ```bash
   # Use the cope-register skill
   curl -X POST https://api.cope.capital/v1/register \
     -H "Content-Type: application/json" \
     -d '{"agent_name": "my-bot", "platform": "openclaw"}'
   ```

2. **Sync your Fomo follows** (optional but recommended):
   ```bash
   # Use the cope-fomo-sync skill  
   curl -X POST https://api.cope.capital/v1/account/sync-fomo \
     -H "Authorization: Bearer cope_<your_key>" \
     -d '{"fomo_handle": "your_handle"}'
   ```

3. **Create a watchlist**:
   ```bash
   # Use the cope-watchlists skill
   curl -X POST https://api.cope.capital/v1/watchlists \
     -H "Authorization: Bearer cope_<your_key>" \
     -d '{"name": "Smart Money", "handles": ["icemandot", "frankdegods"]}'
   ```

4. **Monitor for activity**:
   ```bash
   # Use the cope-activity skill
   curl "https://api.cope.capital/v1/activity/poll" \
     -H "Authorization: Bearer cope_<your_key>"
   ```

## API Overview

**Base URL**: `https://api.cope.capital`

**Authentication**: All endpoints require `Authorization: Bearer cope_<api_key>`

**Free Tier**: 250 API calls per day, 1 watchlist, 10 handles per watchlist

**Upgrade Options**: 
- x402 payments ($0.005/call USDC on Base/Solana) 
- Whop subscription ($5/month)

## Key Features

- **1,000+ Tracked Wallets**: Access to verified smart money traders from Fomo
- **Real-time Activity**: Sub-minute latency on trade detection
- **Multi-chain Support**: Solana and Base networks
- **Efficient Polling**: Free polling endpoint + paid enrichment pattern
- **Auto-resolution**: Fomo handles automatically resolved to wallet addresses
- **Pay-per-use**: x402 micropayments for unlimited access

## Support

- **Documentation**: https://cope.capital
- **API Reference**: https://api.cope.capital/docs
- **Issues**: Open an issue in this repository

## License

Proprietary - See https://cope.capital/terms