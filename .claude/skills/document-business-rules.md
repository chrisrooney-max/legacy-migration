---
name: document-business-rules
description: Generate business rules documentation covering core rules, workflows, validations, edge cases, and inconsistencies
---

Analyse the codebase at $ARGUMENTS and write the output to `business-rules/index.md`.

Use the template at `.claude/skills/templates/10-business-rules.md` as the exact output structure. Fill in every section based on what you can observe in validation logic, service classes, domain models, and test scenarios.

**Section guidance:**
- **Overview** — one paragraph summarising the nature and complexity of the business rules (e.g. simple CRUD vs complex rule engine)
- **Core Rules** — fill the table with each distinct business rule; location in code should be the specific class and method name; flag rules that exist only in stored procedures or external systems
- **Workflows** — describe major multi-step business processes (e.g. order submission flow, user registration flow) as numbered steps
- **Validations** — list input validation rules with their location; distinguish between format validation and business rule validation
- **Edge Cases** — conditions that trigger special handling, found in if/else branches, switch statements, or feature flags
- **Exceptions** — error handling logic specific to business rule violations (vs technical errors)
- **Time-Based Logic** — any scheduling, expiry, timezone handling, or date-dependent behaviour
- **Inconsistencies** — places where the implemented behaviour appears to contradict comments, tests, or other parts of the code
- **Risks** — rules with high complexity, no tests, or that exist in multiple places with potential for divergence

**Formatting rules:**
- Use `> ⚠️ Unclear:` for rules whose intent cannot be determined from the code alone
- Use the core rules table — do not replace with bullet lists
- Be factual — only document rules you can confirm from the code; do not invent business logic
- Note when logic appears to live outside the codebase (e.g. in a database stored procedure or external service)
