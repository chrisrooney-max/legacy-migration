# Module: version

> **Package:** `io.sdkman.broker.version` | **Files:** `VersionHandler.java`, `VersionRepo.java`, `VersionConfig.java` | **Language:** Java

`VersionHandler` serves the `/version` endpoint, returning the application name and version as JSON. `VersionRepo` queries the MongoDB `versions` collection to fetch all entries for a given candidate + version combination, used by `CandidateDownloadHandler`.

---

## Responsibilities

- Serve application name and version at `GET /version`
- Query MongoDB `versions` collection by candidate and version string
- Map MongoDB documents to `Version` domain objects (from `sdkman-persistent-model`)

---

## Public Interface

| Class | Method | Signature | Description |
|---|---|---|---|
| `VersionHandler` | `handle` | `void handle(Context ctx)` | Renders `VersionConfig` as JSON |
| `VersionRepo` | `fetch` | `Promise<List<Version>> fetch(String candidate, String version)` | Fetches all `Version` entries matching candidate and version from MongoDB |

`VersionConfig` is a config POJO with no behaviour:

| Field | Type | Source |
|---|---|---|
| `appName` | String | `version.properties` → `broker.appName` |
| `appVersion` | String | `version.properties` → `broker.appVersion` |

---

## Dependencies

**Internal:**
| Dependency | Purpose |
|---|---|
| `io.sdkman.broker.db.MongoProvider` | MongoDB database handle |

**External:**
| Library | Version | Purpose |
|---|---|---|
| `sdkman-persistent-model` | 1.1.1 | `Version` domain object (Scala) |
| `mongo-java-driver` | 3.2.2 | `MongoCollection`, `Document`, filter builders |
| Ratpack exec | 1.9.0 | `Blocking.get`, `Promise` |
| `scala-library` | transitive | `Option`, `Some`, `JavaConverters` used in document mapping |

---

## Inputs and Outputs

**`VersionHandler`**

Input: none (no path tokens or query params)
Output: JSON object rendered by Ratpack Jackson:
```json
{
  "appName": "SDKMAN! Broker",
  "appVersion": "1.0.0-SNAPSHOT"
}
```

**`VersionRepo.fetch`**

Input: `candidate` (String), `version` (String) — matched against the `versions` collection fields `candidate` and `version`.

Output: `Promise<List<Version>>` — a list of `Version` objects, one per platform entry stored in MongoDB for that candidate/version pair. Each `Version` contains:

| Field | Type | Description |
|---|---|---|
| `candidate` | String | SDK name |
| `version` | String | Version string |
| `platform` | String | Platform identifier (e.g. `MAC_OSX`, `UNIVERSAL`) |
| `url` | String | Binary download URL |
| `vendor` | Option[String] | Vendor name (may be absent) |
| `visible` | Option[Boolean] | Visibility flag (may be absent) |
| `checksums` | Option[Map[String,String]] | Algorithm → checksum value pairs (may be absent) |

---

## Known Issues / Technical Debt

- `VersionRepo.toMap` returns `null` (not an empty map or `Option.empty()`) when the `checksums` field is absent or not a `Document`. This null is wrapped in `Option.apply(null)`, which produces `Option.empty()` in Scala — correct in practice, but relying on `Option.apply(null)` is an implicit convention that is easy to break.
- The `Blocking.get` call in `VersionRepo.fetch` blocks a thread per request for the duration of the MongoDB query. Under high load this may saturate the blocking executor if MongoDB is slow.
