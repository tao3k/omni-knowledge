# 293. rust inline-test zero convergence across xiuxian-config-core and xiuxian-llm (2026-03-03)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-config-core`
  - `packages/rust/crates/xiuxian-llm`
- Goal: remove remaining source-inline `#[cfg(test)]` hooks and converge tests
  to package-top `tests/` binaries with stable test-support boundaries.

## Implementation

1. `xiuxian-config-core` migration:
   - Removed inline mount:
     - `src/paths.rs` (`#[cfg(test)] mod tests;`)
   - Added hidden test-support boundary:
     - `src/test_support.rs`
     - `src/lib.rs` exports `#[doc(hidden)] pub mod test_support;`
   - Moved path-helper tests to package-top:
     - added `tests/paths_unit.rs`
     - deleted `src/paths/tests.rs`
2. `xiuxian-llm` migration:
   - Removed inline mounts:
     - `src/llm/vision/deepseek/config.rs`
     - `src/llm/vision/deepseek/runtime.rs`
   - Added hidden test-support boundary:
     - `src/test_support.rs`
     - `src/lib.rs` exports `#[doc(hidden)] pub mod test_support;`
   - Added crate-internal deepseek test adapters:
     - `src/llm/vision/mod.rs` (`pub(crate) mod deepseek`)
     - `src/llm/vision/deepseek/mod.rs` test helper wrappers
     - `src/llm/vision/deepseek/config.rs` snapshot adapter
     - `src/llm/vision/deepseek/runtime.rs` helper visibility alignment
   - Moved deepseek tests to package-top:
     - added `tests/llm_vision_deepseek_config_unit.rs`
     - added `tests/llm_vision_deepseek_runtime_unit.rs`
     - deleted `src/llm/vision/deepseek/config/tests.rs`
     - deleted `src/llm/vision/deepseek/runtime/tests.rs`
3. Clippy hygiene follow-up:
   - removed unused import in `deepseek/mod.rs`
   - fixed `doc_markdown` nits in `xiuxian-llm/src/test_support.rs`

## Verification

- Inline test marker audit:
  - `rg -n "#\\[cfg\\(test\\)\\]" packages/rust/crates/*/src -g "*.rs"`
  - result: no matches (`0`)
- `xiuxian-config-core` targeted regression:
  - `cargo nextest run -p xiuxian-config-core --test paths_unit --test test_resolve --test test_cache`
  - result: `12 passed`, `0 skipped`, `0 failed`
- `xiuxian-llm` targeted regression:
  - `cargo nextest run -p xiuxian-llm --test llm_vision_deepseek_config_unit --test llm_vision_deepseek_runtime_unit --test llm_vision_deepseek_smoke`
  - result: `9 passed`, `1 skipped`, `0 failed`
- Mandatory touched-crate clippy gates:
  - `cargo clippy -p xiuxian-config-core -- -W clippy::too_many_lines`
  - `cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines`
  - result: pass for both crates

## Outcome

- Remaining source-inline test hooks in Rust crates are converged to zero.
- `tests/` package-top layout is now consistent for these two crates.
