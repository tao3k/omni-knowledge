# 188. Xiuxian-Wendao Agentic Discovery-Quality Signals Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_wendao_cli/agentic/execution/agentic_run_emits_discovery_quality_signals.rs`
  - shared execution helpers in `tests/test_wendao_cli/agentic/execution/mod.rs`

## Why This Wave

This execution scenario file still depended on broad suppression and exceeded
pedantic line-count constraints after suppression removal.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `agentic_run_emits_discovery_quality_signals.rs`

2. Extracted shared execution helpers in module entry:
   - `write_agentic_execution_config(...)`
   - `run_agentic_run_persist(...)`
   - `run_agentic_recent_json(...)`

3. Refactored scenario test to use shared helpers:
   - removed duplicated config/run/recent command boilerplate
   - reduced file size from `135` lines to `92` lines

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
  - Before this wave: `9`
  - After this wave: `8`
  - Net reduction: `1` file

## Engineering Outcome

- Agentic discovery-quality signal scenario is now suppression-free and less
  repetitive.
- Remaining debt is concentrated in the largest CLI and overlay scenario files.
