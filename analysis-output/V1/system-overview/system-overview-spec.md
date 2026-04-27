# System Overview

## Purpose

The `.claude/skills` directory is the **invocable skill layer** of the legacy-migration documentation agent. It provides a library of slash commands (skills) that Claude Code users invoke to trigger automated analysis workflows against external legacy codebases. The skills coordinate subagents, enforce template-based output structures, and produce versioned documentation directories. The problem it solves is the manual effort and inconsistency involved in understanding, documenting, and deciding how to migrate legacy software systems.

## Key Capabilities

- **Full documentation pipeline** (`/analyse`) — spawns the `code-documenter` agent for all 10 doc types, then automatically chains to the `migration-advisor` agent for a financial recommendation.
- **Individual document generation** — 10 granular skills (`/document-system-overview`, `/document-architecture`, etc.) each produce a single document from a template, allowing targeted re-runs without reprocessing the full suite.
- **Migration scoring** (`/assess-migration`) — scores a codebase across 10 weighted dimensions using the `rewrite-vs-refactor.md` template and outputs a decision recommendation.
- **Epic creation** (`/create-epics`) — derives GitHub issues as labelled epics from either output documentation or direct codebase analysis, with dependency ordering and deduplication checks.
- **Documentation audit** (`/audit`) — spawns the `doc-auditor` agent to verify completeness, detect stub sections, and report cross-document inconsistencies.
- **Codebase summarisation** (`/summarise-codebase`) — produces a quick architectural overview before a full documentation run.
- **Ancillary skills** — `/document-api`, `/document-module`, `/document-onboarding` for targeted extractions outside the 10 core doc types.

## Primary Users

- Software engineers and architects who need to understand a legacy codebase before migrating or modernising it.
- Engineering managers or tech leads making a rewrite vs. refactor decision who need a financial case with technical evidence.
- Delivery teams translating analysis output into GitHub epics for sprint planning.

## Business Criticality

- [ ] Mission critical
- [x] Important
- [ ] Supporting

The skills layer is the primary user-facing interface of the legacy-migration tooling. Without it, the subagents cannot be invoked in a structured way. It is not itself customer-facing, but it is the mechanism through which all analysis outputs are produced.

## Current State

- **Stability:** Stable as configuration artefacts (Markdown files with YAML frontmatter). No runtime failures are possible within the skill files themselves; failure risk lies in the subagents they spawn.
- **Known issues:** The `doc-auditor` agent (`audit.md`) hardcodes `documentationV3/` as the target directory in its system prompt, which contradicts its own skill description that says "most recently created `documentationV*/` directory". See [change-risk](../change-risk/change-risk-spec.md) for detail.
- **General health:** All 18 skill files are present and structurally consistent. Templates are complete for all 10 doc types plus `epic.md` and `rewrite-vs-refactor.md`.

## High-Level Risks

- **Hardcoded directory reference in `doc-auditor`** — the agent reads from `documentationV3/` by default, meaning audits against newer version directories (V4, V5) silently read the wrong source unless overridden by argument. See [change-risk](../change-risk/change-risk-spec.md).
- **No automated tests for skill correctness** — skill files are configuration, not executable code, so correctness depends entirely on human review and agent fidelity. See [testing](../testing/testing-spec.md).
- **Output path inconsistency** — individual skills write to `system-overview/index.md`, `architecture/index.md`, etc., but CLAUDE.md specifies `{folder-name}-spec.md` naming convention. The two naming conventions co-exist without enforcement. See [change-risk](../change-risk/change-risk-spec.md).

## Ownership

- **Team:** Not specified in any skill file, CODEOWNERS, or README within the skills directory.
- **Tech Lead:** Not specified.
- **Product Owner:** Not specified.

> ⚠️ Unclear: No CODEOWNERS file is present in the repository. Ownership cannot be confirmed from the codebase alone.

## Related Systems

- **`.claude/agents/`** — the subagents (`code-documenter`, `migration-advisor`, `epic-creator`, `doc-auditor`) that skills spawn via Claude's agent system.
- **`.claude/skills/templates/`** — the 12 Markdown template files that define the required output structure for every skill.
- **`context/migration-inputs.md`** — the financial and team-context input file consumed by the `migration-advisor` agent, referenced in `analyse.md` and `assess-migration.md`.
- **GitHub** (`chrisrooney-max/legacy-migration`) — the target repository for epic issues created by `/create-epics`.
- **External legacy codebases** — the systems that skills are pointed at as `$ARGUMENTS`; these are not part of this repository.
