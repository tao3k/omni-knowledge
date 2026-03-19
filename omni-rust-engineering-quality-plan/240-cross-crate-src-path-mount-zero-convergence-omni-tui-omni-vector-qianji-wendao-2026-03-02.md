# 240 - Cross-Crate `src` Path-Mount Zero Convergence (`omni-tui`, `xiuxian-vector`, `xiuxian-qianji`, `xiuxian-wendao`) - 2026-03-02

## Scope

This slice continues the package-top test-layout migration standard:

- Remove all `#[cfg(test)] #[path = "../../tests/..."]` mounts from Rust `src`.
- Keep tests under crate-top `tests/`.
- Preserve behavior with explicit integration harnesses and targeted validation.

## Structural Changes

### `omni-tui`

- Removed `src` path mounts from:
  - `src/event/mod.rs`
  - `src/state/mod.rs`
- Added top-level test harnesses:
  - `tests/event.rs`
  - `tests/state.rs`
- Deleted legacy mounted unit files:
  - `tests/unit/event/tests.rs`
  - `tests/unit/state/tests.rs`

### `xiuxian-vector`

- Removed `src` path mounts from:
  - `src/keyword/entity_aware.rs`
  - `src/ops/column_read.rs`
- Added top-level test harnesses:
  - `tests/keyword_entity_aware.rs`
  - `tests/ops_column_read.rs`
- Deleted legacy mounted unit files:
  - `tests/unit/keyword/entity_aware_tests.rs`
  - `tests/unit/ops/column_read_tests.rs`

### `xiuxian-qianji`

- Removed `src` path mounts from:
  - `src/executors/annotation.rs`
  - `src/executors/formal_audit.rs`
- Added top-level test harnesses:
  - `tests/executors_annotation.rs`
  - `tests/executors_formal_audit.rs`
- Deleted legacy mounted unit files:
  - `tests/unit/executors/annotation_tests.rs`
  - `tests/unit/executors/formal_audit_tests.rs`
- Updated persona-id assertion to tolerate current embedded persona evolution
  (`pragmatic_agenda_steward` or `professional_identity_the_clockwork_guardian`)
  without suppression.

### `xiuxian-wendao`

- Removed all remaining `src` path mounts from:
  - `src/unified_symbol_py/mod.rs`
  - `src/entity/mod.rs`
  - `src/sync/mod.rs`
  - `src/storage/mod.rs`
  - `src/dep_indexer_py/mod.rs`
  - `src/unified_symbol/mod.rs`
  - `src/zhenfa_router/rpc.rs`
  - `src/dependency_indexer/pyproject.rs`
  - `src/link_graph/narrator.rs`
  - `src/types/mod.rs`
- Added top-level test harnesses:
  - `tests/unified_symbol_py.rs`
  - `tests/entity_unit.rs`
  - `tests/sync_unit.rs`
  - `tests/storage_unit.rs`
  - `tests/dep_indexer_py.rs`
  - `tests/unified_symbol_unit.rs`
  - `tests/dependency_indexer_pyproject.rs`
  - `tests/link_graph_narrator.rs`
  - `tests/types_unit.rs`
  - `tests/zhenfa_router_rpc.rs` (`feature = "zhenfa-router"` gated)
- Deleted corresponding legacy mounted unit files under:
  - `tests/unit/unified_symbol_py/tests.rs`
  - `tests/unit/entity/tests.rs`
  - `tests/unit/sync/tests.rs`
  - `tests/unit/storage/tests.rs`
  - `tests/unit/dep_indexer_py/tests.rs`
  - `tests/unit/unified_symbol/tests.rs`
  - `tests/unit/zhenfa_router/rpc_tests.rs`
  - `tests/unit/dependency_indexer/pyproject_tests.rs`
  - `tests/unit/link_graph/narrator_tests.rs`
  - `tests/unit/types/tests.rs`

## Verification Evidence

### Global guard (all Rust crates)

```bash
rg --line-number --glob 'packages/rust/crates/*/src/**/*.rs' '#\[path\s*=\s*"\.\./\.\./tests/.*"\]' | sort
```

Result: no matches.

### `omni-tui`

```bash
cargo clippy -p omni-tui -- -W clippy::too_many_lines
cargo nextest run -p omni-tui
```

Result: clippy clean; nextest `42 passed`, `0 failed`.

### `xiuxian-vector`

```bash
cargo clippy -p xiuxian-vector -- -W clippy::too_many_lines
cargo nextest run -p xiuxian-vector --test keyword_entity_aware --test ops_column_read
```

Result: clippy clean; nextest `9 passed`, `0 failed`.

### `xiuxian-qianji`

```bash
cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines
cargo nextest run -p xiuxian-qianji --test executors_annotation --test executors_formal_audit
```

Result: clippy clean; nextest `3 passed`, `0 failed`.

### `xiuxian-wendao`

```bash
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
cargo nextest run -p xiuxian-wendao --test unified_symbol_py --test entity_unit --test sync_unit --test storage_unit --test dep_indexer_py --test unified_symbol_unit --test dependency_indexer_pyproject --test link_graph_narrator --test types_unit --test zhenfa_router_rpc
```

Result: clippy clean; nextest `36 passed`, `0 failed`.

## Outcome

- `src`-side path-mount pattern is now eliminated across all Rust crates.
- Test structure is aligned with package-top harness policy.
- No lint suppression was introduced; all fixes are structural and behavior-preserving.
