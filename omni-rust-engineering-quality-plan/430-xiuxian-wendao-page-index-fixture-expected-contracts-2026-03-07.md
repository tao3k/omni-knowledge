# 430. Xiuxian Wendao Page-Index Fixture-Expected Contracts

Date: 2026-03-07

## Scope

This shard records the third Wendao LinkGraph test-architecture migration wave.

The wave brings the `page_index` suite onto the same fixture-first contract
model already established for the hybrid-retrieval lane:

- input corpora live under `tests/fixtures/.../input/`,
- expected behavior contracts live under `tests/fixtures/.../expected/`,
- test bodies focus on runtime behavior instead of inline corpus construction
  and repeated field-level assertions.

## Why This Change Was Needed

Before this slice, `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index.rs`
still relied on repeated inline setup for every scenario:

- hierarchy construction,
- headingless document fallback,
- thinning behavior,
- incremental refresh,
- semantic-document export.

That created the same problems already seen in the quantum-fusion tests:

- duplicated Markdown corpus construction,
- output-shape assertions spread across many lines,
- no single expected contract file for reviewing a page-index regression.

## What Changed

### 1) Added a shared `LinkGraph` fixture-tree materializer

New file:

- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_fixture_tree.rs`

This helper materializes any `tests/fixtures/link_graph/...` input tree into a
fresh temp directory.

Why this matters:

- fixture copying is now centralized instead of reimplemented in each support
  module,
- the existing hybrid fixture support could reuse the same materialization path,
- future LinkGraph fixture migrations can build on one stable mechanism.

### 2) Refactored the hybrid fixture helper to use the shared materializer

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_hybrid_fixture.rs`

Why this matters:

- LinkGraph fixture support now has one physical source of truth for copying
  fixture trees,
- the newer page-index fixture support and the earlier hybrid support no longer
  diverge in implementation style.

### 3) Added page-index-specific snapshot projection support

New file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index_fixture_support.rs`

This module owns:

- scenario-specific page-index fixture materialization,
- expected-contract assertions for page-index scenarios,
- recursive `PageIndexNode` projection,
- ordered semantic-document projection,
- refresh update-fixture loading.

Why this matters:

- page-index serialization logic no longer pollutes the main test suite,
- all page-index contract shaping lives in one support module,
- output normalization rules are explicit and reusable.

### 4) Migrated the entire `page_index` suite to fixture-expected contracts

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index.rs`

New scenario fixtures:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/hierarchy/input/docs/alpha.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/headingless/input/notes/plain.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/thinning/input/docs/thin.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/refresh/input/docs/refresh.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/refresh/update/docs/refresh.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/semantic_documents/input/docs/alpha.md`

New expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/hierarchy/expected/tree.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/headingless/expected/tree.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/thinning/expected/tree.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/refresh/expected/incremental_refresh.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/semantic_documents/expected/documents.json`

Why this matters:

- every major page-index scenario now has a readable expected contract file,
- refresh behavior is preserved as a before/after contract instead of two small
  inline assertions,
- semantic-document export is verified as a stable serialized contract rather
  than ad hoc counting and spot checks.

### 5) Extended the `test_link_graph` support surface cleanly

Updated files:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

Why this matters:

- fixture support remains explicit at the test-root boundary,
- the `page_index` suite can use focused helpers without leaking generic helper
  names into every module,
- the lane still respects the repository rule that test structure should be
  modular and domain-based.

## Architectural Takeaways

### Page-index behavior is a contract surface, not just implementation detail

Hierarchy layout, thinning, incremental refresh, and semantic export all shape
how downstream retrieval stages behave. They deserve expected-contract files,
not just a few tactical assertions.

### Support modules should mirror domain slices

`page_index_fixture_support.rs` is a better home for projection and scenario
loading than a generic `helpers.rs`-style file because the module names exactly
what it owns.

### Reuse the fixture corpus mechanism once it exists

After hybrid retrieval introduced shared fixture-tree materialization, the next
correct step was not another custom tempdir helper. The right move was to extend
that mechanism and keep fixture handling converged.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_fixture_tree.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_hybrid_fixture.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/page_index_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/hierarchy/input/docs/alpha.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/hierarchy/expected/tree.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/headingless/input/notes/plain.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/headingless/expected/tree.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/thinning/input/docs/thin.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/thinning/expected/tree.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/refresh/input/docs/refresh.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/refresh/update/docs/refresh.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/refresh/expected/incremental_refresh.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/semantic_documents/input/docs/alpha.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/semantic_documents/expected/documents.json`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
cargo check -p xiuxian-wendao --tests --message-format short
cargo nextest run -p xiuxian-wendao --test test_link_graph page_index
cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph page_index` passed (`6 passed, 78 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines` completed cleanly.

## Limits and Next Slice

This slice migrated the full page-index suite, but large search-oriented
LinkGraph suites still contain the highest remaining concentration of inline
fixture creation.

The next migration candidate should come from the remaining heavy repeaters,
especially:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/build_scope.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_core.rs`

The better next step is still to prioritize repetition density and stable JSON
contract payoff.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/429-xiuxian-wendao-semantic-ignition-fixture-expected-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/430-xiuxian-wendao-page-index-fixture-expected-contracts-2026-03-07.md`
