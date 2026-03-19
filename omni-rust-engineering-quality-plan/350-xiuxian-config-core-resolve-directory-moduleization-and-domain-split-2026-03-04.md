# 350. xiuxian-config-core resolve directory moduleization and domain split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-config-core`
- Target area:
  - removed: `src/resolve.rs`
  - added:
    - `src/resolve/mod.rs`
    - `src/resolve/discover.rs`
    - `src/resolve/io.rs`
    - `src/resolve/merge.rs`
    - `src/resolve/namespace.rs`
- Goal:
  - replace the single-file resolver implementation with domain-focused
    submodules while preserving public APIs:
    - `resolve_and_merge_toml`
    - `resolve_and_merge_toml_with_paths`
    - `resolve_and_load`
    - `resolve_and_load_with_paths`

## Implementation

1. Converted `resolve` into a directory module:
   - moved resolver entrypoints into `resolve/mod.rs`.
   - kept `mod.rs` focused on orchestration and public API.
2. Split resolver internals by concern:
   - `discover.rs`:
     - candidate-path discovery (`xiuxian.toml`, orphan paths),
       existing-file filtering, tracked-file aggregation.
   - `io.rs`:
     - TOML file read/parse boundary with typed `ConfigCoreError` mapping.
   - `merge.rs`:
     - recursive TOML merge semantics with `ArrayMergeStrategy` handling.
   - `namespace.rs`:
     - dotted-namespace extraction and root-level namespace projection.
3. Reduced duplication:
   - introduced shared `deserialize_merged` helper in `resolve/mod.rs` for
     `resolve_and_load*` API paths.
4. API compatibility:
   - no caller-visible signature changes.
   - no behavior changes intended; tests remain unchanged.
5. Lint hygiene:
   - removed an unused import (`ArrayMergeStrategy`) surfaced during clippy.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-config-core -- -W clippy::too_many_lines`
  - result: pass
- Targeted crate regression:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-config-core`
  - result: `12 passed`, `0 failed`

## Outcome

- Resolver code now follows repository modularization rules:
  - complex concern split by domain (discovery/io/merge/namespace),
  - orchestration retained in interface-level module entrypoint.
- Public API remains stable and regression tests remain green.
