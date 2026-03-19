# 450. Xiuxian Wendao Seed-and-Priors Fixture Contracts

Date: 2026-03-07

## Scope

This shard records the migration of the `test_link_graph_seed_and_priors` lane
from inline corpus setup to fixture-backed `input/expected` contracts.

## Why This Change Was Needed

The `seed_and_priors` lane still carried three forms of structural debt:

- runtime markdown corpus assembly via `TempDir` and `write_file`,
- `mod.rs` carrying implementation helpers instead of acting as an interface
  boundary,
- behavior verification expressed through imperative local assertions rather than
  reusable contract projections.

That structure made the test inputs hard to inspect and left the module layout
behind the standards already applied across the rest of `xiuxian-wendao/tests`.

## What Changed

### 1) Moved the lane onto fixture-backed scenario trees

Added scenario roots under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/seed_accuracy_cluster_grounded/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/structural_priors_architecture_hub/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/journal_semantic_pull_surfaces_agenda/`

Each scenario now keeps its markdown inputs beside an `expected/result.json`
contract.

### 2) Converted the integration test root to use shared fixture modules

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors.rs`

It now exposes the shared crate-level fixture modules used by the nested
`test_link_graph_seed_and_priors/` submodules.

### 3) Restored `mod.rs` to interface-only responsibility

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors/mod.rs`

The file now only declares submodules. Shared implementation moved into a
focused domain support file.

### 4) Added a domain-specific support module for contract projections

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors/fixture_contract_support.rs`

This support file handles:

- fixture materialization,
- stable hit projection,
- structural-prior comparison snapshots,
- expected-fixture assertions.

### 5) Migrated all three scenarios off inline corpus setup

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors/link_graph_related_filter_seed_accuracy_is_cluster_grounded.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors/link_graph_structural_priors_promote_architecture_hub_top3.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors/link_graph_related_journal_semantic_pull_surfaces_agenda_tasks.rs`

Each test now reads from a scenario fixture root and asserts one JSON contract
instead of building the notebook inline and scattering behavior checks.

## Architectural Takeaways

- If a split integration-test lane grows beyond one-off assertions, give it one
  domain-specific fixture support file and keep `mod.rs` interface-only.
- Structural-retrieval tests become much easier to review when both the graph
  corpus and the expected ranked hits live in fixture directories.
- Comparing graph-prior behavior works better as a single structured contract
  containing boosted/baseline views than as several disjoint asserts.
- Seed-grounded retrieval and section-surface behavior should be documented as
  fixture contracts because they are exactly the sort of ranking semantics that
  regress quietly.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors/fixture_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors/link_graph_related_filter_seed_accuracy_is_cluster_grounded.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors/link_graph_structural_priors_promote_architecture_hub_top3.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors/link_graph_related_journal_semantic_pull_surfaces_agenda_tasks.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/seed_accuracy_cluster_grounded/input/docs/arch-seed.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/seed_accuracy_cluster_grounded/input/docs/arch-a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/seed_accuracy_cluster_grounded/input/docs/arch-b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/seed_accuracy_cluster_grounded/input/docs/arch-c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/seed_accuracy_cluster_grounded/input/docs/db-seed.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/seed_accuracy_cluster_grounded/input/docs/db-a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/seed_accuracy_cluster_grounded/input/docs/db-b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/seed_accuracy_cluster_grounded/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/structural_priors_architecture_hub/input/docs/hub.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/structural_priors_architecture_hub/input/docs/leaf-a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/structural_priors_architecture_hub/input/docs/leaf-b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/structural_priors_architecture_hub/input/docs/leaf-c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/structural_priors_architecture_hub/input/docs/ref-0.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/structural_priors_architecture_hub/input/docs/ref-1.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/structural_priors_architecture_hub/input/docs/ref-2.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/structural_priors_architecture_hub/input/docs/ref-3.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/structural_priors_architecture_hub/input/docs/ref-4.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/structural_priors_architecture_hub/input/docs/ref-5.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/structural_priors_architecture_hub/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/journal_semantic_pull_surfaces_agenda/input/docs/journal/journal-entry-2026-02-26.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/journal_semantic_pull_surfaces_agenda/input/docs/agenda/agenda-tasks-2026-02-26.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/journal_semantic_pull_surfaces_agenda/input/docs/agenda/2026-02-27.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/seed_and_priors/journal_semantic_pull_surfaces_agenda/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_link_graph_seed_and_priors --message-format short
cargo nextest run -p xiuxian-wendao --test test_link_graph_seed_and_priors
cargo clippy -p xiuxian-wendao --test test_link_graph_seed_and_priors -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check ...` completed cleanly.
- `cargo nextest run ...` passed (`3 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/449-xiuxian-wendao-ppr-and-cli-related-fixture-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/450-xiuxian-wendao-seed-and-priors-fixture-contracts-2026-03-07.md`
