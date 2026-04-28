---
name: analyse
description: Run the full analysis pipeline against a codebase — documents it across all 10 doc types, runs the migration advisor, then synthesises all outputs into a rebuild PRD.
---

Run the full analysis pipeline against the codebase at $ARGUMENTS.

## Step 1 — Document the codebase

Spawn the `code-documenter` agent against the codebase path provided. Wait for it to complete all 10 documents before proceeding.

The agent will write the 10 spec documents to `analysis-output/V{n}/`, where `V{n}` is the next available version number (e.g. if `analysis-output/V3/` exists, use `analysis-output/V4/`). Check `context/migration-inputs.md` for any explicit version override.

Do not proceed to Step 2 until all 10 documents have been written successfully.

## Step 2 — Run the migration advisor

Once documentation is complete, automatically spawn the `migration-advisor` agent.

The advisor will read `context/migration-inputs.md` for financial inputs and the codebase path. It will write its report (`rewrite-vs-refactor.md`) to `final-docs/V{n}/`, using the same version number as the `analysis-output/V{n}/` directory from Step 1.

Do not proceed to Step 3 until the migration advisor has written its report.

## Step 3 — Produce the rebuild PRD

Once both Step 1 and Step 2 are complete, synthesise all outputs into a PRD using the template at `.claude/skills/templates/11-prd.md`.

Read the following documents as your source material:
- All 10 spec docs from `analysis-output/V{n}/`
- `final-docs/V{n}/rewrite-vs-refactor.md`

Adapt the template fields to codebase analysis context:
- **Research Overview / Participants table** → list the 10 spec documents and migration report as analysis sources
- **Key Themes** → patterns that recur across multiple spec documents (e.g. tight coupling appearing in architecture + change-risk + testing)
- **Pain Points** → draw directly from `change-risk-spec.md`, `testing-spec.md`, and `security-spec.md`; rank by severity
- **User Needs** → what the rebuilt system must/should/could address, grounded in the spec findings
- **Proposed Features** → capabilities the rebuilt system needs, written as user stories; derive from business-rules, integrations, and system-overview
- **Recommendations** → align with the verdict from `rewrite-vs-refactor.md`
- **Open Questions** → unresolved items flagged with `> ⚠️ Unclear:` across the spec docs

Write the completed PRD to `prd-output/V{n}/prd.md`, using the same version number as the `analysis-output/V{n}/` directory from Step 1.

Every claim in the PRD must cite a specific source document and section. Do not invent scope.

## Step 4 — Confirm completion

When all three steps have finished, report back to the user with:
- The output directories where all files were written
- A one-line summary of the migration advisor verdict (score and recommendation)
- The number of proposed features in the PRD
- Any flags or warnings raised across all agents
