# 481. Xiuxian Wendao Internal Skill Manifest Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern
`test_internal_skill_manifest.rs` integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original test file bundled three distinct manifest behaviors into one
entrypoint:

- validated manifest loading and default hardening,
- validation failures for malformed manifests,
- manifest scan summary behavior.

Those concerns share a feature boundary, but they represent different contract
surfaces and should not live together in one top-level implementation file.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_manifest.rs`
so it now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_manifest/`
with focused modules:

- `mod.rs` for the module graph only,
- `support.rs` for fixture and JSON assertion re-exports,
- `load.rs` for validated manifest loading coverage,
- `validation.rs` for invalid manifest error behavior,
- `scan.rs` for scan summary and issue collection behavior.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_internal_skill_manifest --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test test_internal_skill_manifest --no-fail-fast`
  passed (`4 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Load-path success cases, validation failures, and scan summaries should live
  in separate test modules even when they exercise the same API surface.
- Test support should expose fixture and assertion helpers through a single
  local boundary rather than repeating crate-level imports in every module.
- Thin entrypoints make it easier to keep the Skill VFS and internal manifest
  test stack consistently organized.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_manifest.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_manifest/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_manifest/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_manifest/load.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_manifest/validation.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_manifest/scan.rs`
