Run the full analysis pipeline against the codebase at $ARGUMENTS.

## Step 1 — Document the codebase

Spawn the `code-documenter` agent against the codebase path provided. Wait for it to complete all 10 documents before proceeding.

The agent will write output to the versioned documentation directory specified in `context/migration-inputs.md`. If no directory is specified, use the next available version (e.g. if `documentationV3/` exists, use `documentationV4/`).

Do not proceed to Step 2 until all 10 documents have been written successfully.

## Step 2 — Run the migration advisor

Once documentation is complete, automatically spawn the `migration-advisor` agent.

The advisor will read `context/migration-inputs.md` for financial inputs and the codebase path. It will write its report (`rewrite-vs-refactor.md`) to the same versioned output directory as the documentation.

## Step 3 — Confirm completion

When both agents have finished, report back to the user with:
- The output directory where all files were written
- A one-line summary of the migration advisor verdict (score and recommendation)
- Any flags or warnings either agent raised
