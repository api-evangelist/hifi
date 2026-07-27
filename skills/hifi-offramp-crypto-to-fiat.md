---
name: Create a crypto-to-fiat off-ramp payout
description: Attach a payout bank account to a KYC-approved user, then quote and execute a crypto-to-fiat off-ramp with Hifi.
api: openapi/hifi-openapi-original.json
operations:
- POST /v2/users/{userId}/accounts
- GET /v2/offramps/rates
- POST /v2/offramps
- POST /v2/offramps/{transferId}/quote/accept
- GET /v2/offramps/{transferId}
---

# Create a crypto-to-fiat off-ramp payout

Convert a user's crypto/stablecoin balance to fiat and pay it out to a bank
account.

## Auth
`Authorization: Bearer YOUR_API_KEY` over HTTPS. Sandbox:
`https://sandbox.hifibridge.com`; production: `https://production.hifibridge.com`.

## Preconditions
The user must already exist and be KYC-approved (see the on-ramp skill). EUR
payouts are supported across 41 European countries via SEPA / SEPA Instant /
SWIFT.

## Steps
1. **Add a payout account** — `POST /v2/users/{userId}/accounts` with the bank
   account details. Invalid data returns `INVALID_ACCOUNT_DATA` (500001).
2. **Quote** — `GET /v2/offramps/rates` for the token → fiat currency + rail.
3. **Create the off-ramp** — `POST /v2/offramps` with amount, the destination
   `accountId`, the route, and a `requestId` (uuid v4). A bad account/route pair
   returns `INVALID_ACCOUNT_FOR_TRANSACTION` (400002).
4. **Accept the quote** — `POST /v2/offramps/{transferId}/quote/accept` before
   expiry (`INVALID_QUOTE` 400006 otherwise).
5. **Track** — `GET /v2/offramps/{transferId}` or Offramp webhook events.

## Rules
- If transfer approvals are enabled, the payout may enter a pending-approval
  state — see the transfer-approvals surface
  (`GET /v2/transfer-approvals`, `POST /v2/transfer-approvals/{approvalId}/approve`).
- `ACCOUNT_NOT_FOUND` (500301) / `INACTIVE_ACCOUNT` (500501) indicate a bad or
  disabled payout account.
- Set a unique `requestId` per payout for idempotency.
