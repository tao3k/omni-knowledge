# 448. Xiuxian Wendao Link-Graph Inline Corpus Fixture Contracts

Date: 2026-03-07

## Scope

This shard records the migration of the remaining small inline-corpus
`xiuxian-wendao` link-graph tests onto fixture-backed contracts under
`tests/fixtures/.../expected`.

## Why This Change Was Needed

After the earlier snapshot cleanup, the next remaining debt in
`xiuxian-wendao/tests` was not snapshot infrastructure but inline corpus setup:
small tests were still creating markdown notebooks in `tempdir()` at runtime and
asserting behavior with narrow positional checks.

That structure had three drawbacks:

- test inputs were embedded in Rust code instead of living beside their expected
  outputs,
- `#[tokio::test] async fn` remained in places with no async work,
- richer behaviors such as agentic expansion telemetry had no stable fixture
  contract to document the intended surface.

## What Changed

### 1) Moved weighted-seed PPR and mixed-topology lanes onto fixture trees

Replaced inline notebook writes in:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_ppr_weighting.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_mixed_graph_topology.rs`

with fixture-backed inputs under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/ppr_weighting/non_uniform_seed_bias/input/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/mixed_topology/weighted_seed_exposes_linked_entities/input/`

Each lane now projects its related-node result set into stable JSON and asserts
against `expected/result.json`.

### 2) Converted sync-only tests from `#[tokio::test]` to plain `#[test]`

The PPR-weighting and mixed-topology lanes had no async work. They now use plain
`#[test]`, which better matches the execution model and keeps the test surface
honest.

### 3) Migrated agentic-expansion planning and execution onto fixture contracts

Replaced runtime file writes in:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic_expansion.rs`

with scenario fixtures under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/worker_and_pair_budgets/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/query_narrows_candidates/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/execution_without_persistence/`

A dedicated projection helper now converts planner and execution structures into
stable JSON contracts while normalizing volatile timing fields into boolean
invariants such as `elapsed_ms_non_negative`.

### 4) Introduced one domain-specific support module for agentic telemetry

Added:

- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_agentic_expansion_fixture_support.rs`

The support file is intentionally domain-specific. It exists because the
agentic-expansion lane has a wider surface than the small related-node tests and
needs stable projections for:

- worker partitions,
- pair priorities,
- execution config,
- worker telemetry phases,
- uniqueness and budget invariants.

## Architectural Takeaways

- Once a test lane stabilizes around scenario inputs and expected outputs, the
  corpus belongs under `tests/fixtures`, not inline in Rust source.
- Small behavior tests do not need new generic helpers; use the existing
  materialization/assertion infrastructure directly unless a lane has a genuinely
  larger domain surface.
- Runtime telemetry should not be asserted as raw durations. Project unstable
  fields into explicit invariants and keep the contract focused on structure and
  semantics.
- `#[tokio::test]` should be reserved for tests that actually need async
  execution.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_ppr_weighting.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_mixed_graph_topology.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic_expansion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_agentic_expansion_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/ppr_weighting/non_uniform_seed_bias/input/A.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/ppr_weighting/non_uniform_seed_bias/input/B.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/ppr_weighting/non_uniform_seed_bias/input/C.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/ppr_weighting/non_uniform_seed_bias/input/D.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/ppr_weighting/non_uniform_seed_bias/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/mixed_topology/weighted_seed_exposes_linked_entities/input/note.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/mixed_topology/weighted_seed_exposes_linked_entities/input/EntityA.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/mixed_topology/weighted_seed_exposes_linked_entities/input/EntityB.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/mixed_topology/weighted_seed_exposes_linked_entities/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/worker_and_pair_budgets/input/notes/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/worker_and_pair_budgets/input/notes/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/worker_and_pair_budgets/input/notes/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/worker_and_pair_budgets/input/notes/d.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/worker_and_pair_budgets/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/query_narrows_candidates/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/query_narrows_candidates/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/query_narrows_candidates/input/docs/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/query_narrows_candidates/input/docs/d.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/query_narrows_candidates/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/execution_without_persistence/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/execution_without_persistence/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/execution_without_persistence/input/docs/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/execution_without_persistence/input/docs/d.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/agentic_expansion/execution_without_persistence/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_link_graph_ppr_weighting --test test_mixed_graph_topology --test test_link_graph_agentic_expansion --message-format short
cargo nextest run -p xiuxian-wendao --test test_link_graph_ppr_weighting --test test_mixed_graph_topology --test test_link_graph_agentic_expansion
cargo clippy -p xiuxian-wendao --test test_link_graph_ppr_weighting --test test_mixed_graph_topology --test test_link_graph_agentic_expansion -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check ...` completed cleanly.
- `cargo nextest run ...` passed (`5 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/447-xiuxian-wendao-parser-contract-fixture-migration-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/448-xiuxian-wendao-link-graph-inline-corpus-fixture-contracts-2026-03-07.md`
