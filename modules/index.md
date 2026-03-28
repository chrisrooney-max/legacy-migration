# Module Index — SDKMAN! Broker

| Module | File | Description |
|---|---|---|
| [download](./download.md) | `download/` package | Candidate SDK download handler — resolves platform-specific binary URL from MongoDB and redirects |
| [binary](./binary.md) | `binary/` package (Java + Kotlin) | SDKMAN! bash CLI download and version lookup — builds redirect URL from config template |
| [rust](./rust.md) | `rust/` package + `NativeTarget.java` | SDKMAN! native (Rust) CLI download — resolves platform to Rust target triple and redirects |
| [audit](./audit.md) | `audit/` package | Records download events to MongoDB `audit` collection |
| [db](./db.md) | `db/` package | MongoDB client provider and configuration |
| [app + health](./app.md) | `app/` + `health/` packages | MongoDB `application` collection repo; health check and CLI version lookup |
| [version](./version.md) | `version/` package | `GET /version` endpoint and MongoDB `versions` collection repo |
