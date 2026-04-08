# Legacy Migration — Project Context

## Goal

This repo is a documentation and analysis toolkit for legacy codebases. Point it at any codebase and it produces a structured, evidence-based picture of what the system is, how it works, how risky it is to change, and what it would take to migrate it.

The output is designed to give a migration team everything they need to make confident decisions: understand the codebase quickly, scope the work into epics, and decide whether to rewrite or incrementally strangle and refactor.

## What it produces

For each codebase analysed, the toolkit generates:

- **10 documentation specs** covering system overview, architecture, code structure, data model, integrations, operations, change risk, testing, security, and business rules
- **A rewrite vs. refactor recommendation** with a weighted score and evidence for each dimension
- **GitHub epics** as issues on the target repo, covering all functional areas to be tackled
- **An audit report** validating the quality and consistency of the generated docs

Output is versioned by directory (e.g. `documentationV3/`). The most recent run covers `spring-projects/spring-framework`.

---

## Agents

Agents are autonomous subprocesses that handle multi-step analysis tasks. Invoke them via `/agent <name>`.

| Agent | What it does |
|---|---|
| `code-documenter` | Analyses a codebase and writes all 10 documentation specs. Reads templates from `.claude/skills/templates/` and outputs to the current versioned documentation directory. |
| `migration-advisor` | Scores a codebase across 10 dimensions and produces a weighted rewrite vs. strangle & refactor recommendation. Outputs to `output/rewrite-vs-refactor.md`. |
| `epic-creator` | Analyses a codebase or its output docs and creates GitHub issues as epics. Takes a codebase path and `owner/repo` as input. |
| `doc-auditor` | Audits the generated documentation against the 10 templates. Checks completeness, stubs, quality, and cross-document consistency. Outputs an `audit-report.md`. |

---

## Skills

Skills are lightweight slash commands for targeted, single-purpose tasks. Invoke them via `/<skill-name> <arguments>`.

### Documentation skills — generate one doc type at a time

| Skill | What it does |
|---|---|
| `/document-system-overview <path>` | System purpose, capabilities, users, criticality, ownership |
| `/document-architecture <path>` | Components, interaction patterns, data flow, deployment, constraints |
| `/document-code-structure <path>` | Repo layout, key modules, entry points, coding patterns, areas of concern |
| `/document-data-model <path>` | Entities, relationships, storage, lifecycle, migration risks |
| `/document-integrations <path>` | APIs, messaging, external dependencies, failure modes |
| `/document-operations <path>` | Local setup, build & deploy, CI, environments, monitoring, common issues |
| `/document-change-risk <path>` | High-risk areas, coupling, technical debt, obsolete tech, recommendations |
| `/document-testing <path>` | Test strategy, types, coverage, gaps, release validation |
| `/document-security <path>` | Auth, sensitive data, CVEs, dependencies risk, compliance |
| `/document-business-rules <path>` | Core rules, workflows, validations, edge cases, inconsistencies |
| `/document-module <path>` | Focused doc for a specific module, class, or file |
| `/document-api <path>` | All public API endpoints or interfaces |
| `/document-onboarding <path>` | Getting-started guide for new developers |

### Analysis skills

| Skill | What it does |
|---|---|
| `/summarise-codebase <path>` | Quick high-level overview before full documentation begins |
| `/assess-migration <path>` | Runs the rewrite vs. strangle & refactor scoring framework |
| `/create-epics <path> owner/repo` | Creates GitHub issues as epics from a codebase or its output docs |

---

## How agents and skills relate

The agents use the same templates and follow the same output conventions as the individual skills — they just do more in one shot. Use a **skill** when you want to regenerate or update a single document. Use an **agent** when you want the full pipeline run end-to-end.

```
/summarise-codebase          ← quick first look
/document-* or code-documenter  ← produce the 10 docs
/assess-migration or migration-advisor  ← rewrite vs. refactor decision
/create-epics or epic-creator   ← turn findings into GitHub epics
doc-auditor                  ← validate the docs are complete and consistent
```
