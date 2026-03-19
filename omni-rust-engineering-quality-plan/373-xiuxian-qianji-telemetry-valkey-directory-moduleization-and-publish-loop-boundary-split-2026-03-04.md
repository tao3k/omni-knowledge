# 373. xiuxian-qianji telemetry valkey directory moduleization and publish-loop boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed:
    - `src/telemetry/valkey.rs`
  - added:
    - `src/telemetry/valkey/mod.rs`
    - `src/telemetry/valkey/publish.rs`
- Goal:
  - replace mixed Valkey pulse emitter implementation with concern-split
    modules while preserving public `ValkeyPulseEmitter` API and non-blocking
    telemetry behavior.

## Implementation

1. Converted `telemetry::valkey` into a directory module:
   - `mod.rs` now owns emitter struct, constructor/getter APIs, adaptive
     sampling, and `PulseEmitter` trait implementation.
2. Extracted transport-side publishing responsibilities:
   - `publish.rs`:
     - background publish loop, publish retry path, reconnect backoff,
       connection establishment, and one-shot publish command.
3. API compatibility preserved:
   - external path remains:
     `xiuxian_qianji::telemetry::ValkeyPulseEmitter`.
4. Behavior preserved:
   - queue-based non-blocking emission (`try_send`), backpressure drop
     accounting, and reconnect-on-publish-failure flow remain unchanged.
5. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/telemetry/valkey/mod.rs packages/rust/crates/xiuxian-qianji/src/telemetry/valkey/publish.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Telemetry/swarm regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_swarm_orchestration --test test_scheduler_affinity_failover`
  - result: `5 passed`, `0 failed` (`1 skipped`)
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- `telemetry::valkey` now follows interface-only module entry with explicit
  boundaries between emitter-side admission/sampling and publisher-side
  transport/reconnect behavior.
- Scheduler/swarm telemetry lanes remain stable under targeted and core qianji
  regression suites.
