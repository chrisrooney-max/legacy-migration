# Business Rules

## Overview

The `.claude/skills` layer enforces a small set of procedural and structural rules expressed entirely in natural language within Markdown files. There is no rule engine, no domain model, and no conditional business logic in code. Rules govern: which template to use for each output document, how to route the output of one skill into the next, when to refuse to proceed, and how to handle unknown or ambiguous inputs. Complexity is low-to-medium — the rules are clear individually but their interactions (particularly around naming conventions and version resolution) contain inconsistencies.

## Core Rules

| Rule | Description | Location in Code | Notes |
|---|---|---|---|
| Template-driven output | Every document skill must use the matching numbered template as its exact output structure | All `document-*.md` skills; e.g. `document-system-overview.md` references `01-system-overview.md` | Enforced by natural language instruction only |
| Factual constraint | Skills must only document what can be observed in the code; invented scope is forbidden | Every skill file, Formatting rules section: "Be factual — do not invent details" | No programmatic enforcement |
| Unclear callout convention | Any ambiguous or unconfirmable claim must be flagged with `> ⚠️ Unclear:` | Every skill file, Formatting rules section | Consistent across all 18 skill files |
| `migration-advisor` precondition | The advisor must not proceed if `context/migration-inputs.md` fields are blank or "Unknown" | `.claude/agents/migration-advisor.md`: "stop and tell the user to complete the file" | Enforced by natural language instruction; no programmatic gate |
| Sequential pipeline in `/analyse` | Step 2 (migration advisor) must not start until Step 1 (documentation) completes successfully | `analyse.md`: "Do not proceed to Step 2 until all 10 documents have been written successfully" | Natural language gate only |
| Epic deduplication | Before creating epics, check `gh issue list --label epic` and do not create duplicates | `create-epics.md` and `epic-creator.md`: "Check for existing epics before creating new ones" | Enforced by agent's natural language interpretation; not exact-match programmatic |
| Maximum 15 epics | No more than 15 epic issues may be created per run; consolidate smaller areas if needed | `epic-creator.md`: "Do not create more than 15 epics" | Upper bound on scope decomposition |
| Output directory versioning | Output must go to a versioned `documentationV{n}/` directory; version is the next available integer | `analyse.md`: "use the next available version (e.g. if documentationV3/ exists, use documentationV4/)" | Version determination is agent-inferred, not computed |
| Time pressure scoring threshold | Cost-to-revenue ratio maps to a migration urgency score via five defined bands | `.claude/agents/migration-advisor.md`: table mapping ratio ranges (>50%, 26-50%, 11-25%, 5-10%, <5%) to scores 1–5 | Explicitly documented thresholds; only financial rule with numeric precision |
| Rewrite vs. refactor verdict | Final weighted score below 2.5 = Strangle & Refactor; 2.5–3.4 = Hybrid; 3.5+ = Rewrite | `templates/rewrite-vs-refactor.md`: Scoring Key table | Score = weighted sum ÷ 25 |
| Sensitivity analysis requirement | If any scoring dimension is unknown, a sensitivity analysis across scores 1/3/5 is mandatory | `assess-migration.md` and `migration-advisor.md`: "run a sensitivity analysis" | Required for unknowns; not optional |
| Evidence citation requirement | Every dimension score in the migration report must cite a specific file, class, or metric | `migration-advisor.md`: "Every technical score must cite a file, class, or metric from the codebase" | Cannot score without evidence |

## Workflows

**Full analysis pipeline (`/analyse`):**
1. User invokes `/analyse <codebase-path>`.
2. `code-documenter` agent is spawned with the codebase path and output directory.
3. Agent reads all 10 templates from `.claude/skills/templates/`.
4. Agent analyses the target codebase and writes 10 documents to `documentationV{n}/`.
5. Pipeline blocks until all 10 documents are confirmed written.
6. `migration-advisor` agent is spawned; reads `context/migration-inputs.md` and the codebase.
7. Advisor checks all financial fields are populated; refuses and exits if any are blank/Unknown.
8. Advisor scores 10 dimensions, calculates weighted score, produces financial case, writes `rewrite-vs-refactor.md`.
9. Skill reports output directory, verdict summary, and any flags to the user.

**Epic creation (`/create-epics`):**
1. User invokes `/create-epics <path> <owner/repo>`.
2. `epic-creator` agent checks for existing `epic`-labelled issues via `gh issue list`.
3. Agent reads output documentation (preferred) or analyses codebase directly.
4. Agent identifies functional areas, orders by dependency, caps at 15 epics.
5. Agent creates `epic` GitHub label if it does not exist.
6. Agent creates one GitHub issue per epic using `epic.md` template structure.
7. Agent reports issue URLs.

**Documentation audit (`/audit`):**
1. User invokes `/audit [<documentation-dir>]`.
2. `doc-auditor` agent reads all 10 templates to extract required `##` section headings.
3. Agent reads all 10 generated documents from the target directory.
4. Agent checks presence of sections, detects stubs, flags accuracy issues, checks cross-document consistency.
5. Agent writes `audit-report.md` into the same directory as the audited documents.

## Validations

- **Migration inputs completeness** — `migration-advisor` validates that all financial fields in `context/migration-inputs.md` are populated before proceeding. Fields must not be blank or contain the literal string "Unknown".
- **Template structure compliance** — agents are instructed to use templates "exactly"; no programmatic validator enforces this.
- **Epic body structure** — `epic-creator` is instructed to use `epic.md` template "exactly" with `- [ ]` checkbox format for acceptance criteria.
- **Score range** — migration scoring dimensions accept values 1–5; no programmatic enforcement prevents out-of-range values.

## Edge Cases

- **All financial inputs unknown** — `migration-advisor` does not simply warn; it stops entirely. The user must populate the file before the advisor will produce any output.
- **No `documentationV{n}/` directory exists** — `analyse.md` instructs the agent to use `documentationV1/` as the first version; the agent must create the directory.
- **Target codebase is empty or inaccessible** — skill files do not define a fallback; agents will report inability to read files through natural language output and may produce documents with all sections marked unclear.
- **`epic` label already exists in GitHub** — `gh label create ... 2>/dev/null || true` handles this silently; the workflow continues.
- **`doc-auditor` and version mismatch** — if invoked without an argument against a project that has V4 or V5 directories, the agent audits V3, which may not reflect current state. See [change-risk](../change-risk/change-risk-spec.md).

## Exceptions

- **`migration-advisor` refusal** — the only explicit refusal in the system; triggered when financial inputs are unpopulated. No exception type or error code is raised — refusal is expressed as a natural language message to the user.
- **GitHub API failures** — no exception handling is defined for failed `gh issue create` calls; the `gh` CLI's native error output is the only signal.

## Time-Based Logic

- **No scheduling or recurring jobs** — the skills layer has no cron, scheduler, or time-triggered behaviour.
- **Version directory naming** — the only time-adjacent logic is the instruction to use the "next available" `documentationV{n}/` directory, which implicitly reflects the chronological order of analysis runs but has no timestamp.
- **No timezone handling** — not applicable.

## Inconsistencies

- **Output path naming** — `document-system-overview.md` instructs output to `system-overview/index.md`; CLAUDE.md specifies `system-overview/system-overview-spec.md`; `doc-auditor` reads `system-overview/system-overview-spec.md`. Three different conventions exist for the same artefact.
- **`audit.md` default vs. `doc-auditor` default** — `audit.md` says the default is "the most recently created `documentationV*/` directory"; `doc-auditor.md` hardcodes `documentationV3/`. These two definitions directly contradict each other.
- **`assess-migration.md` output path** — writes to `output/rewrite-vs-refactor.md`; `migration-advisor.md` writes to the versioned output directory specified in `migration-inputs.md`. The standalone `/assess-migration` skill and the chained advisor in `/analyse` write to different locations.

## Risks

- **Natural language rule enforcement** — all rules are expressed in prose; an LLM interpreting a skill may satisfy the letter of an instruction while violating its intent (e.g. partially populating a document and not flagging it as a stub).
- **Inconsistent convention creates incorrect audit results** — the naming inconsistency between skill output paths and auditor expected paths means a developer using individual skills will produce documents the auditor cannot find, yielding a falsely empty audit report.
- **No versioning of the rule set** — if a rule in a template or skill is changed, all prior analysis runs were conducted under different rules with no record of the difference.
