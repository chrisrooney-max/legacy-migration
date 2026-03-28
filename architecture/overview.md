# Architecture Overview — SDKMAN! Broker

> **System:** sdkman-broker v1.0.0 | **Languages:** Java 11, Kotlin | **Last active development:** 2023

SDKMAN! Broker is a lightweight HTTP redirect service. It receives download requests for SDK candidates and SDKMAN! CLI binaries, resolves the correct remote URL from MongoDB, issues a `302` redirect, and records an audit entry. It does not serve binaries directly.

---

## Tech Stack

| Layer | Technology | Version | Notes |
|---|---|---|---|
| Language | Java | 11 | Used for all handlers, enums, and DB layer |
| Language | Kotlin | 1.7.21 | Used for `AppRepo`, `BinaryVersionHandler`, `MongoHealthCheck` |
| Framework | Ratpack | 1.9.0 | Non-blocking HTTP; Guice for DI |
| Database | MongoDB | 3.2.2 (driver) | Three collections: `versions`, `audit`, `application` |
| DI | Google Guice | via Ratpack | Singleton-scoped handlers and repos |
| FP library | Arrow | 1.1.5 | `Option` types in Kotlin modules only |
| Build | Gradle + Shadow | 8.1.1 | Produces fat JAR (`*-all.jar`) for Docker |
| Runtime | OpenJDK | 11 | Docker image base |
| External model | sdkman-persistent-model | 1.1.1 | Shared `Version` Scala model, sourced from JitPack |

---

## Top-Level Structure

```
sdkman-broker/
├── src/
│   ├── main/
│   │   ├── java/io/sdkman/broker/
│   │   │   ├── Main.java              # Entry point; Ratpack server config and all route definitions
│   │   │   ├── NativeTarget.java      # Enum: Platform → Rust target triple mapping
│   │   │   ├── audit/                 # AuditEntry model + AuditRepo (MongoDB write)
│   │   │   ├── binary/                # Bash CLI download handler, config, and RequestDetails
│   │   │   ├── db/                    # MongoProvider (client) and MongoConfig
│   │   │   ├── download/              # Candidate download handler, DownloadResolver, Platform/ArchiveType enums
│   │   │   ├── rust/                  # Native (Rust) CLI download handler and config
│   │   │   └── version/               # App version endpoint, VersionRepo, VersionConfig
│   │   └── kotlin/io/sdkman/broker/
│   │       ├── app/                   # AppRepo: MongoDB application collection (health + CLI version lookup)
│   │       ├── binary/                # BinaryVersionHandler: returns CLI version string by channel
│   │       └── health/                # MongoHealthCheck: Ratpack HealthCheck backed by AppRepo
│   └── resources/
│       ├── binary.properties          # Bash CLI download URL template pointing to GitHub Releases
│       ├── native_binary.properties   # Native CLI download URL template pointing to GitHub Releases
│       └── version.properties         # App name and version injected into /version endpoint
├── src/funtest/                       # Cucumber acceptance tests (require live MongoDB)
├── src/test/                          # Spock unit tests (no DB required)
├── Dockerfile                         # Copies fat JAR; runs with -Xmx128m
└── build.gradle
```

---

## Component Diagram

```
  SDKMAN! CLI / client
         │
         │  GET /download/...
         ▼
  ┌─────────────────────────────┐
  │  Ratpack HTTP Server        │  ← Main.java — configures routes and Guice bindings
  └──────────┬──────────────────┘
             │ routes to
   ┌──────────┼───────────────────────────────────┐
   ▼          ▼              ▼            ▼        ▼
┌──────┐ ┌─────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐
│Health│ │ Version │ │Candidate │ │  Binary  │ │ Native │
│Check │ │Handler  │ │Download  │ │ Download │ │Download│
└──┬───┘ └────┬────┘ └────┬─────┘ └────┬─────┘ └───┬────┘
   │          │           │ DB lookup   │ config     │ config
   │          │           ▼            │ template   │ template
   ▼          ▼    ┌────────────┐      │            │
 AppRepo  VersionConfig│VersionRepo │      │            │
(MongoDB) (properties) └─────┬──────┘      │            │
                             │             │            │
                    ┌────────┴─────────────┴────────────┘
                    │        AuditRepo (all download handlers)
                    ▼
               MongoDB
        (versions | audit | application)
                    │
                    │  302 redirect
                    ▼
        Remote binary host (GitHub Releases)
```

---

## Data Flow

1. **Route** — Ratpack matches the `GET` path and dispatches to the appropriate handler.
2. **Validate** — Handler extracts path tokens and headers (`X-Real-IP`, `user-agent`). Returns `400` if required params are absent.
3. **Resolve** — For candidate downloads: `VersionRepo` queries the `versions` collection for matching candidate + version. `DownloadResolver` picks the platform-specific entry, falling back to `UNIVERSAL`. For bash/native CLI downloads: URL is built from a config template (no DB lookup).
4. **Audit** — `AuditRepo` asynchronously writes a record to the `audit` collection: command, candidate, version, platform, host IP, user-agent, and timestamp.
5. **Redirect** — Handler issues a `302` to the resolved URL. Candidate downloads also set checksum headers (`X-Sdkman-Checksum-<algo>`) and an `X-Sdkman-ArchiveType` header.

---

## Key Design Decisions

| Decision | Rationale |
|---|---|
| 302 redirect, not binary proxy | Avoids bandwidth cost on the broker; it is a routing layer only |
| Mixed Java + Kotlin | Kotlin adopted incrementally for newer modules; older handlers remain Java |
| No authentication on any endpoint | Intentional — all download URLs are public |
| Platform resolved by string prefix matching | Accommodates varied platform strings from different OS/shell environments (e.g. `MINGW64_NT-10.0`) |
| `UNIVERSAL` fallback in `DownloadResolver` | Allows single-archive candidates to be served without per-platform entries in MongoDB |
| Audit is fire-and-forget | `AuditRepo.insertAudit` uses `Blocking.exec` with no result handling; failures are silent |

---

## Technical Debt / Risk Areas

> ⚠️ **Unclear:** `GET /download/sdkman/version/:channel` is marked `//TODO: deprecated` in `Main.java` but remains active. It is unknown whether any clients still depend on it.

- **Scala interop** — The `Version` domain model is a Scala class from `sdkman-persistent-model` (JitPack). This pulls in `scala.Option`, `scala.collection.immutable.Map`, and `JavaConverters` boilerplate across `VersionRepo`. JitPack as a dependency source is a reliability risk.
- **Outdated MongoDB driver** — `mongo-java-driver` 3.2.2 is significantly behind current (5.x). Uses the legacy `MongoClient` and `BasicDBObject` API.
- **Silent audit failures** — If MongoDB is unavailable, `AuditRepo.insertAudit` fails without logging at error level or retrying. No dead-letter or queue mechanism exists.
- **Null-initialised config fields** — `BinaryDownloadConfig` and `NativeBinaryDownloadConfig` declare all fields as `final = null`. Missing properties at startup cause `NullPointerException` in the handler rather than a clear startup error.
- **No allowlist validation on path tokens** — `candidate` and `platform` values are used directly in MongoDB queries. Not a SQL-injection risk, but there is no validation against a known-good set of values.
