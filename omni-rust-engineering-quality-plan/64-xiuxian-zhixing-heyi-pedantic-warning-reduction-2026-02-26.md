# Xiuxian-Zhixing Heyi Pedantic Warning Reduction (2026-02-26)

## Scope

This shard records a targeted pedantic-warning reduction pass in
`xiuxian-zhixing`, focused on low-risk, behavior-preserving lint fixes in
`heyi` rendering/task paths plus test `expect()` cleanup in the Wendao indexer
lane.

Targets:

- `packages/rust/crates/xiuxian-zhixing/src/heyi/agenda_render.rs`
- `packages/rust/crates/xiuxian-zhixing/src/heyi/reminders.rs`
- `packages/rust/crates/xiuxian-zhixing/src/heyi/tasks.rs`
- `packages/rust/crates/xiuxian-zhixing/tests/test_wendao_indexer.rs`

## Changes Implemented

### 1) Replaced `map(...).unwrap_or_else(...)` with `map_or_else(...)`

Actions:

- Updated agenda scheduled-time rendering fallback.
- Updated reminder persona-name fallback rendering.

### 2) Fixed doc-markdown pedantic warning

Actions:

- Reworded reminder doc comment to use backticks for `MarkdownV2`.

### 3) Collapsed nested enqueue error branch

Actions:

- Rewrote reminder queue enqueue branch to use `if let ... && let Err(...)`
  style, removing collapsible-if warning without changing behavior.

### 4) Removed `expect()` from Wendao indexer integration test

Actions:

- Replaced `expect()` on optional document lookups with explicit `let Some(...)`
  checks returning typed test errors.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-zhixing
cargo clippy -p xiuxian-zhixing --all-targets -- -W clippy::pedantic
```

Results:

- `xiuxian-zhixing` no longer emits the prior `heyi` warnings
  (`map_unwrap_or`, `doc_markdown`, `collapsible_if`) in this lane.
- `test_wendao_indexer` no longer triggers `clippy::expect_used`.
- Remaining warnings in this command come from:
  - `xiuxian-qianhuan` (`uninlined_format_args`)
  - two existing `xiuxian-zhixing` test-style warnings
    (`float_cmp`, `default_constructed_unit_structs`)

## Outcome

- Pedantic warning noise in `xiuxian-zhixing/heyi` is reduced with
  behavior-stable refactors.
- Test lane moved further toward suppression-free and panic-free assertion
  style in indexer coverage.
