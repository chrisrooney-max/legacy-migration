# Rewrite vs Strangle & Refactor — Decision Framework

## Scoring Key

| Final Score | Recommendation |
|---|---|
| < 2.5 | **Strangle & Refactor** — incremental improvement is lower risk and lower cost |
| 2.5 – 3.4 | **Hybrid** — strangle the riskiest parts, refactor the rest |
| ≥ 3.5 | **Rewrite** — a clean rewrite is justified and viable |

## Dimensions

| # | Dimension | Weight | Score (1–5) | Weighted | Rationale |
|---|---|---|---|---|---|
| 1 | Codebase Size | 2 | | | |
| 2 | Code Complexity | 3 | | | |
| 3 | Test Coverage Quality | 3 | | | |
| 4 | Nature of Technical Debt | 3 | | | |
| 5 | Business Criticality & Risk Tolerance | 3 | | | |
| 6 | Functional Understanding | 3 | | | |
| 7 | Consumer Coupling | 2 | | | |
| 8 | Team Capability in Target Language | 2 | | | |
| 9 | Time Pressure | 2 | | | |
| 10 | Deployment & Cutover Simplicity | 2 | | | |
| | **Total** | **25** | | | |

**Final Score = Weighted Score ÷ 25**

## Score Definitions

### 1. Codebase Size (Weight: 2)
| Score | Meaning |
|---|---|
| 1 | Very large (100k+ lines); rewrite would take months or years |
| 2 | Large (50k–100k lines); significant rewrite effort |
| 3 | Medium (10k–50k lines); rewrite is a substantial project |
| 4 | Small (1k–10k lines); rewrite achievable in weeks |
| 5 | Tiny (< 1k lines); rewrite achievable in days |

### 2. Code Complexity (Weight: 3)
| Score | Meaning |
|---|---|
| 1 | Low complexity; clear structure; easy to change safely |
| 2 | Mostly low complexity with isolated problem areas |
| 3 | Mixed; some complex areas but navigable |
| 4 | High complexity throughout; changes frequently cause regressions |
| 5 | Extremely tangled; no safe path to incremental change |

### 3. Test Coverage Quality (Weight: 3)
| Score | Meaning |
|---|---|
| 1 | No tests; refactoring is blind; rewrite has no specification |
| 2 | Minimal tests; partial safety net; behaviour poorly defined |
| 3 | Moderate coverage; refactoring possible but risky in places |
| 4 | Good coverage; safe to refactor; tests usable as rewrite spec |
| 5 | Comprehensive coverage; tests are a complete rewrite specification |

### 4. Nature of Technical Debt (Weight: 3)
| Score | Meaning |
|---|---|
| 1 | Surface-level debt only; straightforward to clean up in place |
| 2 | Mostly surface-level with some deeper issues |
| 3 | Mix of surface and structural debt |
| 4 | Predominantly structural; refactoring would be a near-rewrite anyway |
| 5 | Entirely structural; no viable incremental path |

### 5. Business Criticality & Risk Tolerance (Weight: 3)
| Score | Meaning |
|---|---|
| 1 | Mission critical; zero tolerance for regression or downtime |
| 2 | Important; low risk tolerance; strangle strongly preferred |
| 3 | Important but recoverable; moderate risk tolerance |
| 4 | Non-critical path; business can absorb some risk |
| 5 | Low criticality; high risk tolerance; rewrite risk acceptable |

### 6. Functional Understanding (Weight: 3)
| Score | Meaning |
|---|---|
| 1 | Behaviour poorly understood; rewrite would likely miss edge cases |
| 2 | Core behaviour known but many edge cases unclear |
| 3 | Good understanding of main flows; some edge cases uncertain |
| 4 | Strong understanding of all flows and most edge cases |
| 5 | Complete understanding; behaviour fully documented and testable |

### 7. Consumer Coupling (Weight: 2)
| Score | Meaning |
|---|---|
| 1 | Many consumers with complex contracts; cutover is a large coordinated effort |
| 2 | Several consumers; some contract complexity |
| 3 | A few consumers; manageable cutover with coordination |
| 4 | One or two consumers; simple API contract; easy to cut over |
| 5 | No external consumers; or consumers fully controlled by the same team |

### 8. Team Capability in Target Language (Weight: 2)
| Score | Meaning |
|---|---|
| 1 | No experience in target language; would require significant upskilling |
| 2 | Limited experience; would need support or training |
| 3 | Moderate experience; could execute with some ramp-up |
| 4 | Strong experience; team confident in target language |
| 5 | Expert level; target language is the team's primary stack |

### 9. Time Pressure (Weight: 2)
| Score | Meaning |
|---|---|
| 1 | Immediate pressure; need improvements in days/weeks; rewrite too slow |
| 2 | Short-term pressure; months available; strangle preferred |
| 3 | Moderate timeline; 3–6 months available |
| 4 | No urgent deadline; 6–12 months acceptable |
| 5 | No time pressure; business can wait for the right solution |

### 10. Deployment & Cutover Simplicity (Weight: 2)
| Score | Meaning |
|---|---|
| 1 | Complex deployment; many dependencies; high cutover risk |
| 2 | Moderately complex; requires careful coordination |
| 3 | Manageable; some coordination needed |
| 4 | Simple deployment; easy to run old and new in parallel |
| 5 | Trivial cutover; stateless service; DNS/load balancer switch only |

## Sensitivity Analysis

If any dimension score is unknown (e.g. team capability), calculate the final score across low/mid/high values and show the range:

| Unknown Dimension Score | Final Weighted Score | Verdict |
|---|---|---|
| 1 | | |
| 3 | | |
| 5 | | |

## Recommendation

State the recommendation clearly: **Rewrite**, **Hybrid**, or **Strangle & Refactor**.

Include:
- The key factors that drove the score
- Any dimensions that are close to a threshold and could tip the decision
- 3–5 specific questions the business should answer before committing

## Key Questions for the Business

- _Add 3–5 questions specific to this codebase that would resolve uncertainty in the scores_
