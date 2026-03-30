---
name: document-architecture
description: Generate architecture documentation covering components, data flow, deployment, and constraints
---

Analyse the codebase at $ARGUMENTS and write the output to `architecture/index.md`.

Use the template at `.claude/skills/templates/02-architecture.md` as the exact output structure. Fill in every section based on what you can observe in the code, configuration files, Dockerfiles, and infrastructure definitions.

**Section guidance:**
- **Overview** — one short paragraph describing the architectural style (monolith, microservice, event-driven, etc.)
- **Context Diagram** — produce an ASCII diagram if no image can be linked; show the system and its external actors
- **Components** — fill the table with every major service, module, or layer; include file/package location
- **Interaction Patterns** — identify sync vs async, REST vs messaging, blocking vs non-blocking
- **Data Flow** — numbered steps from ingestion to output
- **External Dependencies** — third-party APIs, databases, queues, and cloud services
- **Deployment Architecture** — infer from Dockerfiles, docker-compose, CI/CD config, or cloud provider config
- **Key Constraints** — language version pins, platform limits, network restrictions evident from config
- **Known Issues** — architectural problems only; module-level issues belong in `03-code-structure.md`

**Formatting rules:**
- Use `> ⚠️ Unclear:` for anything that cannot be confirmed from the code alone
- Use the components table — do not replace it with a bullet list
- ASCII diagrams are preferred over placeholder comments
- Be factual — only document what you can observe
