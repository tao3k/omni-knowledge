# 522. Xiuxian Wendao Saliency Mget Batching

Date: 2026-03-11

## Scope

Reduce Valkey MGET response size for saliency state reads by batching and add coverage for large candidate sets.

## What Changed

- Added `VALKEY_SALIENCY_MGET_BATCH_SIZE` and chunked `MGET` in `valkey_saliency_get_many_with_valkey`.
- Added a Valkey integration test that touches 600 nodes and validates batch retrieval.

## Validation Evidence

Executed and passed:

```bash
cargo nextest run -p xiuxian-wendao -E "test(test_saliency_get_many_batches)"
```

Outcome: passed with existing warnings about missing docs and unused test helpers.

Executed and failed:

```bash
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Outcome: failed with 6 existing errors (unwrap_used in `graph/core.rs` and `graph/dedup/operations.rs`) and 171 warnings across the crate and workspace dependencies.
