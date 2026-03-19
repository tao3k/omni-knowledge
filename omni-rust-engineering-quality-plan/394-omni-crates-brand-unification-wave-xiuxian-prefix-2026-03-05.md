# 394) Omni Crates Brand Unification Wave (xiuxian Prefix)

Date: 2026-03-05
Scope: Rust workspace crate branding convergence (`omni-*` -> `xiuxian-*`).

## Goal

Complete package-level brand unification for legacy `omni-*` crates under `packages/rust/crates`, while keeping the workspace buildable and testable.

## Renamed Crates

- `omni-ast` -> `xiuxian-ast`
- `omni-edit` -> `xiuxian-edit`
- `omni-executor` -> `xiuxian-executor`
- `omni-io` -> `xiuxian-io`
- `omni-lance` -> `xiuxian-lance`
- `omni-sandbox` -> `xiuxian-sandbox`
- `omni-security` -> `xiuxian-security`
- `omni-tags` -> `xiuxian-tags`
- `omni-tokenizer` -> `xiuxian-tokenizer`
- `xiuxian-vector` -> `xiuxian-vector`
- `omni-window` -> `xiuxian-window`
- `omni-memory` -> `xiuxian-memory-engine` (collision-safe with existing `xiuxian-memory`)

Additional directory cleanup:
- `packages/rust/crates/omni-mcp-client` -> `packages/rust/crates/xiuxian-mcp-client`

## Implementation Notes

- Updated physical crate directories and workspace member paths in root `Cargo.toml`.
- Updated crate package names in each `Cargo.toml`.
- Updated inter-crate dependencies (`Cargo.toml`) and import identifiers (`omni_*` -> `xiuxian_*`) in Rust sources/tests.
- Updated script/CI/doc references through deterministic token migration for renamed crates.

## Validation Evidence

### TIER-2

- `cargo check` (workspace): PASS

### TIER-3 (mandatory touched-crate clippy)

- `cargo clippy -p xiuxian-ast -p xiuxian-edit -p xiuxian-executor -p xiuxian-io -p xiuxian-lance -p xiuxian-memory-engine -p xiuxian-sandbox -p xiuxian-security -p xiuxian-tags -p xiuxian-tokenizer -p xiuxian-vector -p xiuxian-window -p xiuxian-daochang -p xiuxian-skills -p xiuxian-qianhuan -p xiuxian-qianji -p xiuxian-wendao -p omni-core-rs -- -W clippy::too_many_lines`: PASS

### Nextest spot checks

- `cargo nextest run -p xiuxian-window`: PASS (3/3)
- `cargo nextest run -p xiuxian-memory-engine`: PASS (69/69)
- `cargo nextest run -p xiuxian-vector --test test_store`: PASS (6/6)
- `cargo nextest run -p xiuxian-wendao --test test_wendao_cli`: PASS (34/34)
- `cargo nextest run -p xiuxian-daochang --test gateway_validation`: PASS (3/3)

Note:
- A concurrent run of `xiuxian-daochang` + `xiuxian-wendao` nextest once failed during linker stage (SIGTERM from system toolchain process kill). Re-run in isolation passed.

## Final Audit Snapshot

- `packages/rust/crates` contains no `omni-*` directories.
- No code/config references remain for the renamed crate set (`omni-{ast,edit,executor,io,lance,memory,sandbox,security,tags,tokenizer,vector,window}` and underscore forms).

## Follow-up Candidate

- `packages/rust/bindings/python` package name remains `omni-core-rs`.
  - This was intentionally left as-is in this wave because it impacts Python package identity and import contracts.
  - If full-stack branding convergence is required, handle as a dedicated migration with Python API compatibility plan.
