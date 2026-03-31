# Operations

## Running Locally

**1. Start MongoDB**
```bash
docker run -d --net=host mongo:latest
```

**2. Seed the application collection** (required for health check and CLI version endpoints)

Run the functional tests — they seed MongoDB automatically:
```bash
./gradlew funtest
```

Or seed manually using the document shape in `data-model/index.md`.

**3. Start the application**
```bash
./gradlew run
```

Service starts on port `5050`. Verify:
```bash
curl http://localhost:5050/health/alive
curl http://localhost:5050/version
```

## Build & Deploy

**Build steps:**
```bash
./gradlew build        # compiles, runs unit + functional tests, produces fat JAR
./gradlew build -x funtest  # skip functional tests (no live MongoDB needed)
```

Output: `build/libs/sdkman-broker-*-all.jar`

**Deployment process:**
```bash
docker build -t sdkman-broker .
docker run -p 5050:5050 sdkman-broker
```

The `Dockerfile` copies the fat JAR and runs it with `-Xmx128m`. MongoDB connection must be provided via environment variables.

## Environments

| Environment | Purpose | Notes |
|---|---|---|
| Local | Development and testing | Requires Docker MongoDB; seed via functests or manually |
| Production | Live service | Not defined in codebase |

> ⚠️ Unclear: Staging, CI, and production environment configs are not present in the repository. Environment-specific config (hostnames, credentials) must be supplied at runtime via env vars.

## Configuration

| Variable | Default | Required | Description |
|---|---|---|---|
| `MONGO_HOST` | `localhost` | No | MongoDB host |
| `MONGO_PORT` | `27017` | No | MongoDB port |
| `MONGO_DB_NAME` | `sdkman` | No | MongoDB database name |
| `MONGO_USERNAME` | none | No | MongoDB auth username; omit for unauthenticated |
| `MONGO_PASSWORD` | none | No | MongoDB auth password |

Binary URL templates (`binary.properties`, `native_binary.properties`) are bundled in the JAR and point to GitHub Releases. They cannot be overridden at runtime without rebuilding.

**Secrets:** MongoDB credentials only. No secrets are committed to the repository.

## Monitoring
- **Health endpoint:** `GET /health/alive` — returns `200` healthy or `503` unhealthy based on MongoDB `application` collection
- **No metrics framework** — no Prometheus, Micrometer, or similar instrumentation is present
- **No distributed tracing** — no trace headers or tracing library observed

> ⚠️ Unclear: Production monitoring setup (dashboards, alerting, on-call rotation) is not defined in the codebase.

## Logging
- **Library:** SLF4J Simple (`slf4j-simple`)
- **Output:** stdout
- **Format:** plain text (SLF4J Simple default format)
- **Log level:** not explicitly configured; defaults to SLF4J Simple default (INFO)
- **What is logged:** handler receives download request (INFO), audit entry written (DEBUG), MongoDB client initialisation (INFO)
- **No structured logging** — logs are not JSON; cannot be easily parsed by log aggregation tools

> ⚠️ Note: `slf4j-simple` appears twice in `build.gradle` — at version `1.7.5` (implementation) and `2.0.17` (runtimeOnly). This will produce a SLF4J binding conflict warning at startup.

## Jobs / Schedulers
None. The service is purely request-driven with no background jobs, scheduled tasks, or polling.

## Common Issues

| Issue | Cause | Resolution |
|---|---|---|
| `GET /health/alive` returns `503` | MongoDB not running or `application` collection not seeded | Start MongoDB and seed the `application` collection |
| `./gradlew check` fails with connection errors | `funtest` requires live MongoDB | Run `docker run -d --net=host mongo:latest` first |
| SLF4J binding conflict warning at startup | Duplicate `slf4j-simple` in `build.gradle` | Warning only; service functions correctly |
| `NullPointerException` in `BinaryDownloadHandler` | Missing `binary.properties` entry | Verify all 4 binary config properties are present |
| Port conflict on `5050` | Another service using the port | Change Ratpack port via system property `-Dratpack.port=<port>` |

## Backup & Recovery
> ⚠️ Unclear: No backup scripts, MongoDB dump procedures, or recovery runbooks are present in this codebase. The `versions` and `application` collections are managed by external platform tooling; their backup strategy is not visible here. The `audit` collection is append-only and its retention/backup policy is not defined.
