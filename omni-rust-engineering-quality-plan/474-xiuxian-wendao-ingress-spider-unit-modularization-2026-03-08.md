# 474. Xiuxian Wendao `ingress_spider_unit` Modularization

Date: 2026-03-08

## Scope

This shard records the structural decomposition of
`packages/rust/crates/xiuxian-wendao/tests/ingress_spider_unit.rs`.

## Why This Change Was Needed

The integration test entrypoint mixed URI canonicalization, bridge ingestion,
knowledge-graph persistence, and test-only sink/reindex fixtures in one file.

That obscured the boundary between protocol-level URI contracts and bridge-side
assimilation behavior.

## What Changed

Reduced the root integration-test entrypoint to a thin launcher:

- `packages/rust/crates/xiuxian-wendao/tests/ingress_spider_unit.rs`

Created a dedicated directory module:

- `packages/rust/crates/xiuxian-wendao/tests/ingress_spider_unit/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/ingress_spider_unit/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/ingress_spider_unit/uri.rs`
- `packages/rust/crates/xiuxian-wendao/tests/ingress_spider_unit/bridge.rs`

Responsibility split:

- `support.rs`: recording sink and partial-reindex hook fixtures;
- `uri.rs`: canonical URI and namespace extraction contracts;
- `bridge.rs`: deduplication, washed-content, malformed-payload, and graph persistence coverage.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test ingress_spider_unit --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test ingress_spider_unit --no-fail-fast`
  passed (`6 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/ingress_spider_unit.rs`
- `packages/rust/crates/xiuxian-wendao/tests/ingress_spider_unit/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/ingress_spider_unit/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/ingress_spider_unit/uri.rs`
- `packages/rust/crates/xiuxian-wendao/tests/ingress_spider_unit/bridge.rs`
