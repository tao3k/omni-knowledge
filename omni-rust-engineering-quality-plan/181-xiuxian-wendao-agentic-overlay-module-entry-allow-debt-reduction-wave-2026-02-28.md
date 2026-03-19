# 181. Xiuxian-Wendao Agentic Overlay Module Entry Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_wendao_cli/agentic/overlay/mod.rs`

## Why This Wave

This module-entry file is a compact orchestration layer and an efficient
candidate for incremental suppression-debt reduction.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `tests/test_wendao_cli/agentic/overlay/mod.rs`

2. No additional source changes were required after removal.

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
  - Before this wave: `17`
  - After this wave: `16`
  - Net reduction: `1` file

## Engineering Outcome

- Agentic overlay module-entry tests are now suppression-free.
- Remaining debt is now concentrated in CLI scenario leaves and seed-priors
  scenario files.
