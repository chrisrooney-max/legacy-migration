# Data Model

## Overview
MongoDB document store with three collections. The service only writes to `audit`; `versions` and `application` are written by external systems (SDKMAN! platform tooling). No schema validation is enforced at the database level — the Java/Kotlin code implicitly defines the expected document shape.

## Core Entities

| Entity | Description | Owner | Notes |
|---|---|---|---|
| `Version` | A binary download entry for a specific candidate, version, and platform | External platform tooling | Scala model from `sdkman-persistent-model` |
| `AuditEntry` | A record of a single download event | This service (`AuditRepo`) | Append-only; never updated or deleted |
| `Application` | Config document holding health sentinel and CLI version strings | External platform tooling | Single document serving multiple purposes |

## Relationships
No cross-collection references. Each collection is independent. The broker reads `versions` and `application` to serve requests, and writes to `audit` as a side effect.

## Data Storage
- **Database:** MongoDB
- **Type:** Document store (NoSQL)
- **Database name:** `sdkman` (default; configurable via `MONGO_DB_NAME`)
- **Driver version:** `mongo-java-driver` 3.2.2 (legacy synchronous API)

## Key Tables / Collections

| Name | Purpose | Key Fields | Risk |
|---|---|---|---|
| `versions` | Binary download URL catalogue | `candidate`, `version`, `platform`, `url`, `checksums`, `vendor`, `visible` | Written externally; no schema enforcement; Scala model tightly coupled to document shape |
| `audit` | Append-only download event log | `command`, `candidate`, `version`, `host`, `agent`, `platform`, `dist`, `timestamp` | Fire-and-forget writes; events silently lost if MongoDB is unavailable |
| `application` | Health sentinel and CLI version config | `alive`, `stableCliVersion`, `betaCliVersion`, `stableNativeCliVersion`, `betaNativeCliVersion` | Single document serves two unrelated purposes; `AppRepo.findVersion` reads the first document with no filter |

### `versions` document shape (from `MongoHelper` test helper)
```json
{
  "_id": ObjectId("..."),
  "_class": "Version",
  "candidate": "java",
  "version": "11.0.9-adpt",
  "platform": "MAC_OSX",
  "url": "https://example.org/java.tar.gz",
  "vendor": "Adoptium",
  "visible": true,
  "checksums": {
    "SHA-256": "438dd6...",
    "SHA-1": "6f1b95..."
  }
}
```

### `audit` document shape (from `AuditRepo`)
```json
{
  "_id": ObjectId("..."),
  "command": "install",
  "candidate": "java",
  "version": "11.0.9-adpt",
  "host": "203.0.113.1",
  "agent": "curl/7.54.0",
  "platform": "DarwinX64",
  "dist": "MAC_OSX",
  "timestamp": 1710000000000
}
```

### `application` document shape (from `MongoHelper`)
```json
{
  "_id": "0",
  "alive": "OK",
  "stableCliVersion": "5.18.2",
  "betaCliVersion": "latest+abcdef",
  "stableNativeCliVersion": "0.4.0",
  "betaNativeCliVersion": "0.5.0-beta"
}
```

## Data Lifecycle
- **`versions`** — Created and updated by external SDKMAN! platform tooling. Never written by this service. Deletion policy unknown.
- **`audit`** — Append-only. Each download request adds one document. No TTL, archival, or deletion logic observed in this codebase.
- **`application`** — Created and updated by external tooling. Read-only from this service's perspective.

## Data Quality Issues
- `application` collection mixes health sentinel (`alive: "OK"`) with CLI version fields in the same document. If the document structure changes, both health checks and version lookups are affected simultaneously.
- `AppRepo.findVersion` calls `.find().first()` with no filter — it returns the first document MongoDB happens to return. If multiple documents exist, results are non-deterministic.
- No MongoDB schema validation rules defined — invalid documents (missing `url`, wrong field types) would cause runtime exceptions in `VersionRepo`.
- `checksums` field in `versions` is optional and may be absent or null; `VersionRepo.toMap` returns `null` in that case, relying on `Option.apply(null)` to produce `None`.

## Reporting Dependencies
> ⚠️ Unclear: No reporting or analytics queries are present in this codebase. The `audit` collection likely feeds external analytics or reporting tools, but these are not visible here.

## Migration Risks
- The `versions` collection document shape is implicitly defined by the Scala `Version` model in `sdkman-persistent-model`. Any change to that library's model requires corresponding changes to `VersionRepo.fetch()`.
- The `application` collection's field names (`stableCliVersion`, etc.) are hardcoded strings in `AppRepo.cliVersionField()`. Renaming them requires coordinated changes across all writers and this service.
