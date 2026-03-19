# 456. Xiuxian Wendao Skill VFS URI And Asset Contract Deduplication

Date: 2026-03-07

## Scope

This shard records the consolidation of two more Skill-VFS behavior families
onto their existing fixture-backed contract tests:

- URI parsing
- asset request APIs

## Why This Change Was Needed

The repository still kept parallel integration files that duplicated behavior
already covered by fixture-backed contract binaries:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_uri.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_asset_request_api.rs`

Those files repeated simple assertions for canonical URIs, traversal rejection,
callback reads, embedded reads, and shared-cache behavior that were already
captured more coherently in:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_uri_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_asset_request_api_contracts.rs`

## What Changed

### 1) Removed the duplicate URI parser integration file

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_uri.rs`

The URI parsing behavior now lives only in the fixture-backed
`skill_vfs_uri_contract`.

### 2) Removed the duplicate asset request integration file

Removed:

- `packages/rust/crates/xiuxian-wendao/tests/test_asset_request_api.rs`

Asset request behavior now lives only in the fixture-backed
`asset_request_api_contract`.

### 3) Preserved plain stripped-body coverage inside the contract surface

Updated:

- `packages/rust/crates/xiuxian-wendao/tests/test_asset_request_api_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/asset_request_api/expected/result.json`

Before removing `test_asset_request_api.rs`, the contract test was expanded to
record one more public behavior that had only been covered by the duplicate
file:

- plain `read_stripped_body()` preview via `teacher_plain_stripped_preview`

This ensured the contract binary retained both shared and non-shared read-path
coverage.

## Architectural Takeaways

- Contract tests are the right home for stable public API behavior such as URI
  normalization, traversal rejection, and asset-read semantics.
- Removing duplicated assertion files is safe when the contract test preserves
  both success and failure semantics in explicit fixture output.
- When deleting a duplicate test, first audit for any unique behavior it still
  covers. If needed, migrate that behavior into the contract before deletion.
- High-quality Rust testing prefers one authoritative test surface over several
  partially overlapping ones.

## Files Changed

- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_uri_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_asset_request_api_contracts.rs`
- `packages/rust/crates/xiuxian-wendao/tests/fixtures/skill_vfs/asset_request_api/expected/result.json`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_uri.rs` (removed)
- `packages/rust/crates/xiuxian-wendao/tests/test_asset_request_api.rs` (removed)

## Validation Evidence

Executed and passed:

```bash
cargo check -p xiuxian-wendao --tests --message-format short
cargo nextest run -p xiuxian-wendao --test test_skill_vfs_uri_contracts --test test_asset_request_api_contracts --no-fail-fast
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
```

Observed outcomes:

- `cargo check ... --tests` completed cleanly.
- The targeted `cargo nextest run ...` passed (`2 passed, 0 skipped`).
- `cargo clippy ...` completed cleanly.

## Artifacts and Notes

- Related prior shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/455-xiuxian-wendao-skill-vfs-resolver-contract-deduplication-2026-03-07.md`
- New knowledge shard:
  - `assets/knowledge/omni-rust-engineering-quality-plan/456-xiuxian-wendao-skill-vfs-uri-and-asset-contract-deduplication-2026-03-07.md`
