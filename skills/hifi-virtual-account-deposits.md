---
name: Provision a virtual account and settle deposits
description: Create a virtual account for a user, attach a settlement rule that routes incoming funds, and simulate a deposit in sandbox.
api: openapi/hifi-openapi-original.json
operations:
- POST /v2/users/{userId}/virtual-accounts
- POST /v2/virtual-accounts/settlement-rules
- POST /v2/users/{userId}/virtual-accounts/{accountId}/settlement-rules/{ruleId}
- POST /v2/users/{userId}/virtual-accounts/{accountId}/simulate-deposit
- GET /v2/users/{userId}/virtual-accounts/{accountId}
---

# Provision a virtual account and settle deposits

Give a KYC-approved user a virtual account to receive fiat, and auto-route what
lands in it (e.g. convert USD deposits to a stablecoin wallet).

## Auth
`Authorization: Bearer YOUR_API_KEY` over HTTPS. Sandbox host:
`https://sandbox.hifibridge.com`.

## Steps
1. **Create the virtual account** — `POST /v2/users/{userId}/virtual-accounts`.
   USD virtual accounts (incl. for Nigerian businesses) receive bank deposits.
2. **Create a settlement rule** — `POST /v2/virtual-accounts/settlement-rules`
   describing how incoming funds should convert/route.
3. **Apply the rule** —
   `POST /v2/users/{userId}/virtual-accounts/{accountId}/settlement-rules/{ruleId}`.
4. **Simulate a deposit (sandbox only)** —
   `POST /v2/users/{userId}/virtual-accounts/{accountId}/simulate-deposit` to
   trigger the settlement flow without moving real money.
5. **Inspect** — `GET /v2/users/{userId}/virtual-accounts/{accountId}` and watch
   Account webhook events for deposit + settlement transitions.

## Rules
- Use orchestration addresses / settlement rules rather than polling to route
  funds automatically.
- Deactivate/reactivate with the `/deactivate` and `/reactivate` sub-resources;
  remove a rule with `DELETE .../settlement-rules`.
- The error envelope is `{ code, error, errorDetails }`.
