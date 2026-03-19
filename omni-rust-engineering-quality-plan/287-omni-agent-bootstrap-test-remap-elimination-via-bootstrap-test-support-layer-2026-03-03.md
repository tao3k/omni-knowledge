# 287. xiuxian-daochang bootstrap test remap elimination via bootstrap test-support layer (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Target lane: `tests/agent_bootstrap.rs`
- Goal: remove `service_mount` and `qianhuan` source includes, and converge bootstrap tests to stable package-top integration boundaries.

## Implementation

1. Enabled crate-internal bootstrap access for test adapters:
   - `src/agent/mod.rs`: `bootstrap` -> `pub(crate) mod bootstrap`
   - `src/agent/bootstrap.rs`: exposed minimal submodules as `pub(crate)`
2. Exposed minimal bootstrap internals for test-support routing:
   - `src/agent/bootstrap/hot_reload/mod.rs`: re-exported watch-policy helpers
   - `src/agent/bootstrap/hot_reload/prepared.rs`: watch-policy helper visibility to `pub(crate)`
   - `src/agent/bootstrap/memory.rs`: `resolve_memory_embed_base_url` -> `pub(crate)`
   - `src/agent/bootstrap/qianhuan.rs`: `init_persona_registries` + `internal_len()` test-facing accessor
   - `src/agent/bootstrap/service_mount.rs`: `ServiceMountCatalog` visibility/methods to `pub(crate)`
   - `src/agent/bootstrap/zhenfa.rs`: `build_skill_vfs_resolver_from_roots` -> `pub(crate)`
   - `src/agent/bootstrap/zhixing.rs`: exposed test-path helper functions and skill-template load summary fields to `pub(crate)`
3. Added stable bootstrap test-support boundary:
   - `src/test_support/bootstrap.rs`
   - Added wrapper catalog `BootstrapServiceMountCatalog`
   - Added helper surface for hot-reload policy, zhixing paths/templates, persona-registry count, and skill-vfs resolver bootstrapping
4. Wired exports:
   - `src/test_support/mod.rs`
5. Migrated bootstrap tests to `xiuxian_daochang::test_support` API:
   - `tests/agent/bootstrap/tests.rs`
6. Replaced top-level harness with standard package-top entrypoint:
   - `tests/agent_bootstrap.rs`
   - Reduced to `#[path = "agent/bootstrap/tests.rs"] mod tests;`
7. Documentation debt cleanup:
   - `src/agent/bootstrap/service_mount.rs`
   - Added docs for `ServiceMountStatus` variants instead of lint suppression.

## Verification

- Targeted regression:
  - `cargo nextest run -p xiuxian-daochang --test agent_bootstrap --test agent_memory_recall_state_unit`
  - result: `22 passed`, `2 skipped`, `0 failed`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass
- Remap debt counter:
  - `rg -n "include!\(\"\.\./src/|#\[path\s*=\s*\"\.\./src/|#\[path\s*=\s*\"\.\./\.\./src/" packages/rust/crates/xiuxian-daochang/tests --glob "*.rs" | wc -l`
  - result: `21 -> 19`
