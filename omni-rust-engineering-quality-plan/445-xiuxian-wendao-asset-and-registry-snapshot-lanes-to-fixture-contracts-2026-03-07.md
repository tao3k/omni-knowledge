# 445. Xiuxian Wendao Asset and Registry Snapshot Lanes to Fixture Contracts

Date: 2026-03-07

## Scope

This shard records the migration of three additional snapshot-backed Wendao test
lanes onto fixture-backed contracts:

- Wendao asset request API
- embedded skill resource API
- embedded dynamic discovery queries

## Why This Change Was Needed

These lanes were still using `tests/snapshots/...` even after the surrounding
Skill-VFS and registry surfaces had already started converging on
`tests/fixtures/.../expected`.

That left the crate with mixed naming and assertion patterns for conceptually
similar contract tests.

## What Changed

### 1) Renamed the lanes to `*_contracts.rs`

Replaced:

- `packages/rust/crates/xiuxian-wendao/tests/test_asset_request_api_snapshots.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_embedded_skill_api_snapshots.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_dynamic_discovery_snapshots.rs`

With:

- `packages/rust/crates/xiuxian-wendao/tests/test_asset_request_api_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_embedded_skill_api_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_dynamic_discovery_contracts.rs`

### 2) Relocated expected outputs into fixture trees

New expected contracts:

- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/asset_request_api/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_registry/embedded_skill_api/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_registry/dynamic_discovery/expected/result.json`

### 3) Removed the corresponding snapshot files

Deleted:

- `packages/rust/crates/xiuxian-wendao/tests/snapshots/skill_vfs/asset_request_api.json`
- `packages/rust/crates/xiuxian-wendao/tests/snapshots/wendao_registry/embedded_skill_api.json`
- `packages/rust/crates/xiuxian-wendao/tests/snapshots/wendao_registry/dynamic_discovery.json`

## Architectural Takeaways

- Asset, registry, and discovery contracts should share the same fixture-backed
  assertion model as other Wendao contract lanes.
- Small API tests still deserve consistent naming; `*_contracts.rs` is clearer
  than `*_snapshots.rs` once the expected state lives in fixtures.
- When migrating a lane that already has a stable JSON contract, promoting the
  snapshot payload directly into `expected/result.json` minimizes risk and keeps
  the change focused on structure.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_asset_request_api_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_embedded_skill_api_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_wendao_dynamic_discovery_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/asset_request_api/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_registry/embedded_skill_api/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/wendao_registry/dynamic_discovery/expected/result.json`

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --test test_asset_request_api_contracts --test test_embedded_skill_api_contracts --test test_wendao_dynamic_discovery_contracts --message-format short
cargo nextest run -p xiuxian-wendao --test test_asset_request_api_contracts --test test_embedded_skill_api_contracts --test test_wendao_dynamic_discovery_contracts
cargo clippy -p xiuxian-wendao --test test_asset_request_api_contracts --test test_embedded_skill_api_contracts --test test_wendao_dynamic_discovery_contracts -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check ...` completed cleanly for the three target test binaries.
- `cargo nextest run ...` passed (`3 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly for the three target test binaries.

## Artifacts and Notes

- Prior prerequisite shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/444-xiuxian-wendao-small-snapshot-lanes-to-fixture-contracts-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/445-xiuxian-wendao-asset-and-registry-snapshot-lanes-to-fixture-contracts-2026-03-07.md`
