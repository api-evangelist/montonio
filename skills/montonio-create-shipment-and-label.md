---
name: Create a shipment and print a label
description: Fetch shipping methods, create a Shipment, generate a label file, and fetch the printable label.
api: Montonio Shipping API (v2)
base_url: https://shipping.montonio.com/api/v2
sandbox_url: https://sandbox-shipping.montonio.com/api/v2
operations:
  - "GET /shipping-methods"
  - "POST /shipments"
  - "POST /label-files"
  - "GET /label-files/{labelFileId}"
---

# Create a shipment and print a label

## Auth
All Shipping v2 endpoints require an HS256 JWT in the `Authorization` header as a **Bearer** token, signed with the store Secret Key and carrying `accessKey`.

## Steps
1. **Fetch shipping methods** — `GET /shipping-methods` for the store; for pickup-point carriers also `GET /shipping-methods/pickup-points`. Optionally `POST /shipping-methods/rates` to price parcels.
2. **Create the Shipment** — `POST /shipments` with the chosen `shippingMethod`, parcels, and (optionally) products. Save the `shipmentId`.
3. **Generate a label** — `POST /label-files` for the shipment. Labels may be produced **asynchronously**: if not returned synchronously, wait for the `label_file.ready` webhook.
4. **Fetch the label** — `GET /label-files/{labelFileId}` and print/store the file.

## Rules
- Register webhooks with `POST /webhooks` (a single URL, branch on event type) and delete with `DELETE /webhooks/{webHookId}`.
- In **sandbox** (`sandbox-shipping.montonio.com/api/v2`), carrier calls are mocked, labels are dummy, and phone/address validation is skipped; activate carriers with dummy credentials and enable test mode in the Partner System first.
- Errors: `400` validation, `401` bad JWT, `404` not found.
- Carriers include Omniva, DPD, Venipak, SmartPosti.
