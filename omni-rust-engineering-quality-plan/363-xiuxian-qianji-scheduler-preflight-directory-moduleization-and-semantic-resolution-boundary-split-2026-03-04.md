# 363. xiuxian-qianji scheduler preflight directory moduleization and semantic-resolution boundary split (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - removed: `src/scheduler/preflight.rs`
  - added:
    - `src/scheduler/preflight/mod.rs`
    - `src/scheduler/preflight/context_path.rs`
    - `src/scheduler/preflight/mounts.rs`
    - `src/scheduler/preflight/wendao_uri.rs`
    - `src/scheduler/preflight/query.rs`
    - `src/scheduler/preflight/semantic.rs`
- Goal:
  - replace mixed-responsibility scheduler preflight module with concern-split
    directory modules while preserving internal scheduler/preflight APIs and
    runtime semantic-resolution behavior.

## Implementation

1. Converted `scheduler::preflight` to directory module:
   - `mod.rs` is interface-only and re-exports existing scheduler-facing APIs:
     - `RuntimeWendaoMount`
     - `install_runtime_wendao_mounts`
     - `resolve_wendao_placeholders_in_context`
     - `resolve_semantic_content`
     - `resolve_semantic_reference`
     - `lookup_context_path`
     - `context_value_to_text`
     - `resolve_wendao_uri_with_zhenfa`
2. Split responsibilities by concern:
   - `mounts.rs`:
     - runtime mount registry, install guard, and lifecycle handling.
   - `context_path.rs`:
     - context path parser + path lookup + value-to-text conversion.
   - `wendao_uri.rs`:
     - runtime mount + embedded fallback URI dereference and Zhenfa bridge.
   - `query.rs`:
     - dynamic query expansion into XML-Lite aggregated payload.
   - `semantic.rs`:
     - recursive placeholder traversal and semantic/reference resolution policy.
3. API and behavior compatibility:
   - call-sites in scheduler state, bootcamp workflow, executors (`annotation`,
     `command`, `llm`, `write_file`) remain unchanged at import level.
4. No broad lint suppressions introduced.

## Verification

- Formatting:
  - `rustfmt packages/rust/crates/xiuxian-qianji/src/scheduler/preflight/mod.rs packages/rust/crates/xiuxian-qianji/src/scheduler/preflight/context_path.rs packages/rust/crates/xiuxian-qianji/src/scheduler/preflight/mounts.rs packages/rust/crates/xiuxian-qianji/src/scheduler/preflight/query.rs packages/rust/crates/xiuxian-qianji/src/scheduler/preflight/semantic.rs packages/rust/crates/xiuxian-qianji/src/scheduler/preflight/wendao_uri.rs`
  - result: pass
- Tier-2 compile check:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Preflight/bootcamp focused regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_scheduler_preflight --test test_bootcamp_api`
  - result: `8 passed`, `0 failed`
- Core qianji regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Scheduler preflight now follows interface-only module entry with explicit
  boundaries for mount registry, path parsing, URI dereference, query expansion,
  and placeholder-resolution policy.
- Runtime semantic behavior remains stable across preflight, bootcamp, and
  compiler dispatch execution lanes.
