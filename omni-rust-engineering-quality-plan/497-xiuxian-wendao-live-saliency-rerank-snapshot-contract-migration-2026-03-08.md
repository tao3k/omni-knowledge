# 497. Xiuxian Wendao Live Saliency Rerank Snapshot Contract Migration

Date: 2026-03-08

## Scope

This shard records two related changes in `xiuxian-wendao`:

- Wendao search now consumes learned live saliency during final ranking.
- The new live saliency regression was migrated into the repository's snapshot-style contract pattern, and the superseded direct assertion test was removed.

## Why This Change Was Needed

Wendao already had most of the Route A "living brain" machinery:

- search hits were already touching saliency state asynchronously,
- GraphMem bootstrap already seeded document saliency into Valkey,
- edge weights already mirrored saliency into outbound ZSET scores.

But the primary search ranking path still had a blind spot: it did not consume learned saliency state during final result ordering. That meant the system could learn, but not yet use what it learned.

At the same time, the first live-rerank regression had been added as a direct assertion integration test. That did not match the repository's ongoing fixture/snapshot migration standard.

## What Changed

### 1) Added batched saliency reads

New read surface:

- `valkey_saliency_get_many(...)`
- `valkey_saliency_get_many_with_valkey(...)`

Why this matters:

- search finalization can load candidate saliency state in one Valkey round trip,
- invalid payload cleanup still happens best-effort,
- the live rerank path avoids one network lookup per candidate.

### 2) Search finalization now applies learned live saliency boosts

Updated:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/pipeline/finalize.rs`

Key behavior:

- the final ranking stage loads candidate saliency states in batch,
- only learned state (`activation_count > 0`) contributes a boost,
- seeded/default saliency does not perturb baseline ranking,
- boosted hits append `live_saliency` to `match_reason`.

Why this matters:

- Wendao now closes the learning loop in the user-visible retrieval path,
- the feature remains low-risk because untouched documents keep baseline behavior,
- the new signal composes cleanly with existing provisional agentic boosts.

### 3) Migrated the live-rerank regression into snapshot-style contract coverage

Added:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/pipeline/tests.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/live_saliency_rerank/learned_saliency_reorders_hits/expected/result.json`

Updated:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/pipeline/mod.rs`

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_live_saliency_rerank.rs`

Why this matters:

- the regression now follows the repository's fixture-backed contract style,
- expected output is stored as stable JSON rather than scattered direct assertions,
- the original migrated test was removed immediately after replacement, keeping the suite tidy.

## Architectural Takeaways

### Learned saliency should be consumed only after it becomes learned

Using `activation_count > 0` as the eligibility gate keeps the feature aligned with the intended Hebbian behavior. Seeded bootstrap state is initialization data, not evidence of user- or retrieval-driven reinforcement.

### Batch reads belong at the ranking boundary

The ranking stage owns the final candidate set, so it is the correct layer to batch-load live saliency state. This keeps the rest of search planning and scoring free from unnecessary runtime coupling.

### Snapshot migration is the right fit for deterministic ranking deltas

The live-rerank behavior is small, stable, and output-oriented. A fixture-backed JSON contract captures it more clearly than a direct assertion test and fits the crate's broader migration direction.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/src/lib.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/saliency/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/saliency/store/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/saliency/store/read.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/pipeline/finalize.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/pipeline/mod.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/index/search/pipeline/tests.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_saliency/saliency_get_many_with_valkey.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/live_saliency_rerank/learned_saliency_reorders_hits/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_live_saliency_rerank.rs` (removed)

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-live-saliency cargo check -p xiuxian-wendao --tests --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-live-saliency NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_link_graph_saliency
CARGO_TARGET_DIR=/tmp/xiuxian-live-saliency NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao live_saliency_rerank_snapshot_reorders_hits
CARGO_TARGET_DIR=/tmp/xiuxian-live-saliency cargo clippy -p xiuxian-wendao --tests -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_link_graph_saliency` passed (`6 passed, 0 skipped`).
- `cargo nextest run -p xiuxian-wendao live_saliency_rerank_snapshot_reorders_hits` passed (`1 passed, 351 skipped`).
- `cargo clippy -p xiuxian-wendao --tests -- -W clippy::too_many_lines` completed successfully.
- `cargo clippy ...` still reported pre-existing unrelated pedantic warnings in:
  - `packages/rust/crates/xiuxian-wendao/tests/test_link_graph_hybrid_benchmark/...`
  - `packages/rust/crates/xiuxian-wendao/tests/dependency_indexer_indexer_unit/support.rs`
  These warnings were not introduced by this change.

## Artifacts and Notes

- Active plan: `.cache/codex/execplans/wendao-live-saliency-rerank.md`
- Snapshot contract fixture:
  - `packages/rust/crates/xiuxian-wendao/tests/fixtures/link_graph/live_saliency_rerank/learned_saliency_reorders_hits/expected/result.json`
