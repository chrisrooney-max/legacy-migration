# Migration Decision — Human Inputs

Fill in this file before running the `migration-advisor` agent. The agent will combine your answers here with its own codebase analysis to produce a scored recommendation and financial case.

Leave any field as `Unknown` if you genuinely don't know — the agent will run a sensitivity analysis for those dimensions.

---

## The System

**System name:**
Spring Boot

**Codebase path:**
https://github.com/spring-projects/spring-boot

**Target output directory:**
documentationV5

**What is this system?**
Key system for us.

---

## Financials (Monthly)

**Maintenance costs (monthly):**
$50,000

**Engineering costs (monthly):**
$40,000

**Call center costs (monthly):**
$5,000

**Revenue (monthly):**
$1,000,000

---

## Team

**Team capacity available for migration:**
5 engineers full-time

**Risk tolerance:**
Moderate

**Bus factor (how many people understand each major module):**
<!-- e.g. "Most modules known by 2 people", "3 modules known only by 1 person" -->

**Onboarding time for new engineers:**
<!-- How long before a new engineer can make changes safely? e.g. "3 months", "6+ months" -->

**% of codebase understood by fewer than 2 people:**
<!-- Estimate. e.g. "~30%", "Unknown" -->

---

## Operational Health (Monthly)

**Incident rate:**
<!-- How many production incidents per month are caused by or involve this system? e.g. "8 per month" -->

**Mean time to recovery (MTTR):**
<!-- Average time to resolve an incident. e.g. "4 hours", "Unknown" -->

**Deployment frequency:**
<!-- How often is the system deployed to production? e.g. "Weekly", "Monthly", "Ad hoc" -->

**Change failure rate:**
<!-- What % of deployments cause an incident or rollback? e.g. "~15%", "Unknown" -->

**Lead time from commit to production:**
<!-- e.g. "2 weeks", "Same day", "Unknown" -->

---

## Financial Detail

**Cost per feature delivered (estimate):**
<!-- Engineering cost to deliver one average feature. e.g. "$8,000", "Unknown" -->

**Cost per bug fixed (estimate):**
<!-- Engineering + call center cost to resolve one bug end-to-end. e.g. "$2,500", "Unknown" -->

**Opportunity cost:**
<!-- Features or revenue the business is missing because the system can't support them. e.g. "$50,000/month in blocked features", "Unknown" -->
