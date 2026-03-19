# 286. xiuxian-daochang memory-recall-state test remap elimination via test-support payload adapter (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Target lane: `tests/agent_memory_recall_state_unit.rs`
- Goal: remove `include!("../src/agent/memory_recall_state/mod.rs")` remap and keep persistence-compat tests on stable integration APIs.

## Implementation

1. Added crate-internal test adapter for raw snapshot payload injection:
   - `src/agent/memory_recall_state/mod.rs`
   - New helper: `test_append_memory_recall_snapshot_payload(...) -> anyhow::Result<()>`
2. Added stable test-support wrapper:
   - `src/test_support/memory_recall_state.rs`
   - New export: `append_memory_recall_snapshot_payload(...)`
3. Wired exports:
   - `src/test_support/mod.rs`
4. Migrated tests to the stable wrapper surface:
   - `tests/agent/memory_recall_state/tests.rs`
   - Removed direct `agent.session.append(...)` access.
   - Replaced direct method calls with `record_memory_recall_snapshot(...)` adapter.
5. Replaced top-level harness with standard package-top entrypoint:
   - `tests/agent_memory_recall_state_unit.rs`
   - Reduced to `#[path = "agent/memory_recall_state/tests.rs"] mod tests;`

## Verification

- Targeted regression:
  - `cargo nextest run -p xiuxian-daochang --test agent_memory_recall_state_unit`
  - result: `4 passed`, `2 skipped`, `0 failed`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass
- Remap debt counter:
  - `rg -n "include!\(\"\.\./src/|#\[path\s*=\s*\"\.\./src/|#\[path\s*=\s*\"\.\./\.\./src/" packages/rust/crates/xiuxian-daochang/tests --glob "*.rs" | wc -l`
  - result: `22 -> 21`
