# Modern Rust And Software Standards (Target State)

This file defines the target standards for `omni-dev-fusion`.
Use this as a long-term contract for implementation and review.

## 1. Rust Workspace Standards

1. Every crate MUST include:
   - `[lints]`
   - `workspace = true`
2. CI MUST run:
   - `cargo check --workspace --all-targets`
   - `cargo clippy --workspace --all-targets --all-features -- -D warnings`
   - `cargo nextest run --workspace --all-features`
3. Library crates that are part of protocol/runtime paths SHOULD deny:
   - `clippy::print_stdout`
   - `clippy::print_stderr`

## 2. Error-Handling Standards

1. External boundaries (CLI, HTTP, MCP handlers) may use `anyhow` ergonomically.
2. Core libraries should expose typed error enums for stable behavior contracts.
3. Error variants must carry actionable context for observability.

## 3. Module And File Standards

1. Split by concern, not by line count.
2. No `include!` to compose large runtime modules in production crates unless there is strong compile-time rationale.
3. Large modules should be split into domain submodules and re-exported via `mod.rs`.
4. Hotspot threshold for review:
   - warning at >500 lines,
   - mandatory split review at >900 lines for production paths.

## 4. Test Standards

1. Use `nextest` as primary Rust test runner in CI.
2. Maintain focused integration harnesses for protocol clients (MCP, gateway, subprocess cleanup).
3. Snapshot tests are valid for contract stability; they must include intentional update workflow.
4. Every bugfix touching runtime behavior requires at least one regression test.

## 5. Dependency Security Standards

1. Add and maintain `deny.toml`.
2. CI MUST include `cargo-deny` and `cargo-audit`.
3. Every ignored advisory MUST include:
   - reason,
   - affected path,
   - removal trigger.

## 6. CI And Release Standards

1. Separate lanes:
   - format/lint,
   - build,
   - tests,
   - security,
   - release verification.
2. Use changed-path detection to avoid unnecessary heavy jobs.
3. Release lane should verify:
   - tag/version alignment,
   - platform matrix health,
   - artifact integrity.

## 7. Rust/Python Boundary Standards

1. Rust should own performance-critical operations and stable contracts.
2. Python should remain orchestration-friendly but must not duplicate core protocol ownership.
3. Cross-language payloads should have schema-driven contracts (JSON or Arrow IPC where relevant).
4. Boundary decisions must be documented per feature:
   - owner,
   - interface,
   - failure model,
   - test coverage.

## 8. Documentation And Governance Standards

1. Every crate requires README with:
   - purpose,
   - public API,
   - test command.
2. Keep repository-level rules in `AGENTS.md`.
3. Add subsystem-level guidance where ambiguity is high (MCP, vector, bridge, release).
