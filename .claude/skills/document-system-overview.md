---
name: document-system-overview
description: Generate a high-level system overview covering purpose, capabilities, users, criticality, and ownership
---

Analyse the codebase at $ARGUMENTS and write the output to `system-overview/index.md`.

Use the template at `.claude/skills/templates/01-system-overview.md` as the exact output structure. Fill in every section based on what you can observe in the code, configuration files, README, and build files.

**Section guidance:**
- **Purpose** — derive from README, entry points, and top-level package names
- **Key Capabilities** — infer from route definitions, public interfaces, and feature directories
- **Primary Users** — infer from auth roles, UI flows, or API consumer patterns
- **Business Criticality** — check the one box most supported by evidence (e.g. payment handling = mission critical)
- **Current State** — assess stability from test coverage, open TODOs, error handling quality, and dependency age
- **High-Level Risks** — surface the top risks only; detail belongs in `07-change-risk.md`
- **Ownership** — populate only if found in CODEOWNERS, README, or package metadata; otherwise leave blank
- **Related Systems** — infer from integration code, config URLs, and import statements

**Formatting rules:**
- Use `> ⚠️ Unclear:` for anything that cannot be confirmed from the code alone
- Be factual — do not invent details not present in the codebase
- Keep each bullet concise — this is an orientation document, not exhaustive detail
