# Module: download

> **Package:** `io.sdkman.broker.download` | **Files:** `CandidateDownloadHandler.java`, `DownloadResolver.java`, `Platform.java`, `ArchiveType.java` | **Language:** Java

Handles download requests for third-party SDK candidates (e.g. Java, Groovy, Kotlin). Resolves the platform-appropriate binary URL from MongoDB and issues a `302` redirect. Also writes checksum and archive type headers on successful resolution.

---

## Responsibilities

- Validate incoming download request path tokens (candidate, version, platform)
- Query MongoDB `versions` collection for matching entries
- Resolve the best-matching URL for the requested platform, with `UNIVERSAL` fallback
- Write `X-Sdkman-Checksum-<algo>` response headers for all available checksum algorithms
- Write `X-Sdkman-ArchiveType` response header (`zip` or `tar`)
- Issue `302` redirect to the resolved binary URL
- Record an audit entry via `AuditRepo` on every successful download

---

## Public Interface

| Class | Method | Signature | Description |
|---|---|---|---|
| `CandidateDownloadHandler` | `handle` | `void handle(Context ctx)` | Ratpack handler entry point. Orchestrates validation, resolution, audit, and redirect. |
| `DownloadResolver` | `resolve` | `Optional<Version> resolve(List<Version> versions, String platform)` | Selects the platform-specific version or falls back to `UNIVERSAL`. |
| `Platform` | `of` | `static Optional<Platform> of(String id)` | Maps a raw platform string (case-insensitive, prefix match) to a `Platform` enum value. |
| `Platform` | `id` | `String id()` | Returns the canonical platform identifier (first alias in the list). |
| `ArchiveType` | `fromUrl` | `static ArchiveType fromUrl(String url)` | Infers archive type from URL file extension. Defaults to `ZIP` if no match. |

`CandidateDownloadHandler.RequestDetails` (inner static class):

| Method | Signature | Description |
|---|---|---|
| `of` | `static Optional<RequestDetails> of(Context ctx)` | Extracts and validates path tokens and headers from the Ratpack context. |

---

## Dependencies

**Internal:**
| Dependency | Purpose |
|---|---|
| `io.sdkman.broker.audit.AuditRepo` | Writes audit record on successful download |
| `io.sdkman.broker.audit.AuditEntry` | Audit record model |
| `io.sdkman.broker.version.VersionRepo` | Fetches candidate version entries from MongoDB |

**External:**
| Library | Version | Purpose |
|---|---|---|
| `sdkman-persistent-model` | 1.1.1 | Provides `Version`, `MD5`, `SHA1`, `SHA256`, etc. (Scala types) |
| Ratpack core | 1.9.0 | `Handler`, `Context`, `PathTokens`, `Request` |
| `scala-library` | transitive | Required by `Version` and checksum model classes |

---

## Inputs and Outputs

**Input:** HTTP `GET` request with path tokens:
- `candidate` — SDK name (e.g. `java`, `groovy`)
- `version` — version string (e.g. `11.0.9-adpt`)
- `platform` — platform string (e.g. `darwinx64`, `linuxx64`); can also be provided as `?platform=` query param

Headers read: `X-Real-IP` (client IP), `user-agent`

**Output:** HTTP `302` redirect to the binary URL, with headers:
- `X-Sdkman-Checksum-<algo>` — one header per available checksum algorithm, ordered by priority (SHA-512 first)
- `X-Sdkman-ArchiveType` — `zip` or `tar`

**Side effects:**
- Inserts a row into the MongoDB `audit` collection on successful resolution
- Returns `400` if path tokens are missing; `404` if platform is unrecognised or no matching version exists

---

## Known Issues / Technical Debt

> ⚠️ **Unclear:** `algoComparator` in `CandidateDownloadHandler` defines a priority ordering for checksum algorithms but the priority values come from the Scala `MD5.priority()`, `SHA1.priority()` etc. methods. The actual numeric values are not visible in this codebase — they are defined in `sdkman-persistent-model`.

- `DownloadResolver` has no constructor — `@Inject` is not present. It is instantiated by Guice via `bind(DownloadResolver.class)` but not explicitly listed in `Main.java`. Injection must succeed because no binding is declared; this works because Guice can construct classes with a default constructor, but it is inconsistent with the rest of the DI setup.
- Platform string matching is case-insensitive prefix match. A platform string of `linuxx64extra` would incorrectly match `LINUX_64`. No length or exact-match guard is applied.
- `RequestDetails.determineNormalisedPlatform` lowercases the query param platform value but not the path token value. The `Platform.of()` method handles case-insensitivity itself, but the asymmetry is a latent inconsistency.
