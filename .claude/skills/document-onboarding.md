---
name: document-onboarding
description: Generate a getting started guide for new developers working on the legacy codebase
---

Analyse the codebase at $ARGUMENTS and write the output to `onboarding/getting-started.md`.

Follow this structure exactly:

```
# Getting Started — <System Name>

> Estimated setup time: <estimate based on complexity observed>

---

## Prerequisites

Table with columns: Tool | Version | Notes
Include any version-specific constraints or known incompatibilities in the Notes column.

---

## Setup

Numbered steps. Each step should have:
- A bold title
- Prose or command block explaining exactly what to do
- Any warnings inline as blockquotes

---

## Running the Application

How to start the application. Include the command, what success looks like (log line, HTTP response, etc.), and where to find logs.

---

## Running Tests

How to run the full test suite and how to run a single test. Include any prerequisites (e.g. live DB required).

---

## Key Concepts

Numbered list of 3–6 things a new developer must understand to be productive. Each item should:
- Have a bold title
- Explain the concept in 2–4 sentences
- Explain why it matters or what goes wrong if you don't understand it

---

## Common Gotchas

Table with columns: Gotcha | Detail
```

**Formatting rules:**
- Use `> ⚠️ Unclear:` callouts for anything that cannot be confirmed from the code or config — e.g. environment details that must come from ops
- Use tables for prerequisites and gotchas
- Use numbered steps (not bullets) for setup sequences where order matters
- Be factual — only document setup steps you can verify from the code, build files, and config
