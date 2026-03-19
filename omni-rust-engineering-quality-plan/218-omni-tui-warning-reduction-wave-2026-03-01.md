# 218. `omni-tui` Warning Reduction Wave (2026-03-01)

## Scope

Continue warning debt cleanup after strict-clippy convergence, focusing on
mechanical and low-risk fixes in `omni-tui`:

- `must_use_candidate`
- `uninlined_format_args`
- `doc_markdown`
- `redundant_closure_for_method_calls`
- `needless_borrows_for_generic_args`

No lint suppression attributes were added.

## Changed Files

- `packages/rust/crates/omni-tui/src/components/collection.rs`
- `packages/rust/crates/omni-tui/src/components/panel.rs`
- `packages/rust/crates/omni-tui/src/components/tui_app.rs`
- `packages/rust/crates/omni-tui/src/event/mod.rs`
- `packages/rust/crates/omni-tui/src/socket.rs`
- `packages/rust/crates/omni-tui/tests/unit/state/tests.rs`
- `packages/rust/crates/omni-tui/tests/unit/main_demo_tests.rs`
- `packages/rust/crates/omni-tui/tests/test_state_comprehensive.rs`
- `packages/rust/crates/omni-tui/tests/test_socket_comprehensive.rs`
- `packages/rust/crates/omni-tui/examples/demo.rs`

## Key Changes

1. Added targeted `#[must_use]` annotations on getters/constructors where the
   return value is semantically meaningful.
2. Replaced closure patterns like `map(|s| s.to_string())` with direct method
   references.
3. Inlined format variables (`format!("... {var} ...")`) in core and test code.
4. Normalized doc comments for markdown lint (`TuiEvent`, `JoinHandle`,
   `AppState`, `ReceivedEvent` etc.).
5. Added missing `# Errors` sections on touched `Result`-returning APIs in
   `event` and `socket` modules.
6. Reduced needless pass-by-value warnings in event mapping and server loop
   signature (`map_crossterm_event` now takes `&CrosstermEvent`,
   `SocketServer::run_loop` now takes references).

## Verification Evidence

### Formatting

```bash
cargo fmt
```

Result: pass.

### Strict clippy (`omni-tui`)

```bash
cargo clippy -p omni-tui --all-targets -- -W clippy::too_many_lines
```

Result: pass (exit code `0`).

Observed warning baseline during this wave:

- before this wave slice: `omni-tui (lib) generated 87 warnings`
- after this wave slice: `omni-tui (lib) generated 58 warnings`

### Targeted nextest smoke

```bash
cargo nextest run -p omni-tui test_socket_server_start_stop
cargo nextest run -p omni-tui test_state_operations
```

Result: both pass.

## Outcome

This wave reduced warning noise while preserving strict policy:
no blanket `allow`, no behavior-changing refactor in runtime logic, and
smoke validation for touched test paths.
