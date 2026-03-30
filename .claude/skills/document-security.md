---
name: document-security
description: Generate security documentation covering authentication, authorisation, sensitive data, vulnerabilities, and compliance
---

Analyse the codebase at $ARGUMENTS and write the output to `security/index.md`.

Use the template at `.claude/skills/templates/09-security.md` as the exact output structure. Fill in every section based on what you can observe in auth code, middleware, config, dependency versions, and data handling.

**Section guidance:**
- **Overview** — one paragraph summarising the security posture (e.g. public API with no auth, internal service with JWT, etc.)
- **Authentication** — method (JWT, session, API key, none) and the flow as observable from middleware and route config
- **Authorization** — roles and permissions found in code; how access control is enforced
- **Sensitive Data** — types of sensitive data handled (PII, credentials, payment info) inferred from field names and models
- **Data Protection** — encryption at rest/in transit (infer from config and libraries); any masking or redaction in logs
- **Audit Logging** — what events are logged for security/audit purposes
- **Vulnerabilities** — specific issues observable in the code: hardcoded secrets, missing input validation, SQL injection risk, outdated dependencies with known CVEs
- **Dependencies Risk** — libraries that are significantly out of date and may carry known vulnerabilities
- **Compliance** — any compliance indicators in the code (GDPR data deletion, audit trails, data residency config)
- **Recommendations** — specific, actionable security improvements

**Formatting rules:**
- Use `> ⚠️ Unclear:` for security posture assessments that require runtime or infrastructure knowledge
- Be factual — only flag vulnerabilities you can confirm from the code; do not speculate
- Be specific — name the exact file or pattern where an issue is found
