---
name: Orderful convert and send an EDI document
description: Convert JSON to X12 (or X12 to JSON) and submit an EDI transaction to a trading partner, or push a raw X12/XML/CSV document through Orderful.
api: openapi/orderful-openapi-original.yml
operations:
- ConversionController_convertData
- TransactionControllerV3_create
- TransactionControllerV3_createRaw
- TransactionControllerV3_getMessage
- RelationshipController_listV3
---

# Orderful convert and send an EDI document

Use this skill to translate between JSON and X12 and to submit a document to a partner.

## Auth & base URL
- Header `orderful-api-key: <token>`; region base URL `https://api.orderful.com` (US) or `https://api-eu.orderful.com` (EU).

## Steps
1. **(Optional) Convert formats.** `ConversionController_convertData` (`POST /v3/convert`) converts JSON↔X12. Set `Content-Type` to what you send and `Accept` to what you want back.
2. **Confirm the partnership exists.** `RelationshipController_listV3` (`GET /v3/relationships`) to resolve the trading partner relationship you will route to.
3. **Send a structured transaction.** `TransactionControllerV3_create` (`POST /v3/transactions`) with a Mosaic or OrderfulJSON body (exactly one of `message` or `summary`).
4. **Or send a raw document.** `TransactionControllerV3_createRaw` (`POST /v3/transactions/raw`) with the raw X12/XML/CSV body; routing metadata goes in **headers**, not the body. Use an HTTP client (not the docs "Try It!") for non-JSON content types.
5. **Verify the wire content.** `TransactionControllerV3_getMessage` (`GET /v3/transactions/{transactionId}/message`) returns the generated message.

## Rules
- Raw transactions carry routing in headers; structured transactions carry it in the JSON body.
- No idempotency key — check `TransactionControllerV3_list` before retrying a failed create.
- X12 passthrough bypasses Orderful's validation engine; prefer Mosaic/OrderfulJSON to get validation and partner-agnostic mapping.
