# 352. xiuxian-wendao skill-vfs index directory moduleization and build/preload/semantic split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-wendao`
- Target area:
  - removed: `src/skill_vfs/index.rs`
  - added:
    - `src/skill_vfs/index/mod.rs`
    - `src/skill_vfs/index/build.rs`
    - `src/skill_vfs/index/preload.rs`
    - `src/skill_vfs/index/semantic.rs`
- Goal:
  - split monolithic namespace index implementation into focused modules while
    preserving `SkillNamespaceIndex`/`SkillNamespaceMount` public behavior.

## Implementation

1. Converted `skill_vfs::index` to directory module:
   - `index/mod.rs` now owns type definitions and public query surface:
     - `mounts_for`
     - `namespace_count`
     - `path_for_uri`
2. Split internal concerns:
   - `build.rs`:
     - root traversal and descriptor scan orchestration
     - `build_from_roots` implementation.
   - `preload.rs`:
     - reference directory preload and URI-key projection
     - relative-entity normalization.
   - `semantic.rs`:
     - skill descriptor detection
     - semantic-name extraction from scanner/frontmatter with typed error
       mapping.
3. Kept resolver/index contracts stable:
   - no method signature changes on `SkillNamespaceIndex`.
   - URI key semantics and mount ordering behavior preserved.
4. No lint suppression introduced.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
  - result: pass
- Targeted skill-vfs regression:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-wendao --test test_skill_vfs_resolver --test test_skill_vfs_uri`
  - result: `13 passed`, `0 failed`

## Outcome

- `skill_vfs::index` now follows modularization standards with clear ownership
  boundaries (build/preload/semantic/query).
- Existing VFS resolver and URI contracts remain stable and validated.
