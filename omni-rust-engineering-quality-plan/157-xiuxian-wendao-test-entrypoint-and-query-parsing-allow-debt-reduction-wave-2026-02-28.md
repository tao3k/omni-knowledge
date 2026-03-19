# 157. Xiuxian-Wendao Test Entrypoint and Query-Parsing Allow-Debt Reduction Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - integration test entrypoints (`tests/test_*.rs`)
  - core test module roots (`tests/*/mod.rs`)
  - `tests/test_link_graph/query_parsing/*.rs`

## Why This Wave

After wave `156`, `xiuxian-wendao/tests` still had high residual file-level
`#![allow(...)]` usage. The next high-yield slice was module-entry files and
small query-parsing tests, where suppressions could be removed with minimal
behavior risk.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from test entrypoint wrappers:
   - `packages/rust/crates/xiuxian-wendao/tests/test_graph.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_knowledge.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_agentic.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_ppr_benchmark.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_seed_and_priors.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_sync.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli.rs`

2. Removed file-level `#![allow(...)]` from module roots:
   - `packages/rust/crates/xiuxian-wendao/tests/test_graph/mod.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_knowledge/mod.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_sync/mod.rs`
   - `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/mod.rs`

3. Removed file-level `#![allow(...)]` from `query_parsing` lane:
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/mod.rs`
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
   - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/link_graph_parse_search_query_supports_tree_filter_directives.rs`

4. Fixed newly exposed `doc_markdown` warnings without suppression:
   - backticked `KnowledgeGraph`, `LinkGraph`, and `SyncEngine` in module docs
   - touched files:
     - `packages/rust/crates/xiuxian-wendao/tests/test_graph.rs`
     - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph.rs`
     - `packages/rust/crates/xiuxian-wendao/tests/test_graph/mod.rs`
     - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
     - `packages/rust/crates/xiuxian-wendao/tests/test_sync/mod.rs`

## Validation Evidence

1. Format:

```bash
cargo fmt -p xiuxian-wendao
```

- Result: pass

2. Strict clippy:

```bash
CARGO_TARGET_DIR=target/clippy-wendao cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass (exit code `0`)

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`

## Debt-Burndown Snapshot

- `rg -n '^#!\\[allow\\(' packages/rust/crates/xiuxian-wendao/tests -g '*.rs' | wc -l`
  - Before this wave: `143`
  - After this wave: `116`
  - Net reduction: `27` files

## Engineering Outcome

- Module-entry and query-parsing lanes now run suppression-free under strict
  clippy gates.
- The burndown shifted from broad wrapper debt to deeper behavioral tests,
  enabling next waves to focus on real algorithm/test-logic warnings.

## Next Slice

- Continue with `test_link_graph/search_filters/*` and
  `test_link_graph/tree_scope_filters/*` (small-file clusters with high
  suppression density and low migration risk).
