# Tasks.nix Thin Wrapper and Rust Env Segmentation (2026-02-24)

## Objective

Keep `nix/modules/tasks.nix` focused on dependency sandboxing and task entrypoints,
while moving execution logic to `just` and `scripts`.

## Scope

- `nix/modules/tasks.nix` only.
- Verification through both `just` and `devenv tasks run` paths.

## Implemented Changes

1. Rust task environments were split by responsibility:
   - `rustBaseEnv`: shared compile/runtime baseline.
   - `rustQualityEnv`: `rustBaseEnv` + `cargo-nextest` + security tools.
   - `rustSecurityEnv`: `rustBaseEnv` + security tools.

2. Rust task constructors were normalized:
   - Added `mkRustTaskWith` to bind a task to a specific env set.
   - Kept command body as `just <recipe>` wrappers (no cargo logic added to
     `tasks.nix`).

3. Task-to-env mapping was tightened:
   - `ci:rust-quality-gate` now uses `mkRustQualityTask`.
   - `ci:rust-security-gate` now uses `mkRustSecurityTask`.
   - Other Rust tasks keep `mkRustTask` (`rustBaseEnv`) for lighter installs.

## Validation Evidence

Executed from repository root.

1. `just rust-xiuxian-daochang-dependency-assertions`
   - Result: pass.

2. `just rust-xiuxian-mcp`
   - Result: pass.

3. `just rust-xiuxian-daochang-profiles`
   - Result: pass.

4. `devenv tasks run ci:rust-xiuxian-daochang-profiles`
   - Result: pass.

5. `devenv tasks run ci:rust-xiuxian-daochang-dependency-assertions`
   - Result: pass.

6. `devenv tasks run ci:rust-security-gate`
   - Result: fail due pre-existing `cargo-audit` denied warnings
     (`js-sys 0.3.88` yanked via transitive chain), not caused by this refactor.

## Why This Matters

- Preserves the intended boundary:
  `tasks.nix` = env isolation + dispatch, `just/scripts` = executable logic.
- Improves CI lane efficiency by avoiding unnecessary tool downloads for
  lightweight Rust gates.
- Keeps local reproduction consistent with CI through the same `just` entrypoints.

## Next Slice

1. Continue dependency-security remediation from existing plan items
   (`17` / `18` / `19`), starting with the `js-sys` transitive path owner map.
2. Optionally add a dedicated `ci:rust-xiuxian-mcp` wrapper if package-level MCP
   checks are promoted to first-class CI gates.
