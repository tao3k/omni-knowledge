# Omni Vector Skill Scanner Lint-Debt Reduction (2026-02-26)

## Objective

Reduce suppression debt in `xiuxian-vector` skill scanning logic by fixing clippy
root causes instead of keeping local `allow` directives.

## Scope

### Changed file

- `packages/rust/crates/xiuxian-vector/src/skill/scanner.rs`

### What changed

1. Removed `#[allow(clippy::unused_self)]` on `scan_skill`.
2. Removed `#[allow(clippy::collapsible_if)]` on `scan_all`.
3. Refactored scanner construction to keep meaningful instance state:
   - `SkillScannerModule` now stores `skill_file_name`,
   - `scan_skill` uses this field when building the file path.
4. Collapsed nested `if` in directory scan loop using idiomatic conditional
   `let` form.

## Verification Evidence

- `cargo fmt -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic`
- `cargo test -p xiuxian-vector --tests`

Result: all passed.

## Outcome

The scanner path now satisfies strict pedantic gates without extra suppressions
and has a clearer object boundary for future scanner-level configuration.
