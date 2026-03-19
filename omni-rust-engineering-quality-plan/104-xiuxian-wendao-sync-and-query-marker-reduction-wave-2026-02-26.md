# Xiuxian-Wendao Sync and Query Marker Reduction Wave (2026-02-26)

## Scope

Continue marker-debt convergence in `xiuxian-wendao/tests` by targeting
low-occurrence files in `sync` and `link_graph` query lanes.

## Implemented Changes

1. Removed file-level `clippy::expect_used|clippy::unwrap_used` and replaced
   panic extraction in:
   - `tests/test_sync/batch_diff_computation.rs`
   - `tests/test_sync/deleted_files_detection.rs`
   - `tests/test_sync/manifest_load_save.rs`
   - `tests/test_link_graph/query_parsing/link_graph_parse_search_query_supports_related_ppr_key_variants.rs`
   - `tests/test_link_graph/tree_scope_filters/link_graph_search_options_validate_rejects_invalid_tree_filters.rs`
2. Refactor patterns used:
   - test functions converted to `Result`-returning style where needed.
   - `unwrap/expect/expect_err` replaced with `?`, `let Some(...) = ... else`,
     and `let Err(...) = ... else`.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-wendao
cargo clippy -p xiuxian-wendao --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
rg -l "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-wendao/tests | wc -l
```

Result:

- `xiuxian-wendao` test targets pass pedantic checks.
- `xiuxian-wendao` passes `too_many_lines` policy verification.
- marker-file count in `xiuxian-wendao/tests` dropped from `38` to `33`.

## Outcome

Residual marker queue in `xiuxian-wendao/tests` is reduced again, with stable
clippy quality-gate conformance.
