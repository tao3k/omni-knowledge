# 256. xiuxian-vector and xiuxian-tui Test Include-Indirection Reduction Wave (2026-03-02)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-vector`
  - `packages/rust/crates/xiuxian-tui`
- Objective:
  - remove `include!("unit/...")` indirection in package-top harnesses,
  - keep current test seams stable,
  - preserve behavior with targeted nextest + mandatory clippy.

## Changes

### 1) xiuxian-tui

Updated:

- `packages/rust/crates/xiuxian-tui/tests/main_demo_unit.rs`

Change:

- switched nested test mount from:
  - `mod tests { include!("unit/main_demo_tests.rs"); }`
- to:
  - `mod tests;`

Moved:

- `packages/rust/crates/xiuxian-tui/tests/unit/main_demo_tests.rs`
  -> `packages/rust/crates/xiuxian-tui/tests/main_demo_module/tests.rs`

Result:

- `xiuxian-tui/tests` include audit is now zero.

### 2) xiuxian-vector

Updated:

- `packages/rust/crates/xiuxian-vector/tests/keyword_fusion_match_util_unit.rs`
- `packages/rust/crates/xiuxian-vector/tests/search_impl_unit.rs`

Change:

- switched nested test mounts from:
  - `mod tests { include!("unit/..."); }`
- to:
  - `mod tests;`

Moved:

- `packages/rust/crates/xiuxian-vector/tests/unit/keyword/fusion/match_util_tests.rs`
  -> `packages/rust/crates/xiuxian-vector/tests/match_util_module/tests.rs`
- `packages/rust/crates/xiuxian-vector/tests/unit/search/search_impl/tests.rs`
  -> `packages/rust/crates/xiuxian-vector/tests/search_impl_module/tests.rs`

Note:

- `xiuxian-vector` still intentionally includes:
  - `../src/search/search_impl/confidence.rs`
  - `../src/search/search_impl/ipc.rs`
  for crate-private seam testing in this lane.

## Validation Evidence

### 1) xiuxian-tui target lane

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-tui --test main_demo_unit
```

Result:

- `3 passed`, `0 failed`, `0 skipped`

### 2) xiuxian-vector target lanes

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-vector \
  --test keyword_fusion_match_util_unit \
  --test search_impl_unit
```

Result:

- `10 passed`, `0 failed`, `0 skipped`

### 3) Mandatory touched-crate clippy gates

```bash
RUSTC_WRAPPER= cargo clippy -p xiuxian-tui -- -W clippy::too_many_lines
RUSTC_WRAPPER= cargo clippy -p xiuxian-vector -- -W clippy::too_many_lines
```

Result:

- both commands succeeded (exit 0)

### 4) Post-wave include audits

```bash
rg -n 'include!\(' packages/rust/crates/xiuxian-tui/tests --glob '*.rs'
rg -n 'include!\(' packages/rust/crates/xiuxian-vector/tests --glob '*.rs'
```

Result:

- `xiuxian-tui/tests`: zero matches
- `xiuxian-vector/tests`: two remaining matches (`../src/search/search_impl/*`)

## Outcome

- removed `unit/...` include indirection from 3 harnesses across 2 crates.
- `xiuxian-tui` test tree reached include-zero.
- `xiuxian-vector` test includes are reduced to only explicit source seam mounts.
