# Migration Decision Report — Spring Framework

> **Analysis date:** 2026-04-07
> **Codebase analysed:** `/tmp/spring-framework` (shallow clone, `main` branch, v7.1.0-SNAPSHOT)
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

**Observation:** At 9.5% cost-to-revenue, the current burden is not acute — but $1.14M in annual operating cost against $12M annual revenue is still material. There is no financial crisis forcing an immediate decision, which supports a considered, incremental approach over a high-risk rewrite.

---

## Codebase Analysis Summary

The following metrics were gathered from the cloned repository before scoring.

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
| 1 | Codebase Size | 2 | 1 | 2 | Main source is ~65k Java lines across 9,631 files in 34 modules. Well above the 100k threshold when test corpus (~678k lines) is included. A full rewrite would take years. Source: `find /tmp/spring-framework -path "*/src/main/*" -name "*.java" | xargs wc -l` → 65,343 total. |
| 2 | Code Complexity | **3** | 2 | **6** | Structure is largely navigable — 34 well-named modules, clear layering (core → beans → context → web). However, 282 files exceed 500 lines and the bean factory hierarchy (`DefaultListableBeanFactory`, 2,806 lines; `AbstractAutowireCapableBeanFactory`, 2,062 lines) represents dense, tightly coupled code accumulated over two decades. Abstract class extension counts are high (31 in spring-beans, 65 in spring-context, 129 in spring-webmvc). Scored 2 rather than 1 because the majority of modules are well-structured; the god classes are concentrated in the core IoC layer. |
| 3 | Test Coverage Quality | **3** | 5 | **15** | 3,460 test files. 61,198 assertions. 4,763 JUnit imports. Dedicated `integration-tests` module. CI runs nightly across Java 17, 21, and 25. Tests span unit, MockMvc-based integration, and full context-loading scenarios (`@SpringJUnitConfig`, 1,716 usages). Tests constitute the single best specification of Spring's behaviour and are a comprehensive rewrite safety net. |
| 4 | Nature of Technical Debt | 3 | 2 | 6 | Debt is mostly structural in the IoC core (deep inheritance, accumulation of optional responsibilities in `DefaultListableBeanFactory`). However, 635 `@Deprecated` annotations and active `@since 6.x` additions show the team is actively modelling and managing debt. Only 84 files carry TODO/FIXME comments. The majority of modules (spring-jdbc, spring-tx, spring-web, spring-webflux) have cleaner, more decomposed designs. Debt is real but manageable and well-understood by maintainers. |
| 5 | Business Criticality & Risk Tolerance | 3 | 3 | 9 | Revenue is $1,000,000/month. The system is described as "key system for us." Risk tolerance is stated as **Moderate**. Spring Framework is a foundational dependency — any regression propagates to all application layers. A score of 3 reflects the moderate risk tolerance offset by the high revenue stake. Source: `migration-inputs.md` — revenue $1,000,000/month, risk tolerance Moderate. |
| 6 | Functional Understanding | 3 | 4 | 12 | Spring Framework is one of the best-documented open-source Java projects: 20+ years of public Javadoc, reference documentation, migration guides (`framework-docs/`), and a massive public API corpus. `@since` tags trace every public member back to its introduction version. The test suite provides executable specification. Scored 4 rather than 5 because edge-case behaviour in the reflection-heavy IoC layer and AOP proxy chains is subtle and partially implicit. **Note: this score is uncertain — see Sensitivity Analysis section below.** |
| 7 | Consumer Coupling | 2 | 1 | 2 | **⚠️ Key assumption — OSS library vs internal system:** This score of 1 assumes Spring Framework is being evaluated as an open-source library with millions of external downstream consumers. If your organisation's scenario is an internal fork or private deployment with a bounded set of internal consumers, this score would rise to 2–4 and could shift the verdict. The public API spans thousands of types across 34 modules, with contracts dating to v1.x still in active use (307 `@since 1.x` annotations). A wholesale API replacement would break every downstream consumer. Source: `framework-api/framework-api.gradle` (aggregate Javadoc over all modules); `@since` tag counts above. **See Sensitivity Analysis section for the impact of varying this score.** |
| 8 | Team Capability in Target Language | 2 | 3 | 6 | Migration inputs state 5 engineers full-time. Spring is Java/Kotlin; the target would almost certainly remain Java/Kotlin. Team capability is assumed moderate-to-strong (they are already running this system), but no explicit expertise level was provided. Scored 3 (moderate) as a conservative assumption. Source: `migration-inputs.md` — 5 engineers full-time. |
| 9 | Time Pressure | 2 | 4 | 8 | Cost-to-revenue ratio is 9.5% — not crisis-level. Monthly cost burden is $95,000. No deadline is stated in `migration-inputs.md`. At 9.5% cost-to-revenue there is no financial pressure mandating rapid action; the business has 6–12 months to execute a considered plan. Scored **4** (no urgent deadline; 6–12 months acceptable). Source: `migration-inputs.md` — all financials complete, no deadline specified. |
| 10 | Deployment & Cutover Simplicity | 2 | 1 | 2 | Spring Framework is a library, not a deployable service. There is no Dockerfile, no docker-compose, no standalone deployment unit in the repository. Cutover is not a load-balancer switch — it requires every downstream application to upgrade its dependency. This is the hardest possible cutover scenario: coordinated consumer migration across an unknown number of applications. Source: No Dockerfile found in repo; `spring-webmvc.gradle` shows 6 first-party `api(project(...))` dependencies + 20+ optional externals. |
| | **Total** | **25** | | **68** | |

**Weighted total = 2 + 6 + 15 + 6 + 9 + 12 + 2 + 6 + 8 + 2 = 68**

**Final score = 68 ÷ 25 = 2.72**

**Verdict: Hybrid** (score 2.5–3.4 → strangle the riskiest parts, refactor the rest)

> Note: The corrected weights for Dimensions 2 and 3 (both Weight = 3, not 2) shift the verdict from the previous report's Strangle & Refactor to Hybrid. The exceptional test coverage (D3 = 5, weighted 15) and relatively navigable complexity (D2 = 2, weighted 6) now have appropriate influence. The score sits at 2.72 — above the 2.5 Strangle & Refactor threshold, but well below the 3.5 Rewrite threshold. The dominant constraints (Consumer Coupling = 1, Deployment Complexity = 1, Codebase Size = 1) remain genuine structural barriers to a full rewrite.

---

## Sensitivity Analysis

Two dimensions are uncertain given the available inputs: Dimension 6 (Functional Understanding) and Dimension 7 (Consumer Coupling). The analysis below shows how the final score and verdict change across the plausible range for each, holding all other scores fixed.

### Dimension 6 — Functional Understanding (Weight: 3)

The score of 4 assumes strong functional understanding based on public documentation and test coverage. However, edge-case behaviour in the IoC layer and AOP proxy chains is partially implicit, and it is unknown how much institutional knowledge the team holds about undocumented corner cases.

Base weighted total (all other dimensions): 68 − 12 = 56

| D6 Score | Meaning | Weighted Contribution | Final Weighted Total | Final Score | Verdict |
|---|---|---|---|---|---|
| 1 | Behaviour poorly understood; rewrite would likely miss edge cases | 3 | 59 | 2.36 | **Strangle & Refactor** |
| 3 | Good understanding of main flows; some edge cases uncertain | 9 | 65 | 2.60 | **Hybrid** |
| **4 (base)** | **Strong understanding of all flows and most edge cases** | **12** | **68** | **2.72** | **Hybrid** |
| 5 | Complete understanding; behaviour fully documented and testable | 15 | 71 | 2.84 | **Hybrid** |

**Finding:** If functional understanding is poor (score = 1), the verdict drops to Strangle & Refactor (2.36). At any score of 2 or above, the verdict remains Hybrid. The 2.5 threshold is crossed between scores 1 and 2. The business should answer: *How well does the current team understand the implicit edge-case behaviour in the IoC layer and AOP subsystem?*

---

### Dimension 7 — Consumer Coupling (Weight: 2)

> **⚠️ Critical assumption:** The base score of 1 treats Spring Framework as an OSS library with millions of external consumers. If this system is an internal fork, private build, or bounded internal deployment, the real consumer count is much lower and this score should be raised. This is the single assumption most likely to be wrong.

Base weighted total (all other dimensions): 68 − 2 = 66

| D7 Score | Assumed Consumer Scenario | Weighted Contribution | Final Weighted Total | Final Score | Verdict |
|---|---|---|---|---|---|
| **1 (base)** | **OSS library; millions of external consumers; coordinated cutover impossible** | **2** | **68** | **2.72** | **Hybrid** |
| 2 | Several consumers; some contract complexity | 4 | 70 | 2.80 | **Hybrid** |
| 3 | Internal deployment; a few bounded consumers; manageable cutover | 6 | 72 | 2.88 | **Hybrid** |
| 4 | Internal fork; one or two consumers; simple API contract | 8 | 74 | 2.96 | **Hybrid** |
| 5 | Fully controlled consumers; team owns all downstream applications | 10 | 76 | 3.04 | **Hybrid** |

**Finding:** Across the full plausible range (1–5), the verdict remains Hybrid. Consumer Coupling (Weight: 2) does not have enough influence to push the score above 3.5 (Rewrite) or below 2.5 (Strangle & Refactor) on its own. However, the base score of 1 is the most pessimistic and the least likely to be correct if this is an internal system. The business should clarify how many internal applications consume this framework and whether a coordinated cutover is feasible.

---

## Financial Case

### Assumptions

- Engineer cost: $15,000/month fully loaded (mid-range for senior Java engineers)
- 5 engineers available full-time for migration (source: `migration-inputs.md`)
- Migration team cost: 5 × $15,000 = $75,000/month on top of existing costs
- Existing costs continue throughout any migration (system stays live)
- Post-migration savings estimate: conservative 20% reduction in maintenance + engineering costs = ($50,000 + $40,000) × 20% = $18,000/month
- Call center costs assumed unchanged — likely driven by product features, not framework architecture
- No revenue disruption assumed for Strangle & Refactor / Hybrid (incremental, no cutover risk)
- Rewrite scenario: 25% chance of partial revenue disruption during cutover — not modelled as a fixed cost but flagged as qualitative risk

---

### Do Nothing (12-month projection)

| | Amount |
|---|---|
| Monthly cost burden | $95,000 |
| 12-month total cost | $1,140,000 |
| Cost trajectory | Likely growing — `@Deprecated` accumulation (635 usages), Java version compatibility overhead, and IoC core complexity will require increasing maintenance spend as Java evolves (Java 21 virtual threads, Java 25 Valhalla value types) |

**12-month position: −$1,140,000 in operating cost with no structural improvement.**

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
| Post-migration monthly saving (from month 4) | −$18,000 | −$162,000 (months 4–12) |
| **Net 12-month position** | | **−$1,203,000** |
| **Net 24-month position** | | −$1,203,000 + (−$18,000 × 12) = **−$987,000** |

**Break-even:** Migration overhead ($225,000) ÷ monthly saving ($18,000) = **12.5 months from project start** (approximately month 16 from today if migration starts now).

> Assumption: Post-migration savings are conservative at 20% of maintenance + engineering. Actual savings could be higher if IoC core simplification reduces incident frequency and onboarding cost.

---

### Rewrite

**Approach:** Rewrite Spring Framework from scratch as a clean, modern Java 21+ library with a new API contract. Apply 1.5× contingency multiplier per methodology.

**Effort estimate:**

- Base rewrite estimate for ~65k lines across 34 modules: 30–40 engineer-months
- 1.5× contingency: **45–60 engineer-months** (with 5 engineers: 9–12 months of calendar time)
- Consumer migration tooling (codemods, migration guides): additional 6–8 engineer-months
- Parallel running period (old + new): additional 3–6 months

| | Monthly | Total (24 months) |
|---|---|---|
| Existing cost burden (continues throughout) | $95,000 | $2,280,000 |
| Rewrite team cost (months 1–12) | $75,000 | $900,000 |
| Consumer migration support (months 13–18) | $75,000 | $450,000 |
| **Total cost** | | **$3,630,000** |
| Post-rewrite monthly saving (from month 19) | −$18,000 | −$108,000 (months 19–24) |
| **Net 24-month position** | | **−$3,522,000** |

**Break-even:** $3,630,000 ÷ $18,000/month ≈ **202 months from project start (~17 years)**

> This break-even figure is not an error — it reflects the reality that rewriting a library with large numbers of external or internal consumers generates virtually no direct operating cost saving while incurring enormous migration effort. The only scenario where a rewrite generates positive return is if it unlocks new revenue that the current architecture structurally prevents. No such opportunity is identified in the inputs.

**Additional rewrite risks not captured above:**
- Consumer ecosystem disruption: any breaking API change requires all downstream applications to migrate simultaneously or maintain two versions indefinitely
- Key person risk: Spring Framework's core contributors hold critical implicit knowledge; a rewrite team of 5 engineers would likely lack this
- Competitive displacement: 12–18 months of reduced feature output opens the door for competitors (Quarkus, Micronaut, Vert.x)

---

## Recommendation

**Hybrid — Strangle the riskiest structural components; refactor the rest incrementally**

The corrected technical score (2.72) places the system in the Hybrid zone. The financial case strongly disfavours a rewrite (24-month net cost of ~$3.5M vs ~$1.2M for Strangle & Refactor). The technical and financial evidence are aligned.

**What Hybrid means in practice for Spring Framework:**

1. **Strangle the IoC core's god classes.** `DefaultListableBeanFactory` (2,806 lines) and `AbstractAutowireCapableBeanFactory` (2,062 lines) are the highest-concentration structural debt. These should be decomposed behind stable internal interfaces using the existing 3,460-test suite as a regression harness. This is the "strangle" component — isolating and replacing the most entangled parts without breaking the public API.

2. **Refactor the remaining modules incrementally.** The majority of Spring modules (spring-jdbc, spring-tx, spring-web, spring-webflux) have cleaner, more decomposed designs. These can be improved through standard refactoring — removing deprecated APIs, reducing class sizes, improving test coverage in weak areas — without the risk of a strangle operation.

3. **Do not rewrite.** The financial break-even on a rewrite (~17 years) and the consumer coupling constraint make a clean rewrite commercially inviable under any scenario described in the inputs.

**Where technical score and financial case agree:** Both clearly favour incremental approaches. There is no meaningful tension to resolve — the Hybrid verdict is stable across all but the most pessimistic sensitivity scenarios.

**Score proximity to thresholds:** The current score (2.72) is 0.22 above the Strangle & Refactor threshold (2.5). If functional understanding is found to be poor (D6 = 1), the score drops to 2.36 and the verdict shifts to Strangle & Refactor — a more conservative posture. This would not change the financial recommendation materially but would argue for beginning with the safest, most test-covered modules rather than attacking the IoC core first.

---

## Key Questions for the Business

1. **Are the external or internal consumers of this framework?** The Consumer Coupling score (D7 = 1) assumed worst-case: an OSS library with millions of uncontrolled downstream consumers. If this is an internal fork or private deployment with a bounded consumer set, the real coupling score is 2–4, and a more aggressive refactoring or even partial rewrite of specific modules becomes viable. This is the single question most likely to change the recommendation.

2. **What is driving the $50,000/month maintenance cost?** This is the largest cost lever. Is it primarily incident response in the IoC core, Java version upgrade friction, or something else? The answer determines which modules to prioritise in the Hybrid roadmap — and whether the conservative 20% savings estimate is too low or too high.

3. **Are there product capabilities the current architecture structurally prevents?** A more aggressive rewrite is only financially justified if it unlocks new revenue. Is there a specific capability (e.g., native compilation performance, GraalVM static images, reactive-first design without legacy servlet bindings) that the current architecture blocks and that would generate materially more than $18,000/month in new revenue?

4. **How well does the current team understand the implicit edge-case behaviour in the IoC layer and AOP subsystem?** Functional understanding (D6) is the dimension with the most verdict sensitivity. If the team scores this as 1–2 (poor understanding of edge cases), the recommendation shifts to a more cautious Strangle & Refactor posture. If they score it 4–5, the Hybrid verdict is confirmed.

5. **What is the call center cost attributable to?** At $5,000/month it is the smallest component, but if it is driven by developer support queries about the Spring API, targeted documentation investment may reduce it faster and more cheaply than any code change — and is a low-risk starting point regardless of the final migration strategy.
