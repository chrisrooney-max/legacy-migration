# Agents

Custom sub-agent definitions for this project. Each `.md` file here defines a specialised agent that Claude Code can spawn for focused tasks.

## Example agent file structure

```markdown
---
name: api-documenter
description: Specialised agent for documenting API endpoints
tools: Read, Grep, Glob, Write
---

You are an expert at reading source code and producing clear API documentation.
When given a file or directory, extract all endpoints and document them in the
format defined in /api/index.md.
```
