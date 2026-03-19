# 495. Xiuxian Wendao Zhenfa Router Integration Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern `test_zhenfa_router.rs`
integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original feature-gated router test file mixed multiple transport and setup
concerns in one top-level implementation file:

- notebook fixture construction,
- gateway spawn/setup,
- RPC success and invalid-params behavior,
- HTTP search behavior.

These contracts belong to the same router surface, but the transport-specific
assertions and reusable setup logic should not remain mixed together.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_router.rs` so it
now acts as a thin feature-gated integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_router/` with
focused modules:

- `mod.rs` for the module graph only,
- `support.rs` for gateway construction, notebook fixture setup, and app spawn,
- `rpc.rs` for RPC success and invalid-params behavior,
- `http.rs` for HTTP search behavior.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests --features zhenfa-router
cargo nextest run -p xiuxian-wendao --features zhenfa-router --test test_zhenfa_router --no-fail-fast
cargo clippy -p xiuxian-wendao --features zhenfa-router -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests --features zhenfa-router` passed.
- `cargo nextest run -p xiuxian-wendao --features zhenfa-router --test test_zhenfa_router --no-fail-fast`
  passed (`3 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao --features zhenfa-router -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- RPC and HTTP transport checks should be split into separate modules even for
  small gateway integration suites.
- Feature-gated gateway tests still benefit from dedicated setup helpers for app
  spawn and notebook fixture creation.
- Thin entrypoints keep the router integration suite consistent with the rest of
  the crate's package-level test structure.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_router.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_router/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_router/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_router/rpc.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_zhenfa_router/http.rs`
