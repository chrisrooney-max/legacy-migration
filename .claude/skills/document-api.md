---
name: document-api
description: Extract and document all API endpoints or public interfaces from the legacy codebase
---

Analyse the codebase at $ARGUMENTS and write the output to `api/index.md`.

Follow this structure exactly:

```
# API Documentation — <System Name>

> **Base URL:** <base URL> | **Protocol:** <HTTP version or other> | **Format:** <request/response format>

---

## Authentication

Describe the auth mechanism. Include an example header or token format if one exists.

---

## Endpoint Index

Table with columns: Method | Path | Description

---

## Endpoints

For each endpoint, use this sub-structure:

### <METHOD> <path>

One sentence describing what this endpoint does.

**Path parameters:** (table: Parameter | Type | Description) — omit if none
**Query parameters:** (table: Parameter | Type | Required | Default | Description) — omit if none
**Request body:** code block showing example payload, followed by a table of fields (Field | Type | Required | Notes)
**Response — <status> <label>:** code block showing example response
**Error responses:** table with columns: Status | Code | Meaning
```

**Formatting rules:**
- Use `> ⚠️ Unclear:` callouts for anything ambiguous — e.g. undocumented behaviour, auth gaps, or inconsistent error handling observed in code
- Use tables for parameters and errors
- Show realistic example payloads in code blocks, using the actual format the API uses (JSON, XML, etc.)
- Be factual — only document endpoints you can confirm exist in the code
