# Business Rules

## Overview
Business rules are focused and few. The core logic centres on platform resolution (mapping raw OS strings to canonical platform identifiers), binary URL resolution (platform-specific vs UNIVERSAL fallback), and CLI version routing (stable vs beta, bash vs native). Rules are implemented entirely in Java/Kotlin — no stored procedures or external rule engines.

## Core Rules

| Rule | Description | Location in Code | Notes |
|---|---|---|---|
| Platform normalisation | Raw platform string mapped to `Platform` enum via case-insensitive prefix match | `Platform.of()` in `Platform.java` | Null input returns empty; unrecognised string returns empty |
| UNIVERSAL fallback | If no platform-specific version entry exists in MongoDB, fall back to the `UNIVERSAL` entry | `DownloadResolver.resolve()` | If neither platform nor UNIVERSAL exists, return empty (→ 404) |
| Archive type detection | Binary URL file extension determines `X-Sdkman-ArchiveType` header value | `ArchiveType.fromUrl()` | Defaults to `zip` if extension does not match `.zip` or `.tar.gz` |
| Checksum header priority | Multiple checksum algorithms returned in priority order (strongest first) | `algoComparator` in `CandidateDownloadHandler` | Priority values come from Scala `MD5.priority()`, `SHA1.priority()` etc. in `sdkman-persistent-model` |
| Native target mapping | `Platform` enum mapped to Rust target triple for native CLI downloads | `NativeTarget.of()` in `NativeTarget.java` | Only 6 of 12 `Platform` values have a native target; others return empty (→ 404) |
| CLI version by channel + impl | `stable`/`beta` channel and `bash`/`native` implementation determine which field to read from `application` collection | `AppRepo.cliVersionField()` | Any channel other than `stable` is treated as `beta`; any impl other than `native` is treated as `bash` |
| Latest bash CLI redirect | Version string prefixed with `latest` triggers redirect to GitHub's `latest` release path | `BinaryDownloadHandler.handle()` | Non-`latest` versions redirect to the versioned release path |
| Audit every download | All download handlers record a MongoDB audit entry on every request | `AuditRepo.insertAudit()` — called in `CandidateDownloadHandler`, `BinaryDownloadHandler`, `NativeBinaryDownloadHandler` | Fire-and-forget; audit entry may not be recorded if MongoDB is unavailable |

## Workflows

**SDK candidate download flow:**
1. Client sends `GET /download/:candidate/:version/:platform`
2. Validate path tokens present; return `400` if missing
3. Resolve `platform` string → `Platform` enum; return `400` if unrecognised
4. Query MongoDB `versions` collection for matching `candidate` + `version`
5. Resolve platform-specific entry; fall back to `UNIVERSAL`; return `404` if neither found
6. Write audit entry to MongoDB (async, fire-and-forget)
7. Set `X-Sdkman-Checksum-*` headers for each available algorithm
8. Set `X-Sdkman-ArchiveType` header from URL extension
9. Return `302` redirect to binary URL

**SDKMAN! bash CLI download flow:**
1. Client sends `GET /download/sdkman/:command/:version/:platform`
2. Validate path tokens; return `400` if missing
3. Construct redirect URL from `binary.properties` template
4. If version starts with `latest`, use `latest` release path; otherwise use versioned path
5. Write audit entry (async)
6. Return `302` redirect

**Native CLI download flow:**
1. Client sends `GET /download/native/:command/:version/:platform`
2. Validate path tokens; return `400` if missing
3. Resolve `platform` string → `Platform` enum; return `404` if unrecognised
4. Map `Platform` → `NativeTarget` (Rust triple); return `404` if no native target for this platform
5. Construct redirect URL from `native_binary.properties` template + Rust triple
6. Write audit entry (async)
7. Return `302` redirect

## Validations

| Validation | Rule | Handler | Error |
|---|---|---|---|
| Required path tokens | `command`, `version`, `platform` must all be non-null | All download handlers via `RequestDetails.of()` | `400` |
| Candidate + version path tokens | `candidate` and `version` must be non-null | `CandidateDownloadHandler` via inner `RequestDetails.of()` | `404` |
| Platform recognisability | Raw platform string must match at least one `Platform` alias | `Platform.of()` | `400` (binary/native) or `404` (candidate, via DownloadResolver) |
| Native platform support | Platform must have a corresponding `NativeTarget` | `NativeTarget.of()` | `404` |
| Version existence | Candidate + version must exist in MongoDB `versions` collection | `VersionRepo.fetch()` → `DownloadResolver.resolve()` | `404` |

## Edge Cases
- **UNIVERSAL platform entry** — a candidate with a single `UNIVERSAL` platform entry will be served to all recognised platforms, including exotic ones like `SunOS` and `FreeBSD`
- **`Exotic` platform** — `Platform.EXOTIC` exists in the enum and resolves from the string `"Exotic"`. A `UNIVERSAL` binary will be returned for exotic platforms; a platform-specific exotic binary is theoretically possible but unusual.
- **`latest` version prefix** — only the bash CLI handler (`BinaryDownloadHandler`) has special handling for `latest`-prefixed versions. The candidate and native handlers do not.
- **Multiple checksum algorithms** — `CandidateDownloadHandler` returns one `X-Sdkman-Checksum-*` header per algorithm present in the `checksums` map, ordered by priority. Clients may choose which to use.
- **Platform as query param** — `GET /download/:candidate/:version?platform=Darwin` is supported as an alternative to the path token form; both are handled by `CandidateDownloadHandler`

## Exceptions
- `400` — required path tokens missing (all handlers)
- `404` — platform unrecognised, version not in MongoDB, or no native target for platform
- `503` — health check fails (MongoDB `application` collection empty or unreachable)
- `500` — unhandled exceptions (e.g. MongoDB connection failure during version lookup; NPE from missing config)

## Time-Based Logic
- `AuditEntry.timestamp` is set to `System.currentTimeMillis()` at object construction time in the handler, before the async MongoDB write. If there is a scheduling delay between construction and write, the recorded timestamp may slightly precede the actual write time.
- No scheduled jobs, cron tasks, or time-windowed business logic.
- No timezone-specific logic.

## Inconsistencies
- `GET /download/sdkman/version/:channel` is marked `//TODO: deprecated` in `Main.java` but remains active and is still tested in `resource.feature` (with `# TODO: Retire these legacy endpoints!!!` comment). The endpoint returns the same result as `GET /version/sdkman/bash/:channel`.
- `BinaryVersionHandler.handle()` uses a `when` expression on `channel` with a `null` branch followed by a catch-all branch — the catch-all accepts any non-null string, meaning invalid channel values (e.g. `"nightly"`) silently return the beta version instead of a `400` error.

## Risks
- **Platform prefix matching** — `Platform.of("linuxx64extra")` would incorrectly match `LINUX_64` because the match is prefix-based with no length guard. Unusual client-sent platform strings could resolve to an unintended platform.
- **`AppRepo.findVersion` no-filter read** — business rule for CLI version routing depends on the first document in the `application` collection. If multiple documents are inserted, version routing becomes non-deterministic.
- **UNIVERSAL binary served to all platforms** — a misconfigured `UNIVERSAL` entry could cause the wrong binary to be served to platform-specific clients with no error indication.
