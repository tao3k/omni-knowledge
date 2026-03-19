# 178. Xiuxian-Wendao LinkGraph Agentic Expansion Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - `tests/test_link_graph_agentic_expansion.rs`

## Why This Wave

This integration test is an isolated LinkGraph agentic-expansion lane and a
good candidate for suppression-debt reduction without broad cross-module
changes.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from:
   - `tests/test_link_graph_agentic_expansion.rs`

2. Root-cause cleanup:
   - replaced three `map_err(|err| err.to_string())` calls with
     `map_err(|err| err.clone())` to satisfy pedantic clone semantics.

No suppression attributes were reintroduced.

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
  - Before this wave: `23`
  - After this wave: `22`
  - Net reduction: `1` file

## Engineering Outcome

- LinkGraph agentic-expansion integration test is now suppression-free.
- Remaining suppression debt is concentrated in benchmark/topology and
  larger CLI scenario suites.
