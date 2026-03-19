# 353. xiuxian-wendao skill-vfs asset-request directory moduleization and API stability (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-wendao`
- Target area:
  - removed: `src/skill_vfs/asset_request.rs`
  - added:
    - `src/skill_vfs/asset_request/mod.rs`
    - `src/skill_vfs/asset_request/types.rs`
    - `src/skill_vfs/asset_request/build.rs`
    - `src/skill_vfs/asset_request/normalize.rs`
    - `src/skill_vfs/asset_request/read.rs`
- Goal:
  - split `asset_request` into focused modules while preserving public API and
    behavior of `WendaoAssetHandle` and `AssetRequest`.

## Implementation

1. Converted `skill_vfs::asset_request` into directory module:
   - `mod.rs` now defines interface surface and re-exports public request types.
2. Split concerns by domain:
   - `types.rs`:
     - type definitions and stable basic API (`AssetRequest::new`, `AssetRequest::uri`).
   - `build.rs`:
     - semantic URI request construction via
       `WendaoAssetHandle::skill_reference_asset`.
   - `normalize.rs`:
     - package-id and relative-path normalization with typed error mapping.
   - `read.rs`:
     - all read/strip APIs (`read_utf8*`, `read_stripped_body*`)
     - process-level stripped-body `Arc<str>` cache.
3. API compatibility:
   - no public method signatures changed.
   - URI canonicalization, callback behavior, embedded resolver behavior, and
     stripped-body cache semantics preserved.
4. No broad lint suppression introduced.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
  - result: pass
- Targeted asset/VFS regression:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-wendao --test test_asset_request_api --test test_skill_vfs_resolver --test test_skill_vfs_uri`
  - result: `19 passed`, `0 failed`

## Outcome

- `asset_request` now follows repository modularization standards with clearer
  ownership boundaries (types/build/normalize/read).
- Existing public APIs and tests remain stable, with no behavior regression.
