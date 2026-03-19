# 453. Xiuxian Wendao CLI Search Link Filters And Provisional Fixture Contracts

Date: 2026-03-07

## Scope

This shard records the migration of the remaining `test_wendao_cli/search`
lanes from inline setup to fixture-backed `input/expected` contracts:

- `link_filters`
- `provisional_overlay`

## Why This Change Was Needed

These two lanes were the last CLI search modules still encoding their contract
semantics inside test bodies. They mixed runtime corpus assembly, imperative
assertions, and search-payload plumbing directly inside the Rust tests.

That structure had three costs:

- it hid stable CLI behavior inside per-test setup code,
- it duplicated result projection logic across neighboring scenarios,
- it kept the final search lanes out of the same fixture-first standard already
  adopted by `search/basic` and `search/directives`.

## What Changed

### 1) Added lane-specific support modules

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/link_filters_fixture_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/provisional_overlay_fixture_contract_support.rs`

These support files now own scenario materialization and stable payload
projection for their respective lanes.

### 2) Converted the remaining search lane tests to fixture contracts

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/link_filters.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/provisional_overlay.rs`

The tests now read scenario roots from `tests/fixtures/...`, execute the CLI
surface, project only the stable semantic payload, and compare against a single
expected JSON contract.

### 3) Added explicit scenario trees for link filters and provisional overlay

Added fixture trees under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/link_filters/link_to_filter/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/link_filters/related_ppr_filter/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/provisional_overlay/include_provisional_cli_flag/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/provisional_overlay/include_provisional_engine_default/`

Each scenario now keeps its corpus and contract in one rooted directory.

### 4) Closed the remaining CLI search fixture gap

With these two lanes migrated, the `test_wendao_cli/search` surface is now
uniformly fixture-backed across:

- `basic`
- `directives`
- `link_filters`
- `provisional_overlay`

## Architectural Takeaways

- Search-filter semantics belong in explicit JSON contracts, not in scattered
  assertions that reconstruct expectations inside test code.
- Provisional overlay behavior is especially suited to contract testing because
  the stable API surface is semantic: injected paths, suggestions, and final
  result rows. That is more durable than snapshotting raw CLI text.
- Lane-specific support modules keep projection logic modular without polluting
  neighboring lanes or collapsing everything into a generic helper.
- Completing the whole `test_wendao_cli/search` family under the same fixture
  discipline materially improves maintainability and makes new scenarios cheap
  to add.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/link_filters.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/link_filters_fixture_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/provisional_overlay.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/provisional_overlay_fixture_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/link_filters/link_to_filter/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/link_filters/link_to_filter/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/link_filters/link_to_filter/input/docs/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/link_filters/link_to_filter/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/link_filters/related_ppr_filter/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/link_filters/related_ppr_filter/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/link_filters/related_ppr_filter/input/docs/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/link_filters/related_ppr_filter/input/docs/d.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/link_filters/related_ppr_filter/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/provisional_overlay/include_provisional_cli_flag/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/provisional_overlay/include_provisional_cli_flag/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/provisional_overlay/include_provisional_cli_flag/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/provisional_overlay/include_provisional_engine_default/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/provisional_overlay/include_provisional_engine_default/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/provisional_overlay/include_provisional_engine_default/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo nextest run -p xiuxian-wendao --test test_wendao_cli test_wendao_search_link_filters_flags test_wendao_search_related_ppr_flags test_wendao_search_can_include_provisional_suggestions test_wendao_search_uses_engine_default_for_provisional_injection --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- The targeted `cargo nextest run ...` passed (`4 passed, 32 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- Prior prerequisite shards:
  - `assets/knowledge/omni-rust-engineering-quality-plan/451-xiuxian-wendao-cli-search-basic-fixture-contracts-2026-03-07.md`
  - `assets/knowledge/omni-rust-engineering-quality-plan/452-xiuxian-wendao-cli-search-directives-fixture-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/453-xiuxian-wendao-cli-search-link-filters-and-provisional-fixture-contracts-2026-03-07.md`
