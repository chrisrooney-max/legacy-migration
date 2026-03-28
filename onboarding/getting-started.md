# Getting Started — SDKMAN! Broker

> Estimated setup time: ~15 minutes (MongoDB via Docker, then Gradle run)

---

## Prerequisites

| Tool | Version | Notes |
|---|---|---|
| JDK | 11 | Must be JDK 11 — Dockerfile uses `openjdk:11`; Ratpack 1.9 requires Java 8+ but the project targets 11 |
| Gradle | via wrapper | Use `./gradlew` — do not install Gradle separately |
| Docker | any recent | Only needed to run MongoDB locally |
| MongoDB | any (via Docker) | The README recommends `mongo:latest`; the driver is 3.2.2 which is compatible with Mongo 3.x–6.x |
| Git | any | |

> ⚠️ **Unclear:** The `.sdkmanrc` file specifies a Java version via SDKMAN! itself. If you use SDKMAN! locally, running `sdk env` in the repo root will install and switch to the correct version automatically.

---

## Setup

**1. Clone the repository**

```bash
git clone https://github.com/sdkman/sdkman-broker.git
cd sdkman-broker
```

**2. Start MongoDB**

```bash
docker run -d --net=host mongo:latest
```

This starts MongoDB on the default port `27017` on localhost. The application connects to `sdkman` database by default (see `MongoConfig` defaults).

> The `application` collection must contain a document with `{ alive: "OK" }` for the health check to pass and for CLI version fields (`stableCliVersion`, `betaCliVersion`, etc.) to be returned. The functional tests seed this data — see the test step definitions in `src/funtest/groovy/steps/persistence.groovy` for the document structure.

**3. Build**

```bash
./gradlew build -x funtest
```

The `-x funtest` flag skips the functional tests, which require additional setup (see [Running Tests](#running-tests)). The shadow JAR is written to `build/libs/sdkman-broker-*-all.jar`.

---

## Running the Application

```bash
./gradlew run
```

Ratpack starts on port `5050` by default. Verify it is running:

```bash
curl http://localhost:5050/version
# Expected: {"appName":"SDKMAN! Broker","appVersion":"1.0.0-SNAPSHOT"}

curl http://localhost:5050/health/alive
# Expected: {"name":"alive","healthy":true}  (requires seeded MongoDB)
```

Logs are written to stdout.

To override MongoDB connection details at runtime, pass system properties:

```bash
./gradlew run -Dmongo.host=myhost -Dmongo.port=27017 -Dmongo.dbName=mydb
```

---

## Running Tests

**Unit tests** (no MongoDB required):

```bash
./gradlew test
```

Tests are written in Spock (Groovy). Results in `build/reports/tests/test/index.html`.

**Functional (acceptance) tests** (require live MongoDB):

```bash
docker run -d --net=host mongo:latest   # if not already running
./gradlew funtest
```

The functional tests use Cucumber with Groovy step definitions. They start the Ratpack server internally via `ratpack.test.MainClassApplicationUnderTest` and seed/clean MongoDB before each scenario using `MongoHelper.groovy`.

Results in `build/reports/tests/funtest/index.html`.

**Run all checks (unit + functional):**

```bash
./gradlew check
```

---

## Key Concepts

**1. The broker never serves binary files.**
Every download endpoint issues a `302` redirect to the actual binary hosted on GitHub Releases or a CDN. The broker's job is solely to look up and validate the URL. Do not expect file content in any response.

**2. Three MongoDB collections are used.**

| Collection | Purpose |
|---|---|
| `versions` | Stores binary URLs per candidate + version + platform. Queried by `CandidateDownloadHandler`. |
| `audit` | Append-only log of every download event. Written by all download handlers. |
| `application` | Single-document config: health sentinel (`alive: "OK"`) and CLI version strings. |

**3. Platform strings are normalised by prefix matching.**
The client sends a raw platform string (e.g. `MINGW64_NT-10.0`, `darwinx64`). `Platform.of()` matches this case-insensitively against a list of known aliases for each platform enum value. A string of `linuxx64` and `Linux64` and `Linux` all resolve to `LINUX_64`.

**4. Ratpack uses non-blocking I/O — blocking DB calls must use `Blocking.get` or `Blocking.exec`.**
All MongoDB calls are wrapped in `Blocking.get` (for calls that return a value) or `Blocking.exec` (for fire-and-forget). If you add a new DB call, do not call it directly from a handler — always wrap it. Calling blocking code outside `Blocking` will cause a runtime error.

**5. The `application` collection drives both health and CLI version responses.**
The `MongoHealthCheck` passes only if `{ alive: "OK" }` exists in `application`. The same collection's first document also contains the CLI version fields. If you run the service against a fresh, empty MongoDB, the health check will return `503` and version lookups will return empty.

---

## Common Gotchas

| Gotcha | Detail |
|---|---|
| Health check returns 503 on fresh MongoDB | The `application` collection must be seeded manually or by running the functional tests. See `src/funtest/groovy/support/MongoHelper.groovy` for the required document structure. |
| `./gradlew check` runs functional tests by default | `check` depends on `funtest`. If MongoDB is not running, the build will fail. Use `./gradlew test` for unit tests only. |
| Config uses `/mongo` key prefix | Environment variables for MongoDB must be prefixed: `MONGO_HOST`, `MONGO_PORT`, `MONGO_DB_NAME`. Ratpack maps these via its env config source. |
| `BinaryDownloadConfig` fields are null until Ratpack injects them | `BinaryDownloadConfig` and `NativeBinaryDownloadConfig` declare fields as `final = null`. If `binary.properties` or `native_binary.properties` is missing or malformed, handlers will throw `NullPointerException` at request time, not at startup. |
| Functional tests require `--net=host` MongoDB | The test infrastructure connects to `localhost:27017`. The Docker `--net=host` flag is required on Linux. On macOS, `localhost` resolves correctly without it. |
