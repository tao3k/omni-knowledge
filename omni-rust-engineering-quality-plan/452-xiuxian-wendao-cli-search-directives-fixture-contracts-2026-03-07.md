# 452. Xiuxian Wendao CLI Search Directives Fixture Contracts

Date: 2026-03-07

## Scope

This shard records the migration of the `test_wendao_cli/search/directives`
lane from inline notebook setup to fixture-backed `input/expected` contracts.

## Why This Change Was Needed

The directives lane still relied on runtime corpus assembly and imperative
assertions across five different CLI surfaces:

- inline query directives,
- query-level limit override,
- legacy `--sort` flag rejection,
- semantic filter flags,
- temporal filter flags.

Those tests were stable contract checks, but their input corpora and expected
payload shapes lived only inside Rust source.

## What Changed

### 1) Added a lane-specific directives contract support module

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/fixture_contract_support.rs`

This support file handles:

- materializing scenario inputs,
- projecting directive search payloads,
- projecting stable legacy-error semantics,
- normalizing filter seed counts, sort terms, and result rows.

### 2) Converted the directives module to use the support layer

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/mod.rs`

The lane now has one explicit support module and five focused contract tests.

### 3) Migrated all directive scenarios onto fixture-backed roots

Added scenario trees under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/query_directives_without_cli_flags/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/query_limit_directive_overrides_cli_limit/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/rejects_legacy_sort_flag/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/semantic_filter_flags/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/temporal_flags_filter_results/`

Each scenario now keeps its input corpus and one expected contract together.

### 4) Replaced imperative asserts with fixture-backed directive contracts

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/search_query_directives_apply_without_cli_flags.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/search_query_limit_directive_overrides_cli_limit.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/search_rejects_legacy_sort_flag.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/search_semantic_filter_flags.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/search_temporal_flags_filter_results.rs`

The lane now asserts one stable JSON contract per scenario instead of embedding
its expected semantics inside local assertions.

## Architectural Takeaways

- Directive parsing is a contract surface, not an implementation detail; the
  parsed query, limits, filters, and results belong in explicit fixtures.
- Error cases should also use fixture contracts, but only for stable semantics.
  Capturing `status_success = false` and key hint presence is better than
  snapshotting the whole stderr body.
- Lane-specific support files scale better than repeating payload plumbing in
  every test file.
- With `search/basic` and `search/directives` both migrated, the remaining CLI
  search debt is now concentrated in `link_filters` and `provisional_overlay`.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/fixture_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/search_query_directives_apply_without_cli_flags.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/search_query_limit_directive_overrides_cli_limit.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/search_rejects_legacy_sort_flag.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/search_semantic_filter_flags.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/directives/search_temporal_flags_filter_results.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/query_directives_without_cli_flags/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/query_directives_without_cli_flags/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/query_directives_without_cli_flags/input/docs/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/query_directives_without_cli_flags/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/query_limit_directive_overrides_cli_limit/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/query_limit_directive_overrides_cli_limit/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/query_limit_directive_overrides_cli_limit/input/docs/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/query_limit_directive_overrides_cli_limit/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/rejects_legacy_sort_flag/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/rejects_legacy_sort_flag/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/semantic_filter_flags/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/semantic_filter_flags/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/semantic_filter_flags/input/docs/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/semantic_filter_flags/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/temporal_flags_filter_results/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/temporal_flags_filter_results/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/temporal_flags_filter_results/input/docs/c.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_cli/search/directives/temporal_flags_filter_results/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_wendao_cli --message-format short
cargo nextest run -p xiuxian-wendao --test test_wendao_cli test_wendao_search_query_directives_apply_without_cli_flags test_wendao_search_query_limit_directive_overrides_cli_limit test_wendao_search_rejects_legacy_sort_flag test_wendao_search_semantic_filter_flags test_wendao_search_temporal_flags_filter_results --no-fail-fast
cargo clippy -p xiuxian-wendao --test test_wendao_cli -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check ...` completed cleanly.
- The targeted `cargo nextest run ...` passed (`5 passed, 31 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/451-xiuxian-wendao-cli-search-basic-fixture-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/452-xiuxian-wendao-cli-search-directives-fixture-contracts-2026-03-07.md`
