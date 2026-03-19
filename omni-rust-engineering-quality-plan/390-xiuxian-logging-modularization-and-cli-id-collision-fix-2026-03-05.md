# 390. `xiuxian-logging` modularization and CLI ID-collision fix (2026-03-05)

## Scope

- Goal:
  - keep Rust crate entry modules (`lib.rs`/`mod.rs`) interface-only,
  - complete unified logging rollout without breaking existing CLI contracts,
  - preserve high modularity for runtime/bootstrap code.

- Touched crates:
  - `xiuxian-logging`
  - `xiuxian-tui`
  - validation on `xiuxian-wendao` CLI integration lane

## Implementation

1. `xiuxian-tui` module boundary hardening:
   - moved runtime implementation out of `src/lib.rs` into `src/runtime.rs`.
   - `src/lib.rs` now exposes only module declarations and re-exports:
     - `pub mod runtime;`
     - `pub use runtime::{init_logger, run_tui};`
   - this enforces the project rule that root modules stay declarative and structured.

2. Unified logging regression fix (`clap` arg ID collision):
   - root cause:
     - `LogCliArgs` used field name `verbose: u8` with `ArgAction::Count`.
     - some binaries already define a domain flag named `verbose` as `bool`.
     - clap keyed both by the same internal argument ID (`verbose`), causing runtime panic:
       - "Mismatch between definition and access of `verbose`. Could not downcast to u8, need to downcast to bool"
   - fix:
     - renamed logging field to `log_verbose: u8` while preserving UX flags:
       - `-v`
       - `--log-verbose`
     - kept existing domain `--verbose` semantics untouched.
     - updated conversion path from CLI args to `LogSettings` accordingly.

3. Documentation/lint hygiene:
   - added crate-level doc comment in `xiuxian-logging/tests/logging_args.rs`
   - removed `missing_docs` warning from the new logging test target.

## Verification

- Compile checks:
  - `cargo check -p xiuxian-logging -p xiuxian-tui -p xiuxian-wendao`
  - result: pass

- Mandatory clippy gate:
  - `cargo clippy -p xiuxian-logging -- -W clippy::too_many_lines`
  - result: pass
  - `cargo clippy -p xiuxian-tui -- -W clippy::too_many_lines`
  - result: pass
  - `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`
  - result: pass

- Logging crate tests:
  - `cargo nextest run -p xiuxian-logging --test logging_args`
  - result: `4 passed`, `0 failed`

- TUI regression lane:
  - `cargo nextest run -p xiuxian-tui`
  - result: `43 passed`, `0 failed`

- Wendao CLI compatibility lane:
  - first run (before fix):
    - `cargo nextest run -p xiuxian-wendao --test test_wendao_cli`
    - result: failed with clap downcast panic caused by `verbose` ID collision.
  - second run (after fix):
    - `cargo nextest run -p xiuxian-wendao --test test_wendao_cli`
    - result: `34 passed`, `0 failed`

## Outcome

- `lib.rs`/module structure is now stricter and more modular for the new logging rollout path.
- Unified logging no longer breaks existing `--verbose` domain flags.
- Shared logging UX is stable across crates, and compatibility is validated on real CLI regression suites.
