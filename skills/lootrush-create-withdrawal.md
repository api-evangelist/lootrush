---
name: Create and track a LootRush crypto withdrawal
description: Initiate a cryptocurrency withdrawal for a user and poll its status to completion.
api: openapi/lootrush-openapi-original.json
operations: [createWithdrawal, listWithdrawals]
---

# Create and track a LootRush crypto withdrawal

Use the LootRush Withdraw API to move crypto out on a user's behalf and follow the
transaction to settlement.

## Auth
- Bearer token: `Authorization: Bearer <token>` (per-user API key from your LootRush account manager).
- Base URL: `https://third-party.lootrush.com`.

## Steps
1. **Create the withdrawal** — `createWithdrawal` (`POST /api/crypto/{userId}/withdraw`).
   Supply the recipient wallet, amount, and currency. Optionally pass an `externalId`
   you generate so you can correlate/deduplicate on your side (there is no server-side
   idempotency key). The entry is created **queued** and processed asynchronously.
2. **Poll status** — `listWithdrawals` (`GET /api/crypto/{userId}/withdraws`). Filter by
   `externalId`, `bulkId`, `transactionHash`, or `status` and page with `page` (1-indexed)
   + `perPage`. Watch `status`, `transferTokenStatus`, and `transactionHash` (populated once
   on-chain). `errorMessage` is set if the withdrawal fails.

## Rules
- **403** = recipient wallet is blocked from receiving tokens; **404** = user/recipient/wallet
  not resolved. Surface these to the operator, do not retry blindly.
- Errors come back as `{ "error": "<message>" }` — not RFC 9457.
- This is a value-moving, on-chain action: confirm intent before calling `createWithdrawal`.
