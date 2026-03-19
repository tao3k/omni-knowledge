# 431. Xiuxian Wendao Build-Scope Fixture-Expected Contracts

Date: 2026-03-07

## Scope

This shard records the fourth Wendao LinkGraph test-architecture migration wave.

The wave moves the `build_scope` suite onto the same fixture-first testing model
already established for:

- hybrid quantum-retrieval tests,
- semantic-ignition tests,
- page-index tests.

The migration keeps the suite centered on per-scenario input corpora and stable
expected JSON contracts.

## Why This Change Was Needed

Before this slice, `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/build_scope.rs`
still relied on inline fixture construction for every scenario:

- excluded-directory filtering,
- hidden-directory default skipping,
- include-directory scoping,
- skill metadata promotion.

Although the module was smaller than `search_core`, it still repeated setup and
used assertion-only output checks for state that is easier to review as a full
contract.

## What Changed

### 1) Added build-scope-specific fixture projection support

New file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/build_scope_fixture_support.rs`

This support module owns:

- scenario materialization for build-scope tests,
- expected-fixture assertions,
- stable projection for `LinkGraphStats`,
- stable projection for `toc()` rows,
- stable projection for promoted skill documents.

Why this matters:

- build-scope contract shaping is no longer mixed into the main test module,
- the suite now follows the same domain-specific support pattern used by
  `page_index` and the hybrid retriever,
- later build-scope scenarios can extend one focused module instead of adding
  inline serialization logic to the tests themselves.

### 2) Migrated the full `build_scope` suite to per-scenario fixture contracts

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/build_scope.rs`

New input fixtures:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/excluded_dirs/input/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/hidden_dirs/input/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/include_dirs/input/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/skill_metadata/input/...`

New expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/excluded_dirs/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/hidden_dirs/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/include_dirs/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/skill_metadata/expected/result.json`

Why this matters:

- build filtering is now reviewed through full `stats + toc` contracts instead
  of scattered assertion pairs,
- skill metadata promotion is verified through one stable promoted-document
  contract,
- the suite no longer rebuilds test trees inline.

### 3) Extended the LinkGraph test lane without introducing generic helpers

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

Why this matters:

- `build_scope_fixture_support.rs` is named by the capability it owns,
- the lane keeps following the repository rule that modularity should reflect
  feature/domain intent,
- fixture-first support remains discoverable and bounded.

## Architectural Takeaways

### Filtering behavior deserves contract-level verification

Directory inclusion/exclusion and hidden-tree skipping are not trivial details.
They define which graph corpus exists at all, so a single `stats + toc` contract
is a better regression shape than isolated numeric assertions.

### Metadata promotion should be snapshotted at the document boundary

Skill metadata promotion is easier to reason about when the final promoted
`LinkGraphDocument` row is visible directly in the expected contract.

### Small modules are still worth migrating when they repeat setup

This suite was not large, but it still had enough repeated corpus setup and
multi-field output checks to benefit from the fixture-first model.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/build_scope.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/build_scope_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/excluded_dirs/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/excluded_dirs/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/excluded_dirs/input/.cache/huge.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/excluded_dirs/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/hidden_dirs/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/hidden_dirs/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/hidden_dirs/input/.github/hidden.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/hidden_dirs/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/include_dirs/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/include_dirs/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/include_dirs/input/assets/knowledge/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/include_dirs/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/skill_metadata/input/skills/demo/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/skill_metadata/input/skills/demo/references/rules.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/build_scope/skill_metadata/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
cargo check -p xiuxian-wendao --tests --message-format short
cargo nextest run -p xiuxian-wendao --test test_link_graph build_scope
cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph build_scope` passed (`4 passed, 80 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines` completed cleanly.

## Limits and Next Slice

The remaining highest-value migration target inside `test_link_graph` is now the
larger search-oriented lane, especially:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_core.rs`

That module still contains the heaviest concentration of inline corpus setup and
repeated output-shape assertions. It should be the next major fixture-first
migration candidate.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/430-xiuxian-wendao-page-index-fixture-expected-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/431-xiuxian-wendao-build-scope-fixture-expected-contracts-2026-03-07.md`
