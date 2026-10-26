# Lalamove

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
