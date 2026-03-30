---
name: document-code-structure
description: Generate code structure documentation covering repository layout, modules, patterns, and areas of concern
---

Analyse the codebase at $ARGUMENTS and write the output to `code-structure/index.md`.

Use the template at `.claude/skills/templates/03-code-structure.md` as the exact output structure. Fill in every section based on what you can observe in the source files, build config, and directory layout.

**Section guidance:**
- **Repository Overview** — repo name from git remote or package.json/build.gradle; one-line purpose
- **Directory Structure** — produce an actual tree of the real directories, not the placeholder in the template
- **Key Modules** — fill the table; entry points are the public-facing methods, routes, or main classes per module
- **Entry Points** — list the actual file paths and class/function names where execution begins
- **Configuration** — list actual config files found (e.g. `application.properties`, `.env.example`) and the env vars they define
- **Coding Patterns** — identify patterns present in the code (e.g. repository pattern, handler pattern, DI, functional style)
- **Dependencies** — internal: shared packages or modules; external: key third-party libraries with versions
- **Areas of Concern** — modules with high complexity, long files, missing tests, or heavy coupling

**Formatting rules:**
- Use `> ⚠️ Unclear:` for anything that cannot be confirmed from the code alone
- Use the key modules table — do not replace with prose
- Directory structure must reflect the actual repo, not a generic placeholder
- Be factual — only document what you can observe
