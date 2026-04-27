# Security

## Overview

The `.claude/skills` layer presents a **low but non-trivial security surface**. It is a local CLI tool with no network-exposed endpoints, no user authentication layer, and no database. The primary security considerations are: the forwarding of a `GITHUB_TOKEN` or `gh` CLI credential to create GitHub issues; the possibility that financial data in `context/migration-inputs.md` could be committed to a public repository; and the risk that an adversarial or malformed target codebase path could cause unintended file system reads. There is no authentication, authorisation, or encryption managed within the skills layer itself.

## Authentication

- **Method:** None within the skills layer. The `gh` CLI handles GitHub authentication externally (OAuth token via `gh auth login` or `GITHUB_TOKEN` environment variable).
- **Flow:** Not applicable — skills are invoked locally by an authenticated developer; there is no login flow, no session management, and no token issuance within this system.

## Authorization

- **Roles:** None defined. Any user with access to the local machine and Claude Code CLI can invoke any skill.
- **Permissions:** The GitHub operations (`/create-epics`) require `repo` scope on the GitHub token; this is not validated or documented within the skill files. Read operations on the target codebase require local file system read permissions — these are also not validated.

> ⚠️ Unclear: No access control, role separation, or permission model is implemented or described for the skills layer.

## Sensitive Data

- **Financial data** — `context/migration-inputs.md` contains maintenance costs, engineering costs, call center costs, and revenue figures. If this file is committed to a public GitHub repository, commercial financial data is exposed.
- **GitHub credentials** — the `gh` CLI stores an OAuth token locally (typically in `~/.config/gh/hosts.yml`). This is outside the repository but is a dependency of the `/create-epics` skill.
- **Target codebase contents** — skills read arbitrary files from the target codebase path. If the target codebase contains secrets, API keys, or PII, those contents pass through the Claude Code model context. There is no filtering or redaction.

## Data Protection

- **Encryption:** None managed by the skills layer. File system encryption and token storage are the responsibility of the OS and `gh` CLI respectively.
- **Masking:** No log masking or redaction is implemented. Financial figures in `context/migration-inputs.md` appear verbatim in agent output and could appear in Claude Code session logs.
- **In transit:** All GitHub API calls made by `gh` CLI use HTTPS. No HTTP-only calls are present in the skill files.

## Audit Logging

- **What is logged:** No audit logging is implemented within the skills layer. Claude Code may log session history depending on its configuration, but this is outside the skills layer's control.
- Generated documents written to `documentationV{n}/` serve as an implicit record of what was analysed and when, but they contain no timestamps or operator identity.

## Vulnerabilities

- **Financial data in a potentially public repository** — `context/migration-inputs.md` is tracked by Git in `chrisrooney-max/legacy-migration`. If the repository is public (or becomes public), commercially sensitive figures are exposed. No `.gitignore` entry excludes this file.
- **No input sanitisation on `$ARGUMENTS`** — the target codebase path passed as `$ARGUMENTS` is used directly in file system read commands (Read, Grep, Glob, Bash). A path traversal attack is theoretically possible if an untrusted user can control the `$ARGUMENTS` value, though in practice invocation is local and interactive.
- **`2>/dev/null || true` in `create-epics.md`** — suppresses all errors from `gh label create`, including authentication failures that might indicate a compromised or expired token.

> ⚠️ Unclear: The repository visibility (public vs. private) of `chrisrooney-max/legacy-migration` cannot be confirmed from the skill files alone. If the repository is public, `context/migration-inputs.md` containing real financial data is a significant exposure.

## Dependencies Risk

- **`gh` CLI** — no version is pinned. CVEs in `gh` CLI would affect the `/create-epics` skill. No dependency scanning is configured.
- **Claude Code CLI** — no version is pinned. Model and runtime updates could change behaviour without notice; no lock file exists.
- **No `package.json`, `requirements.txt`, or `go.mod`** — there are no code dependencies to audit with standard security tooling (e.g. `npm audit`, `pip-audit`, `govulncheck`).

## Compliance

- **GDPR:** Not directly applicable — the skills layer does not process personal data in its own operation. However, if a target legacy codebase analysed via these skills contains PII, that data passes through the Claude Code model context. No data retention policy, deletion mechanism, or consent flow is present.
- **Financial data handling:** No compliance controls exist for the financial figures in `context/migration-inputs.md`. Storage in a Git repository without encryption may conflict with internal data classification policies depending on the organisation using this tool.
- No HIPAA, SOC 2, or other compliance indicators are present in the codebase.

## Recommendations

1. **Add `context/migration-inputs.md` to `.gitignore`** — prevent accidental commitment of financial figures to the repository, especially if the repository could ever be made public.
2. **Document the required `gh` CLI token scope** in `create-epics.md` and `README.md` — engineers should know they need `repo` scope, not just any valid token.
3. **Replace `2>/dev/null || true` on `gh label create`** with an explicit exit-code check so authentication failures are surfaced rather than silently discarded. See [change-risk](../change-risk/change-risk-spec.md).
4. **Add a repository visibility note** in CLAUDE.md advising users not to store real financial data in `migration-inputs.md` if the repository is or could become public.
