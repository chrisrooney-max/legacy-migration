---
name: document-module
description: Generate documentation for a specific module, class, or file in the legacy codebase
---

Analyse the module at $ARGUMENTS and produce documentation covering:

1. **Purpose and responsibilities** — what does this module do and why does it exist?
2. **Key functions/classes** — list each with a brief description of what it does
3. **Dependencies** — internal modules it relies on and external libraries it uses
4. **Inputs and outputs** — what does it consume and what does it produce?
5. **Known issues or technical debt** — anything that looks fragile, unclear, or outdated

Write the output to `modules/<module-name>.md` and add an entry to `modules/index.md`.

Be factual — only document what you can observe in the code. Mark anything unclear with a `> ⚠️ Unclear:` callout.
