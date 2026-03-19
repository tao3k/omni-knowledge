# 268. xiuxian-daochang memory_recall and reflection Test Remap Elimination (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-daochang`
- Objective:
  - remove source remap dependency in `agent_memory_recall_unit` and
    `agent_reflection_unit`,
  - route these lanes through stable test-support boundaries,
  - keep touched crate green under required nextest/clippy gates.

## Changes

### 1) Removed direct `#[path = "../src/..."]` from two agent test harnesses

Updated:

- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_recall_unit.rs`
- `packages/rust/crates/xiuxian-daochang/tests/agent_reflection_unit.rs`

Actions:

- deleted source remap includes for:
  - `src/agent/memory_recall/mod.rs`,
  - `src/agent/reflection/mod.rs`,
- rewired harnesses to import only from `xiuxian_daochang::test_support`,
- kept package-top `tests/agent/**` test bodies unchanged.

### 2) Added stable test-support wrappers for memory recall and reflection

Added:

- `packages/rust/crates/xiuxian-daochang/src/test_support/memory_recall.rs`
- `packages/rust/crates/xiuxian-daochang/src/test_support/reflection.rs`

Updated:

- `packages/rust/crates/xiuxian-daochang/src/test_support/mod.rs`

Actions:

- introduced test-facing wrappers/types for:
  - memory recall planning/filtering/context synthesis,
  - reflection runtime transition flow and policy-hint derivation,
- mapped wrapper payloads to internal implementation types without exposing
  private module layout to integration tests.

### 3) Internal visibility alignment for wrapper access

Updated:

- `packages/rust/crates/xiuxian-daochang/src/agent/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/reflection/mod.rs`

Actions:

- made `agent::memory_recall` and `agent::reflection` crate-visible to allow
  test-support adapters,
- added crate-visible export for deterministic ranking helper:
  - `filter_recalled_episodes_at`,
- kept reflection symbols crate-visible (not public API) to avoid
  `missing_docs` warning expansion.

## Validation Evidence

### 1) Targeted nextest

```bash
cargo nextest run -p xiuxian-daochang --test agent_memory_recall_unit --test agent_reflection_unit
```

Result:

- `11 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory touched-crate clippy gate

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0), no warnings/errors.

### 3) Structural proof command

```bash
rg -n "#\\[path\\s*=\\s*\\\"\\.\\./src/|#\\[path\\s*=\\s*\\\"\\.\\./\\.\\./src/\" \
  packages/rust/crates/xiuxian-daochang/tests/agent_memory_recall_unit.rs \
  packages/rust/crates/xiuxian-daochang/tests/agent_reflection_unit.rs
```

Result:

- no matches.

## Outcome

- `agent_memory_recall_unit` and `agent_reflection_unit` now validate through
  stable test-support contracts rather than source remapping,
- test harness layout remains package-top and module-local,
- touched crate remains green under required quality gates.
