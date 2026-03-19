# 297. xiuxian-qianji compiler feature split removes unused-self suppression (2026-03-03)

## Scope

- Crate: `packages/rust/crates/xiuxian-qianji`
- Target file: `src/engine/compiler.rs`
- Goal: remove remaining `cfg_attr(... allow(clippy::unused_self))` suppressions
  in mechanism builders while preserving behavior under both `llm` and
  non-`llm` feature sets.

## Implementation

1. Split `formal_audit` builder by feature:
   - `#[cfg(feature = "llm")]` keeps `&self` variant for LLM-augmented audit.
   - `#[cfg(not(feature = "llm"))]` provides static variant without `self`.
   - `build_mechanism` dispatch now uses cfg-specific call sites.
2. Split `llm` builder by feature:
   - `#[cfg(feature = "llm")]` keeps `&self` variant and resolves runtime client.
   - `#[cfg(not(feature = "llm"))]` provides static variant returning feature
     requirement error.
   - `build_mechanism` dispatch now uses cfg-specific call sites.
3. Removed suppression attributes:
   - no remaining `allow(clippy::unused_self)` in `compiler.rs`.

## Verification

- Local suppression audit:
  - `rg -n "allow\\(" packages/rust/crates/xiuxian-qianji/src/engine/compiler.rs`
  - result: no matches
- Global Rust suppression audit:
  - `rg -n "#\\[allow\\(|#!\\[allow\\(|cfg_attr\\([^\\)]*allow\\(" packages/rust/crates -g "*.rs" | wc -l`
  - result: `0`
- Mandatory touched-crate lint gate:
  - `cargo clippy -p xiuxian-qianji -- -W clippy::too_many_lines`
  - result: pass
- Targeted regression:
  - `cargo nextest run -p xiuxian-qianji --test test_probabilistic_routing --test test_qianji_yaml_orchestration`
  - result: `2 passed`, `0 skipped`, `0 failed`

## Outcome

- Compiler feature boundaries are now encoded in function structure instead of
  lint suppression.
- `xiuxian-qianji` remains behavior-compatible with targeted regression proof.
