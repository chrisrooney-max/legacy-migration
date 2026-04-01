---
name: assess-migration
description: Score a codebase against the rewrite vs strangle & refactor decision framework and produce a weighted recommendation
---

Analyse the codebase at $ARGUMENTS and write the output to `output/rewrite-vs-refactor.md`.

Use the template at `.claude/skills/templates/rewrite-vs-refactor.md` as the exact output structure.

## Your Task

Score the codebase against all 10 dimensions in the template. For each dimension:
1. Read the score definitions in the template
2. Assess the codebase using evidence you can observe in the code, tests, config, and build files
3. Assign a score of 1–5
4. Write a concise rationale citing specific files, classes, or patterns as evidence

Then:
- Calculate the weighted score (score × weight for each dimension)
- Sum the weighted scores and divide by 25 to get the final score
- If any dimension cannot be determined from the code alone (e.g. team capability, time pressure), mark it as `⚠️ Unknown`, exclude it from the calculation, and run a sensitivity analysis showing how different values would change the result
- State the recommendation clearly based on the final score
- Write 3–5 specific questions the business should answer before committing to a decision

## Evidence to Look For

| Dimension | Where to look |
|---|---|
| Codebase Size | Count source files and lines; check directory structure |
| Code Complexity | Look for god classes, deep nesting, cyclomatic complexity, tight coupling |
| Test Coverage | Count test files; identify test types; look for untested handlers or modules |
| Nature of Debt | Distinguish structural debt (architecture, dependencies) from surface debt (naming, duplication) |
| Business Criticality | Look for SLA indicators, README descriptions, health check presence, rate limiting |
| Functional Understanding | Assess completeness of test specs, documentation, and comments |
| Consumer Coupling | Count external consumers from API definitions, CORS config, client references |
| Team Capability | Mark as unknown unless team info is present in CODEOWNERS or README |
| Time Pressure | Mark as unknown unless deadlines are referenced in docs or comments |
| Deployment Simplicity | Assess from Dockerfile, docker-compose, CI config, statelessness of service |

## Formatting Rules

- Use `> ⚠️ Unclear:` for any score that cannot be determined from the code alone
- Every score must have a rationale citing specific evidence — do not assign scores without justification
- Be factual — do not inflate or deflate scores; the recommendation is only useful if the scoring is honest
