# 343. xiuxian-llm deepseek engine cache-io read/write split and write contract tests (2026-03-04)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-llm`
  - `packages/rust/crates/xiuxian-qianji` (regression revalidation only)
- Target area:
  - `src/llm/vision/deepseek/native/engine/cache_io.rs` (retired)
  - `src/llm/vision/deepseek/native/engine/cache_io/mod.rs`
  - `src/llm/vision/deepseek/native/engine/cache_io/read.rs`
  - `src/llm/vision/deepseek/native/engine/cache_io/write.rs`
  - `src/llm/vision/deepseek/native/engine/mod.rs`
  - `src/llm/vision/deepseek/native/mod.rs`
  - `src/llm/vision/deepseek/test_api/native_cache.rs`
  - `src/llm/vision/deepseek/test_api/mod.rs`
  - `src/llm/vision/deepseek/mod.rs`
  - `src/test_support.rs`
  - `tests/llm_vision_deepseek_cache_write_contract_unit.rs` (new)
- Goal:
  - split deepseek cache I/O orchestration by responsibility (read vs write),
    preserve behavior, and add crate-top write-path contract tests.

## Implementation

1. Converted monolithic `engine/cache_io.rs` into directory module:
   - `cache_io/mod.rs` as interface-only re-export layer.
   - `cache_io/read.rs` for cache-hit read orchestration.
   - `cache_io/write.rs` for cache-store path and non-empty markdown guard.
2. Preserved existing core call surface:
   - `core.rs` continues consuming `read_cache_entry`,
     `store_markdown_in_cache`, and `non_empty_markdown` via `cache_io`.
3. Added explicit write-path test hook:
   - `store_markdown_in_cache_for_tests` exported through:
     - `native/engine/mod.rs`
     - `native/mod.rs`
     - `deepseek/test_api/native_cache.rs`
     - `deepseek/test_api/mod.rs`
     - `deepseek/mod.rs`
     - `src/test_support.rs`
4. Added crate-top write contract tests:
   - `tests/llm_vision_deepseek_cache_write_contract_unit.rs`.
   - validates:
     - empty markdown payload is skipped by cache write path.
     - non-empty markdown payload is written to local cache.
5. No broad lint suppression was added.

## Verification

- Formatting:
  - `cargo fmt --all`
  - result: pass
- Mandatory touched-crate lint gate:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - result: pass
- Targeted deepseek tests:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-llm --test llm_vision_deepseek_runtime_unit --test llm_vision_deepseek_config_unit --test llm_vision_deepseek_cache_key_unit --test llm_vision_deepseek_valkey_failures_unit --test llm_vision_deepseek_local_cache_policy_unit --test llm_vision_deepseek_cache_text_normalization_unit --test llm_vision_deepseek_cache_write_contract_unit`
  - result: `27 passed`, `0 failed`
- Regression revalidation for qianji dispatch contracts:
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --test test_compiler_dispatch_routes --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `19 passed`, `0 failed`
  - `CARGO_TARGET_DIR=.cache/target-qianji-clippy cargo nextest run -p xiuxian-qianji --features llm --test test_compiler_dispatch_routes_llm`
  - result: `3 passed`, `0 failed`

## Outcome

- Deepseek cache I/O layer now follows single-responsibility separation across
  read and write paths.
- Cache write behavior is protected by crate-top contract tests.
- Strict clippy and targeted regression gates remain green.
