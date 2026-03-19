# 467. Xiuxian Wendao Page Index Line Range Contract Realignment

Date: 2026-03-08

## Scope

This shard records the follow-up fix for the four `page_index` snapshot failures
that remained after the structural test-boundary cleanup.

## Why This Change Was Needed

After the `test_link_graph` modular cleanup, the remaining failures were limited
to these page-index contract tests:

- `test_link_graph_page_index_builds_hierarchy_and_line_ranges`
- `test_link_graph_page_index_exports_semantic_documents`
- `test_link_graph_page_index_thins_small_parent_sections`
- `test_link_graph_page_index_refresh_updates_incremental_tree`

The failures were not structural. Tree shape, titles, semantic paths, token
counts, and document content all matched. Only `line_range` fields diverged.

## Root Cause

The runtime now reports 1-based inclusive physical line ranges that include the
blank lines physically contained inside a section span.

That behavior is consistent with the current parser and page-index builder:

- section parsing closes a section on the line immediately before the next
  heading, which naturally includes trailing blank lines inside the section;
- page-index nodes and exported semantic documents reuse those physical ranges;
- thinning merges child ranges by taking the physical min/max span.

The existing fixtures still reflected an older line-range contract that skipped
interior blank lines.

## What Changed

Updated fixture contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/hierarchy/expected/tree.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/thinning/expected/tree.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/semantic_documents/expected/documents.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/page_index/refresh/expected/incremental_refresh.json`

The update realigns fixture expectations with the current physical line-range
semantics.

## Contract Clarification

The effective page-index contract is now:

- `line_range` is 1-based and inclusive.
- `line_range` tracks the physical section span in the markdown source.
- Blank lines between a heading and its text, or between text and the next
  heading, remain part of that physical span.
- Thinned parent nodes inherit the min/max physical span of the folded subtree.

## Validation Evidence

Executed and passed:

```bash
cargo nextest run -p xiuxian-wendao --test test_link_graph test_link_graph_page_index_builds_hierarchy_and_line_ranges test_link_graph_page_index_thins_small_parent_sections test_link_graph_page_index_refresh_updates_incremental_tree test_link_graph_page_index_exports_semantic_documents --no-fail-fast
cargo nextest run -p xiuxian-wendao --test test_link_graph --no-fail-fast
cargo nextest run -p xiuxian-wendao --test test_wendao_cli --test test_link_graph --test test_link_graph_hybrid_benchmark --test test_link_graph_ppr_benchmark --no-tests pass --no-fail-fast
```

Observed outcomes:

- The four previously failing page-index tests passed (`4 passed, 80 skipped`).
- The full `test_link_graph` binary passed (`84 passed, 0 skipped`).
- The combined validation run for the touched binaries passed (`120 passed, 2 skipped`).
- The earlier `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` result from the same cleanup wave remains valid because this follow-up only changed fixture JSON, not Rust code.

## Architectural Takeaways

- Snapshot failures should be decomposed into structural failures versus
  contract-drift failures before changing runtime code.
- Physical source-location metadata must be documented precisely; otherwise,
  fixture suites accumulate silent assumptions.
- When a runtime contract is already internally consistent, updating fixtures is
  the correct fix; changing the runtime only to preserve stale snapshots would
  degrade correctness.

## Artifacts and Notes

- Follow-up to:
  - `assets/knowledge/omni-rust-engineering-quality-plan/466-xiuxian-wendao-link-graph-support-boundaries-and-page-index-validation-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/467-xiuxian-wendao-page-index-line-range-contract-realignment-2026-03-08.md`
