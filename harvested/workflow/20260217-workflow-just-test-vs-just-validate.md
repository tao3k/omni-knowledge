# Just Test vs Just Validate

> **Category**: WORKFLOW | **Date**: 2026-02-17

## Rule

- **`just test`** – Use for normal development. Runs tests only (fast iteration).
- **`just validate`** – Use only for release pre-release. Full fmt + lint + test (unified validation).

Do not run `just validate` for every small change; it takes 60s+ and is reserved for release validation.
