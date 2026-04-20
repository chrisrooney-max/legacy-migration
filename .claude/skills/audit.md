---
name: audit
description: Run the doc-auditor agent over a documentation directory and write an audit-report.md.
---

Run the doc-auditor agent over the documentation directory at $ARGUMENTS.

If no argument is provided, audit the most recently created `documentationV*/` directory in the project root.

## Step 1 — Run the audit

Spawn the `doc-auditor` agent against the documentation directory provided. The agent will:
- Check all documents for completeness against their templates
- Detect stub sections
- Flag accuracy issues and cross-document inconsistencies
- Write `audit-report.md` into the same directory

## Step 2 — Confirm completion

When the agent has finished, report back to the user with:
- The path to the audit report
- The counts from the Key Metrics table (sections present, missing, stubs, flags, inconsistencies)
- Any critical findings that need immediate attention
