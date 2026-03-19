# 232. `xiuxian-daochang` All-Target Warning Reduction Follow-up and Targeted Nextest Proof (2026-03-01)

## Scope

- Continue warning reduction after all-target clippy unblock.
- Address high-friction warning categories in touched tests:
  - `too_many_lines`
  - `unnecessary_wraps`
  - `missing_docs` (crate-level test module)
- Revalidate touched lanes with strict clippy and targeted nextest.

## Changes

1. Reduced `too_many_lines` pressure in valkey hook live test
- File: `packages/rust/crates/xiuxian-daochang/tests/unit/agent/zhenfa/valkey_hooks_tests.rs`
- Extracted large test-body logic into focused helpers:
  - `assert_cached_dispatch_uses_valkey_cache(...)`
  - `assert_mutation_lock_contention(...)`
- Kept existing assertions and behavior unchanged.

2. Removed `unnecessary_wraps` and `missing_docs` warnings in config defaults test lane
- File: `packages/rust/crates/xiuxian-daochang/tests/config_embedded_defaults.rs`
- Added crate-level test doc header.
- Changed `embedded_defaults_child_probe` from `Result<()>` to `()` and removed
  needless `Ok(())` return path.
- Parent integration test (`load_runtime_settings_uses_embedded_defaults_when_system_file_missing`)
  remains `Result<()>` and unchanged in behavior.

3. Follow-up polish in refresh executor docs
- File: `packages/rust/crates/xiuxian-qianji/src/executors/wendao_refresh.rs`
- Normalized doc-markdown type mentions with backticks for `LinkGraph` fields.

## Validation Evidence

1. All-target strict clippy (`xiuxian-daochang`)

```bash
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`

2. Targeted nextest (`config_embedded_defaults`)

```bash
cargo nextest run -p xiuxian-daochang --test config_embedded_defaults
```

- Exit code: `0`
- Result: `2 passed`, `0 failed`

3. Targeted nextest (`valkey_hooks` focused subset in lib target)

```bash
cargo nextest run -p xiuxian-daochang --lib -E 'test(resolve_zhenfa_valkey_hook_config_returns_none_without_url) | test(resolve_zhenfa_valkey_hook_config_applies_defaults) | test(build_zhenfa_orchestrator_hooks_returns_hooks_when_url_is_configured)'
```

- Exit code: `0`
- Result: `3 passed`, `261 skipped`

## Outcome

- The previously touched all-target warning categories are reduced with no
  suppression-based shortcuts.
- `xiuxian-daochang` all-target strict clippy remains green.
- Regression proof exists for both modified test lanes (`config_embedded_defaults`
  and `valkey_hooks` subset).
