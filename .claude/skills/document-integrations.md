---
name: document-integrations
description: Generate integrations documentation covering APIs, messaging, dependencies, failure modes, and known issues
---

Analyse the codebase at $ARGUMENTS and write the output to `integrations/index.md`.

Use the template at `.claude/skills/templates/05-integrations.md` as the exact output structure. Fill in every section based on what you can observe in HTTP clients, message queue config, SDK imports, and environment variable names.

**Section guidance:**
- **Overview** — summarise the number and nature of integrations (e.g. "3 outbound REST APIs, 1 message queue")
- **Integration Catalogue** — one row per external system; direction is Inbound / Outbound / Both; method is REST, gRPC, MQ, SFTP, etc.
- **APIs** — for each API integration: endpoint base URL (or env var name if URL is not hardcoded), auth method, and whether a contract/spec exists
- **Events / Messaging** — topic or queue name, which service produces and which consumes
- **Error Handling** — retry strategy (fixed, exponential, none), circuit breaker presence, and what happens on failure
- **Dependencies** — upstream systems this service depends on; downstream systems that depend on this service
- **Failure Modes** — what the system does when each integration is unavailable (fail fast, degrade gracefully, silent failure)
- **Known Issues** — missing retries, broken contracts, undocumented endpoints, version mismatches

**Formatting rules:**
- Use `> ⚠️ Unclear:` for anything that cannot be confirmed — e.g. undocumented endpoints or unknown failure behaviour
- Use the integration catalogue table — do not replace with prose
- Be factual — only document integrations you can confirm from the code or config
