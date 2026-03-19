# 265. xiuxian-vector match_util Public Export and Test Remap Elimination (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-vector`
- Objective:
  - remove `keyword_fusion_match_util_unit` dependency on source-path remapping,
  - validate `match_util` via `keyword::fusion` public API,
  - keep touched crate warning-clean after API exposure.

## Changes

### 1) Exposed match_util helpers through fusion public boundary

Updated:

- `packages/rust/crates/xiuxian-vector/src/keyword/fusion/mod.rs`
- `packages/rust/crates/xiuxian-vector/src/keyword/fusion/match_util.rs`

Actions:

- added public re-exports in `keyword::fusion` for:
  - `lowercase_string_array`
  - `build_name_lower_arrow`
  - `count_name_token_matches_and_exact`
  - `build_name_token_automaton_with_phrase`
  - `NameMatchResult`
- added docs for `NameMatchResult` public fields.
- added `#[must_use]` to returned-value utility functions to remove
  `must_use_candidate` warnings.

### 2) Migrated unit harness to crate public API

Updated:

- `packages/rust/crates/xiuxian-vector/tests/keyword_fusion_match_util_unit.rs`

Actions:

- removed:
  - `#[path = "../src/keyword/fusion/match_util.rs"]`
- switched harness imports to:
  - `omni_vector::keyword::fusion::{...}`

## Validation Evidence

### 1) Targeted nextest

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-vector --test keyword_fusion_match_util_unit
```

Result:

- `4 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory clippy gate

```bash
RUSTC_WRAPPER= cargo clippy -p xiuxian-vector -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0), no warnings/errors.

### 3) Structural proof command

```bash
rg -n "#\\[path\\s*=\\s*\\\"\\.\\./src/keyword/fusion/match_util\\.rs\\\"" \
  packages/rust/crates/xiuxian-vector/tests/keyword_fusion_match_util_unit.rs
```

Result:

- no matches.

## Outcome

- `keyword_fusion_match_util_unit` now uses stable public module imports,
- path remapping debt removed in this lane,
- touched crate remains test-green and warning-clean under required gate.
