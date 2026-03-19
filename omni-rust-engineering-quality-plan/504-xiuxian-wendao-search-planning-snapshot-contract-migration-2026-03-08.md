# 504. Xiuxian Wendao Search Planning Snapshot Contract Migration

Date: 2026-03-08

## Scope

This shard records the full-wave migration of the `LinkGraph` search-planning suite in `xiuxian-wendao` from internal direct tests to one fixture-backed snapshot contract binary.

The old suite lived under:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_core.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_core_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_policy.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_policy_fixture_support.rs`

It has now been replaced by:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_search_planning_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_search_planning_contract_support.rs`

## Why This Change Was Needed

After the quantum retrieval wave, the next strongest migration target was the planning surface around `search_planned` and `search_planned_payload` because it already behaved like a contract suite:

- search-core scenarios already lived in stable fixture trees,
- semantic-policy scenarios were fixture-backed and planning-specific,
- both families asserted the same planner-facing payload and option surfaces.

Migrating them together kept the review surface coherent and avoided another round of single-file churn.

## What Changed

### 1) Added a dedicated search-planning contract binary

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_search_planning_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_search_planning_contract_support.rs`

The new binary covers the whole planning family:

- baseline stats + hits,
- limit enforcement,
- direct-id short-circuit behavior,
- planned payload accounting,
- phrase/path-sort core search behavior,
- semantic-policy parsing and propagation.

Why this matters:

- one binary now owns the planner-facing `LinkGraph` contract surface,
- `search_core` and `semantic_policy` no longer evolve as isolated internal modules,
- support code moved under `tests/support/`, matching the external-contract pattern from earlier waves.

### 2) Reused existing fixtures and corrected one stale expectation

Reused the existing fixture roots under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/semantic_policy/`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/sort_path/expected/result.json`

Why this matters:

- the migration preserved the existing snapshot tree instead of inventing a new hierarchy,
- only one fixture needed adjustment, and it reflected real runtime behavior,
- the updated path-sort snapshot now captures live-saliency reranking on the leading FTS hit.

### 3) Removed the superseded internal module tree

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_core.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_core_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_policy.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_policy_fixture_support.rs`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

Why this matters:

- the migrated planning suite no longer coexists with the older internal-module form,
- the `test_link_graph` module graph is smaller and easier to audit,
- this wave continues the user's requirement to delete superseded direct tests immediately.

## Architectural Takeaways

### `search_planned` and `search_planned_payload` belong in one test family

The planning layer spans both hit production and retrieval-payload explanation. Keeping them in one contract binary makes regressions more legible.

### Semantic policy is a planning concern, not a quantum concern

Deferring `semantic_policy` out of the quantum wave was the right call. It fits naturally with `search_core` because both assert planner-visible behavior.

### Snapshot drift can reveal genuine product evolution

The `sort_path` mismatch was not flaky test noise; it surfaced a real contract change caused by live saliency. Snapshot migrations make these changes explicit instead of silently normalizing them away.

## Files Changed

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_search_planning_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_search_planning_contract_support.rs`
- `.cache/codex/execplans/wendao-test-snapshot-migration-wave-4-search-planning.md`

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_core.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_core_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_policy.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_policy_fixture_support.rs`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/sort_path/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-search-planning-contracts cargo check -p xiuxian-wendao --test test_link_graph_search_planning_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-search-planning-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph_search_planning_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-search-planning-contracts cargo clippy -p xiuxian-wendao --test test_link_graph_search_planning_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --test test_link_graph_search_planning_contracts --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph_search_planning_contracts` passed (`11 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph_search_planning_contracts -- -W clippy::too_many_lines` completed cleanly.

## Artifacts and Notes

- Contract binary: `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_search_planning_contracts.rs`
- Shared helper: `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_search_planning_contract_support.rs`
- Wave plan: `.cache/codex/execplans/wendao-test-snapshot-migration-wave-4-search-planning.md`
- Updated fixture due real contract drift: `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/sort_path/expected/result.json`
