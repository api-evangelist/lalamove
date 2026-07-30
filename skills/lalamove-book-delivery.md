---
name: lalamove-book-delivery
description: >-
  Quote and book an on-demand courier delivery with the Lalamove Delivery API v3
  — price a route of stops, then commit that quotation into a dispatched order.
  Use when a user wants to send a package point-to-point in a Lalamove market.
api: lalamove:lalamove-delivery-api
base_url: https://rest.lalamove.com/v3
sandbox_url: https://rest.sandbox.lalamove.com/v3
operations:
  - POST /v3/cities
  - POST /v3/quotations
  - GET /v3/quotations/{quotationId}
  - POST /v3/orders
  - GET /v3/orders/{orderId}
generated: '2026-07-19'
method: generated
source: >-
  https://developers.lalamove.com/ — every operation, header, error code and
  status below is taken from the published Lalamove documentation. Lalamove
  publishes no OpenAPI document, so operations are identified by method+path
  rather than operationId.
---

# Book a Lalamove delivery

The Lalamove flow is two committed steps: you **quote** a route, then you **place an order against that quotation**. You cannot place an order without a valid, unexpired `quotationId`.

## Before you call anything

Every request needs three headers beyond `Content-Type: application/json`:

- `Authorization: hmac {KEY}:{TIMESTAMP}:{SIGNATURE}`
- `Market: {UN/LOCODE}` — one of HK, SG, MY, TH, PH, ID, VN, TW, JP, MX, BR
- `Request-ID: {uuid}` — a fresh nonce per request

Build the signature as `HMAC-SHA256({TIMESTAMP}\r\n{METHOD}\r\n{PATH}\r\n\r\n{BODY}, API_SECRET)` rendered as lowercase hex. Omit the body segment for GET. The timestamp in the header must be the same value you signed, or you get HTTP 401. See `authentication/lalamove-authentication.yml`.

Use `pk_test`/`sk_test` against `rest.sandbox.lalamove.com` and `pk_prod`/`sk_prod` against `rest.lalamove.com`. Never mix them.

## Step 1 — Confirm what the market supports

Call `GET /v3/cities` for the target market. This returns the valid service types (vehicle classes) and special requests. Skipping this is the most common cause of `ERR_INVALID_SERVICE_TYPE` and `ERR_INVALID_SPECIAL_REQUEST`, because vehicle classes differ per market.

## Step 2 — Quote the route

Call `POST /v3/quotations` with the payload wrapped in a top-level `data` object.

Constraints that will reject the request:

- At least 2 stops, or you get `ERR_INSUFFICIENT_STOPS`.
- At most 1 pickup plus 15 drop-offs, or `ERR_TOO_MANY_STOPS`.
- `scheduleAt` must be in the future, or `ERR_INVALID_SCHEDULE_TIME`.
- Every stop must be inside the service area, or `ERR_OUT_OF_SERVICE_AREA`.
- If address geocoding fails you get `ERR_REVERSE_GEOCODE_FAILURE` — resend the stop with explicit lat/lng coordinates.
- Phone numbers are validated per market: `ERR_INVALID_PHONE_NUMBER`.

The response carries a `quotationId` and the price breakdown. Note that price breakdown shape varies by market — Malaysia uses precision length 1, and the Philippines carries an extra breakdown item.

## Step 3 — Place the order

Call `POST /v3/orders` referencing the `quotationId`.

**Quotations expire.** If you get `ERR_INVALID_QUOTATION_ID`, do not retry the order — go back to Step 2 and re-quote, then place against the fresh ID.

If you get HTTP 402 / `ERR_INSUFFICIENT_CREDIT`, the market wallet needs a top-up in the Partner Portal. This is not retryable from the API.

### Retry discipline — read this before you write retry logic

The Lalamove API **does not support idempotency keys**. The `Request-ID` header is a support/tracing nonce, not a deduplication key. If `POST /v3/orders` times out or returns 5xx, retrying may dispatch a **second real courier and charge the wallet twice**.

On a timeout or 5xx from `POST /v3/orders`:

1. Do not retry immediately.
2. Poll `GET /v3/orders/{orderId}` if you captured an ID, or reconcile against your own records.
3. Only re-place the order once you have confirmed no order was created.

## Step 4 — Confirm dispatch

Call `GET /v3/orders/{orderId}`. A freshly placed order starts in `ASSIGNING_DRIVER`. Do not poll aggressively — read limits are 300/min in production, 50/min in sandbox, signaled per-endpoint via `RateLimit-Limit-Post{API}`, `RateLimit-Remaining-Post{API}` and `RateLimit-Reset-Post{API}`. Prefer the `ORDER_STATUS_CHANGED` webhook over polling; see `lalamove-track-delivery`.

## Human confirmation

`POST /v3/orders` spends real money from a prepaid wallet and dispatches a physical courier. Confirm the route, service type and quoted price with the user before calling it. Do not place orders autonomously.

## Reference

- Errors: `errors/lalamove-problem-types.yml`
- Conventions (envelope, rate limits, market scoping): `conventions/lalamove-conventions.yml`
- Entity graph: `data-model/lalamove-data-model.yml`
- Docs: https://developers.lalamove.com/
