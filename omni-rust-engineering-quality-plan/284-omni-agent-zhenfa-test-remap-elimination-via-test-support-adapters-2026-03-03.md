# 284. xiuxian-daochang zhenfa test remap elimination via test-support adapters (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Target lane: `tests/agent_zhenfa_unit.rs`
- Goal: remove `include!("../src/agent/zhenfa/mod.rs")` remap and migrate to
  stable test-support boundaries.

## Implementation

1. Added crate-internal zhenfa test adapters and visibility hardening:
   - `src/agent/zhenfa/mod.rs`
     - `valkey_hooks` upgraded to `pub(crate) mod`.
     - re-exported test helper constructors from bridge.
   - `src/agent/zhenfa/valkey_hooks.rs`
     - `ZhenfaValkeyHookConfig` + resolver/hook-builder made `pub(crate)`.
   - `src/agent/zhenfa/bridge.rs`
     - added test constructors:
       - `test_runtime_deps(...)`
       - `test_memory_reward_signal_sink(...)`
       - `test_memory_reward_signal_sink_with_valkey_backend(...)`
2. Added stable test-support wrapper:
   - `src/test_support/zhenfa.rs`
   - Exposed:
     - `ZhenfaRuntimeDeps`, `ZhenfaToolBridge` wrapper
     - `resolve_zhenfa_valkey_hook_config`, `build_zhenfa_orchestrator_hooks`
     - memory reward signal sink constructors.
3. Wired exports:
   - `src/test_support/mod.rs`
4. Migrated top-level harness:
   - `tests/agent_zhenfa_unit.rs` now package-top module entrypoint.
5. Migrated zhenfa test imports:
   - `tests/agent/zhenfa/tests.rs`
   - `tests/agent/zhenfa/valkey_hooks_tests.rs`
6. Addressed visibility warning:
   - `src/agent/memory_state.rs`: `MemoryStateBackend` widened to `pub(crate)`
     to match `ZhenfaRuntimeDeps` field reachability.

## Verification

- Targeted regression:
  - `cargo nextest run -p xiuxian-daochang --test agent_zhenfa_unit --test agent_memory_stream_consumer_unit --test nodes_warmup --test channels_managed_runtime_unit --test session_redis_backend_unit --test agent_memory_recall_metrics`
  - result: `47 passed`, `5 skipped`, `0 failed`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`
  - result: pass
- Remap debt counter:
  - `... | wc -l`
  - result: `24 -> 23`
