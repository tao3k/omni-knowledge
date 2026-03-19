# 192. Xiuxian-Wendao CLI Final Allow-Debt Zero Convergence Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_wendao_cli/ambiguity.rs`
  - `tests/test_wendao_cli/agentic/planning.rs`
  - `tests/test_wendao_cli/agentic/overlay/provisional_links_are_isolated_before_promotion.rs`
  - `tests/test_wendao_cli/agentic/overlay/promoted_links_materialize_in_neighbors_and_related.rs`

## Why This Wave

After wave `191`, the remaining suppression debt in `xiuxian-wendao/tests` was
concentrated in four high-complexity CLI scenario files. The target for this
wave was strict root-cause cleanup only: remove file-level `#![allow(...)]`
attributes and satisfy `clippy::too_many_lines` through structural refactoring.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from the four remaining files.

2. Refactored oversized test functions into focused helpers instead of lint
   suppression:
   - In `agentic/overlay/provisional_links_are_isolated_before_promotion.rs`:
     extracted config writer, command runner wrappers, and overlay assertions.
   - In `agentic/overlay/promoted_links_materialize_in_neighbors_and_related.rs`:
     extracted setup, promotion flow, and per-command assertion helpers.
   - In `agentic/planning.rs`:
     extracted doc fixtures, root-scoped command execution helper, runtime
     config/counter assertions, and worker telemetry checks.

3. Preserved behavioral coverage:
   - ambiguity candidate reporting,
   - agentic planning/runtime budget assertions,
   - provisional-overlay isolation behavior,
   - promoted-overlay materialization behavior.

No new lint suppressions were introduced.

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

- Result: pass (no warnings)

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `5`
  - After this wave: `0`
  - Net reduction: `5` files

## Engineering Outcome

- `xiuxian-wendao/tests` now has zero file-level `#![allow(...)]` debt.
- Large CLI scenario tests remain behavior-equivalent but are now structured
  into reusable helpers, improving readability and future maintenance.
