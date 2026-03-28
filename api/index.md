# API Documentation — SDKMAN! Broker

> **Base URL:** not fixed — service is deployed behind a reverse proxy | **Protocol:** HTTP/1.1 | **Format:** plain text or JSON (endpoint-dependent)

No authentication is required on any endpoint. The service is public.

> ⚠️ **Unclear:** The actual production base URL is not present in the codebase. The README refers only to `localhost`. The `X-Real-IP` header used for auditing implies a reverse proxy sits in front.

---

## Authentication

None. All endpoints are unauthenticated.

---

## Endpoint Index

| Method | Path | Description |
|---|---|---|
| `GET` | `/health/:name?` | Health check — pass `alive` as `:name` |
| `GET` | `/version` | App name and version |
| `GET` | `/version/sdkman/:impl/:channel` | SDKMAN! CLI version string for a given implementation and channel |
| `GET` | `/download/sdkman/version/:channel` | **Deprecated** — SDKMAN! CLI version by channel (bash only) |
| `GET` | `/download/sdkman/:command/:version/:platform` | Download SDKMAN! bash CLI binary |
| `GET` | `/download/native/:command/:version/:platform` | Download SDKMAN! native (Rust) CLI binary |
| `GET` | `/download/:candidate/:version/:platform` | Download a candidate SDK binary by platform path token |
| `GET` | `/download/:candidate/:version` | Download a candidate SDK binary with `?platform=` query param |

---

## Endpoints

### GET /health/:name?

Reports service health. Pass `alive` as `:name` to check MongoDB connectivity.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| `name` | string (optional) | Health check name. Only `alive` is registered. |

**Response — 200 OK:** (healthy)
```
{"name":"alive","healthy":true}
```

**Response — 503 Service Unavailable:** (unhealthy — MongoDB unreachable or `application` collection not seeded)
```
{"name":"alive","healthy":false,"message":"Nothing found at application/alive in database."}
```

---

### GET /version

Returns the application name and current version.

**Response — 200 OK:**
```json
{
  "appName": "SDKMAN! Broker",
  "appVersion": "1.0.0-SNAPSHOT"
}
```

---

### GET /version/sdkman/:impl/:channel

Returns the current SDKMAN! CLI version string for a given implementation and release channel. Used by the CLI to determine whether an update is available.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| `impl` | string | `bash` or `native`. Defaults to `bash` if not a recognised value. |
| `channel` | string | `stable` or `beta`. Any other value is treated as `beta`. |

**Response — 200 OK:** Plain text version string.
```
5.18.2
```

**Error responses:**

| Status | Meaning |
|---|---|
| `400` | `:channel` path token is absent |

> ⚠️ **Unclear:** The response is rendered by Ratpack's default renderer for `Option<String>`. If the version field is absent in MongoDB, the response behaviour (empty body, 204, or 500) is not confirmed from the code.

---

### GET /download/sdkman/version/:channel *(deprecated)*

> ⚠️ Marked `//TODO: deprecated` in `Main.java`. Retained for backwards compatibility. Prefer `GET /version/sdkman/bash/:channel`.

Functionally equivalent to `GET /version/sdkman/bash/:channel`. Returns the bash CLI version for the given channel.

---

### GET /download/sdkman/:command/:version/:platform

Downloads the SDKMAN! bash CLI binary. Redirects to the appropriate GitHub Releases asset.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| `command` | string | CLI command (e.g. `install`). Included in the audit record; does not affect the redirect URL. |
| `version` | string | CLI version (e.g. `5.18.2`). Prefix `latest` triggers a redirect to the GitHub `latest` release. |
| `platform` | string | Platform string (see platform table below). Included in the audit record only; does not affect the redirect URL. |

**Response — 302 Found:** Redirect to:
```
https://github.com/sdkman/sdkman-cli/releases/download/{version}/sdkman-cli-{version}.zip
```
For `latest`-prefixed versions:
```
https://github.com/sdkman/sdkman-cli/releases/download/latest/sdkman-cli-{version}.zip
```

**Error responses:**

| Status | Meaning |
|---|---|
| `400` | One or more required path tokens are absent |

---

### GET /download/native/:command/:version/:platform

Downloads the SDKMAN! native (Rust-compiled) CLI binary. Redirects to the GitHub Releases asset for the resolved Rust target triple.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| `command` | string | CLI command (e.g. `install`). Audit only. |
| `version` | string | Native CLI version (e.g. `0.4.0`) |
| `platform` | string | Platform string — must resolve to a supported native target (see table below) |

**Response — 302 Found:** Redirect to:
```
https://github.com/sdkman/sdkman-cli-native/releases/download/v{version}/sdkman-cli-native-{version}-{triple}.zip
```

**Supported platforms for native download:**

| Platform string (examples) | Rust triple |
|---|---|
| `linuxx64`, `linux64`, `linux` | `x86_64-unknown-linux-gnu` |
| `linuxx32`, `linux32` | `i686-unknown-linux-gnu` |
| `linuxarm64` | `aarch64-unknown-linux-gnu` |
| `darwinx64`, `darwin` | `x86_64-apple-darwin` |
| `darwinarm64` | `aarch64-apple-darwin` |
| `mingw64`, `cygwin`, `msys` | `x86_64-pc-windows-msvc` |

**Error responses:**

| Status | Meaning |
|---|---|
| `400` | Required path tokens absent |
| `404` | Platform string unrecognised, or platform has no native binary (e.g. `FreeBSD`, `Windows 32-bit`) |

---

### GET /download/:candidate/:version/:platform
### GET /download/:candidate/:version?platform=

Downloads a third-party SDK candidate binary. Looks up the URL in MongoDB, resolves to the best platform match, and redirects.

**Path parameters:**

| Parameter | Type | Description |
|---|---|---|
| `candidate` | string | SDK name (e.g. `java`, `groovy`, `kotlin`) |
| `version` | string | Version string (e.g. `11.0.9-adpt`, `2.4.7`) |
| `platform` | string | Platform string — path token or `?platform=` query param |

**Response — 302 Found:** Redirect to the binary URL stored in MongoDB for the matching candidate, version, and platform.

**Response headers on 302:**

| Header | Description |
|---|---|
| `X-Sdkman-Checksum-<algo>` | One header per checksum algorithm available (e.g. `X-Sdkman-Checksum-SHA-256`). Ordered by algorithm priority (strongest first). |
| `X-Sdkman-ArchiveType` | `zip` or `tar` — inferred from the binary URL file extension |

**Platform resolution:**
1. Looks for a MongoDB entry matching the exact normalised platform name
2. Falls back to `UNIVERSAL` if no platform-specific entry exists
3. Returns `404` if neither is found

**Error responses:**

| Status | Meaning |
|---|---|
| `400` | Required path tokens absent (candidate, version, and platform all required) |
| `404` | Platform string unrecognised, or no matching version entry in MongoDB |
