# Xiuxian-Wendao Test Marker Reduction Wave (2026-02-26)

## Scope

Continue test-lane quality convergence in `xiuxian-wendao` by removing
unnecessary file-level `clippy::expect_used|clippy::unwrap_used` markers and
fixing low-cost residual panic extraction paths.

## Implemented Changes

1. Marker-only cleanup pass:
   - removed `clippy::expect_used` / `clippy::unwrap_used` from wendao test
     files that had no real `expect/unwrap` calls.
2. Follow-up fixes for newly unsuppressed failures:
   - `tests/test_link_graph/search_filters/link_graph_search_options_validate_rejects_invalid_related_ppr_alpha.rs`
     - replaced `expect_err(...)` with `let Err(err) = ... else`.
   - `tests/test_link_graph/query_parsing/link_graph_parse_search_query_supports_parenthesized_boolean_tags.rs`
     - replaced Option `expect(...)` with explicit `let Some(...) = ... else`.
   - `tests/test_link_graph/tree_scope_filters/link_graph_search_options_deserialize_accepts_tree_filters.rs`
     - converted to `Result`-returning test and used `serde_json::from_value(...) ?`.
   - `tests/test_link_graph_seed_and_priors/link_graph_related_journal_semantic_pull_surfaces_agenda_tasks.rs`
     - replaced `expect(...)` on `.find(...)` with `ok_or_else(...) ?`.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-wendao
cargo clippy -p xiuxian-wendao --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::pedantic
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
rg -l "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-wendao/tests | wc -l
```

Result:

- `xiuxian-wendao` tests and all targets pass pedantic checks.
- `xiuxian-wendao` passes `too_many_lines` policy verification.
- marker-file count in `xiuxian-wendao/tests` dropped from `161` to `38`.

## Outcome

The `xiuxian-wendao` test lane moved from broad suppression-heavy baseline to a
much smaller residual queue, preserving strict clippy quality gates.
