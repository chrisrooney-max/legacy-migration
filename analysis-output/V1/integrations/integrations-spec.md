# Integrations

## Overview

The `.claude/skills` layer has **one outbound integration**: the GitHub API accessed via the `gh` CLI, used exclusively by the `/create-epics` skill and the `epic-creator` agent. There are no inbound integrations, no message queues, no REST client code, and no database connections. All other interactions are local file system reads and writes against the target legacy codebase path provided as `$ARGUMENTS`.

## Integration Catalogue

| System | Direction | Method | Purpose | Criticality |
|---|---|---|---|---|
| GitHub API (via `gh` CLI) | Outbound | CLI (`gh issue create`, `gh label create`, `gh issue list`) | Create and list epic issues in `chrisrooney-max/legacy-migration` | Medium — only required for `/create-epics`; documentation and analysis runs do not require it |
| Target legacy codebase (file system) | Inbound (read) | File system (Read, Grep, Glob, Bash) | Source material for all analysis and documentation | High — without access to the target codebase, no analysis can be performed |
| `context/migration-inputs.md` (file system) | Inbound (read) | File system (Read) | Financial and team inputs for `migration-advisor` | Medium — required only for `/analyse` and `/assess-migration`; individual document skills do not need it |

## APIs

- **GitHub API**
  - Endpoint: Accessed indirectly via `gh` CLI; underlying base URL is `https://api.github.com`.
  - Auth: `gh` CLI authentication (`gh auth login` or `GITHUB_TOKEN` environment variable); no auth logic exists within the skill files themselves.
  - Contract: `gh issue create`, `gh issue list --label epic`, `gh label create` — standard GitHub Issues API v3 via CLI wrapper.

> ⚠️ Unclear: No explicit documentation of required `gh` CLI version or GitHub token scopes exists within the skills layer. The `create-epics.md` skill assumes `repo` scope (create issues, create labels) but does not state this.

## Events / Messaging

- No event-driven or messaging architecture is present. There are no topics, queues, producers, or consumers.

## Error Handling

- **Retry strategy:** None defined in skill files. No retry logic is present for the `gh` CLI calls.
- **Failure behaviour for `gh` commands:** `create-epics.md` includes `2>/dev/null || true` on the `gh label create` command to suppress errors if the label already exists. No other explicit error handling is present for `gh issue create` failures.
- **Failure behaviour for file system reads:** If the target codebase path is inaccessible, the agent will report an error via its natural language output; no structured error handling is defined in the skill files.

## Dependencies

- **Upstream (this system depends on):**
  - `gh` CLI — authenticated and on `$PATH`; required for `/create-epics`.
  - File system access to target legacy codebase — required for all analysis skills.
  - `context/migration-inputs.md` populated — required for `migration-advisor` to proceed.
- **Downstream (systems that depend on this system):**
  - GitHub repository `chrisrooney-max/legacy-migration` — receives epic issues created by `/create-epics`.
  - Human users — consume generated documentation directories (`documentationV{n}/`).

## Failure Modes

- **`gh` CLI unavailable or unauthenticated:** `/create-epics` will fail at the issue-creation step. The skill does not include a pre-flight check for `gh auth status`. Documentation and analysis runs are unaffected.
- **Target codebase path inaccessible:** All analysis and documentation skills will fail to produce meaningful output; the agent will report inability to read files.
- **`context/migration-inputs.md` missing or unpopulated:** `migration-advisor` explicitly refuses to proceed, displaying a message to the user to complete the file. The `code-documenter` phase of `/analyse` is unaffected.
- **GitHub API rate limit or network failure:** No retry or fallback behaviour is defined; `gh` CLI's own error handling applies.

## Known Issues

- **No deduplication guard beyond label check:** `create-epics.md` instructs checking `gh issue list --label epic` before creating, but the matching logic relies on the agent's natural language interpretation — there is no exact-title deduplication enforced programmatically.
- **`gh label create` suppresses all errors:** The `2>/dev/null || true` pattern hides not just "label exists" errors but also permission errors and network failures, making silent failures possible during label creation.
- **No contract/spec for target codebase:** The skills assume the target codebase is a software repository accessible on the local file system, but there is no validation of this assumption. Passing a non-codebase path as `$ARGUMENTS` will produce incorrect output without an explicit error.
