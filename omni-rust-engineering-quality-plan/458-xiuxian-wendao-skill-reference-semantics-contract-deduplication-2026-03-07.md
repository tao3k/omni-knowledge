# 458. Xiuxian Wendao Skill Reference Semantics Contract Deduplication

Date: 2026-03-07

## Scope

This shard records the removal of a duplicate skill-reference semantics test
binary in favor of the existing fixture-backed contract surface.

## Why This Change Was Needed

The repository had two parallel test entries for the same behavior family:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_reference_semantics.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_reference_semantics_contracts.rs`

The non-contract file only asserted three cases that were already included in
`skill_reference_classification_matrix_contract`:

- persona hint classification,
- qianji-flow hint classification,
- attachment inference by extension.

Keeping both binaries added maintenance cost without adding new signal.

## What Changed

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_reference_semantics.rs`

The behavior family now lives only in:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_reference_semantics_contracts.rs`

No contract expansion was required because the existing fixture-backed matrix
already covered the removed assertions and more.

## Architectural Takeaways

- When a fixture-backed matrix already subsumes a smaller assertion file, the
  smaller file is duplication, not defense in depth.
- Contract matrices are especially effective for classification surfaces because
  they keep a whole semantic decision table in one place.
- Removing a duplicate test binary is the right move when the authoritative
  contract remains more expressive than the deleted entry.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_reference_semantics.rs` (removed)

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests --message-format short
cargo nextest run -p xiuxian-wendao --test test_skill_reference_semantics_contracts --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check ... --tests` completed cleanly.
- `test_skill_reference_semantics_contracts` passed (`1 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- Authoritative classification contract:
  - `packages/rust/crates/xiuxian-wendao/tests/test_skill_reference_semantics_contracts.rs`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/458-xiuxian-wendao-skill-reference-semantics-contract-deduplication-2026-03-07.md`
