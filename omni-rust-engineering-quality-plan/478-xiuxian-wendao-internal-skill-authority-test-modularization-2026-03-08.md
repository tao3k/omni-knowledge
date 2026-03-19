# 478. Xiuxian Wendao Internal Skill Authority Test Modularization

Date: 2026-03-08

## Scope

This shard records the modularization of the mixed-concern
`test_internal_skill_authority.rs` integration test in `xiuxian-wendao`.

## Why This Change Was Needed

The original test file mixed several authority-oriented contract surfaces in one
entrypoint:

- internal manifest authority audit,
- fast-path vs slow-path intent catalog equivalence,
- authorized manifest scan summaries,
- native alias preparation reports,
- empty-root behavior.

Those behaviors belong to the same domain, but they are distinct enough that
keeping them in a single top-level file made the suite harder to extend and
review.

## What Changed

### Thin Entrypoint

Updated `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_authority.rs`
so it now acts as a thin integration-test launcher.

### Directory Module Layout

Added `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_authority/`
with focused modules:

- `mod.rs` for the module graph only,
- `support.rs` for fixture and projection re-exports,
- `audit.rs` for manifest-authority audit behavior,
- `catalog.rs` for fast-path catalog parity coverage,
- `authorized_scan.rs` for authorized scan and native alias preparation.

### Shared Support Reuse

Kept the fixture projection logic in the crate-level `tests/support/`
namespace and consumed it through a local `support.rs` boundary instead of
repeating helper code in each test module.

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests
cargo nextest run -p xiuxian-wendao --test test_internal_skill_authority --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --tests` passed.
- `cargo nextest run -p xiuxian-wendao --test test_internal_skill_authority --no-fail-fast`
  passed (`6 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines` passed.

## Architectural Takeaways

- Audit-oriented integration suites still benefit from directory modules when
  they cover multiple contract surfaces.
- Fast-path/slow-path parity checks deserve their own module so equivalence
  logic does not disappear inside unrelated scan assertions.
- Shared support imports should stay behind one local support boundary even when
  the canonical helper implementations live in `tests/support/`.

## Artifacts and Notes

Changed paths:

- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_authority.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_authority/mod.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_authority/support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_authority/audit.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_authority/catalog.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_authority/authorized_scan.rs`
