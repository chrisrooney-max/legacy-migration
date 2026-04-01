# Rewrite vs Strangle & Refactor — Decision Framework

## How to Use This Document

Score each dimension 1–5 using the definitions below. Multiply each score by its weight. Sum the weighted scores and divide by the total weight to get a final weighted average.

| Result | Recommendation |
|---|---|
| < 2.5 | **Strangle & Refactor** — incremental improvement is lower risk and lower cost |
| 2.5 – 3.4 | **Hybrid** — strangle the riskiest parts, refactor the rest |
| ≥ 3.5 | **Rewrite** — a clean rewrite is justified and viable |

---

## Scoring Dimensions

### 1. Codebase Size
*Weight: 2*

How large is the codebase? Larger codebases make rewrites more expensive and risky.

| Score | Meaning |
|---|---|
| 1 | Very large (100k+ lines); rewrite would take months or years |
| 2 | Large (50k–100k lines); significant rewrite effort |
| 3 | Medium (10k–50k lines); rewrite is a substantial project |
| 4 | Small (1k–10k lines); rewrite is achievable in weeks |
| 5 | Tiny (< 1k lines); rewrite is achievable in days |

---

### 2. Code Complexity
*Weight: 3*

How difficult is the code to understand and change incrementally? High complexity makes refactoring risky.

| Score | Meaning |
|---|---|
| 1 | Low complexity; clear structure; easy to change safely |
| 2 | Mostly low complexity with isolated problem areas |
| 3 | Mixed; some complex areas but navigable |
| 4 | High complexity throughout; changes frequently cause regressions |
| 5 | Extremely tangled; no safe path to incremental change |

---

### 3. Test Coverage Quality
*Weight: 3*

Does the test suite provide a safety net for refactoring, AND does it define behaviour clearly enough to guide a rewrite?

| Score | Meaning |
|---|---|
| 1 | No tests; refactoring is blind; rewrite has no specification |
| 2 | Minimal tests; partial safety net; behaviour poorly defined |
| 3 | Moderate coverage; refactoring possible but risky in places |
| 4 | Good coverage; safe to refactor; tests usable as rewrite spec |
| 5 | Comprehensive coverage; tests are a complete rewrite specification |

---

### 4. Nature of Technical Debt
*Weight: 3*

Is the debt surface-level (patchable by refactoring) or structural (requires a rebuild to fix)?

| Score | Meaning |
|---|---|
| 1 | Surface-level debt only; straightforward to clean up in place |
| 2 | Mostly surface-level with some deeper issues |
| 3 | Mix of surface and structural debt |
| 4 | Predominantly structural; refactoring would be a near-rewrite anyway |
| 5 | Entirely structural; no viable incremental path |

---

### 5. Business Criticality & Risk Tolerance
*Weight: 3*

How critical is the system, and how much risk can the business absorb during a change?

| Score | Meaning |
|---|---|
| 1 | Mission critical; zero tolerance for regression or downtime; strangle is the only safe approach |
| 2 | Important; low risk tolerance; strangle strongly preferred |
| 3 | Important but recoverable; moderate risk tolerance |
| 4 | Non-critical path; business can absorb some risk |
| 5 | Low criticality; high risk tolerance; rewrite risk is acceptable |

---

### 6. Functional Understanding
*Weight: 3*

How well does the team understand what the system does? Poor understanding makes a rewrite dangerous.

| Score | Meaning |
|---|---|
| 1 | Behaviour poorly understood; rewrite would likely miss edge cases |
| 2 | Core behaviour known but many edge cases unclear |
| 3 | Good understanding of main flows; some edge cases uncertain |
| 4 | Strong understanding of all flows and most edge cases |
| 5 | Complete understanding; behaviour fully documented and testable |

---

### 7. Consumer Coupling
*Weight: 2*

How many external consumers depend on this system, and how hard would it be to cut them over?

| Score | Meaning |
|---|---|
| 1 | Many consumers with complex contracts; cutover would be a large coordinated effort |
| 2 | Several consumers; some contract complexity |
| 3 | A few consumers; manageable cutover with coordination |
| 4 | One or two consumers; simple API contract; easy to cut over |
| 5 | No external consumers; or consumers are fully controlled by the same team |

---

### 8. Team Capability in Target Language
*Weight: 2*

Does the team have sufficient skills in the target language to execute a rewrite confidently?

| Score | Meaning |
|---|---|
| 1 | No experience in target language; rewrite would require significant upskilling |
| 2 | Limited experience; would need support or training |
| 3 | Moderate experience; could execute with some ramp-up |
| 4 | Strong experience; team confident in target language |
| 5 | Expert level; target language is the team's primary stack |

---

### 9. Time Pressure
*Weight: 2*

How urgently does the system need to improve? Rewrites take longer to deliver value than refactoring.

| Score | Meaning |
|---|---|
| 1 | Immediate pressure; need improvements in days/weeks; rewrite too slow |
| 2 | Short-term pressure; months available; strangle preferred |
| 3 | Moderate timeline; 3–6 months available |
| 4 | No urgent deadline; 6–12 months acceptable |
| 5 | No time pressure; business can wait for the right solution |

---

### 10. Deployment & Cutover Simplicity
*Weight: 2*

How easy would it be to deploy the rewritten service and cut traffic over from the old one?

| Score | Meaning |
|---|---|
| 1 | Complex deployment; many dependencies; high cutover risk |
| 2 | Moderately complex; requires careful coordination |
| 3 | Manageable; some coordination needed |
| 4 | Simple deployment; easy to run old and new in parallel |
| 5 | Trivial cutover; stateless service; DNS/load balancer switch only |

---

## Scoring Formula

```
Weighted Score = Σ (dimension score × dimension weight)
Total Weight   = Σ (dimension weights) = 25
Final Score    = Weighted Score ÷ Total Weight
```

---

## Applied to sdkman-broker

| Dimension | Weight | Score | Weighted | Rationale |
|---|---|---|---|---|
| Codebase Size | 2 | 4 | 8 | ~15 classes; rewrite achievable in days |
| Code Complexity | 3 | 2 | 6 | Low overall; Scala interop is the main pain point but isolated to `VersionRepo` |
| Test Coverage Quality | 3 | 4 | 12 | 7 Cucumber feature files fully define expected behaviour; strong rewrite specification |
| Nature of Technical Debt | 3 | 3 | 9 | Mixed: outdated MongoDB driver and Scala model are structural; silent audit failures are fixable in place |
| Business Criticality & Risk Tolerance | 3 | 3 | 9 | Important service but not mission critical; moderate risk tolerance |
| Functional Understanding | 3 | 5 | 15 | Excellent — 10 analysis documents + full feature file spec |
| Consumer Coupling | 2 | 4 | 8 | Single consumer (SDKMAN! CLI); simple HTTP contract; easy cutover |
| Team Capability in Target Language | 2 | — | — | ⚠️ Unknown — score this based on your team's skills |
| Time Pressure | 2 | 3 | 6 | No urgent deadline visible in the codebase |
| Deployment & Cutover Simplicity | 2 | 4 | 8 | Stateless Docker service; straightforward to run old and new in parallel |

**Weighted Score (excluding team capability):** 81
**Total Weight (excluding team capability):** 23
**Interim Score:** 81 ÷ 23 = **3.52**

### Sensitivity to Team Capability

| Team Capability Score | Final Weighted Score | Result |
|---|---|---|
| 1 (no target language experience) | (81+2) ÷ 25 = **3.32** | Hybrid |
| 2 | (81+4) ÷ 25 = **3.40** | Hybrid |
| 3 | (81+6) ÷ 25 = **3.48** | Rewrite |
| 4 | (81+8) ÷ 25 = **3.56** | Rewrite |
| 5 | (81+10) ÷ 25 = **3.64** | Rewrite |

---

## Recommendation for sdkman-broker

**If the team has moderate or better experience in the target language (score ≥ 3): Rewrite.**

The codebase is small, the behaviour is completely understood and specified by the Cucumber test suite, and there is only one consumer with a simple HTTP contract. The main structural debts (Scala model interop, outdated MongoDB driver) are easier to eliminate in a rewrite than to migrate in place.

**If the team has limited target language experience (score ≤ 2): Hybrid.**

Strangle the Scala model dependency by replacing `sdkman-persistent-model` with a native Java/Kotlin model, and upgrade the MongoDB driver in place. These two changes eliminate the highest-risk debt without a full rewrite.

---

## Key Questions to Answer Before Deciding

1. What is the team's experience level in the target language?
2. Is there a hard deadline for delivery?
3. Can the SDKMAN! CLI team coordinate a cutover window?
4. Is there appetite to run old and new services in parallel during validation?
