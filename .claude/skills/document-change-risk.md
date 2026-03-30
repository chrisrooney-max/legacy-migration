---
name: document-change-risk
description: Generate change risk and technical debt documentation covering high-risk areas, coupling, debt, and recommendations
---

Analyse the codebase at $ARGUMENTS and write the output to `change-risk/index.md`.

Use the template at `.claude/skills/templates/07-change-risk.md` as the exact output structure. Fill in every section based on what you can observe in the code quality, dependency versions, test coverage, and coupling patterns.

**Section guidance:**
- **Overview** — one paragraph summarising the overall risk profile (low/medium/high and why)
- **High-Risk Areas** — fill the table with specific files, classes, or modules that are risky to change; reason should be concrete (e.g. "no tests", "called by 8 other modules", "undocumented side effects")
- **Known Fragile Components** — components where a small change is likely to break something elsewhere
- **Coupling Issues** — identify tight coupling: circular dependencies, shared mutable state, god classes
- **Technical Debt** — fill the table with specific debt items; priority is High / Medium / Low
- **Obsolete Technology** — libraries or frameworks that are end-of-life, unmaintained, or significantly out of date
- **Safe-to-Change Areas** — well-tested, well-isolated modules that can be modified with low risk
- **Recommendations** — specific, actionable suggestions ordered by impact

**Formatting rules:**
- Use `> ⚠️ Unclear:` for risk assessments that cannot be fully confirmed from the code alone
- Use the provided tables — do not replace with bullet lists
- Be specific — name the exact file, class, or method; avoid vague statements like "the codebase has high coupling"
- Be factual — only flag risks you can observe in the code
