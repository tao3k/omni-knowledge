# 501. Xiuxian Wendao Query Parsing Snapshot Contract Migration

Date: 2026-03-08

## Scope

This shard records the full-wave migration of the `LinkGraph` query parsing suite in `xiuxian-wendao` from direct assertion tests to fixture-backed snapshot contracts.

The old suite lived under:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/`

It has now been replaced by:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_query_parsing_contracts.rs`

## Why This Change Was Needed

The repository was still carrying a dense cluster of parser-only direct assertion files. They were deterministic and tightly related, which made them a poor fit for continued fragmented maintenance.

This migration was chosen as a full thematic wave because:

- all cases exercised the same parser surface,
- expected behavior could be normalized into a compact JSON contract,
- the batch was large enough to justify one signed-off migration rather than file-by-file churn.

## What Changed

### 1) Added a dedicated parser contract binary

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_query_parsing_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_query_parsing_contract_support.rs`

The new binary captures the parser surface in six scenario snapshots:

- regex mode inference,
- query/id/limit directives,
- temporal and related-PPR directives,
- sort directives,
- negated link directives and boolean tags,
- tree filters and semantic policy directives.

Why this matters:

- parser behavior is now reviewed as stable JSON contracts,
- future parser refactors can update a few coherent fixtures instead of many inline assertions,
- downstream CLI and search tests can rely on a stable parser baseline.

### 2) Added expected snapshot fixtures for the full parser surface

Added fixture roots:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/query_parsing/regex_mode_inference/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/query_parsing/query_id_limit_directives/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/query_parsing/temporal_related_ppr_directives/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/query_parsing/sort_directives/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/query_parsing/negated_link_and_boolean_tags/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/query_parsing/tree_and_semantic_directives/expected/result.json`

Why this matters:

- the parser surface is now grouped by behavior rather than by source file,
- snapshots preserve the full merged `LinkGraphSearchOptions` state,
- regressions become more obvious because the resulting JSON shows the whole parsed payload.

### 3) Removed the superseded parser module tree

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

Why this matters:

- the migrated parser contracts no longer coexist with the obsolete direct-assertion files,
- the `test_link_graph` module graph stays clean,
- the migration follows the requested delete-after-replacement rule.

## Architectural Takeaways

### Parser waves are best migrated by semantic surface

Grouping by parser capability is more maintainable than snapshotting one old test per file. The new contract layout expresses what the parser does, not how the old suite happened to be split.

### Full `options` snapshots are worth keeping

Serializing the merged `LinkGraphSearchOptions` object gives better coverage than checking only one or two fields. It also makes subtle changes in defaults and directive merging visible.

### Audit first, then migrate one full wave

The audit showed many remaining direct-assertion suites. Choosing one cohesive batch and finishing it completely is a better workflow than scattering partial migrations across unrelated areas.

## Files Changed

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_query_parsing_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_query_parsing_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/query_parsing/regex_mode_inference/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/query_parsing/query_id_limit_directives/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/query_parsing/temporal_related_ppr_directives/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/query_parsing/sort_directives/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/query_parsing/negated_link_and_boolean_tags/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/query_parsing/tree_and_semantic_directives/expected/result.json`
- `.cache/codex/execplans/wendao-test-snapshot-migration-wave-1.md`

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/link_graph_parse_search_query_does_not_infer_regex_from_plain_parentheses.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/link_graph_parse_search_query_infers_regex_from_regex_markers.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/link_graph_parse_search_query_keeps_fts_for_extension_only_query.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/link_graph_parse_search_query_supports_directives_and_time_filters.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/link_graph_parse_search_query_supports_id_directive.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/link_graph_parse_search_query_supports_limit_directive.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/link_graph_parse_search_query_supports_multi_sort_terms_in_directive.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/link_graph_parse_search_query_supports_negated_directives_and_pipe_values.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/link_graph_parse_search_query_supports_parenthesized_boolean_tags.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/link_graph_parse_search_query_supports_query_directive.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/link_graph_parse_search_query_supports_related_ppr_key_variants.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/link_graph_parse_search_query_supports_semantic_policy_directives.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/link_graph_parse_search_query_supports_tree_filter_directives.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/support.rs`

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-query-parsing-contracts cargo check -p xiuxian-wendao --test test_link_graph_query_parsing_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-query-parsing-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph_query_parsing_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-query-parsing-contracts cargo clippy -p xiuxian-wendao --test test_link_graph_query_parsing_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --test test_link_graph_query_parsing_contracts --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph_query_parsing_contracts` passed (`6 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_link_graph_query_parsing_contracts -- -W clippy::too_many_lines` completed cleanly.

## Artifacts and Notes

- Contract binary: `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_query_parsing_contracts.rs`
- Shared helper: `packages/rust/crates/xiuxian-wendao/tests/support/link_graph_query_parsing_contract_support.rs`
- Migration wave plan: `.cache/codex/execplans/wendao-test-snapshot-migration-wave-1.md`
- Next cohesive target after this wave: `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/tree_scope_filters/`
