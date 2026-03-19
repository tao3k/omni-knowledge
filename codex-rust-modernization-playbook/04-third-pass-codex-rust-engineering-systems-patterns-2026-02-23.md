# Third-Pass Codex Rust Engineering Systems Patterns (2026-02-23)

This file is codex-only.
It records third-pass findings from `.cache/researcher/openai/codex/codex-rs`.

## 1. Workspace Architecture And Build Policy

1. Large multi-crate workspace with explicit membership and resolver policy.
   - Evidence: `.cache/researcher/openai/codex/codex-rs/Cargo.toml:2`
2. Workspace dependency table centralizes internal crates for consistent crate
   naming and shared dependency governance.
   - Evidence: `.cache/researcher/openai/codex/codex-rs/Cargo.toml:77`
3. Release profile is optimized for shipped binaries (`lto="fat"`,
   `strip="symbols"`, `codegen-units=1`).
   - Evidence: `.cache/researcher/openai/codex/codex-rs/Cargo.toml:347`
4. Dedicated CI test profile keeps low optimization with debuggable symbols.
   - Evidence: `.cache/researcher/openai/codex/codex-rs/Cargo.toml:356`
5. `cargo-shear` metadata controls known false positives in dependency hygiene
   tooling.
   - Evidence: `.cache/researcher/openai/codex/codex-rs/Cargo.toml:337`

## 2. Lint And Supply-Chain Governance

6. Workspace-level clippy denial policy is codified centrally.
   - Evidence: `.cache/researcher/openai/codex/codex-rs/Cargo.toml:299`
7. Domain-specific clippy policy (`clippy.toml`) adds UI constraints and tuned
   lint thresholds.
   - Evidence: `.cache/researcher/openai/codex/codex-rs/clippy.toml:1`
8. `cargo-deny` policy tracks acknowledged advisories as explicit debt with
   controlled exceptions.
   - Evidence: `.cache/researcher/openai/codex/codex-rs/deny.toml:70`
9. License and source restrictions are enforced by policy, not tribal
   convention.
   - Evidence: `.cache/researcher/openai/codex/codex-rs/deny.toml:91`
   - Evidence: `.cache/researcher/openai/codex/codex-rs/deny.toml:183`
   - Evidence: `.cache/researcher/openai/codex/codex-rs/deny.toml:260`
10. Vulnerability audit is an automated GitHub Actions lane.
    - Evidence:
      `.cache/researcher/openai/codex/codex-rs/.github/workflows/cargo-audit.yml:1`

## 3. Runtime Safety And Error Contracts

11. Process hardening is centralized and invoked pre-main (`#[ctor::ctor]`)
    by binaries.
    - Evidence:
      `.cache/researcher/openai/codex/codex-rs/process-hardening/src/lib.rs:12`
    - Evidence:
      `.cache/researcher/openai/codex/codex-rs/responses-api-proxy/src/main.rs:4`
12. Sandboxing is represented by typed execution contracts (`CommandSpec`,
    `ExecRequest`, `SandboxManager`) in core modules.
    - Evidence:
      `.cache/researcher/openai/codex/codex-rs/core/src/sandboxing/mod.rs:32`
    - Evidence:
      `.cache/researcher/openai/codex/codex-rs/core/src/sandboxing/mod.rs:89`
13. Core crate boundary policy denies direct stdout/stderr prints in library
    surfaces.
    - Evidence: `.cache/researcher/openai/codex/codex-rs/core/src/lib.rs:1`
14. Transport errors are typed and structured (`thiserror`-backed error model)
    in client crates.
    - Evidence:
      `.cache/researcher/openai/codex/codex-rs/codex-client/src/error.rs:5`
15. CLI/TUI layers use rich error reports (`color_eyre`, `WrapErr`) and panic
    tracing to preserve diagnostics.
    - Evidence: `.cache/researcher/openai/codex/codex-rs/tui/src/lib.rs:443`
    - Evidence: `.cache/researcher/openai/codex/codex-rs/tui/src/app.rs:73`

## 4. Test Engineering Reliability

16. Shared test harnesses provide deterministic IDs and hermetic fixtures
    rooted in `TempDir`.
    - Evidence:
      `.cache/researcher/openai/codex/codex-rs/core/tests/common/lib.rs:23`
    - Evidence:
      `.cache/researcher/openai/codex/codex-rs/core/tests/common/lib.rs:102`
    - Evidence:
      `.cache/researcher/openai/codex/codex-rs/core/tests/common/lib.rs:130`

## 5. Third-Pass Summary

Codex demonstrates a layered engineering system:

- workspace policy as the base layer,
- lint/security governance as mandatory controls,
- runtime hardening and typed error boundaries for safety,
- deterministic harnesses for repeatable reliability.

This file intentionally contains no project-specific adaptation roadmap.

