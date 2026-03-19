# 442. Xiuxian Wendao Skill-VFS Snapshot-to-Fixture Contracts

Date: 2026-03-07

## Scope

This shard records the Skill-VFS contract migration that replaces
`tests/snapshots/skill_vfs/...` assertions in the former
`test_skill_vfs_snapshots.rs` lane with fixture-backed `tests/fixtures/skill_vfs/.../expected`
contracts.

## Why This Change Was Needed

The repository had already converged on fixture-first test architecture for the
modernized Wendao suites, but the Skill-VFS contract lane still depended on a
separate `tests/snapshots/skill_vfs` tree.

That was no longer aligned with the desired repository standard:

- the input trees still lived as inline `TempDir + fs::write(...)` setup,
- the expected outputs lived outside the fixture tree in a dedicated snapshot
  root,
- the test file had grown into a mixed concern surface with both contract cases
  and projection helpers in one place.

## What Changed

### 1) Renamed the test lane to reflect its real role

Replaced:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_snapshots.rs`

With:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts.rs`

The lane is now explicitly about contract fixtures, not a separate snapshot
mechanism.

### 2) Extracted domain-specific support instead of keeping a monolithic test file

New file:

- `packages/rust/crates/xiuxian-wendao/tests/support/skill_vfs_contract_support.rs`

This support module owns:

- Skill-VFS JSON fixture assertion routing,
- URI projection,
- internal manifest projection,
- authority scan projection,
- native alias scan projection,
- issue normalization and relative-path normalization.

This keeps the test file focused on case setup and behavior only.

### 3) Moved contract inputs and expected outputs under `tests/fixtures/skill_vfs`

New fixture scenarios:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_skill_resolution/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_errors/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_support/...`

Extended existing fixture scenarios:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_loaded/expected/contract.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_scan_issues/expected/contract.json`

Notable design point:

- the authorized-manifest and native-alias contract tests now share one input
  scenario (`authorized_internal_scans`) with two expected JSON contracts. This
  reduces duplicated corpus trees while keeping per-surface assertions explicit.

### 4) Removed the obsolete Skill-VFS snapshot files

Deleted obsolete snapshot contracts:

- `packages/rust/crates/xiuxian-wendao/tests/snapshots/skill_vfs/internal_skill_resolution.json`
- `packages/rust/crates/xiuxian-wendao/tests/snapshots/skill_vfs/internal_skill_authority_scan.json`
- `packages/rust/crates/xiuxian-wendao/tests/snapshots/skill_vfs/internal_skill_native_alias_scan.json`
- `packages/rust/crates/xiuxian-wendao/tests/snapshots/skill_vfs/internal_skill_manifest_loaded.json`
- `packages/rust/crates/xiuxian-wendao/tests/snapshots/skill_vfs/internal_skill_manifest_errors.json`
- `packages/rust/crates/xiuxian-wendao/tests/snapshots/skill_vfs/internal_skill_manifest_scan_issues.json`
- `packages/rust/crates/xiuxian-wendao/tests/snapshots/skill_vfs/resolver_support.json`

## Architectural Takeaways

- Fixture trees should contain both the input corpus and the expected contract
  surface for the same scenario; splitting expected state into a separate
  snapshot root creates unnecessary architectural drift.
- When a contract lane grows, extract domain-specific projection helpers instead
  of leaving serialization, normalization, and assertions inside the test file.
- Shared input scenarios with multiple expected files are a better fit than
  duplicated corpus trees when multiple contracts exercise the same physical
  setup.
- When fixture failures reveal a mismatch, check the input corpus before blaming
  the production code. During this migration, the only failing contract came
  from stale fixture input that had dropped `mcp_contract.category`, not from a
  resolver regression.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/skill_vfs_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_skill_resolution/input/internal_skills/agenda/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_skill_resolution/input/internal_skills/agenda/scripts/qianji.toml`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_skill_resolution/input/internal_skills/agenda/references/add/qianji.toml`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_skill_resolution/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/input/internal_skills/agenda/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/input/internal_skills/agenda/references/guide.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/input/internal_skills/agenda/references/add/qianji.toml`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/input/internal_skills/agenda/references/view/qianji.toml`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/expected/manifest_scan.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/expected/native_alias_scan.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_errors/input/internal_skills/agenda/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_errors/input/internal_skills/agenda/references/add/qianji.toml`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_errors/input/internal_skills/agenda/references/view/qianji.toml`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_errors/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_loaded/input/internal_skills/agenda/references/add/qianji.toml`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_loaded/expected/contract.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_scan_issues/input/internal_skills/agenda/references/add/qianji.toml`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_scan_issues/expected/contract.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_support/input/internal/agenda_skill/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_support/input/internal/agenda_skill/references/steward.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_support/input/user/agenda_skill/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_support/input/user/agenda_skill/references/steward.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_support/input/internal-first/agenda/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_support/input/internal-second/agenda/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_support/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_skill_vfs_contracts --message-format short
cargo nextest run -p xiuxian-wendao --test test_skill_vfs_contracts
cargo clippy -p xiuxian-wendao --test test_skill_vfs_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --test test_skill_vfs_contracts --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_skill_vfs_contracts` passed (`7 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_skill_vfs_contracts -- -W clippy::too_many_lines` completed cleanly.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/441-xiuxian-wendao-skill-vfs-fixture-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/442-xiuxian-wendao-skill-vfs-snapshot-to-fixture-contracts-2026-03-07.md`
