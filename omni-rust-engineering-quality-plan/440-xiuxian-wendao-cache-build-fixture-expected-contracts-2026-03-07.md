# 440. Xiuxian Wendao Cache-Build Fixture-Expected Contracts

Date: 2026-03-07

## Scope

This shard records the Wendao LinkGraph test-architecture slice that migrates
`cache_build` from inline corpus setup to fixture-backed `input/expected`
contracts.

## Why This Change Was Needed

`packages/rust/crates/xiuxian-wendao/tests/test_link_graph/cache_build.rs`
was one of the last remaining LinkGraph modules still creating full corpora via
`TempDir` plus repeated `write_file(...)` calls.

That was especially undesirable here because the tests are not about file-tree
construction. They are about cache reuse, cache invalidation after mutation, and
saliency seeding. The corpus should be a fixture; the behavior under test should
be the cache boundary.

## What Changed

### 1) Added a cache-build fixture support module

New file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/cache_build_fixture_support.rs`

This module owns:

- cache-build scenario materialization,
- stats projection for cached index builds,
- search-hit projection for cache invalidation checks,
- saliency-state projection with stable floating-point rounding.

### 2) Migrated the `cache_build` suite to expected JSON contracts

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/cache_build.rs`

New scenarios:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/cache_build/reuses_snapshot/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/cache_build/detects_file_change/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/cache_build/seeds_saliency/...`

New expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/cache_build/reuses_snapshot/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/cache_build/detects_file_change/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/cache_build/seeds_saliency/expected/result.json`

What the contracts now cover:

- snapshot reuse across repeated cached builds,
- cache invalidation after file mutation,
- saliency-state seeding from frontmatter.

## Architectural Takeaways

- Cache behavior deserves fixture-backed contracts because the key question is
  state reuse and invalidation, not ad hoc corpus setup.
- Saliency seeding is best validated as a structured state contract instead of a
  single floating-point assertion in test code.
- After this slice, the remaining `write_file(...)` usage in `test_link_graph`
  is limited to true mutation/update flows rather than full inline corpus
  creation.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/cache_build.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/cache_build_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/cache_build/reuses_snapshot/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/cache_build/reuses_snapshot/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/cache_build/reuses_snapshot/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/cache_build/detects_file_change/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/cache_build/detects_file_change/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/cache_build/detects_file_change/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/cache_build/seeds_saliency/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/cache_build/seeds_saliency/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/cache_build/seeds_saliency/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_link_graph --message-format short
cargo nextest run -p xiuxian-wendao --test test_link_graph cache_build
cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --test test_link_graph --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph cache_build`
  passed (`3 passed, 81 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines`
  completed cleanly.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/439-xiuxian-wendao-tree-scope-fixture-expected-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/440-xiuxian-wendao-cache-build-fixture-expected-contracts-2026-03-07.md`
