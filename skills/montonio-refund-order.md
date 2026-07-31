---
name: Refund a Montonio Order idempotently
description: Issue a full or partial refund against a paid Order and track the refund status via webhook.
api: Montonio Payments API (Stargate)
base_url: https://stargate.montonio.com/api
operations:
  - "POST /refunds"
---

# Refund a Montonio Order idempotently

## Auth
`POST /refunds` authenticates like all Stargate POST endpoints: the request body is an HS256 JWT signed with the store Secret Key, carrying `accessKey` and `exp`.

## Steps
1. **Generate an idempotency key** — a V4 UUID. Store it against the refund in your database so retries reuse it.
2. **Build and sign the token** with `accessKey`, the target Order UUID, the refund `amount`, and `idempotencyKey`.
3. **`POST /refunds`** with the token as the body.
4. **Track status via webhook** — Montonio POSTs a `refundToken` JWT; verify it with your Secret Key and read `status`.

## Rules
- **Idempotency is required.** Reusing a key for the same order returns `400` ("already has a refund with same idempotency key"); this prevents duplicate refunds. A fresh key = a new refund.
- Refund `status`: `PENDING` -> `PROCESSING` -> `SUCCESSFUL`, or `REJECTED` / `CANCELED`. `PENDING` may auto-retry.
- On `REJECTED`/retrying `PENDING`, read `refundStatusDescription` (e.g. `INSUFFICIENT_FUNDS`, `DECLINED`, `EXPIRED_OR_CANCELLED_CARD`, `REFUND_EXCEEDS_ORDER_PAID_AMOUNT`) — see errors/montonio-decline-codes.yml.
- A successful refund moves the Order to `PARTIALLY_REFUNDED` or `REFUNDED`.
