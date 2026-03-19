# 262. xiuxian-wendao Fusion Public-API Test Decoupling and Warning Cleanup (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-wendao`
- Objective:
  - remove `tests` dependency on source-path remapping for fusion boost logic,
  - expose stable public API surface for fusion computation tests,
  - keep the touched crate warning-clean under mandatory clippy gate,
  - clean obsolete test residue that no longer participates in harness execution.

## Changes

### 1) Removed source-path remap from fusion test harness

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/fusion_unit.rs`

Actions:

- removed `#[path = "../src/fusion.rs"] mod fusion_impl;`
- switched imports to crate public API:
  - `use xiuxian_wendao::{RecallResult, apply_link_graph_proximity_boost};`

### 2) Public API export for fusion types/functions

Updated:

- `packages/rust/crates/xiuxian-wendao/src/lib.rs`

Actions:

- added explicit crate re-export:
  - `pub use fusion::{RecallResult, apply_link_graph_proximity_boost};`

### 3) Fusion module warning cleanup after API exposure

Updated:

- `packages/rust/crates/xiuxian-wendao/src/fusion.rs`

Actions:

- resolved `clippy::implicit_hasher` by making hasher parameters explicit in
  `HashMap`/`HashSet` signature types.
- resolved `missing_docs` by documenting public fields/constructor.
- resolved `clippy::must_use_candidate` by adding `#[must_use]` to
  `RecallResult::new`.

### 4) Removed obsolete dead test file

Deleted:

- `packages/rust/crates/xiuxian-wendao/tests/unit/fusion_tests.rs`

Rationale:

- no harness referenced this file after migration (`rg` no-match),
- coverage is maintained by `tests/fusion_unit.rs`.

## Validation Evidence

### 1) Targeted nextest

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-wendao --test fusion_unit
```

Result:

- `2 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory clippy gate

```bash
RUSTC_WRAPPER= cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0), no warnings/errors.

### 3) Structural proof command

```bash
rg -n "#\\[path\\s*=\\s*\\\"\\.\\./src/fusion\\.rs\\\"" \
  packages/rust/crates/xiuxian-wendao/tests --glob '*.rs'
```

Result:

- no matches.

## Outcome

- `fusion_unit` now validates the same behavior through public API boundary
  instead of source remapping,
- fusion API surfaced with explicit docs and warning-clean signatures,
- touched crate remains test-green and lint-green under required gates.
