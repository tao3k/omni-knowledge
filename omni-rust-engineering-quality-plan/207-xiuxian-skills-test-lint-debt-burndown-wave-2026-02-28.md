# 207. Xiuxian-Skills Test Lint-Debt Burndown Wave (2026-02-28)

## Scope

This wave continued strict-lint convergence for `xiuxian-skills` after package
rename follow-up. The focus stayed on root-cause cleanup of test `unwrap/expect`
usage without adding lint suppressions.

## What Changed

1. Converted multiple test modules to `Result`-style tests with `?` propagation
   and explicit option checks:
   - `packages/rust/crates/xiuxian-skills/tests/skill_scanner.rs`
   - `packages/rust/crates/xiuxian-skills/tests/test_skill_scanner.rs`
   - `packages/rust/crates/xiuxian-skills/tests/test_schema_validation.rs`
   - `packages/rust/crates/xiuxian-skills/tests/test_benchmark.rs`
   - `packages/rust/crates/xiuxian-skills/tests/test_schema_generation.rs`
2. Removed `unwrap/expect` usage from multiple unit-test modules under
   `src/**/tests.rs`:
   - `packages/rust/crates/xiuxian-skills/src/knowledge/scanner/tests.rs`
   - `packages/rust/crates/xiuxian-skills/src/skills/tools/tests/scan_scripts.rs`
   - `packages/rust/crates/xiuxian-skills/src/skills/tools/tests/parse_content.rs`
   - `packages/rust/crates/xiuxian-skills/src/skills/tools/tests/scan_paths.rs`
   - `packages/rust/crates/xiuxian-skills/src/skills/metadata/records/reference/tests.rs`
   - `packages/rust/crates/xiuxian-skills/src/skills/prompt/tests.rs`
   - `packages/rust/crates/xiuxian-skills/src/skills/resource/tests.rs`
   - `packages/rust/crates/xiuxian-skills/src/skills/scanner/references/tests.rs`
   - `packages/rust/crates/xiuxian-skills/src/skills/skill_command/parser/tests.rs`
3. Split-wave cleanup in `packages/rust/crates/xiuxian-skills/tests/tools_scanner.rs`
   (tail section), removing a large chunk of `unwrap/expect` debt in
   annotation/schema/serialization test blocks.

## Validation Evidence

Commands executed:

1. `cargo fmt -p xiuxian-skills`
2. `CARGO_TARGET_DIR=target/nextest-xiuxian-skills cargo nextest run -p xiuxian-skills`
3. `CARGO_TARGET_DIR=target/nextest-xiuxian-skills cargo nextest run -p xiuxian-skills -E 'binary(tools_scanner)' --no-tests=pass`
4. `CARGO_TARGET_DIR=target/clippy-xiuxian-skills cargo clippy -p xiuxian-skills --all-targets -- -W clippy::too_many_lines`

Outcomes:

- Full crate `nextest` passed: `189 passed`, `0 failed`, `0 skipped`.
- `tools_scanner` test binary passed: `45 passed`, `0 failed`, `0 skipped`.
- Strict clippy still fails, but error concentration is now narrowed:
  - `tools_scanner` target currently reports `154` errors, `42` warnings.
  - Previous checkpoint for the same target was `205` errors, `52` warnings.
  - Net reduction in this slice: `-51` errors, `-10` warnings.

## Result

`xiuxian-skills` lint-debt surface has been materially reduced and consolidated
into the remaining `tools_scanner` historical lane, enabling continued
deterministic burndown by file/section without suppression shortcuts.
