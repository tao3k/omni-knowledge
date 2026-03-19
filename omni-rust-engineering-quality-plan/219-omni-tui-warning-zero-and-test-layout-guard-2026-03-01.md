# 219. `omni-tui` Warning-Zero and Rust Test-Layout Guard (2026-03-01)

## Scope

Follow-up wave after item 218:

1. finish remaining warning cleanup in `omni-tui` touched surfaces,
2. add an enforceable guard so `src` inline test implementations and
   legacy `src/**/tests*.rs` patterns do not regress.

## What Changed

### A. `omni-tui` warning-zero convergence (targeted slice)

Updated and validated in:

- `packages/rust/crates/omni-tui/src/state/mod.rs`
- `packages/rust/crates/omni-tui/src/event/mod.rs`
- `packages/rust/crates/omni-tui/src/socket.rs`
- `packages/rust/crates/omni-tui/src/renderer/mod.rs`
- `packages/rust/crates/omni-tui/src/lib.rs`
- plus previously touched test/example files from item 218.

Convergence actions:

- Added missing `#[must_use]` on relevant APIs.
- Replaced remaining redundant closures with method references.
- Continued `uninlined_format_args` normalization.
- Added missing `# Errors` docs for touched public `Result` APIs.
- Fixed doc-markdown items for Rust identifiers and type names.
- Adjusted signatures where low-risk and warning-reducing
  (`on_custom_event(&[u8])`, reference-based helpers).

### B. New test-layout guard script

Added:

- `scripts/rust/check_test_layout.sh`

Guard checks:

1. Forbid test implementation files under `src`:
   - `tests.rs`, `*_tests.rs`, `test_*.rs`
2. Forbid `src/**/tests/*.rs` trees.
3. Forbid inline `#[cfg(test)] mod ... { ... }` blocks in `src`.
4. Validate `#[cfg(test)] mod tests;` / `mod *_tests;` path mounts:
   - `#[path = "..."]` must exist,
   - target file must exist.

### C. Gate integration

Integrated the guard into quality pipelines:

- `justfile`:
  - new target `rust-test-layout`
  - `rust-quality-gate` now depends on `rust-test-layout`
- `scripts/ci/rust_quality_gate_ci.sh`:
  - added `just rust-test-layout` near lint inheritance checks.

## Verification Evidence

### Layout guard

```bash
bash scripts/rust/check_test_layout.sh
```

Result: pass.

### Strict clippy (`omni-tui`)

```bash
cargo clippy -p omni-tui --all-targets -- -W clippy::too_many_lines
```

Result: pass; no warning output for this targeted crate lane in this run.

### Targeted smoke

```bash
cargo nextest run -p omni-tui test_socket_server_start_stop
cargo nextest run -p omni-tui test_state_operations
```

Result: both pass.

## Outcome

`omni-tui` touched warning slice is converged, and the Rust test-structure
policy is now enforced as a reusable, CI-wired guard to prevent regression.
