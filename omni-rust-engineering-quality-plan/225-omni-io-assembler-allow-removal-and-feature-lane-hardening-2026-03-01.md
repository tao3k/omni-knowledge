# 225. `omni-io` Assembler Allow Removal and Feature-Lane Hardening (2026-03-01)

## Scope

- Remove the remaining assembler-level suppression in `omni-io`:
  `#[allow(clippy::needless_pass_by_value, clippy::unnecessary_wraps)]`.
- Replace suppression-based behavior with structural API and error-path
  improvements.
- Revalidate default and `assembler` feature lanes with strict clippy and
  targeted nextest.

## Changes

1. Assembler API and error-path hardening
- File: `packages/rust/crates/omni-io/src/assembler.rs`
- Removed suppression attribute from `ContextAssembler::assemble_skill`.
- Updated `assemble_skill` signature to generic borrowed forms:
  - `main_path: impl AsRef<Path>`
  - `ref_paths: impl AsRef<[PathBuf]>`
  - `variables: impl Borrow<Value>`
- Introduced a real error path for missing/unreadable main file:
  - `IoError::NotFound` for missing main file
  - `IoError::System` for other main-file I/O failures
- Kept reference-file handling behavior (missing references accumulated in
  `missing_refs`) and template-render fallback behavior unchanged.

2. Test contract update for missing-main behavior
- File: `packages/rust/crates/omni-io/tests/test_assembler.rs`
- Updated `test_assemble_skill_missing_main_file` to assert explicit `Err`
  contract instead of previous empty-content fallback assumption.

3. Feature-lane test hygiene cleanup (strict clippy compatibility)
- Files:
  - `packages/rust/crates/omni-io/tests/test_assembler.rs`
  - `packages/rust/crates/omni-io/tests/unit/assembler_tests.rs`
- Replaced `unwrap`/`expect_err` in assembler-focused tests with explicit
  panic-on-error branching to satisfy strict `unwrap_used`/`expect_used` lanes.
- Removed redundant assembler-module inclusion from
  `packages/rust/crates/omni-io/tests/mod.rs` to avoid duplicate test wiring.
- Normalized raw string literals in `test_assembler.rs` where hash delimiters
  were unnecessary.

## Validation Evidence

1. Strict clippy (default target set)

```bash
cargo clippy -p omni-io --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`

2. Strict clippy (`assembler` feature lane)

```bash
cargo clippy -p omni-io --all-targets --features assembler -- -W clippy::too_many_lines
```

- Exit code: `0`

3. Assembler integration lane

```bash
cargo nextest run -p omni-io --features assembler --test test_assembler
```

- Exit code: `0`
- Result: `15 passed`, `0 failed`

4. Aggregated integration lane (`tests/mod.rs`)

```bash
cargo nextest run -p omni-io --features assembler --test mod
```

- Exit code: `0`
- Result: `10 passed`, `0 failed`

## Outcome

- `omni-io` assembler no longer relies on a broad lint suppression.
- Main-file read failures now have explicit typed error semantics.
- Default and feature-specific quality gates for touched lanes are green.
