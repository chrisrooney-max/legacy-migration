---
name: summarise-codebase
description: Produce a quick high-level overview of a codebase before full documentation begins
---

Analyse the codebase at $ARGUMENTS and produce a concise high-level summary covering:

1. **What it does** — one paragraph describing the system's purpose
2. **Tech stack** — languages, frameworks, databases, and infrastructure
3. **Top-level structure** — major directories and what lives in each
4. **Key components** — the most important modules or services and how they relate
5. **Entry points** — where execution starts (e.g. main files, routes, handlers)
6. **Immediate observations** — anything notable about code quality, patterns, or areas of concern

Write the output to `architecture/overview.md`.

Keep it concise — this is a quick orientation, not exhaustive documentation. Mark anything unclear with a `> ⚠️ Unclear:` callout.
