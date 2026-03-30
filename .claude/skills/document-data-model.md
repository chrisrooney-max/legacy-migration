---
name: document-data-model
description: Generate data model documentation covering entities, relationships, storage, lifecycle, and migration risks
---

Analyse the codebase at $ARGUMENTS and write the output to `data-model/index.md`.

Use the template at `.claude/skills/templates/04-data-model.md` as the exact output structure. Fill in every section based on what you can observe in schema files, ORM models, migration scripts, and database config.

**Section guidance:**
- **Overview** — describe the overall data structure approach (relational, document, key-value, mixed)
- **Core Entities** — list every significant domain entity; owner is the service or module responsible for it
- **Relationships** — describe foreign keys, embedded documents, or join patterns between entities
- **Data Storage** — list every database and its type (SQL, NoSQL, cache, etc.) found in config or connection strings
- **Key Tables / Collections** — fill the table with actual table/collection names, their purpose, key fields, and any migration risk
- **Data Lifecycle** — how records are created, updated, and deleted (soft delete, hard delete, archiving)
- **Data Quality Issues** — nullable fields that should not be, missing constraints, inconsistent naming, duplicate data
- **Reporting Dependencies** — any reports, exports, or analytics queries that depend on the data model
- **Migration Risks** — tables or fields that are risky to change (used by multiple systems, no foreign key enforcement, etc.)

**Formatting rules:**
- Use `> ⚠️ Unclear:` for anything that cannot be confirmed from the code alone
- Use the provided tables — do not replace with bullet lists
- Be factual — only document entities and fields you can confirm exist in schema or model files
