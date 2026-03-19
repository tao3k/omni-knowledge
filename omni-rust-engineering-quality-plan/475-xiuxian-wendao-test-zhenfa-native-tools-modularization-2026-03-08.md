# 475. Xiuxian Wendao `test_zhenfa_native_tools` Modularization

Date: 2026-03-08

## Scope

This shard records the structural decomposition of
`packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools.rs`.

## Why This Change Was Needed

The integration test entrypoint mixed four distinct concerns under one file:

- shared notebook fixture and injected-index context setup,
- native dispatch behavior,
- semantic hit-type classification behavior,
- cache-key and request-context utility coverage.

Because the file is feature-gated with `zhenfa-router`, keeping those concerns
co-located made it harder to see which behavior surface failed under feature
validation.

## What Changed

Reduced the root integration-test entrypoint to a thin launcher:

- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools.rs`

Created a dedicated directory module:

- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools/dispatch.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools/classification.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools/cache_key.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools/request_context.rs`

Responsibility split:

- `support.rs`: notebook fixture and injected-index context builders;
- `dispatch.rs`: native dispatch success and extension reuse/failure behavior;
- `classification.rs`: journal/path/tag/frontmatter type inference behavior;
- `cache_key.rs`: cache-key stability contract;
- `request_context.rs`: `WendaoContextExt` asset-request contract.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests --features zhenfa-router
cargo nextest run -p xiuxian-wendao --features zhenfa-router --test test_zhenfa_native_tools --no-tests pass --no-fail-fast
cargo clippy -p xiuxian-wendao --features zhenfa-router -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests --features zhenfa-router` passed.
- `cargo nextest run -p xiuxian-wendao --features zhenfa-router --test test_zhenfa_native_tools --no-tests pass --no-fail-fast`
  passed (`8 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao --features zhenfa-router -- -W clippy::too_many_lines` passed.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools/dispatch.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools/classification.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools/cache_key.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools/request_context.rs`
