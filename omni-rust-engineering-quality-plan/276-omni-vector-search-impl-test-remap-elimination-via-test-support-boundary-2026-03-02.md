# 276. xiuxian-vector search_impl Test Remap Elimination via test_support Boundary (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-vector`
- Objective:
  - remove remaining `include!("../src/search/search_impl/...")` usage in
    package-top integration tests,
  - replace test access with a stable crate-owned `test_support` boundary,
  - keep behavior stable with targeted test/lint proof.

## Changes

### 1) Added stable `test_support` APIs

Added:

- `packages/rust/crates/xiuxian-vector/src/test_support.rs`

Updated:

- `packages/rust/crates/xiuxian-vector/src/lib.rs`
- `packages/rust/crates/xiuxian-vector/src/search/search_impl/mod.rs`

Actions:

- introduced public test-support functions:
  - `keyword_boost()`,
  - `search_results_to_ipc(...)`,
  - `tool_search_results_to_ipc(...)`,
- added crate-internal adapters in `search_impl/mod.rs` to forward to existing
  production implementations without behavior changes.

### 2) Removed source includes from `search_impl` integration test lane

Updated:

- `packages/rust/crates/xiuxian-vector/tests/search_impl_unit.rs`
- `packages/rust/crates/xiuxian-vector/tests/search_impl_module/tests.rs`

Actions:

- removed:
  - `include!("../src/search/search_impl/confidence.rs")`,
  - `include!("../src/search/search_impl/ipc.rs")`,
- switched harness to call `omni_vector::test_support` APIs directly,
- preserved all existing assertions and test semantics.

## Validation Evidence

### 1) Targeted nextest

```bash
cargo nextest run -p xiuxian-vector --test search_impl_unit
```

Result:

- `6 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory touched-crate clippy gate

```bash
cargo clippy -p xiuxian-vector -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0),
- no new warnings introduced by this migration.

### 3) Cross-crate structural snapshot

```bash
rg -n "include!\\(\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./\\.\\./src/" \
  packages/rust/crates/*/tests --glob "*.rs" \
  | awk -F/ '{print $4}' | sort | uniq -c | sort -nr
```

Result:

- `38 xiuxian-daochang`,
- `xiuxian-vector` reduced to zero matches.

## Outcome

- `xiuxian-vector` test lane now follows package-top integration boundary rules
  without source remapping,
- regression and lint gates are green,
- remaining remap debt is isolated to `xiuxian-daochang`.
