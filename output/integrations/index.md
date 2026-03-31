# Integrations

## Overview
3 integrations: one inbound HTTP (SDKMAN! CLI), one bidirectional MongoDB (read versions/application, write audit), and one outbound redirect to GitHub Releases. No message queues, no events, no webhooks.

## Integration Catalogue

| System | Direction | Method | Purpose | Criticality |
|---|---|---|---|---|
| SDKMAN! CLI | Inbound | HTTP GET | Sends download and version lookup requests | High — primary consumer |
| MongoDB | Both | MongoDB driver (sync) | Read version catalogue and CLI config; write audit entries | High — all download resolution depends on it |
| GitHub Releases | Outbound | HTTP 302 redirect | Hosts the actual SDKMAN! bash and native CLI binaries | High — redirect target for CLI downloads |
| Third-party CDNs / hosts | Outbound | HTTP 302 redirect | Hosts SDK candidate binaries (e.g. Java, Groovy) | Medium — URL stored in MongoDB; broker does not validate |

## APIs

**Inbound — served by this service:**

| Endpoint | Auth | Contract |
|---|---|---|
| `GET /download/:candidate/:version/:platform` | None | Implicit; defined by SDKMAN! CLI expectations |
| `GET /download/sdkman/:command/:version/:platform` | None | Implicit |
| `GET /download/native/:command/:version/:platform` | None | Implicit |
| `GET /version/sdkman/:impl/:channel` | None | Implicit |
| `GET /health/alive` | None | Ratpack HealthCheck standard |

**Outbound — no direct API calls made.** All outbound interactions are HTTP 302 redirects; the broker does not make HTTP requests itself. The client (SDKMAN! CLI) follows the redirect to GitHub Releases or the CDN.

## Events / Messaging
None. The service uses no message queues, event buses, or pub/sub mechanisms.

## Error Handling

| Integration | Retry Strategy | Failure Behaviour |
|---|---|---|
| MongoDB (version reads) | None | Handler returns `404` to the client if no version is found; no retry |
| MongoDB (audit writes) | None | `Blocking.exec` with no error handler — failure is silent and invisible |
| GitHub Releases redirect | N/A | Broker issues the redirect; if GitHub is down the CLI follows and fails independently |

## Dependencies
- **Upstream (this service depends on):** MongoDB (must be running and seeded for downloads to work), GitHub Releases (must be available for the CLI to complete downloads after redirect)
- **Downstream (depends on this service):** SDKMAN! CLI — all SDK installations and CLI self-updates route through this broker

## Failure Modes

| Integration | What happens if it fails |
|---|---|
| MongoDB unavailable at startup | Service may start but `GET /health/alive` returns `503`; download requests that require DB lookup return `500` |
| MongoDB unavailable during request | `VersionRepo.fetch` throws; Ratpack returns `500`. `AuditRepo.insertAudit` fails silently with no client impact. |
| GitHub Releases unavailable | Broker issues `302` successfully; CLI follows the redirect and fails independently — broker reports no error |
| `application` collection not seeded | Health check returns `503`; CLI version endpoints return empty response |

## Known Issues
- Audit writes to MongoDB use `Blocking.exec` with no error handler. MongoDB write failures are silently swallowed — there is no alerting, retry, or dead-letter mechanism.
- No circuit breaker on MongoDB reads. If MongoDB becomes slow, Ratpack blocking threads will back up.
- Binary URLs stored in the `versions` collection are not validated by the broker — a malformed or unreachable URL will result in a `302` redirect to a broken destination with no error returned to the client.
