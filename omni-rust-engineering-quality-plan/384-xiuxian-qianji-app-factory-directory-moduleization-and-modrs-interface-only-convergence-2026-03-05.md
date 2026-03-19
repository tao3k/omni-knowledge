# 384. xiuxian-qianji app-factory directory moduleization and modrs interface-only convergence (2026-03-05)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-qianji`
- Target area:
  - `src/lib.rs`
  - `src/app/mod.rs`
  - `src/app/build.rs`
  - `src/app/presets.rs`
  - `src/app/qianji_app.rs`
- Goal:
  - extract `QianjiApp` factory logic from `lib.rs` into a focused directory
    module and enforce repository rule that `mod.rs` remains interface-only.

## Implementation

1. Replaced mixed root-module implementation with explicit app module boundary:
   - introduced `src/app/` directory module,
   - moved `QianjiApp` implementation out of `lib.rs`,
   - moved built-in pipeline constants to `app/presets.rs`.

2. Enforced interface-only `mod.rs` contract:
   - `src/app/mod.rs` now only declares child modules and re-exports
     (`QianjiApp`, `RESEARCH_TRINITY_TOML`, `MEMORY_PROMOTION_PIPELINE_TOML`),
   - all business logic lives in child files.

3. Concern split:
   - `app/build.rs`:
     - common scheduler compilation path (compiler build + engine compile +
       optional consensus manager wrapping),
   - `app/qianji_app.rs`:
     - public factory API methods and preset-specific entry points,
   - `app/presets.rs`:
     - static built-in manifest payloads.

4. Public API compatibility preserved:
   - crate-root re-exports unchanged for callers:
     - `xiuxian_qianji::QianjiApp`
     - `xiuxian_qianji::RESEARCH_TRINITY_TOML`
     - `xiuxian_qianji::MEMORY_PROMOTION_PIPELINE_TOML`

5. No lint-suppression attributes were introduced.

## Verification

- Formatting:
  - `rustfmt --edition 2024 packages/rust/crates/xiuxian-qianji/src/lib.rs packages/rust/crates/xiuxian-qianji/src/app/mod.rs packages/rust/crates/xiuxian-qianji/src/app/build.rs packages/rust/crates/xiuxian-qianji/src/app/presets.rs packages/rust/crates/xiuxian-qianji/src/app/qianji_app.rs`
  - result: pass
- Type/syntax gate:
  - `cargo check -p xiuxian-qianji`
  - result: pass
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Extended lint surface check:
  - `cargo clippy -p xiuxian-qianji --all-targets --features llm -- -W clippy::too_many_lines`
  - result: pass
- Regression lanes:
  - `cargo nextest run -p xiuxian-qianji --test test_memory_promotion_pipeline --test test_agenda_validation_pipeline --test runtime_config --test test_compiler_dispatch_routes`
  - result: `31 passed`, `0 failed`
  - `cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm --test test_memory_promotion_pipeline`
  - result: `5 passed`, `0 failed`

## Outcome

- `lib.rs` is thinner and focused on module exposure/re-export responsibilities.
- App-level scheduler factory concerns now have explicit internal boundaries.
- `mod.rs` interface-only rule is restored in this slice.
