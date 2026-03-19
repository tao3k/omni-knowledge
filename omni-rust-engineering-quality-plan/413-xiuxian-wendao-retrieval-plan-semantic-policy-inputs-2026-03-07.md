# 413. Wendao Retrieval-Plan Semantic Policy Inputs

Date: 2026-03-07

## Scope

This shard records the fifth pure-Rust hybrid-retriever slice for `xiuxian-wendao`.

The slice extends Wendao planning surfaces so retrieval plans can carry semantic ignition policy as first-class input and output data.

## Why This Change Was Needed

Phase 4 added semantic-policy behavior in the semantic ignition seam and vector adapter, but Wendao planning still had a blind spot:

- retrieval planning could choose graph-only, hybrid, or vector-only,
- retrieval-plan telemetry could not express which semantic policy would apply if vector escalation ran,
- callers had no canonical Wendao-side input surface for `summary_only` or `min_vector_score`.

That meant semantic policy existed only at the seam and adapter layers. The plan layer could not describe or track it.

## What Changed

### 1) Wendao search options now carry semantic ignition policy

`LinkGraphSearchOptions` now includes:

- `semantic_policy: LinkGraphSemanticSearchPolicy`

Current semantic-policy contract:

- `document_scope: LinkGraphSemanticDocumentScope`
- `min_vector_score: Option<f64>`

Current supported scope values:

- `All`
- `SummaryOnly`

Why this matters:

- Wendao planning can now represent semantic policy explicitly,
- this keeps semantic policy out of graph filters,
- the plan input remains backend-agnostic.

### 2) Query directives now support semantic policy

`parse_search_query` now understands:

- `semantic_scope:summary`
- `summary_only:true`
- `semantic_min_vector_score:0.55`
- `min_vector_score:0.55`

Why this matters:

- CLI and text-query entry points can now express semantic policy without a separate wrapper layer,
- the policy enters Wendao through the same normalized planning path as other query directives.

### 3) Retrieval-plan records now snapshot semantic policy

`LinkGraphRetrievalPlanRecord` now includes:

- `semantic_policy: LinkGraphSemanticSearchPolicy`

This field is populated through `LinkGraphRetrievalPlanInput` and attached during policy evaluation.

Why this matters:

- the retrieval plan now documents the full hybrid contract,
- downstream telemetry and debugging can see which semantic policy would have applied,
- the semantic policy is traceable even when graph-only routing wins.

### 4) Runtime defaults now merge into planning cleanly

`LinkGraphRetrievalPolicyRuntimeConfig` now includes a default `semantic_policy` block.

Current runtime ingress:

- settings: `link_graph.semantic.summary_only`
- settings: `link_graph.semantic.min_vector_score`
- env: `XIUXIAN_WENDAO_LINK_GRAPH_SEMANTIC_SUMMARY_ONLY`
- env: `XIUXIAN_WENDAO_LINK_GRAPH_SEMANTIC_MIN_VECTOR_SCORE`

Request-local policy merges with runtime defaults during retrieval-plan evaluation.

Why this matters:

- runtime configuration remains the default policy source,
- explicit request options still override cleanly,
- the merge happens at the planning boundary, which is the correct ownership layer.

### 5) The retrieval-plan schema is now aligned with the new contract

`resources/omni.link_graph.retrieval_plan.v1.schema.json` now includes the `semantic_policy` object.

Why this matters:

- schema registry consumers see the same contract that Rust now emits,
- the retrieval-plan resource is not left in a stale state after the model change.

## Architectural Takeaways

### Keep plan inputs separate from graph filters

`summary_only` and `min_vector_score` do not change graph matching, graph traversal, or graph scoring. They are semantic escalation controls.

Putting them into `LinkGraphSearchFilters` would blur boundaries and make the graph layer own behavior that belongs to hybrid planning.

### Let the plan own the execution contract

The retrieval plan should not only say whether graph or vector won. It should also describe the semantic policy that governs vector execution when escalation is allowed.

This is a modern Rust engineering pattern:

- input model expresses capability,
- plan record snapshots the resolved contract,
- lower execution layers implement that contract.

### Merge defaults at the owning layer

Runtime defaults and request-local overrides meet in retrieval-plan evaluation.

That is the correct place because:

- the plan owns routing policy,
- the adapter should execute policy, not invent it,
- callers can see the resolved semantic policy in the emitted plan record.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/semantic_policy.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/query/options.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/models/records/retrieval_plan.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/query/parse/state.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/query/parse/merge.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/query/parse/scan/directives/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/query/parse/scan/directives/semantic.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/plan/payload/core.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/plan/payload/policy.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/constants.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/models.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/policy.rs`
- `packages/rust/crates/xiuxian-wendao/resources/omni.link_graph.retrieval_plan.v1.schema.json`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/semantic_policy.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/link_graph_parse_search_query_supports_semantic_policy_directives.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/query_parsing/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/search_core.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_cli/search/basic/search_verbose_includes_monitor_summary.rs`

## Validation Evidence

Executed and passed:

```bash
cargo fmt -p xiuxian-wendao
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-plan-semantic cargo check -p xiuxian-wendao
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-plan-semantic cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-plan-semantic NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph semantic_policy
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-plan-semantic NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph test_link_graph_search_planned_payload_has_consistent_counts
CARGO_TARGET_DIR=/tmp/xiuxian-wendao-plan-semantic NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_wendao_cli search_verbose_includes_monitor_summary
```

Observed outcomes:

- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` completed cleanly.
- The semantic-policy lane passed (`3 passed`).
- The existing planned-payload consistency test passed after semantic-policy assertions were added.
- The verbose CLI smoke test passed with `retrieval_plan.semantic_policy` exposed in the serialized payload.

## Next Step

The next clean Rust-only step is to bridge `LinkGraphSemanticSearchPolicy` into concrete `xiuxian-wendao-vector` execution wiring so the plan record and the runtime ignition path share one canonical semantic-policy model end to end.
