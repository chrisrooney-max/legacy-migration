# Testing & Quality

## Testing Strategy
Two-layer approach: Spock unit tests (no external dependencies) and Cucumber functional acceptance tests (require live MongoDB). There are no E2E tests against a deployed environment. Handler classes have no unit tests — coverage of handler logic relies entirely on the functional test suite.

## Test Types

| Type | Framework | Location | DB Required |
|---|---|---|---|
| Unit | Spock 2.3 / Groovy 3.0 | `src/test/groovy/` | No |
| Functional (acceptance) | Cucumber 6.11.0 / Groovy | `src/funtest/` | Yes (live MongoDB) |

## Coverage

| Area | Coverage Level | Notes |
|---|---|---|
| `download.Platform` | High | `PlatformSpec` covers 30+ platform string mappings including edge cases |
| `download.DownloadResolver` | High | `DownloadResolverSpec` covers platform match, UNIVERSAL fallback, and no-match cases |
| `binary.RequestDetails` | High | `BinaryDownloadRequestDetailsSpec` covers all valid/invalid path token combinations |
| `download.CandidateDownloadHandler` | Medium | Covered by functional tests (`download.feature`, `url_structure.feature`, `content_headers.feature`) only; no unit tests |
| `binary.BinaryDownloadHandler` | Medium | Covered by `resource.feature` functional tests; no unit tests |
| `rust.NativeBinaryDownloadHandler` | Medium | Covered by `native.feature` functional tests; no unit tests |
| `app.AppRepo` + `health.MongoHealthCheck` | Medium | Covered by `alive.feature`; no unit tests |
| `version.VersionRepo` | Medium | Exercised by download functional tests indirectly |
| `version.VersionHandler` | Low | `version.feature` checks response code and field presence only |
| `db.MongoProvider` + `db.MongoConfig` | None | No tests — only exercised implicitly via functional tests |
| `binary.BinaryVersionHandler` | Medium | Covered by `resource.feature` version scenarios |
| `audit.AuditRepo` | Low | Tested as a side effect in functional tests (audit entry assertions); no unit tests |

## Critical Test Scenarios
- Platform string → `Platform` enum resolution including all Windows shell variants (CYGWIN, MINGW, MSYS) — `PlatformSpec`
- UNIVERSAL fallback when no platform-specific binary exists — `DownloadResolverSpec`, `download.feature`
- No match returns empty (e.g. FreeBSD with no UNIVERSAL) — `DownloadResolverSpec`, `download.feature`
- Audit entry recorded for every successful download — asserted in `download.feature`, `resource.feature`, `native.feature`
- `X-Sdkman-Checksum-*` headers returned with correct algorithm and value — `download.feature`
- `X-Sdkman-ArchiveType` header reflects URL extension — `content_headers.feature`
- Native binary redirect includes correct Rust target triple — `native.feature`
- Health check returns `503` when `application` collection is empty — `alive.feature`
- Legacy `GET /download/sdkman/version/:channel` returns correct version — `resource.feature` (marked TODO for retirement)

## Manual Testing
- Database-inaccessible health check scenario — tagged `@manual` in `alive.feature` with comment "can't implement because of infrastructure"; requires stopping MongoDB mid-test

## Gaps
- No unit tests on any handler class (`CandidateDownloadHandler`, `BinaryDownloadHandler`, `NativeBinaryDownloadHandler`, `BinaryVersionHandler`)
- No tests for `MongoProvider` connection failure or timeout behaviour
- No tests for `BinaryDownloadConfig` / `NativeBinaryDownloadConfig` missing-property behaviour (NPE path)
- No tests for `download.CandidateDownloadHandler.RequestDetails` (inner static class) — only the outer handler is tested via functional tests
- No tests for `VersionRepo.toMap` null-handling edge case
- No load or performance tests

## Release Validation
Based on CI config (`.github/workflows/pr.yml`, `release.yml`):
1. Run `./gradlew check` — executes unit tests and functional tests (requires MongoDB)
2. Verify `GET /health/alive` returns `200` after deployment
3. Verify `GET /version` returns expected app name and version

> ⚠️ Unclear: The exact release validation steps and any manual smoke tests are not documented in the repository beyond the CI workflow files.

## Known Issues
- `@manual` scenario in `alive.feature` for inaccessible MongoDB — permanently skipped; no automated coverage for this failure mode
- `cucumber-groovy` 6.10.4 is deprecated; the project has not migrated to a supported Cucumber integration
- Functional tests call `db.drop()` in cleanup — this drops the entire database, which could cause issues if tests run against a shared dev MongoDB instance
