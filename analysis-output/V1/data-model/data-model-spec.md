# Data Model

## Overview

The `.claude/skills` layer has no database, no ORM, and no schema. Its "data" consists entirely of **structured Markdown files** that act as both configuration (skill definitions) and output containers (templates and generated documentation). There is no persistent data store; all state is in the file system. The relevant data structures are the document templates (which define expected output shape) and the financial inputs file (`context/migration-inputs.md`), which is the only structured input consumed at runtime.

## Core Entities

| Entity | Description | Owner | Notes |
|---|---|---|---|
| Skill file | A Markdown file with YAML frontmatter that defines a slash command prompt | `.claude/skills/` directory | 18 files; `name` and `description` are the structured fields |
| Template file | A Markdown file defining required sections for one output document type | `.claude/skills/templates/` | 12 files; section headings (`##`) are the enforced schema |
| Migration inputs | Financial and team-context parameters read by the `migration-advisor` | `context/migration-inputs.md` | Fields: maintenance costs, engineering costs, call center costs, revenue, team capacity, risk tolerance |
| Generated document | A Markdown output file produced by an agent following a template | `documentationV{n}/` | 10 per analysis run; naming convention: `<type>-spec.md` per CLAUDE.md |
| Audit report | A Markdown file produced by `doc-auditor` summarising documentation completeness | `documentationV{n}/audit-report.md` | One per audit run |
| Epic issue | A GitHub issue with `epic` label, created by `epic-creator` using `epic.md` template | GitHub repository | Not stored locally; lives in GitHub |

## Relationships

- Each **skill file** references one or more **template files** by path (e.g. `document-system-overview.md` references `01-system-overview.md`).
- The **`analyse.md` skill** has a dependency chain: it triggers `code-documenter` which produces 10 **generated documents**, then triggers `migration-advisor` which reads those documents plus **migration inputs** to produce a **rewrite-vs-refactor report**.
- The **`doc-auditor`** agent reads all 10 **generated documents** and all 10 **template files** to produce one **audit report**.
- **Epic issues** are derived from **generated documents** (or directly from the target codebase) by the `epic-creator` agent.

## Data Storage

- **Database(s):** None.
- **Type:** File system only (Markdown files in a Git repository).
- **Git repository:** `chrisrooney-max/legacy-migration` — all skill files, templates, and versioned output directories are tracked in Git.

## Key Tables / Collections

> ⚠️ Unclear: There are no tables or collections in the traditional sense. The following represents the structured Markdown files that carry the system's key data.

| Name | Purpose | Key Fields | Risk |
|---|---|---|---|
| `context/migration-inputs.md` | Financial and team input for migration advisor | Maintenance costs, engineering costs, call center costs, revenue, team capacity, risk tolerance | If fields are blank/Unknown, `migration-advisor` refuses to run |
| `templates/rewrite-vs-refactor.md` | Scoring framework for migration decisions | 10 dimension rows, weight column, score column, rationale column | Changes to weights or score definitions silently alter all future migration reports |
| `templates/epic.md` | Structure for GitHub epic issues | Overview, Scope, Acceptance Criteria, Dependencies, Reference | All `create-epics` output depends on this structure |
| `templates/01–10-*.md` | Required section schemas for the 10 doc types | `##` section headings | Adding or removing a heading changes what `doc-auditor` flags as missing |

## Data Lifecycle

- **Creation:** Skill files and templates are created by developers maintaining this repository. Generated documents and audit reports are created at analysis runtime by agents writing to the file system.
- **Updates:** Skill files and templates are updated via Git commits. Generated documents are overwritten on each analysis run (no versioning within a single `documentationV{n}/` directory — re-running overwrites).
- **Deletion:** No deletion logic is present. Old `documentationV{n}/` directories accumulate; there is no cleanup script or retention policy.

## Data Quality Issues

- **No schema validation** — template compliance is enforced only by the agent's interpretation of a natural-language prompt; there is no programmatic validator.
- **Migration inputs have no defaults or type constraints** — the file is free-form Markdown; a numeric field like "revenue" could contain any text without triggering an error until the `migration-advisor` attempts to calculate ratios.
- **Versioned directory naming relies on convention** — the `analyse.md` skill instructs the agent to "use the next available version", but this determination is made by the agent through natural language reasoning, not a counter or registry file. Version collisions or gaps are possible.

## Reporting Dependencies

- **Audit report** (`audit-report.md`) — depends on the 10 generated documents existing at their expected `<type>-spec.md` paths within the target `documentationV{n}/` directory.
- **Migration advisor report** (`rewrite-vs-refactor.md`) — depends on `context/migration-inputs.md` being populated and on the target codebase being accessible at the path specified within that file.
- **GitHub epics** — depend on generated documents existing (preferred) or the target codebase being directly accessible.

## Migration Risks

- **Template changes break auditor** — the `doc-auditor` derives its list of required sections by reading the template `##` headings at runtime. Any change to a template immediately changes what the auditor considers complete or missing, with no version locking.
- **`rewrite-vs-refactor.md` weight changes** — the scoring weights are embedded directly in the template table. Changing a weight retroactively invalidates the scores of all previously generated reports without any indication.
- **No migration history** — there are no schema migration scripts, changelogs for templates, or version identifiers within template files. It is not possible to determine from the files themselves which version of a template was used to generate a given output document.
