# 500. Xiuxian Wendao Markdown Syntax Snapshot Contract Migration

Date: 2026-03-08

## Scope

This shard records the migration of the markdown syntax integration suite in `xiuxian-wendao` from direct assertion tests to fixture-backed snapshot contracts.

The old suite lived in:

- `packages/rust/crates/xiuxian-wendao/tests/test_markdown_syntax_algorithm_fixtures.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_markdown_syntax_algorithm_fixtures/`

It has now been replaced by a single contract binary:

- `packages/rust/crates/xiuxian-wendao/tests/test_markdown_syntax_algorithm_contracts.rs`

## Why This Change Was Needed

The markdown syntax suite still used hard-coded direct assertions even though the repository is steadily standardizing on snapshot-style fixture contracts.

That old layout had three drawbacks:

- expected behavior was scattered across imperative assertions,
- the shared fixture corpus had no stable JSON contract surface,
- the migrated test policy was incomplete because the superseded suite still existed.

## What Changed

### 1) Added a dedicated markdown syntax contract binary

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_markdown_syntax_algorithm_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/markdown_syntax_algorithm_contract_support.rs`

The new binary covers four contract scenarios:

- corpus graph shape,
- search behavior over frontmatter and heading markers,
- graph traversal surface for neighbors and related results,
- edge-filter behavior for fenced pseudo-links and attachment/embed handling.

Why this matters:

- behavior is now expressed as stable JSON output,
- the suite is easier to diff during future parser or graph changes,
- the tests align with Wendao's broader contract-first test structure.

### 2) Added dedicated expected snapshot fixtures

Added fixture roots:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/markdown_syntax_algorithm_contracts/corpus_shape/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/markdown_syntax_algorithm_contracts/search_surface/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/markdown_syntax_algorithm_contracts/traversal_surface/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/markdown_syntax_algorithm_contracts/edge_filters/expected/result.json`

The input corpus remains the shared markdown tree under:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/markdown-syntax-algorithm/`

Why this matters:

- the input corpus stays canonical and reusable,
- expected outputs are isolated from the indexed markdown files,
- snapshot diffs remain easy to inspect.

### 3) Removed the superseded direct-assertion suite

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_markdown_syntax_algorithm_fixtures.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_markdown_syntax_algorithm_fixtures/`

Why this matters:

- there is no longer duplicate maintenance between old assertions and new contracts,
- the user-requested delete-after-migration rule is satisfied,
- the repository gets one step closer to a uniform snapshot testing surface.

## Architectural Takeaways

### Shared corpora can still use contract fixtures cleanly

The markdown corpus did not need to be duplicated into per-scenario `input/` folders. A contract helper can materialize the shared corpus into a temp directory while expected JSON lives in a separate namespace.

### Search snapshots should capture the real ranking surface

The new search contract intentionally records current ranking details such as `best_section`, `match_reason`, and the graph-rank contribution observed under `PathFuzzy`. That makes future ranking changes explicit.

### Delete the old suite at the same time as migration

Removing the direct-assertion suite immediately after the contract binary is added avoids drift and keeps ownership clear.

## Files Changed

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_markdown_syntax_algorithm_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/markdown_syntax_algorithm_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/markdown_syntax_algorithm_contracts/corpus_shape/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/markdown_syntax_algorithm_contracts/search_surface/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/markdown_syntax_algorithm_contracts/traversal_surface/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/markdown_syntax_algorithm_contracts/edge_filters/expected/result.json`
- `.cache/codex/execplans/wendao-markdown-syntax-snapshot-migration.md`

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_markdown_syntax_algorithm_fixtures.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_markdown_syntax_algorithm_fixtures/corpus.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_markdown_syntax_algorithm_fixtures/edge_filters.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_markdown_syntax_algorithm_fixtures/search.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_markdown_syntax_algorithm_fixtures/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_markdown_syntax_algorithm_fixtures/traversal.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_markdown_syntax_algorithm_fixtures/mod.rs`

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-markdown-contracts cargo check -p xiuxian-wendao --test test_markdown_syntax_algorithm_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-markdown-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_markdown_syntax_algorithm_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-markdown-contracts cargo clippy -p xiuxian-wendao --test test_markdown_syntax_algorithm_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --test test_markdown_syntax_algorithm_contracts --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_markdown_syntax_algorithm_contracts` passed (`4 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_markdown_syntax_algorithm_contracts -- -W clippy::too_many_lines` completed cleanly.

## Artifacts and Notes

- Contract binary: `packages/rust/crates/xiuxian-wendao/tests/test_markdown_syntax_algorithm_contracts.rs`
- Shared helper: `packages/rust/crates/xiuxian-wendao/tests/support/markdown_syntax_algorithm_contract_support.rs`
- Shared markdown corpus: `packages/rust/crates/xiuxian-wendao/tests/fixtures/markdown-syntax-algorithm/`
- Expected snapshot root: `packages/rust/crates/xiuxian-wendao/tests/fixtures/markdown_syntax_algorithm_contracts/`
