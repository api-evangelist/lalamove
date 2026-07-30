# Lalamove

Lalamove is a Hong Kong-founded on-demand logistics and same-day delivery platform operating across Asia and Latin America, matching businesses and consumers with a fleet of motorcycle, car, van and truck drivers.

Its **Delivery API (v3)** lets partners request quotations, place and edit delivery orders, track assigned drivers, add priority fees, and receive order lifecycle webhooks across eleven markets.

- Developer portal: https://developers.lalamove.com/
- Production base URL: `https://rest.lalamove.com/v3`
- Sandbox base URL: `https://rest.sandbox.lalamove.com/v3`
- Partner Portal (credentials, wallet, webhooks): https://partnerportal.lalamove.com
- Status: https://status.lalamove.com
- GitHub: https://github.com/lalamove

Backed by: hongshan — https://lalamove.com

## API at a glance

| | |
|---|---|
| Auth | HMAC SHA256 request signing (`Authorization: hmac {KEY}:{TIMESTAMP}:{SIGNATURE}`) |
| Required headers | `Authorization`, `Market` (UN/LOCODE), `Request-ID`, `Content-Type` |
| Markets | HK, SG, MY, TH, PH, ID, VN, TW, JP, MX, BR |
| Envelope | Top-level `data` object on requests and responses |
| Errors | Custom `{"errors":[{"id","message","detail"}]}` array — not RFC 9457 |
| Versioning | URI path major version (`/v3`), month-dated changelog |
| Idempotency | **Not supported** — no idempotency key documented |
| Pagination | Not documented |
| Spec | No published OpenAPI or AsyncAPI |
| Events | 10 webhook events, partner-configured URL |

## Artifacts in this repo

| Artifact | Path | Method |
|---|---|---|
| Authentication | `authentication/lalamove-authentication.yml` | searched |
| Conventions | `conventions/lalamove-conventions.yml` | searched |
| Error catalog | `errors/lalamove-problem-types.yml` | searched |
| Webhooks | `asyncapi/lalamove-delivery-webhooks.yml` | searched |
| Lifecycle | `lifecycle/lalamove-lifecycle.yml` | searched |
| Changelog | `changelog/lalamove-changelog.yml` | searched |
| Sandbox | `sandbox/lalamove-sandbox.yml` | searched |
| Packages / SDKs | `packages/lalamove-packages.yml` | searched |
| Conformance | `conformance/lalamove-conformance.yml` | derived |
| Data model | `data-model/lalamove-data-model.yml` | derived |
| MCP (candidate) | `mcp/lalamove-mcp.yml` | derived |
| Well-known probe | `well-known/lalamove-well-known.yml` | searched (none found) |
| Domain security | `security/lalamove-domain-security.yml` | probed |
| llms.txt | `llms/lalamove-llms.txt` | generated |
| Agent Skills | `skills/` | generated |

## Notable findings

- **No idempotency key.** The required `Request-ID` header is documented as a support/tracing nonce, not a deduplication key. Blind-retrying `POST /v3/orders` can dispatch a duplicate courier and double-charge the prepaid wallet.
- **No published OpenAPI or AsyncAPI**, despite a well-documented endpoint surface and a ten-event webhook catalog.
- **No deprecation policy.** The April 2025 order ID widening from 12 to 19 digits shipped as a dated changelog line with no deprecation window.
- **No webhook signature** is documented, so payloads cannot be cryptographically verified from the public docs.
- **No security.txt, VDP or trust center** found on any Lalamove host. Note that `partnerportal.lalamove.com` returns HTTP 200 with an SPA HTML shell for every path — those are soft 404s, not documents.
