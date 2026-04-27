# Code Structure

## Repository Overview

- **Repo name:** `chrisrooney-max/legacy-migration`
- **Purpose:** Legacy migration documentation agent — a scaffold of slash commands (skills), subagent definitions, and output templates for automatically generating structured documentation and migration analysis of external legacy codebases.

## Directory Structure

```
.claude/skills/
├── README.md                        # Explains skill file structure and frontmatter convention
├── analyse.md                       # /analyse — full pipeline (doc + migration advisor)
├── assess-migration.md              # /assess-migration — migration scoring only
├── audit.md                         # /audit — doc completeness auditor
├── create-epics.md                  # /create-epics — GitHub epic issue creator
├── document-api.md                  # /document-api — extract API endpoints
├── document-architecture.md         # /document-architecture — architecture doc
├── document-business-rules.md       # /document-business-rules — business rules doc
├── document-change-risk.md          # /document-change-risk — change risk doc
├── document-code-structure.md       # /document-code-structure — code structure doc
├── document-data-model.md           # /document-data-model — data model doc
├── document-integrations.md         # /document-integrations — integrations doc
├── document-module.md               # /document-module — single module doc
├── document-onboarding.md           # /document-onboarding — getting started guide
├── document-operations.md           # /document-operations — operations doc
├── document-security.md             # /document-security — security doc
├── document-system-overview.md      # /document-system-overview — system overview doc
├── document-testing.md              # /document-testing — testing doc
├── summarise-codebase.md            # /summarise-codebase — quick architecture overview
└── templates/
    ├── .gitkeep
    ├── 01-system-overview.md        # Required sections for system overview output
    ├── 02-architecture.md           # Required sections for architecture output
    ├── 03-code-structure.md         # Required sections for code structure output
    ├── 04-data-model.md             # Required sections for data model output
    ├── 05-integrations.md           # Required sections for integrations output
    ├── 06-operations.md             # Required sections for operations output
    ├── 07-change-risk.md            # Required sections for change risk output
    ├── 08-testing.md                # Required sections for testing output
    ├── 09-security.md               # Required sections for security output
    ├── 10-business-rules.md         # Required sections for business rules output
    ├── epic.md                      # Required structure for GitHub epic issues
    └── rewrite-vs-refactor.md       # Weighted scoring framework for migration decisions
```

## Key Modules

| Module | Responsibility | Entry Points | Notes |
|---|---|---|---|
| `analyse.md` | Full analysis pipeline orchestrator | `/analyse $ARGUMENTS` | Two-step: `code-documenter` then `migration-advisor`; blocks between steps |
| `assess-migration.md` | Standalone migration scoring | `/assess-migration $ARGUMENTS` | Outputs to `output/rewrite-vs-refactor.md`; uses `rewrite-vs-refactor.md` template |
| `create-epics.md` | GitHub epic creation | `/create-epics $ARGUMENTS owner/repo` | Checks for existing `epic` label; deduplicates before creating |
| `audit.md` | Documentation quality auditing | `/audit $ARGUMENTS` | Spawns `doc-auditor` agent; writes `audit-report.md` |
| `summarise-codebase.md` | Quick architecture summary | `/summarise-codebase $ARGUMENTS` | Writes to `architecture/overview.md`; produces ASCII diagram and data flow |
| `document-*.md` (10 files) | Individual document generation | `/document-<type> $ARGUMENTS` | Each writes one document using the matching numbered template |
| `document-api.md` | API endpoint extraction | `/document-api $ARGUMENTS` | Writes to `api/index.md`; outside the core 10 doc types |
| `document-module.md` | Single module documentation | `/document-module $ARGUMENTS` | Writes to `modules/<module-name>.md`; updates `modules/index.md` |
| `document-onboarding.md` | Getting-started guide | `/document-onboarding $ARGUMENTS` | Writes to `onboarding/getting-started.md`; outside the core 10 doc types |
| `templates/` (12 files) | Canonical output structure definitions | Read by agents and individual skills | Templates are never executed directly |

## Entry Points

- **CLI (primary):** Every `.md` file with YAML frontmatter `name:` is a slash command entry point invoked as `/<name>` in Claude Code.
- **Agent entry (secondary):** `code-documenter`, `migration-advisor`, `epic-creator`, and `doc-auditor` agents are spawned programmatically by skills; their definitions live in `.claude/agents/`, not in this directory.
- **No API, no jobs, no HTTP entry points** — this is purely CLI-invoked tooling.

## Configuration

- **Where config lives:** `context/migration-inputs.md` — financial and team context inputs for the `migration-advisor`. The skills directory itself contains no `.env` files, no `application.properties`, and no environment variable declarations.
- **Environment variables:** None defined within the skills layer. The `gh` CLI used by `create-epics` requires `GITHUB_TOKEN` or an authenticated `gh auth login` session, but this is not explicitly documented within the skill files.

> ⚠️ Unclear: No explicit documentation of required environment variables exists anywhere in the skills layer. The `gh` CLI authentication requirement is implied but not stated.

## Coding Patterns

- **Prompt template pattern** — each skill file is a reusable prompt template; `$ARGUMENTS` is the single substitution variable injected at invocation time.
- **YAML frontmatter convention** — all skill files use `---`-delimited frontmatter with `name` and `description` fields, following Claude Code's slash command registration format.
- **Section guidance pattern** — each skill includes a `**Section guidance:**` block that maps template sections to evidence sources in the codebase; this is a consistent cross-cutting pattern.
- **Formatting rules block** — every skill ends with a `**Formatting rules:**` section enforcing `> ⚠️ Unclear:` usage and factual constraint.
- **Agent delegation pattern** — orchestrator skills (`analyse.md`, `audit.md`) spawn named agents and wait for completion before proceeding; individual document skills embed all logic inline.
- **Template-driven output** — all output structure is defined in `templates/`; skills reference templates by path rather than duplicating structure inline.

## Dependencies

- **Internal:** Skills depend on `templates/` (read at runtime by agents/skills); `analyse.md` depends on both `code-documenter` and `migration-advisor` agents; all skills implicitly depend on `context/migration-inputs.md` being populated for advisor runs.
- **External:** `gh` CLI (GitHub operations); Claude Code runtime (skill execution); file system access to target codebase paths.

## Areas of Concern

- **`doc-auditor` hardcoded path** — the `doc-auditor` agent (`.claude/agents/doc-auditor.md`) hardcodes `documentationV3/` in its own system prompt, diverging from the `audit.md` skill's claimed default behaviour. This is a fragile coupling between the skill and agent definitions. See [change-risk](../change-risk/change-risk-spec.md).
- **Dual output naming conventions** — individual skills write to `<type>/index.md` but CLAUDE.md and the `doc-auditor` agent expect `<type>-spec.md`. Documents written by `/document-system-overview` would not be found by `/audit` without manual path correction.
- **No version pinning** — neither the skills layer nor the agents reference a specific Claude model version or Claude Code CLI version. Behaviour may change silently across model upgrades.
- **`$ARGUMENTS` is singular** — `create-epics.md` requires two arguments (path and `owner/repo`), but the `$ARGUMENTS` substitution provides a single string; splitting is left to the agent's interpretation with no validation.
