# Module: app + health

> **Packages:** `io.sdkman.broker.app`, `io.sdkman.broker.health` | **Files:** `AppRepo.kt`, `MongoHealthCheck.kt` | **Language:** Kotlin

`AppRepo` queries the MongoDB `application` collection for two purposes: health checking (is the collection reachable and populated?) and CLI version lookup (what is the current stable/beta CLI version?). `MongoHealthCheck` wraps `AppRepo.healthCheck()` as a Ratpack `HealthCheck` and is exposed at `/health/alive`.

---

## Responsibilities

- Query the `application` collection for a document where `alive = "OK"` (health check)
- Query the `application` collection for a CLI version field by channel and implementation type
- Report healthy/unhealthy to Ratpack's health check system

---

## Public Interface

| Class | Method | Signature | Description |
|---|---|---|---|
| `AppRepo` | `healthCheck` | `fun healthCheck(): Promise<Option<String>>` | Returns `Some("OK")` if the application collection contains the sentinel document; `None` otherwise |
| `AppRepo` | `findVersion` | `fun findVersion(impl: String, channel: String): Promise<Option<String>>` | Returns the CLI version string for the given impl/channel combination, or `None` if not found |
| `MongoHealthCheck` | `getName` | `override fun getName(): String` | Returns `"alive"` — the name used in the `/health/alive` URL |
| `MongoHealthCheck` | `check` | `override fun check(registry: Registry): Promise<Result>` | Returns `healthy` or `unhealthy` based on `AppRepo.healthCheck()` |

---

## Dependencies

**Internal:**
| Dependency | Purpose |
|---|---|
| `io.sdkman.broker.db.MongoProvider` | MongoDB database handle |

**External:**
| Library | Version | Purpose |
|---|---|---|
| Arrow | 1.1.5 | `Option`, `Some`, `None` — functional null-safe return types |
| Ratpack exec | 1.9.0 | `Blocking.get`, `Promise` |
| Ratpack health | 1.9.0 | `HealthCheck`, `HealthCheck.Result` |

---

## Inputs and Outputs

**`AppRepo.healthCheck`**

Queries `application` collection for `{ alive: "OK" }`. Returns `Some("OK")` if found, `None` if not.

**`AppRepo.findVersion`**

Reads the first document in `application` and extracts the field determined by:

| `channel` | `impl` | Field name |
|---|---|---|
| `stable` | `bash` (or anything else) | `stableCliVersion` |
| `stable` | `native` | `stableNativeCliVersion` |
| anything else | `bash` (or anything else) | `betaCliVersion` |
| anything else | `native` | `betaNativeCliVersion` |

Returns the field value as `Some<String>`, or `None` if the document is absent or the field is null.

**`MongoHealthCheck.check`**

Returns `HealthCheck.Result.healthy()` on `Some`, `HealthCheck.Result.unhealthy("Nothing found at application/alive in database.")` on `None`.

---

## Known Issues / Technical Debt

> ⚠️ **Unclear:** `AppRepo.findVersion` calls `.find().first()` with no filter — it reads the first document in the `application` collection regardless of its content. If the collection has multiple documents, version data is read from whichever document MongoDB returns first (insertion order is not guaranteed in all configurations).

- `channel` values other than `stable` are treated as beta. An unrecognised channel (e.g. a typo) silently returns the beta version rather than an error.
- The `application` collection serves dual purpose (health sentinel and version storage). The health check depends on the same document containing `alive: "OK"`, which couples the health signal to the version document structure.
