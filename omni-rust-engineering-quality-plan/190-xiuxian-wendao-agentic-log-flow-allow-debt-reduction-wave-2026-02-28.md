# 190. Xiuxian-Wendao Agentic Log-Flow Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_wendao_cli/agentic/log_flow.rs`
  - shared agentic helpers in `tests/test_wendao_cli/agentic/mod.rs`

## Why This Wave

The agentic log/recent/decide/decisions flow test was still suppression-based
and significantly above pedantic file-length limits.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `tests/test_wendao_cli/agentic/log_flow.rs`

2. Added shared helper set in `agentic/mod.rs`:
   - `run_agentic_log_default(...)`
   - `run_agentic_recent_provisional(...)`
   - `run_agentic_decide_promoted(...)`
   - `run_agentic_decisions(...)`

3. Refactored flow test to use helpers:
   - retained behavior and assertions
   - removed repeated command wiring and argument boilerplate
   - reduced `log_flow.rs` from `160` lines to `63` lines
   - kept `agentic/mod.rs` at `100` lines to satisfy `too_many_lines`

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
  - Before this wave: `7`
  - After this wave: `6`
  - Net reduction: `1` file

## Engineering Outcome

- Agentic log-flow scenario is now suppression-free with better helper reuse.
- Remaining suppression debt is concentrated in the largest CLI and overlay
  integration files.
