---
name: create-epics
description: Analyse a codebase or its output documentation and create GitHub issues as epics covering all functional areas that need to be tackled
---

Analyse the codebase or output documentation at $ARGUMENTS and create GitHub epics as issues in the GitHub repository.

## Your Task

1. **Identify functional areas** — read the output docs in `output/` (if available) or analyse the codebase directly to identify the distinct functional areas that need to be tackled (e.g. for a rewrite: project setup, data layer, each handler group, testing, deployment)

2. **Draft one epic per functional area** — use the template at `.claude/skills/templates/epic.md` as the structure for each epic. Each epic should be:
   - Focused on one coherent functional area
   - Specific enough to be actionable
   - Sized so it could be broken into user stories (not too granular, not too broad)

3. **Order by dependency** — identify which epics depend on others and reflect this in the Dependencies section of each epic

4. **Create GitHub issues** — for each epic, run:
   ```bash
   gh issue create --repo <owner/repo> --label "epic" --title "Epic: <title>" --body "<body>"
   ```
   Create the `epic` label first if it does not exist:
   ```bash
   gh label create "epic" --repo <owner/repo> --color "0075ca" --description "Large body of work"
   ```

5. **Report results** — list each epic title and its GitHub issue URL

## What Makes a Good Epic

- **One coherent concern** — e.g. "Data Layer", not "Data Layer and Auth and Deployment"
- **Clear scope** — explicit about what is and is not included
- **Testable acceptance criteria** — each criterion should be verifiable
- **Grounded in the codebase** — reference specific files, modules, or output docs; do not invent scope
- **Correct dependency order** — infrastructure epics before feature epics; data layer before handlers

## Formatting Rules

- Use the epic template exactly — do not add or remove sections
- Acceptance criteria must use `- [ ]` checkbox format
- Reference section must link to actual files that exist in the repo or codebase
- Be factual — only include scope items that are justified by the codebase analysis
