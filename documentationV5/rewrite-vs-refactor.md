# Migration Decision Report — Spring Boot

## Financial Snapshot

| | Monthly |
|---|---|
| Maintenance costs | $50,000 |
| Engineering costs | $40,000 |
| Call center costs | $5,000 |
| **Total cost burden** | **$95,000** |
| Revenue | $1,000,000 |
| Cost-to-revenue ratio | 9.5% |

**Derived figures:**
- Annual cost burden: $1,140,000
- Cost burden as % of revenue: 9.5% — relatively healthy; no acute financial emergency driving urgency.

---

## Technical Scoring

> **Codebase analysed:** `https://github.com/spring-projects/spring-boot` (shallow clone, April 2026)

| # | Dimension | Weight | Score (1–5) | Weighted | Rationale |
|---|---|---|---|---|---|
| 1 | Codebase Size | 2 | 1 | 2 | ~326,000 lines of Java across 8,550 source files and 438 Gradle submodules. Comfortably in the "100k+ lines" band that scores 1. Source: `wc -l` totals from `/tmp/spring-boot`. |
| 2 | Code Complexity | 3 | 2 | 6 | Well-structured but non-trivial. `SpringApplication.java` is 1,876 lines with 30+ imports; `ConfigurationPropertyName.java` 1,218 lines; `OnBeanCondition.java` 936 lines. 68 production files exceed 500 lines. Conditional wiring is pervasive (509 files use `@ConditionalOn*`). Complexity is real but localised — not tangled globally. Score: 2. |
| 3 | Test Coverage Quality | 3 | 5 | 15 | Exceptional test suite. 4,027 test files; 2,798 files matching `*Test*.java`; 951 use `@SpringBootTest`/`@MockitoBean`; 2,394 use AssertJ/JUnit assertions. Both unit and integration layers are populated. Tests serve as a functional specification. Score: 5. |
| 4 | Nature of Technical Debt | 3 | 2 | 6 | Debt is largely surface-level or isolated. The modular Gradle layout (438 subprojects) enforces separation of concerns. Large files like `KafkaProperties.java` (1,759 lines) and `RabbitProperties.java` (1,409 lines) represent configuration aggregation, not broken abstraction. No evidence of god objects tightly coupling unrelated concerns. Score: 2. |
| 5 | Business Criticality & Risk Tolerance | 3 | 3 | 9 | Revenue $1,000,000/month; team stated risk tolerance is "Moderate". The system is described as a "key system". Moderate risk tolerance maps to score 3 per the rubric ("important but recoverable; moderate risk tolerance"). Source: `migration-inputs.md` — risk tolerance field. |
| 6 | Functional Understanding | 3 | 2 | 6 | Spring Boot is an open-source framework with extensive official documentation, Javadoc, and test coverage that acts as a specification. However, the user's own context is sparse ("Key system for us") with no documented edge cases, bus factor, or onboarding time. Unknown operational knowledge of how their application layers on top scores this conservatively. Score: 2. |
| 7 | Consumer Coupling | 2 | 1 | 2 | 171 starter modules expose a public API consumed by potentially millions of downstream projects. Even an internal upgrade scenario involves highly stable public contracts — `SpringApplication`, auto-configuration SPI, property binding, actuator endpoints. Cutover complexity is very high. Score: 1. Source: `ls /tmp/spring-boot/starter/` (171 entries). |
| 8 | Team Capability in Target Language | 2 | 3 | 6 | Inputs state 5 engineers full-time available. Target language unspecified. Assuming the team is Java-proficient (operating Spring Boot implies Java expertise). Score: 3 (moderate experience; could execute with some ramp-up if targeting a modern Java revision or a different framework). |
| 9 | Time Pressure | 2 | 4 | 8 | Cost-to-revenue ratio: $95,000 ÷ $1,000,000 = **9.5%**. Applicable threshold band: **5–10%** ("Low — costs are manageable; 6–12 months available"). This band maps directly to Score 4 per the agent spec threshold table. No operational health data indicates acute instability; financial inputs confirm no emergency. Source: `migration-inputs.md`. |
| 10 | Deployment & Cutover Simplicity | 2 | 2 | 4 | GitHub Actions CI matrix (`ci.yml`) tests across Linux/Windows × Java 17/21/25/26. No Dockerfiles in main source (one test-only Dockerfile). Deployment complexity for a framework library is high — consumers control deployment timing; coordinated cutover is required. Score: 2. |

**Raw weighted sum:** 2 + 6 + 15 + 6 + 9 + 6 + 2 + 6 + 8 + 4 = **64**

**Final score: 64 ÷ 25 = 2.56 — Hybrid**

> Per the scoring key: 2.5–3.4 = **Hybrid** — strangle the riskiest parts, refactor the rest.

---

## Financial Case

### Assumptions

| Assumption | Value | Basis |
|---|---|---|
| Average engineer cost (fully loaded) | $12,000/month | Industry midpoint for senior Java engineers |
| Team capacity | 5 engineers FTE | `migration-inputs.md` |
| Post-migration maintenance savings | 30% of maintenance costs | Conservative — modular refactor reduces complexity; does not eliminate it |
| Post-migration engineering savings | 20% of engineering costs | Conservative — cleaner codebase accelerates future work |
| Call center cost reduction | 10% | Marginal; primary driver is product quality, not architecture |
| Revenue at risk during migration | 0 (strangle & refactor) / $50,000/month (rewrite) | Rewrite risks 5% revenue during parallel-run period |

---

### Do Nothing (12-month projection)

| | |
|---|---|
| Monthly cost burden | $95,000 |
| 12-month total cost | $1,140,000 |
| Revenue preserved | $12,000,000 |
| Net position vs. revenue | Costs consume 9.5% of revenue |

**Trajectory:** Flat to slightly increasing. Spring Boot has a healthy open-source ecosystem — maintenance costs are unlikely to spike unless the team diverges from upstream significantly. However, failing to invest in modernisation increases the risk of a compounding debt spiral over 3–5 years. No immediate financial crisis, but opportunity cost of blocked features is unquantified.

---

### Strangle & Refactor

**Scope:** Incrementally replace the highest-complexity, highest-coupling modules (e.g. auto-configuration conditions, core `SpringApplication` bootstrap) while leaving stable modules in place. Introduce seams via interfaces; migrate module by module.

| Parameter | Value | Basis |
|---|---|---|
| Estimated effort | 12–18 engineer-months | 438 modules; 326k lines; complexity score 2; team of 5 |
| Duration at 5 FTE | 2.5–3.5 months elapsed | Parallel to existing work |
| Migration team cost | $60,000/month (5 × $12,000) | Replaces existing engineering cost; not additive if same team |
| Additional cost during migration | $0 net (team redirected) or +$60,000/month if augmented | Depends on whether migration team is additive |
| Monthly savings post-migration | $15,000 + $8,000 + $500 = **$23,500/month** | 30% maint + 20% eng + 10% CC |
| Break-even (additive team scenario) | ~8 months post-completion | ($60k/mo × 3mo migration cost) ÷ $23,500 savings/mo |
| Break-even (redirected team) | ~0 months | No additional cost incurred |
| 12-month net position | +$188,000 savings (redirected) / +$118,000 (additive, 9mo savings) | |
| 24-month net position | +$564,000 savings (redirected) / +$494,000 (additive) | |

**Risk:** Low. The comprehensive test suite (score 5) makes incremental refactoring safe. No big-bang cutover required.

---

### Rewrite

**Scope:** Full rewrite of Spring Boot (or the internal application built on Spring Boot — see Key Questions below). This section treats it as rewriting the internal system/application that depends on Spring Boot.

| Parameter | Value | Basis |
|---|---|---|
| Base effort estimate | 15–20 engineer-months | Proportional to codebase size; guided by test spec |
| Contingency multiplier | 1.5× | Applied per agent instructions for rewrites |
| Adjusted effort | 22–30 engineer-months | |
| Duration at 5 FTE | 4.5–6 months elapsed | |
| Ongoing costs during rewrite | $95,000/month (existing) + $60,000/month (rewrite team if additive) | Old system must run in parallel |
| Revenue at risk | $50,000/month (5% of revenue) | Risk of disruption during cutover window |
| Total cost during rewrite (additive, 6mo) | ($155,000 × 6) = $930,000 | |
| Monthly savings post-rewrite | $28,500/month (30% maint savings conservatively) | Rewrite savings front-loaded but higher risk |
| Break-even (additive) | ~33 months | $930,000 additional spend ÷ $28,500/mo |
| 12-month net position | –$588,000 (still in payback) | |
| 24-month net position | –$246,000 (still in payback) | |

**Risk:** High. Consumer coupling score of 1 (171 starter modules; millions of downstream users) means a rewrite carries significant contract-breakage risk. The 1.5× contingency multiplier is likely conservative for a framework of this scale.

---

### Scenario Comparison Summary

| Scenario | 12-Month Net | 24-Month Net | Risk |
|---|---|---|---|
| Do Nothing | –$1,140,000 (costs only) | –$2,280,000 (costs only) | Low short-term; rising long-term |
| Strangle & Refactor (redirected) | +$188,000 savings | +$564,000 savings | Low |
| Strangle & Refactor (additive team) | +$118,000 savings | +$494,000 savings | Low |
| Rewrite (additive team) | –$588,000 | –$246,000 | High |

---

## Recommendation

**Verdict: Hybrid — lean strongly toward Strangle & Refactor**

The technical score of **2.56** places the decision firmly in Hybrid territory, and the financial case reinforces the lower-risk path.

**Key drivers:**

1. **Test coverage is exceptional (score 5).** With 4,027 test files and comprehensive integration coverage, incremental refactoring is safe and verifiable. This is the single strongest argument against a rewrite — the tests already constitute a functional specification.

2. **Consumer coupling is the hardest constraint (score 1).** 171 starter modules and a public API consumed by a vast ecosystem mean a rewrite would require coordinated, multi-release migrations. The business cannot absorb this disruption.

3. **The financial case strongly favours Strangle & Refactor.** The rewrite does not break even within 24 months in any modelled scenario. Strangle & Refactor pays back within 8 months (additive) or immediately (redirected).

4. **No time pressure (score 4).** Cost-to-revenue ratio is 9.5% ($95,000 ÷ $1,000,000), falling in the **5–10% threshold band** ("Low — costs are manageable; 6–12 months available"). This is well below the 26–50% "High urgency" band that would force a faster decision. There is no financial emergency that would justify the risk of a rewrite.

5. **Technical debt is surface-level to moderate (score 2).** Large files exist (`SpringApplication.java` at 1,876 lines, `KafkaProperties.java` at 1,759 lines) but these are addressable in place. The 438-module Gradle structure demonstrates genuine modular discipline.

**The Hybrid approach recommended:**
- Begin with the highest-complexity modules: `SpringApplication`, `OnBeanCondition`, and the loader/JAR handling code (`ZipContent.java`, `NestedJarFile.java`).
- Introduce interface seams before refactoring; use the existing test suite as the regression harness.
- Defer any module that scores low on complexity and has stable consumers — those are working fine.
- Avoid a full rewrite unless a specific module is found to have no viable incremental path (none currently identified).

**Where the tension lies:** If the system described is not Spring Boot itself but an application *built on top of* Spring Boot, the analysis changes materially — the codebase complexity, test coverage, and consumer coupling of the application layer would need independent assessment. See Key Questions below.

---

## Key Questions for the Business

1. **Is the migration target Spring Boot the framework itself, or an application built using Spring Boot?** The inputs point to the Spring Boot GitHub repo, but the financial context ("key system," call center costs, $1M/month revenue) suggests an internal product. If it is an internal application, the codebase metrics above do not apply — the application layer must be analysed separately before any score can be trusted.

2. **What is the bus factor and onboarding time for each major module?** These fields were left blank. If fewer than 2 people understand the core bootstrap or autoconfigure modules, the functional understanding score (currently 2) may be too optimistic, which would push the overall recommendation further toward Strangle & Refactor over Rewrite.

3. **What is the current incident rate and change failure rate?** No operational health data was provided. If the system is experiencing frequent production incidents caused by the areas flagged for migration (e.g. complex conditional wiring), that would increase urgency (raising Dimension 9) and strengthen the case for acting on the highest-risk modules first.

4. **What is the target language or technology for the rewrite?** Team capability (Dimension 8) was scored at 3 assuming Java continuity. If the intended rewrite targets a different language or framework (e.g. moving away from Spring entirely), the team capability score could drop to 1–2, increasing the total score and potentially pushing toward Hybrid/Rewrite territory — but also raising execution risk significantly.

5. **What is the opportunity cost of the current architecture?** The inputs left this blank. If there are blocked features worth $50,000–$200,000/month in revenue that cannot be delivered due to architectural constraints, that materially shifts the financial case and could justify a more aggressive timeline for the Strangle & Refactor programme.

---

> Note: All financial estimates assume the migration team is the existing 5-engineer team (redirected capacity scenario). If additional engineers are hired, adjust the break-even calculations using the additive scenario figures above. All savings estimates are conservative; actual savings may be higher if incident rates and call center costs are more closely coupled to technical debt than the inputs reveal.
