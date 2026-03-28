# Module: rust

> **Package:** `io.sdkman.broker.rust` + `io.sdkman.broker.NativeTarget` | **Files:** `NativeBinaryDownloadHandler.java`, `NativeBinaryDownloadConfig.java`, `NativeTarget.java` | **Language:** Java

Handles download requests for the SDKMAN! native (Rust-compiled) CLI binary. Resolves the requested platform to a Rust target triple, builds a GitHub Releases redirect URL from config, and records an audit entry.

---

## Responsibilities

- Validate incoming download request path tokens (command, version, platform)
- Map the requested platform to a Rust target triple via `NativeTarget`
- Build a GitHub Releases redirect URL using a config URL template
- Return `404` for platforms with no native binary (e.g. `FreeBSD`, `Windows 32-bit`)
- Record an audit entry on every successful redirect

---

## Public Interface

| Class | Method | Signature | Description |
|---|---|---|---|
| `NativeBinaryDownloadHandler` | `handle` | `void handle(Context ctx)` | Validates request, resolves native target, audits, and redirects |
| `NativeTarget` | `of` | `static Optional<NativeTarget> of(Platform platform)` | Maps a `Platform` enum to a `NativeTarget`. Returns empty for unsupported platforms. |
| `NativeTarget` | `getTriple` | `String getTriple()` | Returns the Rust target triple string (e.g. `x86_64-apple-darwin`) |
| `NativeTarget` | `getPlatform` | `Platform getPlatform()` | Returns the associated `Platform` enum value |

---

## Dependencies

**Internal:**
| Dependency | Purpose |
|---|---|
| `io.sdkman.broker.audit.AuditRepo` | Writes audit record on download |
| `io.sdkman.broker.audit.AuditEntry` | Audit record model |
| `io.sdkman.broker.binary.RequestDetails` | Shared request extraction from path tokens and headers |
| `io.sdkman.broker.download.Platform` | Platform string normalisation |

**External:**
| Library | Version | Purpose |
|---|---|---|
| Ratpack core | 1.9.0 | `Handler`, `Context` |

---

## Inputs and Outputs

**Input:** HTTP `GET /download/native/:command/:version/:platform`

Path tokens:
- `command` — CLI command (e.g. `install`)
- `version` — CLI version string (e.g. `0.4.0`)
- `platform` — platform string (e.g. `darwinx64`, `linuxx64`)

Headers read: `X-Real-IP`, `user-agent`

**Output:** HTTP `302` redirect to:
```
{protocol}://{host}{uri}{name}
```
Template resolved from `native_binary.properties`:
```
native.protocol = https
native.host     = github.com
native.uri      = /sdkman/sdkman-cli-native/releases/download/v%s/
native.name     = sdkman-cli-native-%s-%s.zip
```
The three `%s` placeholders receive: `version`, `version`, `triple` (e.g. `x86_64-apple-darwin`).

**Side effects:**
- Inserts a row into the MongoDB `audit` collection on successful redirect

**Supported native targets:**

| NativeTarget | Rust Triple | Platform |
|---|---|---|
| `LINUX_32` | `i686-unknown-linux-gnu` | `LINUX_32` |
| `LINUX_64` | `x86_64-unknown-linux-gnu` | `LINUX_64` |
| `LINUX_ARM64` | `aarch64-unknown-linux-gnu` | `LINUX_ARM64` |
| `MAC_OSX` | `x86_64-apple-darwin` | `MAC_OSX` |
| `MAC_ARM64` | `aarch64-apple-darwin` | `MAC_ARM64` |
| `WINDOWS_64` | `x86_64-pc-windows-msvc` | `WINDOWS_64` |

All other platforms return `404`.

---

## Known Issues / Technical Debt

- `NativeBinaryDownloadConfig` declares all fields as `private final = null` — same null-initialisation issue as `BinaryDownloadConfig`. Missing config causes `NullPointerException` at request time.
- `NativeTarget.of` uses a `switch` statement rather than a map or stream. Adding a new platform requires editing two places: `Platform.java` and `NativeTarget.java`.
- The `rust` package name is a misnomer as an organisational choice — the package contains nothing Rust-specific beyond the word "native". Future developers may not look here for native binary handling.
