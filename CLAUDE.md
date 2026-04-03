# CLAUDE.md

Instructions for Claude when working in this repository.

## Project Purpose

This is a **legacy migration documentation agent** — a scaffold for automatically generating structured documentation and migration analysis for legacy codebases. It is not itself a legacy system; it is the tooling used to analyse them.

## Repository Structure

| Path | Purpose |
|------|---------|
| `.claude/agents/` | Custom Claude subagents (code-documenter, migration-advisor, epic-creator) |
| `.claude/skills/` | User-invocable skills (`/assess-migration`, `/create-epics`, `/document-code`) |
| `.claude/skills/templates/` | Output templates for all 10 doc types, epic format, and rewrite-vs-refactor scoring |
| `output/` | v2 analysis output for sdkman-broker |
| `documentationV3/` | v3 analysis output for spring-projects/spring-framework |
| `examples/` | Example output documents |
| `architecture/`, `modules/`, `api/`, `onboarding/` | Legacy scaffold folders from initial setup |

## Documentation Framework

Analysis produces **10 documents** per codebase, stored in versioned directories (e.g. `documentationV3/`):

| # | Folder | File | Purpose |
|---|--------|------|---------|
| 1 | `system-overview/` | `system-overview-spec.md` | What the system does and why |
| 2 | `architecture/` | `architecture-spec.md` | Components, patterns, external dependencies |
| 3 | `code-structure/` | `code-structure-spec.md` | Repo layout, modules, entry points, coding patterns |
| 4 | `data-model/` | `data-model-spec.md` | Entities, storage, lifecycle, migration risks |
| 5 | `integrations/` | `integrations-spec.md` | External systems, APIs, messaging, failure modes |
| 6 | `operations/` | `operations-spec.md` | Build, deploy, CI, environments, monitoring |
| 7 | `change-risk/` | `change-risk-spec.md` | High-risk areas, technical debt, coupling |
| 8 | `testing/` | `testing-spec.md` | Test strategy, coverage, gaps, release validation |
| 9 | `security/` | `security-spec.md` | Auth, sensitive data, CVEs, compliance |
| 10 | `business-rules/` | `business-rules-spec.md` | Core rules, workflows, validations, edge cases |

## Naming Conventions

- Documentation files: `{folder-name}-spec.md` (e.g. `architecture-spec.md`)
- Output directories: `documentationV{n}/` for versioned analysis runs
- GitHub issues: titled `Epic: <title>`, labelled `epic`

## GitHub Repository

Target repo for issues and pushes: `chrisrooney-max/legacy-migration`

## Agents & Skills

- **`/document-code <path>`** — runs all 10 doc types against a codebase
- **`/assess-migration <path>`** — scores codebase on rewrite vs strangle & refactor (outputs to `output/rewrite-vs-refactor.md`)
- **`/create-epics <path> owner/repo`** — creates GitHub issues as epics from codebase or output docs

## Workflow Conventions

- Always push changes to GitHub after completing analysis or creating epics
- Check for existing epics (`gh issue list --label epic`) before creating new ones to avoid duplicates
- Use shallow clones (`git clone --depth 1`) for large external codebases analysed locally
- All output docs must be grounded in evidence from the codebase — do not invent scope
