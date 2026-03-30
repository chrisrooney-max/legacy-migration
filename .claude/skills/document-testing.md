---
name: document-testing
description: Generate testing documentation covering strategy, test types, coverage, gaps, and release validation
---

Analyse the codebase at $ARGUMENTS and write the output to `testing/index.md`.

Use the template at `.claude/skills/templates/08-testing.md` as the exact output structure. Fill in every section based on what you can observe in the test directories, test frameworks, CI config, and coverage reports.

**Section guidance:**
- **Testing Strategy** — describe the overall approach: is there a clear testing pyramid? Are tests unit, integration, or E2E heavy?
- **Test Types** — identify the frameworks used for each type (e.g. JUnit, Spock, Cypress, Cucumber) and where the tests live
- **Coverage** — fill the table per major module or package; coverage level is High / Medium / Low / None based on presence and density of tests
- **Critical Test Scenarios** — identify the most important scenarios covered by existing tests (from test names and feature files)
- **Manual Testing** — areas with no automated tests that appear to require manual validation
- **Gaps** — specific modules, flows, or edge cases with no test coverage
- **Release Validation** — steps to verify the system works before a release, based on CI config and README
- **Known Issues** — flaky tests, tests requiring live dependencies, tests marked skip/ignore/manual

**Formatting rules:**
- Use `> ⚠️ Unclear:` for coverage estimates that cannot be confirmed without running a coverage tool
- Use the coverage table — do not replace with bullet lists
- Be factual — only report on test files and frameworks you can observe in the codebase
