# 229. Test-Structure Wrapper Zero and `xiuxian-mcp` Dead-Code Allow Removal (2026-03-01)

## Scope

- Continue test-structure normalization in touched Rust crates.
- Remove suppression-based dead-code handling in `xiuxian-mcp` integration test.
- Ensure changed crates pass strict clippy and targeted nextest.

## Changes

1. Removed dead-code suppressions in `xiuxian-mcp` integration test
- File: `packages/rust/crates/xiuxian-mcp/tests/streamable_http_integration.rs`
- Removed:
  - `#[allow(dead_code)]` on `REAL_PORT`
  - `#[allow(dead_code)]` on `port_open`
- Structural replacement:
  - Reused `port_open(REAL_PORT)` in the real-server ignored test.
  - If no listener is present, test exits early with `Ok(())` and message.
  - This preserves ignore-based workflow while eliminating stale dead code.

2. Removed redundant `mod tests` wrapper under `tests/`
- File: `packages/rust/crates/omni-ast/tests/unit/python_tree_sitter_tests.rs`
- Removed inline `#[cfg(test)] mod tests { ... }` wrapper and kept test functions
  directly in the module file.
- This aligns with the project test-layout rule: test files under `tests/`
  should not wrap all cases inside another `mod tests`.

3. Repository-level test-structure verification (current scope)
- Confirmed no remaining `mod tests` wrappers under
  `packages/rust/crates/*/tests/**/*.rs` after this wave.

## Validation Evidence

1. Strict clippy (`xiuxian-mcp`)

```bash
cargo clippy -p xiuxian-mcp --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`

2. Strict clippy (`omni-ast`)

```bash
cargo clippy -p omni-ast --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0`

3. Targeted nextest (`xiuxian-mcp` integration lane)

```bash
cargo nextest run -p xiuxian-mcp --test streamable_http_integration
```

- Exit code: `0`
- Result: `1 passed`, `1 skipped`

4. Targeted nextest (`omni-ast` lib/unit lane)

```bash
cargo nextest run -p omni-ast --lib
```

- Exit code: `0`
- Result: `48 passed`, `0 failed`

5. Wrapper scan (`tests` tree)

```bash
rg -n "#\\[cfg\\(test\\)\\]\\s*mod tests|mod tests\\s*\\{" packages/rust/crates/*/tests --glob '*.rs'
```

- Exit code: `1` (no matches)

## Outcome

- `xiuxian-mcp` no longer uses stale dead-code suppression in the touched
  integration lane.
- `tests` directory wrapper style (`mod tests` envelope) is zero for the current
  workspace crates in this scan scope.
- Strict clippy and targeted nextest are green for all touched workspace crates.
