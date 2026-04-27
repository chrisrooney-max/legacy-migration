# Architecture

## Overview

The `.claude/skills` layer is a **flat collection of Markdown configuration files** — not executable code. It does not have a runtime architecture in the traditional sense. Instead, it acts as a declarative command registry: each `.md` file defines a slash command that Claude Code interprets, and the file's content is the prompt template executed when the command is invoked. Heavy processing is delegated to subagents defined in `.claude/agents/`. The architectural style is **orchestrator-with-specialist-agents**: skills orchestrate one or more agents, agents do the actual analysis and writing.

## Context Diagram

```
  User (Claude Code CLI)
          |
          | invokes /skill-name $ARGUMENTS
          v
  .claude/skills/<skill>.md   <-- prompt template, routing logic
          |
     +---------+------------------+-------------------+
     |                            |                   |
     v                            v                   v
code-documenter            migration-advisor      doc-auditor / epic-creator
  (.claude/agents/)         (.claude/agents/)      (.claude/agents/)
          |                       |
          v                       v
  .claude/skills/             context/
   templates/ (12 files)    migration-inputs.md
          |
          v
  documentationV{n}/   <-- versioned output directory
          |
          v
  GitHub Issues (gh CLI)   <-- for /create-epics only
```

## Components

| Component | Responsibility | Location | Notes |
|---|---|---|---|
| Skill files | Define slash commands; specify agent to spawn, output path, and section-level guidance | `.claude/skills/*.md` | 18 files; YAML frontmatter provides `name` and `description` |
| Templates | Define required section structure for each of the 10 doc types, epics, and rewrite scoring | `.claude/skills/templates/` | 12 files; consumed by skills and agents at runtime |
| `analyse.md` | Full pipeline orchestrator: spawns `code-documenter` then `migration-advisor` sequentially | `.claude/skills/analyse.md` | Step 2 is conditional on Step 1 completing |
| `assess-migration.md` | Standalone migration scoring skill; outputs to `output/rewrite-vs-refactor.md` | `.claude/skills/assess-migration.md` | Delegates to `rewrite-vs-refactor.md` template |
| `create-epics.md` | Derives GitHub epics from docs or codebase; invokes `gh issue create` | `.claude/skills/create-epics.md` | Requires `gh` CLI authenticated |
| `audit.md` | Spawns `doc-auditor` agent; verifies completeness and consistency | `.claude/skills/audit.md` | Default target: most recently created `documentationV*/` |
| `code-documenter` agent | Analyses a codebase; writes all 10 documents using templates | `.claude/agents/code-documenter.md` | Tools: Read, Grep, Glob, Write, Bash, WebFetch |
| `migration-advisor` agent | Reads financial inputs + codebase; produces financial decision report | `.claude/agents/migration-advisor.md` | Reads `context/migration-inputs.md` |
| `epic-creator` agent | Creates GitHub issues as labelled epics from docs or codebase | `.claude/agents/epic-creator.md` | Tools: Read, Grep, Glob, Bash, Write |
| `doc-auditor` agent | Audits documentation completeness; writes `audit-report.md` | `.claude/agents/doc-auditor.md` | Hardcoded default path `documentationV3/` — see Known Issues |
| `rewrite-vs-refactor.md` template | Weighted scoring framework across 10 migration dimensions | `.claude/skills/templates/rewrite-vs-refactor.md` | Score definitions, sensitivity analysis scaffold |
| `epic.md` template | Standard structure for GitHub epic issues | `.claude/skills/templates/epic.md` | Used by `epic-creator` agent |

## Interaction Patterns

- **Synchronous / sequential** — `analyse.md` explicitly blocks Step 2 until Step 1 completes; all skills invoke agents in a request-response pattern.
- **Prompt-driven / declarative** — no message queues, no events; all coordination is expressed in natural language within Markdown files.
- **CLI-mediated external calls** — `create-epics.md` and `epic-creator` agent use `gh issue create` via Bash; this is the only external I/O besides reading the target codebase.

## Data Flow

1. User invokes a slash command, e.g. `/analyse /path/to/legacy-codebase`.
2. Claude Code loads the matching skill file (e.g. `analyse.md`) and interprets its content as the task prompt, substituting `$ARGUMENTS` with the provided path.
3. The skill spawns the `code-documenter` agent with the codebase path and the output directory derived from `context/migration-inputs.md` or the next available `documentationV{n}/` version.
4. `code-documenter` reads templates from `.claude/skills/templates/`, analyses the target codebase, and writes 10 documents to the versioned output directory.
5. On completion, `analyse.md` spawns the `migration-advisor` agent, which reads `context/migration-inputs.md` and the codebase, then writes `rewrite-vs-refactor.md` to the same output directory.
6. For `/create-epics`, the `epic-creator` agent reads either output docs or the codebase directly, then calls `gh issue create` for each epic using the `epic.md` template as body structure.
7. For `/audit`, the `doc-auditor` agent reads the documentation directory and writes `audit-report.md` into it.

## External Dependencies

- **Claude Code CLI** — runtime that interprets skill files and manages agent spawning; no version pinned in the skills layer.
- **`gh` CLI (GitHub CLI)** — required for `/create-epics`; authenticated access to `chrisrooney-max/legacy-migration` assumed.
- **File system access** — skills assume read access to the target legacy codebase path passed as `$ARGUMENTS`.
- **`context/migration-inputs.md`** — financial input file; must be populated before `migration-advisor` will proceed.

## Deployment Architecture

- **Environments:** Not applicable — the skills layer is configuration, not a deployed service.
- **Hosting:** Runs inside the Claude Code CLI process on the developer's local machine.
- **Regions:** Not applicable.

> ⚠️ Unclear: There is no CI/CD configuration, Dockerfile, or infrastructure definition for the skills layer itself. It is entirely local-execution tooling.

## Key Constraints

- Skills are interpreted only by Claude Code; they cannot be executed by any other runtime.
- The `$ARGUMENTS` substitution is a Claude Code convention — skills are not shell scripts and cannot be tested with standard tooling.
- `migration-advisor` will refuse to run if `context/migration-inputs.md` fields are blank or "Unknown".
- `create-epics` requires `gh` CLI to be authenticated; the skill does not handle unauthenticated states beyond what `gh` CLI returns.

## Known Issues

- `doc-auditor` agent hardcodes `documentationV3/` as its read target in the agent system prompt body, which conflicts with the `audit.md` skill description that promises "most recently created `documentationV*/` directory". Running `/audit` against V4 or V5 without an explicit argument argument will audit V3 silently. See [change-risk](../change-risk/change-risk-spec.md).
- Output file naming is inconsistent: skills write to `<type>/index.md` but CLAUDE.md convention specifies `<type>-spec.md`. The `doc-auditor` agent reads `{folder-name}-spec.md` filenames, so documents written by individual skills to `index.md` would not be found by the auditor.
