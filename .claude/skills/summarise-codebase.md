---
name: summarise-codebase
description: Produce a quick high-level overview of a codebase before full documentation begins
---

Analyse the codebase at $ARGUMENTS and write the output to `architecture/overview.md`.

Follow this structure exactly:

```
# Architecture Overview — <System Name>

> **System:** <name and version> | **Language:** <language and version> | **Last active development:** <year>

One paragraph describing what the system does and its role.

---

## Tech Stack

Table with columns: Layer | Technology | Version | Notes

---

## Top-Level Structure

Directory tree (code block) showing major folders and a comment on each explaining what lives there.

---

## Component Diagram

ASCII diagram showing major components and how they connect, with a brief label on each arrow describing the interaction type.

---

## Data Flow

Numbered steps describing how data moves through the system from entry point to output.

---

## Key Design Decisions

Table with columns: Decision | Rationale

---

## Technical Debt / Risk Areas

Bullet list of risk areas. Each item should be specific and actionable, not vague.
```

**Formatting rules:**
- Use `> ⚠️ Unclear:` callouts for anything ambiguous or that cannot be confirmed from the code alone
- Use tables for anything comparative
- Keep prose minimal — prefer bullets and tables over paragraphs
- Be factual — only document what you can observe in the code
