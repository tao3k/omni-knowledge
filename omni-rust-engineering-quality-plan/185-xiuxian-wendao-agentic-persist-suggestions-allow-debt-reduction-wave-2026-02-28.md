# 185. Xiuxian-Wendao Agentic Persist-Suggestions Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_wendao_cli/agentic/execution/agentic_run_can_persist_suggestions.rs`

## Why This Wave

This CLI scenario leaf still used broad file-level suppression and exceeded the
pedantic line-count limit after suppression removal.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `agentic_run_can_persist_suggestions.rs`

2. Root-cause cleanup:
   - extracted repeated `agentic run --persist` invocation into
     `run_agentic_persist(...)` helper
   - inlined assertion formatting by binding stderr payloads and using
     `{stderr}`/`{recent_stderr}`
   - reduced file size from `124` lines to `98` lines to satisfy
     `clippy::too_many_lines`

No suppressions were reintroduced.

## Validation Evidence

1. Format:

```bash
cargo fmt -p xiuxian-wendao
```

- Result: pass

2. Strict clippy:

```bash
CARGO_TARGET_DIR=target/clippy-wendao cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `12`
  - After this wave: `11`
  - Net reduction: `1` file

## Engineering Outcome

- Agentic persist-suggestions CLI scenario test is now suppression-free and
  structurally cleaner.
- Remaining debt is concentrated in larger CLI/agentic modules.
