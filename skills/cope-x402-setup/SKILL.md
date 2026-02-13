---
name: cope-x402-setup
description: Set up x402 payments to unlock unlimited API access. Configure USDC payments on Base or Solana to bypass daily limits and get higher rate limits.
metadata:
  author: cope-capital
  version: "1.0"
---

# Instructions

Set up x402 pay-per-request payments to unlock unlimited access beyond the 250 daily free calls. Pay with USDC on Base or Solana chains at $0.005 per API call.

## When to use this skill
- User hits the 250 daily call limit
- User needs higher rate limits (300 req/min vs 10 req/min)
- User wants unlimited API access for production usage
- User prefers pay-per-use over monthly subscriptions

## What is x402?

x402 is a HTTP status code for "Payment Required" that enables automated, per-request payments for APIs. Instead of managing API credits or subscriptions, your requests automatically include micropayments.

## Benefits of x402 setup

**Before x402 (Free tier):**
- 250 API calls per day
- 1 watchlist maximum
- 10 handles per watchlist
- 10 requests per minute rate limit

**After x402 (Upgraded):**
- Unlimited API calls (pay $0.005 each after 250 free)
- 10 watchlists maximum
- 100 handles per watchlist
- 300 requests per minute rate limit

## Step 1: Add payment address to your account

Provide a Base or Solana wallet address that can send USDC:

```bash
curl -X PATCH https://api.cope.capital/v1/account \
  -H "Authorization: Bearer cope_<your_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "payment_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18"
  }'
```

### Supported chains and tokens
- **Base USDC**: Native USDC on Base mainnet
- **Solana USDC**: USDC on Solana mainnet

### Response
```json
{
  "agent_name": "my-trading-bot", 
  "x402_enabled": true,
  "payment_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18",
  "updated": true
}
```

## Step 2: Making paid requests

After setup, paid endpoints work automatically:

1. **First 250 calls per day**: Free (no payment required)
2. **After 250 calls**: Include payment proof in request header

When you exceed daily limits, you'll get a 402 response:

```json
{
  "error": {
    "code": "DAILY_LIMIT_REACHED",
    "message": "Daily limit of 250 free calls reached. Include x402 payment header to continue.",
    "details": {
      "daily_limit": 250,
      "x402_enabled": true,
      "price_usd": 0.005,
      "chain": "base",
      "token": "USDC"
    }
  }
}
```

## Step 3: How x402 payments work

When you exceed free limits, the x402 protocol handles payments automatically:

1. **API returns 402** with payment details
2. **Your agent** sends USDC to the specified address  
3. **Your agent** retries the request with payment proof
4. **API validates** the payment and processes your request

Example with payment header:
```bash
curl -X GET https://api.cope.capital/v1/activity \
  -H "Authorization: Bearer cope_<your_api_key>" \
  -H "X-Payment: base:0x1234...txhash"
```

## Pricing breakdown

| Endpoint | Free tier | x402 cost |
|----------|-----------|-----------|
| Registration, account management | Always free | - |
| Watchlist CRUD | Always free | - |
| Fomo sync | Always free | - |
| Activity polling (`/v1/activity/poll`) | Counts toward 250 | $0.005 |
| Full activity (`/v1/activity`) | Counts toward 250 | $0.005 |
| Leaderboard | Counts toward 250 | $0.005 |
| Token lookup | Counts toward 250 | $0.01 |

## View your payment history

```bash
curl -X GET https://api.cope.capital/v1/account/payments \
  -H "Authorization: Bearer cope_<your_api_key>"
```

### Response
```json
{
  "payments": [
    {
      "endpoint": "/v1/activity",
      "amount_usd": 0.005,
      "token_mint": "USDC",
      "tx_hash": "0xabc123...",
      "chain": "base",
      "timestamp": 1707600000
    }
  ],
  "total_usd": 1.25,
  "total_count": 250
}
```

## Remove payment address (disable x402)

Set payment_address to null to disable x402 and return to free tier limits:

```bash
curl -X PATCH https://api.cope.capital/v1/account \
  -H "Authorization: Bearer cope_<your_api_key>" \
  -H "Content-Type: application/json" \
  -d '{
    "payment_address": null
  }'
```

## Example: Check if x402 is enabled

```bash
curl -X GET https://api.cope.capital/v1/account \
  -H "Authorization: Bearer cope_<your_api_key>"
```

Look for `x402_enabled: true` in the response.

## Wallet requirements

Your payment wallet must:
- **Hold USDC** on Base or Solana
- **Have sufficient balance** for anticipated API usage
- **Be accessible** by your agent/script for automated payments

## Notes

- x402 payments are **fully automated** - no manual intervention needed
- Payments are **per-request** - you only pay for what you use
- **Base chain is recommended** for lower transaction fees
- Free tier quota (250 calls) resets daily at midnight UTC
- All payment records are stored and available via `/v1/account/payments`
- x402 setup is **immediate** - upgraded limits apply instantly

## Alternative: Whop subscription

If you prefer monthly billing over pay-per-request, consider the Whop subscription option at https://cope.capital for unlimited access.