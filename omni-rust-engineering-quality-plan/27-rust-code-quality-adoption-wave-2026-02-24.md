# Rust Code-Quality Adoption Wave (2026-02-24)

## Objective

Apply Codex-derived Rust code-quality practices to Omni source code with a
code-first focus:

- stronger error boundaries,
- clearer module ownership,
- explicit cancellation/observability semantics,
- suppression-debt reduction.

This wave intentionally excludes CI workflow work.

## Current Baseline (Code-Level)

### Workspace-level lint baseline

- Workspace denies `unwrap_used` / `expect_used`, but currently lacks crate-level
  `print_stdout`/`print_stderr` deny policy:
  - `Cargo.toml:120`
  - `Cargo.toml:122`
  - `Cargo.toml:123`
  - matches in source: `0` crate-level `#![deny(clippy::print_stdout...)]` declarations.

### Error model shape

- `anyhow::Result`-related usage is high (well over 200 match hits),
  with `xiuxian-daochang` as the dominant hotspot (`207` grouped hits).
- `thiserror::Error` usage is limited (`12` matches) and not dominant in
  `xiuxian-daochang` boundaries.

### Observability shape

- `tracing::instrument` usage currently `0` across
  `packages/rust/crates` + `packages/rust/bindings/python`.

### Suppression profile (selected)

- `allow(clippy::too_many_lines)`: `42` occurrences
  - `xiuxian-daochang: 23`, `xiuxian-wendao: 8`, `xiuxian-vector: 6`.
- `allow(clippy::too_many_arguments)`: `34` occurrences
  - `xiuxian-daochang: 19`, `xiuxian-wendao: 10`.
- `allow(clippy::wildcard_imports)`: `8` occurrences (all in `xiuxian-daochang`).

### Large-file concentration

Top files over 1000 lines:

1. `packages/rust/crates/xiuxian-daochang/src/mcp_pool.rs` (`~1504`)
2. `packages/rust/crates/xiuxian-vector/src/search/search_impl.rs` (`~1479`)
3. `packages/rust/crates/xiuxian-vector/src/skill/ops_impl.rs` (`~1329`)
4. `packages/rust/crates/xiuxian-skills/src/skills/tools.rs` (`~1251`)
5. `packages/rust/crates/xiuxian-skills/src/skills/metadata.rs` (`~1148`)
6. `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs` (`~1135`)

## Codex Pattern -> Omni Adoption Mapping

## A) Error Taxonomy First (Replace Generic Boundary Errors)

Codex reference:

- typed domain errors in
  `.cache/researcher/openai/codex/codex-rs/core/src/error.rs:63`
  and
  `.cache/researcher/openai/codex/codex-rs/core/src/unified_exec/errors.rs:4`.

Omni wave target:

1. Introduce `xiuxian-daochang` domain error modules for high-churn subsystems:
   - MCP pool
   - embedding client
   - session backends
2. Keep `anyhow` for top-level glue and tests only; use typed errors for reusable
   public/internal subsystem boundaries.

Acceptance:

- New typed error enums exist for MCP/session paths.
- `xiuxian-daochang` public subsystem APIs stop returning naked `anyhow::Result`.

## B) Orchestration/Runtime Split for Monoliths

Codex reference:

- orchestrator boundary:
  `.cache/researcher/openai/codex/codex-rs/core/src/tools/orchestrator.rs:1`
- runtime decomposition:
  `.cache/researcher/openai/codex/codex-rs/core/src/unified_exec/mod.rs:20`.

Omni wave target:

1. Split `packages/rust/crates/xiuxian-daochang/src/mcp_pool.rs` into focused
   directory modules:
   - `mcp_pool/connect.rs`
   - `mcp_pool/call.rs`
   - `mcp_pool/cache.rs`
   - `mcp_pool/health.rs`
   - `mcp_pool/errors.rs`
   - `mcp_pool/mod.rs` (interface-only)
2. Keep policy/orchestration logic separate from per-client runtime calls.

Acceptance:

- `mcp_pool.rs` no longer contains all concerns in one file.
- `mod.rs` re-exports stable API surface.

## C) Cancellation + Instrumented Async Boundaries

Codex reference:

- explicit cancellation + span boundaries:
  `.cache/researcher/openai/codex/codex-rs/core/src/tools/parallel.rs:49`.

Omni wave target:

1. Add `tracing` span instrumentation for key async boundaries:
   - MCP connect/reconnect
   - tools list cache refresh
   - call_tool timeout/retry path
2. Introduce explicit cancellation plumbing in long-lived async loops where absent.

Acceptance:

- `tracing::instrument` exists in major latency/IO paths.
- Structured events include identifiers (server/tool/session/request id).

## D) Suppression-Debt Governance

Codex reference:

- strict workspace lint + localized exceptions:
  `.cache/researcher/openai/codex/codex-rs/Cargo.toml:302`.

Omni wave target:

1. Convert broad `#[allow(...)]` in hotspot modules to:
   - local narrow scope,
   - rationale comments,
   - tracked removal conditions.
2. Prioritize these files:
   - `packages/rust/crates/xiuxian-daochang/src/mcp_pool.rs`
   - `packages/rust/crates/xiuxian-vector/src/search/search_impl.rs`
   - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs`

Acceptance:

- Count of broad file-level allows declines in prioritized files.
- Exceptions are local and explained.

## E) Module Contract Documentation

Codex reference:

- module-level responsibility/flow docs:
  `.cache/researcher/openai/codex/codex-rs/core/src/unified_exec/mod.rs:1`.

Omni wave target:

1. Add module contract headers (responsibility + invariants + key flow) to
   complex runtime modules.
2. Start with:
   - `xiuxian-daochang/mcp_pool/*`
   - `xiuxian-vector/search/*`
   - `xiuxian-daochang/session/*`

Acceptance:

- Complex runtime modules start with concise contract docs.
- New contributors can identify ownership boundaries without reverse-engineering.

## Wave-1 Execution Order (Code Quality Only)

1. `xiuxian-daochang` error taxonomy seed (`mcp_pool` path).
2. `mcp_pool` structural split (`mod.rs` interface-only).
3. add tracing instrumentation on split boundaries.
4. targeted suppression cleanup in the same files.
5. repeat pattern for `xiuxian-vector/search`.

## Evidence Update Rule

For each completed slice, update this file with:

1. changed files,
2. before/after suppression counts for that slice,
3. compile/test evidence for touched crates.

## Hard Rule Update (2026-02-24)

`clippy::missing_errors_doc` must not be bypassed with `#[allow(...)]` in
production paths.

- Required practice: add explicit `# Errors` documentation for public `Result`
  APIs.
- Prohibited practice: suppressing missing error docs to make warnings pass.
- Enforcement scope in this wave: `xiuxian-vector/src/skill/ops_impl*` and all
  newly split module files.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-vector` Error Boundary Hardening)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/ops/writer_impl.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/migration.rs`
- `packages/rust/crates/xiuxian-vector/src/keyword/index.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_keyword_index.rs`

### Suppression debt delta (this slice)

- Removed two helper-level suppressions for panic-style dictionary creation:
  - `#[allow(clippy::expect_used, clippy::missing_panics_doc)]` in
    `writer_impl.rs` dictionary builders.
  - `#[allow(clippy::expect_used, clippy::missing_panics_doc)]` in
    `migration.rs` dictionary builder.
- Removed one method-level suppression:
  - `#[allow(clippy::unnecessary_wraps)]` in `KeywordIndex::count_documents`.
- Removed one impl-level suppression in migration path by adding explicit
  `# Errors` docs:
  - `#[allow(clippy::missing_errors_doc)]` from
    `packages/rust/crates/xiuxian-vector/src/ops/migration.rs`.

### Quality changes adopted

- Converted dictionary builders from implicit fallback behavior (`unwrap_or`)
  to explicit `Result`-based failure on:
  - dictionary key space overflow (`usize` -> `i32`),
  - unexpected missing dictionary key mapping.
- Propagated typed errors (`VectorStoreError`) to call sites with `?`.
- Simplified read-only API surface:
  - `KeywordIndex::count_documents` now returns `u64` directly.

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
- `cargo check -p xiuxian-vector`
- `cargo check -p xiuxian-vector` (re-run after migration doc update)
- `cargo test -p xiuxian-vector --no-run`
- `cargo test -p xiuxian-vector test_keyword_index -- --nocapture`
  - result: 8 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-vector` Missing Errors Doc Reduction)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/keyword/index.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/writer_impl.rs`

### Suppression debt delta (this slice)

- Removed impl-level suppression in keyword index:
  - `#[allow(clippy::missing_errors_doc, clippy::doc_markdown)]`
  - now: `#[allow(clippy::doc_markdown)]`.
- Removed impl-level suppression in writer path:
  - `#[allow(clippy::missing_errors_doc, clippy::doc_markdown)]`
  - now: `#[allow(clippy::doc_markdown)]`.
- Added explicit `# Errors` documentation to public `Result` APIs in both files.

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
- `cargo check -p xiuxian-vector`
- `cargo test -p xiuxian-vector --no-run`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-vector` Pedantic Warning Burn-Down)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/checkpoint/store/schema.rs`
- `packages/rust/crates/xiuxian-vector/src/checkpoint/store/timeline_ops.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/core.rs`
- `packages/rust/crates/xiuxian-vector/src/skill/ops_impl.rs`

### Quality changes adopted

- Replaced single-pattern `match` constructs with `if let` in dimension/step
  fallback paths while keeping explicit overflow logs.
- Collapsed nested `if` in source-filter scanner setup (`skill/ops_impl.rs`)
  to remove `collapsible_if` warning noise.
- Kept explicit fallback semantics (no silent truncation path introduced).

### Warning delta (this slice)

- `cargo clippy -p xiuxian-vector -- -W clippy::too_many_lines` warning count:
  - before: 5 warnings
  - after: 0 warnings

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
- `cargo check -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-vector --no-run`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-vector` List-Tools Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/skill/ops_impl.rs`

### Quality changes adopted

- Refactored `list_all_tools` into focused helpers:
  - `parse_source_filters`
  - `build_source_filter_expr`
  - `list_all_tools_projection`
  - `append_list_all_tools_rows_from_batch`
- Extracted metadata assembly/normalization into dedicated helpers to reduce
  cognitive load and preserve behavior:
  - `parse_list_all_tools_metadata`
  - `merge_non_empty_tool_columns`
  - `ensure_skill_and_tool_in_metadata`
- Preserved existing output contract (`{ id, content, metadata }`) and source
  filter semantics (`a||b`, metadata fallback when `file_path` is empty).

### Suppression debt delta (this slice)

- `skill/ops_impl.rs` `#[allow(clippy::too_many_lines)]` count:
  - before: 3
  - after: 2
- Removed `#[allow(clippy::too_many_lines)]` from `list_all_tools`.

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
- `cargo check -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-vector test_list_all_tools -- --nocapture`
  - result: 7 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-vector` Search Decode Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/search/search_impl.rs`

### Quality changes adopted

- Extracted vector search row decoding into focused helpers:
  - `extract_vector_row_columns`
  - `build_search_result_row`
  - `resolve_vector_row_metadata`
- Extracted Lance FTS row decoding into focused helpers:
  - `build_fts_result_row`
  - `parse_fts_metadata_row`
  - `parse_fts_score`
- Introduced shared Lance column access helpers with explicit typed error
  boundaries (`required_lance_string_column`, `required_lance_f32_column`).
- Kept public API and result contracts unchanged while reducing function size
  and improving internal ownership boundaries.

### Suppression debt delta (this slice)

- `search_impl.rs` `#[allow(clippy::too_many_lines)]` count:
  - before: 2
  - after: 0
- Verification:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-vector/src/search/search_impl.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
- `cargo check -p xiuxian-vector`
- `cargo test -p xiuxian-vector --no-run`
- `cargo clippy -p xiuxian-vector -- -W clippy::too_many_lines`
  - result: no `too_many_lines` warning in `search_impl.rs`; remaining warnings
    are in unrelated files (`checkpoint/store/schema.rs`,
    `checkpoint/store/timeline_ops.rs`, `ops/core.rs`, `skill/ops_impl.rs`).
- `cargo test -p xiuxian-vector test_lance_fts -- --nocapture`
  - result: 3 passed, 0 failed.
- `cargo test -p xiuxian-vector test_search_results_to_ipc -- --nocapture`
  - result: 4 passed, 0 failed.
- `cargo test -p xiuxian-vector test_tool_search_results_to_ipc -- --nocapture`
  - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-vector` Search IPC Function Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/search/search_impl.rs`

### Quality changes adopted

- Refactored `search_results_to_ipc` into focused helpers:
  - `collect_vector_ipc_data`
  - `resolve_vector_ipc_projection`
  - `append_vector_ipc_column`
  - `record_batch_to_ipc_bytes` (shared IPC writer path)
- Removed one `clippy::too_many_lines` suppression from vector IPC search encoder.
- Kept behavior and contract intact by preserving the same field names and projection checks.

### Suppression debt delta (this slice)

- `search_impl.rs` `#[allow(clippy::too_many_lines)]` count:
  - before: 4
  - after: 3

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
- `cargo check -p xiuxian-vector`
- `cargo test -p xiuxian-vector --no-run`
- `cargo test -p xiuxian-vector test_search_results_to_ipc -- --nocapture`
  - result: 4 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-vector` Tool IPC Function Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/search/search_impl.rs`

### Quality changes adopted

- Decomposed `tool_search_results_to_ipc` into focused helpers:
  - `empty_tool_search_ipc_batch`
  - `collect_tool_ipc_data`
  - `build_tool_search_ipc_batch`
- Reused shared IPC writer helper (`record_batch_to_ipc_bytes`) for both vector
  and tool search IPC output.
- Removed one additional `clippy::too_many_lines` suppression from search IPC path.

### Suppression debt delta (this slice)

- `search_impl.rs` `#[allow(clippy::too_many_lines)]` count:
  - before: 3
  - after: 2

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
- `cargo check -p xiuxian-vector`
- `cargo test -p xiuxian-vector --no-run`
- `cargo test -p xiuxian-vector test_tool_search_results_to_ipc -- --nocapture`
  - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-vector` TryFrom Fallback Elimination)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/batch.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/core.rs`
- `packages/rust/crates/xiuxian-vector/src/checkpoint/store/write_ops.rs`
- `packages/rust/crates/xiuxian-vector/src/checkpoint/store/schema.rs`
- `packages/rust/crates/xiuxian-vector/src/checkpoint/store/timeline_ops.rs`

### Quality changes adopted

- Removed all `i32::try_from(...).unwrap_or(...)` fallback patterns from
  `xiuxian-vector/src`.
- In `Result`-returning paths (`batch.rs`, `write_ops.rs`), changed to explicit
  error returns on dimension overflow.
- In schema-construction paths that are non-fallible API surfaces
  (`ops/core.rs`, `checkpoint/store/schema.rs`), replaced silent fallback with
  explicit branch handling and warning logs.
- Replaced timeline step conversion fallback with explicit `match` to keep
  saturating behavior intentional and visible.

### Validation evidence

- `rg "i32::try_from\\([^\\)]*\\)\\.unwrap_or" packages/rust/crates/xiuxian-vector/src -n`
  - result: no matches.
- `cargo fmt -p xiuxian-vector`
- `cargo check -p xiuxian-vector`
- `cargo test -p xiuxian-vector --no-run`
- `cargo test -p xiuxian-vector test_keyword_index -- --nocapture`
  - result: 8 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-vector` Dimension Boundary Hardening)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/ops/writer_impl.rs`

### Quality changes adopted

- Removed hidden fallback for vector list dimension conversion in document batch build:
  - before: `i32::try_from(self.dimension).unwrap_or(1536)`
  - now: explicit checked conversion with `VectorStoreError::General` on overflow.
- This prevents silent schema drift when a configured dimension exceeds `i32`.

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
- `cargo check -p xiuxian-vector`
- `cargo test -p xiuxian-vector --no-run`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-vector` Skill Ops Missing-Errors-Doc Compliance)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/indexing.rs`
- `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/listing.rs`
- `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/search.rs`
- `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/registry.rs`

### Quality changes adopted

- Removed `clippy::missing_errors_doc` suppressions from all split skill ops
  implementation files.
- Kept only `clippy::doc_markdown` local allowances where needed.
- Added explicit `# Errors` sections to all public `Result` APIs in these files:
  - `index_skill_tools`
  - `index_skill_tools_dual`
  - `scan_skill_tools_raw`
  - `list_all_resources`
  - `list_all_tools`
  - `search_tools`
  - `search_tools_with_options`
  - `load_tool_registry`
  - `get_tools_by_skill`

### Suppression debt delta (this slice)

- `ops_impl` split files `#[allow(clippy::missing_errors_doc, ...)]`:
  - before: 4
  - after: 0

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
- `cargo check -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector -- -W clippy::missing_errors_doc -W clippy::too_many_lines`
- `cargo test -p xiuxian-vector test_list_all_tools -- --nocapture`
  - result: 7 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-vector` Search Tools Pipeline Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/search.rs`

### Quality changes adopted

- Refactored `search_tools_with_options` into orchestration-only flow:
  - vector scan collection (`collect_vector_tool_results`)
  - keyword fusion (`fuse_tool_results_with_keyword`)
  - final threshold/sort/truncate (`finalize_tool_results`)
- Extracted batch decoding and row resolution helpers for clearer ownership:
  - `extract_search_batch_columns`
  - `build_vector_result_row`
  - `resolve_search_row_fields`
  - `resolve_search_row_from_columns`
  - `resolve_search_row_from_metadata`
- Preserved external API and ranking behavior while removing monolithic control
  flow from the public search entrypoint.

### Suppression debt delta (this slice)

- `search_tools_with_options` local suppression:
  - before: `#[allow(clippy::too_many_arguments, clippy::too_many_lines, ...)]`
  - after: `#[allow(clippy::too_many_arguments)]`
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-vector/src/skill/ops_impl/search.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
- `cargo check -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector -- -W clippy::too_many_lines -W clippy::missing_errors_doc`
- `cargo test -p xiuxian-vector --test test_hybrid_search -- --nocapture`
  - result: 9 passed, 0 failed.
- `cargo test -p xiuxian-vector --test test_skill_index_robustness -- --nocapture`
  - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-vector` Entity-Aware Boost Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/keyword/entity_aware.rs`

### Quality changes adopted

- Refactored `apply_entity_boost` into focused helpers:
  - automaton build (`build_entity_name_automaton`)
  - text/entity match collection (`collect_entity_matches_in_text`)
  - metadata pass (`collect_metadata_entity_matches`)
  - duplicate-safe insertion (`push_unique_entity_match`)
  - score computation (`calculate_entity_boost`)
- Kept output contract and sort behavior unchanged while removing monolithic
  matching logic from the top-level function.
- Adjusted helper signatures to avoid new pedantic debt
  (`needless_pass_by_value`) by passing match-type overrides by reference.

### Suppression debt delta (this slice)

- `entity_aware.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `apply_entity_boost`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-vector/src/keyword/entity_aware.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
- `cargo check -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-vector entity_aware -- --nocapture`
  - result: 5 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-vector` Writer Batch Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/ops/writer_impl.rs`

### Quality changes adopted

- Refactored `build_document_batch` into focused helpers:
  - input validation (`validate_document_batch_inputs`)
  - vector list construction (`build_vector_list_array`)
  - metadata column extraction (`parse_document_metadata_columns`)
- Reduced control-flow density in the batch path while preserving schema/layout
  and existing error behavior.
- Removed stale `too_many_lines` suppression from `get_or_create_dataset` after
  verification that current implementation no longer needs it.

### Suppression debt delta (this slice)

- `writer_impl.rs` `clippy::too_many_lines` suppressions:
  - before: 2 (`build_document_batch`, `get_or_create_dataset`)
  - after: 0
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-vector/src/ops/writer_impl.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
- `cargo check -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector -- -W clippy::too_many_lines -W clippy::missing_errors_doc`
- `cargo test -p xiuxian-vector --test test_merge_insert -- --nocapture`
  - result: 1 passed, 0 failed.
- `cargo test -p xiuxian-vector --test test_store -- --nocapture`
  - result: 6 passed, 0 failed.
- `cargo test -p xiuxian-vector --test test_data_layer_snapshots -- --nocapture`
  - result: 6 passed, 0 failed.
- `cargo test -p xiuxian-vector --test test_vector_index -- --nocapture`
  - result: 6 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-vector` Weighted RRF Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/keyword/fusion/weighted_rrf.rs`

### Quality changes adopted

- Refactored `apply_weighted_rrf` into orchestration-only flow with focused helpers:
  - effective weight policy (`compute_effective_fusion_weights`)
  - sparse-keyword debug logging (`maybe_log_sparse_keyword_fallback`)
  - vector seed merge (`seed_vector_fusion_map`)
  - keyword merge (`merge_keyword_fusion_scores`)
  - parallel boost delta computation (`compute_name_metadata_boost_deltas`)
  - delta apply + final sort (`apply_boost_deltas`, `sorted_fusion_results`)
- Preserved ranking/score contract while removing monolithic control flow.
- Kept existing API compatibility; retained only `needless_pass_by_value` local
  allow for public signature stability.

### Suppression debt delta (this slice)

- `weighted_rrf.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines, clippy::needless_pass_by_value)]`
  - after: `#[allow(clippy::needless_pass_by_value)]`
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-vector/src/keyword/fusion/weighted_rrf.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
- `cargo check -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector -- -W clippy::too_many_lines -W clippy::missing_errors_doc`
- `cargo test -p xiuxian-vector --test test_fusion -- --nocapture`
  - result: 16 passed, 0 failed.
- `cargo test -p xiuxian-vector --test test_fusion_snapshots -- --nocapture`
  - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-wendao` Query-Parse Merge Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-wendao/src/link_graph/query/parse/merge.rs`

### Quality changes adopted

- Refactored `merge_into_base` into focused merge stages:
  - match strategy (`merge_match_strategy`)
  - case + sort (`merge_case_and_sort`)
  - tag/link filters (`merge_tag_and_link_filters`)
  - related filters (`merge_related_filters`)
  - search filters (`merge_search_filters`)
  - temporal filters (`merge_time_filters`)
- Preserved option-merge semantics while reducing function cognitive load and
  removing monolithic flow.

### Suppression debt delta (this slice)

- `merge.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-wendao/src/link_graph/query/parse/merge.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-wendao`
- `cargo check -p xiuxian-wendao`
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-wendao query_parsing -- --nocapture`
  - result: 11 passed, 0 failed (query parsing cases in `test_link_graph`).

## Execution Evidence: 2026-02-24 (Slice `xiuxian-wendao` Query-Parse Link Directives Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-wendao/src/link_graph/query/parse/scan/directives/links.rs`

### Quality changes adopted

- Refactored directive parsing into focused handlers:
  - `apply_link_to_directive`
  - `apply_linked_by_directive`
  - `apply_related_directive`
- Extracted parsing helpers for repeated numeric constraints:
  - `parse_positive_usize`
  - `parse_alpha`
  - `parse_positive_f64`
- Kept directive aliases and merge semantics unchanged while reducing monolithic
  branching in the main `apply` function.

### Suppression debt delta (this slice)

- `links.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-wendao/src/link_graph/query/parse/scan/directives/links.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-wendao`
- `cargo check -p xiuxian-wendao`
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-wendao query_parsing -- --nocapture`
  - result: 11 passed, 0 failed (query parsing cases in `test_link_graph`).

## Execution Evidence: 2026-02-24 (Slice `xiuxian-wendao` Link-Graph PPR Kernel Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/ppr/kernel.rs`

### Quality changes adopted

- Refactored `run_related_ppr_kernel` into orchestration-only flow.
- Extracted graph/PPR pipeline helpers:
  - node index construction (`build_node_index`)
  - passage-entity edge extraction (`build_passage_entity_edges`)
  - adjacency construction (`build_ppr_adjacency`)
  - teleport + restart setup (`build_teleport_and_restart_nodes`)
  - iterative solver (`run_ppr_iterations`)
  - result materialization (`scores_by_graph_node`)
- Added small type aliases (`RestartNodes`, `TeleportSetup`) to avoid new
  `type_complexity` lint debt.

### Suppression debt delta (this slice)

- `kernel.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `run_related_ppr_kernel`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-wendao/src/link_graph/index/ppr/kernel.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-wendao`
- `cargo check -p xiuxian-wendao`
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-wendao --test test_link_graph_ppr_weighting -- --nocapture`
  - result: 1 passed, 0 failed.
- `cargo test -p xiuxian-wendao --test test_link_graph_seed_and_priors -- --nocapture`
  - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `omni-sandbox` SandboxConfig Constructor Arity Refactor)

### Changed files

- `packages/rust/crates/omni-sandbox/src/executor/mod.rs`

### Quality changes adopted

- Replaced high-arity `SandboxConfig::__new__` signature with a low-arity parser entrypoint:
  - `#[new] #[pyo3(signature = (*args, **kwargs))]`.
- Added explicit constructor parsing helpers:
  - `SandboxConfig::from_positional_args` for legacy positional call shape,
  - `SandboxConfig::from_keyword_args` for named Python call shape.
- Preserved runtime compatibility for existing Python usage:
  - supports 11 positional arguments,
  - supports keyword-based creation used by current tests.
- Added explicit argument-shape error handling:
  - rejects mixed positional + keyword payloads,
  - returns clear errors for missing required keyword fields.

### Suppression debt delta (this slice)

- Removed `#[allow(clippy::too_many_arguments)]` from:
  - `packages/rust/crates/omni-sandbox/src/executor/mod.rs` (`SandboxConfig::new`).
- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates/omni-sandbox -g '*.rs'`
  - result: no matches.
- Workspace snapshot after this slice:
  - `allow(clippy::too_many_arguments)` in Rust crates: `18` -> `17`.

### Compile and test evidence

- `cargo fmt -p omni-sandbox -p omni-core-rs`
  - result: passed.
- `cargo check -p omni-sandbox -p omni-core-rs`
  - result: passed.
- `cargo clippy -p omni-sandbox -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo clippy -p omni-core-rs -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p omni-sandbox -- --nocapture`
  - result: 17 passed, 0 failed.
- `cargo test -p omni-core-rs --no-run`
  - result: passed (test binaries built successfully).

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Schedule Mode Request Refactor)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/nodes/schedule.rs`
- `packages/rust/crates/xiuxian-daochang/src/nodes/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/main.rs`

### Quality changes adopted

- Replaced long positional argument list in schedule runtime entrypoint with
  a typed request struct:
  - added `ScheduleModeRequest`,
  - changed `run_schedule_mode` to accept `ScheduleModeRequest`.
- Updated module re-exports and the sole call site in CLI dispatch path.
- Preserved behavior:
  - scheduler config values and runtime wiring are unchanged,
  - only parameter plumbing and signature clarity were updated.

### Suppression debt delta (this slice)

- Removed `#[allow(clippy::too_many_arguments)]` from:
  - `packages/rust/crates/xiuxian-daochang/src/nodes/schedule.rs`.
- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates/xiuxian-daochang -g '*.rs'`
  - result: `17` -> `16`.
- Workspace snapshot after this slice:
  - `allow(clippy::too_many_arguments)` in Rust crates: `17` -> `16`.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-daochang --lib --bins --no-run`
  - result: passed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Telegram Webhook Builder Core/API Refactor)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/webhook/builders/core.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/webhook/builders/api.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/webhook/builders/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/webhook/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/run_webhook/run.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/lib.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels_webhook.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels_webhook_stress.rs`

### Quality changes adopted

- Added typed request objects across webhook builder layers:
  - `TelegramWebhookCoreBuildRequest`,
  - `TelegramWebhookControlPolicyBuildRequest`,
  - `TelegramWebhookPartitionBuildRequest`.
- Refactored webhook builder APIs to use request objects instead of long positional signatures.
- Kept convenience `build_telegram_webhook_app(...)` entrypoint intact while making
  high-arity intermediate helpers private and request-based.
- Updated runtime and test call sites to construct explicit request structs.
- Promoted new request types through runtime/channel/crate re-exports for stable top-level access.

### Suppression debt delta (this slice)

- Removed `#[allow(clippy::too_many_arguments)]` from:
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/webhook/builders/core.rs`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/webhook/builders/api.rs` (all remaining entries)
- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates/xiuxian-daochang -g '*.rs'`
  - result: `6` -> `0`.
- Workspace snapshot after this slice:
  - `allow(clippy::too_many_arguments)` in Rust crates: `6` -> `0`.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-daochang --lib --bins --no-run`
  - result: passed.
- `cargo test -p xiuxian-daochang --test channels_webhook -- --nocapture`
  - result: 7 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_webhook_stress webhook_concurrent_chat_user_partition_keeps_isolated_session_keys -- --nocapture`
  - result: 1 passed, 0 failed (6 filtered).

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Telegram Webhook Runtime Request Refactor)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/run_webhook/run.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/run_webhook/loop_control.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/run_webhook/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/lib.rs`
- `packages/rust/crates/xiuxian-daochang/src/nodes/channel/telegram.rs`
- `packages/rust/crates/xiuxian-daochang/tests/telegram_runtime/webhook_security.rs`

### Quality changes adopted

- Replaced high-arity webhook runtime APIs with typed request/context objects:
  - `TelegramWebhookRunRequest`,
  - `TelegramWebhookPolicyRunRequest`,
  - `WebhookEventLoopContext`.
- Updated node runtime entrypoint and webhook security test call sites to use
  request objects.
- Promoted new request types through runtime/channel/crate re-exports for stable
  API access from higher layers.
- Preserved runtime behavior:
  - secret token normalization and webhook startup flow unchanged,
  - foreground/background dispatch wiring unchanged,
  - event loop shutdown and telemetry snapshot semantics unchanged.

### Suppression debt delta (this slice)

- Removed `#[allow(clippy::too_many_arguments)]` from:
  - `run_telegram_webhook`,
  - `run_telegram_webhook_with_control_command_policy`,
  - `run_webhook_event_loop`.
- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates/xiuxian-daochang -g '*.rs'`
  - result: `9` -> `6`.
- Workspace snapshot after this slice:
  - `allow(clippy::too_many_arguments)` in Rust crates: `9` -> `6`.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-daochang --lib --bins --no-run`
  - result: passed.
- `cargo test -p xiuxian-daochang --test channels_webhook webhook_security -- --nocapture`
  - result: command passed; 0 tests matched filter in this target (7 filtered).

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Discord Channel Core Init Refactor)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/channels/discord/channel/constructors.rs`

### Quality changes adopted

- Replaced high-arity Discord channel core constructor with typed init payload:
  - added `DiscordChannelCoreInit`,
  - changed internal core constructor to consume this struct.
- Updated all internal call sites (default/base-url/client/parsed-policy paths)
  to construct and pass explicit init values.
- Preserved behavior:
  - control/slash policy normalization unchanged,
  - allowed user/guild normalization unchanged,
  - client/session-partition wiring unchanged.

### Suppression debt delta (this slice)

- Removed `#[allow(clippy::too_many_arguments)]` from:
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/channel/constructors.rs`.
- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates/xiuxian-daochang -g '*.rs'`
  - result: `10` -> `9`.
- Workspace snapshot after this slice:
  - `allow(clippy::too_many_arguments)` in Rust crates: `10` -> `9`.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-daochang --lib --bins --no-run`
  - result: passed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Discord Ingress Build Request Refactor)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/ingress.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/lib.rs`
- `packages/rust/crates/xiuxian-daochang/tests/channels_discord_ingress.rs`

### Quality changes adopted

- Replaced high-arity Discord ingress builder signature with typed request object:
  - added `DiscordIngressBuildRequest`,
  - changed `build_discord_ingress_app_with_partition_and_control_command_policy` to accept it.
- Updated runtime wiring and integration tests to pass explicit request values.
- Promoted the new request type through runtime/channel/crate re-exports so API
  remains consumable from crate boundaries.
- Preserved behavior:
  - ingress secret validation unchanged,
  - message parse/enqueue flow unchanged,
  - session partition wiring unchanged.

### Suppression debt delta (this slice)

- Removed `#[allow(clippy::too_many_arguments)]` from:
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/ingress.rs`.
- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates/xiuxian-daochang -g '*.rs'`
  - result: `11` -> `10`.
- Workspace snapshot after this slice:
  - `allow(clippy::too_many_arguments)` in Rust crates: `11` -> `10`.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-daochang --lib --bins --no-run`
  - result: passed.
- `cargo test -p xiuxian-daochang --test channels_discord_ingress -- --nocapture`
  - result: 5 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Telegram Media Upload Payload Refactor)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/send_api/media.rs`

### Quality changes adopted

- Replaced duplicated high-arity media-upload helper signatures with a typed payload:
  - added `MediaFileUpload<'a>`,
  - changed retry and single-send helpers to consume `&MediaFileUpload`.
- Updated call flow to construct explicit upload payloads for:
  - first attempt with Markdown parse mode (when present),
  - retry path without parse mode fallback.
- Preserved send behavior:
  - retry gate logic unchanged,
  - transient retry policy unchanged,
  - multipart payload shape unchanged.

### Suppression debt delta (this slice)

- Removed `#[allow(clippy::too_many_arguments)]` from:
  - `send_media_file_with_retry_mode`,
  - `send_media_file_once`.
- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates/xiuxian-daochang -g '*.rs'`
  - result: `13` -> `11`.
- Workspace snapshot after this slice:
  - `allow(clippy::too_many_arguments)` in Rust crates: `13` -> `11`.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-daochang --lib --bins --no-run`
  - result: passed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Telegram Constructor Core Init Refactor)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/constructor/core.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/constructor/base_url.rs`

### Quality changes adopted

- Replaced high-arity Telegram channel core constructor call with typed init payload:
  - added `TelegramChannelCoreInit`,
  - changed `new_with_base_url_and_partition_and_client_impl` to take the init struct.
- Updated constructor call sites in `base_url.rs` to pass explicit field-mapped init values.
- Preserved behavior:
  - ACL normalization and runtime settings path wiring unchanged,
  - send-rate-limit and client initialization flow unchanged.

### Suppression debt delta (this slice)

- Removed `#[allow(clippy::too_many_arguments)]` from:
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/constructor/core.rs`.
- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates/xiuxian-daochang -g '*.rs'`
  - result: `14` -> `13`.
- Workspace snapshot after this slice:
  - `allow(clippy::too_many_arguments)` in Rust crates: `14` -> `13`.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-daochang --lib --bins --no-run`
  - result: passed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Managed Foreground Turn Request Refactor)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/channels/managed_runtime/turn.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/dispatch/turn.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch/turn.rs`

### Quality changes adopted

- Replaced high-arity managed runtime call signature with a typed payload:
  - added `ForegroundTurnRequest`,
  - changed `run_foreground_turn_with_interrupt` to accept the request struct.
- Updated Telegram and Discord dispatch paths to build and pass request objects.
- Preserved runtime behavior:
  - identical timeout and interrupt semantics,
  - unchanged foreground outcome mapping and reply rendering.

### Suppression debt delta (this slice)

- Removed `#[allow(clippy::too_many_arguments)]` from:
  - `packages/rust/crates/xiuxian-daochang/src/channels/managed_runtime/turn.rs`.
- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates/xiuxian-daochang -g '*.rs'`
  - result: `15` -> `14`.
- Workspace snapshot after this slice:
  - `allow(clippy::too_many_arguments)` in Rust crates: `15` -> `14`.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-daochang --lib --bins --no-run`
  - result: passed.
- `cargo test -p xiuxian-daochang --no-run`
  - result: failed due pre-existing unrelated integration-test import error in
    `tests/agent_graph_bridge.rs` (`GraphBridgeRequest`/`validate_graph_bridge_request`
    unresolved in crate root); not introduced by this slice.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Reflection Turn Report Refactor)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/agent/reflection_runtime_state.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/conversation.rs`

### Quality changes adopted

- Replaced high-arity reflection API with a typed payload:
  - added `ReflectionTurnReport`,
  - changed `reflect_turn_and_update_policy_hint` to take the report struct.
- Updated both conversation finalization call sites (`success`/`error`) to pass
  structured reflection data.
- Preserved reflection behavior:
  - lifecycle transition checks unchanged,
  - policy-hint derivation and storage unchanged,
  - observability fields unchanged.

### Suppression debt delta (this slice)

- Removed `#[allow(clippy::too_many_arguments)]` from:
  - `packages/rust/crates/xiuxian-daochang/src/agent/reflection_runtime_state.rs`.
- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates/xiuxian-daochang -g '*.rs'`
  - result: `16` -> `15`.
- Workspace snapshot after this slice:
  - `allow(clippy::too_many_arguments)` in Rust crates: `16` -> `15`.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-daochang --lib --bins --no-run`
  - result: passed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-vector` Tool Search Request-Object Refactor)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/skill/mod.rs`
- `packages/rust/crates/xiuxian-vector/src/lib.rs`
- `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/search.rs`
- `packages/rust/crates/xiuxian-vector/src/search/search_impl/mod.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/agentic.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_rust_cortex.rs`
- `packages/rust/bindings/python/src/vector/search_ops.rs`

### Quality changes adopted

- Added `ToolSearchRequest<'a>` as a typed request envelope for tool-search APIs.
- Refactored high-arity entry points to consume the request object:
  - `VectorStore::search_tools_with_options(request)`
  - `VectorStore::search_tools_ipc(request)`
- Updated internal/vector-agentic/python-binding/test call sites to pass explicit
  request structs instead of long positional argument lists.
- Kept runtime behavior intact:
  - same tool-scoring flow,
  - same optional keyword fusion,
  - same threshold and limit semantics.

### Suppression debt delta (this slice)

- Removed `#[allow(clippy::too_many_arguments)]` from:
  - `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/search.rs`
  - `packages/rust/crates/xiuxian-vector/src/search/search_impl/mod.rs`
- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates/xiuxian-vector -g '*.rs'`
  - result: no matches.
- Workspace snapshot after this slice:
  - `allow(clippy::too_many_arguments)` in Rust crates: `21` -> `18`.
  - Directly attributable to this slice in `xiuxian-vector`: `2` -> `0`.

### Compile and test evidence

- `cargo fmt -p xiuxian-vector -p omni-core-rs`
  - result: passed.
- `cargo check -p xiuxian-vector -p omni-core-rs`
  - result: passed.
- `cargo clippy -p xiuxian-vector -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo clippy -p omni-core-rs -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-vector --test test_rust_cortex -- --nocapture`
  - result: 19 passed, 0 failed.
- `cargo test -p omni-core-rs --no-run`
  - result: passed (test binaries built successfully).

## Execution Evidence: 2026-02-25 (Slice `xiuxian-wendao` Agentic Run and Retrieval Plan Constructor Refactor)

### Changed files

- `packages/rust/crates/xiuxian-wendao/src/bin/wendao/execute/agentic/plan_run.rs`
- `packages/rust/crates/xiuxian-wendao/src/bin/wendao/execute/agentic/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/retrieval_plan.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/plan/payload/policy.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/mod.rs`

### Quality changes adopted

- Replaced high-arity CLI run handler parameters with structured options:
  - added `AgenticRunOptions`,
  - changed `handle_run` to accept `&AgenticRunOptions`.
- Replaced high-arity retrieval-plan constructor parameters with a typed input payload:
  - added `LinkGraphRetrievalPlanInput`,
  - changed `LinkGraphRetrievalPlanRecord::new` to accept this input struct.
- Updated all call sites and module re-exports to keep public namespace consistency.
- Added missing field-level documentation for new public retrieval-plan input struct.
- Resolved follow-up pedantic quality issue by using `clone_from` for mutable config strings.

### Suppression debt delta (this slice)

- Removed function-level suppressions:
  - `packages/rust/crates/xiuxian-wendao/src/bin/wendao/execute/agentic/plan_run.rs`
  - `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/retrieval_plan.rs`
- `allow(clippy::too_many_arguments)` count:
  - Rust crates overall: `23` -> `21`
  - `xiuxian-wendao/src`: `2` -> `0`
- Validation:
  - `(rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates/xiuxian-wendao/src -g '*.rs' || true) | wc -l`
  - result: `0`
  - `(rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates -g '*.rs' || true) | wc -l`
  - result: `21`

### Compile and test evidence

- `cargo fmt -p xiuxian-wendao`
  - result: passed.
- `cargo check -p xiuxian-wendao`
  - result: passed.
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-wendao --no-run`
  - result: passed.
- `cargo test -p xiuxian-wendao --test test_link_graph search_core -- --nocapture`
  - result: 7 passed, 0 failed (48 filtered).
- `cargo test -p xiuxian-wendao --test test_wendao_cli -- --nocapture`
  - result: 34 passed, 0 failed.
- `cargo test -p xiuxian-wendao --test test_link_graph_ppr_benchmark -- --nocapture`
  - result: 0 passed, 1 ignored (heavy benchmark marked ignored).

## Execution Evidence: 2026-02-24 (Slice `xiuxian-wendao` HMAS Blackboard Validation Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-wendao/src/hmas/blackboard/validate.rs`

### Quality changes adopted

- Refactored `validate_blackboard_markdown` into dedicated validators:
  - `validate_task_block`
  - `validate_evidence_block`
  - `validate_conclusion_block`
  - `validate_digital_thread_block`
  - `report_missing_digital_thread_requirements`
- Kept report counters/issue codes and cross-block consistency checks intact
  while removing monolithic match logic from the entry function.

### Suppression debt delta (this slice)

- `validate.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-wendao/src/hmas/blackboard/validate.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-wendao`
- `cargo check -p xiuxian-wendao`
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-wendao --test test_hmas -- --nocapture`
  - result: 4 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-wendao` Agentic Worker Execute Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/agentic_expansion/execute/worker.rs`

### Quality changes adopted

- Refactored `execute_worker` into orchestration-first flow with focused helpers:
  - worker lifecycle initialization (`initialize_worker_run`)
  - budget gate checks (`is_global_budget_exhausted`, `is_worker_budget_exhausted`)
  - request construction (`build_suggested_link_request`)
  - retryable persistence (`persist_request_with_retries`)
  - bounded error accumulation (`push_bounded_errors`)
  - telemetry finalization (`finalize_worker_run`)
- Introduced `WorkerPersistOutcome` to make persistence effects explicit and reduce
  side-effect coupling in the main loop.
- Preserved idempotency semantics, retry accounting, and phase telemetry contract.

### Suppression debt delta (this slice)

- `worker.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `execute_worker`
  - after: removed
- Validation:
  - `rg -n "too_many_lines|missing_errors_doc" packages/rust/crates/xiuxian-wendao/src/link_graph/index/agentic_expansion/execute/worker.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-wendao`
- `cargo check -p xiuxian-wendao`
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-wendao --test test_link_graph_agentic_expansion -- --nocapture`
  - result: 3 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-wendao` Link-Graph Build Assemble Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/build/assemble.rs`

### Quality changes adopted

- Refactored `build_with_filters` into staged assembly helpers:
  - root validation/canonicalization (`canonicalize_root_dir`)
  - include/exclude filter normalization (`normalize_directory_filters`)
  - candidate path collection (`collect_candidate_note_paths`)
  - parallel parse stage (`parse_candidate_notes`)
  - document/section/alias maps (`build_note_maps`)
  - edge graph construction (`build_graph_edges`)
  - final index assembly (`build_index_from_parts`)
- Added small phase structs (`NormalizedDirectoryFilters`, `ParsedNoteMaps`, `GraphEdges`)
  to keep dataflow explicit and reduce temporary mutation in the entrypoint.
- Preserved parse ordering (`doc_sort_key`), alias resolution, edge dedupe, and rank recomputation.

### Suppression debt delta (this slice)

- `assemble.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `build_with_filters`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-wendao/src/link_graph/index/build/assemble.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-wendao`
- `cargo check -p xiuxian-wendao`
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-wendao --test test_link_graph build_scope -- --nocapture`
  - result: 3 passed, 0 failed.
- `cargo test -p xiuxian-wendao --test test_link_graph cache_build -- --nocapture`
  - result: 3 passed, 0 failed.
  - note: rerun outside sandbox due local OS permission restriction in sandbox mode.
- `cargo test -p xiuxian-wendao --test test_link_graph search_core::test_link_graph_build_search_and_stats -- --nocapture`
  - result: 1 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-wendao` Graph Tool-Relevance Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-wendao/src/graph/query/tool_relevance.rs`

### Quality changes adopted

- Refactored `query_tool_relevance` into staged helpers:
  - seed collection pipeline (`collect_seed_entities` + exact/alias/substring/token match helpers)
  - hop traversal decomposition (`collect_seed_tool_scores`, `hop_decay`)
  - neighbor expansion split by direction (`extend_outgoing_neighbors`, `extend_incoming_neighbors`)
  - scoring/materialization helpers (`score_tool_entity`, `sort_and_truncate_tool_scores`)
- Introduced `ToolRelevanceContext` to centralize graph read-only references and
  reduce parameter drift across helpers.
- Preserved matching/scoring semantics and relevance sorting contract while
  removing monolithic function flow.

### Suppression debt delta (this slice)

- `tool_relevance.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-wendao/src/graph/query/tool_relevance.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-wendao`
- `cargo check -p xiuxian-wendao`
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-wendao --test test_graph tool_relevance -- --nocapture`
  - result: 3 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-wendao` Python Refresh Plan-Apply Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-wendao/src/link_graph_py/engine/refresh/plan_apply.rs`

### Quality changes adopted

- Refactored `refresh_plan_apply_impl` into explicit planning/execution helpers:
  - strategy selection (`select_refresh_strategy`, `strategy_label_and_reason`)
  - event construction (`plan_event`, `delta_apply_event`, `full_rebuild_event`)
  - payload serialization (`serialize_payload`)
  - execution branches (`run_full_refresh_with_events`, `run_delta_refresh_with_events`)
  - plan telemetry append (`push_plan_event`)
- Removed repeated JSON assembly branches by consolidating common payload/event paths.
- Preserved Python payload schema (`mode`, `changed_count`, `force_full`, `fallback`, `events`)
  and delta-fallback semantics.

### Suppression debt delta (this slice)

- `plan_apply.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-wendao/src/link_graph_py/engine/refresh/plan_apply.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-wendao`
- `cargo check -p xiuxian-wendao`
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-wendao --test test_link_graph refresh -- --nocapture`
  - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-wendao` Graph Skill-Registry Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-wendao/src/graph/skill_registry.rs`

### Quality changes adopted

- Refactored `register_skill_entities` into dedicated registration phases:
  - doc-phase collectors (`register_skill_doc`, `register_command_doc`, `collect_skill_collection`)
  - relation phases (`register_contains_relations`, `register_keyword_relations`)
  - concept/entity phase (`register_keyword_entities`)
  - small normalization/build helpers (`resolved_tool_name`, `normalized_keywords`, `skill_entity`, `tool_entity`)
- Introduced `SkillCollection` as explicit intermediate state, reducing mutable
  cross-phase coupling in the entrypoint.
- Upgraded relation writes from silent `is_ok` branching to explicit `Result`
  propagation (`?`), keeping failure signals visible to callers.
- Kept idempotent add behavior and registration counters intact via `usize::from(bool)`
  and saturating counters.

### Suppression debt delta (this slice)

- `skill_registry.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines, clippy::unnecessary_wraps)]`
  - after: removed (entry function now has real error propagation path)
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-wendao/src/graph/skill_registry.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-wendao`
- `cargo check -p xiuxian-wendao`
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-wendao --test test_graph skill_registration -- --nocapture`
  - result: 4 passed, 0 failed.
- `cargo test -p xiuxian-wendao --test test_graph tool_relevance -- --nocapture`
  - result: 3 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Milestone `xiuxian-wendao` Source `too_many_lines` Suppression Cleared)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-wendao/src`
  - result: no matches.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-llm` MCP Pool List Ops Cleanup)

### Changed files

- `packages/rust/crates/xiuxian-llm/src/mcp/pool/list_ops.rs`

### Quality changes adopted

- Removed stale `#[allow(clippy::too_many_lines)]` from `list_tools_uncached`.
- Kept round-robin fallback and cache-miss refresh behavior unchanged.

### Suppression debt delta (this slice)

- `list_ops.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-llm/src/mcp/pool/list_ops.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-llm`
- `cargo check -p xiuxian-llm`
- `cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-llm --test mcp_pool -- --nocapture`
  - result: 1 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-llm` MCP Pool Retry Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-llm/src/mcp/pool_retry.rs`

### Quality changes adopted

- Refactored `run_tools_list_with_fallback` into focused retry helpers:
  - attempt error recording (`push_attempt_error`)
  - fallback success telemetry (`log_fallback_success`)
  - reconnect+retry path (`retry_tools_list_after_reconnect`)
  - per-attempt error handling (`handle_tools_list_attempt_error`)
  - terminal error aggregation (`joined_attempt_errors`)
- Preserved retry policy semantics:
  - retryable transport errors reconnect and retry same client once
  - non-retryable errors move to next client
  - final error includes per-client attempt trace.

### Suppression debt delta (this slice)

- `pool_retry.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `run_tools_list_with_fallback`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-llm/src/mcp/pool_retry.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-llm`
- `cargo check -p xiuxian-llm`
- `cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-llm --test mcp_pool_retry -- --nocapture`
  - result: 4 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-llm` MCP Connect Attempt Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-llm/src/mcp/connect.rs`

### Quality changes adopted

- Refactored `connect_one_client_with_retry` orchestration by extracting attempt lifecycle:
  - attempt metadata struct (`ConnectAttemptMeta`)
  - attempt execution wrapper (`connect_attempt`)
  - started/success/failure logging helpers
  - retry sleep helper (`maybe_sleep_before_retry`)
- Preserved readiness gating + bounded retry behavior:
  - initial health-ready wait remains mandatory
  - per-attempt timeout/backoff calculation unchanged
  - worker-task timeout abort path and diagnostics retained.
- Reduced function complexity without introducing lint suppressions.

### Suppression debt delta (this slice)

- `connect.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `connect_one_client_with_retry`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-llm/src/mcp/connect.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-llm`
- `cargo check -p xiuxian-llm`
- `cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-llm --test mcp_pool_reconnect -- --nocapture`
  - result: 5 passed, 0 failed.
- `cargo test -p xiuxian-llm --test mcp_pool_hard_timeout -- --nocapture`
  - result: 3 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Milestone `xiuxian-llm` Source `too_many_lines` Suppression Cleared)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-llm/src -g '*.rs'`
  - result: no matches.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-qianji` Engine Compiler Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs`

### Quality changes adopted

- Refactored `compile` into staged compiler helpers:
  - manifest parse (`parse_manifest`)
  - mechanism builders per domain (`build_*_mechanism`)
  - node/edge assembly (`add_manifest_nodes`, `add_manifest_edges`)
  - ID resolution boundary (`node_index_by_id`)
- Extracted router branch parsing and persona/calibration parameter normalization.
- Preserved task-type dispatch and topology validation behavior while reducing
  monolithic compile flow.
- Kept feature-gated LLM wiring with explicit `TopologyError` fallback when `llm`
  feature is disabled.

### Suppression debt delta (this slice)

- `compiler.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `compile`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-qianji`
- `cargo check -p xiuxian-qianji`
- `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-qianji --test test_qianji_yaml_orchestration -- --nocapture`
  - result: 1 passed, 0 failed.
- `cargo test -p xiuxian-qianji --test test_probabilistic_routing -- --nocapture`
  - result: 1 passed, 0 failed.
- `cargo test -p xiuxian-qianji --test unit_qianji_execution -- --nocapture`
  - result: 1 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-qianji` Remaining Production `too_many_lines`)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-qianji/src -g '*.rs'`
  - remaining: `packages/rust/crates/xiuxian-qianji/src/scheduler/mod.rs:36`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-qianji` Scheduler Run Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-qianji/src/scheduler/mod.rs`

### Quality changes adopted

- Refactored `QianjiScheduler::run` into focused execution helpers:
  - readiness scan (`collect_pending_nodes`, `node_is_ready`, `branch_label_matches`)
  - node execution spawn/join (`spawn_node_execution_task`, `execute_pending_nodes`)
  - instruction/data application (`apply_instruction`, `apply_execution_results`, `merge_output_data`)
  - retry reset traversal (`reset_retry_nodes`)
  - step budget gate (`exceeds_step_budget`)
- Preserved scheduler runtime semantics:
  - same max-step drift guard behavior
  - same dependency + branch-label readiness criteria
  - same RetryNodes subtree reset via BFS
  - same handling policy for failed/join-error task outputs (ignored in merge phase).

### Suppression debt delta (this slice)

- `scheduler/mod.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `run`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-qianji/src/scheduler/mod.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-qianji`
- `cargo check -p xiuxian-qianji`
- `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-qianji --test unit_qianji_execution -- --nocapture`
  - result: 1 passed, 0 failed.
- `cargo test -p xiuxian-qianji --test unit_adversarial_loop -- --nocapture`
  - result: 1 passed, 0 failed.
- `cargo test -p xiuxian-qianji --test test_probabilistic_routing -- --nocapture`
  - result: 1 passed, 0 failed.
- `cargo test -p xiuxian-qianji --test test_qianji_yaml_orchestration -- --nocapture`
  - result: 1 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Milestone `xiuxian-qianji` Source `too_many_lines` Suppression Cleared)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-qianji/src -g '*.rs'`
  - result: no matches.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Graph Plan Contract Validation Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/contracts/graph_plan.rs`

### Quality changes adopted

- Refactored `GraphExecutionPlan::validate_shortcut_contract` into focused helper stages:
  - plan metadata validation (`validate_plan_metadata`)
  - deterministic step ordering and shape checks (`collect_ordered_steps`)
  - deterministic step-kind sequence checks (`validate_step_kinds`)
  - step-specific semantic guards (`validate_prepare_step`, `validate_invoke_step`, `validate_fallback_step`)
- Preserved existing contract behavior and error semantics while removing monolithic validation flow.
- Kept fallback action allowlist enforcement centralized via `is_supported_fallback_action`.

### Suppression debt delta (this slice)

- `graph_plan.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `validate_shortcut_contract`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/contracts/graph_plan.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-daochang --test contracts -- --nocapture`
  - result: 7 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test agent_injection -- --nocapture`
  - result: 3 passed, 0 failed, 1 ignored (`requires live valkey server (VALKEY_URL)`).

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Turn Shortcut Execution Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/shortcut.rs`

### Quality changes adopted

- Decomposed `Agent::handle_shortcuts` into focused async helpers:
  - workflow bridge dispatch (`handle_workflow_bridge_shortcut`)
  - graph route orchestration (`handle_graph_route_shortcut`)
  - graph execution outcome handling (`process_graph_shortcut_execution`)
  - graph completion/error side effects (`finalize_graph_shortcut_completion`, `handle_graph_shortcut_error`)
  - crawl shortcut path (`handle_crawl_shortcut`, `handle_crawl_shortcut_error`)
- Preserved runtime semantics:
  - shortcut parsing and route decision behavior unchanged
  - graph plan execution and route-to-react rewrite flow unchanged
  - recall feedback/session append/reflection side effects preserved for success and error paths.
- Resolved follow-up clippy style warnings (`collapsible_if`) to keep the slice lint-clean.

### Suppression debt delta (this slice)

- `shortcut.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `handle_shortcuts`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/shortcut.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-daochang --test shortcuts -- --nocapture`
  - result: 10 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test agent_injection -- --nocapture`
  - result: 3 passed, 0 failed, 1 ignored (`requires live valkey server (VALKEY_URL)`).

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Persistence Consolidation Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/agent/persistence/consolidation.rs`

### Quality changes adopted

- Refactored `Agent::try_consolidate` into focused helpers:
  - consolidated summary payload construction and summary-segment append (`build_consolidation_payload`)
  - embedding + episode package creation (`build_consolidated_episode`)
  - persistence orchestration (`persist_consolidated_episode`)
  - async/sync persistence branches (`spawn_async_consolidation_store_task`, `persist_consolidated_episode_sync`)
  - reward policy extraction (`consolidation_reward`)
- Preserved behavior:
  - same threshold/take gating and bounded-window drain behavior
  - same embedding failure skip path with stream event + warning
  - same async enqueue semantics and sync store/update/persist semantics
  - same completion diagnostics fields (`drained_turns`, `drained_slots`, `drained_tool_calls`).

### Suppression debt delta (this slice)

- `consolidation.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `try_consolidate`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/agent/persistence/consolidation.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-daochang --test agent_injection -- --nocapture`
  - result: 3 passed, 0 failed, 1 ignored (`requires live valkey server (VALKEY_URL)`).
- `cargo test -p xiuxian-daochang --test session_summary -- --nocapture`
  - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines`)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/agent_builder.rs:236`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/send_text.rs:73`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/parsing.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/markdown/markdown_v2.rs:12`
  - `packages/rust/crates/xiuxian-daochang/src/main.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/listen.rs:16`
  - `packages/rust/crates/xiuxian-daochang/src/agent/context_budget.rs:135`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor.rs:53`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:358`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:507`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs:38`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_support.rs:190`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/agent/persistence/turn_store.rs:43`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/command_handlers/session_commands/session_injection.rs:22`
  - `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs:141`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Persistence Turn Store Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/agent/persistence/turn_store.rs`

### Quality changes adopted

- Refactored `Agent::try_store_turn` into explicit orchestration + helpers:
  - turn outcome/reward normalization (`turn_store_outcome`)
  - existing episode lookup + new episode creation (`find_existing_turn_episode_id`, `resolve_turn_episode`)
  - embedding fallback strategy isolation (`build_turn_embedding_with_fallback`)
  - store failure handling (`handle_turn_store_failure`)
  - memory gate evaluation pipeline (`evaluate_turn_memory_gate`, `build_turn_memory_ledger`, `maybe_purge_obsolete_episode`)
  - event payload builders (`memory_gate_event_fields`, `memory_promoted_event_fields`, `publish_turn_stored_event`)
- Preserved runtime behavior:
  - same episode reuse semantics by normalized intent within scope
  - same embedding fallback to hash encoder + stream event emission
  - same gate scoring/decision inputs and promote/obsolete side effects
  - same `turn_stored` stream emission and memory state persistence/decay hooks.

### Suppression debt delta (this slice)

- `turn_store.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `try_store_turn`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/agent/persistence/turn_store.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-daochang --test agent_memory_gate_flow -- --nocapture`
  - result: 4 passed, 0 failed, 1 ignored (`requires live valkey server`).
- `cargo test -p xiuxian-daochang --test agent_memory_persistence_backend -- --nocapture`
  - result: 10 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Turn Support Route Trace Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/agent/turn_support.rs`

### Quality changes adopted

- Refactored `record_route_trace` into staged helpers:
  - trace serialization (`serialize_route_trace`)
  - stream payload assembly (`build_route_trace_stream_fields`)
  - emitted observability log (`log_route_trace_emitted`)
- Preserved behavior:
  - same `route.events` stream schema fields (including `graph_steps_json`, `failure_taxonomy_json`, injection counters)
  - same publish-failure warning path
  - same structured route-trace info logging payload.

### Suppression debt delta (this slice)

- `turn_support.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `record_route_trace`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/agent/turn_support.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-daochang --test contracts -- --nocapture`
  - result: 7 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test observability_session_events -- --nocapture`
  - result: 4 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test agent_injection -- --nocapture`
  - result: 3 passed, 0 failed, 1 ignored (`requires live valkey server (VALKEY_URL)`).

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Turn-Support Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/main.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:358`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:507`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/agent_builder.rs:236`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor.rs:53`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs:38`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/listen.rs:16`
  - `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs:141`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/send_text.rs:73`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/markdown/markdown_v2.rs:12`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/parsing.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/agent/context_budget.rs:135`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/command_handlers/session_commands/session_injection.rs:22`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Context Budget Pruning Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/agent/context_budget.rs`

### Quality changes adopted

- Refactored `prune_messages_for_token_budget_with_strategy` into focused helpers:
  - message classification and input accounting (`classify_messages_for_budget`)
  - latest non-system pinning (`select_latest_non_system`)
  - strategy-specific candidate ordering (`build_budget_candidates`)
  - bounded candidate selection (`select_candidate_messages`)
  - final packing + report accounting (`pack_selected_messages`)
- Preserved algorithm semantics:
  - same effective-budget derivation and empty/zero-budget behavior
  - same strategy order (`recent_first` vs `summary_first`)
  - same truncation and class-level token/message accounting rules.

### Suppression debt delta (this slice)

- `context_budget.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `prune_messages_for_token_budget_with_strategy`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/agent/context_budget.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-daochang --test agent_context_budget -- --nocapture`
  - result: 6 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test agent_context_window_recovery -- --nocapture`
  - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Context-Budget Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/main.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:358`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:507`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/agent_builder.rs:236`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor.rs:53`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs:38`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/listen.rs:16`
  - `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs:141`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/send_text.rs:73`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/markdown/markdown_v2.rs:12`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/parsing.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/command_handlers/session_commands/session_injection.rs:22`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Agent Builder Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/agent_builder.rs`

### Quality changes adopted

- Refactored monolithic `build_agent` into orchestration + domain helpers:
  - MCP/inference/model resolution (`resolve_runtime_mcp_servers`, `resolve_runtime_inference_url`, `resolve_runtime_model`)
  - runtime option groups (`resolve_runtime_mcp_options`, `resolve_runtime_session_options`)
  - memory runtime composition (`resolve_runtime_memory_options`) with focused sub-stages:
    - runtime embedding/persistence/recall-gate/stream settings
    - env override stages for embedding/persistence/recall-gate/stream
  - centralized runtime diagnostics logging (`log_runtime_agent_options`)
- Preserved behavior and precedence semantics:
  - same inference URL validation against MCP origin conflicts
  - same session/budget defaults and env precedence
  - same memory config merge order (runtime settings + env overrides).

### Suppression debt delta (this slice)

- `agent_builder.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `build_agent`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/agent_builder.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-daochang --bin xiuxian-daochang -- --nocapture`
  - result: 13 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test valkey_url_precedence -- --nocapture`
  - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Agent-Builder Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/main.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/send_text.rs:73`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/markdown/markdown_v2.rs:12`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/parsing.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/command_handlers/session_commands/session_injection.rs:22`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs:38`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/listen.rs:16`
  - `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor.rs:53`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:358`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:507`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs:141`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` CLI Main Dispatch Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/main.rs`

### Quality changes adopted

- Refactored `main` into focused stages:
  - tracing bootstrap (`init_tracing`)
  - command dispatch orchestration (`dispatch_command`)
- Preserved runtime behavior:
  - same `--conf` config-home override flow
  - same tracing filter precedence (`RUST_LOG` first, channel `--verbose` fallback)
  - same command routing and argument forwarding for gateway/stdio/repl/schedule/channel modes.

### Suppression debt delta (this slice)

- `main.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `main`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/main.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-daochang --bin xiuxian-daochang -- --nocapture`
  - result: 13 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Main Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/command_handlers/session_commands/session_injection.rs:22`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/markdown/markdown_v2.rs:12`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/parsing.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/listen.rs:16`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/send_text.rs:73`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor.rs:53`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:358`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:507`
  - `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs:141`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs:38`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Telegram Channel Parsing Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/parsing.rs`

### Quality changes adopted

- Refactored `parse_update_message` into focused parsing/authorization helpers:
  - update extraction (`extract_update_message` + `ParsedTelegramUpdate`)
  - sender/group ACL resolution (`resolve_sender_acl`, `log_unauthorized_sender`)
  - group policy gate (`group_policy_allows_message`)
  - final message assembly (`build_channel_message_from_parsed`)
- Preserved behavior:
  - same ACL checks and unauthorized warning output
  - same group-policy handling (`disabled`, `allowlist`, `require_mention`)
  - same session key + recipient derivation and channel message id format.

### Suppression debt delta (this slice)

- `parsing.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `parse_update_message`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/parsing.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-daochang --test channels_telegram -- --nocapture`
  - result: 29 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_telegram_group_policy -- --nocapture`
  - result: 20 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_telegram_slash_authorization -- --nocapture`
  - result: 5 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Telegram-Parsing Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:358`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:507`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs:38`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/markdown/markdown_v2.rs:12`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/send_text.rs:73`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/listen.rs:16`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor.rs:53`
  - `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs:141`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/command_handlers/session_commands/session_injection.rs:22`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Telegram Polling Listen Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/listen.rs`

### Quality changes adopted

- Refactored `listen_updates` into focused polling helpers:
  - poll request and response classification (`poll_updates`, `PollOutcome`)
  - HTTP error handling with retry policy (`handle_http_poll_error_response`)
  - API payload/error handling (`handle_api_poll_response`, `handle_api_poll_error`)
  - update stream processing (`process_polled_updates`)
- Preserved behavior:
  - same retry/backoff semantics for transport failures, 409 conflicts, and 429 `retry_after`
  - same fail-fast behavior for unauthorized/forbidden responses
  - same offset progression and message dispatch termination when receiver channel closes.

### Suppression debt delta (this slice)

- `listen.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `listen_updates`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/listen.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
- `cargo test -p xiuxian-daochang --test channels_telegram_polling -- --nocapture`
  - result: 3 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_telegram -- --nocapture`
  - result: 29 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Telegram-Listen Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:358`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:507`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/send_text.rs:73`
  - `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs:141`
  - `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor.rs:53`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/command_handlers/session_commands/session_injection.rs:22`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/markdown/markdown_v2.rs:12`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs:38`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Telegram Outbound Send Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/send_text.rs`

### Quality changes adopted

- Refactored `send_text_chunks` into focused helpers with unchanged outbound behavior:
  - chunk guard + truncation accounting (`split_chunks_with_guard`)
  - per-chunk render preparation (`prepare_chunks`)
  - payload overflow diagnostics (`warn_chunk_payload_fallbacks`)
  - deterministic send pipeline (`send_prepared_chunks`, `send_single_chunk`)
  - isolated fallback stages (`send_forced_plain_chunk`, `send_oversized_markdown_chunk`, `send_preferred_html_chunk`, `send_markdown_chunk_with_fallback`, `send_markdown_html_plain_fallback`, `send_html_then_plain_retry`)
  - truncation notice dispatch (`send_truncation_notice`)
- Preserved behavior:
  - same MarkdownV2 -> HTML -> plain fallback order
  - same parse-mode retry handling and warning logs
  - same inter-chunk throttle delay and truncation notice text.

### Suppression debt delta (this slice)

- `send_text.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `send_text_chunks`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/send_text.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - no new `too_many_lines` warning introduced by this slice.
- `cargo test -p xiuxian-daochang --test channels_telegram -- --nocapture`
  - result: 29 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_telegram_markdown -- --nocapture`
  - result: 16 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Telegram-Send Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs:141`
  - `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor.rs:53`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:358`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:507`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/command_handlers/session_commands/session_injection.rs:22`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/markdown/markdown_v2.rs:12`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs:38`

### Additional clippy follow-up surfaced in this run

- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines` also reports:
  - `packages/rust/crates/xiuxian-daochang/src/agent_builder.rs:483` (`apply_memory_runtime_embedding_settings`) at 109 lines without suppression.
- This item should be split in a dedicated follow-up slice to keep function-size policy consistent.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Namespace Refactor `agent_builder` -> `runtime_agent_factory`)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/agent_builder.rs`
- added: `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/mod.rs`
- added: `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/inference.rs`
- added: `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/memory.rs`
- added: `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/logging.rs`
- moved test: `packages/rust/crates/xiuxian-daochang/tests/agent_builder/inference_url.rs` -> `packages/rust/crates/xiuxian-daochang/tests/runtime_agent_factory/inference.rs`
- updated call sites:
  - `packages/rust/crates/xiuxian-daochang/src/main.rs`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/gateway.rs`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/stdio.rs`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/repl.rs`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/schedule.rs`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/telegram.rs`

### Quality changes adopted

- Replaced ambiguous top-level namespace `agent_builder` with feature-specific namespace `runtime_agent_factory`.
- Converted single-file implementation into directory modules with separated concerns:
  - `mod.rs`: orchestration entrypoint (`build_agent`), MCP/session runtime option assembly.
  - `inference.rs`: inference URL/model resolution, embedding backend/base-url mode resolution, MCP-origin validation.
  - `memory.rs`: memory runtime defaults + env overlays.
  - `logging.rs`: runtime option observability log emission.
- Preserved existing behavior and test coverage contract while improving code navigation and maintainability boundaries.

### Structural outcome

- previous monolith: `agent_builder.rs` (~1037 lines)
- new module layout:
  - `runtime_agent_factory/mod.rs` (272 lines)
  - `runtime_agent_factory/inference.rs` (210 lines)
  - `runtime_agent_factory/memory.rs` (536 lines)
  - `runtime_agent_factory/logging.rs` (59 lines)

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - no new `too_many_lines` regression introduced by this namespace refactor.
- `cargo test -p xiuxian-daochang --bin xiuxian-daochang -- --nocapture`
  - result: 15 passed, 0 failed (includes `runtime_agent_factory` inference/runtime resolution tests).
- `cargo test -p xiuxian-daochang --test config_settings -- --nocapture`
  - result: 5 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test valkey_url_precedence -- --nocapture`
  - result: 2 passed, 0 failed.

### Follow-up note

- `runtime_agent_factory/memory.rs` remains the largest submodule and should be split further (for example into `memory/runtime.rs`, `memory/env_overrides.rs`, `memory/recall_gate.rs`) in a dedicated next slice.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Namespace Refactor)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor.rs:53`
  - `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs:141`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:358`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:507`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/markdown/markdown_v2.rs:12`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs:38`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/command_handlers/session_commands/session_injection.rs:22`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` `runtime_agent_factory::memory` Second-Level Modularization)

### Changed files

- updated: `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/memory.rs`
- added: `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/memory/embedding.rs`
- added: `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/memory/runtime.rs`
- added: `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/memory/env_overrides.rs`

### Quality changes adopted

- Converted `runtime_agent_factory/memory.rs` from a large implementation file into a thin orchestration module.
- Introduced second-level domain modules:
  - `memory/embedding.rs`: runtime embedding defaults + env embedding overrides.
  - `memory/runtime.rs`: runtime persistence, recall gate, stream defaults.
  - `memory/env_overrides.rs`: env-based persistence/recall/stream override layering.
- Preserved external API and behavior:
  - `resolve_runtime_memory_options` remains the single entrypoint used by `runtime_agent_factory`.
  - backend-mode and base-url resolution order stays unchanged.
  - runtime settings precedence and env override precedence stay unchanged.

### Structural outcome

- pre-split: `runtime_agent_factory/memory.rs` at 536 lines.
- post-split:
  - `runtime_agent_factory/memory.rs` (33 lines)
  - `runtime_agent_factory/memory/embedding.rs` (221 lines)
  - `runtime_agent_factory/memory/runtime.rs` (190 lines)
  - `runtime_agent_factory/memory/env_overrides.rs` (112 lines)

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - no new `too_many_lines` suppression or regression introduced by this slice.
- `cargo test -p xiuxian-daochang --bin xiuxian-daochang -- --nocapture`
  - result: 15 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test config_settings -- --nocapture`
  - result: 5 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test valkey_url_precedence -- --nocapture`
  - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Memory Submodule Split)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:358`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:507`
  - `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor.rs:53`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs:141`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/markdown/markdown_v2.rs:12`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/command_handlers/session_commands/session_injection.rs:22`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs:38`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Telegram MarkdownV2 Renderer Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/markdown/markdown_v2.rs`

### Quality changes adopted

- Replaced monolithic `markdown_to_telegram_markdown_v2` implementation with a focused stateful renderer (`MarkdownV2Renderer`).
- Extracted event/tag handling into explicit methods:
  - `handle_event`, `handle_start_tag`, `handle_end_tag`
  - specialized helpers for lists, code blocks, links, task markers, footnotes.
- Preserved behavior:
  - same MarkdownV2 escape strategy for text/code/url
  - same list numbering and bullet output
  - same paragraph/code-block termination behavior and trailing-blank trimming fallback.

### Suppression debt delta (this slice)

- `markdown_v2.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `markdown_to_telegram_markdown_v2`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/markdown/markdown_v2.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - no new `too_many_lines` regression introduced by this slice.
- `cargo test -p xiuxian-daochang --test channels_telegram_markdown -- --nocapture`
  - result: 16 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_telegram -- --nocapture`
  - result: 29 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After MarkdownV2 Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:358`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:507`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs:141`
  - `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor.rs:53`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs:38`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/command_handlers/session_commands/session_injection.rs:22`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Telegram Session Injection Command Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/command_handlers/session_commands/session_injection.rs`

### Quality changes adopted

- Refactored `try_handle_session_injection_command` into focused helpers:
  - admin-required reply path (`send_session_injection_admin_required_response`)
  - event selection (`session_injection_command_event`)
  - action dispatch (`build_session_injection_response`)
  - per-action response builders (`status`, `clear`, `set_xml`).
- Preserved behavior:
  - same authorization gate and admin-required reply semantics
  - same JSON/text output contracts for status/clear/set paths
  - same observability event emission and session key attachment.

### Suppression debt delta (this slice)

- `session_injection.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `try_handle_session_injection_command`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/jobs/command_handlers/session_commands/session_injection.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - no new `too_many_lines` regression introduced by this slice.
- `cargo test -p xiuxian-daochang --test channels_telegram -- --nocapture`
  - result: 29 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_telegram_slash_authorization -- --nocapture`
  - result: 5 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Session-Injection Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs:141`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:358`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:507`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs:38`
  - `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor.rs:53`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Graph Plan Executor Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor.rs`

### Quality changes adopted

- Refactored `execute_graph_shortcut_plan` from monolithic control flow into explicit execution phases:
  - context/state model (`GraphPlanExecutionContext`, `GraphPlanExecutionState`)
  - ordered-step resolution (`resolve_ordered_steps_or_exit`)
  - per-step dispatch (`execute_graph_plan_step`)
  - focused step handlers (`prepare`, `invoke`, `evaluate_fallback`)
  - terminal finalization (`finish_graph_plan_execution`)
  - centralized failure/trace emission helpers (`fail_step_and_exit`, `fail_terminal_and_exit`, `emit_graph_trace_for_state`).
- Preserved behavior contracts:
  - same route-trace emission timing and payload fields
  - same fallback semantics (`retry_bridge_without_metadata`, `route_to_react`, `abort`)
  - same step status vocabulary in trace records
  - same tool-attempt summary accounting and terminal outcomes.

### Suppression debt delta (this slice)

- `executor.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `execute_graph_shortcut_plan`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/agent/graph/executor.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - no new `too_many_lines` regression introduced by this slice.
- `cargo test -p xiuxian-daochang --test agent_injection -- --nocapture`
  - result: 3 passed, 0 failed, 1 ignored (`requires live valkey server`).
- `cargo test -p xiuxian-daochang --test observability_session_events -- --nocapture`
  - result: 4 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test contracts -- --nocapture`
  - result: 7 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Graph-Executor Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:358`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:507`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs:38`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs:141`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Injection Role-Mix Selection Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs`

### Quality changes adopted

- Refactored role-mix selection from a monolithic branch chain into focused helpers:
  - role candidate collection (`collect_role_candidates`)
  - profile construction by mode (`build_role_mix_profile`)
  - category-driven role insertion (`maybe_push_role_for_categories`, `has_any_category`)
  - deterministic default role factory (`default_role_mix_role`).
- Preserved behavior:
  - same category-to-role mapping and role weights
  - same deterministic default role when no signals are present
  - same profile IDs/rationale text semantics across `single`, `classified`, `hybrid`.

### Suppression debt delta (this slice)

- `assembler.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `select_role_mix`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - no new `too_many_lines` regression introduced by this slice.
- `cargo test -p xiuxian-daochang --test agent_injection -- --nocapture`
  - result: 3 passed, 0 failed, 1 ignored (`requires live valkey server`).
- `cargo test -p xiuxian-daochang --test contracts -- --nocapture`
  - result: 7 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Injection-Assembler Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:358`
  - `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs:507`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs:38`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Embedding Dispatch Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs`

### Quality changes adopted

- Refactored concurrent embedding chunk dispatch into focused helpers:
  - task seeding (`spawn_chunk_task`)
  - join result normalization (`collect_concurrent_chunk_result`)
  - abort/drain helper (`abort_pending_chunks`)
  - deterministic merge stage (`merge_concurrent_chunk_results`).
- Refactored per-chunk dispatch path into layered backend helpers:
  - permit acquisition/metrics (`acquire_in_flight_permit`)
  - backend dispatch router (`dispatch_chunk_by_backend`)
  - backend-specific dispatch (`dispatch_http_backend`, `dispatch_openai_backend`, `dispatch_litellm_backend`)
  - feature-gated LiteLLM implementations split by cfg.
- Preserved behavior:
  - same chunk ordering and vector-count validation
  - same concurrency gate semantics and dispatch telemetry
  - same fallback sequence across HTTP/OpenAI/LiteLLM/MCP branches.

### Suppression debt delta (this slice)

- `embedding/client.rs` function-level suppressions:
  - before: `#[allow(clippy::too_many_lines)]` on `dispatch_embeddings_concurrent`
  - before: `#[allow(clippy::too_many_lines)]` on `dispatch_chunk_with_runtime`
  - after: both removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/embedding/client.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - no new `too_many_lines` regression introduced by this slice.
- `cargo test -p xiuxian-daochang --test embedding_client -- --nocapture`
  - result: 8 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test embedding_client_cache -- --nocapture`
  - result: 2 passed, 0 failed.
- note: `cargo test -p xiuxian-daochang --test embedding/transport_litellm -- --nocapture`
  - no matching test target (path-style target name is invalid for this crate layout).

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Embedding-Dispatch Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs:38`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Discord Runtime Dispatch Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs`

### Quality changes adopted

- Refactored `process_discord_message_with_interrupt` into focused helpers:
  - preemption and inbound logging (`log_preempted_turn`, `log_inbound_user_message`)
  - generation lifecycle setup (`begin_active_generation`)
  - typing + foreground execution wrapper (`run_foreground_turn_with_typing`)
  - result-to-reply projection (`render_foreground_turn_reply`)
  - outbound reply send/log (`send_discord_reply`).
- Introduced `ForegroundTurnInput` to avoid wide helper signatures and keep dispatch boundaries explicit.
- Preserved behavior:
  - same stop-command handling and managed-command short-circuit
  - same interrupt generation semantics and timeout/interrupt reply behavior
  - same logging event keys for failed/timed-out/interrupted turn paths.

### Suppression debt delta (this slice)

- `dispatch.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `process_discord_message_with_interrupt`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - no new `too_many_lines` regression introduced by this slice.
- `cargo test -p xiuxian-daochang --test channels_discord -- --nocapture`
  - result: 9 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_discord_slash_authorization -- --nocapture`
  - result: 7 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_managed_commands -- --nocapture`
  - result: 4 passed, 0 failed.
- note: `cargo test -p xiuxian-daochang --test discord_runtime/managed_commands -- --nocapture`
  - no matching test target (path-style target name is invalid for this crate layout).

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Discord-Dispatch Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs:49`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Discord Runtime Gateway/Ingress Orchestration Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/foreground.rs` (new)
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/lib.rs`
- `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs`

### Quality changes adopted

- Extracted shared foreground runtime orchestration into a dedicated module:
  - `DiscordForegroundRuntime` now owns foreground task concurrency gate, job manager bridge, interrupt controller, and completion fan-out.
  - shared lifecycle helpers introduced: `spawn_foreground_turn`, `push_completion`, `join_next_foreground_task`, `abort_and_drain_foreground_tasks`.
- Reduced duplication between gateway and ingress runtime loops:
  - both runtime entrypoints now reuse one foreground orchestration abstraction.
  - gateway/ingress files now focus on transport-specific wiring only.
- Replaced ingress wide argument list with typed request object:
  - introduced `DiscordIngressRunRequest` and updated callsite wiring in `nodes/channel/discord.rs`.
  - API re-exported from runtime/channel/lib surfaces for stable downstream usage.
- Preserved behavior contracts:
  - same inbound queue semantics and foreground concurrency cap.
  - same managed command completion push behavior.
  - same startup banners and graceful shutdown semantics.

### Suppression debt delta (this slice)

- `gateway.rs` function-level suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `run_discord_gateway`
  - after: removed
- `run.rs` function-level suppressions:
  - before: `#[allow(clippy::too_many_arguments, clippy::too_many_lines)]` on `run_discord_ingress`
  - after: removed (replaced by `DiscordIngressRunRequest` + shared runtime extraction)
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime -g '*.rs'`
    returns no matches in `gateway.rs` and `run.rs`.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass for `too_many_lines` target set.
  - note: pre-existing non-target warnings remain (`gateway/http/runtime.rs` `unnested_or_patterns`; `agent/bootstrap.rs` `map_unwrap_or`).
- `cargo test -p xiuxian-daochang --test channels_discord -- --nocapture`
  - result: 9 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_discord_slash_authorization -- --nocapture`
  - result: 7 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_discord_ingress -- --nocapture`
  - result: 5 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_managed_commands -- --nocapture`
  - result: 4 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Discord Runtime Orchestration Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs:24`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Discord Node Runtime Wiring Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs`

### Quality changes adopted

- Refactored node-level Discord command execution into explicit staged configs:
  - runtime launch resolution (`resolve_discord_runtime_launch_config`)
  - ACL/policy resolution (`resolve_discord_acl_launch_config`)
  - transport dispatch (`run_discord_channel_mode`).
- Introduced focused request/config carriers to avoid wide function signatures:
  - `DiscordRuntimeLaunchConfig`
  - `DiscordAclLaunchConfig`
  - `DiscordChannelModeRequest`.
- Preserved behavior contracts:
  - same environment/setting precedence for token, partition, ingress bind/path/secret, queue and timeout settings
  - same ACL override semantics and slash/control policy wiring
  - same gateway vs ingress runtime mode routing.

### Suppression debt delta (this slice)

- `nodes/channel/discord.rs` suppressions:
  - before: `#[allow(clippy::similar_names, clippy::too_many_lines)]` on `run_discord_channel_command`
  - before: `#[allow(clippy::similar_names, clippy::too_many_arguments)]` on `run_discord_channel_mode`
  - after: removed and replaced with decomposed config-oriented flow
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/nodes/channel/discord.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass for target warning (`too_many_lines`); only pre-existing non-target warnings remain.
- `cargo test -p xiuxian-daochang --bin xiuxian-daochang -- --nocapture`
  - result: 16 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_discord -- --nocapture`
  - result: 9 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_discord_slash_authorization -- --nocapture`
  - result: 7 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_discord_ingress -- --nocapture`
  - result: 5 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Discord Node Wiring Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs:485`
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`
  - `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs:125`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Memory Stream Consumer Loop Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs`

### Quality changes adopted

- Decomposed monolithic consumer loop into focused stages:
  - client/bootstrap (`open_stream_consumer_client`, `stream_consumer_response_timeout_ms`)
  - connect and group readiness (`connect_stream_consumer`, `ensure_consumer_group_before_read`)
  - read loop orchestration (`consume_stream_until_reconnect`, `handle_stream_read_success`, `handle_stream_read_error`)
  - event processing (`process_stream_events`, `process_stream_event`)
  - structured failure logging/retry helpers (`log_stream_event_failure`, `log_stream_read_reconnect`, `sleep_reconnect_backoff`).
- Preserved behavior contracts:
  - same exponential reconnect backoff strategy
  - same missing-consumer-group recovery behavior
  - same promoted-candidate queue and ack/metrics sequencing
  - same warn/trace throttling semantics via repeated-failure policy.

### Suppression debt delta (this slice)

- `memory_stream_consumer.rs` suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `run_consumer_loop`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - no new `too_many_lines` regressions.
- `cargo test -p xiuxian-daochang --lib memory_stream_consumer -- --nocapture`
  - result: 14 passed, 0 failed, 4 ignored (`requires live Valkey/Redis`).

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Redis Stream Publish Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs`

### Quality changes adopted

- Extracted stream publish Lua script into a dedicated constant (`PUBLISH_STREAM_EVENT_SCRIPT`).
- Introduced publish context model (`StreamEventPublishContext`) and helpers:
  - input validation (`validate_stream_event_publish_input`)
  - field extraction (`find_stream_field`)
  - command construction (`build_publish_stream_event_cmd`)
  - context construction (`build_stream_event_publish_context`).
- Reduced `publish_stream_event` to orchestration-only logic with unchanged side effects.

### Suppression debt delta (this slice)

- `redis_backend.rs` suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `publish_stream_event`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - no new `too_many_lines` regressions.
- `cargo test -p xiuxian-daochang --test session_redis -- --nocapture`
  - result: 0 passed, 0 failed, 5 ignored (`requires live Valkey/Redis`).

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After Memory-Consumer + Redis-Publish Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs:5`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` ReAct Loop Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop.rs`

### Quality changes adopted

- Re-architected `run_react_loop` from monolithic flow into staged orchestration:
  - decision stage (`prepare_react_decision`)
  - message bootstrap stage (`prepare_react_messages`, summary + injection prep)
  - memory recall stage (`apply_memory_recall_if_enabled` and recall recording helpers)
  - normalization/context-budget stage (`normalize_and_pack_react_messages`)
  - round execution stage (`execute_react_rounds`) with focused helpers for:
    - round-limit guard,
    - context-window repair retries,
    - tool-call execution,
    - terminal success/error finalization.
- Added explicit state/context carriers to reduce cross-branch coupling:
  - `ReactConversationState`
  - `TurnRuntimeContext`
  - `MemoryRecallPlanContext` / `MemoryRecallExecutionContext`
  - `ContextRepairResult`.
- Preserved behavior contracts:
  - same policy-hint application and omega decision recording
  - same memory recall metrics/snapshot semantics
  - same context-window repair strategy order (`drop_tools_only` -> budget pruning with/without tools)
  - same tool-call round accounting and reflection feedback semantics.

### Suppression debt delta (this slice)

- `react_loop.rs` suppression:
  - before: `#[allow(clippy::too_many_lines)]` on `run_react_loop`
  - after: removed
- Validation:
  - `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
    returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass for target warning (`too_many_lines`).
  - note: pre-existing non-target warnings remain in untouched files (`gateway/http/runtime.rs`, `agent/bootstrap.rs`).
- `cargo test -p xiuxian-daochang --bin xiuxian-daochang -- --nocapture`
  - result: 16 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test agent_context_window_recovery -- --nocapture`
  - result: 2 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test shortcuts -- --nocapture`
  - result: 10 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test agent_summary -- --nocapture`
  - result: 3 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test agent_integration -- --nocapture`
  - result: 0 passed, 0 failed, 1 ignored (`requires OPENAI_API_KEY`/external MCP).
- note: `cargo test -p xiuxian-daochang --test agent_memory_recall -- --nocapture`
  - no matching test target in this crate layout.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Remaining Production `too_many_lines` After ReAct Loop Cleanup)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
- remaining:
  - none

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Residual Pedantic Warning Cleanup)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/gateway/http/runtime.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap.rs`

### Quality changes adopted

- Resolved `unnested_or_patterns` in embedding backend resolution branching and mistral backend hint matcher.
- Replaced `Option::map(...).unwrap_or(...)` with `Option::map_or(...)` in embedding timeout resolution.
- Preserved behavior contracts while removing pedantic warning noise in touched runtime/bootstrap paths.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass with no emitted warnings in this command context.
- `cargo test -p xiuxian-daochang --bin xiuxian-daochang -- --nocapture`
  - result: 16 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test agent_context_window_recovery -- --nocapture`
  - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Snapshot `xiuxian-daochang` Production `too_many_lines` + tracked pedantic warnings)

### Validation evidence

- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src -g '*.rs'`
  - remaining: none
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - tracked warning set from this command context: none

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Memory Stream Consumer Directory Modularization)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/types.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/parsing.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/stream.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/processing.rs`
- deleted: `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer_old.rs`
- `packages/rust/crates/xiuxian-daochang/src/gateway/http/runtime.rs`

### Quality changes adopted

- Converted memory stream consumer from one large file implementation bridge into a directory module with explicit concern boundaries:
  - `mod.rs`: orchestration loop and reconnect/error routing.
  - `types.rs`: runtime config, event models, retry/backoff and shared value helpers.
  - `parsing.rs`: `XREADGROUP` reply decoding and value normalization.
  - `stream.rs`: connection lifecycle, consumer-group bootstrap, stream read path, timeout/error summarization.
  - `processing.rs`: event processing, promoted-candidate queueing, ACK + metrics updates.
- Kept behavior stable while reducing maintenance coupling and making ownership clearer for follow-up changes.
- Removed a residual clippy `needless_borrow` warning in gateway embedding runtime setup (`gateway/http/runtime.rs`) without changing behavior.

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced in production paths.
- Previous `too_many_lines` cleanup status remains stable after modularization.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass; no warnings emitted in this command context.
- `cargo test -p xiuxian-daochang --lib memory_stream_consumer -- --nocapture`
  - result: 14 passed, 0 failed, 4 ignored (`requires live Valkey/Redis`).
- `cargo test -p xiuxian-daochang --test gateway_http -- --nocapture`
  - result: 7 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Redis Backend Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/session/redis_backend.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/message_store.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/window_store.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/summary_store.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/stream_events.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/snapshots.rs`

### Quality changes adopted

- Converted `redis_backend` from one large module file into a directory module with explicit ownership boundaries:
  - `mod.rs`: runtime config, backend struct/core keys, connection bootstrap, command/pipeline retry runtime.
  - `message_store.rs`: message append/replace/load/clear + legacy metadata payload decode compatibility.
  - `window_store.rs`: bounded window append/read/stats/clear/drain behavior.
  - `summary_store.rs`: summary append/read/count/clear behavior.
  - `stream_events.rs`: stream publish + metrics update script and context shaping helpers.
  - `snapshots.rs`: atomic reset/resume/drop snapshot lifecycle.
- Preserved API surface (`RedisSessionBackend` methods unchanged) while reducing per-file cognitive load and making concern ownership explicit.

### Structural outcome

- Previous single-file size (`redis_backend.rs`): `1181` lines.
- New split sizes:
  - `redis_backend/mod.rs`: `307`
  - `redis_backend/message_store.rs`: `209`
  - `redis_backend/window_store.rs`: `198`
  - `redis_backend/summary_store.rs`: `125`
  - `redis_backend/stream_events.rs`: `183`
  - `redis_backend/snapshots.rs`: `201`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced in production paths.
- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/session -g '*.rs'` returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass in this command context.
- `cargo test -p xiuxian-daochang --test session_redis -- --nocapture`
  - result: 0 passed, 0 failed, 5 ignored (`requires live valkey server`).
- `cargo test -p xiuxian-daochang --test config_and_session -- --nocapture`
  - result: 7 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` LLM Client Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/llm/client.rs`
- `packages/rust/crates/xiuxian-daochang/src/llm/client/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/llm/client/init.rs`
- `packages/rust/crates/xiuxian-daochang/src/llm/client/chat.rs`

### Quality changes adopted

- Split LLM client implementation by concern:
  - `client/mod.rs`: client struct and module surface.
  - `client/init.rs`: construction path (`new`), backend/provider selection metadata, HTTP client builder.
  - `client/chat.rs`: chat dispatch, backend routing, HTTP chat path, litellm-rs path.
- Preserved public API and behavior:
  - `LlmClient::new(...)` signature unchanged.
  - `LlmClient::chat(...)` signature and backend routing semantics unchanged.
  - in-flight semaphore gate behavior unchanged.

### Structural outcome

- In-session baseline before this split:
  - `llm/client.rs`: `295` lines.
- Current split sizes:
  - `llm/client/mod.rs`: `35`
  - `llm/client/init.rs`: `124`
  - `llm/client/chat.rs`: `157`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(clippy::wildcard_imports\\)|allow\\(clippy::too_many_lines\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/llm/client -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed (no new warnings in touched paths).
- `cargo test -p xiuxian-daochang --lib llm::backend_tests -- --nocapture`
  - result: 6 passed, 0 failed.
- `cargo test -p xiuxian-daochang --lib llm::provider_mode_tests -- --nocapture`
  - result: 7 passed, 0 failed.
- `cargo test -p xiuxian-daochang --lib llm::http_request_tests -- --nocapture`
  - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Runtime Settings Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/config/settings.rs`
- `packages/rust/crates/xiuxian-daochang/src/config/settings/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/config/settings/types.rs`
- `packages/rust/crates/xiuxian-daochang/src/config/settings/merge.rs`
- `packages/rust/crates/xiuxian-daochang/src/config/settings/loader.rs`

### Quality changes adopted

- Converted runtime settings from one monolithic file into an explicit directory module by concern:
  - `types.rs`: settings schema and serde models.
  - `merge.rs`: overlay merge policy and nested merge helpers.
  - `loader.rs`: system/user path resolution, YAML loading, config-home override plumbing.
  - `mod.rs`: interface-only re-export surface.
- Preserved current behavior and API contract:
  - `load_runtime_settings`
  - `runtime_settings_paths`
  - `load_runtime_settings_from_paths`
  - `set_config_home_override`
  - existing settings structs used by config/channels/tests.
- Removed newly introduced pedantic warnings during split by:
  - restoring `#[must_use]` on `load_runtime_settings`.
  - replacing wildcard import in `merge.rs` with explicit imports.

### Structural outcome

- Previous single-file size (`settings.rs`): `1026` lines.
- New split sizes:
  - `settings/merge.rs`: `632`
  - `settings/types.rs`: `285`
  - `settings/loader.rs`: `112`
  - `settings/mod.rs`: `24`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced in production paths.
- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/config -g '*.rs'` returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass in this command context.
- `cargo test -p xiuxian-daochang --test config_settings -- --nocapture`
  - result: 5 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test config_and_session -- --nocapture`
  - result: 7 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Runtime Settings Merge Submodule Split)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/config/settings/merge.rs`
- `packages/rust/crates/xiuxian-daochang/src/config/settings/merge/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/config/settings/merge/core.rs`
- `packages/rust/crates/xiuxian-daochang/src/config/settings/merge/telegram.rs`
- `packages/rust/crates/xiuxian-daochang/src/config/settings/merge/discord.rs`

### Quality changes adopted

- Further decomposed settings merge logic into domain-focused modules:
  - `merge/core.rs`: runtime/session/memory/embedding/mistral + base config merge.
  - `merge/telegram.rs`: telegram + ACL + group/topic deep-merge behavior.
  - `merge/discord.rs`: discord + ACL + map merge behavior.
  - `merge/mod.rs`: interface-only module declaration.
- Preserved merge semantics while tightening visibility boundaries:
  - `RuntimeSettings::merge` exposed only within `settings` module scope.
  - Telegram/Discord top-level merge methods exposed to sibling merge modules only.
- Removed wildcard imports and kept explicit imports to avoid pedantic warning debt.

### Structural outcome

- Previous `settings/merge.rs`: `632` lines.
- New split sizes:
  - `settings/merge/core.rs`: `203`
  - `settings/merge/telegram.rs`: `265`
  - `settings/merge/discord.rs`: `171`
  - `settings/merge/mod.rs`: `3`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced in production paths.
- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/config/settings -g '*.rs'` returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass in this command context.
- `cargo test -p xiuxian-daochang --test config_settings -- --nocapture`
  - result: 5 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test config_and_session -- --nocapture`
  - result: 7 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Embedding Client Directory Modularization)

### Changed files

- renamed: `packages/rust/crates/xiuxian-daochang/src/embedding/client.rs` -> `packages/rust/crates/xiuxian-daochang/src/embedding/client/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/embedding/client/chunk_dispatch.rs`
- `packages/rust/crates/xiuxian-daochang/src/embedding/client/backend_dispatch.rs`
- `packages/rust/crates/xiuxian-daochang/src/embedding/client/support.rs`

### Quality changes adopted

- Converted embedding client from single-file implementation into domain-focused directory modules:
  - `client/mod.rs`: public API, runtime model, constructors, batch orchestration.
  - `client/chunk_dispatch.rs`: chunk-level dispatch pipeline, in-flight permit handling, concurrent result collection/merge.
  - `client/backend_dispatch.rs`: backend routing (`http` / `openai_http` / `litellm_rs`) and fallback policy.
  - `client/support.rs`: chunk range planning, HTTP client builder, env parsing, LiteLLM API key resolution.
- Preserved public behavior and API while tightening boundaries between orchestration, backend strategy, and utility concerns.

### Structural outcome

- Previous single-file size (`embedding/client.rs`): `880` lines.
- New split sizes:
  - `embedding/client/mod.rs`: `439`
  - `embedding/client/backend_dispatch.rs`: `196`
  - `embedding/client/chunk_dispatch.rs`: `191`
  - `embedding/client/support.rs`: `85`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced in production paths.
- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/embedding -g '*.rs'` returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass in this command context.
- `cargo test -p xiuxian-daochang --test embedding_client -- --nocapture`
  - result: 9 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test embedding_client_cache -- --nocapture`
  - result: 2 passed, 0 failed.

### Addendum (2026-02-24, Embedding Client Split Validation Update)

- Fixed feature-compatibility in litellm API-key resolution wiring after split:
  - `client/support.rs` returns structured key resolution metadata.
  - `client/mod.rs` now stores `litellm_api_key` using `resolution.api_key` and logs source.
- Additional regression validation:
  - `cargo test -p xiuxian-daochang --test gateway_http -- --nocapture`
    - result: 7 passed, 0 failed.
- Final split file sizes after compatibility fix:
  - `embedding/client/mod.rs`: `450`
  - `embedding/client/backend_dispatch.rs`: `196`
  - `embedding/client/chunk_dispatch.rs`: `191`
  - `embedding/client/support.rs`: `117`

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Graph Executor Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor/types.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor/steps.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor/trace.rs`

### Quality changes adopted

- Converted graph shortcut executor from one large file into a directory module with explicit separation of responsibilities:
  - `executor/mod.rs`: entrypoint orchestration, terminal completion/failure handling, route-trace emission.
  - `executor/types.rs`: execution input/output/error contracts and runtime state/context metadata.
  - `executor/steps.rs`: per-step execution and fallback routing logic.
  - `executor/trace.rs`: ordered-step validation, fallback-action decoding, failure taxonomy, step-trace helpers.
- Preserved external API surface used by `graph/mod.rs` (`GraphPlanExecutionInput`, `GraphPlanExecutionOutcome`, `GraphPlanExecutionError`) while narrowing internal visibility where possible.
- Restored test compatibility by exposing `ordered_steps` into module test scope (`#[cfg(test)] use ...`) so existing `super::ordered_steps` assertions remain valid.

### Structural outcome

- Previous single-file size (`agent/graph/executor.rs`): `828` lines.
- New split sizes:
  - `agent/graph/executor/steps.rs`: `468`
  - `agent/graph/executor/mod.rs`: `196`
  - `agent/graph/executor/trace.rs`: `100`
  - `agent/graph/executor/types.rs`: `99`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced in production paths.
- `rg -n "too_many_lines" packages/rust/crates/xiuxian-daochang/src/agent/graph -g '*.rs'` returned no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
- `cargo check -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: `xiuxian-daochang` passed in this command context.
  - note: an existing warning remained in untouched crate `xiuxian-llm/src/embedding/openai_compat.rs`.
- `cargo test -p xiuxian-daochang --test agent_graph_bridge -- --nocapture`
  - result: 3 passed, 0 failed.
- `cargo test -p xiuxian-daochang --lib ordered_steps -- --nocapture`
  - result: 2 passed, 0 failed.
- `cargo test -p xiuxian-daochang --lib execute_graph_shortcut_plan -- --nocapture`
  - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Graph Executor Steps Sub-Module Decomposition, Pass 2)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor/steps.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor/steps/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor/steps/dispatch.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor/steps/invoke.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor/steps/fallback.rs`

### Quality changes adopted

- Replaced the remaining large `steps.rs` single file with directory modules split by responsibility:
  - `steps/dispatch.rs`: ordered-step resolution and step-kind dispatch.
  - `steps/invoke.rs`: invoke-step execution, tool-name resolution, transport/result handling.
  - `steps/fallback.rs`: fallback-step evaluation and retry/reroute/abort flows.
  - `steps/mod.rs`: interface-only module declaration.
- Preserved execution behavior and state transitions while narrowing cross-file method visibility via `pub(super)` only where sibling modules require access.
- No broad lint suppression added; this slice keeps the existing hard rule of fixing structure instead of muting warnings.

### Structural outcome

- Previous `agent/graph/executor/steps.rs`: `468` lines.
- New split sizes:
  - `agent/graph/executor/steps/dispatch.rs`: `91`
  - `agent/graph/executor/steps/invoke.rs`: `139`
  - `agent/graph/executor/steps/fallback.rs`: `260`
  - `agent/graph/executor/steps/mod.rs`: `3`

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: blocked by existing compile error in untouched dependency crate `xiuxian-llm`:
    - `packages/rust/crates/xiuxian-llm/src/embedding/openai_compat.rs:104`
    - `packages/rust/crates/xiuxian-llm/src/embedding/openai_compat.rs:118`
    - error: `E0505 cannot move out of resp because it is borrowed`.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: blocked by the same upstream `xiuxian-llm` compile error.
- `cargo test -p xiuxian-daochang --test agent_graph_bridge -- --nocapture`
- `cargo test -p xiuxian-daochang --lib ordered_steps -- --nocapture`
- `cargo test -p xiuxian-daochang --lib execute_graph_shortcut_plan -- --nocapture`
  - result: all three blocked by the same `xiuxian-llm` compile error above.

### Addendum (2026-02-24, Graph Executor Steps Pass-2 Revalidation)

- Revalidated the pass-2 `steps` directory split after dependency baseline stabilized.
- Validation evidence:
  - `cargo test -p xiuxian-daochang --test agent_graph_bridge -- --nocapture`
    - result: 3 passed, 0 failed.
  - `cargo test -p xiuxian-daochang --lib ordered_steps -- --nocapture`
    - result: 2 passed, 0 failed.
  - `cargo test -p xiuxian-daochang --lib execute_graph_shortcut_plan -- --nocapture`
    - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Bounded Session Store Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/session/bounded_store.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/bounded_store/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/bounded_store/window_ops.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/bounded_store/snapshot_ops.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/bounded_store/summary_ops.rs`

### Quality changes adopted

- Converted `bounded_store` from a single large module to directory modules by domain responsibility:
  - `mod.rs`: store shape + constructors + backend bootstrapping.
  - `window_ops.rs`: bounded window read/write/replace/drain/stats/clear operations.
  - `snapshot_ops.rs`: atomic reset/resume/drop snapshot operations.
  - `summary_ops.rs`: summary append/read/count operations and char-truncation helper.
- Preserved external API contract (`BoundedSessionStore` type + public methods) while isolating runtime paths.
- Kept lint policy strict: no new broad `#[allow(...)]` added.

### Structural outcome

- Previous single-file size (`session/bounded_store.rs`): `724` lines.
- New split sizes:
  - `session/bounded_store/mod.rs`: `139`
  - `session/bounded_store/window_ops.rs`: `377`
  - `session/bounded_store/snapshot_ops.rs`: `77`
  - `session/bounded_store/summary_ops.rs`: `157`

### Suppression debt delta (this slice)

- No new suppression added in production paths.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed for `xiuxian-daochang`; remaining warning in untouched `xiuxian-llm` function length.
- `cargo test -p xiuxian-daochang --test session_summary -- --nocapture`
  - result: 2 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test session_redis -- --nocapture`
  - result: 0 passed, 0 failed, 5 ignored (requires live valkey server).

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Persistence Turn Store Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/agent/persistence/turn_store.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/persistence/turn_store/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/persistence/turn_store/episode.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/persistence/turn_store/gate.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/persistence/mod.rs`

### Quality changes adopted

- Replaced monolithic turn-store logic with responsibility-focused directory modules:
  - `turn_store/mod.rs`: append + turn-store orchestration and publish event.
  - `turn_store/episode.rs`: episode resolution, embedding fallback, store-failure handling.
  - `turn_store/gate.rs`: memory gate evaluation, purge path, gate/promote stream payload shaping.
- Removed the previous file-level wildcard import suppression from turn-store path by replacing wildcard dependencies with explicit module imports.
- Kept behavior and public API unchanged while reducing mixed concerns in one file.

### Structural outcome

- Previous single-file size (`agent/persistence/turn_store.rs`): `517` lines.
- New split sizes:
  - `agent/persistence/turn_store/mod.rs`: `125`
  - `agent/persistence/turn_store/episode.rs`: `152`
  - `agent/persistence/turn_store/gate.rs`: `270`

### Suppression debt delta (this slice)

- `turn_store` path `#[allow(clippy::wildcard_imports)]`:
  - before: 1
  - after: 0

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed for `xiuxian-daochang`; remaining warning in untouched `xiuxian-llm` function length.
- `cargo test -p xiuxian-daochang --test agent_memory_persistence_backend -- --nocapture`
  - result: 10 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test agent_memory_gate_flow -- --nocapture`
  - result: 4 passed, 0 failed, 1 ignored (requires live valkey server).

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Persistence Consolidation Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/agent/persistence/consolidation.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/persistence/consolidation/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/persistence/consolidation/payload.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/persistence/mod.rs`

### Quality changes adopted

- Converted consolidation persistence path from single-file implementation into directory modules:
  - `consolidation/mod.rs`: orchestration flow, enqueue/store routing, async/sync persist branches.
  - `consolidation/payload.rs`: drained-turn payload assembly, summary-segment append, consolidated-episode construction.
- Removed implicit dependency coupling via wildcard imports by replacing `super::*` usage with explicit imports.
- Preserved runtime behavior (`try_consolidate` thresholds, async/sync persistence semantics, stream event payload contracts).

### Structural outcome

- Previous single-file size (`agent/persistence/consolidation.rs`): `286` lines.
- New split sizes:
  - `agent/persistence/consolidation/mod.rs`: `206`
  - `agent/persistence/consolidation/payload.rs`: `103`

### Suppression debt delta (this slice)

- `agent/persistence` path `#[allow(clippy::wildcard_imports)]`:
  - before: 1 (remaining in `consolidation.rs` before split)
  - after: 0

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: completed; remaining warnings are in untouched paths:
    - `packages/rust/crates/xiuxian-llm/src/embedding/openai_compat.rs`
    - `packages/rust/crates/xiuxian-daochang/src/embedding/client/backend_dispatch.rs`
- `cargo test -p xiuxian-daochang --test session_summary -- --nocapture`
  - result: 2 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test agent_injection -- --nocapture`
  - result: 3 passed, 0 failed, 1 ignored (requires live valkey server).

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Embedding LiteLLM Dispatch Function Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/embedding/client/backend_dispatch.rs`

### Quality changes adopted

- Decomposed the previous large LiteLLM dispatch function into focused helpers while preserving fallback order and behavior:
  - `litellm_api_key_is_present`
  - `dispatch_ollama_model_with_feature`
  - `dispatch_standard_litellm_model_with_feature`
- Kept all provider fallback semantics unchanged:
  - `ollama/*` path: OpenAI-compatible direct -> HTTP fallback -> LiteLLM provider fallback (when API key exists) -> MCP fallback.
  - standard path: LiteLLM provider (when API key exists) -> HTTP fallback -> MCP fallback.
- Preserved existing observability events and messages across each fallback branch.

### Structural outcome

- `embedding/client/backend_dispatch.rs` remains module-focused while removing the prior single oversized LiteLLM dispatcher branch.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: `too_many_lines` warning for `xiuxian-daochang/src/embedding/client/backend_dispatch.rs` is resolved.
  - remaining warning is in untouched crate path:
    - `packages/rust/crates/xiuxian-llm/src/embedding/openai_compat.rs`
- `cargo test -p xiuxian-daochang --test embedding_client -- --nocapture`
  - result: 12 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test embedding_client_cache -- --nocapture`
  - result: 2 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test gateway_http -- --nocapture`
  - result: 7 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-llm` OpenAI-Compatible Embedding Decomposition)

### Changed files

- `packages/rust/crates/xiuxian-llm/src/embedding/openai_compat.rs`

### Quality changes adopted

- Decomposed `embed_openai_compatible` into focused phases with behavior preserved:
  - request body assembly (`build_openai_request_body`)
  - request send + retry decision (`send_openai_request`)
  - HTTP status branch handling (`handle_openai_response`, `handle_non_success_status`)
  - success-body decode and vector extraction (`read_success_vectors`)
- Introduced local attempt control enum (`AttemptOutcome<T>`) to make retry/terminal/success flow explicit.
- Preserved all existing fallback semantics and observability event fields.
- Removed the previous `clippy::too_many_lines` hotspot in this transport path without adding lint suppressions.

### Compile and test evidence

- `cargo fmt -p xiuxian-llm`
  - result: passed.
- `cargo check -p xiuxian-llm`
  - result: passed.
- `cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - result: passed for `too_many_lines` target; no `too_many_lines` warning in `openai_compat.rs`.
- `cargo test -p xiuxian-llm --test embedding_openai_compat -- --nocapture`
  - result: 2 passed, 0 failed.
- `cargo test -p xiuxian-llm --test embedding_backend -- --nocapture`
  - result: 4 passed, 0 failed.
- `cargo test -p xiuxian-llm --test llm_backend -- --nocapture`
  - result: 3 passed, 0 failed.

### Cross-crate revalidation

- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: completed with no `too_many_lines` warning in `xiuxian-llm` or `xiuxian-daochang` touched paths.
  - note: remaining warnings are unrelated `dead_code` warnings in `xiuxian-daochang` embedding MCP compatibility types.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Embedding MCP Dead-Code Decommission)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/embedding/mod.rs`
- deleted: `packages/rust/crates/xiuxian-daochang/src/embedding/transport_mcp.rs`
- `packages/rust/crates/xiuxian-daochang/src/embedding/types.rs`
- `packages/rust/crates/xiuxian-daochang/src/embedding/client/backend_dispatch.rs`

### Quality changes adopted

- Removed unused MCP embedding transport compatibility path now that runtime behavior is rust-only fallback (`http` / `openai_http` / `litellm_rs`).
- Removed unused deserialize contract type (`McpEmbedResult`) tied to deleted MCP transport path.
- Collapsed duplicate backend dispatch match arms (`OpenAiHttp` + `MistralSdk`) into a single arm to remove pedantic duplication warning.
- Kept embedding behavior aligned with existing tests that already assert rust-only failure semantics when primary transports fail.

### Suppression debt delta (this slice)

- No `#[allow(...)]` added.
- Removed dead-code warning sources by deleting unused code instead of suppressing warnings.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed with no `too_many_lines` warning and no previous `dead_code` warnings in embedding MCP compatibility path.
- `cargo test -p xiuxian-daochang --test embedding_client -- --nocapture`
  - result: 12 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test embedding_client_cache -- --nocapture`
  - result: 2 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test gateway_http -- --nocapture`
  - result: 7 passed, 0 failed.

### Addendum (2026-02-24, Warning-State Refresh)

- Previous quality notes that listed `xiuxian-llm/src/embedding/openai_compat.rs` as remaining `too_many_lines` hotspot are now obsolete after the decomposition slice above.
- Current `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines` run finishes without warning output in touched paths.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Config Agent Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/config/agent.rs`
- `packages/rust/crates/xiuxian-daochang/src/config/agent/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/config/agent/types.rs`
- `packages/rust/crates/xiuxian-daochang/src/config/agent/memory_defaults.rs`
- `packages/rust/crates/xiuxian-daochang/src/config/agent/agent_defaults.rs`

### Quality changes adopted

- Split monolithic agent config module into explicit concerns:
  - `types.rs`: `McpServerEntry`, `MemoryConfig`, `AgentConfig`, `ContextBudgetStrategy` and serialization contracts.
  - `memory_defaults.rs`: memory-domain default value providers + `Default for MemoryConfig`.
  - `agent_defaults.rs`: agent-domain default value providers + `Default for AgentConfig` + `AgentConfig` runtime helpers.
  - `mod.rs`: interface-only re-export surface.
- Kept public export surface stable for callers through `config/mod.rs` (`AgentConfig`, `MemoryConfig`, `ContextBudgetStrategy`, `McpServerEntry`, `LITELLM_DEFAULT_URL`).
- Removed implicit single-file coupling by converting serde defaults to explicit module paths (`memory_defaults::...`, `agent_defaults::...`).

### Structural outcome

- Previous single-file size (`config/agent.rs`): `524` lines.
- New split sizes:
  - `config/agent/types.rs`: `245`
  - `config/agent/memory_defaults.rs`: `173`
  - `config/agent/agent_defaults.rs`: `135`
  - `config/agent/mod.rs`: `8`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- `rg -n "allow\(clippy::wildcard_imports\)|too_many_lines" packages/rust/crates/xiuxian-daochang/src/config/agent -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed in touched paths.
- `cargo test -p xiuxian-daochang --test config_settings -- --nocapture`
  - result: 5 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test config_and_session -- --nocapture`
  - result: 7 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test config_mcp -- --nocapture`
  - result: 5 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Context Budget Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/agent/context_budget.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/context_budget/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/context_budget/types.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/context_budget/classify.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/context_budget/truncate.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/context_budget/selection.rs`

### Quality changes adopted

- Split monolithic context-budget implementation into focused modules:
  - `types.rs`: report/stats data contracts and internal message-index containers.
  - `classify.rs`: message class partitioning + token estimation helpers.
  - `truncate.rs`: token-budget-aware message truncation logic.
  - `selection.rs`: candidate ordering, selection, and packed output materialization.
  - `mod.rs`: public API entrypoints and pruning orchestration.
- Preserved external API surface used by runtime and tests:
  - `prune_messages_for_token_budget`
  - `prune_messages_for_token_budget_with_strategy`
  - `SESSION_SUMMARY_MESSAGE_NAME`
  - `ContextBudgetClassStats`
  - `ContextBudgetReport`
- No lint suppressions added.

### Structural outcome

- Previous single-file size (`agent/context_budget.rs`): `382` lines.
- New split sizes:
  - `agent/context_budget/types.rs`: `121`
  - `agent/context_budget/classify.rs`: `82`
  - `agent/context_budget/truncate.rs`: `33`
  - `agent/context_budget/selection.rs`: `92`
  - `agent/context_budget/mod.rs`: `91`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- `rg -n "allow\(clippy::wildcard_imports\)|too_many_lines" packages/rust/crates/xiuxian-daochang/src/agent/context_budget -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed in touched paths.
- `cargo test -p xiuxian-daochang --test agent_context_budget -- --nocapture`
  - result: 6 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test config_and_session -- --nocapture`
  - result: 7 passed, 0 failed.

### Wider-suite note

- Attempting additional broader integration targets exposed an existing unrelated compile issue in `xiuxian-daochang` LLM backend matching:
  - `packages/rust/crates/xiuxian-daochang/src/llm/client.rs:167`
  - `packages/rust/crates/xiuxian-daochang/src/llm/backend.rs:7`
  - error: non-exhaustive patterns for `LlmBackendKind::MistralSdk`.
- This issue is outside the context-budget module touched in this slice.

### Addendum (2026-02-24, Wider-Suite Revalidation)

- Re-ran broader integration targets after the initial mismatch report.
- Validation evidence:
  - `cargo test -p xiuxian-daochang --test channels_commands -- --nocapture`
    - result: 7 passed, 0 failed.
  - `cargo test -p xiuxian-daochang --test telegram_runtime_config -- --nocapture`
    - result: 4 passed, 0 failed.
- Outcome: the previously observed `LlmBackendKind::MistralSdk` non-exhaustive-match compile error did not reproduce in this revalidation run.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Turn Support Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/agent/turn_support.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_support/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_support/observability.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_support/route_trace.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_support/shortcut_injection.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/shortcut.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/messages.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor/steps/dispatch.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor/steps/fallback.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/graph/executor/steps/invoke.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/persistence/turn_store/gate.rs`

### Quality changes adopted

- Split mixed concerns from legacy `turn_support.rs` into focused modules:
  - `observability.rs`: route/plan/fallback runtime event logging helpers.
  - `route_trace.rs`: route-trace serialization, stream fields, and emit logging.
  - `shortcut_injection.rs`: shortcut injection snapshot build + injection snapshot observability.
- Restored missing shortcut snapshot pipeline after split:
  - `record_injection_snapshot`
  - `build_shortcut_injection_snapshot`
- Converted stateless helper methods from `&self` receivers to associated
  functions (`Self::...`) to remove `clippy::unused_self` warnings without
  adding suppressions.
- Removed stale top-level imports in `agent/mod.rs` created by the file split.

### Structural outcome

- Previous single-file size (`agent/turn_support.rs`): `453` lines.
- New split sizes:
  - `agent/turn_support/observability.rs`: `183`
  - `agent/turn_support/route_trace.rs`: `134`
  - `agent/turn_support/shortcut_injection.rs`: `206`
  - `agent/turn_support/mod.rs`: `3`

### Suppression debt delta (this slice)

- Removed legacy suppressions from split path:
  - `#[allow(clippy::wildcard_imports)]`
  - `#[allow(clippy::unused_self)]`
  - `#[allow(clippy::too_many_lines)]` on route-trace path.
- Validation:
  - `rg -n "allow\\(clippy::wildcard_imports\\)|allow\\(clippy::unused_self\\)|allow\\(clippy::too_many_lines\\)|allow\\(clippy::missing_errors_doc\\)" packages/rust/crates/xiuxian-daochang/src/agent/turn_support -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed (no new warnings in this slice).
- `cargo test -p xiuxian-daochang --test agent_injection -- --nocapture`
  - result: 3 passed, 0 failed, 1 ignored (`requires live valkey server`).
- `cargo test -p xiuxian-daochang --test agent_context_budget -- --nocapture`
  - result: 6 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test config_and_session -- --nocapture`
  - result: 7 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_commands -- --nocapture`
  - result: 7 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test telegram_runtime_config -- --nocapture`
  - result: 4 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Shortcut Execution Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/shortcut.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/shortcut/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/shortcut/workflow_bridge.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/shortcut/crawl.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/mod.rs`

### Quality changes adopted

- Split shortcut runtime concerns into directory modules:
  - `shortcut/mod.rs`: entrypoint orchestration (`handle_shortcuts`).
  - `shortcut/workflow_bridge.rs`: workflow bridge decision/graph execution flow.
  - `shortcut/crawl.rs`: crawl shortcut execution and error handling.
- Replaced wildcard-based symbol coupling with explicit imports in the new
  shortcut modules.
- Kept behavior and API unchanged for turn execution path:
  - workflow bridge still supports graph route + React fallback.
  - crawl shortcut still updates recall feedback/session history/policy hints.

### Structural outcome

- Previous single-file size (`turn_execution/shortcut.rs`): `200` lines.
- New split sizes:
  - `turn_execution/shortcut/mod.rs`: `39`
  - `turn_execution/shortcut/workflow_bridge.rs`: `212`
  - `turn_execution/shortcut/crawl.rs`: `82`

### Suppression debt delta (this slice)

- Removed file-level wildcard suppression in shortcut path:
  - `#[allow(clippy::wildcard_imports)]` from legacy `shortcut.rs`.
- Validation:
  - `rg -n "allow\\(clippy::wildcard_imports\\)|allow\\(clippy::too_many_lines\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/shortcut -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed (no new warnings in this slice).
- `cargo test -p xiuxian-daochang --test agent_injection -- --nocapture`
  - result: 3 passed, 0 failed, 1 ignored (`requires live valkey server`).
- `cargo test -p xiuxian-daochang --test agent_graph_bridge -- --nocapture`
  - result: 3 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_commands -- --nocapture`
  - result: 7 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Embedding Client Runtime Modularization)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/embedding/client/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/embedding/client/init.rs`
- `packages/rust/crates/xiuxian-daochang/src/embedding/client/batch.rs`

### Quality changes adopted

- Split `embedding/client` runtime by responsibility while keeping API stable:
  - `init.rs`: constructor/configuration path (`new*` family, env/runtime tuning, backend selection wiring).
  - `batch.rs`: embedding execution orchestration (`embed_batch_with_model`, chunk dispatch flow, cache writeback).
  - `mod.rs`: interface-focused type/constant definitions (`EmbeddingClient`, `EmbeddingDispatchRuntime`).
- Kept existing dispatch and transport behavior unchanged:
  - backend mode routing (`http`/`openai_http`/`mistral_sdk`/`litellm_rs`) remains in `backend_dispatch.rs`.
  - chunk concurrency, in-flight gate semantics, and cache semantics unchanged.
- Removed stale imports introduced by moving logic out of `mod.rs`.

### Structural outcome

- In-session baseline before this split:
  - `embedding/client/mod.rs`: `450` lines.
- Current split sizes:
  - `embedding/client/mod.rs`: `55`
  - `embedding/client/init.rs`: `176`
  - `embedding/client/batch.rs`: `234`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(clippy::wildcard_imports\\)|allow\\(clippy::too_many_lines\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/embedding/client -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed (no new warnings in touched paths).
- `cargo test -p xiuxian-daochang --test embedding_client -- --nocapture`
  - result: 12 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test embedding_client_cache -- --nocapture`
  - result: 2 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_webhook_embedding -- --nocapture`
  - result: 1 passed, 0 failed.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Memory Stream Consumer Runtime Modularization)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/bootstrap.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/runtime.rs`

### Quality changes adopted

- Split `memory_stream_consumer` responsibilities into focused runtime modules:
  - `bootstrap.rs`: runtime config assembly + startup gating + task spawn.
  - `runtime.rs`: consumer loop/read-retry/reconnect flow and stream-read error classification.
  - `mod.rs`: interface-only wiring and test-surface imports.
- Preserved existing behavior and test API expectations:
  - external call site still uses `memory_stream_consumer::spawn_memory_stream_consumer`.
  - unit-test access to helper symbols (`classify_stream_read_error`, stream/parsing/metrics helpers) retained via `#[cfg(test)]` imports.

### Structural outcome

- In-session baseline before this split:
  - `agent/memory_stream_consumer/mod.rs`: `427` lines.
- Current split sizes:
  - `agent/memory_stream_consumer/mod.rs`: `30`
  - `agent/memory_stream_consumer/bootstrap.rs`: `96`
  - `agent/memory_stream_consumer/runtime.rs`: `329`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(clippy::wildcard_imports\\)|allow\\(clippy::too_many_lines\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed (no new warnings in touched paths).
- `cargo test -p xiuxian-daochang --lib memory_stream_consumer -- --nocapture`
  - result: 14 passed, 0 failed, 4 ignored (`requires live Valkey/Redis on VALKEY_URL`).
- `cargo test -p xiuxian-daochang --test agent_memory_gate_flow -- --nocapture`
  - result: 4 passed, 0 failed, 1 ignored (`requires live valkey server`).

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Telegram Send Text Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/send_text.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/send_text/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/send_text/dispatch.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/send_text/helpers.rs`

### Quality changes adopted

- Split Telegram text delivery flow by concern:
  - `send_text/mod.rs`: entry orchestration (`send_text`, chunk loop, truncation notice).
  - `send_text/dispatch.rs`: parse-mode fallback chain (MarkdownV2 -> HTML -> plain).
  - `send_text/helpers.rs`: chunk preparation and payload-size heuristics.
- Kept runtime behavior intact:
  - attachment marker handling and path-only attachment path unchanged.
  - markdown/html/plain fallback order unchanged.
  - chunk guard and truncation notice semantics unchanged.

### Structural outcome

- In-session baseline before this split:
  - `channels/telegram/channel/send_text.rs`: `407` lines.
- Current split sizes:
  - `channels/telegram/channel/send_text/mod.rs`: `113`
  - `channels/telegram/channel/send_text/dispatch.rs`: `205`
  - `channels/telegram/channel/send_text/helpers.rs`: `105`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(clippy::wildcard_imports\\)|allow\\(clippy::too_many_lines\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/send_text -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed (no new warnings in touched paths).
- `cargo test -p xiuxian-daochang --test channels_telegram -- --nocapture`
  - result: 29 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_telegram_markdown -- --nocapture`
  - result: 16 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test channels_telegram_chunking -- --nocapture`
  - result: 5 passed, 0 failed.

## Process Evidence: 2026-02-24 (Persistent Clippy Policy in `AGENTS.md`)

### Changed file

- `AGENTS.md`

### Policy update

- Added `Rust Clippy Validation Policy` section with persistent rules:
  - mandatory clippy run for touched Rust crates,
  - no suppression-first fixes,
  - `missing_errors_doc` hard rule,
  - narrow-scope exception policy with removal condition,
  - command/outcome evidence requirement.

## Execution Evidence: 2026-02-24 (Slice `xiuxian-daochang` Bounded Session Window Ops Modularization)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/session/bounded_store/window_ops/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/bounded_store/window_ops/read_ops.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/bounded_store/window_ops/write_ops.rs`

### Quality changes adopted

- Split bounded session window operations by responsibility:
  - `read_ops.rs`: `get_recent_messages`, `get_recent_slots`, `get_stats`.
  - `write_ops.rs`: `append_turn`, `replace_window_slots`, `clear`, `drain_oldest_turns`.
  - `mod.rs`: interface-only module wiring and shared conversion helper.
- Preserved all public method signatures and behavior of `BoundedSessionStore`.
- Kept Valkey and in-memory paths functionally equivalent to previous flow.

### Structural outcome

- In-session baseline before this split:
  - `session/bounded_store/window_ops.rs`: `377` lines.
- Current split sizes:
  - `session/bounded_store/window_ops/mod.rs`: `19`
  - `session/bounded_store/window_ops/read_ops.rs`: `164`
  - `session/bounded_store/window_ops/write_ops.rs`: `211`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(clippy::wildcard_imports\\)|allow\\(clippy::too_many_lines\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/session/bounded_store/window_ops -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `cargo check -p xiuxian-daochang`
  - result: passed.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed (no new warnings in touched paths).
- `cargo test -p xiuxian-daochang --test session_summary -- --nocapture`
  - result: 2 passed, 0 failed.
- `cargo test -p xiuxian-daochang --test session_redis -- --nocapture`
  - result: 0 passed, 0 failed, 5 ignored (`requires live valkey server`).
- `cargo test -p xiuxian-daochang --test config_and_session -- --nocapture`
  - result: 7 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Managed Command Classifier Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/channels/managed_commands.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/managed_commands/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/managed_commands/types.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/managed_commands/input_normalization.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/managed_commands/slash_detection.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/managed_commands/control_detection.rs`

### Quality changes adopted

- Converted `managed_commands` from a single file into a directory module with domain-focused boundaries:
  - `types.rs`: slash scope constants and command enums.
  - `slash_detection.rs`: non-privileged slash classification.
  - `control_detection.rs`: privileged control-command classification.
  - `input_normalization.rs`: shared command-prefix/tag normalization.
  - `mod.rs`: interface-only module wiring and re-exports.
- Preserved the external API surface used by runtime and tests:
  - `detect_managed_slash_command`
  - `detect_managed_control_command`
  - `ManagedSlashCommand` / `ManagedControlCommand`
  - `SLASH_SCOPE_*` constants
- Kept behavior intact for command aliases and accepted/rejected shapes (session/window/context scopes, feedback direction aliases, resume/control variants, partition/admin/injection classifiers).

### Structural outcome

- Baseline before split:
  - `channels/managed_commands.rs`: `347` lines.
- Current split sizes:
  - `channels/managed_commands/mod.rs`: `17`
  - `channels/managed_commands/types.rs`: `69`
  - `channels/managed_commands/input_normalization.rs`: `12`
  - `channels/managed_commands/slash_detection.rs`: `135`
  - `channels/managed_commands/control_detection.rs`: `119`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(clippy::wildcard_imports\\)|allow\\(clippy::too_many_lines\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/channels/managed_commands -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-managed-commands cargo check -p xiuxian-daochang --quiet`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-managed-commands cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-managed-commands cargo test -p xiuxian-daochang --test channels_managed_commands -- --nocapture`
  - result: 4 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-managed-commands cargo test -p xiuxian-daochang --test channels_commands -- --nocapture`
  - result: 7 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-managed-commands cargo test -p xiuxian-daochang --test test_support_parsers -- --nocapture`
  - result: 3 passed, 0 failed.

### Environment note

- Default workspace `target/` was locked by another active cargo process in this local environment. Validation used a dedicated `CARGO_TARGET_DIR` to avoid cross-process interference while preserving toolchain parity.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Discord Runtime Dispatch Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch/generation.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch/preview.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch/stop.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch/turn.rs`

### Quality changes adopted

- Replaced single-file `dispatch` runtime flow with directory module boundaries:
  - `mod.rs`: inbound orchestration, managed-command pre-routing, and foreground dispatch glue.
  - `generation.rs`: active-generation guard lifecycle (`begin_generation`/`end_generation` drop semantics).
  - `turn.rs`: foreground turn execution + reply rendering + outbound send logging.
  - `stop.rs`: `/stop` control command handling and stop-marker persistence.
  - `preview.rs`: log-preview truncation helper.
- Preserved runtime API and call sites:
  - `process_discord_message` (test surface)
  - `process_discord_message_with_interrupt` (runtime entrypoint used by `foreground.rs`)
- Kept behavior unchanged for:
  - stop interrupt semantics and persistence marker,
  - managed control/slash detection logs,
  - preemption of in-flight foreground generation,
  - timeout/interrupted/failure reply paths and send logging.

### Structural outcome

- Baseline before split:
  - `channels/discord/runtime/dispatch.rs`: `327` lines.
- Current split sizes:
  - `channels/discord/runtime/dispatch/mod.rs`: `122`
  - `channels/discord/runtime/dispatch/generation.rs`: `37`
  - `channels/discord/runtime/dispatch/preview.rs`: `17`
  - `channels/discord/runtime/dispatch/stop.rs`: `65`
  - `channels/discord/runtime/dispatch/turn.rs`: `105`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(clippy::wildcard_imports\\)|allow\\(clippy::too_many_lines\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/dispatch -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-dispatch cargo check -p xiuxian-daochang --quiet`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-dispatch cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-dispatch cargo test -p xiuxian-daochang --lib "channels::discord::runtime::tests::" -- --nocapture`
  - result: 18 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-dispatch cargo test -p xiuxian-daochang --test channels_discord -- --nocapture`
  - result: 9 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-dispatch cargo test -p xiuxian-daochang --test channels_discord_ingress -- --nocapture`
  - result: 5 passed, 0 failed.

### Environment note

- There is no standalone integration target named `discord_runtime`; runtime tests are library tests under `channels::discord::runtime::tests::*`, so validation used filtered `--lib` execution for that suite.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Telegram Session Gate Valkey Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/channels/telegram/session_gate/valkey.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/session_gate/valkey/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/session_gate/valkey/acquire.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/session_gate/valkey/commands.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/session_gate/valkey/guard.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/session_gate/valkey/token.rs`

### Quality changes adopted

- Converted session-gate Valkey backend into domain modules:
  - `mod.rs`: interface-only type definitions and module wiring.
  - `acquire.rs`: backend construction, lease acquire loop, renew-task orchestration.
  - `commands.rs`: Valkey command execution path (SET NX PX, EVAL renew/release, reconnect retry logic).
  - `guard.rs`: distributed lease drop/release behavior.
  - `token.rs`: lease owner token generation.
- Preserved external usage surface from `session_gate/core.rs` and `session_gate/types.rs`:
  - `ValkeySessionGateBackend::new`
  - `ValkeySessionGateBackend::acquire_lease`
  - `DistributedLeaseGuard`
- Kept behavior intact for:
  - acquire-timeout semantics,
  - lease renewal cadence and failure handling,
  - retry-on-command-failure reconnect path,
  - drop-time best-effort lease release.

### Structural outcome

- Baseline before split:
  - `channels/telegram/session_gate/valkey.rs`: `316` lines.
- Current split sizes:
  - `channels/telegram/session_gate/valkey/mod.rs`: `30`
  - `channels/telegram/session_gate/valkey/acquire.rs`: `127`
  - `channels/telegram/session_gate/valkey/commands.rs`: `137`
  - `channels/telegram/session_gate/valkey/guard.rs`: `40`
  - `channels/telegram/session_gate/valkey/token.rs`: `13`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(clippy::wildcard_imports\\)|allow\\(clippy::too_many_lines\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/channels/telegram/session_gate/valkey -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-session-gate cargo check -p xiuxian-daochang --quiet`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-session-gate cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-session-gate cargo test -p xiuxian-daochang --test telegram_session_gate -- --nocapture`
  - result: 4 passed, 0 failed, 2 ignored (`requires live valkey server`).
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-session-gate cargo test -p xiuxian-daochang --test telegram_runtime_config -- --nocapture`
  - result: 4 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Discord ACL Config Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/channels/discord/acl_config.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/acl_config/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/acl_config/role_aliases.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/acl_config/principals.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/acl_config/control_rules.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/acl_config/slash.rs`

### Quality changes adopted

- Converted Discord ACL override builder into focused directory modules:
  - `mod.rs`: public `DiscordAclOverrides` and top-level assembly (`build_discord_acl_overrides`).
  - `role_aliases.rs`: role-id parsing, alias normalization, role principal resolution.
  - `principals.rs`: principal collection for users/roles and allow-list extraction.
  - `control_rules.rs`: command-scoped control-rule parsing and typed rule construction.
  - `slash.rs`: slash ACL extraction logic (all scopes).
- Preserved public API surface used by callers/tests:
  - `DiscordAclOverrides`
  - `build_discord_acl_overrides`
- Replaced tuple-based `slash_overrides` return with a named `SlashOverrides` struct and removed the previous `#[allow(clippy::type_complexity)]` path.
- Addressed clippy naming feedback in `SlashOverrides` without adding any suppression attributes.

### Structural outcome

- Baseline before split:
  - `channels/discord/acl_config.rs`: `316` lines.
- Current split sizes:
  - `channels/discord/acl_config/mod.rs`: `87`
  - `channels/discord/acl_config/role_aliases.rs`: `88`
  - `channels/discord/acl_config/principals.rs`: `56`
  - `channels/discord/acl_config/control_rules.rs`: `48`
  - `channels/discord/acl_config/slash.rs`: `68`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Removed one suppression path from previous monolith:
  - old `#[allow(clippy::type_complexity)]` on `slash_overrides` is no longer needed.
- Validation:
  - `rg -n "allow\\(clippy::wildcard_imports\\)|allow\\(clippy::too_many_lines\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)|allow\\(clippy::type_complexity\\)" packages/rust/crates/xiuxian-daochang/src/channels/discord/acl_config -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-acl-v3 cargo check -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-acl-v3 cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-acl-v3 cargo test -p xiuxian-daochang --test discord_acl_overrides -- --nocapture`
  - result: 3 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-acl-v3 cargo test -p xiuxian-daochang --test config_settings -- --nocapture`
  - result: 5 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-acl-v3 cargo test -p xiuxian-daochang --test channels_discord -- --nocapture`
  - result: 9 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Memory Recall Snapshot State Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall_state.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall_state/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall_state/types.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall_state/storage.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall_state/agent_ops.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/memory_recall.rs`

### Quality changes adopted

- Replaced monolithic memory recall state module with focused directory modules:
  - `types.rs`: recall snapshot decision/runtime model, persisted model, source normalization.
  - `storage.rs`: session-id mapping + snapshot payload encode/decode.
  - `agent_ops.rs`: `Agent` persistence/inspection IO behavior.
  - `mod.rs`: interface-only wiring and test-only exports.
- Preserved external behavior:
  - `Agent::record_memory_recall_snapshot`
  - `Agent::inspect_memory_recall_snapshot`
  - stream event payload shape for `recall_snapshot_updated`.
- Removed suppression-based API shape:
  - replaced `SessionMemoryRecallSnapshot::from_plan` (14+ args) with
    `SessionMemoryRecallSnapshotInput` struct argument to eliminate
    `#[allow(clippy::too_many_arguments)]` path while keeping call semantics unchanged.

### Structural outcome

- Baseline before split:
  - `agent/memory_recall_state.rs`: `340` lines.
- Current split sizes:
  - `agent/memory_recall_state/mod.rs`: `19`
  - `agent/memory_recall_state/types.rs`: `205`
  - `agent/memory_recall_state/storage.rs`: `35`
  - `agent/memory_recall_state/agent_ops.rs`: `107`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Removed one suppression path from previous monolith:
  - old `#[allow(clippy::too_many_arguments)]` on `SessionMemoryRecallSnapshot::from_plan` is no longer needed.
- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)|allow\\(clippy::type_complexity\\)|allow\\(clippy::too_many_lines\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/agent/memory_recall_state packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/memory_recall.rs -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-recall-state cargo check -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-recall-state cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-recall-state cargo test -p xiuxian-daochang --lib "memory_recall_state" -- --nocapture`
  - result: 4 passed, 0 failed, 2 ignored (`requires live valkey server`).
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-recall-state cargo test -p xiuxian-daochang --lib "session_memory" -- --nocapture`
  - result: 11 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` React Loop Memory Recall Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/memory_recall.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/memory_recall/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/memory_recall/plan.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/memory_recall/execution.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/memory_recall/observability.rs`

### Quality changes adopted

- Replaced the monolithic React-loop memory recall module with focused directory modules:
  - `plan.rs`: recall planning context construction and feedback-adjusted plan selection.
  - `execution.rs`: embedding success path orchestration and recall-credit candidate extraction.
  - `observability.rs`: injected/skipped/failure logging, metrics, and snapshot persistence.
  - `mod.rs`: interface-only module wiring.
- Preserved behavior and call flow:
  - `Agent::apply_memory_recall_if_enabled`
  - memory recall embedding failure fallback semantics
  - injected/skipped decision snapshot payload shape
- Fixed module-boundary visibility explicitly instead of using suppressions:
  - `pub(in super::super)` for the parent react-loop API entrypoint.
  - `pub(super)` for cross-submodule helper methods used within `memory_recall/`.

### Structural outcome

- Baseline before split:
  - `agent/turn_execution/react_loop/memory_recall.rs`: `358` lines.
- Current split sizes:
  - `agent/turn_execution/react_loop/memory_recall/mod.rs`: `3`
  - `agent/turn_execution/react_loop/memory_recall/plan.rs`: `61`
  - `agent/turn_execution/react_loop/memory_recall/execution.rs`: `151`
  - `agent/turn_execution/react_loop/memory_recall/observability.rs`: `159`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(clippy::too_many_lines\\)|allow\\(clippy::too_many_arguments\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::type_complexity\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/memory_recall -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-recall-modular-2 cargo check -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-recall-modular-2 cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-recall-modular-2 cargo test -p xiuxian-daochang --lib "memory_recall_state" -- --nocapture`
  - result: 4 passed, 0 failed, 2 ignored (`requires live valkey server`).
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-recall-modular-2 cargo test -p xiuxian-daochang --lib "session_memory" -- --nocapture`
  - result: 11 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Injection Assembler Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler/ordering.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler/budget.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler/role_mix.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler/util.rs`

### Quality changes adopted

- Refactored injection snapshot assembly into domain-focused directory modules:
  - `mod.rs`: top-level snapshot assembly orchestration (`assemble_snapshot`).
  - `ordering.rs`: ordering strategy and category ranking.
  - `budget.rs`: char-budget trimming, truncation, and anchor-priority ordering.
  - `role_mix.rs`: role candidate selection and role-mix profile construction.
  - `util.rs`: deterministic dedup helper.
- Preserved behavior:
  - anchor-preserving eviction policy under block limit pressure.
  - role-mix profile selection across `single/classified/hybrid` modes.
  - dropped/truncated block-id normalization and dedup semantics.
- No suppression shortcuts introduced; kept logic explicit and test-backed.

### Structural outcome

- Baseline before split:
  - `agent/injection/assembler.rs`: `307` lines.
- Current split sizes:
  - `agent/injection/assembler/mod.rs`: `69`
  - `agent/injection/assembler/ordering.rs`: `31`
  - `agent/injection/assembler/budget.rs`: `67`
  - `agent/injection/assembler/role_mix.rs`: `143`
  - `agent/injection/assembler/util.rs`: `9`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(clippy::too_many_lines\\)|allow\\(clippy::too_many_arguments\\)|allow\\(clippy::type_complexity\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/agent/injection/assembler -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-injection-assembler-modular cargo check -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-injection-assembler-modular cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-injection-assembler-modular cargo test -p xiuxian-daochang --lib "injection::tests" -- --nocapture`
  - result: 4 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Memory Stream Consumer Runtime Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/runtime.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/runtime/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/runtime/loop_control.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/runtime/read_error.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/runtime/read_error_logging.rs`

### Quality changes adopted

- Replaced monolithic runtime loop module with focused directory modules:
  - `mod.rs`: runtime interface wiring (`run_consumer_loop`, test-only `classify_stream_read_error`).
  - `loop_control.rs`: connection lifecycle, pending/backfill flow, and reconnect loop control.
  - `read_error.rs`: stream read error classification via error-chain normalization.
  - `read_error_logging.rs`: centralized structured logging for recovery/reconnect paths.
- Preserved behavior:
  - consumer group recovery behavior on `NOGROUP`.
  - reconnect/backoff sequencing and idle polling transitions.
  - stream read error classification semantics validated by existing tests.
- Kept test ergonomics stable:
  - `classify_stream_read_error` remains accessible to existing `memory_stream_consumer` test imports via `runtime` module, but only under `#[cfg(test)]` to avoid non-test warning noise.

### Structural outcome

- Baseline before split:
  - `agent/memory_stream_consumer/runtime.rs`: `329` lines.
- Current split sizes:
  - `agent/memory_stream_consumer/runtime/mod.rs`: `7`
  - `agent/memory_stream_consumer/runtime/loop_control.rs`: `198`
  - `agent/memory_stream_consumer/runtime/read_error.rs`: `40`
  - `agent/memory_stream_consumer/runtime/read_error_logging.rs`: `116`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(clippy::too_many_lines\\)|allow\\(clippy::too_many_arguments\\)|allow\\(clippy::type_complexity\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/runtime -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-stream-runtime-split cargo check -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-stream-runtime-split cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-stream-runtime-split cargo test -p xiuxian-daochang --lib "memory_stream_consumer" -- --nocapture`
  - result: 14 passed, 0 failed, 4 ignored (`requires running Valkey/Redis on VALKEY_URL`).

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Session Redis Backend Core Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/mod.rs` (monolithic version)
- `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/mod.rs` (interface-only replacement)
- `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/config.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/backend.rs`
- `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/executor.rs`
- updated imports for helper path stability:
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/summary_store.rs`
  - `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/window_store.rs`

### Quality changes adopted

- Split core redis backend responsibilities into focused modules:
  - `config.rs`: runtime env/settings resolution and snapshot/config types.
  - `backend.rs`: backend shape, constructor/getters, key builders, and small numeric helpers.
  - `executor.rs`: connection lifecycle and retry-aware command/pipeline execution.
  - `mod.rs`: module wiring and public surface only.
- Preserved external API used by session store/bounded store:
  - `RedisSessionBackend::{from_env,new,new_from_parts,key_prefix,ttl_secs,runtime_snapshot}`
  - `RedisSessionRuntimeSnapshot` re-export path under `session` module.
- Kept behavior unchanged for retry logging, reconnect behavior, key naming, and TTL handling.

### Structural outcome

- Baseline before split:
  - `session/redis_backend/mod.rs`: `307` lines.
- Current split sizes:
  - `session/redis_backend/mod.rs`: `13`
  - `session/redis_backend/config.rs`: `69`
  - `session/redis_backend/backend.rs`: `110`
  - `session/redis_backend/executor.rs`: `127`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(clippy::too_many_lines\\)|allow\\(clippy::too_many_arguments\\)|allow\\(clippy::type_complexity\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/session/redis_backend -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-session-redis-split cargo check -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-session-redis-split cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-session-redis-split cargo test -p xiuxian-daochang --test session_redis -- --nocapture`
  - result: 0 passed, 0 failed, 5 ignored (`requires live valkey server`).
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-session-redis-split cargo test -p xiuxian-daochang --test session_redis -- --ignored --nocapture`
  - result: 5 passed, 0 failed (environment without `VALKEY_URL` prints `skip: set VALKEY_URL` and returns `Ok(())`).

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Telegram Session Command Parser Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/channels/telegram/commands/session.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/commands/session/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/commands/session/injection.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/commands/session/admin.rs`
- updated module path wiring:
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/commands.rs`

### Quality changes adopted

- Converted telegram session command parser from a single file into focused directory modules:
  - `mod.rs`: public command surface, shared aliases/types, partition mode mapping, and lightweight delegations.
  - `injection.rs`: `/session|window|context inject*` parse flow.
  - `admin.rs`: `/session|window|context admin*` parse flow and user-id normalization.
- Preserved all public parser entrypoints and data types consumed by runtime handlers/tests:
  - `parse_session_injection_command`
  - `parse_session_admin_command`
  - `parse_session_partition_command`
  - `parse_session_context_{status,budget,memory}_command`
  - `parse_session_feedback_command`
  - `parse_resume_context_command`, `is_reset_context_command`, `is_stop_command`
  - `SessionPartitionMode`, `SessionInjectionAction`, `SessionAdminAction`
- Maintained compatibility with existing command aliases and output format handling.

### Structural outcome

- Baseline before split:
  - `channels/telegram/commands/session.rs`: `307` lines.
- Current split sizes:
  - `channels/telegram/commands/session/mod.rs`: `131`
  - `channels/telegram/commands/session/injection.rs`: `88`
  - `channels/telegram/commands/session/admin.rs`: `96`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(clippy::too_many_lines\\)|allow\\(clippy::too_many_arguments\\)|allow\\(clippy::type_complexity\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/channels/telegram/commands/session -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-telegram-session-split cargo check -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-telegram-session-split cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-telegram-session-split cargo test -p xiuxian-daochang --test channels_commands -- --nocapture`
  - result: 7 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-telegram-session-split cargo test -p xiuxian-daochang --test test_support_parsers -- --nocapture`
  - result: 3 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Discord Managed Parser Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/managed/parsing.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/managed/parsing/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/managed/parsing/admin.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/managed/parsing/injection.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/managed/parsing/partition.rs`

### Quality changes adopted

- Replaced monolithic managed command parser with domain-focused directory modules:
  - `mod.rs`: public parse surface (`parse_managed_command`), command/type definitions, and parser routing order.
  - `admin.rs`: session-admin command parsing and user-id token normalization.
  - `injection.rs`: session-injection command parsing.
  - `partition.rs`: discord partition mode mapping and partition command parsing.
- Preserved behavior:
  - parser precedence order in `parse_managed_command`.
  - session/admin/injection/partition alias compatibility.
  - `ManagedCommand` payload shapes consumed by command dispatch.
- Removed a small duplicate local assignment (`let command = tokens[0];` repeated) while preserving semantics.

### Structural outcome

- Baseline before split:
  - `channels/discord/runtime/managed/parsing.rs`: `305` lines.
- Current split sizes:
  - `channels/discord/runtime/managed/parsing/mod.rs`: `118`
  - `channels/discord/runtime/managed/parsing/admin.rs`: `93`
  - `channels/discord/runtime/managed/parsing/injection.rs`: `86`
  - `channels/discord/runtime/managed/parsing/partition.rs`: `26`

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(clippy::too_many_lines\\)|allow\\(clippy::too_many_arguments\\)|allow\\(clippy::type_complexity\\)|allow\\(clippy::missing_errors_doc\\)|allow\\(clippy::unused_self\\)" packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/managed/parsing -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-managed-parse-split cargo check -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-managed-parse-split cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-managed-parse-split cargo test -p xiuxian-daochang --test channels_discord -- --nocapture`
  - result: 9 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Agent Memory Recall Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall/planning.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall/ranking.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall/context.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_recall/token_estimation.rs`

### Quality changes adopted

- Replaced a monolithic memory recall module with focused directory modules:
  - `mod.rs`: interface-only exports and test-only helper re-export.
  - `planning.rs`: budget-pressure-aware recall planning logic.
  - `ranking.rs`: recall ranking and threshold filtering logic.
  - `context.rs`: memory context message construction and trimming.
  - `token_estimation.rs`: model-token estimation helper.
- Preserved API surface used by `agent` runtime and tests:
  - `MEMORY_RECALL_MESSAGE_NAME`
  - `MemoryRecallInput`
  - `plan_memory_recall`
  - `estimate_messages_tokens`
  - `filter_recalled_episodes`
  - `build_memory_context_message`
- Kept `filter_recalled_episodes_at` exported for tests only (`#[cfg(test)]`) to avoid non-test symbol noise.

### Structural outcome

- Baseline before split:
  - `agent/memory_recall.rs`: `272` lines.
- Current split sizes:
  - `agent/memory_recall/mod.rs`: `44`
  - `agent/memory_recall/planning.rs`: `67`
  - `agent/memory_recall/ranking.rs`: `82`
  - `agent/memory_recall/context.rs`: `63`
  - `agent/memory_recall/token_estimation.rs`: `31`
  - total: `287` lines.

### Suppression debt delta (this slice)

- No new `#[allow(...)]` debt introduced by this split.
- Validation:
  - `rg -n "allow\\(clippy::|allow\\(unused|allow\\(dead_code" packages/rust/crates/xiuxian-daochang/src/agent/memory_recall -g '*.rs'`
    - result: 2 matches, both `#[allow(clippy::cast_precision_loss)]` in `planning.rs` and `ranking.rs`.
  - `git show HEAD:packages/rust/crates/xiuxian-daochang/src/agent/memory_recall.rs | rg -n "allow\\(clippy::cast_precision_loss\\)"`
    - result: 2 matches in baseline monolithic file.
  - conclusion: suppression count is unchanged; no new lint bypass added.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-recall-split cargo check -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-recall-split cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-recall-split cargo test -p xiuxian-daochang --lib "memory_recall" -- --nocapture`
  - result: 27 passed, 0 failed, 4 ignored (`requires live valkey server`).
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-recall-split cargo test -p xiuxian-daochang --lib "memory_recall_feedback" -- --nocapture`
  - result: 13 passed, 0 failed, 2 ignored (`requires live valkey server`).

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Runtime Agent Factory `mod.rs` Responsibility Split)

### Changed files

- `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/mcp.rs`
- `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/session.rs`
- `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/shared.rs`
- `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/types.rs`
- import/path rewiring to consume split modules:
  - `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/inference.rs`
  - `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/logging.rs`
  - `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/memory.rs`
  - `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/memory/embedding.rs`
  - `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/memory/env_overrides.rs`
  - `packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory/memory/runtime.rs`

### Quality changes adopted

- Reduced `runtime_agent_factory/mod.rs` to orchestration-only responsibilities:
  - module wiring,
  - test-only exports/imports for existing path-based tests,
  - `build_agent` assembly flow.
- Extracted domain responsibilities into dedicated modules:
  - `mcp.rs`: MCP server loading and MCP runtime option resolution.
  - `session.rs`: context-budget parsing and session runtime option resolution.
  - `shared.rs`: local env helper (`non_empty_env`) and range guard helper (`normalize_unit_f32`).
  - `types.rs`: runtime option/value-object definitions shared across sub-modules.
- Preserved behavior and API paths:
  - `build_agent` call graph unchanged.
  - inference/memory/logging sub-modules continue to consume the same runtime option fields.
  - existing `runtime_agent_factory` unit tests kept under the same `#[path]` test wiring.

### Structural outcome

- Baseline before split (derived from `git diff --numstat` against previous file state):
  - `runtime_agent_factory/mod.rs`: `272` lines.
- Current split sizes:
  - `runtime_agent_factory/mod.rs`: `73`
  - `runtime_agent_factory/mcp.rs`: `68`
  - `runtime_agent_factory/session.rs`: `103`
  - `runtime_agent_factory/shared.rs`: `18`
  - `runtime_agent_factory/types.rs`: `32`
  - total extracted surface: `294` lines.

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-daochang/src/runtime_agent_factory -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-runtime-agent-factory-split cargo check -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-runtime-agent-factory-split cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-runtime-agent-factory-split cargo test -p xiuxian-daochang --bin xiuxian-daochang "runtime_agent_factory::tests" -- --nocapture`
  - result: 18 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Session Context Window Ops Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/agent/session_context/window_ops.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/session_context/window_ops/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/session_context/window_ops/backup_lifecycle.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/session_context/window_ops/inspection.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/session_context/window_ops/append.rs`

### Quality changes adopted

- Replaced a mixed-responsibility `window_ops.rs` with domain-focused directory modules:
  - `backup_lifecycle.rs`: reset/resume/drop/peek lifecycle over snapshot backup state.
  - `inspection.rs`: bounded/unbounded context window inspection counters.
  - `append.rs`: session append helper methods (`append_turn_*`).
  - `mod.rs`: module wiring only.
- Preserved all existing public method signatures on `Agent`:
  - `reset_context_window`
  - `resume_context_window`
  - `drop_context_window_backup`
  - `peek_context_window_backup`
  - `inspect_context_window`
  - `append_turn_for_session`
  - `append_turn_with_tool_count_for_session`
- Kept behavior unchanged for:
  - bounded atomic snapshot path precedence,
  - unbounded fallback backup/restore flow,
  - observability event names and payload fields.

### Structural outcome

- Baseline before split:
  - `agent/session_context/window_ops.rs`: `296` lines.
- Current split sizes:
  - `agent/session_context/window_ops/mod.rs`: `3`
  - `agent/session_context/window_ops/backup_lifecycle.rs`: `217`
  - `agent/session_context/window_ops/inspection.rs`: `60`
  - `agent/session_context/window_ops/append.rs`: `31`
  - total: `311` lines.

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-daochang/src/agent/session_context/window_ops -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-session-context-window-split cargo check -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-session-context-window-split cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-session-context-window-split cargo test -p xiuxian-daochang --test agent_session_context -- --nocapture`
  - result: 7 passed, 0 failed, 1 ignored (`requires live valkey server`).

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Telegram Parsing Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/parsing.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/parsing/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/parsing/types.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/parsing/acl.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/parsing/group_policy.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/parsing/update_message.rs`
- prerequisite build unblock for test execution:
  - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/runtime/run_polling/channel_listener.rs`

### Quality changes adopted

- Replaced monolithic telegram parsing module with focused directory modules:
  - `mod.rs`: parse entrypoint orchestration only.
  - `types.rs`: parsed update data model and derived identities.
  - `acl.rs`: user/group ACL checks, allowlist normalization, unauthorized sender logging.
  - `group_policy.rs`: group policy evaluation and mention/reply/entity trigger checks.
  - `update_message.rs`: update JSON extraction and `ChannelMessage` construction.
- Removed the prior `#[allow(clippy::unused_self)]` compatibility pattern by moving
  `is_identity_in_allowlist` to a pure helper function (no receiver needed).
- Preserved behavior:
  - same ACL gate semantics and warning log shape for unauthorized senders.
  - same group policy resolution order and mention trigger rules.
  - same message/session-key/id construction format.

### Structural outcome

- Baseline before split:
  - `channels/telegram/channel/parsing.rs`: `217` lines.
- Current split sizes:
  - `channels/telegram/channel/parsing/mod.rs`: `36`
  - `channels/telegram/channel/parsing/types.rs`: `27`
  - `channels/telegram/channel/parsing/acl.rs`: `60`
  - `channels/telegram/channel/parsing/group_policy.rs`: `102`
  - `channels/telegram/channel/parsing/update_message.rs`: `96`
  - total: `321` lines.

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/parsing -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-telegram-parsing-split cargo check -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-telegram-parsing-split cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed (`xiuxian-daochang` clean for this slice; unrelated existing warnings surfaced in `xiuxian-llm`).
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-telegram-parsing-split cargo test -p xiuxian-daochang --test channels_telegram -- --nocapture`
  - result: 29 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-telegram-parsing-split cargo test -p xiuxian-daochang --test channels_telegram_group_policy -- --nocapture`
  - result: 20 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-telegram-parsing-split cargo test -p xiuxian-daochang --test channels_telegram_slash_authorization -- --nocapture`
  - result: 5 passed, 0 failed.

### Prerequisite build unblock note

- While running telegram integration tests, compilation failed with an existing move error in
  polling runtime startup (`E0382`: moved `tx` then reused).
- Minimal non-behavioral fix applied:
  - clone sender before spawn (`listener_tx = tx.clone()`), pass cloned sender into `listen`,
    keep original sender for return tuple.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Memory Stream Consumer Processing Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/processing.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/processing/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/processing/ack_metrics.rs`
- `packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/processing/promotion.rs`

### Quality changes adopted

- Replaced mixed processing module with focused sub-modules:
  - `mod.rs`: event loop orchestration and failure logging path.
  - `ack_metrics.rs`: redis script path for `XACK` + metrics update.
  - `promotion.rs`: promotion dedup + ingest-queue path and payload serialization.
- Preserved caller/test surface:
  - `process_stream_events` remains in `processing` module.
  - `ack_and_record_metrics` and `queue_promoted_candidate` are still re-exported from
    `memory_stream_consumer` for existing test imports.
- Constrained visibility explicitly with `pub(in super::super)` to keep exposure scoped to
  `memory_stream_consumer` module boundary.

### Structural outcome

- Baseline before split:
  - `agent/memory_stream_consumer/processing.rs`: `292` lines (`git show :<path>` from index).
- Current split sizes:
  - `agent/memory_stream_consumer/processing/mod.rs`: `133`
  - `agent/memory_stream_consumer/processing/ack_metrics.rs`: `91`
  - `agent/memory_stream_consumer/processing/promotion.rs`: `80`
  - total: `304` lines.

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-daochang/src/agent/memory_stream_consumer/processing -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-stream-processing-split cargo check -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-stream-processing-split cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed (`xiuxian-daochang` compiled successfully for this slice; unrelated existing warnings surfaced in `xiuxian-llm` and other pre-existing files).
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-memory-stream-processing-split cargo test -p xiuxian-daochang --lib "memory_stream_consumer" -- --nocapture`
  - result: 14 passed, 0 failed, 4 ignored (`requires running Valkey/Redis on VALKEY_URL`).

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Discord Gateway Runtime Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway/event_handler.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway/loop_control.rs`

### Quality changes adopted

- Split monolithic gateway runtime into focused modules:
  - `mod.rs`: runtime assembly/wiring, snapshot tick setup, startup banner, shutdown path.
  - `event_handler.rs`: serenity gateway message intake and inbound queue backpressure logging.
  - `loop_control.rs`: select-loop coordination (inbound, completions, foreground join, telemetry tick, signal, gateway task).
- Preserved behavior:
  - same `run_discord_gateway` public API and startup/shutdown semantics.
  - same gateway intents, message parse path, and inbound queue backpressure warning.
  - same periodic runtime snapshot emission and Ctrl+C handling.
- Addressed pedantic style in loop selector by using `() = async { ... }` in snapshot branch.

### Structural outcome

- Baseline before split:
  - `channels/discord/runtime/gateway.rs`: `173` lines.
- Current split sizes:
  - `channels/discord/runtime/gateway/mod.rs`: `131`
  - `channels/discord/runtime/gateway/event_handler.rs`: `50`
  - `channels/discord/runtime/gateway/loop_control.rs`: `65`
  - total: `246` lines.

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/gateway -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-gateway-split cargo check -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-gateway-split cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed (`xiuxian-daochang` clean for this slice; unrelated existing warnings surfaced in `xiuxian-llm`).
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-gateway-split cargo test -p xiuxian-daochang --test channels_discord -- --nocapture`
  - result: 9 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-gateway-split cargo test -p xiuxian-daochang --test channels_discord_parsing -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-gateway-split cargo test -p xiuxian-daochang --test channels_discord_ingress -- --nocapture`
  - result: 5 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-daochang` Discord Ingress Runtime Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run/mod.rs`
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run/loop_control.rs`

### Quality changes adopted

- Split ingress runtime into focused modules:
  - `mod.rs`: request/config deconstruction, ingress app setup, snapshot ticker setup, server spawn, shutdown trigger.
  - `loop_control.rs`: select-loop orchestration for inbound messages, completions, foreground joins, snapshot emission, signal, and server task outcomes.
- Preserved external API and behavior:
  - `DiscordIngressRunRequest` and `run_discord_ingress` public signatures unchanged.
  - inbound queue, foreground runtime, and periodic telemetry flow preserved.
  - graceful shutdown signal path (`oneshot`) preserved.
- Updated select pattern to `() = async { ... }` for clearer unit branch semantics.

### Structural outcome

- Baseline before split:
  - `channels/discord/runtime/run.rs`: `140` lines.
- Current split sizes:
  - `channels/discord/runtime/run/mod.rs`: `153`
  - `channels/discord/runtime/run/loop_control.rs`: `60`
  - total: `213` lines.

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-daochang/src/channels/discord/runtime/run -g '*.rs'`
    - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-daochang`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-run-split cargo check -p xiuxian-daochang`
  - result: passed (with existing unrelated warnings outside this slice).
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-run-split cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: passed (`xiuxian-daochang` compiled successfully for this slice; unrelated existing warnings surfaced in `xiuxian-llm` and pre-existing `agent/admission.rs`).
- `CARGO_TARGET_DIR=.cache/target-xiuxian-daochang-discord-run-split cargo test -p xiuxian-daochang --test channels_discord_ingress -- --nocapture`
  - result: 5 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-vector` Search Runtime Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-vector/src/search/search_impl.rs`
- `packages/rust/crates/xiuxian-vector/src/search/search_impl/mod.rs`
- `packages/rust/crates/xiuxian-vector/src/search/search_impl/rows.rs`
- `packages/rust/crates/xiuxian-vector/src/search/search_impl/confidence.rs`
- `packages/rust/crates/xiuxian-vector/src/search/search_impl/filter.rs`
- `packages/rust/crates/xiuxian-vector/src/search/search_impl/ipc.rs`
- `packages/rust/crates/xiuxian-vector/src/search/search_impl/tests.rs`
- `packages/rust/crates/xiuxian-vector/src/lib.rs`
- `packages/rust/crates/xiuxian-vector/src/skill/ops_impl/registry.rs`

### Quality changes adopted

- Split monolithic `search_impl` into focused directory modules:
  - `mod.rs`: external search API surface (`impl VectorStore`), hybrid/FTS entrypoints, filter-plan orchestration.
  - `rows.rs`: row decode and metadata resolution for ANN/FTS batches.
  - `confidence.rs`: confidence calibration, ranking reason assembly, schema digest normalization.
  - `filter.rs`: JSON-filter to Lance WHERE translator (`json_to_lance_where`).
  - `ipc.rs`: Arrow IPC encoding contracts for vector/tool search payloads.
  - `tests.rs`: moved inline tests out of complex runtime file.
- Preserved external behavior and API:
  - `VectorStore` method signatures unchanged.
  - `json_to_lance_where` kept as public crate export through `lib.rs` re-export.
  - IPC schema contract and projection behavior preserved.
- Surfaced and fixed one hidden compile dependency:
  - `skill/ops_impl/registry.rs` now imports `arrow::array::Array` explicitly.
  - This removed accidental reliance on top-level `use` leakage from the previous giant include file.

### Structural outcome

- Baseline before split:
  - `search/search_impl.rs`: `1639` lines.
- Current split sizes:
  - `search/search_impl/mod.rs`: `479`
  - `search/search_impl/rows.rs`: `405`
  - `search/search_impl/confidence.rs`: `149`
  - `search/search_impl/filter.rs`: `96`
  - `search/search_impl/ipc.rs`: `416`
  - `search/search_impl/tests.rs`: `134`
  - total: `1679` lines.

### Suppression debt delta (this slice)

- No new `#[allow(...)]` added.
- Validation:
  - `rg -n "allow\(" packages/rust/crates/xiuxian-vector/src/search/search_impl -g '*.rs'`
  - result:
    - `rows.rs:#[allow(clippy::cast_possible_truncation)]` (existing behavior-preserving cast site)
    - `mod.rs:#[allow(clippy::too_many_arguments)]` (existing API signature)
    - `mod.rs:#[allow(clippy::cast_possible_truncation)]` (existing score cast sites)

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-vector-search-split cargo check -p xiuxian-vector`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-vector-search-split cargo clippy -p xiuxian-vector -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-vector-search-split-test-a cargo test -p xiuxian-vector --lib test_search_results_to_ipc_projection -- --nocapture`
  - result: 1 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-vector-search-split-test-b cargo test -p xiuxian-vector --lib test_tool_search_results_to_ipc_one_row -- --nocapture`
  - result: 1 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-vector-search-split-test-c cargo test -p xiuxian-vector --test filter_expr -- --nocapture`
  - result: 14 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Tools Scanner Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/tools.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/parse.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/schema.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/tests.rs`

### Quality changes adopted

- Converted monolithic `skills/tools.rs` into a directory module with clear responsibility boundaries:
  - `mod.rs`: public surface (`ToolsScanner`), module wiring, default implementation.
  - `scan.rs`: filesystem/virtual-path traversal and file filtering.
  - `parse.rs`: script/content parsing and tool record assembly.
  - `schema.rs`: input-schema generation from parsed parameters.
  - `tests.rs`: moved unit tests out of the implementation file.
- Removed duplicated parsing flow between `parse_script` and `parse_content` by consolidating into shared parse helpers while preserving behavior.
- Preserved external API signatures for:
  - `scan_scripts`
  - `scan_skill_scripts`
  - `scan_with_structure`
  - `parse_content`
  - `scan_paths`

### Structural outcome

- Baseline before split:
  - `skills/tools.rs`: `1251` lines.
- Current split sizes:
  - `skills/tools/mod.rs`: `29`
  - `skills/tools/scan.rs`: `270`
  - `skills/tools/parse.rs`: `215`
  - `skills/tools/schema.rs`: `62`
  - `skills/tools/tests.rs`: `539`
  - total: `1115` lines.

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/tools -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-split-unit cargo test -p xiuxian-skills --lib test_parse_content_single_tool -- --nocapture`
  - result: 1 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-split-it cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Metadata Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/metadata.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/core.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/index.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/structure.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/records.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/sync.rs`

### Quality changes adopted

- Converted monolithic `skills/metadata.rs` into a directory module with responsibility-based split:
  - `mod.rs`: module wiring + stable re-export surface.
  - `core.rs`: core metadata types (`SkillMetadata`, `ToolRecord`, `ReferencePath`, annotations/decorator args).
  - `index.rs`: index-facing records (`SkillIndexEntry`, `IndexToolEntry`, `DocsAvailable`).
  - `structure.rs`: canonical skill structure model (`SkillStructure`, `StructureItem`).
  - `records.rs`: content/resource/prompt/reference-style record models.
  - `sync.rs`: scan/sync configuration and sync diff logic (`calculate_sync_ops`).
- Preserved external call sites by keeping original `pub use` API on `crate::skills::metadata::*`.
- Preserved behavior and data contract for scanner/index generation paths.

### Structural outcome

- Baseline before split:
  - `skills/metadata.rs`: `1148` lines.
- Current split sizes:
  - `skills/metadata/mod.rs`: `18`
  - `skills/metadata/core.rs`: `374`
  - `skills/metadata/index.rs`: `125`
  - `skills/metadata/structure.rs`: `74`
  - `skills/metadata/records.rs`: `438`
  - `skills/metadata/sync.rs`: `112`
  - total: `1141` lines.

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced.
- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/metadata -g '*.rs'`
  - result: existing carries only
    - `core.rs:#[allow(clippy::struct_excessive_bools)]`
    - `core.rs:#[allow(clippy::too_many_arguments)]`
    - `records.rs:#[allow(clippy::too_many_arguments)]`

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-metadata-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-metadata-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-metadata-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-metadata-split-it cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Skill Scanner Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/scanner.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/frontmatter.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/rules.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/references.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/index_build.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/scan.rs`

### Quality changes adopted

- Converted monolithic `skills/scanner.rs` into a directory module with focused responsibilities:
  - `mod.rs`: module interface + `SkillScanner` type surface.
  - `frontmatter.rs`: SKILL.md frontmatter data model (`SkillFrontmatter`, `SkillMetadataBlock`).
  - `rules.rs`: `extensions/sniffer/rules.toml` parsing (`SnifferRule` extraction).
  - `references.rs`: `references/*.md` frontmatter scan + tool/skill binding derivation.
  - `index_build.rs`: index payload composition (`build_index_entry`, `build_canonical_payload`).
  - `scan.rs`: scan lifecycle (`scan_skill`, `scan_all`, structure validation, SKILL.md parsing).
- Preserved external API and behavior:
  - `SkillScanner` public methods and signatures unchanged.
  - reference wiring semantics in canonical payload unchanged.
  - sniffer rule loading behavior unchanged.
- Maintained behavior-first refactor posture:
  - no new feature flags,
  - no broad lint suppression added,
  - no runtime contract changes for scanner consumers.

### Structural outcome

- Baseline before split:
  - `skills/scanner.rs`: `715` lines.
- Current split sizes:
  - `skills/scanner/mod.rs`: `54`
  - `skills/scanner/scan.rs`: `294`
  - `skills/scanner/index_build.rs`: `125`
  - `skills/scanner/references.rs`: `148`
  - `skills/scanner/rules.rs`: `66`
  - total: `687` lines.

### Suppression debt delta (this slice)

- No new `#[allow(...)]` introduced beyond existing carried annotation semantics.
- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/scanner -g '*.rs'`
  - result:
    - `references.rs:#[allow(dead_code)]` (existing optional reference-frontmatter field carry-over).

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scanner-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scanner-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scanner-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scanner-split-skill cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scanner-split-test-skill cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Skill Command Parser Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/decorator.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/docstring.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/parameters.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/tests.rs`

### Quality changes adopted

- Converted monolithic `skill_command/parser.rs` into a directory module with clear separation:
  - `mod.rs`: stable parser surface + re-exports.
  - `decorator.rs`: decorator location scan and argument parsing.
  - `docstring.rs`: docstring extraction and `Args:` parameter-description parsing.
  - `parameters.rs`: signature/parameter parsing + `ParsedParameter` schema inference helpers.
  - `tests.rs`: moved parser tests out of the production module.
- Preserved public API path and symbols via re-exports:
  - `ParsedParameter`
  - `find_skill_command_decorators`
  - `parse_decorator_args`
  - `extract_docstring_from_text`
  - `parse_parameters`
  - `extract_parameters_from_text`
  - `extract_parsed_parameters`
  - `extract_param_descriptions`
- Brought implementation in line with repository modularization rules:
  - removed inline `#[cfg(test)]` block from a complex production module by moving tests to a dedicated file.

### Structural outcome

- Baseline before split:
  - `skills/skill_command/parser.rs`: `864` lines.
- Current split sizes:
  - `skills/skill_command/parser/mod.rs`: `19`
  - `skills/skill_command/parser/decorator.rs`: `272`
  - `skills/skill_command/parser/docstring.rs`: `93`
  - `skills/skill_command/parser/parameters.rs`: `305`
  - `skills/skill_command/parser/tests.rs`: `183`
  - total: `872` lines.

### Suppression debt delta (this slice)

- No `#[allow(...)]` usage in the new parser directory.
- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-parser-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-parser-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-parser-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-parser-split-tools cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-parser-split-test-skill cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Knowledge Scanner Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/knowledge/scanner.rs`
- `packages/rust/crates/xiuxian-skills/src/knowledge/scanner/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/knowledge/scanner/metadata.rs`
- `packages/rust/crates/xiuxian-skills/src/knowledge/scanner/document.rs`
- `packages/rust/crates/xiuxian-skills/src/knowledge/scanner/scan.rs`
- `packages/rust/crates/xiuxian-skills/src/knowledge/scanner/tests.rs`

### Quality changes adopted

- Converted monolithic `knowledge/scanner.rs` into focused directory modules:
  - `mod.rs`: public scanner surface and module wiring.
  - `metadata.rs`: frontmatter schema (`KnowledgeFrontmatter`).
  - `document.rs`: single-document scan path (`scan_document`) and hash/id helpers.
  - `scan.rs`: directory traversal and query-style filters (`scan_all`, `scan_category`, `scan_with_tags`, `get_tags`).
  - `tests.rs`: moved tests out of production code.
- Preserved external API and behavior:
  - `KnowledgeScanner` method signatures unchanged.
  - category/tag filtering semantics unchanged.
  - frontmatter fallback behavior and title derivation unchanged.
- Improved adherence to modularization rule:
  - removed inline tests from complex runtime module.

### Structural outcome

- Baseline before split:
  - `knowledge/scanner.rs`: `572` lines.
- Current split sizes:
  - `knowledge/scanner/mod.rs`: `53`
  - `knowledge/scanner/metadata.rs`: `29`
  - `knowledge/scanner/document.rs`: `111`
  - `knowledge/scanner/scan.rs`: `189`
  - `knowledge/scanner/tests.rs`: `217`
  - total: `599` lines.

### Suppression debt delta (this slice)

- No `#[allow(...)]` usage in the new `knowledge/scanner` directory.
- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/knowledge/scanner -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-knowledge-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-knowledge-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-knowledge-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-knowledge-split-tools cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-knowledge-split-skill cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Prompt Scanner Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/prompt.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/prompt/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/prompt/scan.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/prompt/tests.rs`

### Quality changes adopted

- Converted `skills/prompt.rs` to a directory module:
  - `mod.rs`: scanner type + module wiring.
  - `scan.rs`: production scanning path (`scan`, `scan_paths`, `scan_file`) and shared extraction helper.
  - `tests.rs`: dedicated prompt scanner tests.
- Removed duplicated prompt-extraction logic:
  - both filesystem scan and virtual-path scan now share a single `build_prompt_records` parser helper.
- Preserved public API and behavior:
  - `PromptScanner::new`
  - `PromptScanner::scan`
  - `PromptScanner::scan_paths`
  - decorator/description/parameter extraction semantics unchanged.

### Structural outcome

- Baseline before split:
  - `skills/prompt.rs`: `254` lines.
- Current split sizes:
  - `skills/prompt/mod.rs`: `12`
  - `skills/prompt/scan.rs`: `184`
  - `skills/prompt/tests.rs`: `32`
  - total: `228` lines.

### Suppression debt delta (this slice)

- No `#[allow(...)]` usage in the new `skills/prompt` directory.
- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/prompt -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-prompt-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-prompt-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-prompt-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-prompt-split-skill cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Resource Scanner Directory Modularization)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/resource.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/resource/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/resource/scan.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/resource/tests.rs`

### Quality changes adopted

- Converted `skills/resource.rs` to a directory module:
  - `mod.rs`: scanner type + module declarations.
  - `scan.rs`: production logic (`scan`, `scan_paths`, `scan_file`) and shared extraction helper.
  - `tests.rs`: moved unit tests out of production module.
- Removed duplicated decorator extraction between file-scan and virtual file-scan:
  - unified through `build_resource_records`.
- Preserved public API and behavior:
  - `ResourceScanner::new`
  - `ResourceScanner::scan`
  - `ResourceScanner::scan_paths`
  - description/resource-uri fallback semantics unchanged.

### Structural outcome

- Baseline before split:
  - `skills/resource.rs`: `278` lines.
- Current split sizes:
  - `skills/resource/mod.rs`: `12`
  - `skills/resource/scan.rs`: `187`
  - `skills/resource/tests.rs`: `51`
  - total: `250` lines.

### Suppression debt delta (this slice)

- No `#[allow(...)]` usage in the new `skills/resource` directory.
- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/resource -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-resource-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-resource-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-resource-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-resource-split-skill cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Constructor Quality Refactor: Remove `too_many_arguments`)

### Changed files

- `packages/rust/crates/xiuxian-skills/src/skills/metadata/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/records.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/parse.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/resource/scan.rs`

### Quality changes adopted

- Removed `clippy::too_many_arguments` suppression by changing constructor shape instead of muting lint:
  - adopted `ToolRecord::with_enrichment(..., ToolEnrichment)` payload-based construction at call sites.
  - exported `ToolEnrichment` from `metadata/mod.rs` and updated scanner parse flow.
  - `tools/parse.rs` now builds `ToolEnrichment` once and passes it into `ToolRecord::with_enrichment`.
- Removed `clippy::too_many_arguments` suppression from `ResourceRecord::new(...)`:
  - `mime_type` now defaults to `application/json` inside constructor.
  - added `ResourceRecord::with_mime_type(...)` for non-default MIME cases.
  - updated resource scan call sites accordingly.
- Outcome:
  - no behavior change in current scanner flows,
  - fewer long-argument APIs,
  - lower suppression debt.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates/xiuxian-skills/src/skills/metadata -g '*.rs'`
  - result: no matches.
- Remaining known allow in metadata:
  - `core.rs:#[allow(clippy::struct_excessive_bools)]` (intentional MCP annotation bool flags).

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-argfix cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-argfix cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-argfix-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-argfix-tools cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-argfix-skill cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Tools Unit Test Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/tools/tests.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/tests/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/tests/scan_scripts.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/tests/parse_content.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/tests/scan_paths.rs`

### Quality changes adopted

- Split large `tools` unit test module into focused files by responsibility:
  - `scan_scripts.rs`: filesystem script scanning behavior.
  - `parse_content.rs`: direct content parser behavior and hash/category/intents logic.
  - `scan_paths.rs`: virtual-path scanning behavior and filter rules.
  - `mod.rs`: test module wiring only.
- Preserved all test logic and assertions; no behavior changes in production code.
- Improved test maintainability and discoverability for targeted runs/debugging.

### Structural outcome

- Baseline before split:
  - `skills/tools/tests.rs`: `538` lines.
- Current split sizes:
  - `skills/tools/tests/mod.rs`: `3`
  - `skills/tools/tests/scan_scripts.rs`: `193`
  - `skills/tools/tests/parse_content.rs`: `184`
  - `skills/tools/tests/scan_paths.rs`: `165`
  - total: `545` lines.

### Suppression debt delta (this slice)

- No `#[allow(...)]` usage in the new `skills/tools/tests` directory.
- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/tools/tests -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-tests-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-tests-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-tests-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-tests-split-it cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Annotation Model Refactor: Remove `struct_excessive_bools`)

### Changed files

- `packages/rust/crates/xiuxian-skills/src/skills/metadata/core.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/annotations.rs`
- `packages/rust/crates/xiuxian-skills/tests/tools_scanner.rs`

### Quality changes adopted

- Replaced bool-heavy `ToolAnnotations` shape with a nested behavior model:
  - introduced `ToolBehaviorAnnotations { idempotent, open_world }`,
  - flattened behavior fields into external JSON via `#[serde(flatten)]` to preserve wire shape.
- Added intent-revealing accessors and mutators:
  - `is_idempotent`, `set_idempotent`,
  - `is_open_world`, `set_open_world`.
- Updated annotation inference and tests to use the new API instead of direct field mutation/access.
- Outcome:
  - removed `#[allow(clippy::struct_excessive_bools)]` suppression,
  - kept MCP annotation serialization compatibility,
  - improved model maintainability and extensibility.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(clippy::struct_excessive_bools\\)" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-records-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-records-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-records-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-records-split-it cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `omni-tui` State Reducer Decomposition)

### Changed files

- `packages/rust/crates/omni-tui/src/state/mod.rs`

### Quality changes adopted

- Refactored `AppState::reduce` from a monolithic match into focused reducers:
  - domain dispatch (`reduce_system_event`, `reduce_cortex_event`, `reduce_task_event`, `reduce_log_event`),
  - event-specific handlers (`handle_cortex_*`, `handle_task_*`),
  - payload extraction helpers (`payload_str`, `payload_u64`, `payload_f64`, `payload_bool`).
- Preserved topic behavior and side effects for:
  - system lifecycle topics,
  - cortex execution topics,
  - task lifecycle topics,
  - log ingestion topics.
- Removed blanket lint suppression:
  - deleted `#[allow(clippy::too_many_lines)]` from reducer path.

### Suppression debt delta (this slice)

- Validation:
- `rg -n "allow\\(clippy::too_many_lines\\)" packages/rust/crates/omni-tui/src/state/mod.rs`
  - result: no matches.
- Workspace scan:
- `rg -n "allow\\(clippy::too_many_lines\\)" packages/rust/crates -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p omni-tui`
  - result: passed.
- `cargo check -p omni-tui`
  - result: passed.
- `cargo clippy -p omni-tui -- -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p omni-tui -- --nocapture`
  - result: passed (`11` lib tests, `3` main tests, `2` component tests, `9` socket tests, `17` state tests; 0 failed).

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Tools Filesystem Scripts Submodule Decomposition)

### Changed files

- deleted (worktree): `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/filesystem/scripts.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/filesystem/scripts/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/filesystem/scripts/entries.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/filesystem/scripts/collect.rs`

### Quality changes adopted

- Split script-scan responsibilities into focused submodules:
  - `entries.rs`: filesystem walk and candidate path collection,
  - `collect.rs`: per-file parse loop and tool aggregation,
  - `mod.rs`: public `ToolsScanner::scan_scripts` entrypoint and module wiring.
- Improved signature quality in helpers:
  - replaced `&PathBuf` argument usage with `&Path`,
  - removed redundant closure in path mapping (`map(DirEntry::into_path)`).
- Preserved scanner behavior and API contract for script discovery and parsing.

### Structural outcome

- Current split sizes:
  - `skills/tools/scan/filesystem/scripts/mod.rs`: `68`
  - `skills/tools/scan/filesystem/scripts/entries.rs`: `15`
  - `skills/tools/scan/filesystem/scripts/collect.rs`: `51`
  - total: `134` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/tools/scan/filesystem/scripts -g '*.rs'`
  - result: no matches.
- Workspace scan:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-filesystem-scripts-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-filesystem-scripts-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-filesystem-scripts-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 78 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-filesystem-scripts-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-filesystem-scripts-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-filesystem-scripts-split cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Parameter Model Submodule Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/parameters/model.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/parameters/model/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/parameters/model/infer.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/parameters/model/tests.rs`

### Quality changes adopted

- Split parameter-model responsibilities into focused modules:
  - `mod.rs`: `ParsedParameter` model and public methods (`is_optional`, `infer_json_type`, `to_json_schema_property`),
  - `infer.rs`: type-annotation to JSON-schema inference helpers,
  - `tests.rs`: dedicated unit tests for scalar/list/dict/literal/default behavior.
- Preserved external API:
  - `ParsedParameter` path and method signatures unchanged for parser and schema builders.
- Added focused unit coverage for inference contracts instead of relying only on higher-level parser tests.

### Structural outcome

- Baseline before split:
  - `skills/skill_command/parser/parameters/model.rs`: `115` lines.
- Current split sizes:
  - `skills/skill_command/parser/parameters/model/mod.rs`: `47`
  - `skills/skill_command/parser/parameters/model/infer.rs`: `75`
  - `skills/skill_command/parser/parameters/model/tests.rs`: `57`
  - total: `179` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/parameters/model -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-parameters-model-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-parameters-model-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-parameters-model-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 78 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-parameters-model-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-parameters-model-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-parameters-model-split cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Skill Single-Scan Submodule Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/scanner/scan/single.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/scan/single/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/scan/single/core.rs`

### Quality changes adopted

- Split single-skill scan flow into focused modules:
  - `mod.rs`: public `SkillScanner` method surface (`scan_skill`, `scan_skill_inner`) and wiring,
  - `core.rs`: shared scan implementation and logging/structure-check helpers.
- Removed duplicated logic between strict (`Result`) and best-effort (`Option`) paths:
  - both now flow through one shared core implementation (`scan_skill_result`),
  - `scan_skill_inner` remains best-effort by converting errors to `None`.
- Preserved external behavior:
  - `scan_skill` still returns `Err` on read/parse failures,
  - `scan_skill_inner` still returns `None` on read/parse failures,
  - structure mismatch warning behavior unchanged.

### Structural outcome

- Baseline before split:
  - `skills/scanner/scan/single.rs`: `101` lines.
- Current split sizes:
  - `skills/scanner/scan/single/mod.rs`: `54`
  - `skills/scanner/scan/single/core.rs`: `56`
  - total: `110` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/scanner/scan/single -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scan-single-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scan-single-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scan-single-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 78 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scan-single-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scan-single-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scan-single-split cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Skill Markdown Parse Submodule Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/scanner/scan/parse.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/scan/parse/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/scan/parse/extract.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/scan/parse/metadata.rs`

### Quality changes adopted

- Split SKILL frontmatter parsing responsibilities into focused modules:
  - `extract.rs`: skill-name extraction and raw YAML frontmatter decode,
  - `metadata.rs`: metadata block normalization and `SkillMetadata` assembly,
  - `mod.rs`: public `parse_skill_md` API wiring for `SkillScanner`.
- Preserved external behavior:
  - missing frontmatter still returns default metadata with warning,
  - malformed YAML still returns parse error,
  - missing `metadata` block still logs warning and returns defaults.
- Removed one introduced clippy warning (`derivable_impls`) by switching the local
  metadata-fields carrier to `#[derive(Default)]`.

### Structural outcome

- Baseline before split:
  - `skills/scanner/scan/parse.rs`: `97` lines.
- Current split sizes:
  - `skills/scanner/scan/parse/mod.rs`: `42`
  - `skills/scanner/scan/parse/extract.rs`: `27`
  - `skills/scanner/scan/parse/metadata.rs`: `76`
  - total: `145` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/scanner/scan/parse -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scan-parse-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scan-parse-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scan-parse-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 78 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scan-parse-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scan-parse-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-scan-parse-split cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Tools Virtual-Paths Scan Submodule Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/virtual_paths.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/virtual_paths/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/virtual_paths/filter.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/virtual_paths/scan.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/virtual_paths/tests.rs`

### Quality changes adopted

- Split `virtual_paths` scan responsibilities into focused modules:
  - `filter.rs`: virtual file skip policy (`__init__.py`, private files, non-`.py` extension),
  - `scan.rs`: `scan_paths` orchestration and parse/log pipeline,
  - `mod.rs`: interface-only module declaration.
- Preserved external behavior:
  - `ToolsScanner::scan_paths` signature and output semantics unchanged.
- Added focused unit tests for skip-policy behavior to harden the scan filter contract.

### Structural outcome

- Baseline before split:
  - `skills/tools/scan/virtual_paths.rs`: `112` lines.
- Current split sizes:
  - `skills/tools/scan/virtual_paths/mod.rs`: `5`
  - `skills/tools/scan/virtual_paths/filter.rs`: `17`
  - `skills/tools/scan/virtual_paths/scan.rs`: `95`
  - `skills/tools/scan/virtual_paths/tests.rs`: `19`
  - total: `136` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/tools/scan/virtual_paths -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-virtual-paths-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-virtual-paths-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-virtual-paths-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 74 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-virtual-paths-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-virtual-paths-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-virtual-paths-split cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Knowledge Document Scan Submodule Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/knowledge/scanner/document.rs`
- `packages/rust/crates/xiuxian-skills/src/knowledge/scanner/document/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/knowledge/scanner/document/identity.rs`
- `packages/rust/crates/xiuxian-skills/src/knowledge/scanner/document/frontmatter.rs`

### Quality changes adopted

- Split `scan_document` internals into focused modules:
  - `identity.rs`: content hash and stable path-id generation helpers,
  - `frontmatter.rs`: frontmatter/content parsing and title fallback resolution,
  - `mod.rs`: orchestration (`scan_document`) and final `KnowledgeEntry` assembly.
- Preserved external behavior:
  - `KnowledgeScanner::scan_document` signature and return contract unchanged,
  - markdown-only filter, category fallback, and preview generation semantics unchanged.
- Kept implementation lint-clean without suppression additions.

### Structural outcome

- Baseline before split:
  - `knowledge/scanner/document.rs`: `111` lines.
- Current split sizes:
  - `knowledge/scanner/document/mod.rs`: `72`
  - `knowledge/scanner/document/identity.rs`: `17`
  - `knowledge/scanner/document/frontmatter.rs`: `28`
  - total: `117` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/knowledge/scanner/document -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-document-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-document-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-document-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 74 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-document-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-document-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-document-split cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Tools Decorated Build Submodule Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/tools/parse/decorated/build.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/parse/decorated/build/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/parse/decorated/build/resolve.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/parse/decorated/build/convert.rs`

### Quality changes adopted

- Split `decorated/build` responsibilities into focused modules:
  - `resolve.rs`: decorator and fallback resolution (`tool_name`, `description`, `category`, keyword merge),
  - `convert.rs`: parser-to-model conversion (`ParsedParameter`, decorator args, parameter names),
  - `mod.rs`: orchestration boundary for record materialization.
- Preserved external behavior:
  - `parse_decorated_tools` output contract unchanged,
  - enrichment assembly path (`annotations`, `input_schema`, routing fields) unchanged.
- Kept the slice lint-clean with no suppression additions.

### Structural outcome

- Baseline before split (current working branch state):
  - `skills/tools/parse/decorated/build.rs`: `135` lines.
- Current split sizes:
  - `skills/tools/parse/decorated/build/mod.rs`: `70`
  - `skills/tools/parse/decorated/build/resolve.rs`: `44`
  - `skills/tools/parse/decorated/build/convert.rs`: `38`
  - total: `152` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/tools/parse/decorated/build -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-build-subsplit cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-build-subsplit cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-build-subsplit cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 68 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-build-subsplit cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-build-subsplit cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-build-subsplit cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Metadata Sync Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/metadata/sync.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/sync/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/sync/config.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/sync/report.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/sync/calculate.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/sync/tests.rs`

### Quality changes adopted

- Split mixed `sync.rs` concerns into focused modules:
  - `config.rs`: `ScanConfig` defaults and builder helpers,
  - `report.rs`: `SyncReport` model and constructor,
  - `calculate.rs`: sync-diff algorithm and lookup helpers,
  - `tests.rs`: unit tests for config defaults and sync classification behavior.
- Improved deterministic behavior in deletion reporting:
  - `deleted` list now follows the existing index slice order instead of hash-map iteration order.
- Preserved external API surface:
  - `ScanConfig`, `SyncReport`, and `calculate_sync_ops` continue to be re-exported via `metadata/mod.rs`.

### Structural outcome

- Baseline before split:
  - `skills/metadata/sync.rs`: `112` lines.
- Current split sizes:
  - `skills/metadata/sync/mod.rs`: `10`
  - `skills/metadata/sync/config.rs`: `37`
  - `skills/metadata/sync/report.rs`: `22`
  - `skills/metadata/sync/calculate.rs`: `55`
  - `skills/metadata/sync/tests.rs`: `70`
  - total: `194` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/metadata/sync -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-sync-dir-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-sync-dir-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-sync-dir-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 71 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-sync-dir-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-sync-dir-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-sync-dir-split cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` References Scan Submodule Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/scanner/references/scan.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/references/scan/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/references/scan/filesystem.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/references/scan/build.rs`

### Quality changes adopted

- Split `scan` responsibilities into dedicated submodules:
  - `filesystem.rs`: references directory discovery + markdown file identity/content loading,
  - `build.rs`: frontmatter parse + `ReferenceRecord` materialization rules,
  - `mod.rs`: orchestration and logging boundary.
- Preserved external scanner behavior:
  - `scan_references` signature/semantics unchanged,
  - fallback skill-name and keyword merge behavior unchanged.
- Kept existing reference tests green without changing test contracts.

### Structural outcome

- Baseline before split (current working branch state):
  - `skills/scanner/references/scan.rs`: `124` lines.
- Current split sizes:
  - `skills/scanner/references/scan/mod.rs`: `43`
  - `skills/scanner/references/scan/filesystem.rs`: `40`
  - `skills/scanner/references/scan/build.rs`: `62`
  - total: `145` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/scanner/references/scan -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-references-scan-split cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-references-scan-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-references-scan-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-references-scan-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 68 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-references-scan-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-references-scan-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-references-scan-split cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Metadata Index Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/metadata/index.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/index/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/index/docs_available.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/index/tool_entry.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/index/skill_entry.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/index/tests.rs`

### Quality changes adopted

- Split index-model concerns into focused modules:
  - `skill_entry.rs`: `SkillIndexEntry` and its behavior methods,
  - `tool_entry.rs`: `IndexToolEntry` schema model,
  - `docs_available.rs`: `DocsAvailable` model + default policy,
  - `tests.rs`: unit tests for default shape and tool insertion behavior,
  - `mod.rs`: interface-only re-export layer.
- Preserved external API consumed by scanner/indexing paths:
  - `pub use index::{DocsAvailable, IndexToolEntry, SkillIndexEntry}` unchanged.
- Added explicit unit coverage for index defaults and `add_tool`/`has_tools` behavior.

### Structural outcome

- Baseline before split (current working branch state):
  - `skills/metadata/index.rs`: `125` lines.
- Current split sizes:
  - `skills/metadata/index/mod.rs`: `10`
  - `skills/metadata/index/docs_available.rs`: `27`
  - `skills/metadata/index/tool_entry.rs`: `20`
  - `skills/metadata/index/skill_entry.rs`: `84`
  - `skills/metadata/index/tests.rs`: `34`
  - total: `175` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/metadata/index -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-index-metadata-split cargo fmt -p xiuxian-skills --check`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-index-metadata-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-index-metadata-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-index-metadata-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 68 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-index-metadata-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-index-metadata-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-index-metadata-split cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Reference Record Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/metadata/records/reference.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/records/reference/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/records/reference/record.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/records/reference/serde_helpers.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/records/reference/tests.rs`

### Quality changes adopted

- Split `ReferenceRecord` concerns into focused modules:
  - `record.rs`: schema model + behavior methods (`new`, `with_for_tools`, `applies_to_tool`),
  - `serde_helpers.rs`: string/array compatibility deserializers,
  - `tests.rs`: serde compatibility and behavior contract tests,
  - `mod.rs`: interface-only export layer.
- Preserved compatibility features:
  - `for_skills` accepts string or string array,
  - `for_tools` accepts string or string array and supports `for_tool` alias.
- Added dedicated unit tests for key backward-compat deserialization paths.

### Structural outcome

- Baseline before split (current working branch state):
  - `skills/metadata/records/reference.rs`: `164` lines.
- Current split sizes:
  - `skills/metadata/records/reference/mod.rs`: `7`
  - `skills/metadata/records/reference/record.rs`: `83`
  - `skills/metadata/records/reference/serde_helpers.rs`: `82`
  - `skills/metadata/records/reference/tests.rs`: `71`
  - total: `243` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/metadata/records/reference -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-reference-record-split cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-reference-record-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-reference-record-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-reference-record-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 66 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-reference-record-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-reference-record-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-reference-record-split cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Tool Record Core Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/metadata/core/tool_record.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/core/tool_record/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/core/tool_record/model.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/core/tool_record/construct.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/core/tool_record/tests.rs`

### Quality changes adopted

- Split `ToolRecord` concerns into focused modules:
  - `model.rs`: data models (`ToolRecord`, `ToolEnrichment`) and serde defaults,
  - `construct.rs`: constructor logic (`new`, `with_enrichment`),
  - `tests.rs`: constructor contract verification tests,
  - `mod.rs`: interface-only re-export boundary.
- Preserved public API surface from `skills::metadata`:
  - `ToolRecord` and `ToolEnrichment` remain re-exported via parent module path.
- Added explicit unit tests for constructor invariants instead of relying only
  on integration coverage.

### Structural outcome

- Baseline before split (current working branch state):
  - `skills/metadata/core/tool_record.rs`: `152` lines.
- Current split sizes:
  - `skills/metadata/core/tool_record/mod.rs`: `7`
  - `skills/metadata/core/tool_record/model.rs`: `91`
  - `skills/metadata/core/tool_record/construct.rs`: `63`
  - `skills/metadata/core/tool_record/tests.rs`: `71`
  - total: `232` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/metadata/core/tool_record -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tool-record-split cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tool-record-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tool-record-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tool-record-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 63 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tool-record-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tool-record-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tool-record-split cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Tools Decorated Parse Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/tools/parse/decorated.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/parse/decorated/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/parse/decorated/build.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/parse/decorated/collect.rs`

### Quality changes adopted

- Split decorated function parsing into focused modules:
  - `mod.rs`: parser orchestration (`tree-sitter` discovery + pipeline wiring),
  - `build.rs`: `ToolRecord`/`ToolEnrichment` materialization and field resolution helpers,
  - `collect.rs`: docstring extraction helper.
- Preserved `parse_decorated_tools` behavior and output contract:
  - decorator override precedence for `name`, `description`, and `category`,
  - fallback description/category semantics,
  - annotation/schema generation path unchanged.
- Reduced per-file cognitive load in one of the central tool metadata build paths.

### Structural outcome

- Baseline before split (current working branch state):
  - `skills/tools/parse/decorated.rs`: `125` lines.
- Current split sizes:
  - `skills/tools/parse/decorated/mod.rs`: `49`
  - `skills/tools/parse/decorated/build.rs`: `135`
  - `skills/tools/parse/decorated/collect.rs`: `13`
  - total: `197` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/tools/parse/decorated -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-decorated-split cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-decorated-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-decorated-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-decorated-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 61 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-decorated-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-decorated-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-decorated-split cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Tools Filesystem Scan Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/filesystem.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/filesystem/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/filesystem/filters.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/filesystem/scripts.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/filesystem/structure.rs`

### Quality changes adopted

- Split filesystem scanning responsibilities into focused modules:
  - `filters.rs`: Python file inclusion/exclusion rule (`__init__.py`, private files, extension),
  - `scripts.rs`: `scan_scripts` orchestration and walkdir-based parsing pipeline,
  - `structure.rs`: `scan_skill_scripts` and `scan_with_structure` entrypoints,
  - `mod.rs`: interface-only module boundary.
- Preserved external `ToolsScanner` API signatures and behavior for:
  - `scan_scripts`,
  - `scan_skill_scripts`,
  - `scan_with_structure`.
- Kept parse error propagation and scan logging behavior unchanged.

### Structural outcome

- Baseline before split (current working branch state):
  - `skills/tools/scan/filesystem.rs`: `171` lines.
- Current split sizes:
  - `skills/tools/scan/filesystem/mod.rs`: `3`
  - `skills/tools/scan/filesystem/filters.rs`: `17`
  - `skills/tools/scan/filesystem/scripts.rs`: `99`
  - `skills/tools/scan/filesystem/structure.rs`: `80`
  - total: `199` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/tools/scan/filesystem -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-filesystem-split cargo fmt -p xiuxian-skills --check`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-filesystem-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-filesystem-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-filesystem-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 61 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-filesystem-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-filesystem-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-filesystem-split cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Skill Index Build Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/scanner/index_build.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/index_build/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/index_build/index_entry.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/index_build/canonical.rs`

### Quality changes adopted

- Split index construction responsibilities into dedicated modules:
  - `index_entry.rs`: `build_index_entry` path (metadata aggregation + unique tool projection),
  - `canonical.rs`: `build_canonical_payload` path (reference map + per-tool reference wiring),
  - `mod.rs`: interface-only module declaration.
- Preserved public API on `SkillScanner`:
  - `build_index_entry` signature/behavior unchanged,
  - `build_canonical_payload` signature/behavior unchanged.
- Extracted per-tool reference matching into focused helper to keep canonical
  path orchestration readable and explicit.

### Structural outcome

- Baseline before split:
  - `skills/scanner/index_build.rs`: `125` lines.
- Current split sizes:
  - `skills/scanner/index_build/mod.rs`: `2`
  - `skills/scanner/index_build/index_entry.rs`: `66`
  - `skills/scanner/index_build/canonical.rs`: `77`
  - total: `145` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/scanner/index_build -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-index-build-split cargo fmt -p xiuxian-skills --check`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-index-build-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-index-build-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-index-build-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 61 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-index-build-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-index-build-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-index-build-split cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Skill References Scanner Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/scanner/references.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/references/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/references/model.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/references/values.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/references/scan.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/references/tests.rs`

### Quality changes adopted

- Split mixed responsibilities into focused modules:
  - `model.rs`: frontmatter schema types for reference metadata,
  - `values.rs`: YAML value normalization + tool-to-skill derivation helpers,
  - `scan.rs`: filesystem scan orchestration and `ReferenceRecord` construction,
  - `tests.rs`: focused unit tests for value conversion and reference parsing behavior,
  - `mod.rs`: interface-only module boundary.
- Preserved scanner behavior used by both index and canonical payload builders:
  - `for_tools` parsing from scalar/sequence YAML,
  - `for_skills` derivation from tool prefixes with fallback to current skill,
  - merged keyword behavior (`routing_keywords` + `intents`).
- Added three new unit tests for this module to harden conversion and scan behavior.

### Structural outcome

- Baseline before split:
  - `skills/scanner/references.rs`: `148` lines.
- Current split sizes:
  - `skills/scanner/references/mod.rs`: `19`
  - `skills/scanner/references/model.rs`: `27`
  - `skills/scanner/references/values.rs`: `39`
  - `skills/scanner/references/scan.rs`: `124`
  - `skills/scanner/references/tests.rs`: `91`
  - total: `300` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/scanner/references -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-references-split cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-references-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-references-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-references-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 61 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-references-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-references-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-references-split cargo test -p xiuxian-skills --test test_skill_scanner -- --nocapture`
  - result: 17 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Skill Command Annotations Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/skill_command/annotations.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/annotations/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/annotations/build.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/annotations/heuristics.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/annotations/tests.rs`

### Quality changes adopted

- Split annotation construction responsibilities into dedicated modules:
  - `build.rs`: orchestration for explicit overrides + heuristics + final invariants,
  - `heuristics.rs`: read-only/destructive/open-world indicator tables and helpers,
  - `tests.rs`: focused behavior tests for heuristics and explicit override rules,
  - `mod.rs`: interface-only export surface.
- Preserved external API:
  - `build_annotations` function name/signature unchanged,
  - annotation semantics preserved for read-only, destructive, idempotent, and open-world flags.
- Removed inline tests from implementation file to match modular test-placement practice.

### Structural outcome

- Baseline before split:
  - `skills/skill_command/annotations.rs`: `140` lines.
- Current split sizes:
  - `skills/skill_command/annotations/mod.rs`: `12`
  - `skills/skill_command/annotations/build.rs`: `52`
  - `skills/skill_command/annotations/heuristics.rs`: `47`
  - `skills/skill_command/annotations/tests.rs`: `35`
  - total: `146` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/skill_command/annotations -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-annotations-split cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-annotations-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-annotations-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-annotations-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-annotations-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-annotations-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Prompt Scanner Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/prompt/scan.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/prompt/scan/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/prompt/scan/build.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/prompt/scan/filesystem.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/prompt/scan/paths.rs`

### Quality changes adopted

- Split `PromptScanner` by concern with interface-only `mod.rs`:
  - `build.rs`: tree-sitter parsing and `PromptRecord` construction,
  - `filesystem.rs`: directory traversal and script scanning from disk,
  - `paths.rs`: in-memory path/content scan pipeline used by tests.
- Preserved external scanner API surface (`new`, `scan`, `scan_paths`) and output behavior.
- Kept this slice lint-clean without adding suppression attributes.

### Structural outcome

- Baseline before split:
  - `skills/prompt/scan.rs`: `184` lines.
- Current split sizes:
  - `skills/prompt/scan/mod.rs`: `19`
  - `skills/prompt/scan/build.rs`: `54`
  - `skills/prompt/scan/filesystem.rs`: `95`
  - `skills/prompt/scan/paths.rs`: `34`
  - total: `202` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/prompt/scan -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-prompt-scan-split-skill cargo fmt -p xiuxian-skills --check`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-prompt-scan-split-skill cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-prompt-scan-split-skill cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-prompt-scan-split-skill cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-prompt-scan-split-skill cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-prompt-scan-split-skill cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Skill Command Category Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/skill_command/category.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/category/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/category/infer.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/category/rules.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/category/tests.rs`

### Quality changes adopted

- Split category inference into focused modules:
  - `infer.rs`: public API (`infer_category_from_skill`) and fallback behavior,
  - `rules.rs`: category-rule table and keyword-match helper,
  - `tests.rs`: dedicated module tests for all category branches,
  - `mod.rs`: interface-only exports.
- Preserved API and behavior contract:
  - unchanged exported function name and return semantics,
  - unknown categories still return original skill name.
- Moved inline tests out of implementation module into `tests.rs` to align with
  project modularization/test placement rules.

### Structural outcome

- Baseline before split:
  - `skills/skill_command/category.rs`: `176` lines.
- Current split sizes:
  - `skills/skill_command/category/mod.rs`: `12`
  - `skills/skill_command/category/infer.rs`: `21`
  - `skills/skill_command/category/rules.rs`: `54`
  - `skills/skill_command/category/tests.rs`: `73`
  - total: `160` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/skill_command/category -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-category-split cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-category-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-category-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-category-split cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-category-split cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-category-split cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Resource Scanner Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/resource/scan.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/resource/scan/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/resource/scan/build.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/resource/scan/filesystem.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/resource/scan/paths.rs`

### Quality changes adopted

- Split `ResourceScanner` concerns into focused submodules:
  - `build.rs`: tree-sitter extraction and `ResourceRecord` construction,
  - `filesystem.rs`: directory traversal + on-disk file scanning,
  - `paths.rs`: in-memory path/content scanning (test-oriented),
  - `mod.rs`: constructor/default wiring only.
- Preserved existing `ResourceScanner` public API (`new`, `scan`, `scan_paths`).
- Kept behavior and decorator parsing semantics unchanged.

### Structural outcome

- Baseline before split:
  - `skills/resource/scan.rs`: `186` lines.
- Current split sizes:
  - `skills/resource/scan/mod.rs`: `19`
  - `skills/resource/scan/build.rs`: `52`
  - `skills/resource/scan/filesystem.rs`: `95`
  - `skills/resource/scan/paths.rs`: `34`
  - total: `200` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/resource/scan -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-resource-scan-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-resource-scan-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-resource-scan-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-resource-scan-split-tools cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-resource-scan-split-skill cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Knowledge Scanner Scan Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/knowledge/scanner/scan.rs`
- `packages/rust/crates/xiuxian-skills/src/knowledge/scanner/scan/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/knowledge/scanner/scan/core.rs`
- `packages/rust/crates/xiuxian-skills/src/knowledge/scanner/scan/filters.rs`

### Quality changes adopted

- Split knowledge scanning responsibilities by concern:
  - `core.rs`: filesystem traversal + depth validation + parallel document scan (`scan_all`),
  - `filters.rs`: category/tag filtering and tag aggregation (`scan_category`, `scan_with_tags`, `get_tags`),
  - `mod.rs`: constructor/default wiring only.
- Kept public `KnowledgeScanner` API stable and behavior-equivalent.
- Reduced hot-path scanner file complexity while preserving test coverage.

### Structural outcome

- Baseline before split:
  - `knowledge/scanner/scan.rs`: `189` lines.
- Current split sizes:
  - `knowledge/scanner/scan/mod.rs`: `18`
  - `knowledge/scanner/scan/core.rs`: `87`
  - `knowledge/scanner/scan/filters.rs`: `99`
  - total: `204` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/knowledge/scanner/scan -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-knowledge-scan-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-knowledge-scan-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-knowledge-scan-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-knowledge-scan-split-tools cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-knowledge-scan-split-skill cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Knowledge Types Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/knowledge/types.rs`
- `packages/rust/crates/xiuxian-skills/src/knowledge/types/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/knowledge/types/category.rs`
- `packages/rust/crates/xiuxian-skills/src/knowledge/types/metadata.rs`
- `packages/rust/crates/xiuxian-skills/src/knowledge/types/entry.rs`
- updated: `packages/rust/crates/xiuxian-skills/src/lib.rs` (architecture comment path)

### Quality changes adopted

- Split mixed knowledge type concerns into focused files:
  - `category.rs`: `KnowledgeCategory` enum + `FromStr` + `Display`,
  - `metadata.rs`: `KnowledgeMetadata` model and builder-style helpers,
  - `entry.rs`: `KnowledgeEntry` model + category string projection,
  - `mod.rs`: interface-only re-export surface.
- Preserved existing import surface:
  - `crate::knowledge::types::{KnowledgeCategory, KnowledgeEntry, KnowledgeMetadata}`
- Updated top-level architecture comment to reflect directory module layout.

### Structural outcome

- Baseline before split:
  - `knowledge/types.rs`: `217` lines.
- Current split sizes:
  - `knowledge/types/mod.rs`: `9`
  - `knowledge/types/category.rs`: `70`
  - `knowledge/types/metadata.rs`: `61`
  - `knowledge/types/entry.rs`: `89`
  - total: `229` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/knowledge/types -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-knowledge-types-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-knowledge-types-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-knowledge-types-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-knowledge-types-split-tools cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-knowledge-types-split-skill cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Skill Metadata Scan Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/scanner/scan.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/scan/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/scan/all.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/scan/single.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/scanner/scan/parse.rs`

### Quality changes adopted

- Split broad `SkillScanner` implementation by responsibility:
  - `mod.rs`: constructor/defaults + structure validation,
  - `single.rs`: single-skill scan flow (`scan_skill`, `scan_skill_inner`),
  - `all.rs`: parallel multi-skill scanning (`scan_all`),
  - `parse.rs`: SKILL.md frontmatter parsing (`parse_skill_md`).
- Kept all public `SkillScanner` APIs and signatures stable.
- Added explicit focused integration validation for `skill_scanner` in this slice.

### Structural outcome

- Baseline before split:
  - `skills/scanner/scan.rs`: `294` lines.
- Current split sizes:
  - `skills/scanner/scan/mod.rs`: `55`
  - `skills/scanner/scan/all.rs`: `55`
  - `skills/scanner/scan/single.rs`: `101`
  - `skills/scanner/scan/parse.rs`: `97`
  - total: `308` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/scanner/scan -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-skill-scan-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-skill-scan-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-skill-scan-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-skill-scan-split-tools cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-skill-scan-split-skill cargo test -p xiuxian-skills --test skill_scanner -- --nocapture`
  - result: 15 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Decorator Parser Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/decorator.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/decorator/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/decorator/find.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/decorator/args.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/decorator/strings.rs`

### Quality changes adopted

- Split decorator parsing into focused modules:
  - `find.rs`: `@skill_command` decorator discovery with quote-aware paren matching,
  - `args.rs`: key-value argument parsing and triple-quote-safe argument extraction,
  - `strings.rs`: shared string/token helpers for argument splitting and literal cleanup,
  - `mod.rs`: parser surface and stable re-export wiring.
- Kept public parser API stable:
  - `find_skill_command_decorators`
  - `parse_decorator_args`
- Improved readability of one of the highest-complexity parser files without changing behavior.

### Structural outcome

- Baseline before split:
  - `skills/skill_command/parser/decorator.rs`: `272` lines.
- Current split sizes:
  - `skills/skill_command/parser/decorator/mod.rs`: `8`
  - `skills/skill_command/parser/decorator/find.rs`: `93`
  - `skills/skill_command/parser/decorator/args.rs`: `102`
  - `skills/skill_command/parser/decorator/strings.rs`: `64`
  - total: `267` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/decorator -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-decorator-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-decorator-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-decorator-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-decorator-split-it cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Parameter Parser Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/parameters.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/parameters/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/parameters/model.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/parameters/parse.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/parameters/signature.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/parameters/split.rs`

### Quality changes adopted

- Split parameter parsing and schema inference into focused modules:
  - `model.rs`: `ParsedParameter` data model and JSON Schema type inference,
  - `split.rs`: bracket-aware parameter segmentation,
  - `parse.rs`: parameter-name parsing and detailed parameter extraction,
  - `signature.rs`: function-signature slicing and high-level extract APIs,
  - `mod.rs`: parser parameter surface and re-exports.
- Kept external parser API stable via unchanged re-export names in
  `skills/skill_command/parser/mod.rs`.
- Reduced single-file cognitive load in a high-change parser lane.

### Structural outcome

- Baseline before split:
  - `skills/skill_command/parser/parameters.rs`: `305` lines.
- Current split sizes:
  - `skills/skill_command/parser/parameters/mod.rs`: `10`
  - `skills/skill_command/parser/parameters/model.rs`: `115`
  - `skills/skill_command/parser/parameters/parse.rs`: `73`
  - `skills/skill_command/parser/parameters/signature.rs`: `39`
  - `skills/skill_command/parser/parameters/split.rs`: `37`
  - total: `274` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/parameters -g '*.rs'`
  - result: no matches.
- Workspace-level check:
  - `rg -n "allow\\(clippy::" packages/rust/crates/xiuxian-skills/src -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-params-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-params-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-params-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-params-split-it cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Tools Scanner Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/tools/scan.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/filesystem.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/scan/virtual_paths.rs`

### Quality changes adopted

- Split scanner concerns by execution surface:
  - `filesystem.rs`: directory walking and on-disk script scanning (`scan_scripts`, `scan_skill_scripts`, `scan_with_structure`),
  - `virtual_paths.rs`: in-memory path/content scanning (`scan_paths`),
  - `mod.rs`: module composition only.
- Preserved all external `ToolsScanner` scanning APIs and semantics.
- Isolated skip-filter logic close to each scan surface for easier maintenance and testing.

### Structural outcome

- Baseline before split:
  - `skills/tools/scan.rs`: `270` lines.
- Current split sizes:
  - `skills/tools/scan/mod.rs`: `4`
  - `skills/tools/scan/filesystem.rs`: `171`
  - `skills/tools/scan/virtual_paths.rs`: `112`
  - total: `287` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/tools/scan -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-scan-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-scan-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-scan-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-scan-split-it cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Tools Parser Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/tools/parse.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/parse/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/parse/content.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/parse/decorated.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/tools/parse/hashing.rs`

### Quality changes adopted

- Split mixed parser responsibilities into focused modules:
  - `content.rs`: robust script content loading with UTF-8 fallback,
  - `hashing.rs`: source-content SHA-256 generation,
  - `decorated.rs`: tree-sitter decorated-function extraction and `ToolRecord` assembly,
  - `mod.rs`: `ToolsScanner` parse entrypoints (`parse_script`, `parse_content`) and module wiring.
- Preserved public scanner API and behavior.
- Kept parser construction explicit and test-surface compatible.

### Structural outcome

- Baseline before split:
  - `skills/tools/parse.rs`: `220` lines.
- Current split sizes:
  - `skills/tools/parse/mod.rs`: `90`
  - `skills/tools/parse/content.rs`: `15`
  - `skills/tools/parse/decorated.rs`: `125`
  - `skills/tools/parse/hashing.rs`: `7`
  - total: `237` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/tools/parse -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-parse-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-parse-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-parse-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-tools-parse-split-it cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Metadata Core Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/metadata/core.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/core/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/core/skill_metadata.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/core/sniffer_rule.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/core/tool_annotations.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/core/decorator_args.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/core/tool_record.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/core/reference_path.rs`

### Quality changes adopted

- Split mixed-responsibility `core.rs` into focused domain modules:
  - `skill_metadata.rs` for SKILL frontmatter schema,
  - `sniffer_rule.rs` for activation rule data model,
  - `tool_annotations.rs` for MCP annotation model and helpers,
  - `decorator_args.rs` for parsed decorator kwargs,
  - `tool_record.rs` for tool metadata and enrichment payload,
  - `reference_path.rs` for validated reference path type.
- `core/mod.rs` now acts as interface-only module and re-export layer.
- Removed one accidental unused re-export during cleanup to keep the slice warning-free.

### Structural outcome

- Baseline before split:
  - `skills/metadata/core.rs`: `438` lines.
- Current split sizes:
  - `skills/metadata/core/mod.rs`: `15`
  - `skills/metadata/core/skill_metadata.rs`: `67`
  - `skills/metadata/core/sniffer_rule.rs`: `22`
  - `skills/metadata/core/tool_annotations.rs`: `104`
  - `skills/metadata/core/decorator_args.rs`: `25`
  - `skills/metadata/core/tool_record.rs`: `152`
  - `skills/metadata/core/reference_path.rs`: `61`
  - total: `446` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-skills/src/skills/metadata/core -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-core-split-clean cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-core-split-clean cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-core-split-clean-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-core-split-clean-it cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-skills` Metadata Records Module Decomposition)

### Changed files

- deleted: `packages/rust/crates/xiuxian-skills/src/skills/metadata/records.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/records/mod.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/records/template.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/records/reference.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/records/asset.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/records/data.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/records/test_record.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/records/resource.rs`
- `packages/rust/crates/xiuxian-skills/src/skills/metadata/records/prompt.rs`

### Quality changes adopted

- Split mixed-concern `records.rs` into domain-specific files by record type.
- Kept public module surface stable through `records/mod.rs` re-exports.
- Isolated serde helper logic for references into `reference.rs`.
- Outcome:
  - clearer ownership per record domain,
  - lower cognitive load for incremental changes,
  - better fit with project modularization rules (`mod.rs` as interface-only).

### Structural outcome

- Baseline before split:
  - `skills/metadata/records.rs`: `443` lines.
- Current split sizes:
  - `skills/metadata/records/mod.rs`: `17`
  - `skills/metadata/records/template.rs`: `48`
  - `skills/metadata/records/reference.rs`: `164`
  - `skills/metadata/records/asset.rs`: `39`
  - `skills/metadata/records/data.rs`: `48`
  - `skills/metadata/records/test_record.rs`: `48`
  - `skills/metadata/records/resource.rs`: `54`
  - `skills/metadata/records/prompt.rs`: `45`
  - total: `463` lines.

### Suppression debt delta (this slice)

- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates/xiuxian-skills/src/skills/metadata -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-records-split cargo check -p xiuxian-skills`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-records-split cargo clippy -p xiuxian-skills -- -W clippy::too_many_lines`
  - result: passed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-records-split-lib cargo test -p xiuxian-skills --lib -- --nocapture`
  - result: 58 passed, 0 failed.
- `CARGO_TARGET_DIR=.cache/target-xiuxian-skills-records-split-it cargo test -p xiuxian-skills --test tools_scanner -- --nocapture`
  - result: 45 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-vector` Missing-Errors-Doc Reduction in Scalar and Maintenance Ops)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/ops/scalar.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/maintenance.rs`

### Quality changes adopted

- Removed `clippy::missing_errors_doc` suppressions from two ops modules:
  - `ops/scalar.rs`,
  - `ops/maintenance.rs`.
- Added explicit `# Errors` documentation for every public `Result` API touched in these modules:
  - scalar index creation and cardinality estimation paths,
  - maintenance index-presence checks, auto-indexing, and compaction paths.
- Kept `clippy::doc_markdown` local allowance where already in use.
- Preserved runtime behavior; this slice is documentation-compliance and lint-governance hardening only.

### Suppression debt delta (this slice)

- `allow(clippy::missing_errors_doc)` count in Rust crates:
  - before: `8`
  - after: `6`
- Remaining scope after this slice:
  - `packages/rust/crates/xiuxian-vector/src/ops/admin_impl.rs`
  - `packages/rust/crates/xiuxian-wendao/src/sync/*.rs`
  - `packages/rust/crates/xiuxian-wendao/src/storage/mod.rs`

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
  - result: passed.
- `cargo check -p xiuxian-vector`
  - result: passed.
- `cargo clippy -p xiuxian-vector -- -W clippy::missing_errors_doc -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-vector --no-run`
  - result: passed (test binaries built successfully).

## Execution Evidence: 2026-02-25 (Slice `missing_errors_doc` Suppression Cleared Across Rust Crates)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/ops/admin_impl.rs`
- `packages/rust/crates/xiuxian-wendao/src/sync/discovery.rs`
- `packages/rust/crates/xiuxian-wendao/src/sync/diff.rs`
- `packages/rust/crates/xiuxian-wendao/src/sync/manifest.rs`
- `packages/rust/crates/xiuxian-wendao/src/sync/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/storage/mod.rs`

### Quality changes adopted

- Removed final `#[allow(clippy::missing_errors_doc)]` suppressions in production Rust code.
- Added explicit `# Errors` documentation on public `Result` APIs in `xiuxian-vector` admin ops.
- Removed stale suppressions from `xiuxian-wendao` modules where no public `Result` API required this lint bypass.
- Added `# Errors` to `SyncEngine::save_manifest` in `xiuxian-wendao` for explicit IO failure contracts.

### Suppression debt delta (this slice)

- `allow(clippy::missing_errors_doc)` count in Rust crates:
  - before: `6`
  - after: `0`
- Validation:
  - `rg -n "allow\\(clippy::missing_errors_doc" packages/rust/crates -g '*.rs'`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
  - result: passed.
- `cargo check -p xiuxian-vector`
  - result: passed.
- `cargo clippy -p xiuxian-vector -- -W clippy::missing_errors_doc -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-vector --no-run`
  - result: passed.
- `cargo fmt -p xiuxian-wendao`
  - result: passed.
- `cargo check -p xiuxian-wendao`
  - result: passed.
- `cargo clippy -p xiuxian-wendao -- -W clippy::missing_errors_doc -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-wendao --no-run`
  - result: passed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-wendao` Related-PPR Compute Context Refactor)

### Changed files

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/ppr/compute.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/ppr/compute/orchestrate.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/ppr/compute/finalize.rs`

### Quality changes adopted

- Introduced explicit context/config structures for PPR phases:
  - `RelatedPprKernelConfig` for kernel/orchestration runtime knobs,
  - `RelatedPprFinalizeContext` for diagnostics/finalization inputs.
- Removed broad argument lists from internal compute helpers:
  - `run_related_ppr_orchestration(..., &RelatedPprKernelConfig)`,
  - `finalize_related_ppr_result(..., &RelatedPprFinalizeContext, ...)`.
- Preserved ranking/diagnostics behavior and kept call flow unchanged in `related_ppr_compute`.

### Suppression debt delta (this slice)

- Removed function-level suppressions:
  - `packages/rust/crates/xiuxian-wendao/src/link_graph/index/ppr/compute/orchestrate.rs`
  - `packages/rust/crates/xiuxian-wendao/src/link_graph/index/ppr/compute/finalize.rs`
- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates/xiuxian-wendao/src/link_graph/index/ppr/compute/orchestrate.rs packages/rust/crates/xiuxian-wendao/src/link_graph/index/ppr/compute/finalize.rs`
  - result: no matches.

### Compile and test evidence

- `cargo fmt -p xiuxian-wendao`
  - result: passed.
- `cargo check -p xiuxian-wendao`
  - result: passed.
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines -W clippy::too_many_arguments`
  - result: passed.
- `cargo test -p xiuxian-wendao --test test_link_graph_ppr_weighting -- --nocapture`
  - result: 1 passed, 0 failed.
- `cargo test -p xiuxian-wendao --test test_link_graph_seed_and_priors -- --nocapture`
  - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-wendao` Search Runtime Policy Refactor)

### Changed files

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/context.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/plan/core.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/pipeline/collect.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/row_evaluator/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/row_evaluator/sections.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/strategy.rs`

### Quality changes adopted

- Added `SearchRuntimePolicy` to capture search execution knobs once:
  - scope/structural/semantic switches,
  - collapse policy and section caps,
  - heading and tree-hop limits.
- Added `DocScoreContext` to carry section/path scoring context into strategy scoring.
- Updated call chain to pass structured context instead of long primitive argument lists:
  - `collect_search_rows`,
  - `evaluate_doc_rows`,
  - `prepare_section_context`,
  - `score_doc_for_strategy`.
- Fixed follow-up pedantic warning by passing `DocScoreContext` by reference.

### Suppression debt delta (this slice)

- Removed function-level suppressions from:
  - `search/pipeline/collect.rs`
  - `search/row_evaluator/mod.rs`
  - `search/row_evaluator/sections.rs`
  - `search/strategy.rs`
- `allow(clippy::too_many_arguments)` count in Rust crates:
  - before: `31`
  - after: `25`
- Validation:
  - `rg -n "allow\\(clippy::too_many_arguments\\)" packages/rust/crates -g '*.rs' | wc -l`
  - result: `25`.

### Compile and test evidence

- `cargo fmt -p xiuxian-wendao`
  - result: passed.
- `cargo check -p xiuxian-wendao`
  - result: passed.
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-wendao --test test_link_graph search_core -- --nocapture`
  - result: 7 passed, 0 failed (48 filtered).

## Execution Evidence: 2026-02-25 (Slice `xiuxian-wendao` Search Section and Structured-Filter Argument Refactor)

### Changed files

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/score/sections.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/structured_filters/matches.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/row_evaluator/sections.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/row_evaluator/prefilter.rs`

### Quality changes adopted

- Replaced long argument lists with existing execution contexts:
  - `section_candidates` now accepts `SearchExecutionContext` + `SearchRuntimePolicy`,
  - `matches_structured_filters` now accepts `SearchExecutionContext`.
- Updated call sites in row-evaluator modules to pass context objects directly.
- Preserved filter/section scoring semantics; this slice only changes parameter plumbing and API hygiene.

### Suppression debt delta (this slice)

- Removed function-level suppressions:
  - `search/score/sections.rs`
  - `search/structured_filters/matches.rs`
- `allow(clippy::too_many_arguments)` count:
  - Rust crates overall: `25` -> `23`
  - `xiuxian-wendao/src`: `4` -> `2`
- Remaining `xiuxian-wendao/src` scope:
  - `packages/rust/crates/xiuxian-wendao/src/bin/wendao/execute/agentic/plan_run.rs`
  - `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/retrieval_plan.rs`

### Compile and test evidence

- `cargo fmt -p xiuxian-wendao`
  - result: passed.
- `cargo check -p xiuxian-wendao`
  - result: passed.
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_arguments -W clippy::too_many_lines`
  - result: passed.
- `cargo test -p xiuxian-wendao --test test_link_graph search_core -- --nocapture`
  - result: 7 passed, 0 failed (48 filtered).
- `cargo test -p xiuxian-wendao --test test_link_graph_ppr_weighting -- --nocapture`
  - result: 1 passed, 0 failed.
- `cargo test -p xiuxian-wendao --test test_link_graph_seed_and_priors -- --nocapture`
  - result: 2 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-zhixing` API Hygiene and Documentation Compliance)

### Changed files

- `packages/rust/crates/xiuxian-zhixing/src/alchemist/processor.rs`
- `packages/rust/crates/xiuxian-zhixing/src/heyi/mod.rs`
- `packages/rust/crates/xiuxian-zhixing/src/storage/markdown.rs`
- `packages/rust/crates/xiuxian-zhixing/src/lib.rs`
- `packages/rust/crates/xiuxian-zhixing/src/alchemist/mod.rs`
- `packages/rust/crates/xiuxian-zhixing/src/agenda/entry.rs`
- `packages/rust/crates/xiuxian-zhixing/tests/test_strict_teacher.rs`

### Quality changes adopted

- Removed unnecessary `Result` return wrappers in non-fallible APIs:
  - `AlchemistProcessor::process` now uses a no-return placeholder API,
  - `ZhixingHeyi::sync_from_disk` now returns unit and keeps observability logging.
- Added missing documentation coverage and markdown style fixes:
  - module docs for crate-level public modules in `lib.rs`,
  - processor module docs in `alchemist/mod.rs`,
  - backtick fixes for type identifiers (`LlmInterface`, `MarkdownStorage`).
- Added explicit `# Errors` sections for public fallible APIs:
  - `ZhixingHeyi::check_heart_demon_blocker`,
  - `ZhixingHeyi::render_agenda`,
  - `MarkdownStorage::record_task`.
- Removed pedantic test-quality debt:
  - replaced strict float equality check with epsilon comparison,
  - refactored integration test to avoid `unwrap`/`expect` and return `anyhow::Result<()>`,
  - updated strict-teacher test to new `ZhixingHeyi::new` constructor contract.

### Warning/suppression delta (this slice)

- No new suppressions introduced.
- Workspace tracked suppression categories remain fully cleared:
  - `allow(clippy::too_many_arguments)`: `0`
  - `allow(clippy::too_many_lines)`: `0`
  - `allow(clippy::missing_errors_doc)`: `0`

### Compile and test evidence

- `cargo fmt -p xiuxian-zhixing`
  - result: passed.
- `cargo check -p xiuxian-zhixing`
  - result: passed.
- `cargo clippy -p xiuxian-zhixing --all-targets -- -W missing_docs -W clippy::doc_markdown -W clippy::unnecessary_wraps -W clippy::unused_async`
  - result: passed.
- `cargo test -p xiuxian-zhixing -- --nocapture`
  - result: 4 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-vector` Test Reliability and Clippy-Driven Error Handling Hardening)

### Changed files

- `packages/rust/crates/xiuxian-vector/src/checkpoint/timeline_tests.rs`
- `packages/rust/crates/xiuxian-vector/src/keyword/entity_aware_tests.rs`
- `packages/rust/crates/xiuxian-vector/src/keyword/fusion/match_util.rs`
- `packages/rust/crates/xiuxian-vector/src/ops/column_read.rs`
- `packages/rust/crates/xiuxian-vector/src/search/search_impl/tests.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_keyword_index.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_merge_insert.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_skill_index_robustness.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_list_all_tools.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_vector_index.rs`

### Quality changes adopted

- Replaced panic-style test patterns (`unwrap` / `expect`) with explicit error propagation:
  - converted integration/unit tests to `Result<()>` + `?`,
  - used explicit `match` + `panic!` only for negative assertions where failure is required.
- Added reusable test helpers for stable path creation and JSON parsing:
  - normalized path handling via `to_string_lossy` to avoid `to_str().unwrap()` in tests,
  - centralized JSON row decode in `test_list_all_tools`.
- Removed test-level pedantic debt in touched files:
  - replaced strict float equality with epsilon assertions where appropriate,
  - modernized `format!` usage and literal readability in timeline/vector-index tests,
  - fixed include-based test module structure for `entity_aware_tests` to avoid nested module anti-pattern.
- Preserved behavior/contracts:
  - no production runtime behavior changes in this slice,
  - changes are limited to test reliability, diagnostics, and lint-compliance practices.

### Compile and test evidence

- `cargo fmt -p xiuxian-vector`
  - result: passed.
- `cargo test -p xiuxian-vector timeline -- --nocapture`
  - result: timeline tests passed (5/5).
- `cargo test -p xiuxian-vector entity_aware -- --nocapture`
  - result: entity-aware tests passed (5/5 matched).
- `cargo test -p xiuxian-vector search_results_to_ipc -- --nocapture`
  - result: search IPC tests passed (6/6 matched).
- `cargo test -p xiuxian-vector --test test_keyword_index -- --nocapture`
  - result: 8 passed, 0 failed.
- `cargo test -p xiuxian-vector --test test_merge_insert -- --nocapture`
  - result: 1 passed, 0 failed.
- `cargo test -p xiuxian-vector --test test_skill_index_robustness -- --nocapture`
  - result: 2 passed, 0 failed.
- `cargo test -p xiuxian-vector --test test_list_all_tools -- --nocapture`
  - result: 7 passed, 0 failed.
- `cargo test -p xiuxian-vector --test test_vector_index -- --nocapture`
  - result: 6 passed, 0 failed.

### Remaining scope after this slice

- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic` still reports
  remaining issues in other untouched test targets (notably:
  `test_rust_cortex`, `test_fusion`, `test_hybrid_search`, `test_search_perf_guard`).
- Next recommended wave: continue test-only unwrap/expect elimination in
  `test_rust_cortex` first (highest concentration), then clean doc-markdown and pedantic warnings.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-vector` Pedantic Burn-Down Wave 2 - Test Suite)

### Changed files

- `packages/rust/crates/xiuxian-vector/tests/test_rust_cortex.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_search_cache.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_fusion.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_hybrid_search.rs`
- `packages/rust/crates/xiuxian-vector/tests/test_skill_scanner.rs`

### Quality changes adopted

- Completed `test_rust_cortex` migration to `Result<()>` test style:
  - repaired incomplete mechanical conversion,
  - added missing `Ok(())` tails,
  - removed remaining pedantic warnings (`doc_markdown`, `float_cmp`,
    `type_complexity`, `useless_vec`) without adding suppressions.
- Eliminated panic-style cache test assertions in `test_search_cache`:
  - replaced `unwrap`-based checks with explicit `Option` equality assertions.
- Hardened fusion algorithm tests (`test_fusion`) for clippy pedantic:
  - removed `unwrap` from result lookups,
  - replaced strict float equality with epsilon assertions,
  - normalized doc comments and `format!` usage.
- Refactored `test_hybrid_search` to modern error boundaries:
  - migrated all async tests to `Result<()> + ?`,
  - normalized path handling (`to_string_lossy`) to remove `to_str().unwrap()`,
  - resolved raw-string hash noise and loop formatting/casting warnings.
- Reworked `test_skill_scanner` for robust test error handling:
  - migrated to `TestResult` (`Result<(), Box<dyn Error>>`) to match scanner
    API error model,
  - replaced unwrap/expect path traversal and metadata lookups with typed
    error propagation.

### Compile and test evidence

- `cargo check -p xiuxian-vector --test test_rust_cortex`
  - result: passed.
- `cargo clippy -p xiuxian-vector --test test_rust_cortex -- -W clippy::pedantic`
  - result: passed (no warnings).
- `cargo test -p xiuxian-vector --test test_rust_cortex -- --nocapture`
  - result: 19 passed, 0 failed.
- `cargo clippy -p xiuxian-vector --test test_search_cache -- -W clippy::pedantic`
  - result: passed.
- `cargo test -p xiuxian-vector --test test_search_cache -- --nocapture`
  - result: 3 passed, 0 failed.
- `cargo clippy -p xiuxian-vector --test test_fusion -- -W clippy::pedantic`
  - result: passed.
- `cargo test -p xiuxian-vector --test test_fusion -- --nocapture`
  - result: 16 passed, 0 failed.
- `cargo clippy -p xiuxian-vector --test test_hybrid_search -- -W clippy::pedantic`
  - result: passed.
- `cargo test -p xiuxian-vector --test test_hybrid_search -- --nocapture`
  - result: 9 passed, 0 failed.
- `cargo clippy -p xiuxian-vector --test test_skill_scanner -- -W clippy::pedantic`
  - result: passed.
- `cargo test -p xiuxian-vector --test test_skill_scanner -- --nocapture`
  - result: 7 passed, 0 failed.

### Remaining scope after this slice

- `cargo clippy -p xiuxian-vector --tests -- -W clippy::pedantic` still reports
  remaining failures in untouched test targets, especially:
  - `test_vector_benchmark`,
  - `test_search_perf_guard`,
  - `test_store`,
  - `test_observability`,
  - `test_maintenance`.
- Next recommended wave: continue unwrap/expect elimination in those test files,
  starting with `test_search_perf_guard` + `test_vector_benchmark` for highest
  signal-to-effort ratio.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-vector` Search Perf Guard Pedantic Hardening)

### Changed files

- `packages/rust/crates/xiuxian-vector/tests/test_search_perf_guard.rs`

### Quality changes adopted

- Migrated perf guard async test path to `Result<()> + ?`:
  - removed panic-style `unwrap` on tempdir, vector store init, ingestion, and query execution.
- Reworked numeric helpers to reduce pedantic casting debt:
  - `synthetic_vector` now uses checked integer conversion (`u16::try_from` + `f32::from`),
  - percentile index calculation now uses integer arithmetic (numerator/denominator form),
  - average calculation now uses checked `u32` conversion before `f64` division.
- Cleaned profile metrics model for `struct_field_names` lint:
  - renamed `avg_ms/p50_ms/p95_ms` to `avg/p50/p95`.
- Kept benchmark semantics unchanged (same dataset/query cardinality and guardrails).

### Compile and test evidence

- `cargo clippy -p xiuxian-vector --test test_search_perf_guard -- -W clippy::pedantic`
  - result: passed.
- `cargo test -p xiuxian-vector --test test_search_perf_guard -- --nocapture`
  - result: 1 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-vector` Vector Benchmark Clippy Hardening)

### Changed files

- `packages/rust/crates/xiuxian-vector/tests/test_vector_benchmark.rs`

### Quality changes adopted

- Removed remaining unwrap-based filter assertion in JSON filtering benchmark:
  - extracted `skill_name` filter once via explicit `let-else` guard.
- Collapsed nested filtering conditions into a single `if let` chain and
  removed redundant `continue`.
- Replaced precision-loss and formatting lint hotspots:
  - `f64::from(i)` instead of direct cast in score synthesis,
  - inline `format!` argument style (`vector_{i}` and `file_{i}.py`).
- Replaced strict float equality assertions with epsilon-based checks in
  L2 correctness tests.

### Compile and test evidence

- `cargo clippy -p xiuxian-vector --test test_vector_benchmark -- -W clippy::pedantic`
  - result: passed.
- `cargo test -p xiuxian-vector --test test_vector_benchmark -- --nocapture`
  - result: 8 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-vector` Search Tests Pedantic Hardening)

### Changed files

- `packages/rust/crates/xiuxian-vector/tests/test_search.rs`

### Quality changes adopted

- Removed panic-prone sort ordering in score ranking check:
  - replaced `partial_cmp(...).unwrap()` with `total_cmp`.
- Eliminated strict float equality in keyword-distance and score checks:
  - switched to epsilon-based assertions for stable numeric comparisons.
- Removed low-value lint debt in test fixture setup:
  - replaced manual empty-string construction with `String::new()`,
  - removed no-effect underscore bindings and unnecessary `vec!` usage in
    vector-distance test setup.

### Compile and test evidence

- `cargo clippy -p xiuxian-vector --test test_search -- -W clippy::pedantic`
  - result: passed.
- `cargo test -p xiuxian-vector --test test_search -- --nocapture`
  - result: 16 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-vector` Migration Tests Error-Boundary Hardening)

### Changed files

- `packages/rust/crates/xiuxian-vector/tests/test_migration.rs`

### Quality changes adopted

- Eliminated panic-style migration test setup:
  - converted helper functions and async test to `Result`-based flow (`?`).
- Hardened schema-fixture builders:
  - `dict_from_strings` now returns `Result<DictionaryArray<_>>`,
  - key/index conversions use checked integer conversions (`try_from`) instead
    of unchecked casts and panic-based creation.
- Improved migration fixture clarity and lint compliance:
  - fixed `doc_markdown` issues for `TOOL_NAME`/`skill_name`,
  - replaced `iter().cloned().collect()` with `to_vec()`,
  - removed all `unwrap`/`expect` usage in dataset write/open/migrate checks.

### Compile and test evidence

- `cargo clippy -p xiuxian-vector --test test_migration -- -W clippy::pedantic`
  - result: passed.
- `cargo test -p xiuxian-vector --test test_migration -- --nocapture`
  - result: 1 passed, 0 failed.

## Execution Evidence: 2026-02-25 (Slice `xiuxian-vector` Observability Tests Error-Boundary Hardening)

### Changed files

- `packages/rust/crates/xiuxian-vector/tests/test_observability.rs`

### Quality changes adopted

- Converted observability test helpers and async tests to `Result<()> + ?`:
  - removed all panic-style `unwrap`/`unwrap_err` usage in setup, report calls,
    and metrics reads.
- Improved path handling reliability:
  - replaced `to_str().unwrap()` with owned UTF-8-safe path conversion using
    `to_string_lossy().into_owned()`.
- Cleaned pedantic lint debt in test text/formatting:
  - doc comment backticks for `analyze_table_health`,
  - inline `format!` argument usage in fixture generation.
- Preserved all behavioral assertions (recommendations, report shape, metrics,
  and not-found error contract).

### Compile and test evidence

- `cargo clippy -p xiuxian-vector --test test_observability -- -W clippy::pedantic`
  - result: passed.
- `cargo test -p xiuxian-vector --test test_observability -- --nocapture`
  - result: 5 passed, 0 failed.
