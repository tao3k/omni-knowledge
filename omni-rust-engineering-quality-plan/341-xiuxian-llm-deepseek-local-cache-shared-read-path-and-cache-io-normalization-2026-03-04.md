# 341. xiuxian-llm deepseek local-cache shared-read path and cache-io normalization (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/llm/vision/deepseek/native/cache/local/mod.rs`
  - `src/llm/vision/deepseek/native/cache/local/storage.rs`
  - `src/llm/vision/deepseek/native/cache/mod.rs`
  - `src/llm/vision/deepseek/native/engine/cache_io.rs`
  - `src/llm/vision/deepseek/native/engine/image_decode.rs`
- Goal:
  - reduce redundant allocations on deepseek local-cache read paths without
    changing external behavior (`Option<String>` contract at engine boundary).

## Implementation

1. Added shared local-cache read primitive:
   - `local/storage.rs` now exposes `get_shared(key) -> Option<Arc<str>>`.
   - `local/mod.rs` adds `local_get_shared` to surface shared reads.
   - `local/storage.rs` `insert(...)` now interns directly with
     `Arc::<str>::from(markdown)` (no intermediate `String` allocation).
2. Updated cache interface exports:
   - `native/cache/mod.rs` now re-exports `local_get_shared` for engine usage.
3. Refactored `cache_io` read path normalization:
   - local lane now reads via `Arc<str>` and only allocates when emitting final
     normalized `String`.
   - valkey lane now avoids unnecessary re-allocation when payload is already
     trim-normalized (`normalize_owned_cache_text` keeps original `String`).
   - introduced `complete_cache_read` to centralize empty-entry telemetry and
     cache-hit telemetry while preserving valkey->local backfill behavior.
4. Cleaned a pedantic warning in deepseek image decode config parsing:
   - `prefer_original_payload` switched to `is_none_or(...)` style to keep
     touched-crate clippy output warning-free in this slice.
5. No broad lint suppression was added.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - result: pass
- Targeted deepseek tests:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-llm --test llm_vision_deepseek_runtime_unit --test llm_vision_deepseek_config_unit --test llm_vision_deepseek_cache_key_unit --test llm_vision_deepseek_valkey_failures_unit --test llm_vision_deepseek_local_cache_policy_unit`
  - result: `22 passed`, `0 failed`
- Regression revalidation for qianji dispatch contracts:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Deepseek local-cache read path now follows a shared-string-first pattern and
  avoids avoidable clone/re-allocate cycles.
- Cache read normalization remains deterministic, and cache telemetry semantics
  are preserved.
- Strict clippy and targeted test gates remain green.
