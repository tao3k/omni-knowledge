# 515. Xiuxian Wendao Zhenfa Contract Surface Snapshot Migration

Date: 2026-03-08

## Scope

This shard records the Wave 15 migration of the remaining wrapper-based Zhenfa-facing Wendao contract surfaces into one external snapshot contract binary.

The new external binary is:

- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_contracts.rs`

The extracted shared support helper is:

- `packages/rust/crates/xiuxian-wendao/tests/support/zhenfa_contract_support.rs`

The retired wrappers are:

- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_router.rs`

The retired wrapper trees are:

- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_native_tools/`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_router/`

The committed fixture roots are:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/zhenfa/native_tools/`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/zhenfa/router/`

## What Changed

- Replaced two wrapper families with one standalone `test_zhenfa_contracts` binary.
- Normalized native-tool XML-Lite output into stable JSON hit projections.
- Normalized router markdown responses into stable hit summaries instead of pinning raw score strings.
- Normalized the native-tool cache-key contract by decoding the canonical JSON payload and replacing the temp-root path with `<root>`.
- Preserved request-context URI coverage and RPC invalid-params coverage as snapshot-backed contracts.

## Normalization Strategy

To avoid brittle snapshots while preserving user-visible contracts, the new support module snapshots stable projections instead of volatile runtime details:

- Cache keys snapshot the decoded canonical JSON payload, not the raw tempdir-bearing string.
- XML-Lite tool results snapshot hit `id`, `type`, and visible content, not floating scores.
- Router markdown responses snapshot header, total hits, and visible hit summaries, not score text.
- Native execution errors snapshot semantic kind, code, and message.

## Validation

Executed and passed with `zhenfa-router` enabled:

- `CARGO_TARGET_DIR=/tmp/xiuxian-zhenfa-contracts cargo check -p xiuxian-wendao --features zhenfa-router --test test_zhenfa_contracts --message-format short`
- `CARGO_TARGET_DIR=/tmp/xiuxian-zhenfa-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --features zhenfa-router --test test_zhenfa_contracts`
- `CARGO_TARGET_DIR=/tmp/xiuxian-zhenfa-contracts cargo clippy -p xiuxian-wendao --features zhenfa-router --test test_zhenfa_contracts -- -W clippy::too_many_lines`

Observed non-blocking warning noise remained limited to pre-existing runtime-config dead-code warnings in:

- `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/gateway.rs`
- `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/ui.rs`

## Remaining Wrapper Audit

A follow-up audit of the remaining wrapper+directory families found that most of them should stay modular because they are unit-oriented, stateful integration coverage, or performance suites.

The remaining snapshot-worthy families are:

- `packages/rust/crates/xiuxian-wendao/tests/test_enhancer.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_hmas.rs`

The families that should remain modular include dependency indexer units, entity/types/unified-symbol units, graph/cache/sync integration suites, link-graph agentic stateful coverage, and all benchmark suites.

## Result

The Zhenfa-facing Wendao contract surface is now fixture-backed and wrapper-free. The migration also narrows the remaining snapshot backlog to two clear future waves: `test_enhancer` and `test_hmas`.
