# 443. Xiuxian Wendao Internal-Skill Authority Fixture Contracts

Date: 2026-03-07

## Scope

This shard records the migration of `test_internal_skill_authority.rs` from
inline temporary-tree construction to fixture-backed `tests/fixtures/skill_vfs`
contracts.

## Why This Change Was Needed

The authority lane was still hand-building internal skill trees with repeated
`TempDir`, `create_dir_all`, and `write(...)` calls even after the surrounding
Skill-VFS contract surface had already moved to fixture-first scenarios.

That left the same domain split across two testing styles:

- resolver and manifest contracts were fixture-backed,
- authority and authorization checks were still authored as inline filesystem
  programs.

## What Changed

### 1) Replaced inline authority corpus setup with fixture scenarios

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_authority.rs`

The suite now materializes fixture trees instead of constructing them inline.
Primary scenarios:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_invalid/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/no_internal_roots/...`

### 2) Added focused authority projection support

New file:

- `packages/rust/crates/xiuxian-wendao/tests/support/internal_skill_authority_fixture_support.rs`

This support module owns:

- authority-report projection,
- intent-catalog projection,
- authorized-scan summary projection,
- issue normalization for temporary absolute paths,
- fixture assertion routing for this lane.

That keeps the test file focused on behavior rather than serialization details.

### 3) Converted the lane to fixture-backed expected JSON contracts

New expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/expected/authority_report.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/expected/catalog_fast_path.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/expected/manifest_scan_summary.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/expected/native_alias_summary.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_invalid/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/no_internal_roots/expected/result.json`

The suite now stores its expected authority surface next to the corresponding
input trees, instead of embedding those expectations in code.

## Architectural Takeaways

- Once a domain has a fixture tree, adjacent tests in the same domain should
  converge on it instead of inventing parallel temporary setup logic.
- Authority diagnostics must normalize temporary absolute paths before they are
  asserted; otherwise fixture contracts become host-specific and unstable.
- A minimal domain support module is preferable to importing a larger support
  surface just for one helper. During this migration, reusing a broad Skill-VFS
  contract helper created dead-code warnings, so the authority lane now carries
  only the projection logic it actually needs.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_authority.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/internal_skill_authority_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/expected/authority_report.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/expected/catalog_fast_path.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/expected/manifest_scan_summary.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_scans/expected/native_alias_summary.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_invalid/input/internal_skills/agenda/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_invalid/input/internal_skills/agenda/references/add/qianji.toml`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/authorized_internal_invalid/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/no_internal_roots/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_internal_skill_authority --message-format short
cargo nextest run -p xiuxian-wendao --test test_internal_skill_authority
cargo clippy -p xiuxian-wendao --test test_internal_skill_authority -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --test test_internal_skill_authority --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_internal_skill_authority` passed (`6 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_internal_skill_authority -- -W clippy::too_many_lines` completed cleanly.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/442-xiuxian-wendao-skill-vfs-snapshot-to-fixture-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/443-xiuxian-wendao-internal-skill-authority-fixture-contracts-2026-03-07.md`
