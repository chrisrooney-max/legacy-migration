# Change Risk & Technical Debt

## Overview
Overall risk profile is **low-medium**. The service is small and focused with a clear single responsibility. The main risks are concentrated in the dependency layer (outdated MongoDB driver, Scala interop via JitPack) and in missing error handling on audit writes. Handler classes have no unit tests, meaning changes to handler logic carry higher regression risk than the test suite implies.

## High-Risk Areas

| Area | Description | Reason | Impact |
|---|---|---|---|
| `VersionRepo.fetch()` | Scala `JavaConverters` and `Option.apply(null)` interop | Any change to `sdkman-persistent-model` or Scala version could break silently | Download failures for all SDK candidates |
| `BinaryDownloadConfig` / `NativeBinaryDownloadConfig` | All fields `final = null`; populated by Ratpack reflection | Missing or misnamed config property causes NPE at request time with no startup warning | All bash/native CLI downloads broken |
| `AuditRepo.insertAudit()` | `Blocking.exec` with no error handler | MongoDB write failures are invisible; no observability into audit loss | Silent data loss; no alerting |
| `Main.java` routing | Deprecated route still active; handler list must be manually kept in sync with Guice bindings | Risk of route/binding mismatch on refactoring | 404 or injection failure at startup |
| `AppRepo.findVersion()` | Reads first document from `application` with no filter | Non-deterministic if multiple documents exist | Incorrect CLI version returned |

## Known Fragile Components
- `VersionRepo` — Scala interop is the most complex and least idiomatic code in the service; subtle breakage if Scala or Arrow versions change
- `BinaryDownloadConfig` / `NativeBinaryDownloadConfig` — null-field pattern gives no compile-time or startup-time safety for required config

## Coupling Issues
- `binary.RequestDetails` (Java) is used directly by `rust.NativeBinaryDownloadHandler` — cross-package coupling with no abstraction
- All handler bindings and routes are co-located in `Main.java` — acceptable for this service size, but means Main must be updated whenever a handler is added or removed
- `AppRepo` serves both health check and version lookup — two unrelated concerns in one class backed by one collection

## Technical Debt

| Item | Description | Priority |
|---|---|---|
| `mongo-java-driver` 3.2.2 | Significantly outdated; legacy synchronous API; potential CVEs | High |
| No unit tests on handler classes | `CandidateDownloadHandler`, `BinaryDownloadHandler`, `NativeBinaryDownloadHandler` have no unit tests | High |
| Silent audit failures | `AuditRepo.insertAudit` swallows exceptions with no logging at error level | High |
| Deprecated endpoint active | `GET /download/sdkman/version/:channel` marked TODO in `Main.java` but not removed | Medium |
| Scala model dependency via JitPack | `sdkman-persistent-model` not on Maven Central; JitPack is less reliable for CI | Medium |
| Duplicate SLF4J binding | `slf4j-simple` at both `1.7.5` and `2.0.17` in `build.gradle` | Low |
| `NativeTarget.of` uses switch | Adding a new native platform requires editing both `Platform.java` and `NativeTarget.java` | Low |
| `BinaryVersionHandler` channel validation | Non-`stable` channels silently treated as beta with no validation error | Low |

## Obsolete Technology

| Library | Version in Use | Current Version | Notes |
|---|---|---|---|
| `mongo-java-driver` | 3.2.2 | 5.x | End of support; uses deprecated `MongoClient` and `BasicDBObject` API |
| `slf4j-simple` | 1.7.5 | 2.x | Older binding present alongside the newer 2.0.17 |
| `cucumber-groovy` | 6.10.4 | deprecated | `cucumber-groovy` is no longer maintained; recommend migrating to `cucumber-java` |

## Safe-to-Change Areas
- `Platform` enum — well-tested via `PlatformSpec` (30+ test cases); adding or modifying a platform alias is safe
- `ArchiveType` enum — simple, tested via `content_headers.feature`; adding a new type is low risk
- `NativeTarget` enum — tested via `native.feature`; adding a new target is safe but requires a matching `Platform` entry
- `VersionHandler` — trivial; renders config as JSON; no logic to break
- `MongoConfig` — pure config POJO; adding a new field is safe

## Recommendations
1. Upgrade `mongo-java-driver` to 5.x and migrate from `MongoClient` / `BasicDBObject` to the modern API — this is the highest-impact debt item
2. Add error logging and alerting to `AuditRepo.insertAudit` at minimum; consider a retry or dead-letter mechanism
3. Add unit tests for `CandidateDownloadHandler`, `BinaryDownloadHandler`, and `NativeBinaryDownloadHandler` using Ratpack's `RequestFixture`
4. Replace null-field config pattern in `BinaryDownloadConfig` and `NativeBinaryDownloadConfig` with proper required-field validation at startup
5. Remove the deprecated `/download/sdkman/version/:channel` endpoint after confirming no active clients
