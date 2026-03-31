# System Overview

## Purpose
- SDKMAN! Broker is an HTTP redirect service that sits between the SDKMAN! CLI and remote binary hosts
- It resolves the correct download URL for a given SDK candidate, version, and platform by querying MongoDB, then issues a 302 redirect — it does not serve binaries directly
- It also serves SDKMAN! CLI binary version lookups and self-update download redirects

## Key Capabilities
- Resolve and redirect SDK candidate downloads (Java, Groovy, Kotlin, etc.) by platform
- Redirect SDKMAN! bash CLI binary downloads to GitHub Releases
- Redirect SDKMAN! native (Rust) CLI binary downloads to GitHub Releases by Rust target triple
- Return the current stable/beta CLI version string by implementation type (bash/native)
- Report service health via a MongoDB-backed health check endpoint
- Record an audit entry (command, candidate, version, platform, host, agent) for every download

## Primary Users
- **SDKMAN! CLI** — the automated client; makes all download and version requests
- **SDKMAN! platform operators** — monitor health and audit data; manage MongoDB content

## Business Criticality
- [x] Important
- [ ] Mission critical
- [ ] Supporting

> ⚠️ Unclear: The service is a core part of the SDKMAN! download infrastructure but its exact SLA and on-call requirements are not defined in the codebase.

## Current State
- **Stability:** Stable — small, focused codebase with good functional test coverage of the happy path
- **Known issues:** Silent audit failures; deprecated endpoint still active; outdated MongoDB driver (3.2.2); Scala model dependency via JitPack
- **General health:** Low complexity, low churn. Main risk areas are dependency age and lack of unit tests on handler classes

## High-Level Risks
- `mongo-java-driver` 3.2.2 is significantly outdated and may carry known CVEs
- `sdkman-persistent-model` (Scala) sourced from JitPack introduces Scala interop complexity and an unreliable dependency source
- Audit writes are fire-and-forget — download events can be silently lost if MongoDB is unavailable
- Deprecated `/download/sdkman/version/:channel` endpoint still active with no removal timeline

## Ownership
- **Team:** sdkman (GitHub org)
- **Tech Lead:** Marco Vermeulen (Dockerfile MAINTAINER)
- **Product Owner:** not defined in codebase

## Related Systems
- **SDKMAN! CLI** (`sdkman/sdkman-cli`) — the bash client that calls this service
- **sdkman-cli-native** (`sdkman/sdkman-cli-native`) — the Rust CLI whose binaries are served via this broker
- **GitHub Releases** — hosts all bash and native CLI binaries; candidate binaries may be hosted here or on third-party CDNs
- **MongoDB** — stores version catalogue, audit log, and application config
