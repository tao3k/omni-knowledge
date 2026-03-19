# 477. Xiuxian Wendao Skill VFS Contract Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern
`test_skill_vfs_contracts.rs` integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original integration test file bundled several distinct contract surfaces
into a single top-level test binary:

- internal skill URI and path resolution,
- authorized internal scan contracts,
- manifest load and manifest error contracts,
- resolver runtime and shared-read behavior.

That structure made the test binary harder to navigate and violated the
repository rule that mixed responsibilities should be split by domain concern,
with `mod.rs` kept interface-only.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts.rs`
so it now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts/`
with focused modules:

- `mod.rs` for the module graph only,
- `support.rs` for shared fixture and assertion imports,
- `internal_resolution.rs` for URI/path resolution contracts,
- `authorized_scan.rs` for authorized manifest and native-alias scans,
- `manifests.rs` for manifest load, validation, and scan-issue contracts,
- `resolver_support.rs` for shared-read, embedded mount, and overlay behavior.

### Support Boundary

Kept the shared fixture/assertion helpers in the crate-level `tests/support/`
namespace and consumed them through the new local `support.rs` module. This
preserves a single support implementation without re-inlining helpers into the
entrypoint.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_skill_vfs_contracts --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test test_skill_vfs_contracts --no-fail-fast`
  passed (`7 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Integration test entrypoints should stay thin even when they still need to
  declare crate-level shared support modules.
- Split test modules by contract surface, not by arbitrary size thresholds.
- Keep `mod.rs` interface-only so new contract coverage can extend the module
  graph without reopening a mixed implementation file.
- Shared test fixtures belong in a dedicated support namespace; test modules
  should import them instead of duplicating setup code.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts/internal_resolution.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts/authorized_scan.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts/manifests.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts/resolver_support.rs`
