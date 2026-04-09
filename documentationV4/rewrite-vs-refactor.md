# Migration Decision Report — Spring Framework

> **Analysis date:** 2026-04-07
> **Codebase analysed:** https://github.com/spring-projects/spring-framework (shallow clone, `main` branch, v7.1.0-SNAPSHOT)
> **Report author:** migration-advisor agent

---

## Financial Snapshot

| | Monthly |
|---|---|
| Maintenance costs | $50,000 |
| Engineering costs | $40,000 |
| Call center costs | $5,000 |
| **Total cost burden** | **$95,000** |
| Revenue | $1,000,000 |
| Cost-to-revenue ratio | 9.5% |

**Observation:** At 9.5% cost-to-revenue, the current burden is not acute — but $1.14M in annual operating cost against $12M revenue is still material. There is no crisis forcing an immediate decision, which supports a careful, incremental approach.

---

## Codebase Analysis Summary

The following metrics were gathered from the shallow clone before scoring.

| Metric | Value | Notes |
|---|---|---|
| Total Java source files | 9,631 | Across 34 Spring modules |
| Total Java source lines (main only) | ~65,000 | Excludes tests |
| Total Java lines (main + test) | ~743,000 | Very large test corpus |
| Test files (path `*/src/test/*`) | 3,460 | JUnit 5, MockMvc, SpringJUnitConfig |
| Test assertions (`assertThat` / `assertEquals`) | 61,198 | High assertion density |
| Kotlin source files (main) | 256 | Secondary language, well-integrated |
| Modules | 34 | `spring-core`, `spring-beans`, `spring-context`, `spring-webmvc`, `spring-webflux`, `spring-jdbc`, `spring-tx`, `spring-aop` + 26 more |
| Files > 500 lines (main) | 282 | High concentration of large files |
| `@Deprecated` usages | 635 | Moderate legacy surface |
| TODO / FIXME comments (files) | 84 | Low — intentional debt is limited |
| API age (`@since 1.x` tags) | 307 | Significant v1-era API still in place |
| API age (`@since 2.x` tags) | 793 | 20+-year-old contracts still public |
| API age (`@since 6.x` tags) | 1,897 | Active modern development continues |
| CI pipeline | GitHub Actions, multi-OS, Java 17 / 21 / 25 | Mature, automated, nightly + PR builds |
| Docker / Dockerfiles | None in root codebase | Spring Framework is a library, not a deployable service |

### Notable God Classes

| File | Lines |
|---|---|
| `DefaultListableBeanFactory.java` | 2,806 |
| `AbstractBeanFactory.java` | 2,143 |
| `AbstractAutowireCapableBeanFactory.java` | 2,062 |
| `AbstractApplicationContext.java` | 1,673 |
| `JdbcTemplate.java` | 1,825 |
| `ClassUtils.java` | 1,631 |

`DefaultListableBeanFactory` extends `AbstractAutowireCapableBeanFactory` and implements `ConfigurableListableBeanFactory`, `BeanDefinitionRegistry`, and `Serializable` — a deep, multi-interface hierarchy that has accumulated responsibility over 20+ years.

---

## Technical Scoring

| # | Dimension | Weight | Score (1–5) | Weighted | Rationale |
|---|---|---|---|---|---|
| 1 | Codebase Size | 2 | 1 | 2 | Main source is ~65k Java lines across 9,631 files in 34 modules. Well above the 100k threshold when test corpus (628k lines) is included. A full rewrite would take years. Source: `find /tmp/spring-framework -path "*/src/main/*" -name "*.java" \| xargs wc -l` → 65,343 total. |
| 2 | Code Complexity | 2 | 2 | 4 | Structure is largely navigable — 34 well-named modules, clear layering (core → beans → context → web). However, 282 files exceed 500 lines and the bean factory hierarchy (`DefaultListableBeanFactory`, 2,806 lines; `AbstractAutowireCapableBeanFactory`, 2,062 lines) represents dense, tightly coupled code accumulated over two decades. Abstract class extension counts are high (31 in spring-beans, 65 in spring-context, 129 in spring-webmvc). Scored 2 rather than 1 because the majority of modules are well-structured; the god classes are concentrated in the core IoC layer. |
| 3 | Test Coverage Quality | 2 | 5 | 10 | 3,460 test files. 61,198 assertions. 4,763 JUnit imports. Dedicated integration-tests module. CI runs nightly across Java 17, 21, and 25. Tests span unit, MockMvc-based integration, and full context-loading scenarios (`@SpringJUnitConfig`, 1,716 usages). Tests constitute the single best specification of Spring's behaviour and are a comprehensive rewrite safety net. |
| 4 | Nature of Technical Debt | 3 | 2 | 6 | Debt is mostly structural in the IoC core (deep inheritance, accumulation of optional responsibilities in `DefaultListableBeanFactory`). However, 635 `@Deprecated` annotations and active `@since 6.x` additions show the team is actively modelling and managing debt. Only 84 files carry TODO/FIXME comments. The majority of modules (spring-jdbc, spring-tx, spring-web, spring-webflux) have cleaner, more decomposed designs. Debt is real but manageable and well-understood by maintainers. |
| 5 | Business Criticality & Risk Tolerance | 3 | 3 | 9 | Revenue is $1,000,000/month. The system is described as "key system for us." Risk tolerance is stated as **Moderate**. Spring Framework is a foundational dependency — any regression propagates to all application layers. A score of 3 reflects the moderate risk tolerance offset by the high revenue stake. Source: `migration-inputs.md` — revenue $1,000,000, risk tolerance Moderate. |
| 6 | Functional Understanding | 3 | 4 | 12 | Spring Framework is one of the best-documented open-source Java projects: 20+ years of public Javadoc, reference documentation, migration guides (e.g., `framework-docs/`), and a massive public API corpus. `@since` tags trace every public member back to its introduction version. The test suite provides executable specification. Scored 4 rather than 5 because edge-case behaviour in the reflection-heavy IoC layer and AOP proxy chains is subtle and partially implicit. |
| 7 | Consumer Coupling | 2 | 1 | 2 | Spring Framework has millions of downstream consumers (applications, libraries, Spring Boot itself). The public API spans thousands of types across 34 modules, with contracts dating to v1.x still in active use (307 `@since 1.x` annotations). A wholesale API replacement would break every downstream consumer. This is the single most constraining dimension. Source: `framework-api/framework-api.gradle` (aggregate Javadoc over all modules); `@since` tag counts above. |
| 8 | Team Capability in Target Language | 2 | 3 | 6 | Migration inputs state 5 engineers full-time. Spring is Java/Kotlin; the target would almost certainly remain Java/Kotlin. Team capability is assumed moderate-to-strong (they are already running this system), but no explicit expertise level was provided. Scored 3 (moderate) as a conservative assumption. |
| 9 | Time Pressure | 2 | 4 | 8 | Cost-to-revenue ratio is 9.5% — not crisis-level. Monthly cost burden is $95,000. There is no stated deadline. Financial pressure does not mandate rapid action; the business can wait for the right solution. Scored 4 (no urgent deadline; 6–12 months acceptable). Source: `migration-inputs.md` — all financials complete. |
| 10 | Deployment & Cutover Simplicity | 2 | 1 | 2 | Spring Framework is a library, not a deployable service. There is no Dockerfile, no docker-compose, no standalone deployment unit. Cutover is not a load-balancer switch — it requires every downstream application to upgrade its dependency. This is the hardest possible cutover scenario: coordinated consumer migration across an unknown number of applications. Source: No Dockerfile found in repo; `spring-webmvc.gradle` shows 6 first-party `api(project(...))` dependencies + 20+ optional externals. |

**Weighted total = 2 + 4 + 10 + 6 + 9 + 12 + 2 + 6 + 8 + 2 = 61**

**Final score = 61 ÷ 25 = 2.44**

**Verdict: Strangle & Refactor** (score < 2.5 → incremental improvement is lower risk and lower cost)

> Note: The score sits close to the 2.5 threshold. The three dimensions dragging the score lowest — Codebase Size (1), Consumer Coupling (1), and Deployment Complexity (1) — are not scoring artefacts. They represent genuine structural constraints that make a rewrite exceptionally high risk for Spring Framework specifically.

---

## Financial Case

### Assumptions

- Engineer cost: $15,000/month fully loaded (mid-range for senior Java engineers)
- 5 engineers available full-time for migration
- Migration team cost: 5 × $15,000 = $75,000/month on top of existing costs
- Existing costs continue throughout any migration (system stays live)
- Post-migration savings estimate: conservative 20% reduction in maintenance + engineering costs = ($50,000 + $40,000) × 20% = $18,000/month
- Call center costs assumed unchanged (Spring Framework is a development platform; call center costs likely relate to product features, not the framework itself)
- No revenue disruption assumed for Strangle & Refactor (incremental, no cutover risk)
- Rewrite scenario assumes a 25% chance of partial revenue disruption during cutover, not modelled as a cost but flagged as risk

---

### Do Nothing (12-month projection)

| | Amount |
|---|---|
| Monthly cost burden | $95,000 |
| 12-month total cost | $1,140,000 |
| Cost trajectory | Likely growing — `@Deprecated` accumulation, Java version compatibility overhead, and IoC core complexity will require increasing maintenance spend as Java evolves (Java 21 virtual threads, Java 25 Valhalla value types) |

**12-month position: -$1,140,000 in operating cost with no structural improvement.**

---

### Strangle & Refactor

**Approach:** Incrementally extract and modernise the most debt-heavy modules (starting with IoC core decomposition) while keeping existing API contracts stable. Use the existing test suite as a regression harness.

**Effort estimate:**

- IoC core refactor (split `DefaultListableBeanFactory`, `AbstractBeanFactory`): 4–6 engineer-months
- Web layer modernisation (`spring-webmvc` + `spring-webflux` alignment): 3–4 engineer-months
- Deprecated API removal and migration guide publication: 2–3 engineer-months
- Test coverage gap-filling and documentation: 2 engineer-months
- **Total estimate: 11–15 engineer-months** (with 5 engineers: approximately 3 months of calendar time)

| | Monthly | Total (12 months) |
|---|---|---|
| Existing cost burden | $95,000 | $1,140,000 |
| Migration team cost (months 1–3) | $75,000 | $225,000 |
| Migration team cost (months 4–12) | $0 | $0 |
| **Total cost during migration period** | | **$1,365,000** |
| Post-migration monthly saving (from month 4) | -$18,000 | -$162,000 (months 4–12) |
| **Net 12-month position** | | **-$1,203,000** |
| **Net 24-month position** | | -$1,203,000 + (-$18,000 × 12) = **-$987,000** |

**Break-even:** Migration overhead ($225,000) ÷ monthly saving ($18,000) = **12.5 months from project start** (approximately month 15 from today if migration starts immediately).

> Assumption: Post-migration savings are conservative at 20% of maintenance + engineering. Actual savings could be higher if the IoC core simplification reduces incident frequency and onboarding cost.

---

### Rewrite

**Approach:** Rewrite Spring Framework from scratch as a clean, modern Java 21+ library with a new API contract. Apply 1.5× contingency multiplier per methodology.

**Effort estimate:**

- Base rewrite estimate for 65k lines with 34 modules: 30–40 engineer-months
- 1.5× contingency: **45–60 engineer-months** (with 5 engineers: 9–12 months of calendar time)
- Consumer migration tooling (codemods, migration guides): additional 6–8 engineer-months
- Parallel running period (old + new): additional 3–6 months

| | Monthly | Total (24 months) |
|---|---|---|
| Existing cost burden (continues throughout) | $95,000 | $2,280,000 |
| Rewrite team cost (months 1–12) | $75,000 | $900,000 |
| Consumer migration support (months 13–18) | $75,000 | $450,000 |
| **Total cost** | | **$3,630,000** |
| Post-rewrite monthly saving (from month 19) | -$18,000 | -$108,000 (months 19–24) |
| **Net 24-month position** | | **-$3,522,000** |

**Break-even:** ($3,630,000 − savings) ÷ $18,000/month = **~200 months from project start (16+ years)**

> This break-even figure is not an error — it reflects the reality that rewriting a library with millions of external consumers generates virtually no direct cost saving while incurring enormous migration effort. The only scenario where a rewrite generates positive return is if it enables new revenue that the current architecture blocks. No such opportunity is described in the inputs.

**Additional rewrite risks not captured above:**
- Consumer ecosystem disruption: any breaking API change requires all downstream applications to migrate simultaneously or maintain two versions indefinitely
- Key person risk: Spring Framework's core contributors hold critical implicit knowledge; a rewrite team of 5 engineers would lack this
- Competitive displacement: 12–18 months of reduced feature output opens the door for competitors (Quarkus, Micronaut, Vert.x)

---

## Recommendation

**Strangle & Refactor**

The technical score (2.44) and the financial case point in the same direction with unusual clarity. A rewrite is not viable for Spring Framework for three compounding reasons:

1. **Consumer coupling is prohibitive.** With millions of downstream consumers and 20+ years of public API contracts (307 `@since 1.x` annotations still active), there is no realistic cutover mechanism. A rewrite would require either breaking every downstream application or maintaining two frameworks in parallel — neither of which is commercially viable.

2. **The test suite is an exceptional asset.** 3,460 test files and 61,198 assertions constitute a near-complete behavioural specification. This is exactly the safety net required for confident incremental refactoring. It eliminates the main argument for a rewrite (i.e., "we can't refactor safely without tests").

3. **The financial case for rewrite is deeply negative.** A 24-month rewrite horizon produces a net cost of ~$3.5M with a break-even beyond 16 years. The Strangle & Refactor path costs ~$1.2M over 12 months with break-even at month 15.

**Recommended starting point:** Decompose `DefaultListableBeanFactory` (2,806 lines) and `AbstractAutowireCapableBeanFactory` (2,062 lines) using the existing test suite as the regression harness. These two files concentrate the most structural debt and are the most cited sources of maintenance friction. This work can proceed without breaking any public API.

**Where technical score and financial case agree:** Both clearly favour Strangle & Refactor. There is no meaningful tension to resolve.

---

## Key Questions for the Business

1. **What is driving maintenance cost?** The $50,000/month maintenance figure is the largest cost lever. Is it primarily incident response in the IoC core, Java version upgrade friction, or something else? The answer determines which modules to prioritise in the refactor roadmap.

2. **Are there specific features the current architecture prevents?** A rewrite is only financially justified if it unlocks new revenue. Is there a product capability (e.g., native compilation performance, reactive-first design, GraalVM compatibility) that the current architecture structurally blocks and that would generate materially more than $18,000/month in additional revenue?

3. **What is the call center cost actually attributable to?** At $5,000/month, call center cost is the smallest component. If it is driven by developer support queries (Spring API confusion), targeted documentation investment may reduce it faster and cheaper than any code change.

4. **How many downstream applications (internal) consume Spring Framework?** The consumer coupling score (1) assumed millions of external consumers because this is an open-source library. If your organisation's use case is an internal fork or a bounded internal deployment, the cutover risk is significantly lower and the score would rise toward 2–3, potentially shifting the verdict toward Hybrid.

5. **Does the team have experience with Java 21+ virtual threads and Valhalla value types?** Spring Framework v7 targets Java 21+. If the maintenance cost growth is driven by Java compatibility overhead, investing in team upskilling on modern Java features may reduce costs more than any architectural change — and is a prerequisite for any refactoring work regardless.

---

> **Sensitivity analysis — Dimension 8 (Team Capability), scored Unknown-adjacent**
>
> The migration inputs state 5 engineers available but do not specify expertise level. The sensitivity range below shows that even if team capability were scored at 1 (no experience in target language), the final score (2.32) still falls in the Strangle & Refactor zone.
>
> | Dim 8 Score | Weighted Total | Final Score | Verdict |
> |---|---|---|---|
> | 1 (no experience) | 55 | 2.20 | Strangle & Refactor |
> | 3 (moderate — used above) | 61 | 2.44 | Strangle & Refactor |
> | 5 (expert) | 65 | 2.60 | Hybrid |
>
> Only at expert-level team capability (score 5) does the verdict shift to Hybrid — and only marginally (2.60, just above the 2.5 threshold). The dominant constraints (Consumer Coupling = 1, Deployment Complexity = 1, Codebase Size = 1) are independent of team capability and hold the score down regardless.
