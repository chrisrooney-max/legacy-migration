# Module: binary

> **Package:** `io.sdkman.broker.binary` (Java) + `io.sdkman.broker.binary` (Kotlin) | **Files:** `BinaryDownloadHandler.java`, `BinaryDownloadConfig.java`, `RequestDetails.java`, `BinaryVersionHandler.kt` | **Languages:** Java, Kotlin

Handles download and version lookup requests for the SDKMAN! bash CLI itself (as opposed to third-party SDK candidates). Download requests redirect to GitHub Releases using a URL template from config. Version lookups return the current stable or beta CLI version string from MongoDB.

---

## Responsibilities

- Validate download request path tokens (command, version, platform)
- Build a GitHub Releases redirect URL from config template for bash CLI downloads
- Handle `latest`-prefixed version strings (redirects to the `latest` release path)
- Return the current CLI version string for a given channel (`stable` / `beta`) and implementation (`bash` / `native`)
- Record an audit entry on every download request

---

## Public Interface

| Class | Method | Signature | Description |
|---|---|---|---|
| `BinaryDownloadHandler` | `handle` | `void handle(Context ctx)` | Validates request, audits, and redirects to GitHub Releases URL |
| `BinaryVersionHandler` | `handle` | `override fun handle(ctx: Context)` | Reads `channel` and `impl` path tokens; returns version string from MongoDB |
| `RequestDetails` | `of` | `static Optional<RequestDetails> of(Context ctx)` | Extracts and validates command, version, platform, host, and agent from path tokens and headers |

---

## Dependencies

**Internal:**
| Dependency | Purpose |
|---|---|
| `io.sdkman.broker.audit.AuditRepo` | Writes audit record on download |
| `io.sdkman.broker.audit.AuditEntry` | Audit record model |
| `io.sdkman.broker.download.Platform` | Infers canonical platform ID from raw platform string |
| `io.sdkman.broker.app.AppRepo` | Fetches CLI version string from MongoDB `application` collection |

**External:**
| Library | Version | Purpose |
|---|---|---|
| Ratpack core | 1.9.0 | `Handler`, `Context`, `PathTokens` |
| Arrow | 1.1.5 | `Option` return type from `AppRepo.findVersion` |

---

## Inputs and Outputs

**`BinaryDownloadHandler` input:** HTTP `GET /download/sdkman/:command/:version/:platform`

Path tokens:
- `command` — CLI command (e.g. `install`)
- `version` — CLI version string; prefix `latest` triggers the latest-release redirect path
- `platform` — platform string

Headers read: `X-Real-IP`, `user-agent`

**`BinaryDownloadHandler` output:** HTTP `302` redirect to:
```
{protocol}://{host}{uri}{name}
```
Template resolved from `binary.properties`:
```
binary.protocol = https
binary.host     = github.com
binary.uri      = /sdkman/sdkman-cli/releases/download/%s/
binary.name     = sdkman-cli-%s.zip
```
For `latest` versions: `%s` placeholders receive `"latest"` and the full version string respectively.

**`BinaryVersionHandler` input:** HTTP `GET /version/sdkman/:impl/:channel`

Path tokens:
- `impl` — `bash` or `native`; defaults to `bash` if absent
- `channel` — `stable` or `beta`

**`BinaryVersionHandler` output:** Plain text version string (e.g. `5.18.2`) rendered directly by Ratpack.

**Side effects:**
- `BinaryDownloadHandler` inserts a row into the MongoDB `audit` collection

---

## Known Issues / Technical Debt

> ⚠️ **Unclear:** `BinaryDownloadHandler.handle` formats the redirect URL with `String.format(prepareRemoteBinaryUrl(), "latest", details.getVersion())` for `latest`-prefixed versions. It is not documented what the `details.getVersion()` value looks like in the `latest` case — whether it contains just `"latest"` or a suffix like `"latest+stable"`.

- `BinaryDownloadConfig` declares all fields as `private final` with `= null`. Ratpack populates them reflectively. A missing property silently produces a `NullPointerException` at request time, not at startup.
- `BinaryVersionHandler` uses a `when` expression on `channel` that has a redundant branch — `null -> clientError(400)` is followed by `channel -> { ... }`, but the second branch catches all non-null values including those that are not `stable` or `beta`. The version lookup in `AppRepo` handles the fallback, but no validation of channel values occurs in the handler.
