# 326. xiuxian-qianji leaf dispatch route coverage expansion (2026-03-04)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `tests/test_compiler_dispatch_routes.rs`
- Goal: expand compiler-level integration coverage for leaf dispatch routes so
  mechanism routing regressions are caught before runtime.

## Implementation

1. Added leaf-route success coverage:
   - `write_file` compile path
   - `suspend` compile path
   - `router` compile path
   - `wendao_ingester` compile path
   - `wendao_refresh` compile path
2. Added leaf-route error coverage:
   - router branch weight validation failure via invalid branch payload
     (`"not-a-number"`), asserting topology error text contains
     `Router branch weight`.
3. Kept the test layout policy:
   - all new checks are in top-level crate integration tests
     (`tests/`), no inline `#[cfg(test)]` was introduced in source modules.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `12 passed`, `0 skipped`, `0 failed`

## Outcome

- Compiler dispatch route tests now cover both broader leaf success lanes and a
  concrete leaf error contract, increasing confidence in resolver-chain
  refactors.
- Regression detection moved earlier to compile-time route tests instead of
  waiting for scheduler/runtime execution failures.
