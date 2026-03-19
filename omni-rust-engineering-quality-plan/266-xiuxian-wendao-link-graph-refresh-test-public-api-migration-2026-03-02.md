# 266. xiuxian-wendao link_graph_refresh Test Public-API Migration (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-wendao`
- Objective:
  - remove `link_graph_refresh_unit` dependency on private
    `link_graph_py/engine/refresh/strategy.rs`,
  - validate refresh behavior through public `LinkGraphIndex` contract,
  - keep touched crate test/lint gates warning-clean.

## Changes

### Rewrote refresh unit tests to public behavior assertions

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/link_graph_refresh_unit.rs`

Actions:

- removed:
  - `#[path = "../src/link_graph_py/engine/refresh/strategy.rs"]`
- replaced private planner function tests with public incremental refresh mode
  tests using `LinkGraphIndex::refresh_incremental_with_threshold(...)`.
- asserted three public behavior modes:
  - `Noop` when changed paths are empty,
  - `Full` when threshold is exceeded,
  - `Delta` when threshold is not exceeded.

## Validation Evidence

### 1) Targeted nextest

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-wendao --test link_graph_refresh_unit
```

Result:

- `3 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory clippy gate

```bash
RUSTC_WRAPPER= cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0), no warnings/errors.

### 3) Structural proof command

```bash
rg -n "#\\[path\\s*=\\s*\\\"\\.\\./src/link_graph_py/engine/refresh/strategy\\.rs\\\"" \
  packages/rust/crates/xiuxian-wendao/tests/link_graph_refresh_unit.rs
```

Result:

- no matches.

## Outcome

- refresh test lane now validates via public API semantics only,
- source remapping dependency removed,
- touched crate remains warning-clean and test-green under required gates.
