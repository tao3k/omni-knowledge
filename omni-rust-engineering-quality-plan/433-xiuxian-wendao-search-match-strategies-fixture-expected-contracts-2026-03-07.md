# 433. Xiuxian Wendao Search-Match-Strategies Fixture-Expected Contracts

Date: 2026-03-07

## Scope

This shard records the next Wendao LinkGraph test-architecture slice: migrating
`search_match_strategies` from inline tempdir setup and direct field assertions
to scenario fixtures plus expected JSON contracts.

This slice follows the same repository direction as the earlier Wendao test
migrations:

- scenario inputs live under `tests/fixtures/.../input/`,
- expected contracts live under `tests/fixtures/.../expected/`,
- new support logic stays in a focused domain module instead of a generic helper
  bucket.

It does not introduce a separate `tests/snapshots/` tree.

## Why This Change Was Needed

`packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_match_strategies.rs`
was still using the older pattern:

- each test created its own `TempDir`,
- each corpus was written inline with repeated `write_file(...)` calls,
- the assertions only protected a few top-level fields, while the scenario
  corpus and expected behavior were split across code rather than stored as one
  readable artifact.

That structure made the suite harder to extend and harder to diff when the
match-strategy behavior changed.

## What Changed

### 1) Added a dedicated support module for search-match fixtures

New file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_match_fixture_support.rs`

This module owns:

- scenario materialization for `search_match_strategies` fixture trees,
- query-parse JSON projection,
- hit-outline JSON projection for match-strategy scenarios,
- match-strategy label normalization,
- expected-fixture assertion dispatch.

Why this matters:

- `search_match_strategies.rs` now reads as pure behavior coverage,
- support logic stays domain-named and local,
- later strategy additions can extend one explicit projection surface.

### 2) Migrated the full `search_match_strategies` suite to per-scenario fixtures

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_match_strategies.rs`

New scenario fixture trees:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/parse_path_fuzzy/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/path_fuzzy_prefers_path_and_section/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/path_fuzzy_ignores_fenced_headings/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/path_fuzzy_handles_duplicate_headings/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/exact_strategy/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/regex_strategy/...`

New expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/parse_path_fuzzy/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/path_fuzzy_prefers_path_and_section/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/path_fuzzy_ignores_fenced_headings/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/path_fuzzy_handles_duplicate_headings/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/exact_strategy/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/regex_strategy/expected/result.json`

What the contracts now cover:

- `match:path_fuzzy` query parsing,
- path-fuzzy preference for structural path plus heading matches,
- fenced-heading exclusion,
- duplicate-heading selection behavior,
- exact-match retrieval,
- regex-based retrieval.

Why this matters:

- match-strategy scenarios now live as readable corpus-plus-contract artifacts,
- the suite no longer duplicates inline Markdown setup,
- future ranking changes can be reviewed as fixture diffs rather than scattered
  assertion edits.

### 3) Extended the LinkGraph test root cleanly

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

What changed:

- added the new domain support import surface,
- registered the dedicated support module alongside the existing fixture-based
  slices.

Why this matters:

- the root test namespace stays explicit,
- the new support follows the same modular pattern already established by
  `build_scope`, `page_index`, `search_core`, and `semantic_ignition`.

## Architectural Takeaways

### Match-strategy behavior deserves contract files

`PathFuzzy`, `Exact`, and `Regex` are not implementation trivia. They define
user-visible query semantics. Scenario fixtures make those semantics easier to
inspect and safer to evolve.

### Projection should stay minimal but semantic

This slice intentionally snapshots stable search behavior fields such as path,
stem, best section, and match reason, while avoiding noisy score coupling.
That keeps the contracts meaningful without making them fragile.

### Domain support modules scale better than shared helper sprawl

`search_match_fixture_support.rs` is preferable to growing another generic
support bucket because the ownership boundary is obvious from the name alone.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_match_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_match_strategies.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/parse_path_fuzzy/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/path_fuzzy_prefers_path_and_section/input/docs/architecture/graph.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/path_fuzzy_prefers_path_and_section/input/docs/notes/misc.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/path_fuzzy_prefers_path_and_section/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/path_fuzzy_ignores_fenced_headings/input/docs/architecture/engine.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/path_fuzzy_ignores_fenced_headings/input/docs/notes/misc.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/path_fuzzy_ignores_fenced_headings/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/path_fuzzy_handles_duplicate_headings/input/docs/architecture/api.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/path_fuzzy_handles_duplicate_headings/input/docs/notes/other.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/path_fuzzy_handles_duplicate_headings/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/exact_strategy/input/docs/a.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/exact_strategy/input/docs/b.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/exact_strategy/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/regex_strategy/input/docs/alpha-note.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/regex_strategy/input/docs/beta-note.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/search_match_strategies/regex_strategy/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
cargo check -p xiuxian-wendao --tests --message-format short
cargo nextest run -p xiuxian-wendao --test test_link_graph search_match_strategies
cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph search_match_strategies`
  passed (`6 passed, 78 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph -- -W clippy::too_many_lines`
  completed cleanly.

## Limits and Next Slice

The next high-value fixture-first migration candidates inside `test_link_graph`
are now the suites with the largest remaining concentrations of inline corpus
construction and multi-assert behavior checks, especially:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/graph_navigation.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/markdown_attachments.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/refresh.rs`

`graph_navigation.rs` is probably the best next target because it mixes corpus
setup, traversal outputs, metadata checks, TOC checks, and diagnostics in one
place.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/432-xiuxian-wendao-search-core-fixture-expected-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/433-xiuxian-wendao-search-match-strategies-fixture-expected-contracts-2026-03-07.md`
