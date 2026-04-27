# Change Risk & Technical Debt

## Overview

The `.claude/skills` layer has a **low-to-medium risk profile** overall. As Markdown configuration files, individual skill files are easy to read and modify. However, several structural issues create meaningful risk: a hardcoded version directory in the `doc-auditor` agent, two conflicting output-path naming conventions, no automated validation of skill or template correctness, and implicit coupling between template section headings and auditor behaviour. Changes to templates carry disproportionate downstream risk because they silently alter the auditor's completeness criteria and invalidate previously generated report comparisons.

## High-Risk Areas

| Area | Description | Reason | Impact |
|---|---|---|---|
| `.claude/agents/doc-auditor.md` — default directory | Hardcodes `documentationV3/` as the read target inside the agent system prompt | The `audit.md` skill promises "most recently created `documentationV*/` directory", but the agent reads V3 unconditionally unless an argument overrides it | Running `/audit` without an argument silently audits the wrong version; engineers receive a false quality signal |
| `templates/rewrite-vs-refactor.md` — weight column | Dimension weights (2 or 3) are embedded in a Markdown table with no version identifier | Any weight change retroactively invalidates historical weighted scores without any alert or version mismatch | Migration reports from different time periods become non-comparable without manual review |
| `templates/01–10-*.md` — section headings | `doc-auditor` derives its required-section list by reading `##` headings from templates at runtime | Adding, removing, or renaming a `##` heading in any template immediately changes what the auditor flags as missing or present | A template update that appears cosmetic will cause all future audits to report different completeness counts |
| `analyse.md` — Step 2 conditional | Step 2 (migration advisor) is gated on Step 1 (documentation) completing, enforced only via natural language instruction | No programmatic gate; if the agent misinterprets "do not proceed" in a degraded state, the advisor may run against incomplete documentation | Migration recommendation could be based on partial evidence |
| `create-epics.md` — `$ARGUMENTS` multi-value | The skill requires two arguments (path + `owner/repo`) delivered as a single `$ARGUMENTS` string | No splitting convention is specified; the agent must infer the boundary between path and repo slug | Paths with spaces or unusual formats could cause the repo slug to be parsed incorrectly |

## Known Fragile Components

- **`doc-auditor` agent** — coupling between the agent's hardcoded path and the real output directory version is the single most fragile point in the system. A V4 or V5 audit without an explicit argument will silently audit V3.
- **Output naming convention** — the dual convention (`index.md` in individual skills vs. `*-spec.md` in CLAUDE.md and the auditor) means the auditor and individual skills are incompatible without manual intervention. Adding a new skill without aligning to the spec naming will produce documents invisible to the auditor.
- **`context/migration-inputs.md`** — the migration advisor will refuse entirely if this file is not populated. There is no fallback, no template with safe defaults, and no clear error message beyond the agent's natural language output.

## Coupling Issues

- **Tight coupling between templates and `doc-auditor`** — the auditor's section checklist is derived live from template headings. Template and auditor cannot be changed independently.
- **Tight coupling between `analyse.md` and agent names** — `analyse.md` references `code-documenter` and `migration-advisor` by name. Renaming either agent in `.claude/agents/` would silently break the `analyse.md` pipeline without any error at definition time.
- **Implicit coupling between `audit.md` skill and `doc-auditor` agent** — the skill claims one default behaviour (latest version directory) while the agent implements another (fixed V3 path). These two files must be kept in sync manually.

## Technical Debt

| Item | Description | Priority |
|---|---|---|
| Hardcoded `documentationV3/` in `doc-auditor` | Agent system prompt references a specific version directory rather than resolving dynamically | High |
| Dual output naming convention | Individual skills write `<type>/index.md`; CLAUDE.md and auditor expect `<type>-spec.md`; no single source of truth | High |
| No validation of `$ARGUMENTS` input | All skills pass arguments directly to agent prompts with no type checking, path validation, or multi-argument parsing convention | Medium |
| No template versioning | Templates have no version identifier; changes are undetectable from generated documents | Medium |
| `gh label create` error suppression | `2>/dev/null \|\| true` masks permission and network errors during label creation | Medium |
| `migration-advisor` financial calculation is agent-inferred | Ratios and break-even calculations are performed by the LLM via natural language arithmetic, not deterministic code | Medium |
| No model version pinning | Skills and agents do not pin a Claude model version; output quality may change silently on model upgrades | Low |
| `README.md` in skills directory is minimal | Only describes frontmatter structure; does not document the full skill catalogue or invocation patterns | Low |

## Obsolete Technology

- No obsolete libraries or frameworks are present — the skills layer has no code dependencies beyond Markdown tooling and the `gh` CLI.

> ⚠️ Unclear: No version of Claude Code CLI or `gh` CLI is pinned anywhere in the repository. It is not possible to determine from the files whether the skills are compatible with the current versions of these tools.

## Safe-to-Change Areas

- **Individual `document-*.md` skill files** (excluding `analyse.md` and `audit.md`) — each is self-contained, references one template, and has no dependents within the skills layer. Changes to section guidance or formatting rules in these files affect only that skill's output.
- **`templates/epic.md`** — only consumed by `epic-creator`; no cross-document consistency implications.
- **`summarise-codebase.md`** — standalone skill with no dependents; output goes to `architecture/overview.md` which is outside the 10-document audited set.
- **`document-api.md`, `document-module.md`, `document-onboarding.md`** — ancillary skills outside the 10 core doc types; not read by `doc-auditor`.

## Recommendations

1. **Fix the hardcoded `documentationV3/` path** in `.claude/agents/doc-auditor.md` — replace with dynamic resolution of the latest `documentationV*/` directory, matching the `audit.md` skill's stated behaviour. This is the highest-risk item.
2. **Standardise output naming** — choose one convention (`index.md` or `<type>-spec.md`) and update all skill files, agent definitions, and CLAUDE.md to be consistent. Update the `doc-auditor` agent to expect the chosen format.
3. **Add template versioning** — embed a `version:` field in each template's frontmatter so generated documents can record which template version was used.
4. **Define a multi-argument convention** for `create-epics.md` — document the expected format for `$ARGUMENTS` when multiple values are required (e.g. `--path /foo --repo owner/repo`), or split into two separate skills.
5. **Replace `2>/dev/null \|\| true` on `gh label create`** with an explicit existence check (`gh label list | grep epic`) so permission and network errors surface rather than being silently swallowed.
