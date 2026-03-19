# 414. Wendao Vector Retrieval-Plan Execution Bridge

Date: 2026-03-07

## Scope

This shard records the sixth pure-Rust hybrid-retriever slice.

The slice closes the gap between Wendao planning artifacts and `xiuxian-wendao-vector` execution setup by making the adapter bridge understand retrieval plans directly.

## Why This Change Was Needed

Phase 5 made `LinkGraphRetrievalPlanRecord` the canonical place where Wendao snapshots semantic ignition policy.

But the execution side still had a coordination gap:

- callers had to manually extract `retrieval_plan.semantic_policy`,
- callers still had to remember that `GraphOnly` means no semantic stage should run,
- the canonical planning artifact was not yet the canonical execution input.

That is the kind of manual glue modern Rust engineering should remove.

## What Changed

### 1) A dedicated execution bridge module now owns the translation

`packages/rust/crates/xiuxian-wendao-vector/src/adapter/bridge.rs` now owns the plan-to-execution conversion logic.

Key public surfaces:

- `WendaoVectorDocumentScope::from_link_graph_document_scope`
- `WendaoVectorSemanticIgnitionConfig::with_link_graph_semantic_policy`
- `WendaoVectorSemanticExecutionPlan`

Why this matters:

- planning-to-execution conversion is now explicit and centralized,
- callers no longer need to duplicate semantic-policy normalization logic,
- adapter boundaries stay clean and testable.

### 2) The bridge now accepts retrieval plans directly

`WendaoVectorSemanticExecutionPlan::from_retrieval_plan` now accepts `&LinkGraphRetrievalPlanRecord` and returns `Option<Self>`.

Current behavior:

- `GraphOnly` -> `None`
- `Hybrid` -> `Some(WendaoVectorSemanticExecutionPlan)`
- `VectorOnly` -> `Some(WendaoVectorSemanticExecutionPlan)`

Why this matters:

- graph-only routing is short-circuited at the correct ownership boundary,
- callers can consume the canonical retrieval-plan artifact directly,
- the bridge now respects both plan policy and plan mode.

### 3) Semantic request construction stays coupled to the normalized plan

`WendaoVectorSemanticExecutionPlan` still builds:

- normalized adapter config via `config()` / `into_config()`
- ready-to-run adapter via `build_ignition()`
- normalized semantic requests via `request(...)`

The request inherits `min_vector_score` from the normalized plan policy.

Why this matters:

- the execution path uses the same normalized policy snapshot that the plan recorded,
- score thresholds and summary-only scope are not re-derived ad hoc at call sites.

### 4) Focused integration tests now cover plan-mode gating

`packages/rust/crates/xiuxian-wendao-vector/tests/test_bridge.rs` now covers:

- semantic-policy to config/request mapping,
- graph-only short-circuiting from retrieval-plan mode,
- hybrid retrieval-plan conversion,
- end-to-end adapter execution with summary-only scope and score threshold.

Why this matters:

- the adapter bridge is verified as an ownership boundary, not just a bag of helper functions,
- the test surface reflects the real planning contract Wendao emits.

## Architectural Takeaways

### Accept the canonical upstream artifact

When a planning layer already emits a typed record, downstream execution code should accept that record instead of encouraging field-by-field extraction.

This reduces glue code and prevents drift.

### Short-circuit valid non-execution paths without pretending they are errors

`GraphOnly` is not a failure case for vector execution.

Returning `Option<Self>` is the right model because the plan validly says that no semantic stage should exist.

### Keep translation logic inside one ownership boundary

The adapter crate should own the mapping from Wendao planning semantics into vector-execution semantics.

That keeps `xiuxian-wendao` focused on planning/orchestration and keeps higher-level runtime crates from growing their own unofficial bridge logic.

## Files Changed

- `packages/rust/crates/xiuxian-wendao-vector/src/adapter/bridge.rs`
- `packages/rust/crates/xiuxian-wendao-vector/tests/test_bridge.rs`
- `packages/rust/crates/xiuxian-wendao-vector/README.md`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao-vector
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-vector-bridge cargo check -p xiuxian-wendao-vector
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-vector-bridge cargo clippy -p xiuxian-wendao-vector -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-vector-bridge NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao-vector
```

Observed outcomes:

- `cargo check -p xiuxian-wendao-vector` completed cleanly.
- `cargo clippy -p xiuxian-wendao-vector -- -W clippy::too_many_lines` completed cleanly.
- `cargo nextest run -p xiuxian-wendao-vector` passed all adapter lanes (`13 passed`).

## Next Step

The next clean Rust-only step is to wire this execution bridge into an actual Wendao runtime/search entry point so retrieval plans can launch semantic ignition without each caller rebuilding the execution contract manually.
