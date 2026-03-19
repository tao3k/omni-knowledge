# 494. Xiuxian Wendao Zhenfa Router XML Lite Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern
`zhenfa_router_xml_lite_unit.rs` integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original feature-gated file mixed separate hit-type inference contracts in
one top-level implementation file:

- doc-type driven type inference,
- tag-driven inference,
- path fallback inference,
- attachment fallback inference.

These behaviors belong to the same renderer surface, but they should not remain
bundled in one test module.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/zhenfa_router_xml_lite_unit.rs`
so it now acts as a thin feature-gated integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/zhenfa_router_xml_lite_unit/`
with focused modules:

- `mod.rs` for the module graph only,
- `support.rs` for XML-lite hit and payload rendering helpers,
- `doc_type.rs` for doc-type and tag-priority inference contracts,
- `fallback.rs` for path and attachment fallback behavior.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests --features zhenfa-router
cargo nextest run -p xiuxian-wendao --features zhenfa-router --test zhenfa_router_xml_lite_unit --no-fail-fast
cargo clippy -p xiuxian-wendao --features zhenfa-router -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests --features zhenfa-router` passed.
- `cargo nextest run -p xiuxian-wendao --features zhenfa-router --test zhenfa_router_xml_lite_unit --no-fail-fast`
  passed (`5 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao --features zhenfa-router -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Feature-gated renderer tests should still follow the same thin-entrypoint and
  focused-module structure as default-feature suites.
- Doc-type priority rules and fallback heuristics belong in separate modules so
  the inference contract remains easy to extend.
- Reusable payload rendering should stay in local support helpers instead of
  being rebuilt inside every test.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/zhenfa_router_xml_lite_unit.rs`
- `packages/rust/crates/xiuxian-wendao/tests/zhenfa_router_xml_lite_unit/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/zhenfa_router_xml_lite_unit/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/zhenfa_router_xml_lite_unit/doc_type.rs`
- `packages/rust/crates/xiuxian-wendao/tests/zhenfa_router_xml_lite_unit/fallback.rs`
