---
name: document-api
description: Extract and document all API endpoints or public interfaces from the legacy codebase
---

Analyse the codebase at $ARGUMENTS and identify all exposed API endpoints or public interfaces.

For each endpoint or interface, document:

1. **Method and path** — e.g. `GET /users/:id` or function signature
2. **Description** — what does this endpoint/interface do?
3. **Request parameters / inputs** — path params, query params, request body with types
4. **Response format / outputs** — response body with types and example
5. **Authentication** — is auth required? What type?
6. **Error cases** — known error responses and their meaning

Write the output to `api/index.md`.

Be factual — only document what you can observe in the code. Mark anything unclear with a `> ⚠️ Unclear:` callout.
