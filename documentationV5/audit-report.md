# Documentation Audit Report

**Date:** 2026-04-07
**Documents audited:** 1
**Audited by:** doc-auditor agent

---

## Key Metrics

| Metric | Value |
|---|---|
| Total documents | 1 |
| Sections present | 5 of 5 required top-level sections |
| Sections missing | 0 top-level; 1 required sub-section missing (see below) |
| Stub sections detected | 0 |
| Accuracy flags | 2 |
| Cross-document inconsistencies | N/A (single document) |

---

## Completeness by Document

The required output structure is defined in `/Users/christopherrooney/legacy-migration/.claude/agents/migration-advisor.md`. Required sections:

| Required Section | Present | Notes |
|---|---|---|
| `## Financial Snapshot` | Yes | Table fully populated with all 6 rows |
| `## Technical Scoring` | Yes | All 10 dimensions scored with rationale and source citations |
| `## Financial Case` | Yes | Section present with assumptions table |
| `### Do Nothing (12-month projection)` | Yes | Table and narrative present |
| `### Strangle & Refactor` | Yes | Table with all financial parameters present |
| `### Rewrite` | Yes | Table with all financial parameters present |
| `## Recommendation` | Yes | Combined technical + financial recommendation present |
| `## Key Questions for the Business` | Yes | 5 questions present (required: 3–5) |

**Additional sub-section present (not required by spec):** `### Scenario Comparison Summary` — present after the three sub-sections; not listed as a required section in `migration-advisor.md` output structure.

**Missing required sub-section:** `migration-advisor.md` instructs the agent to state all assumptions clearly. A dedicated `### Assumptions` sub-section appears inside `## Financial Case` but is not listed in the spec's output structure. The section is present in the document; this is an addition beyond the spec, not a gap.

**Summary completeness score:** High — all 5 required top-level sections and all 3 required `## Financial Case` sub-sections are present; no section is missing.

---

## Missing Sections

None. All sections defined in the `migration-advisor.md` output structure are present.

---

## Stub Sections

None. Every section contains substantive content exceeding 30 words with specific data, citations, or analysis. No placeholder text ("TBD", "To be populated", "N/A", template prompt text) was detected anywhere in the document.

---

## Accuracy Flags

| # | Section | Flag |
|---|---|---|
| 1 | `## Technical Scoring` — Dimension 9 (Time Pressure) | Score 4 is defined in the template as "No urgent deadline; 6–12 months acceptable." The rationale states "no acute instability" and a healthy cost-to-revenue ratio, which aligns with score 4. However, `migration-advisor.md` states "High cost-to-revenue ratio → more urgent to act → informs how much time pressure exists." The document assigns score 4 (low time pressure) based on a 9.5% ratio — internally consistent with the financial data, but the score direction for Dimension 9 is inverted relative to its label in the scoring table: a score of 4 means *low* pressure (favours refactor), while the dimension weight logic in the advisor spec frames high ratio as high urgency. The document treats 9.5% as low urgency (score 4), which is consistent with the template definitions, but the rationale does not explicitly address whether 9.5% was judged low or high relative to a threshold. No explicit threshold is defined in either spec. Flag: the threshold used to classify 9.5% as "not a crisis level" is not cited or sourced. |
| 2 | `## Financial Case — Rewrite` | The document states "This section treats it as rewriting the internal system/application that depends on Spring Boot" — a scope different from the codebase analysed (Spring Boot itself). The Technical Scoring section analyses the Spring Boot framework repository. The Rewrite financial case applies to a different, unanalysed codebase. The document acknowledges this tension but does not re-score dimensions for the application codebase. The 12-month and 24-month net figures in the Rewrite scenario are therefore derived from an unevaluated scope. This is disclosed in Key Question 1 but not in the Financial Case section itself. |

---

## Cross-Document Consistency

Single-document audit. Cross-document consistency checks are not applicable.

### Internal consistency within `rewrite-vs-refactor.md`

**Consistent:**

- Financial Snapshot figures are used consistently throughout: $95,000 total monthly cost burden and $1,000,000 revenue appear identically in the Financial Snapshot table, the Do Nothing projection ($95,000 × 12 = $1,140,000), and the Rewrite scenario ($95,000/month ongoing costs).
- The 9.5% cost-to-revenue ratio stated in the Financial Snapshot ($95,000 ÷ $1,000,000) is arithmetically correct.
- The weighted score calculation (64 ÷ 25 = 2.56) is arithmetically correct. The individual weighted scores (2 + 6 + 15 + 6 + 9 + 6 + 2 + 6 + 8 + 4 = 64) match the values in the scoring table.
- The Hybrid verdict (2.56 falls in the 2.5–3.4 range) is consistent with the scoring key in `.claude/skills/templates/rewrite-vs-refactor.md`.
- The 5-engineer team capacity figure from `migration-inputs.md` is applied consistently in the Strangle & Refactor and Rewrite effort duration calculations.
- Post-migration monthly savings ($23,500 = $15,000 + $8,000 + $500) are arithmetically correct given the stated percentages: 30% × $50,000 maintenance = $15,000; 20% × $40,000 engineering = $8,000; 10% × $5,000 call center = $500.
- The 1.5× contingency multiplier for the Rewrite scenario is applied as instructed by `migration-advisor.md` (base 15–20 engineer-months × 1.5 = 22.5–30, reported as 22–30).

**Internal inconsistency:**

| # | Sections involved | Inconsistency |
|---|---|---|
| 1 | `## Technical Scoring` vs. `## Financial Case — Rewrite` | Technical Scoring analyses the Spring Boot framework repository (326,000 lines, 438 Gradle subprojects, 4,027 test files). The Rewrite financial case explicitly states it "treats it as rewriting the internal system/application that depends on Spring Boot" — a different, uncharacterised codebase. Effort estimates (15–20 engineer-months base) and savings projections in the Rewrite scenario are therefore not grounded in the same codebase that produced the dimension scores. The Strangle & Refactor scenario does not contain this explicit scope-shift statement, leaving its scope ambiguous by comparison. |
| 2 | `## Financial Case — Rewrite` (break-even calculation) | Break-even stated as "~33 months": $930,000 ÷ $28,500/month = 32.6 months. This is arithmetically correct. However, the $930,000 is derived from "$155,000 × 6 months" (6 being the high end of the 4.5–6 month duration range). Using the low end (4.5 months): $155,000 × 4.5 = $697,500 ÷ $28,500 = ~24.5 months. The document presents only the high-end figure without noting the range implied by the 4.5–6 month duration. This is a partial presentation, not an arithmetic error. |
