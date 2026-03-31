# Architecture

## Overview
SDKMAN! Broker is a non-blocking HTTP service built on Ratpack with Guice dependency injection. It follows a handler-per-route pattern: each route maps to a singleton handler class that validates the request, optionally queries MongoDB, and issues a 302 redirect. There is no service layer — handlers interact directly with repository classes.

## Context Diagram

```
  ┌─────────────────┐
  │  SDKMAN! CLI    │  ← end user's machine
  └────────┬────────┘
           │  HTTP GET /download/... or /version/...
           ▼
  ┌─────────────────────┐
  │  SDKMAN! Broker     │  ← this service
  │  (Ratpack / Guice)  │
  └──────┬──────────────┘
         │ reads           │ writes audit
         ▼                 ▼
  ┌─────────────┐   ┌─────────────┐
  │   MongoDB   │   │   MongoDB   │
  │  (versions) │   │   (audit)   │
  └─────────────┘   └─────────────┘
         │
         │  302 redirect
         ▼
  ┌─────────────────────┐
  │  GitHub Releases    │  ← actual binary host
  │  (or third-party    │
  │   CDN for SDK       │
  │   candidates)       │
  └─────────────────────┘
```

## Components

| Component | Responsibility | Location | Notes |
|---|---|---|---|
| `Main` | Server config and route definitions | `Main.java` | Entry point; all routes defined here |
| `CandidateDownloadHandler` | SDK candidate download redirect | `download/` | Queries MongoDB; resolves platform; sets checksum headers |
| `BinaryDownloadHandler` | Bash CLI download redirect | `binary/` (Java) | URL built from config template; no DB lookup |
| `NativeBinaryDownloadHandler` | Native CLI download redirect | `rust/` | Resolves Rust triple via `NativeTarget` |
| `BinaryVersionHandler` | CLI version string lookup | `binary/` (Kotlin) | Reads from MongoDB `application` collection |
| `VersionHandler` | App name/version endpoint | `version/` | Renders `VersionConfig` as JSON |
| `MongoHealthCheck` | Health check | `health/` (Kotlin) | Ratpack `HealthCheck`; backed by `AppRepo` |
| `VersionRepo` | MongoDB `versions` collection reads | `version/` | Returns `List<Version>` (Scala model) |
| `AuditRepo` | MongoDB `audit` collection writes | `audit/` | Fire-and-forget; no error handling |
| `AppRepo` | MongoDB `application` collection reads | `app/` (Kotlin) | Health sentinel + CLI version fields |
| `MongoProvider` | MongoDB client singleton | `db/` | Constructed at Guice injection time |
| `DownloadResolver` | Platform resolution + UNIVERSAL fallback | `download/` | Stateless; no DI annotation |
| `Platform` | Platform string normalisation | `download/` | Enum; case-insensitive prefix match |
| `NativeTarget` | Platform → Rust triple mapping | root package | Enum; subset of Platform values |
| `ArchiveType` | URL extension → archive type | `download/` | Enum; defaults to ZIP |

## Interaction Patterns
- **Sync / Request-response** — all HTTP interactions are request/response; no streaming
- **Non-blocking I/O** — Ratpack handles HTTP on a non-blocking event loop; all blocking MongoDB calls are wrapped in `Blocking.get` or `Blocking.exec`
- **Fire-and-forget** — audit writes use `Blocking.exec` with no result or error handling

## Data Flow
1. SDKMAN! CLI sends a `GET` request to the broker
2. Ratpack routes the request to the appropriate handler based on path pattern
3. Handler validates required path tokens; returns `400` if any are missing
4. For candidate downloads: `VersionRepo` queries MongoDB `versions` collection; `DownloadResolver` selects platform-specific or UNIVERSAL entry
5. For bash/native CLI downloads: handler builds the redirect URL directly from config template
6. `AuditRepo` asynchronously writes a record to MongoDB `audit` collection
7. Handler sets response headers (checksum, archive type for candidate downloads) and issues `302` redirect

## External Dependencies
- **MongoDB** — version catalogue (`versions`), download audit log (`audit`), and CLI version/health config (`application`)
- **GitHub Releases** — hosts SDKMAN! bash CLI and native CLI binaries; candidate binary URLs may point here or to third-party hosts

## Deployment Architecture
- **Build:** `./gradlew build` → shadow fat JAR (`build/libs/sdkman-broker-*-all.jar`)
- **Runtime:** Docker (`FROM openjdk:11`); JVM flags: `-Xmx128m`
- **Environments:** not defined in the codebase
- **Regions:** not defined in the codebase

> ⚠️ Unclear: Production hosting infrastructure (cloud provider, region, load balancer config) is not present in the repository.

## Key Constraints
- JDK 11 required (Dockerfile base image is `openjdk:11`)
- MongoDB driver is locked to the legacy 3.x API (`mongo-java-driver` 3.2.2)
- Scala interop required at runtime due to `sdkman-persistent-model` dependency
- Memory limited to 128m heap by Dockerfile `ENTRYPOINT`

## Known Issues
- `mongo-java-driver` 3.2.2 is significantly outdated (current: 5.x); uses deprecated `MongoClient` API
- `sdkman-persistent-model` sourced from JitPack, not Maven Central — reliability risk for builds
- Duplicate SLF4J bindings: `slf4j-simple` appears at both `1.7.5` (implementation) and `2.0.17` (runtimeOnly)
