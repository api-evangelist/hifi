---
name: Create a fiat-to-crypto on-ramp
description: Onboard a user, ensure KYC is approved, then quote and execute a fiat-to-crypto on-ramp with Hifi.
api: openapi/hifi-openapi-original.json
operations:
- POST /v2/users
- POST /v2/users/{userId}/kyc
- GET /v2/users/{userId}/kyc/status
- GET /v2/onramps/rates
- POST /v2/onramps
- POST /v2/onramps/{transferId}/quote/accept
- GET /v2/onramps/{transferId}
---

# Create a fiat-to-crypto on-ramp

Use the Hifi v2 API to move fiat into a stablecoin/crypto position for a user.

## Auth
Send `Authorization: Bearer YOUR_API_KEY` on every request over HTTPS. Use the
sandbox host `https://sandbox.hifibridge.com` while testing and
`https://production.hifibridge.com` for live.

## Steps
1. **Create the user** — `POST /v2/users`. Pass a `requestId` (uuid v4) to make
   creation idempotent; reusing it returns the existing user rather than
   duplicating.
2. **Submit KYC** — `POST /v2/users/{userId}/kyc` with the user's compliance
   data. In sandbox this is typically auto-approved within minutes.
3. **Confirm approval** — poll `GET /v2/users/{userId}/kyc/status` until status
   is approved. Do not attempt a transfer while KYC is pending or you will get a
   `USER_COMPLIANCE_VERIFICATION_FAILED` (300201).
4. **Quote the rate** — `GET /v2/onramps/rates` for the source currency / rail /
   destination token combination.
5. **Create the on-ramp** — `POST /v2/onramps` with the amount, route, and a
   fresh `requestId`. An invalid route returns `INVALID_TRANSACTION_ROUTE`
   (400001).
6. **Accept the quote** — `POST /v2/onramps/{transferId}/quote/accept` before it
   expires; an expired quote returns `INVALID_QUOTE` (400006).
7. **Track status** — `GET /v2/onramps/{transferId}`, or subscribe to Onramp
   webhook events (RS256-JWT signed) for status transitions.

## Rules
- Errors use the envelope `{ code, error, errorDetails }` — branch on `error`.
- `INSUFFICIENT_BALANCE` (400401) / `INSUFFICIENT_CREDIT_BALANCE` (400402) mean
  the source or fee balance is short.
- Always set a unique `requestId` per intended transaction; a duplicate returns
  `RESOURCE_CONFLICT` (100005) / `TRANSACTION_REQUEST_ALREADY_EXISTS` (400003).
