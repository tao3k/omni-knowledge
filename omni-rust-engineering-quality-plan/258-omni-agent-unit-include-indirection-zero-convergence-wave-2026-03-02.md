# 258. xiuxian-daochang unit Include-Indirection Zero Convergence Wave (2026-03-02)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Targets:
  - `tests/channels_managed_runtime_unit.rs`
  - `tests/agent_bootstrap.rs`
  - `tests/agent_zhenfa_unit.rs`
  - `tests/agent_session_context_unit.rs`
  - `tests/session_redis_backend_unit.rs`
- Objective:
  - finish migration from `include!("unit/...")` harness wiring to module-local `mod tests;`,
  - relocate legacy test files from `tests/unit/**` into package-top domain directories,
  - remove the remaining `tests/unit` tree for `xiuxian-daochang`.

## Changes

### 1) `channels_managed_runtime_unit` migration completion

Updated:

- `packages/rust/crates/xiuxian-daochang/tests/channels_managed_runtime_unit.rs`

Change:

- from nested include-based tests:
  - `mod tests { include!("unit/channels/managed_runtime/tests/mod.rs"); }`
- to module-local tests:
  - `mod tests;`

Moved:

- `packages/rust/crates/xiuxian-daochang/tests/unit/channels/managed_runtime/tests/mod.rs`
  -> `packages/rust/crates/xiuxian-daochang/tests/channels/managed_runtime/tests/mod.rs`
- `packages/rust/crates/xiuxian-daochang/tests/unit/channels/managed_runtime/tests/test_session_partition_persistence.rs`
  -> `packages/rust/crates/xiuxian-daochang/tests/channels/managed_runtime/tests/test_session_partition_persistence.rs`
- `packages/rust/crates/xiuxian-daochang/tests/unit/channels/managed_runtime/tests/test_turn.rs`
  -> `packages/rust/crates/xiuxian-daochang/tests/channels/managed_runtime/tests/test_turn.rs`

### 2) Remaining `unit/...` harness normalization

Updated harnesses:

- `packages/rust/crates/xiuxian-daochang/tests/agent_bootstrap.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_zhenfa_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_session_context_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/session_redis_backend_unit.rs`

Change pattern:

- from inline includes:
  - `mod tests { include!("unit/..."); }`
  - `mod valkey_hooks_tests { include!("unit/..."); }`
- to module declarations:
  - `mod tests;`
  - `mod valkey_hooks_tests;`

Moved:

- `packages/rust/crates/xiuxian-daochang/tests/unit/agent/bootstrap_tests.rs`
  -> `packages/rust/crates/xiuxian-daochang/tests/agent/bootstrap/tests.rs`
- `packages/rust/crates/xiuxian-daochang/tests/unit/agent/session_context_tests.rs`
  -> `packages/rust/crates/xiuxian-daochang/tests/agent/session_context/tests.rs`
- `packages/rust/crates/xiuxian-daochang/tests/unit/agent/zhenfa_tests.rs`
  -> `packages/rust/crates/xiuxian-daochang/tests/agent/zhenfa/tests.rs`
- `packages/rust/crates/xiuxian-daochang/tests/unit/agent/zhenfa/valkey_hooks_tests.rs`
  -> `packages/rust/crates/xiuxian-daochang/tests/agent/zhenfa/valkey_hooks_tests.rs`
- `packages/rust/crates/xiuxian-daochang/tests/unit/session/redis_backend_tests.rs`
  -> `packages/rust/crates/xiuxian-daochang/tests/session/redis_backend/tests.rs`

Cleanup:

- removed empty `packages/rust/crates/xiuxian-daochang/tests/unit/**` directories.

## Structural Verification

```bash
rg -n "include!\\(\"unit/|#\\[path = \"unit/|mod tests \\{\\s*include!" packages/rust/crates/xiuxian-daochang/tests
```

Result:

- no matches.

## Validation Evidence

### 1) Targeted nextest (managed runtime lane)

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-daochang --test channels_managed_runtime_unit
```

Result:

- `7 passed`, `0 failed`, `0 skipped`.

### 2) Targeted nextest (remaining migrated harnesses)

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-daochang --test agent_bootstrap --test agent_zhenfa_unit --test agent_session_context_unit --test session_redis_backend_unit
```

Result:

- `39 passed`, `0 failed`, `1 skipped`.

### 3) Mandatory touched-crate clippy gate

```bash
RUSTC_WRAPPER= cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- success (exit 0).

Note:

- clippy still reports pre-existing upstream warnings outside this migration
  scope (`xiuxian-llm` doc-markdown and `xiuxian-daochang` provider-mode style/dead-code
  warnings), but no new failures were introduced by this structural refactor.

## Outcome

- `xiuxian-daochang/tests` no longer contains `unit/...` include-indirection.
- all touched harnesses now use module-local test declarations and package-top
  domain directories.
- legacy `tests/unit` tree for `xiuxian-daochang` is removed.
