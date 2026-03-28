# Module: db

> **Package:** `io.sdkman.broker.db` | **Files:** `MongoProvider.java`, `MongoConfig.java` | **Language:** Java

Provides a singleton MongoDB client and database handle, shared across all repository classes. Configuration is injected by Ratpack's config system from environment variables and `application.properties`.

---

## Responsibilities

- Construct and hold a singleton `MongoClient` instance at startup
- Apply connection, socket, and server-selection timeout configuration
- Support optional credential-based authentication
- Expose the configured database via `database()`

---

## Public Interface

| Class | Method | Signature | Description |
|---|---|---|---|
| `MongoProvider` | `database` | `MongoDatabase database()` | Returns the configured MongoDB database. Called by all repo classes. |

`MongoConfig` is a config POJO with no behaviour — values are populated by Ratpack's config binding:

| Field | Type | Default | Description |
|---|---|---|---|
| `host` | String | `localhost` | MongoDB host |
| `port` | int | `27017` | MongoDB port |
| `username` | String | `null` | Auth username; if null, connects without credentials |
| `password` | String | `null` | Auth password; if null, connects without credentials |
| `dbName` | String | `sdkman` | Database name |
| `serverSelectionTimeout` | int | `5000` ms | How long to wait when selecting a server |
| `connectionTimeout` | int | `5000` ms | How long to wait when opening a connection |
| `socketTimeout` | int | `0` (unlimited) | How long to wait for socket reads |

---

## Dependencies

**Internal:** None.

**External:**
| Library | Version | Purpose |
|---|---|---|
| `mongo-java-driver` | 3.2.2 | `MongoClient`, `MongoClientOptions`, `MongoCredential`, `ServerAddress` |
| Google Guice | via Ratpack | `@Inject`, `@Singleton` |
| SLF4J | 1.7.5 | Startup log message |

---

## Inputs and Outputs

**Input:** `MongoConfig` — populated from:
1. `application.properties` (not present in repo — expected at runtime)
2. Environment variables (via Ratpack `.env()` config source)
3. System properties (via `.sysProps()`)

Config key prefix is `/mongo` as declared in `Main.java`.

**Output:** `MongoDatabase` — the named database handle returned by `database()`.

**Side effects:**
- Constructs a `MongoClient` (and opens connection pool) at Guice injection time, not on first use.

---

## Known Issues / Technical Debt

- **`mongo-java-driver` 3.2.2** is significantly outdated. The legacy synchronous driver API (`MongoClient`, `BasicDBObject`) is used rather than the modern reactive driver. No migration path is evident.
- `socketTimeout = 0` (unlimited) by default. If a MongoDB operation hangs, the Ratpack thread will block indefinitely. Combined with Ratpack's `Blocking` executor, this could exhaust the blocking thread pool under load.
- Credentials are stored as plain `String` in `MongoConfig`. The legacy `MongoCredential.createCredential` call converts the password to `char[]` at the point of use, but the `String` remains in memory.
