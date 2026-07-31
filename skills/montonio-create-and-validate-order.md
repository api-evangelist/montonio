---
name: Create and validate a Montonio Order
description: Collect a payment by listing payment methods, creating an Order, redirecting the customer, and validating the result via webhook.
api: Montonio Payments API (Stargate)
base_url: https://stargate.montonio.com/api
sandbox_url: https://sandbox-stargate.montonio.com/api
operations:
  - "GET /stores/payment-methods"
  - "POST /orders"
  - "GET /orders/:orderUuid"
---

# Create and validate a Montonio Order

Use this skill to collect a payment through Montonio Stargate.

## Auth
All requests authenticate with an HS256 JWT signed with the store **Secret Key** and carrying the store **Access Key**. For `GET` endpoints, send the JWT in the `Authorization` header. For `POST` endpoints, the request body **is** the signed JWT (the token payload contains the request data). Set `exp` ~1 hour ahead. Sandbox and production use separate keys.

## Steps
1. **List payment methods** — `GET /stores/payment-methods` to get the store's enabled methods (`paymentInitiation`, `cardPayments`, `mobilePay`, `blik`, `bnpl`, `hirePurchase`). Present them to the customer.
2. **Create the Order** — build a JWT payload with `accessKey`, `merchantReference`, `returnUrl`, `notificationUrl`, `currency`, `grandTotal`, line items, and a `payment` object (method + amount). Sign it and `POST /orders`. Save the returned Order UUID.
3. **Redirect** the customer to the `paymentUrl` returned by the API.
4. **Validate via webhook** (primary): set `notificationUrl`; on the incoming POST, verify the `orderToken` JWT signature with your Secret Key and read `paymentStatus`. The return-URL `orderToken` query param is a secondary synchronous signal.
5. **Optional double-check** — `GET /orders/:orderUuid` to re-read `paymentStatus`.

## Rules
- `paymentStatus` values: `PENDING` -> `PAID` (or `ABANDONED`); `PAID` can rarely become `VOIDED` for `paymentInitiation`. Only treat `PAID` as success.
- Always verify webhook JWTs; webhooks originate from IPs `35.156.245.42` / `35.156.159.169`, User-Agent `MontonioWebhooks/1.0`.
- Test with card `5577 0000 5577 0004` (exp `03/30`, CVC `737`) or BLIK code `777 123` in sandbox.
- Errors: `400` validation, `401` bad JWT, `404` not found (see errors/montonio-problem-types.yml).
