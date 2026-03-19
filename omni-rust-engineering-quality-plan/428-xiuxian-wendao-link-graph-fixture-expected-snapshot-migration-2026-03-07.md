# 428. Xiuxian Wendao Link-Graph Fixture-Expected Snapshot Migration

Date: 2026-03-07

## Scope

This shard records a Wendao test-architecture refinement for the LinkGraph
hybrid-retrieval lane.

The goal was not to add a new snapshot framework. The goal was to replace
repetitive inline tempdir setup and field-by-field assertions with a cleaner,
fixture-first contract style:

- input documents live under `tests/fixtures/.../input/`,
- expected JSON contracts live under `tests/fixtures/.../expected/`,
- tests project runtime results into stable JSON payloads and compare against
  the expected fixture.

This slice intentionally keeps the directory rooted in `tests/fixtures/`
instead of introducing another `tests/snapshots/` tree for the new coverage.

## Why This Change Was Needed

The previous `test_link_graph/quantum_fusion.rs` and
`test_link_graph/quantum_anchor_batch.rs` suites had three recurring problems:

- the same `alpha`/`beta`/`gamma`/`plain` Markdown corpus was re-created inside
  multiple tests,
- tests mixed fixture construction with behavior assertions, making intent
  harder to read,
- structural assertions were repeated field by field even when the real goal was
  to validate the full retrieved context contract.

That shape creates maintenance drag. Every schema change or output-shape change
forces hand edits across many tests.

## What Changed

### 1) Added fixture-first JSON assertion support

New file:

- `packages/rust/crates/xiuxian-wendao/tests/support/fixture_json_assertions.rs`

This helper compares a runtime `serde_json::Value` against an expected JSON file
stored under `tests/fixtures/<root>/...`.

Why this matters:

- new snapshot-style suites no longer depend on a separate snapshot root,
- JSON comparison stays structural instead of text-fragile,
- expected contracts now live next to the input corpus they describe.

### 2) Added a shared materialized hybrid corpus fixture

New file:

- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_hybrid_fixture.rs`

New fixture tree:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/input/docs/alpha.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/input/docs/beta.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/input/docs/gamma.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/input/notes/plain.md`

The helper copies that corpus into a temp directory, builds `LinkGraphIndex`,
and exposes the stable alpha leaf anchor id for hybrid-retrieval tests.

Why this matters:

- all migrated tests now share one canonical corpus,
- fixture setup is no longer duplicated inline,
- anchor-resolution setup is centralized and explicit.

### 3) Extracted LinkGraph quantum snapshot projection support

New file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fixture_support.rs`

This module owns:

- JSON projection for `QuantumContext`,
- deterministic sorting of `related_clusters`,
- score rounding for stable contract files,
- a dedicated page-index fallback snapshot projection,
- shared hybrid fixture access for the migrated suites.

Why this matters:

- `quantum_fusion.rs` and `quantum_anchor_batch.rs` now read like behavioral
  tests instead of serialization noise,
- all snapshot shaping logic lives in one place,
- future hybrid-retrieval test slices can reuse the same projection rules.

### 4) Migrated `quantum_fusion` to fixture-expected contracts

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fusion.rs`

New expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_fusion/contexts_from_anchors.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_fusion/page_index_doc_fallback.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_fusion/duplicate_anchor_rows.json`

The suite now validates full hybrid-context shape through fixture-backed JSON
contracts rather than repeated inline assertions.

### 5) Migrated `quantum_anchor_batch` success-path coverage to the same pattern

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`

New expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/custom_columns.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/duplicate_rows.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/doc_fallback.json`

The success-path coverage now shares the same fixture corpus and expected JSON
style as `quantum_fusion`.

Error-path coverage stays assertion-based for now because the primary benefit of
fixture-expected migration is output-contract readability, not serializing every
small enum match.

### 6) Cleaned neighboring LinkGraph test-lane lint debt discovered during the migration

Updated files:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/batch_quantum_scorer.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_ignition.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/snapshot_assertions.rs`

What changed:

- replaced `expect_err(...)`-style failures with explicit `match` branches,
- improved local naming in `batch_quantum_scorer.rs`,
- tightened `backend_name()` return types to `&'static str`,
- removed the nested JSON-extension check warning in the legacy
  `snapshot_assertions.rs` helper.

Why this matters:

- the `test_link_graph` target now passes strict clippy cleanly,
- the migration did not leave the touched test lane in a mixed-quality state,
- fixture migration and lint convergence moved together.

## Architectural Takeaways

### Fixtures and expected contracts are different concerns, but they can share one root

For this repository, `tests/fixtures/<suite>/input` and
`tests/fixtures/<suite>/expected` is a cleaner model than multiplying top-level
folders. Inputs and expected contracts stay co-located without collapsing their
roles.

### Snapshot projection should be owned by a named support module

Tests should not inline JSON-shaping noise. When multiple suites assert the same
runtime record type, create one projection helper and keep the suites focused on
behavior.

### Migrate by repetition density first

The best first candidates were the hybrid retrieval tests because they repeated
both corpus setup and output assertions. That produced a visible reduction in
noise with a small, local change set.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/support/fixture_json_assertions.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_hybrid_fixture.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/snapshot_assertions.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/batch_quantum_scorer.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_anchor_batch.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/quantum_fusion.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_ignition.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/input/docs/alpha.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/input/docs/beta.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/input/docs/gamma.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/input/notes/plain.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_fusion/contexts_from_anchors.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_fusion/page_index_doc_fallback.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_fusion/duplicate_anchor_rows.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/custom_columns.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/duplicate_rows.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/hybrid/expected/quantum_anchor_batch/doc_fallback.json`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
cargo check -p xiuxian-wendao --tests --message-format short
cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines
cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_contexts
cargo nextest run -p xiuxian-wendao --test test_link_graph batch_quantum_scorer
cargo nextest run -p xiuxian-wendao --test test_link_graph test_page_index_semantic_path_supports_anchor_and_doc_fallbacks
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests --message-format short` completed cleanly.
- `cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph quantum_contexts` passed (`13 passed, 71 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph batch_quantum_scorer` passed (`3 passed, 81 skipped`).
- `cargo nextest run -p xiuxian-wendao --test test_link_graph test_page_index_semantic_path_supports_anchor_and_doc_fallbacks` passed (`1 passed, 83 skipped`).

Executed and blocked by unrelated pre-existing test-target debt:

```bash
cargo clippy -p xiuxian-wendao --tests -- -W clippy::too_many_lines
```

Observed blockers outside this migrated LinkGraph lane:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_resource_registry_snapshots.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_snapshots.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_resolver.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_uri_snapshots.rs`

The dominant blocker category is existing `expect` / `expect_err` usage in other
integration-test targets. This shard does not treat that crate-wide cleanup as
part of the LinkGraph fixture migration scope.

## Limits and Next Slice

This slice migrated the highest-value repeated LinkGraph hybrid suites, but it
is not the end state for all Wendao tests.

The next migration candidates should be chosen by the same rule:

- shared tempdir corpus setup,
- repeated JSON-shape assertions,
- stable domain outputs that benefit from fixture-backed expected contracts.

The best follow-up candidates are the remaining LinkGraph suites that still
repeat inline Markdown corpus construction and multi-field assertion blocks.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/427-xiuxian-wendao-topology-expansion-stage-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/428-xiuxian-wendao-link-graph-fixture-expected-snapshot-migration-2026-03-07.md`
