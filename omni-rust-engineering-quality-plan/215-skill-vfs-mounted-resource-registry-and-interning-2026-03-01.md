# 215. Skill VFS Mounted Resource Registry and Interning (2026-03-01)

## Scope

- Align resolver internals with ADR-007 performance baseline:
  - mounted in-memory resource images,
  - shared `Arc<str>` payload path,
  - semantic read path with zero runtime filesystem I/O.
- Keep existing URI and downstream behavior compatible.

## Implementation

1. Resolver now owns explicit mount registry and interning cache
- `packages/rust/crates/xiuxian-wendao/src/skill_vfs/resolver.rs`
  - Replaced boolean embedded switch with explicit structures:
    - `mounts: HashMap<String, &'static include_dir::Dir<'static>>`
    - `embedded_mounts_by_semantic: HashMap<String, Vec<EmbeddedSemanticMount>>`
    - `content_cache: Arc<dashmap::DashMap<String, Arc<str>>>`
  - Added generic mount API:
    - `mount(crate_id, dir, semantic_mounts)`
  - `mount_embedded_dir()` now mounts Zhixing embedded resources via metadata export, rather than toggling a flag.
  - `read_semantic()` flow:
    1. parse + canonicalize URI,
    2. check interning cache,
    3. check preloaded root index,
    4. resolve from mounted `Dir` with `get_file`,
    5. intern and return `Arc<str>`.

2. Zhixing resource metadata export for resolver mounting
- `packages/rust/crates/xiuxian-wendao/src/skill_vfs/zhixing/resources.rs`
  - Added:
    - `ZHIXING_EMBEDDED_CRATE_ID`
    - `embedded_resource_dir() -> &'static Dir<'static>`
    - `embedded_semantic_reference_mounts() -> &'static HashMap<String, Vec<PathBuf>>`
  - Keeps one-time `OnceLock` mount index build.

3. Module re-export updates for internal wiring
- `packages/rust/crates/xiuxian-wendao/src/skill_vfs/zhixing/mod.rs`
  - Re-exported internal mounting helpers used by resolver.

4. Dependency addition
- `packages/rust/crates/xiuxian-wendao/Cargo.toml`
  - Added `dashmap = "7.0.0-rc2"` for concurrent interning cache.

## Design Notes

- Resolver runtime semantic reads no longer depend on `std::fs::read_to_string`.
- Disk reads remain in `SkillNamespaceIndex::build_from_roots(...)` preload stage by design, but post-build resolver reads are memory-backed.
- `read_utf8()` remains compatibility API over `read_semantic()`.

## Validation Evidence

1. Strict clippy (`xiuxian-wendao`)

```bash
CARGO_TARGET_DIR=target/clippy-xiuxian-wendao cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`

2. Resolver contract tests (`xiuxian-wendao`)

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-wendao cargo nextest run -p xiuxian-wendao --test test_skill_vfs_resolver
```

- Exit code: `0`
- Result: `7 passed`, `0 failed`

3. Asset request tests (`xiuxian-wendao`)

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-wendao cargo nextest run -p xiuxian-wendao --test test_asset_request_api
```

- Exit code: `0`
- Result: `6 passed`, `0 failed`

4. Downstream semantic hydration (`xiuxian-qianhuan`)

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-qianhuan cargo nextest run -p xiuxian-qianhuan --features zhenfa-router --test test_zhenfa_native_tools
```

- Exit code: `0`
- Result: `5 passed`, `0 failed`

5. Scheduler semantic placeholder lane (`xiuxian-qianji`)

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-qianji cargo nextest run -p xiuxian-qianji --test test_scheduler_preflight
```

- Exit code: `0`
- Result: `3 passed`, `0 failed`

## Outcome

- Resolver architecture now matches mounted-memory VFS direction from ADR-007.
- Semantic reads are cache-first and `Arc<str>`-shared, minimizing duplicate allocations under concurrent access.
- Cross-crate behavior remains stable under targeted integration lanes.
