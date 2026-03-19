# 174. Xiuxian-Wendao Dependency Indexer TOML-Only Config Migration Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-wendao`
- Focus:
  - dependency-indexer config loading path
  - dependency-indexer default config path
  - dependency-indexer Rust/Py test fixtures
  - system config source alignment (`packages/rust/crates/xiuxian-daochang/resources/config/xiuxian.toml`)

## Why This Wave

Dependency indexer still referenced `references.yaml`, while current project
configuration policy is TOML-first (`xiuxian.toml`) for this runtime lane.

## Changes Implemented

1. Switched dependency-indexer config parser to TOML:
   - file: `src/dependency_indexer/config.rs`
   - replaced YAML value-walk parsing with typed TOML deserialization:
     - `DependencyConfigFile { ast_symbols_external: Vec<...> }`
   - kept the existing key contract: `ast_symbols_external`
   - kept defensive filtering (`pkg_type` non-empty and `manifests` non-empty)

2. Switched default dependency-indexer config path:
   - file: `src/dependency_indexer/indexer/core/build.rs`
   - from `packages/rust/crates/xiuxian-daochang/resources/config/xiuxian.toml`
   - to `packages/rust/crates/xiuxian-daochang/resources/config/xiuxian.toml`

3. Migrated dependency-indexer tests to TOML fixtures and paths:
   - `tests/test_dependency_debug.rs`
   - `tests/test_dependency_integration.rs`
   - `tests/test_dependency_indexer.rs`
   - `src/dependency_indexer/indexer/tests.rs`
   - `src/dep_indexer_py/tests.rs`
   - all fixture content moved from YAML list form to TOML array-of-tables:
     - `[[ast_symbols_external]]`

4. Updated Python bridge docs for config loader:
   - `src/dep_indexer_py/config.rs`
   - comment updated from YAML to TOML wording

5. Added dependency-indexer external manifest section to
   `packages/rust/crates/xiuxian-daochang/resources/config/xiuxian.toml`:
   - `[[ast_symbols_external]]` for Rust (`cargo`)
   - `[[ast_symbols_external]]` for Python (`pip`)

## Validation Evidence

1. Format:

```bash
cargo fmt -p xiuxian-wendao
```

- Result: pass

2. Strict clippy (required lane):

```bash
CARGO_TARGET_DIR=target/clippy-wendao cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass

3. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-wendao cargo nextest run -p xiuxian-wendao
```

- Result: pass
- Summary: `286 passed`, `0 failed`, `1 skipped`

## Engineering Outcome

- Dependency-indexer no longer depends on `references.yaml` in code paths or
  tests.
- Config contract is unified on TOML for this lane.
- Validation remains green under strict clippy and `nextest`.
