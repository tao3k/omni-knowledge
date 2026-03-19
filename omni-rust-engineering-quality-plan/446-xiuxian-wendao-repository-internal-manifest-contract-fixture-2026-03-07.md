# 446. Xiuxian Wendao Repository Internal-Manifest Contract Fixture

Date: 2026-03-07

## Scope

This shard records the migration of the repository-backed internal-manifest
regression lane from snapshot assertions to fixture-backed expected contracts.

## Why This Change Was Needed

`test_internal_skill_repository.rs` still used `tests/snapshots/skill_vfs` even
though the rest of the Skill-VFS domain had already moved toward
`tests/fixtures/.../expected`.

Although this lane is repository-backed rather than synthetic-fixture-backed, it
still benefits from the same structural convention for expected outputs.

## What Changed

### 1) Renamed the lane to `*_contracts.rs`

Replaced:

- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_repository.rs`

With:

- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_repository_contracts.rs`

### 2) Relocated the expected repository contract into fixtures

New expected contract:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/repository_internal_manifests/expected/result.json`

Deleted obsolete snapshot:

- `packages/rust/crates/xiuxian-wendao/tests/snapshots/skill_vfs/repository_internal_manifests.json`

### 3) Preserved repository-backed semantics while standardizing layout

The lane still validates the live `internal_skills` tree in the workspace. This
migration changes only the expected-output location and test naming, not the
behavioral scope.

## Architectural Takeaways

- Repository-backed regression tests can still use fixture-backed expected
  contracts even when their inputs are the live workspace.
- Standardizing expected-output layout across synthetic and repository-backed
  lanes reduces cognitive overhead when auditing the crate.
- Once a domain adopts fixture-first expectations, leaving one last snapshot
  file behind adds inconsistency without adding value.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_repository_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/repository_internal_manifests/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_internal_skill_repository_contracts --message-format short
cargo nextest run -p xiuxian-wendao --test test_internal_skill_repository_contracts
cargo clippy -p xiuxian-wendao --test test_internal_skill_repository_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check ...` completed cleanly.
- `cargo nextest run ...` passed (`1 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/445-xiuxian-wendao-asset-and-registry-snapshot-lanes-to-fixture-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/446-xiuxian-wendao-repository-internal-manifest-contract-fixture-2026-03-07.md`
