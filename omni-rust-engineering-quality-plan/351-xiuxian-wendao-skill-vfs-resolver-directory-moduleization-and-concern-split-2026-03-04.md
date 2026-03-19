# 351. xiuxian-wendao skill-vfs resolver directory moduleization and concern split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-wendao`
- Target area:
  - removed: `src/skill_vfs/resolver.rs`
  - added:
    - `src/skill_vfs/resolver/mod.rs`
    - `src/skill_vfs/resolver/core.rs`
    - `src/skill_vfs/resolver/mount.rs`
    - `src/skill_vfs/resolver/resolve_uri.rs`
    - `src/skill_vfs/resolver/read.rs`
- Goal:
  - replace monolithic resolver implementation with domain-focused modules
    while preserving the public API surface of `SkillVfsResolver`.

## Implementation

1. Converted `skill_vfs::resolver` into a directory module:
   - `resolver/mod.rs` now serves as module entrypoint and re-export surface.
2. Split resolver concerns by domain:
   - `core.rs`:
     - resolver struct/state ownership (`index`, mounts, semantic map, cache)
     - constructor and basic access (`from_roots`, `from_roots_with_embedded`,
       `index`).
   - `mount.rs`:
     - embedded mount registration and semantic mount normalization
     - `mount_embedded_dir` assembly.
   - `resolve_uri.rs`:
     - URI/path resolution and typed missing/unknown error mapping.
   - `read.rs`:
     - read orchestration (`read_utf8`, `read_semantic`, `read_utf8_shared`)
     - local+embedded read paths and cache insertion logic
     - embedded path normalization helper.
3. Public behavior preserved:
   - no external method signature changed.
   - cache lookup order and error semantics remain unchanged.
4. No lint suppression introduced.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
  - result: pass
- Targeted resolver regression:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-wendao --test test_skill_vfs_resolver --test test_skill_vfs_uri`
  - result: `13 passed`, `0 failed`

## Outcome

- `SkillVfsResolver` now follows modularization standards:
  - concern split by construction/mounting/resolve/read paths,
  - no single large resolver file.
- Existing resolver contracts remain stable and test-validated.
