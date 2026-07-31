---
name: Orderful Order-to-Cash (seller)
description: Receive a Purchase Order (850), acknowledge it, and send the acknowledgment, ship notice, and invoice back to a retail trading partner over Orderful.
api: openapi/orderful-openapi-original.yml
operations:
- PollingController_list
- TransactionControllerV3_getById
- DeliveryController_handleDeliveryApproved
- TransactionControllerV3_acknowledgeTransaction
- TransactionControllerV3_create
---

# Orderful Order-to-Cash (seller)

Use this skill to run the most common EDI supplier flow: PO in → POA, ASN, and Invoice out.

## Auth & base URL
- Send `orderful-api-key: <token>` and `Content-Type: application/json` on every request.
- Use your region's base URL: US `https://api.orderful.com`, EU `https://api-eu.orderful.com`. Keys are region-scoped.
- While testing, set the transaction `stream` to `test` so nothing reaches real partners (see `sandbox/orderful-sandbox.yml`).

## Steps
1. **Receive the Purchase Order (850).** Poll your inbox/bucket with `PollingController_list` (`GET /v3/polling-buckets/{bucketId}`), or receive it pushed to your Inbound HTTP webhook (`asyncapi/orderful-webhooks.yml`). Fetch full content with `TransactionControllerV3_getById` (`GET /v3/transactions/{transactionId}`).
2. **Confirm delivery.** Approve the delivery with `DeliveryController_handleDeliveryApproved` (`POST /v3/deliveries/{deliveryId}/approve`) once you have durably stored the PO. This removes it from the bucket and marks it DELIVERED.
3. **Acknowledge (997).** Call `TransactionControllerV3_acknowledgeTransaction` (`POST /v3/transactions/{transactionId}/acknowledgment`) with the accepted/rejected status.
4. **Send the Purchase Order Acknowledgment (855).** `POST /v3/transactions` (`TransactionControllerV3_create`) with the 855 Mosaic schema.
5. **Send the Advance Ship Notice (856)** and then the **Invoice (810)** the same way via `TransactionControllerV3_create`, referencing the original PO number.

## Rules
- Provide exactly one of `message` or `summary` on create — supplying both returns `400 Invalid request body` (`errors/orderful-problem-types.yml`).
- There is **no idempotency key**; do not blindly retry a create on timeout — first list/search to see if the transaction already landed.
- Pagination is offset-based (`limit`/`offset`, max 100), with counts in the `meta` object.
