# 206. Rename Follow-Up Test/Lint Convergence for Macros and Wendao Wave (2026-02-28)

## Scope

After the package-rename audit closure, this wave focused on immediate
post-rename clippy failures in two touched crates:

- `xiuxian-macros`
- `xiuxian-wendao`

Goal: remove `unwrap/expect` violations in tests via root-cause fixes (no lint
suppression) and revalidate with crate-local clippy + nextest evidence.

## What Changed

1. `xiuxian-macros` test cleanup:
   - Replaced `unwrap()` in temp-dir cleanup with explicit error handling and
     panic context.
   - Removed constant assertion anti-pattern in benchmark test and kept
     duration-value assertion semantics.
2. `xiuxian-wendao` test cleanup:
   - Replaced `expect_err(...)` usage with direct `matches!(..., Err(...))`
     assertions in URI parser and resolver contract tests.

Edited files:

- `packages/rust/crates/xiuxian-macros/tests/test_macros.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_uri.rs`
- `packages/rust/crates/xiuxian-wendao/tests/test_skill_vfs_resolver.rs`

## Validation Evidence

Commands executed:

1. `cargo fmt -p xiuxian-macros -p xiuxian-wendao`
2. `CARGO_TARGET_DIR=target/clippy-rename cargo clippy -p xiuxian-macros --all-targets -- -W clippy::too_many_lines`
3. `CARGO_TARGET_DIR=target/clippy-rename cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::too_many_lines`
4. `CARGO_TARGET_DIR=target/nextest-rename cargo nextest run -p xiuxian-macros -p xiuxian-wendao`

Outcomes:

- Formatting passed.
- Clippy passed for both crates (`0` warnings, `0` errors in this run).
- Nextest passed (`309 passed`, `0 failed`, `1 skipped`).

## Follow-Up

`xiuxian-skills` still reports large historical test-lint debt in strict clippy
probes (dominant `unwrap/expect` patterns). Keep it as a dedicated split wave
instead of mixing it into rename-closure fixes.
