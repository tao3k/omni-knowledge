# 512. Xiuxian Wendao Internal Skill Contract Snapshot Migration

Date: 2026-03-08

## Scope

This shard records the Wave 12 migration of the remaining wrapper-based internal-skill contract families in `xiuxian-wendao` into one explicit external snapshot contract binary.

The retired wrappers were:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_authority.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_manifest.rs`

The retired wrapper trees were:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts/`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_authority/`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_manifest/`

They have now been replaced by:

- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_surface_contracts.rs`

## Why This Change Was Needed

These three wrapper families all exercised the same internal-skill surface and already shared the committed `skill_vfs` fixture tree. Keeping them behind separate wrapper binaries and nested module trees preserved unnecessary indirection after the Wendao CLI migration was finished.

This wave removed that indirection without changing the fixture source of truth.

## What Changed

### 1) Added one external internal-skill contract binary

Added:

- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_surface_contracts.rs`

The new binary covers:

- internal skill URI resolution,
- authorized internal manifest scanning,
- authorized internal native alias scanning,
- manifest loading and hardened defaults,
- manifest error-chain reporting,
- manifest scan issue reporting,
- resolver support behavior across semantic, embedded, and internal mounts,
- internal authority audit/reporting,
- internal intent catalog fast-path equivalence,
- manifest validation failures.

### 2) Reused the existing shared support surface directly

Reused directly:

- `packages/rust/crates/xiuxian-wendao/tests/support/fixture_json_assertions.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/fixture_read.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/internal_skill_authority_fixture_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/skill_vfs_contract_support.rs`
- `packages/rust/crates/xiuxian-wendao/tests/support/skill_vfs_fixture_tree.rs`

No new fixture trees were needed because the existing `tests/fixtures/skill_vfs/...` structure already covered all scenarios.

### 3) Deleted the superseded wrappers immediately

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_authority.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_manifest.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_contracts/`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_authority/`
- `packages/rust/crates/xiuxian-wendao/tests/test_internal_skill_manifest/`

This keeps the repository aligned with the user's request: once a family is migrated, the superseded tests are deleted immediately.

## Validation Evidence

Executed and passed:

```bash
CARGO_TARGET_DIR=/tmp/xiuxian-internal-skill-contracts cargo check -p xiuxian-wendao --test test_internal_skill_surface_contracts --message-format short
CARGO_TARGET_DIR=/tmp/xiuxian-internal-skill-contracts NEXTEST_HIDE_PROGRESS_BAR=1 cargo nextest run -p xiuxian-wendao --test test_internal_skill_surface_contracts
CARGO_TARGET_DIR=/tmp/xiuxian-internal-skill-contracts cargo clippy -p xiuxian-wendao --test test_internal_skill_surface_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check` completed for `test_internal_skill_surface_contracts`.
- `cargo nextest run` passed for `test_internal_skill_surface_contracts` (`17 passed, 0 skipped`).
- `cargo clippy` completed for the new binary.
- Remaining warnings observed during validation came from pre-existing library code in `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/gateway.rs` and `packages/rust/crates/xiuxian-wendao/src/link_graph/runtime_config/resolve/ui.rs`.
