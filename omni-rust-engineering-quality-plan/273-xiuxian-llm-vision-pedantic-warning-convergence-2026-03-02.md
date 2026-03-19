# 273. xiuxian-llm Vision Pedantic Warning Convergence (2026-03-02)

## Scope

- Crate:
  - `packages/rust/crates/xiuxian-llm`
- Objective:
  - close pedantic warnings surfaced in vision preprocessing and prompt docs,
  - keep zero-suppression policy (root-cause fixes only),
  - preserve behavior with targeted regression evidence.

## Changes

### 1) Removed cast-related warning sources in resize logic

Updated:

- `packages/rust/crates/xiuxian-llm/src/llm/vision/preprocess.rs`

Actions:

- changed `PreparedVisionImage::scale` from `f32` to `f64`,
- changed `fit_dimensions` return type from `(u32, u32, f32)` to `(u32, u32, f64)`,
- replaced float-cast resize math with integer rounding helper
  `fit_edge_with_rounding` using `u64` arithmetic:
  - no `u32 as f32`,
  - no `f32 as u32`,
  - deterministic nearest-integer scaling.

### 2) Closed doc-markdown warnings in vision API docs

Updated:

- `packages/rust/crates/xiuxian-llm/src/llm/vision/cot.rs`
- `packages/rust/crates/xiuxian-llm/src/llm/vision/refiner.rs`
- `packages/rust/crates/xiuxian-llm/src/llm/vision/message.rs`

Actions:

- standardized docs from `CoT` to `` `CoT` `` in public comments.

### 3) Removed avoidable allocation warning in prompt assembly

Updated:

- `packages/rust/crates/xiuxian-llm/src/llm/vision/cot.rs`

Actions:

- replaced `prompt.push_str(&format!(...))` with direct `String::push_str` assembly
  in `push_anchor_line`, removing `format_push_string` warning path.

### 4) Synced test assertions to new scale precision type

Updated:

- `packages/rust/crates/xiuxian-llm/tests/llm_vision.rs`

Actions:

- updated epsilon assertion to `f64::EPSILON` for `fit_dimensions` scale checks.

## Validation Evidence

### 1) Targeted nextest

```bash
cargo nextest run -p xiuxian-llm --test llm_vision
```

Result:

- `8 passed`, `0 failed`, `0 skipped`.

### 2) Mandatory touched-crate clippy gate

```bash
cargo clippy -p xiuxian-llm -- -W clippy::too_many_lines
```

Result:

- succeeded (exit 0),
- no pedantic warning output for this crate in this slice.

## Outcome

- vision resize pipeline no longer depends on lossy cast chains,
- doc-markdown and string-format allocation warnings are closed without
  `#[allow(...)]`,
- targeted behavior remains stable under required test and clippy gates.
