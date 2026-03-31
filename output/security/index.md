# Security

## Overview
The service is intentionally public with no authentication or authorisation on any endpoint. All download URLs are publicly accessible by design. The primary security considerations are dependency vulnerabilities, client IP data storage, and the absence of input validation on path tokens passed to MongoDB.

## Authentication
- **Method:** None
- **Flow:** N/A — all endpoints are unauthenticated and publicly accessible

## Authorization
- **Roles:** None
- **Permissions:** None — any client can call any endpoint

## Sensitive Data

| Type | Where stored | Notes |
|---|---|---|
| Client IP address | MongoDB `audit` collection (`host` field) | Read from `X-Real-IP` header; stored on every download |
| Client user-agent | MongoDB `audit` collection (`agent` field) | Stored on every download |
| MongoDB credentials | Runtime environment variables only | `MONGO_USERNAME`, `MONGO_PASSWORD`; not committed to repo |

> No payment data, passwords, session tokens, or other high-sensitivity PII is handled by this service.

## Data Protection
- **Encryption in transit:** All redirect targets use HTTPS (GitHub Releases URLs use `https://` in `binary.properties` and `native_binary.properties`)
- **Encryption at rest:** Not configured in this codebase — depends on MongoDB deployment configuration
- **Log masking:** No sensitive fields are masked or redacted in logs; client IP and user-agent appear in audit writes but not in application logs

## Audit Logging
- Every download request (candidate, bash CLI, native CLI) writes one record to the MongoDB `audit` collection
- Fields logged: `command`, `candidate`, `version`, `host` (IP), `agent` (user-agent), `platform`, `dist`, `timestamp`
- Health check and version endpoints are not audited

## Vulnerabilities

| Vulnerability | Location | Detail |
|---|---|---|
| No input validation on path tokens | `CandidateDownloadHandler`, `BinaryDownloadHandler`, `NativeBinaryDownloadHandler` | `candidate`, `version`, and `platform` values from the URL are passed directly to MongoDB queries and redirect URL construction with no allowlist or format validation |
| Outdated MongoDB driver | `build.gradle` | `mongo-java-driver` 3.2.2 may carry known CVEs; far below the current 5.x release |
| Duplicate/outdated SLF4J | `build.gradle` | `slf4j-simple` 1.7.5 present alongside 2.0.17; older binding may have known issues |
| NPE information disclosure | `BinaryDownloadConfig`, `NativeBinaryDownloadConfig` | Missing config properties cause unhandled `NullPointerException` — stack traces may be returned to clients depending on Ratpack error handler config |
| Deprecated `cucumber-groovy` | `build.gradle` | `cucumber-groovy` 6.10.4 is unmaintained; transitive dependency vulnerabilities not actively patched |

> ⚠️ Unclear: MongoDB injection via the driver is not possible in the traditional sense (no string query concatenation), but unvalidated `candidate` and `platform` values could cause unexpected query behaviour if they contain special characters interpreted by the MongoDB BSON filter builders.

## Dependencies Risk

| Library | Version | Risk |
|---|---|---|
| `mongo-java-driver` | 3.2.2 | High — significantly outdated; check NVD for known CVEs |
| `slf4j-simple` | 1.7.5 | Low — older binding present as duplicate alongside 2.0.17 |
| `cucumber-groovy` | 6.10.4 | Low — test dependency only; deprecated upstream |

## Compliance
- **GDPR:** Client IP addresses are stored in the `audit` collection on every download. If the service operates in or serves EU users, this constitutes personal data storage. No data retention policy, deletion mechanism, or consent flow is present in the codebase.
- **HIPAA / PCI:** Not applicable — no health or payment data handled.

## Recommendations
1. Add allowlist validation for `candidate` and `platform` path tokens against known-good values before passing to MongoDB
2. Upgrade `mongo-java-driver` to 5.x — this is the highest-priority security dependency update
3. Define and implement a retention/deletion policy for the `audit` collection to address GDPR compliance for stored IP addresses
4. Configure Ratpack's error handler to suppress stack traces in production error responses
