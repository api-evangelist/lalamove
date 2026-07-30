---
name: lalamove-track-delivery
description: >-
  Track, amend, escalate or cancel an in-flight Lalamove delivery — read order
  and driver status, edit drop-offs before pickup, add a priority fee to speed
  matching, change the driver, or cancel. Also covers configuring and consuming
  the order lifecycle webhooks.
api: lalamove:lalamove-delivery-api
base_url: https://rest.lalamove.com/v3
sandbox_url: https://rest.sandbox.lalamove.com/v3
operations:
  - GET /v3/orders/{orderId}
  - PATCH /v3/orders/{orderId}
  - DELETE /v3/orders/{orderId}
  - GET /v3/orders/{orderId}/drivers/{driverId}
  - DELETE /v3/orders/{orderId}/drivers/{driverId}
  - POST /v3/orders/{orderId}/priority-fee
  - PATCH /v3/webhook
generated: '2026-07-19'
method: generated
source: >-
  https://developers.lalamove.com/ — every operation, header, error code, webhook
  event and status below is taken from the published Lalamove documentation.
  Lalamove publishes no OpenAPI document, so operations are identified by
  method+path rather than operationId.
---

# Track and manage a Lalamove delivery

Assumes an order already exists — see `lalamove-book-delivery`. All requests need the HMAC `Authorization` header, the `Market` header matching the market the order was placed in, and a fresh `Request-ID`. A correct order ID with the wrong `Market` returns HTTP 404 / `ERR_ORDER_NOT_FOUND`.

## Reading status

`GET /v3/orders/{orderId}` returns the order and its status:

| Status | Meaning |
|---|---|
| `ASSIGNING_DRIVER` | Matching in progress, no driver yet |
| `ON_GOING` | Driver accepted, en route to pickup |
| `PICKED_UP` | Package collected |
| `COMPLETED` | Delivered, transaction concluded |
| `CANCELED` | Canceled within the policy window |
| `REJECTED` | Rejected twice, no longer broadcast to drivers |
| `EXPIRED` | Two hours elapsed with no driver acceptance |

Order IDs are 19-digit numeric strings (widened from 12 digits in April 2025) — store them as strings, not integers.

## Prefer webhooks over polling

Configure a callback once with `PATCH /v3/webhook` (or in the Partner Portal), per market. Your receiver **must** return HTTP 200; a non-200 raises `ERR_INVALID_RESPONSE`.

Events delivered: `ORDER_CREATED`, `ORDER_STATUS_CHANGED`, `DRIVER_ASSIGNED`, `ORDER_AMOUNT_CHANGED`, `ORDER_EDITED`, `ORDER_REPLACED`, `WALLET_BALANCE_CHANGED`, `POD_STATUS_CHANGED`, `POP_STATUS_CHANGED`, `DELIVERY_CODE_STATUS_CHANGED`.

Lalamove's public documentation does not describe a webhook signature header, so you cannot cryptographically verify payload authenticity from the docs alone. Treat webhook contents as a **hint to re-read authoritative state** via `GET /v3/orders/{orderId}` rather than as trusted input — especially before taking any financial action. Confirm current verification guidance with partner.support@lalamove.com.

## Getting the driver

`GET /v3/orders/{orderId}/drivers/{driverId}` returns courier details. The `driverId` comes from the order details or the `DRIVER_ASSIGNED` webhook. Driver details are only readable from 1 hour before a scheduled pickup, or after driver arrival — calling earlier will not return a driver.

## Amending an order

`PATCH /v3/orders/{orderId}` edits drop-off locations, and only **before pickup**. Emits `ORDER_EDITED`. Attempting it after `PICKED_UP` will be rejected.

## Speeding up matching

If an order sits in `ASSIGNING_DRIVER`, `POST /v3/orders/{orderId}/priority-fee` adds or increases a tip. Rules:

- The new fee must exceed the previous fee — otherwise `ERR_INVALID_TIPS`.
- It must sit within the market's bounds — `ERR_EXCEED_MIN_TIPS` / `ERR_EXCEED_MAX_TIPS`.
- It raises the order total and emits `ORDER_AMOUNT_CHANGED`.

This spends additional real money. Confirm the amount with the user first.

## Changing the driver

`DELETE /v3/orders/{orderId}/drivers/{driverId}` requests a different courier. Only permitted from **15 minutes after matching until pickup** — outside that window you get `ERR_CHANGE_DRIVER`. The order returns to matching, so expect a delay.

## Canceling

`DELETE /v3/orders/{orderId}` cancels within the cancellation policy window and before pickup. Outside it you get `ERR_CANCELLATION_FORBIDDEN` — at that point the delivery must be handled through partner support, not the API.

## Retry discipline

There is no idempotency key on this API. `POST /v3/orders/{orderId}/priority-fee` in particular is **not** safe to blind-retry — a retry that actually succeeded the first time can stack another fee increase. On timeout, re-read `GET /v3/orders/{orderId}` and compare the total before retrying.

Read endpoints (`GET` order, `GET` driver) are safe to retry: 300/min production, 50/min sandbox. Back off on HTTP 429 / `ERR_RATE_LIMIT_EXCEEDED` using the per-endpoint `RateLimit-Reset-Post{API}` header.

## Reference

- Webhook catalog: `asyncapi/lalamove-delivery-webhooks.yml`
- Errors: `errors/lalamove-problem-types.yml`
- Conventions: `conventions/lalamove-conventions.yml`
- Docs: https://developers.lalamove.com/
