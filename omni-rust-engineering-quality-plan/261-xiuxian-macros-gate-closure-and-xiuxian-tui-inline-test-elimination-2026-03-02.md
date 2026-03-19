# 261. xiuxian-macros Gate Closure and xiuxian-tui Inline Test Elimination (2026-03-02)

## Scope

- Crates:
  - `packages/rust/crates/xiuxian-macros`
  - `packages/rust/crates/xiuxian-daochang` (dependency-chain revalidation)
  - `packages/rust/crates/xiuxian-tui`
- Objective:
  - close the remaining `clippy::too_many_lines` convergence gate in
    `xiuxian-macros`,
  - verify the transitive `xiuxian-daochang --tests` clippy lane is clean after the
    macro refactor,
  - remove the last inline test module from `xiuxian-tui` `src/examples`
    surfaces and keep equivalent package-top test coverage.

## Changes

### 1) `xiuxian-macros` split validated as warning-clean

Updated previously and revalidated in this wave:

- `packages/rust/crates/xiuxian-macros/src/xiuxian_config.rs`

Effective state now:

- `expand(...)` reduced to orchestration only.
- helper generators extracted and isolated:
  - `generate_spec_helpers`
  - `generate_api_key_policy_helpers`
  - `generate_loading_helpers`
  - `generate_xiuxian_config_impl_tokens`
- argument resolution extracted into:
  - `resolve_array_merge_strategy`
  - `resolve_xiuxian_config_args`

Outcome:

- `clippy::too_many_lines` no longer triggers in the crate.

### 2) Transitive revalidation for `xiuxian-daochang --tests`

No new code change required in `xiuxian-daochang` for this checkpoint; this step
proves dependency-chain cleanliness after the macro split.

Outcome:

- strict clippy lane for `xiuxian-daochang --tests` is warning-clean.

### 3) `xiuxian-tui` example inline test removed and moved to package-top tests

Updated:

- `packages/rust/crates/xiuxian-tui/examples/demo.rs`
- `packages/rust/crates/xiuxian-tui/src/cli_args.rs`
- `packages/rust/crates/xiuxian-tui/src/main.rs`
- `packages/rust/crates/xiuxian-tui/src/lib.rs`
- `packages/rust/crates/xiuxian-tui/src/demo_cli_args.rs` (new)
- `packages/rust/crates/xiuxian-tui/tests/demo_cli_args_unit.rs` (new)
- `packages/rust/crates/xiuxian-tui/tests/demo_cli_args_module/tests.rs` (new)
- `packages/rust/crates/xiuxian-tui/tests/main_demo_unit.rs`

Actions:

- extracted demo example arg model into reusable crate module
  `demo_cli_args::DemoArgs`.
- switched demo example to consume `DemoArgs` from `src`.
- removed `#[cfg(test)] mod tests` from `examples/demo.rs`.
- added package-top integration harness for demo argument parsing.
- made main binary args reusable as `cli_args::CliArgs` and migrated
  `main_demo_unit` to consume that public module.
- removed `#[path = "../src/cli_args.rs"]` from `main_demo_unit`.
- structural proof:
  - no `#[cfg(test)]` / `mod tests` remains under
    `packages/rust/crates/xiuxian-tui/src` and
    `packages/rust/crates/xiuxian-tui/examples`.
  - no `#[path = "../src/..."]` remains under
    `packages/rust/crates/xiuxian-tui/tests`.

## Validation Evidence

### 1) Mandatory clippy gates

```bash
RUSTC_WRAPPER= cargo clippy -p xiuxian-macros -- -W clippy::too_many_lines
RUSTC_WRAPPER= cargo clippy -p xiuxian-daochang --tests -- -W clippy::too_many_lines
RUSTC_WRAPPER= cargo clippy -p xiuxian-tui -- -W clippy::too_many_lines
```

Result:

- all commands succeeded (exit 0), no warnings.

### 2) Targeted nextest

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-tui --test demo_cli_args_unit --test main_demo_unit
```

Result:

- `4 passed`, `0 failed`, `0 skipped`.

### 3) Structural proof command

```bash
rg -n "#\\[cfg\\(test\\)\\]|mod tests\\b" \
  packages/rust/crates/xiuxian-tui/examples \
  packages/rust/crates/xiuxian-tui/src --glob '*.rs'

rg -n "#\\[path\\s*=\\s*\\\"\\.\\./src/" \
  packages/rust/crates/xiuxian-tui/tests --glob '*.rs'
```

Result:

- no matches.

## Outcome

- macro warning convergence is closed at crate level and dependency-chain level,
- `xiuxian-tui` test-structure policy is tightened by removing inline example
  tests while preserving coverage through package-top harnesses,
- touched crates remain lint-green and test-green under required gates.
