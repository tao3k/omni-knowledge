# 194. Omni-Memory Test Missing-Docs Suppression Removal Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/omni-memory`
- Focus:
  - `tests/test_feedback_tracking.rs`
  - `tests/test_state_backend.rs`

## Why This Wave

Two integration-test files still relied on file-level
`#![allow(missing_docs)]`. This wave removes those suppressions and replaces
them with explicit module documentation while preserving existing test behavior.

## Changes Implemented

1. Removed file-level suppressions:
   - `tests/test_feedback_tracking.rs`
   - `tests/test_state_backend.rs`

2. Added module documentation:
   - `//! Feedback tracking integration tests for \`omni-memory\`.`
   - `//! State backend key-derivation tests for \`omni-memory\`.`

No new lint suppressions were introduced.

## Validation Evidence

1. Format:

```bash
cargo fmt -p omni-memory
```

- Result: pass

2. Strict clippy:

```bash
CARGO_TARGET_DIR=target/clippy-omni-memory cargo clippy -p omni-memory --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-omni-memory cargo nextest run -p omni-memory
```

- Result: pass
- Summary: `67 passed`, `0 failed`, `0 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/omni-memory/tests -g '*.rs'`
  - Before this wave: `2` files
  - After this wave: `0` files
  - Net reduction: `2` files

## Engineering Outcome

- `omni-memory/tests` now has zero file-level `missing_docs` suppression.
- Test entry files remain small and explicit, aligned with documentation-first
  lint compliance.
