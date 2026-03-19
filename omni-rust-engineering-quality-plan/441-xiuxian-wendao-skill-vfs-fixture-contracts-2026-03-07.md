# 441. Xiuxian Wendao Skill-VFS Fixture Contracts

Date: 2026-03-07

## Scope

This shard records the Wendao Skill-VFS test-architecture slice that migrates
`test_skill_vfs_resolver.rs` and `test_internal_skill_manifest.rs` away from
inline tree seeding and onto scenario-based `tests/fixtures/skill_vfs/...`
contracts.

## Why This Change Was Needed

The resolver and manifest suites were still constructing temporary skill trees
inline with repeated `TempDir`, `create_dir_all`, and `write(...)` calls.

That shape had three costs:

- it duplicated the same `SKILL.md` and manifest payloads across tests,
- it made the real contract harder to see because directory setup dominated the
  test bodies,
- it kept fixture semantics inconsistent with the LinkGraph test architecture,
  which had already moved onto `input/expected` scenarios.

## What Changed

### 1) Split fixture responsibilities instead of adding lint suppressions

New file:

- `packages/rust/crates/xiuxian-wendao/tests/support/skill_vfs_fixture_tree.rs`

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/support/skill_vfs_fixture.rs`

The split is intentional:

- `skill_vfs_fixture_tree.rs` now owns only scenario materialization from
  `tests/fixtures/skill_vfs/<scenario>/input/...`.
- `skill_vfs_fixture.rs` now owns only the write-based seed helpers still used
  by the snapshot-oriented Skill-VFS suite.

This removed dead-code warnings structurally instead of hiding them with
`#[allow(...)]`.

### 2) Migrated resolver tests to per-scenario fixtures

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_resolver.rs`

New resolver scenarios:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_semantic_uri/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_documents_and_manifests/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_overlay_prefers_first_root/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/overlay_precedence_by_root_order/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/shared_arc_internal/...`

The resolver contracts now serialize stable outputs such as resolved content,
manifest URIs, and relative path selection instead of hiding those expectations
behind ad hoc string checks.

### 3) Migrated internal-manifest tests to per-scenario fixtures

Updated file:

- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_manifest.rs`

New manifest scenarios:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_loaded/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_invalid_description/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_missing_background_binding/...`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_scan_issues/...`

The loaded-manifest and scan-manifest lanes now compare structured JSON
contracts under `expected/result.json`, while the rejection lanes keep explicit
error-shape assertions.

## Architectural Takeaways

- Skill-VFS tests benefit from the same `input/expected` discipline as
  LinkGraph: the file tree is fixture data, not test logic.
- Support modules should be split by responsibility when different integration
  binaries need different helper surfaces; that is a cleaner fix than tolerating
  dead-code warnings.
- Relative-path serialization keeps fixture expectations portable and avoids
  leaking absolute `TempDir` locations into contracts.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/support/skill_vfs_fixture.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/skill_vfs_fixture_tree.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_resolver.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_manifest.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_semantic_uri/input/internal/agenda_skill/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_semantic_uri/input/internal/agenda_skill/references/steward.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/resolver_semantic_uri/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_documents_and_manifests/input/internal_skills/agenda/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_documents_and_manifests/input/internal_skills/agenda/references/add/qianji.toml`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_documents_and_manifests/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_overlay_prefers_first_root/input/first/agenda/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_overlay_prefers_first_root/input/second/agenda/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/internal_overlay_prefers_first_root/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/overlay_precedence_by_root_order/input/internal/agenda_skill/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/overlay_precedence_by_root_order/input/internal/agenda_skill/references/steward.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/overlay_precedence_by_root_order/input/user/agenda_skill/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/overlay_precedence_by_root_order/input/user/agenda_skill/references/steward.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/overlay_precedence_by_root_order/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/shared_arc_internal/input/internal_skills/agenda/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_loaded/input/internal_skills/agenda/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_loaded/input/internal_skills/agenda/references/add/qianji.toml`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_loaded/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_invalid_description/input/internal_skills/agenda/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_invalid_description/input/internal_skills/agenda/references/view/qianji.toml`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_missing_background_binding/input/internal_skills/agenda/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_missing_background_binding/input/internal_skills/agenda/references/add/qianji.toml`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_scan_issues/input/internal_skills/agenda/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_scan_issues/input/internal_skills/agenda/references/add/qianji.toml`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_scan_issues/input/internal_skills/broken/SKILL.md`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_scan_issues/input/internal_skills/broken/references/view/qianji.toml`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/manifest_scan_issues/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_skill_vfs_resolver --test test_internal_skill_manifest --message-format short
cargo nextest run -p xiuxian-wendao --test test_skill_vfs_resolver --test test_internal_skill_manifest
cargo clippy -p xiuxian-wendao --test test_skill_vfs_resolver --test test_internal_skill_manifest -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check -p xiuxian-wendao --test test_skill_vfs_resolver --test test_internal_skill_manifest --message-format short` completed cleanly.
- `cargo nextest run -p xiuxian-wendao --test test_skill_vfs_resolver --test test_internal_skill_manifest` passed (`17 passed, 0 skipped`).
- `cargo clippy -p xiuxian-wendao --test test_skill_vfs_resolver --test test_internal_skill_manifest -- -W clippy::too_many_lines` completed cleanly.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/440-xiuxian-wendao-cache-build-fixture-expected-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/441-xiuxian-wendao-skill-vfs-fixture-contracts-2026-03-07.md`
