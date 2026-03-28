---
name: document-module
description: Generate documentation for a specific module, class, or file in the legacy codebase
---

Analyse the module at $ARGUMENTS and write the output to `modules/<module-name>.md`. Add an entry to `modules/index.md`.

Follow this structure exactly:

```
# Module: <ClassName or ModuleName>

> **Package:** <package path> | **File:** <filename> | **Lines:** <line count>

One or two sentences describing the module's purpose and why it exists.

---

## Responsibilities

Bullet list of what this module is responsible for.

---

## Public Interface

Table with columns: Method | Signature | Description

List only the public methods/functions. If there are none beyond the constructor, say so.

---

## Dependencies

**Internal:**
Table with columns: Dependency | Purpose

**External:**
Table with columns: Library | Version | Purpose

---

## Inputs and Outputs

**Input:** describe what the module receives (type, shape, constraints)
**Output:** describe what it returns (type, shape)
**Side effects:** list any (DB writes, queue publishes, file I/O, etc.). If none, omit this section.

---

## Known Issues / Technical Debt

Bullet list of specific problems. Each item should name the exact method, line range, or field where the issue exists where possible.
```

**Formatting rules:**
- Use `> ⚠️ Unclear:` callouts for anything ambiguous or that cannot be confirmed from the code alone
- Use tables for interfaces and dependencies
- Be factual — only document what you can observe in the code
- Do not invent behaviour. If a method's purpose is not clear from code or naming, mark it unclear.
