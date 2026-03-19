# 216. 修仙道场 (Xiuxian Daochang) `src/**/tests.rs` Relocation to Top-Level `tests/unit` (2026-03-01)

## Scope

- Enforce repository test-layout policy by moving test implementation files out of `src/`.
- Keep existing unit-test semantics and private-module coverage intact.
- Avoid behavior drift while normalizing test file locations.

## Structural Changes

1. Physical relocation from `src/**/tests.rs` to package-level test tree

- Moved:
  - `src/agent/bootstrap/tests.rs` -> `tests/unit/agent/bootstrap_tests.rs`
  - `src/agent/zhenfa/tests.rs` -> `tests/unit/agent/zhenfa_tests.rs`
  - `src/config/tests.rs` -> `tests/unit/config/config_tests.rs`
  - `src/session/redis_backend/tests.rs` -> `tests/unit/session/redis_backend_tests.rs`
  - `src/agent/session_context/tests.rs` -> `tests/unit/agent/session_context_tests.rs`
  - `src/agent/turn_execution/react_loop/tests.rs` -> `tests/unit/agent/turn_execution/react_loop_tests.rs`

2. Source modules now point to top-level test files explicitly

- Updated `#[cfg(test)] mod tests;` declarations to path-based mounts:
  - `src/agent/bootstrap.rs`
  - `src/agent/zhenfa/mod.rs`
  - `src/config/mod.rs`
  - `src/session/redis_backend/mod.rs`
  - `src/agent/session_context/mod.rs`
  - `src/agent/turn_execution/react_loop/mod.rs`

3. Embedded-only SkillVFS startup contract test retained in relocated suite

- `build_skill_vfs_resolver_from_empty_roots_mounts_embedded_resources` remains active after relocation.
- This protects the no-physical-root startup contract introduced in the SkillVFS modernization wave.

## Design Note

- The `#[cfg(test)]` module mount lines are still required for unit-style private-scope testing.
- The implementation code now lives under top-level `tests/unit`, while source files keep only lightweight compile-time test wiring.

## Validation Evidence

1. Strict clippy (`xiuxian-daochang`)

```bash
CARGO_TARGET_DIR=target/clippy-xiuxian-daochang cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`

2. Embedded-only resolver contract (unit lane)

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-daochang cargo nextest run -p xiuxian-daochang --lib -E 'test(build_skill_vfs_resolver_from_empty_roots_mounts_embedded_resources)'
```

- Exit code: `0`
- Result: `1 passed`

3. Embedded registry bridge contract (integration lane)

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-daochang cargo nextest run -p xiuxian-daochang -E 'test(load_skill_templates_from_embedded_registry_uses_semantic_wendao_uri_links)'
```

- Exit code: `0`
- Result: `1 passed`

## Outcome

- `xiuxian-daochang` test files no longer physically reside under `src/**/tests.rs`.
- Test layout now aligns with package-top test directory policy while preserving existing validation breadth and behavior.
