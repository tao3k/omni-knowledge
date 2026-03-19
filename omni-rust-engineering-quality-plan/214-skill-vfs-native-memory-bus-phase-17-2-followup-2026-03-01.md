# 214. Skill VFS Native Memory Bus Phase 17.2 Follow-Up (2026-03-01)

## Scope

- Close the audit gap where `SkillVfsResolver` behaved as a path resolver instead of a memory-first VFS reader.
- Make embedded resource mounting explicit and deterministic.
- Reduce avoidable allocations in the semantic resource read path.
- Re-validate downstream semantic template hydration in `xiuxian-qianhuan`.

## Key Changes

1. Resolver mount semantics and zero-copy API
- `packages/rust/crates/xiuxian-wendao/src/skill_vfs/resolver.rs`
  - `from_roots(...)` now defaults to `embedded_enabled = false`.
  - Added `from_roots_with_embedded(...)` convenience constructor.
  - Kept `mount_embedded_dir(...)` as the explicit mount toggle.
  - Added `read_semantic(&self, uri: &str) -> Result<Arc<str>, SkillVfsError>` as the primary memory-backed read API.
  - `read_utf8_shared(...)` now delegates to `read_semantic(...)`.
  - `read_utf8(...)` remains a compatibility API and only converts `Arc<str>` into `String`.
  - Embedded cache key switched to canonical URI to avoid duplicate cache entries for equivalent input forms.

2. Canonical URI support for cache normalization
- `packages/rust/crates/xiuxian-wendao/src/skill_vfs/uri.rs`
  - Added `canonical_uri()` to produce normalized `wendao://skills/<name>/references/<entity>` keys.

3. Embedded resolver allocation and traversal optimization
- `packages/rust/crates/xiuxian-wendao/src/skill_vfs/zhixing/resources.rs`
  - `embedded_resource_text_from_wendao_uri(...)` now returns `Option<&'static str>` instead of `Option<String>`.
  - Added parsed variant: `embedded_resource_text_from_parsed_wendao_uri(...)`.
  - Added `OnceLock<HashMap<String, Vec<PathBuf>>>` mount index cache so semantic mount discovery is built once, not per lookup.
  - Removed per-call mount sorting/scanning in the hot path.

4. Asset request path follows borrowed embedded payloads
- `packages/rust/crates/xiuxian-wendao/src/skill_vfs/asset_request.rs`
  - `read_utf8_shared()` now builds `Arc<str>` directly from borrowed embedded payload.
  - `read_stripped_body_shared()` now strips borrowed content and avoids an intermediate full `String` allocation path.

5. Test contract updates and downstream compatibility fixes
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_resolver.rs`
  - Added explicit-mount contract test (`embedded_reference_requires_explicit_mount`).
  - Updated embedded tests to call `.mount_embedded_dir()`.
  - Added alias contract test for `read_semantic`/`read_utf8_shared`.
- `packages/rust/crates/xiuxian-qianji/tests/test_scheduler_preflight.rs`
  - Replaced `expect_err` with explicit match-based failure handling.
- `packages/rust/crates/xiuxian-qianji/tests/test_agenda_validation_pipeline.rs`
  - Migrated embedded manifest helper to `&'static str` and removed unnecessary `.as_str()` conversions.
- `packages/rust/crates/xiuxian-daochang/tests/scenario_adversarial_evolution.rs`
  - Updated callsite to consume borrowed embedded manifest directly.

## Validation Evidence

1. Strict clippy (`xiuxian-wendao`)

```bash
CARGO_TARGET_DIR=target/clippy-xiuxian-wendao cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`

2. Strict clippy (`xiuxian-qianji`)

```bash
CARGO_TARGET_DIR=target/clippy-xiuxian-qianji cargo clippy -p xiuxian-qianji --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`

3. Strict clippy (`xiuxian-daochang`)

```bash
CARGO_TARGET_DIR=target/clippy-xiuxian-daochang cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`
- Note: pre-existing warnings observed in unrelated lanes (`large_enum_variant`, `unused_async`), but strict lane completed successfully.

4. Resolver contract tests (`xiuxian-wendao`)

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-wendao cargo nextest run -p xiuxian-wendao --test test_skill_vfs_resolver
```

- Exit code: `0`
- Result: `7 passed`, `0 failed`

5. Asset request tests (`xiuxian-wendao`)

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-wendao cargo nextest run -p xiuxian-wendao --test test_asset_request_api
```

- Exit code: `0`
- Result: `6 passed`, `0 failed`

6. Semantic template hydration (`xiuxian-qianhuan`, feature-gated)

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-qianhuan cargo nextest run -p xiuxian-qianhuan --features zhenfa-router --test test_zhenfa_native_tools
```

- Exit code: `0`
- Result: `5 passed`, `0 failed`

7. Scheduler preflight semantic placeholder lane (`xiuxian-qianji`)

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-qianji cargo nextest run -p xiuxian-qianji --test test_scheduler_preflight
```

- Exit code: `0`
- Result: `3 passed`, `0 failed`

8. Agenda validation embedded manifest lane (`xiuxian-qianji`)

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-qianji cargo nextest run -p xiuxian-qianji --test test_agenda_validation_pipeline
```

- Exit code: `0`
- Result: `3 passed`, `0 failed`

## Outcome

- `SkillVfsResolver` now serves as a native memory-backed semantic VFS reader instead of runtime filesystem reader logic.
- Embedded semantic resources are explicit-mount and cache-backed with canonical URI keys.
- Hot-path embedded lookups avoid repeated mount graph reconstruction and avoid avoidable `String` churn.
- Downstream semantic template hydration and preflight placeholder resolution remain green under feature-gated and integration tests.
