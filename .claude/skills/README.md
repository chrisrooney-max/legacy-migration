# Skills

Custom slash commands for this project. Each `.md` file here defines a skill invocable via `/skill-name` in Claude Code.

## Example skill file structure

```markdown
---
name: document-module
description: Generate documentation for a given module
---

Analyze the module at $ARGUMENTS and produce a summary covering:
1. Purpose and responsibilities
2. Key functions/classes
3. Dependencies
4. Gotchas or technical debt
```
