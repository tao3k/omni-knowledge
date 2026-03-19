# 412. Wendao Semantic Policy Filters and Score Gates

Date: 2026-03-07

## Scope

This shard records the fourth pure-Rust hybrid-retriever slice for `xiuxian-wendao` and `xiuxian-wendao-vector`.

The slice adds two missing semantic-policy controls:

- adapter-level document filtering for vector ignition,
- request-level minimum semantic score gating before topology diffusion.

## Why This Change Was Needed

Phase 3 established a clean adapter seam, but semantic ignition still lacked two controls that matter in production retrieval:

- callers could not restrict vector ignition to summary rows or another typed metadata subset,
- Wendao had no explicit contract for discarding weak semantic anchors before spending graph-diffusion budget.

Without these controls, hybrid retrieval would either over-fetch noisy anchors or force callers to hand-roll backend-specific filter strings outside the adapter boundary.

## What Changed

### 1) Semantic score gating moved into the semantic request contract

`QuantumSemanticSearchRequest` now carries:

- `min_vector_score: Option<f64>`

This is the correct boundary because score gating belongs to semantic ignition, not to post-hoc topology fusion.

Normalization now:

- trims `query_text`,
- enforces `limit >= 1`,
- clamps `min_vector_score` into `[0.0, 1.0]`.

The request also exposes `allows_vector_score()` so both the adapter and Wendao orchestration can share one canonical check.

### 2) Wendao now guards topology diffusion against weak anchors

`LinkGraphIndex::quantum_contexts_from_semantic_ignition` now defensively filters anchors after backend search and before calling `quantum_contexts_from_anchors`.

Why this matters:

- backends may optimize differently or ignore the threshold entirely,
- Wendao must preserve correctness even if a backend returns over-broad anchor sets,
- topology diffusion should only run for anchors that satisfy the declared semantic-entry contract.

### 3) The vector adapter now owns typed semantic filtering

`WendaoVectorSemanticIgnitionConfig` now includes:

- `metadata_filter: Map<String, Value>`
- `document_scope: WendaoVectorDocumentScope`

Current document-scope support:

- `All`
- `DocType(String)`
- builder sugar: `with_summary_only()`

Why this matters:

- `doc_type` is a metadata semantic, not a Wendao orchestration concern,
- the adapter can serialize this typed filter into `xiuxian-vector::SearchOptions.where_filter` without leaking raw Lance filter strings,
- the config remains composable and auditable.

### 4) Conflicting semantic policies now fail fast

The adapter adds a dedicated error:

- `WendaoVectorSemanticIgnitionError::ConflictingMetadataFilter`

This prevents silent behavior such as:

- caller asks for `doc_type = section` in the metadata filter,
- caller also enables `with_summary_only()`,
- adapter quietly picks one and drops the other.

Fail-fast is the correct Rust engineering choice here because retrieval policy conflicts should surface immediately and deterministically.

## Architectural Takeaways

### Keep each policy at the layer that owns it

- Semantic-entry score gating belongs to the semantic request.
- Vector-row metadata scoping belongs to the vector adapter config.
- Topology fusion weights still belong to `QuantumFusionOptions`.

This keeps the API honest and avoids overloading one config object with unrelated concerns.

### Prefer typed composition over string concatenation

The adapter could have exposed a raw `String` filter field. That would match the lower-level vector API, but it would also encourage unsafe or ambiguous composition.

Using a typed metadata filter instead:

- preserves the semantic meaning of the policy,
- keeps `doc_type` composition explicit,
- allows conflict detection before runtime search execution.

### Defensively enforce contracts at the orchestration boundary

Even when a backend can implement the threshold directly, Wendao still re-checks the policy before topology diffusion.

This is the higher-quality Rust pattern:

- let lower layers optimize,
- let the owning layer enforce the contract.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/quantum_fusion.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/quantum_fusion/semantic_ignition.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_ignition.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/adapter/config.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/adapter/error.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/adapter/mod.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/adapter/policy.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/adapter/runtime.rs`
- `packages/rust/crates/xiuxian-wendao-vector/src/lib.rs`
- `packages/rust/crates/xiuxian-wendao-vector/tests/test_runtime.rs`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao -p xiuxian-wendao-vector
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-phase4 cargo check -p xiuxian-wendao -p xiuxian-wendao-vector
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-phase4 cargo clippy -p xiuxian-wendao -p xiuxian-wendao-vector -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-phase4 NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao-vector
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-phase4 NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_ignition
```

Observed outcomes:

- `xiuxian-wendao-vector` nextest lane passed (`8 passed`).
- `xiuxian-wendao` semantic ignition lane passed (`4 passed`).
- `cargo clippy -p xiuxian-wendao -p xiuxian-wendao-vector -- -W clippy::too_many_lines` completed cleanly.

## Next Step

The next clean Rust-only step is to surface these semantic-policy knobs through Wendao retrieval-plan selection so hybrid retrieval can choose summary-only or thresholded semantic ignition without custom caller wiring.
