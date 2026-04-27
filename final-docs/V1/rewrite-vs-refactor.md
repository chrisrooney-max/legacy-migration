# Migration Decision Report — .claude/skills (Legacy Migration Tooling)

## Financial Snapshot

| | Monthly |
|---|---|
| Maintenance costs | $50,000 |
| Engineering costs | $40,000 |
| Call center costs | $5,000 |
| **Total cost burden** | **$95,000** |
| Revenue | $1,000,000 |
| Cost-to-revenue ratio | 9.5% |

**Cost-to-revenue ratio band:** 5–10% (Low urgency). Costs are manageable; 6–12 months are available before financial pressure becomes acute. This maps to a Time Pressure score of **4**.

---

## Technical Scoring

| # | Dimension | Weight | Score (1–5) | Weighted | Rationale |
|---|---|---|---|---|---|
| 1 | Codebase Size | 2 | 5 | 10 | 18 skill files + 12 templates = 30 Markdown files, ~1,125 total lines across `.claude/skills/`. The entire surface is well under 1k lines of structured content per the `wc -l` counts. A rewrite is achievable in days, not weeks. |
| 2 | Code Complexity | 2 | 2 | 4 | The codebase is flat and readable; the `$ARGUMENTS` substitution pattern is uniform across all 18 skill files (`assess-migration.md`, `document-*.md`). However, two structural coupling issues elevate complexity above the minimum: (a) the template↔doc-auditor live-heading coupling (`change-risk-spec.md`, High Risk row 3) and (b) the `analyse.md` sequential gate enforced only in natural language (`change-risk-spec.md`, High Risk row 4). These are isolated problem areas, not pervasive tangling. Score: 2. |
| 3 | Test Coverage Quality | 1 | 1 | 1 | No tests of any kind exist — no unit, integration, E2E, or CI pipeline (`testing-spec.md`: "no `package.json`, no `pytest`, no Makefile test target, no CI workflow"). The only automated quality signal is the post-hoc `/audit` skill, which checks generated outputs, not skill definitions. Any refactoring or rewrite has no automated safety net and no specification derived from tests. Score: 1. |
| 4 | Nature of Technical Debt | 3 | 2 | 6 | Debt is mostly surface-level: hardcoded `documentationV3/` path in `doc-auditor`, dual naming conventions (`index.md` vs `*-spec.md`), suppressed `gh label create` errors, absence of template versioning (`change-risk-spec.md`, Technical Debt table). One structural issue exists — the tight coupling between template `##` headings and auditor runtime behaviour — but it is a single seam, not a pervasive architectural problem. Incremental fixes are straightforward. Score: 2. |
| 5 | Business Criticality & Risk Tolerance | 3 | 3 | 9 | Risk tolerance is stated as "Moderate" (`migration-inputs.md`). The system-overview doc classifies the system as "Important" (not mission-critical). It is internal-only tooling: failure produces incorrect documentation outputs but does not affect customer-facing systems or revenue directly (`system-overview-spec.md`, Business Criticality section). Moderate risk tolerance with recoverable impact maps to score 3. |
| 6 | Functional Understanding | 3 | 4 | 12 | All 10 document types are fully documented in `documentationV1/`. The `system-overview-spec.md`, `architecture-spec.md`, `code-structure-spec.md`, and `change-risk-spec.md` together provide a complete picture of all flows, known issues, and coupling points. Edge cases (hardcoded paths, naming conflicts, `$ARGUMENTS` multi-value parsing) are explicitly documented. Only minor gaps remain (no CODEOWNERS, no version pinning details). Score: 4. |
| 7 | Consumer Coupling | 2 | 5 | 10 | The system has no external API consumers. It is invoked via Claude Code CLI by the developer who owns the repository. There are no contracts, SLAs, or coordinated cutovers to manage (`integrations-spec.md`: "one outbound integration" only, `gh` CLI for `/create-epics`; no inbound integrations). Score: 5. |
| 8 | Team Capability in Target Language | 2 | 4 | 8 | The target language is Markdown + YAML frontmatter. The existing skill files are themselves written in this format, and the team has already authored 30 well-structured files. No upskilling is needed; rewriting or refactoring any skill file requires only familiarity with Claude Code's prompt template conventions. Score: 4. |
| 9 | Time Pressure | 2 | 4 | 8 | Cost-to-revenue ratio = $95,000 / $1,000,000 = **9.5%**, falling in the **5–10% band** (Low urgency). Costs are manageable; the business is not under immediate financial pressure to act. A 6–12 month window is available. Score: 4. |
| 10 | Deployment & Cutover Simplicity | 2 | 5 | 10 | No deployment infrastructure exists. The skills layer is pure configuration; "deployment" is `git pull` on the developer's machine (`operations-spec.md`: "Skills are deployed by committing changes to the main branch"). No Dockerfile, no CI/CD, no stateful service to migrate. Cutover is trivial. Score: 5. |
| | **Total** | **25** | | **78** | |

**Final Score = 78 ÷ 25 = 3.12 — Hybrid**

Per the scoring key: a score of 2.5–3.4 indicates a **Hybrid** recommendation — strangle the riskiest parts, refactor the rest.

---

## Financial Case

### Assumptions

- Engineer cost rate: $120,000/year fully loaded = $10,000/engineer-month.
- Team capacity: 5 engineers full-time (from `migration-inputs.md`).
- Post-migration savings estimates are conservative: the skills layer is configuration, so "maintenance" savings derive primarily from reduced debugging time, less confusion from naming inconsistencies, and fewer silent audit failures — not infrastructure cost reduction.
- "Maintenance costs" ($50,000/mo) and "Engineering costs" ($40,000/mo) are assumed to cover the broader system this tooling supports, not exclusively the `.claude/skills` layer itself, given the skills layer has no runtime infrastructure costs. The analysis applies these figures as given.
- Call center costs ($5,000/mo) are unlikely to be directly affected by refactoring this internal tooling. Savings assumed at 0% for call center in all scenarios.
- Maintenance savings from fixing the known issues (hardcoded path, dual naming, suppressed errors) are estimated conservatively at 10% of the maintenance cost burden.

---

### Do Nothing (12-month projection)

| | Value |
|---|---|
| Monthly cost burden | $95,000 |
| 12-month total cost | $1,140,000 |
| Known issues that remain unresolved | Hardcoded `documentationV3/` path, dual naming conventions, no CI, no test coverage, silent error suppression |
| Cost trajectory | Flat to slightly increasing — no infrastructure cost growth, but known fragile coupling points (template↔auditor) will accumulate debt as the template set evolves |

The financial cost of doing nothing is not severe (9.5% cost-to-revenue ratio is manageable), but the quality signal risk compounds over time: audit runs will silently read the wrong version directory, and documents generated by individual skills will be invisible to the auditor without manual intervention.

---

### Strangle & Refactor

**Scope:** Fix the three high-priority debt items incrementally without restructuring the whole system:
1. Replace the hardcoded `documentationV3/` path in `doc-auditor` with dynamic latest-version resolution.
2. Standardise the output naming convention to `*-spec.md` across all individual skill files.
3. Replace `2>/dev/null || true` on `gh label create` with an explicit existence check.
4. Add template versioning (frontmatter `version:` field).
5. Define a multi-argument convention for `create-epics.md`.

**Effort estimate:** Given the codebase is ~30 Markdown files, these five changes require careful reading, targeted edits, and end-to-end manual testing. Estimated effort: 1–2 engineer-weeks (0.5 engineer-months).

| | Value |
|---|---|
| Migration effort | 0.5 engineer-months |
| Migration team cost | $5,000 (0.5 × $10,000) |
| Existing costs during migration | $47,500 (0.5 months × $95,000) |
| **Total migration investment** | **$52,500** |
| Estimated monthly savings post-fix | $5,000 (conservative 10% reduction in maintenance debugging overhead; call center unchanged) |
| Break-even point | ~11 months ($52,500 ÷ $5,000/mo) |
| 12-month net position | –$52,500 + (11.5 months remaining × $0 savings) ≈ –$52,500 |
| 24-month net position | –$52,500 + (23.5 months × $5,000) = **+$65,000** |

Note: Because savings are conservative and the investment is small, break-even occurs within a year and the 24-month position is solidly positive.

---

### Rewrite

**Scope:** Replace the entire `.claude/skills` layer with a fresh implementation — new skill files, redesigned templates, a revised agent architecture, and a formal naming and versioning convention from the start.

**Effort estimate:** 30 files, well-understood domain (the team already wrote them once). A clean rewrite of the skill layer is estimated at 1 engineer-month. Applying the 1.5× contingency multiplier for rewrites: **1.5 engineer-months**.

| | Value |
|---|---|
| Rewrite effort (with 1.5× contingency) | 1.5 engineer-months |
| Rewrite team cost | $15,000 (1.5 × $10,000) |
| Existing costs during rewrite | $142,500 (1.5 months × $95,000) |
| **Total rewrite investment** | **$157,500** |
| Estimated monthly savings post-rewrite | $9,500 (conservative 10% maintenance reduction + 1% engineering reduction from cleaner system; call center unchanged) |
| Break-even point | ~17 months ($157,500 ÷ $9,500/mo) |
| 12-month net position | –$157,500 + (10.5 months × $0 savings during ramp) ≈ –$157,500 |
| 24-month net position | –$157,500 + (22.5 months × $9,500) = **+$56,250** |

A full rewrite reaches break-even later than a targeted refactor and delivers only marginally higher savings, because the underlying cost drivers (maintenance, engineering) are not primarily caused by the skills layer's structural deficiencies — they are broader system costs. The rewrite provides a cleaner foundation for future growth but does not justify the extra investment over a focused fix.

---

## Recommendation

**Hybrid — Strangle the Riskiest Parts, Refactor the Rest.**

The technical score of **3.12** sits squarely in the Hybrid band. The financial case reinforces this: at a 9.5% cost-to-revenue ratio, there is no acute financial pressure that would justify the time and risk of a full rewrite. The targeted Strangle & Refactor path costs $52,500, breaks even in ~11 months, and reaches a +$65,000 net position by month 24. The full rewrite costs three times as much ($157,500), breaks even later (~17 months), and provides only marginal additional savings.

**The decisive factors driving this recommendation:**

1. **Codebase size (Score 5) and deployment simplicity (Score 5)** make a rewrite technically easy, but this cuts both ways — they equally make incremental fixes fast and low-risk. There is no argument from technical difficulty for a rewrite.
2. **Zero test coverage (Score 1)** is the single most concerning technical finding. A rewrite without tests delivers no improvement in confidence; a refactor without tests carries similar risk. Both paths require the same first step: establish a quality gate before changing anything. The absence of CI is the highest-priority structural gap.
3. **Mostly surface-level debt (Score 2)** means that the five documented fixes (`change-risk-spec.md` Recommendations 1–5) are discrete, well-scoped, and independently releasable. There is no intractable structural tangle requiring a clean slate.
4. **Strong functional understanding (Score 4)** means the refactoring path carries low risk of missing behaviour. The domain is fully documented in `documentationV1/`.
5. **No external consumers (Score 5)** removes the biggest cutover risk from a rewrite: there are no contracts to honour, no coordinated migrations, and no API versioning concerns.

**Where the technical score and financial case are aligned:** Both point to Hybrid / Strangle & Refactor. There is no tension between the two signals.

**Recommended sequencing:**

1. Fix the `doc-auditor` hardcoded path bug immediately — it is producing incorrect audit results today.
2. Standardise the output naming convention across all skills in a single commit.
3. Add a minimal CI check (GitHub Actions workflow running a Markdown linter) to establish an automated quality gate.
4. Address template versioning and `create-epics` argument convention as follow-on items.
5. Defer any rewrite decision until the refactored system has run for 3–6 months; reassess if new structural problems emerge.

---

## Key Questions for the Business

1. **What share of the $95,000 monthly cost burden is attributable specifically to the skills layer, versus the broader migration documentation program?** If the costs are largely independent of skill layer quality, the financial case for any intervention (rewrite or refactor) weakens further. Clarifying this would sharpen the break-even calculation.

2. **Is there a planned expansion of the skill catalogue (e.g. new document types, new agents)?** If significant new skills are planned, establishing the correct naming convention and template versioning convention before expansion would increase the value of the refactor substantially. A rewrite becomes more attractive if the current architecture cannot accommodate planned growth without accumulating further debt.

3. **How frequently does the `/audit` skill produce incorrect results due to the hardcoded `documentationV3/` bug?** If the team regularly runs `/audit` against V4 and V5 directories and relies on the output for quality decisions, the impact of this bug is ongoing and not just theoretical. Quantifying the rework cost per incorrect audit run would strengthen the business case for fixing it immediately.

4. **Does the team intend to adopt Claude Code CLI version pinning or move to a different agentic runtime?** If a runtime change is planned, a full rewrite of skill files for the new format may be unavoidable regardless of this analysis, which would change the recommendation from Hybrid to Rewrite.

5. **What is the team's tolerance for silent failures in the `create-epics` workflow?** The `2>/dev/null || true` pattern on `gh label create` and the natural-language deduplication check both produce silent failures today. If duplicate epics or label creation failures have already caused rework, the actual cost of the current debt may be higher than the conservative 10% maintenance savings estimate used in this analysis.
