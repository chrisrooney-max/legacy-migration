# Documentation Audit Report

**Date:** 2026-04-07
**Document audited:** `documentationV4/rewrite-vs-refactor.md`
**Template checked against:** `.claude/skills/templates/rewrite-vs-refactor.md` and `migration-advisor.md` output specification
**Overall rating:** Medium

---

## Summary

The report is substantive and well-evidenced for a migration decision document. The codebase metrics are grounded in real file counts, the financial case is clearly structured, and the narrative recommendation is coherent. However, the report contains a critical scoring error: Dimensions 2 and 3 are scored with Weight=2 instead of the template-specified Weight=3. This produces a final score of 2.44 (Strangle & Refactor) when the correct figure is 2.72 (Hybrid) — a verdict change. The sensitivity analysis inherits the same error. Additionally, the report is missing the required standalone Sensitivity Analysis section for Dimension 8, produces only one sensitivity analysis when additional dimensions arguably warrant one (Dimension 6, Dimension 8), and the Time Pressure score rationale describes score 5 behaviour while citing score 4. The consumer-coupling treatment may also misread the user's intent.

---

## Section Completeness

The `migration-advisor.md` output specification requires these top-level sections:

| Required Section | Present | Notes |
|---|---|---|
| Financial Snapshot | Yes | Complete table with all fields populated |
| Technical Scoring | Yes | Present but contains weight errors — see Flags |
| Financial Case — Do Nothing | Yes | Complete |
| Financial Case — Strangle & Refactor | Yes | Complete |
| Financial Case — Rewrite | Yes | Complete |
| Recommendation | Yes | Present and argued |
| Key Questions for the Business | Yes | 5 questions provided |
| Sensitivity Analysis (standalone section) | No | Appears only as an appended callout block after Key Questions; not a dedicated section |

**Non-required but present:** `Codebase Analysis Summary` — this is additive and useful; not a gap.

---

## Findings

### Critical: Incorrect Dimension Weights — Score and Verdict Are Wrong

**Location:** Technical Scoring table, Dimensions 2 and 3.

The template (`.claude/skills/templates/rewrite-vs-refactor.md`, Score Definitions) specifies:

| Dimension | Template Weight | Report Weight |
|---|---|---|
| 2 — Code Complexity | **3** | 2 |
| 3 — Test Coverage Quality | **3** | 2 |

All other dimensions use the correct weight.

**Impact on the score:**

| | Report (incorrect) | Correct |
|---|---|---|
| Dim 2 weighted (score 2) | 2×2 = 4 | 2×3 = **6** |
| Dim 3 weighted (score 5) | 5×2 = 10 | 5×3 = **15** |
| Weighted total | 61 | **68** |
| Final score | 61÷25 = **2.44** | 68÷25 = **2.72** |
| Verdict | Strangle & Refactor | **Hybrid** |

A score of 2.72 falls in the Hybrid band (2.5–3.4). The verdict in the report is therefore incorrect.

**Impact on the sensitivity analysis:**

The sensitivity analysis for Dimension 8 also uses the wrong base, producing wrong totals and wrong final scores:

| Dim 8 Score | Report Total | Report Final | Correct Total | Correct Final | Correct Verdict |
|---|---|---|---|---|---|
| 1 | 55 | 2.20 | **62** | **2.48** | **Strangle & Refactor** |
| 3 (used) | 61 | 2.44 | **68** | **2.72** | **Hybrid** |
| 5 | 65 | 2.60 | **72** | **2.88** | **Hybrid** |

With correct weights, only the Dim 8=1 scenario falls in Strangle & Refactor (and only barely, at 2.48). The report's claim that "only at expert-level team capability does the verdict shift to Hybrid" is incorrect — Hybrid is the central verdict at moderate capability.

---

### Significant: Time Pressure Score Rationale Does Not Match the Score Assigned

**Location:** Technical Scoring table, Dimension 9.

The report assigns **Score 4** to Time Pressure. The template definition for Score 4 is: "No urgent deadline; 6–12 months acceptable."

The rationale written in the report states: "There is no stated deadline. Financial pressure does not mandate rapid action; **the business can wait for the right solution**."

The phrase "the business can wait for the right solution" is the verbatim definition of **Score 5** in the template ("No time pressure; business can wait for the right solution"). The score and the rationale describe different positions. Either the score should be 5 (raising the weighted total by 2 additional points) or the rationale should be revised to reference a 6–12 month planning horizon.

---

### Significant: Sensitivity Analysis Is a Single Appended Callout, Not a Standalone Section

**Location:** End of document, after Key Questions for the Business.

The template (`rewrite-vs-refactor.md`) includes "Sensitivity Analysis" as a named section with its own table. The `migration-advisor.md` spec states: "For any dimension where the input is 'Unknown' and the codebase cannot resolve it, mark it as unknown and run a sensitivity analysis scoring it at 1, 3, and 5."

The sensitivity analysis appears as a `>` blockquote callout appended after Key Questions. It is not a section heading (`##`) and is not structured per the template table format. This makes it easy to overlook and breaks the expected document structure.

---

### Moderate: Consumer Coupling Argument May Misread the Use Case

**Location:** Technical Scoring table, Dimension 7; Recommendation section.

The report scores Consumer Coupling at 1 and argues that Spring Framework "has millions of downstream consumers." This is true of the public open-source project. However, `migration-inputs.md` describes the system only as "Key system for us" — language that suggests an internal organisational use case, not a public library with millions of external consumers.

The report does acknowledge this in Key Question 4: "If your organisation's use case is an internal fork or a bounded internal deployment, the cutover risk is significantly lower and the score would rise toward 2–3, potentially shifting the verdict toward Hybrid." This is a valid hedge, but because it is buried in Key Questions rather than surfaced as a scoring assumption, the score of 1 stands as presented without adequate qualification.

**Required action:** Either (a) add a `> ⚠️ Unclear:` callout in the Dimension 7 row stating the ambiguity and the conditional scoring, or (b) run a sensitivity analysis for Dimension 7 (internal fork vs. public library = score 1 vs. score 3–4) in the same way Dimension 8 is handled.

---

### Minor: Only One Sensitivity Analysis Provided

**Location:** Sensitivity analysis callout at end of document.

Dimension 8 (Team Capability) is the only dimension to receive a sensitivity analysis, justified because the team expertise level is "Unknown-adjacent." However:

- **Dimension 6 (Functional Understanding):** The migration-inputs.md system description is "Key system for us" — five words that provide no information about functional understanding. The score of 4 is driven almost entirely by the fact that Spring Framework has public documentation. If this is an internal system with limited documentation visibility, the score could be 2–3.
- **Dimension 7 (Consumer Coupling):** As noted above, the score is contingent on an ambiguous use-case assumption and merits a sensitivity range.

The `migration-advisor.md` spec requires sensitivity analysis for any dimension where the input is "Unknown." The system description input is effectively unknown for all qualitative dimensions. The single sensitivity analysis for Dim 8 does not satisfy this requirement.

---

### Minor: "Codebase Analysis Summary" Section Not in Template

**Location:** Codebase Analysis Summary table.

This section is not specified in the output structure in `migration-advisor.md`. It is useful and well-populated, but its presence means the document does not strictly follow the required output structure. This is a low-priority finding — the content is additive, not harmful.

---

### Minor: Stub / Thin Content

None. All sections contain substantive content with real data. No placeholder or "TBD" text detected.

---

## Unsupported Claims

| Claim | Location | Assessment |
|---|---|---|
| "millions of downstream consumers" | Dim 7 rationale | True of the public OSS project; unverified for the user's actual deployment context. Needs a `⚠️ Unclear:` callout or sensitivity treatment. |
| "25% chance of partial revenue disruption during cutover" | Rewrite assumptions | The 25% figure is stated without citation. No source in codebase or migration-inputs.md supports this specific probability. Should be labelled as an assumption with a range. |
| "the test suite is an exceptional asset" | Recommendation bullet 2 | Well-supported — 61,198 assertions and 3,460 test files cited. No issue. |
| Break-even "approximately month 15 from today" | Strangle & Refactor table | Math checks out: 3-month migration + 12.5-month payback ≈ 15 months. Supported. |
| Break-even "~200 months (16+ years)" for rewrite | Rewrite table | Math: $3,630,000 ÷ $18,000 = 201.7 months. Correct. Supported. |

---

## Missing Sensitivity Analyses

Per `migration-advisor.md` § "Handle unknowns": sensitivity analysis is required for any dimension whose input is "Unknown" and cannot be resolved from the codebase.

| Dimension | Input completeness | Sensitivity required? | Present? |
|---|---|---|---|
| 6 — Functional Understanding | System described as "Key system for us" only; no internal documentation referenced | Yes — score is codebase-inferred, not input-confirmed | No |
| 7 — Consumer Coupling | Use case is ambiguous (OSS vs. internal) | Yes — score could be 1 or 3–4 depending on answer | No |
| 8 — Team Capability | "5 engineers full-time" — no expertise level stated | Yes | Yes (present but math is wrong) |

---

## Recommended Actions

1. **[Critical] Fix Dimension weights 2 and 3.** Set Dimension 2 (Code Complexity) and Dimension 3 (Test Coverage Quality) to Weight=3 per the template. Recalculate all weighted scores, the final score (correct: 2.72), and the verdict (correct: Hybrid). Update the narrative recommendation to reflect the Hybrid verdict — this changes what the business should do.

2. **[Critical] Recalculate the Dimension 8 sensitivity analysis** using the corrected weights. All three scenario scores and verdicts in the sensitivity table are wrong and must be updated.

3. **[Significant] Resolve the Time Pressure score vs. rationale mismatch.** Either change the score from 4 to 5 (and recalculate) or revise the rationale to describe a 6–12 month planning horizon, not open-ended patience.

4. **[Significant] Promote Sensitivity Analysis to a standalone `##` section** with the table format specified in the template. Remove the buried blockquote callout and replace with a structured section that covers Dimensions 6, 7, and 8.

5. **[Moderate] Add a sensitivity analysis for Dimension 7 (Consumer Coupling).** Score it at 1 (millions of external consumers, OSS interpretation) vs. 3–4 (bounded internal deployment). This is the single most consequential unknown — a score of 3 instead of 1 raises the weighted total by 4 points (to 72+), pushing the final score to ~3.0 (mid-Hybrid range).

6. **[Moderate] Add a sensitivity analysis for Dimension 6 (Functional Understanding).** The system description "Key system for us" provides no information about internal documentation or institutional knowledge. If internal understanding is lower than OSS documentation implies, the score may be 2–3, not 4.

7. **[Minor] Add a `⚠️ Unclear:` callout in Dimension 7** explicitly flagging the OSS-vs-internal ambiguity at the point of scoring, not only in Key Questions. This surfaces the assumption where the reader needs it.

8. **[Minor] Source the 25% revenue disruption probability** in the Rewrite assumptions section. Either cite an industry benchmark, replace with a conservative range (e.g., "10–40%"), or remove the specific figure and describe the risk qualitatively.
