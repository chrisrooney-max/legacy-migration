# Code Structure

## Repository Overview
- **Repo name:** sdkman/sdkman-broker
- **Purpose:** HTTP redirect broker for SDKMAN! binary downloads and CLI version lookups

## Directory Structure

```
sdkman-broker/
├── src/
│   ├── main/
│   │   ├── java/io/sdkman/broker/
│   │   │   ├── Main.java                          # Entry point; Ratpack server + all route definitions
│   │   │   ├── NativeTarget.java                  # Enum: Platform → Rust target triple
│   │   │   ├── audit/
│   │   │   │   ├── AuditEntry.java                # Audit record model
│   │   │   │   └── AuditRepo.java                 # MongoDB audit collection writer
│   │   │   ├── binary/
│   │   │   │   ├── BinaryDownloadConfig.java      # Config POJO for bash CLI download URL template
│   │   │   │   ├── BinaryDownloadHandler.java     # Bash CLI download redirect handler
│   │   │   │   └── RequestDetails.java            # Path token + header extractor (shared with rust/)
│   │   │   ├── db/
│   │   │   │   ├── MongoConfig.java               # MongoDB connection config POJO
│   │   │   │   └── MongoProvider.java             # Singleton MongoDB client
│   │   │   ├── download/
│   │   │   │   ├── ArchiveType.java               # Enum: URL extension → zip/tar
│   │   │   │   ├── CandidateDownloadHandler.java  # SDK candidate download redirect handler
│   │   │   │   ├── DownloadResolver.java          # Platform match + UNIVERSAL fallback logic
│   │   │   │   └── Platform.java                  # Enum: raw platform string → canonical Platform
│   │   │   ├── rust/
│   │   │   │   ├── NativeBinaryDownloadConfig.java # Config POJO for native CLI URL template
│   │   │   │   └── NativeBinaryDownloadHandler.java # Native CLI download redirect handler
│   │   │   └── version/
│   │   │       ├── VersionConfig.java             # App name/version config POJO
│   │   │       ├── VersionHandler.java            # GET /version endpoint handler
│   │   │       └── VersionRepo.java               # MongoDB versions collection reader
│   │   ├── kotlin/io/sdkman/broker/
│   │   │   ├── app/
│   │   │   │   └── AppRepo.kt                     # MongoDB application collection: health + CLI version
│   │   │   ├── binary/
│   │   │   │   └── BinaryVersionHandler.kt        # GET /version/sdkman/:impl/:channel handler
│   │   │   └── health/
│   │   │       └── MongoHealthCheck.kt            # Ratpack HealthCheck backed by AppRepo
│   │   └── resources/
│   │       ├── binary.properties                  # Bash CLI GitHub Releases URL template
│   │       ├── native_binary.properties           # Native CLI GitHub Releases URL template
│   │       └── version.properties                 # App name and version
│   ├── test/groovy/                               # Spock unit tests (no DB required)
│   └── funtest/                                   # Cucumber acceptance tests (live MongoDB required)
│       ├── features/                              # 7 Gherkin feature files
│       └── groovy/                               # Step definitions and MongoHelper
├── Dockerfile                                     # openjdk:11 base; copies fat JAR; -Xmx128m
├── build.gradle                                   # Gradle build; Shadow plugin for fat JAR
└── gradle.properties                             # version=1.0.0-SNAPSHOT
```

## Key Modules

| Module | Responsibility | Entry Points | Notes |
|---|---|---|---|
| `download` | Candidate SDK download routing | `CandidateDownloadHandler.handle()` | Most complex handler; Scala interop via VersionRepo |
| `binary` (Java) | Bash CLI download routing | `BinaryDownloadHandler.handle()` | URL built from config; no DB lookup |
| `binary` (Kotlin) | CLI version lookup | `BinaryVersionHandler.handle()` | Reads from MongoDB application collection |
| `rust` | Native CLI download routing | `NativeBinaryDownloadHandler.handle()` | Resolves Rust triple via NativeTarget enum |
| `audit` | Download event recording | `AuditRepo.insertAudit()` | Fire-and-forget; called by all download handlers |
| `db` | MongoDB client management | `MongoProvider.database()` | Singleton; constructed at Guice startup |
| `app` (Kotlin) | Health + CLI version data | `AppRepo.healthCheck()`, `AppRepo.findVersion()` | Single collection serves two purposes |
| `version` | App version endpoint + version DB reads | `VersionHandler.handle()`, `VersionRepo.fetch()` | Scala Version model returned from DB |

## Entry Points
- **HTTP:** `Main.main()` — starts Ratpack server; all 8 routes defined here
- **Health:** `GET /health/alive` → `MongoHealthCheck`
- **Version:** `GET /version` → `VersionHandler`
- **CLI version:** `GET /version/sdkman/:impl/:channel` → `BinaryVersionHandler`
- **Downloads:** `GET /download/**` → `CandidateDownloadHandler`, `BinaryDownloadHandler`, `NativeBinaryDownloadHandler`

## Configuration

| File / Source | Key Properties |
|---|---|
| `binary.properties` | `binary.protocol`, `binary.host`, `binary.uri`, `binary.name` |
| `native_binary.properties` | `native.protocol`, `native.host`, `native.uri`, `native.name` |
| `version.properties` | `broker.appName`, `broker.appVersion` |
| Environment variables | `MONGO_HOST`, `MONGO_PORT`, `MONGO_DB_NAME`, `MONGO_USERNAME`, `MONGO_PASSWORD` |

Config is loaded by Ratpack's layered config system: properties files → env vars → system properties (later sources override earlier ones).

## Coding Patterns
- **Handler pattern** — each route maps to a dedicated `Handler` implementation; no shared controller layer
- **Repository pattern** — `VersionRepo`, `AuditRepo`, `AppRepo` encapsulate all MongoDB access
- **Guice DI** — all handlers and repos are singleton-scoped and constructor-injected
- **Optional for null safety** — `Optional<T>` used throughout Java code for absent values; Arrow `Option` used in Kotlin modules
- **Blocking wrapper** — all MongoDB calls wrapped in `Blocking.get` or `Blocking.exec` to keep Ratpack's event loop free

## Dependencies

**Internal:**
- `binary.RequestDetails` is reused by `rust.NativeBinaryDownloadHandler` (cross-package coupling)
- `download.Platform` is used by `rust.NativeBinaryDownloadHandler` and `audit.AuditEntry`

**External (key):**

| Library | Version | Purpose |
|---|---|---|
| Ratpack | 1.9.0 | HTTP server and non-blocking runtime |
| Google Guice | via Ratpack | Dependency injection |
| `mongo-java-driver` | 3.2.2 | MongoDB client (legacy API) |
| `sdkman-persistent-model` | 1.1.1 | Scala `Version` domain model (JitPack) |
| Arrow | 1.1.5 | Functional `Option` type in Kotlin modules |
| Spock | 2.3-groovy-3.0 | Unit test framework |
| Cucumber | 6.11.0 | Functional acceptance tests |

## Areas of Concern
- **`VersionRepo`** — Scala interop (`JavaConverters`, `Option.apply(null)`) makes this the most fragile class in the codebase
- **`BinaryDownloadConfig` and `NativeBinaryDownloadConfig`** — all fields declared `final = null`; missing config causes NPE at request time, not startup
- **`AuditRepo.insertAudit`** — no error handling; audit loss is invisible
- **`Main.java`** — deprecated route (`/download/sdkman/version/:channel`) marked TODO but not removed; no removal timeline
