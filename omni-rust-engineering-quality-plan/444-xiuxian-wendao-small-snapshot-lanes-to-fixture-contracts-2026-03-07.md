# 444. Xiuxian Wendao Small Snapshot Lanes to Fixture Contracts

Date: 2026-03-07

## Scope

This shard records the migration of three small Wendao snapshot lanes onto
fixture-backed `tests/fixtures/.../expected` contracts:

- Skill-VFS URI parsing
- skill-reference semantic classification
- embedded Wendao resource registry contracts

## Why This Change Was Needed

After the larger Skill-VFS and LinkGraph migrations, a few smaller snapshot-only
lanes still remained. They were structurally inconsistent with the newer
fixture-first standard because they:

- used `tests/snapshots/...` as a separate expected-state root,
- carried `*_snapshots.rs` names even though they were stable contract tests,
- preserved a parallel assertion style that made test architecture harder to
  reason about across the crate.

## What Changed

### 1) Renamed the lanes to `*_contracts.rs`

Replaced:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_uri_snapshots.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_reference_semantics_snapshots.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_resource_registry_snapshots.rs`

With:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_uri_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_reference_semantics_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_resource_registry_contracts.rs`

### 2) Moved expected outputs under fixture roots

New expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/uri_parser_contract/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_semantics/reference_classification/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_registry/embedded_resource_registry/expected/result.json`

The registry lane still uses the pre-existing embedded fixture inputs under
`tests/fixtures/embedded-registry/...`; only the expected contract location was
relocated.

### 3) Removed the obsolete snapshot files for these lanes

Deleted:

- `packages/rust/crates/xiuxian-wendao/tests/snapshots/skill_vfs/uri_parser_contract.json`
- `packages/rust/crates/xiuxian-wendao/tests/snapshots/skill_semantics/reference_classification.json`
- `packages/rust/crates/xiuxian-wendao/tests/snapshots/wendao_registry/embedded_resource_registry.json`

## Architectural Takeaways

- Small contract lanes should follow the same repository conventions as large
  ones; size is not a justification for keeping an obsolete assertion pattern.
- Naming matters: `*_contracts.rs` makes the test intent clearer than
  `*_snapshots.rs` once the expected state lives in fixtures.
- Fixture migration can be low-risk when the existing snapshot JSON is promoted
  directly into `expected/result.json`, preserving behavior while simplifying
  structure.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_uri_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_reference_semantics_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_resource_registry_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/uri_parser_contract/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_semantics/reference_classification/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_registry/embedded_resource_registry/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_skill_vfs_uri_contracts --test test_skill_reference_semantics_contracts --test test_wendao_resource_registry_contracts --message-format short
cargo nextest run -p xiuxian-wendao --test test_skill_vfs_uri_contracts --test test_skill_reference_semantics_contracts --test test_wendao_resource_registry_contracts
cargo clippy -p xiuxian-wendao --test test_skill_vfs_uri_contracts --test test_skill_reference_semantics_contracts --test test_wendao_resource_registry_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check ...` completed cleanly for the three target test binaries.
- `cargo nextest run ...` passed (`3 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly for the three target test binaries.
- Note: the clippy invocation still surfaced an existing `missing-docs` warning
  from `packages/rust/crates/xiuxian-skills/src/skills/internal_native/bindings.rs`,
  which is upstream to these test lanes and not introduced by this migration.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/443-xiuxian-wendao-internal-skill-authority-fixture-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/444-xiuxian-wendao-small-snapshot-lanes-to-fixture-contracts-2026-03-07.md`
