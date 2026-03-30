---
name: document-operations
description: Generate operations documentation covering local setup, deployment, monitoring, logging, and common issues
---

Analyse the codebase at $ARGUMENTS and write the output to `operations/index.md`.

Use the template at `.claude/skills/templates/06-operations.md` as the exact output structure. Fill in every section based on what you can observe in the README, Dockerfiles, CI/CD config, build files, and environment config.

**Section guidance:**
- **Running Locally** — step-by-step instructions derivable from README, docker-compose, and Makefile/build files
- **Build & Deploy** — build command, output artifact, and deployment mechanism (Docker, JAR, npm, etc.)
- **Environments** — infer from CI/CD config, environment variable names, or deployment scripts (e.g. dev, staging, prod)
- **Configuration** — list actual env var names found in the codebase; flag any with no defaults as required
- **Monitoring** — identify any monitoring libraries, health check endpoints, or metrics endpoints in the code
- **Logging** — logging library used, log level config, and output format if determinable from config
- **Jobs / Schedulers** — any scheduled tasks, cron jobs, or background workers found in the code
- **Common Issues** — known failure modes observable from error handling code, TODO comments, or README warnings
- **Backup & Recovery** — any backup scripts, migration rollback steps, or recovery procedures found in config or docs

**Formatting rules:**
- Use `> ⚠️ Unclear:` for anything requiring knowledge outside the codebase (e.g. infrastructure details not in config)
- Use numbered steps for setup sequences where order matters
- Use the common issues table — do not replace with prose
- Be factual — only document what you can verify from the code and config files
