---
name: document-onboarding
description: Generate a getting started guide for new developers working on the legacy codebase
---

Analyse the codebase at $ARGUMENTS and produce a getting started guide for a new developer.

Cover the following:

1. **Prerequisites** — required tools, languages, runtimes, and versions
2. **Setup** — step-by-step instructions to get the project running locally
3. **Running the application** — how to start the app in development mode
4. **Running tests** — how to run the test suite and any individual tests
5. **Key concepts** — the 3-5 things a developer must understand to be productive quickly
6. **Common gotchas** — known traps, quirks, or non-obvious behaviours to be aware of

Write the output to `onboarding/getting-started.md`.

Be factual — only document what you can observe in the code or config files. Mark anything unclear with a `> ⚠️ Unclear:` callout.
