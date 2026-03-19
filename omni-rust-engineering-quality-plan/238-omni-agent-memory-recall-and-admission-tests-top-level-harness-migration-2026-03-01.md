# 238. `xiuxian-daochang` Memory-Recall and Admission Tests Top-Level Harness Migration (2026-03-01)

## Scope

- Remove remaining `src`-side test path mounts for:
  - `agent/memory_recall_feedback`
  - `agent/memory_recall_metrics`
  - `agent/admission`
- Migrate all three lanes to package-top `tests/` harness targets.
- Keep strict clippy warning-zero without lint suppression attributes.

## Changes

1. Removed `src`-side path mounts
- Files:
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall_feedback.rs`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall_metrics.rs`
  - `packages/rust/crates/xiuxian-daochang/src/agent/admission.rs`
- Removed `#[cfg(test)] #[path = "../../tests/..."] mod tests;` hooks.

2. Added top-level harness for `memory_recall_feedback`
- File:
  - `packages/rust/crates/xiuxian-daochang/tests/agent_memory_recall_feedback.rs`
- Added minimal local `agent::memory_recall::MemoryRecallPlan` model required by
  source feedback module and existing tests.
- Mounted source module and reused existing test file:
  - `packages/rust/crates/xiuxian-daochang/tests/agent/memory_recall_feedback.rs`
- Converted leading test-file inner doc comment to plain comment for stable
  `include!`-based harness compilation.

3. Added top-level harness for `memory_recall_metrics`
- File:
  - `packages/rust/crates/xiuxian-daochang/tests/agent_memory_recall_metrics.rs`
- Added minimal local `agent::memory_recall_state::SessionMemoryRecallDecision`
  and an `Agent` test stub with `memory_recall_metrics` field to satisfy source
  impl boundaries.
- Mounted source module and reused existing test file:
  - `packages/rust/crates/xiuxian-daochang/tests/agent/memory_recall_metrics.rs`
- Converted leading test-file inner doc comment to plain comment for stable
  `include!`-based harness compilation.

4. Added top-level harness for `admission`
- File:
  - `packages/rust/crates/xiuxian-daochang/tests/agent_admission.rs`
- Added minimal local `llm` and `embedding` snapshots/clients plus `Agent`
  test stub to satisfy source module dependencies.
- Mounted source module and reused existing test file:
  - `packages/rust/crates/xiuxian-daochang/tests/agent/admission.rs`

5. No suppression debt introduced
- All warning cleanup uses structural symbol probes (`let _ = ...;`) and typed
  minimal stubs.
- No `#[allow(...)]` was added at file/module scope.

## Validation Evidence

1. Per-lane strict clippy

```bash
cargo clippy -p xiuxian-daochang --test agent_memory_recall_feedback -- -W clippy::too_many_lines
cargo clippy -p xiuxian-daochang --test agent_memory_recall_metrics -- -W clippy::too_many_lines
cargo clippy -p xiuxian-daochang --test agent_admission -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: warning-zero for all three lanes.

2. Per-lane nextest

```bash
cargo nextest run -p xiuxian-daochang --test agent_memory_recall_feedback
cargo nextest run -p xiuxian-daochang --test agent_memory_recall_metrics
cargo nextest run -p xiuxian-daochang --test agent_admission
```

- Exit code: `0`
- Results:
  - `agent_memory_recall_feedback`: `10 passed`
  - `agent_memory_recall_metrics`: `4 passed`
  - `agent_admission`: `8 passed`

3. Aggregated migrated-lane validation

```bash
cargo clippy -p xiuxian-daochang --test agent_memory_recall_feedback --test agent_memory_recall_metrics --test agent_admission -- -W clippy::too_many_lines
cargo nextest run -p xiuxian-daochang --test agent_memory_recall_feedback --test agent_memory_recall_metrics --test agent_admission
```

- Exit code: `0`
- Result: clippy warning-zero; nextest `22 passed`, `0 failed`.

4. Mandatory touched-crate strict clippy

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: warning-zero.

## Outcome

- Three additional `src`-side path-mounted test hooks were removed.
- `xiuxian-daochang` now has only one remaining `src`-side `#[path="../../tests/..."]`
  mount (`agent/bootstrap.rs`), making bootstrap the final migration target in
  this normalization line.
