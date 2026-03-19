# 432. Xiuxian Wendao Search-Core Fixture-Expected Contracts

Date: 2026-03-07

## Scope

This shard records the next Wendao LinkGraph test-architecture slice: migrating
`search_core` from inline corpus setup plus repeated field assertions to the
fixture-first contract model already established for the surrounding suites.

This slice intentionally keeps the expected outputs under
`tests/fixtures/.../expected/`.
It does not introduce a new `tests/snapshots/` root.

## Why This Change Was Needed

`packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_core.rs`
covered the most central `LinkGraph` search behaviors, but it still carried the
same maintenance problems that earlier migrations removed elsewhere:

- search scenarios were coupled directly to runtime setup,
- output-shape checks were repeated test by test,
- retrieval-plan payload assertions were harder to read than the actual search
  behavior they were meant to protect.

That shape is especially expensive in `search_core` because it spans both plain
search hits and richer planned-payload responses.

## What Changed

### 1) Added a focused `search_core` fixture support module

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_core_fixture_support.rs`

This module owns:

- scenario materialization from `tests/fixtures/link_graph/search_core/...`,
- stable JSON projection for `LinkGraphStats`, `LinkGraphHit`,
  `LinkGraphDisplayHit`, and `LinkGraphRetrievalPlanRecord`,
- score rounding for deterministic contract files,
- a single assertion entrypoint for expected JSON comparison.

Why this matters:

- `search_core.rs` stays focused on behavior,
- payload-shaping rules live in one domain-specific place,
- future search-lane additions can extend the same projection surface without
  reintroducing inline assertion sprawl.

### 2) Migrated the full `search_core` suite to fixture-backed expected contracts

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_core.rs`

New scenario fixture trees:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/baseline/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/limit/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/direct_id/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/direct_id_payload/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/fts_boost/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/phrase_specific/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/sort_path/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/payload_counts/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/payload_empty_graph/...`

New expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/baseline/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/limit/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/direct_id/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/direct_id_payload/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/fts_boost/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/phrase_specific/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/sort_path/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/payload_counts/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/payload_empty_graph/expected/result.json`

What the contracts now cover:

- baseline search plus index stats,
- search-limit enforcement,
- `id:` short-circuit behavior,
- planned payload output for direct-id retrieval,
- graph-rank influence on FTS ordering,
- phrase-specific ranking against generic index terms,
- explicit path sorting behavior,
- retrieval-plan count consistency,
- hybrid escalation when graph hits are absent.

Why this matters:

- the most central search lane now has readable contract files,
- payload regressions are diffable as structured JSON instead of scattered
  assertions,
- the fixture tree captures both corpus input and expected behavior in one
  place.

### 3) Removed stale imports from the shared LinkGraph test root

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

What changed:

- removed no-longer-used imports left behind from the old inline-assertion era,
- kept the root module focused on shared helpers and module declarations.

Why this matters:

- the test lane stays warning-clean,
- fixture migration does not leave dead assertion-era scaffolding behind.

## Architectural Takeaways

### Search output shape is a contract surface

`search_core` is not just validating ranking internals. It defines the stable
contract that downstream callers observe: hit ordering, section attribution,
retrieval-plan mode selection, and graph-confidence metadata. That surface is a
better fit for fixture-backed JSON contracts than for piecemeal inline
assertions.

### Domain-specific support beats generic helpers

`search_core_fixture_support.rs` is the right module name because it says
exactly what it owns. The repository should keep following this pattern instead
of growing another `helpers.rs` bucket.

### Fixtures should carry both input and expectation

A scenario folder that contains `input/` plus `expected/` is the cleanest test
artifact for this project. It keeps the corpus, the expected behavior, and the
scenario name aligned without adding a second snapshot taxonomy.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_core.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_core_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/baseline/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/direct_id/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/direct_id_payload/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/fts_boost/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/limit/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/payload_counts/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/payload_empty_graph/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/phrase_specific/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_core/sort_path/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
cargo check -p xiuxian-wendao --tests --message-format short
cargo nextest run -p xiuxian-wendao --test test_link_graph search_core
cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph search_core`
  passed (`9 passed, 75 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines`
  completed cleanly.

## Limits and Next Slice

This slice closes the biggest remaining repetitive search-contract lane inside
`test_link_graph`, but there is still follow-up work available in the broader
Wendao test surface:

- convert other remaining assertion-heavy suites to the same
  `tests/fixtures/<suite>/<scenario>/{input,expected}` structure,
- continue removing stale root-module imports or helper leakage as suites move,
- keep the migrated lane strict-clippy clean as new scenarios are added.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/431-xiuxian-wendao-build-scope-fixture-expected-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/432-xiuxian-wendao-search-core-fixture-expected-contracts-2026-03-07.md`
