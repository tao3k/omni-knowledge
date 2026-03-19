# Dependency Security Lane Bootstrap (2026-02-24)

## Objective

Bootstrap a Codex-aligned dependency security lane in `omni-dev-fusion` with
real execution signals, then convert that signal into a staged remediation plan.

## Implemented Changes

1. Local Rust gate integration (`justfile`)
   - Added `rust-security-audit` (`cargo audit --deny warnings`).
   - Added `rust-security-deny` (`cargo deny check advisories bans sources`).
   - Added `rust-security-gate` and wired it into `rust-quality-gate`.
   - Switched `cargo audit` invocation to centralized script:
     `scripts/rust/cargo_audit_gate.sh`.

2. Nix Rust toolchain/runtime support (`nix/modules/rust.nix`, `nix/modules/tasks.nix`)
   - Added `pkgs.cargo-audit` and `pkgs.cargo-deny` to Rust package inputs.
   - Added CI task: `ci:rust-security-gate`.

3. CI wiring (`.github/workflows/ci.yaml`, `.github/workflows/checks.yaml`)
   - Added a dedicated `ci:rust-security-gate` step in both workflows.
   - CI rust security steps are now mandatory (no `continue-on-error` on the
     rust security gate steps).

4. Policy baseline (`deny.toml`)
   - Added repository-level `cargo-deny` baseline config.
   - Added temporary transitive advisory ignore entries with removal notes.

5. P0 dependency updates (`Cargo.lock`)
   - `bytes` `1.11.0 -> 1.11.1` (fix for `RUSTSEC-2026-0007`).
   - `time` `0.3.45 -> 0.3.47` (fix for `RUSTSEC-2026-0009`).
   - `git2` `0.20.3 -> 0.20.4` (removes `RUSTSEC-2026-0008` signal).
   - `oneshot` `0.1.11 -> 0.1.13` (removes `RUSTSEC-2026-0005` signal).

6. Exception governance
   - Added dedicated exception register:
     `18-dependency-security-exception-register-2026-02-24.md`.
   - Added chain-specific elimination plans:
     - `19-reqwest011-transitive-decommission-plan-2026-02-24.md`
     - `20-lru0125-transitive-elimination-plan-2026-02-24.md`

## Verification Evidence

1. Syntax/registration checks
   - `nix-instantiate --parse nix/modules/tasks.nix` -> pass
   - `nix-instantiate --parse nix/modules/rust.nix` -> pass
   - `devenv tasks list` includes:
     - `ci:rust-quality-gate`
     - `ci:rust-security-gate`

2. Real gate execution (baseline)
   - `devenv tasks run ci:rust-security-gate` -> `EXIT:1`
   - Baseline failure:
     - `cargo audit`: `3 vulnerabilities found`, `6 denied warnings found`.

3. `cargo-deny` baseline execution
   - `devenv shell -- cargo deny check advisories bans sources` -> `EXIT:1`
   - Result summary:
     - `advisories FAILED`
     - `bans ok`
     - `sources ok`

4. Real gate execution (after P0 updates + temporary exceptions)
   - `devenv tasks run ci:rust-security-gate` -> `EXIT:0`
   - `devenv shell -- just rust-security-gate` -> `EXIT:0`
   - Result summary:
     - `cargo audit`: pass with explicit temporary ignore list.
     - `cargo deny check advisories bans sources`: `advisories ok, bans ok, sources ok`.

## Current Temporary Exception Set

The enforced lane is currently green with temporary exceptions for unresolved
transitive advisories:

1. `RUSTSEC-2023-0071` (`rsa` 0.9.10)
   - Root path: `sqlx-mysql -> sea-orm -> litellm-rs -> xiuxian-daochang`
   - Removal condition: upstream stack migrates to a fixed/non-affected RSA path.
2. `RUSTSEC-2025-0141` (`bincode` 1.3.3, unmaintained)
   - Root path: `litellm-rs -> xiuxian-daochang`
   - Removal condition: upstream removes `bincode` 1.x dependency.
3. `RUSTSEC-2024-0436` (`paste` 1.0.15, unmaintained)
   - Root path: `lance/datafusion -> xiuxian-vector`
   - Removal condition: upstream graph no longer resolves `paste` 1.0.x.
4. `RUSTSEC-2025-0134` (`rustls-pemfile` 1.0.4, unmaintained)
   - Root path: `reqwest 0.11` via `litellm-rs` / `serenity` in `xiuxian-daochang`
   - Removal condition: migration to maintained PEM parser stack.
5. `RUSTSEC-2026-0002` (`lru` 0.12.5, unsound)
   - Root path: `tantivy 0.24/0.25` via `lance` / `xiuxian-vector`
   - Removal condition: `tantivy/lance` chain resolves to non-affected `lru`.

## Staged Remediation Plan

1. P0 patch upgrades (completed in this pass)
   - `bytes` and `time` advisories removed from active gate output.
   - `git2` and `oneshot` advisories removed from active gate output.

2. P1 transitive dependency pressure
   - Track upstream update paths for `rsa`, `bincode`, `paste`, and
     `rustls-pemfile` transitive chains.
   - Prefer upstream version upgrades over permanent exceptions.
   - Current revalidation signal: `litellm-rs`, `serenity`, `lance`, and
     `tantivy` are already at current published versions in this workspace, so
     immediate dependency bumps do not remove the remaining transitive advisories.
   - Active execution artifact:
     `19-reqwest011-transitive-decommission-plan-2026-02-24.md`.

3. P1/P2 unsound package elimination
   - Continue with `lru` (`RUSTSEC-2026-0002`) chain elimination.
   - Active execution artifact:
     `20-lru0125-transitive-elimination-plan-2026-02-24.md`.

4. CI graduation
   - Current gate is green with explicit temporary exceptions.
   - Rust security gate has been promoted to mandatory mode in workflows.

## Exit Criteria For This Slice

1. `ci:rust-security-gate` remains green in both local and CI environments.
2. Temporary exceptions decrease monotonically release-by-release.
3. Any temporary warnings/ignores include clear removal conditions.
4. Workflow steps are converted from advisory to mandatory mode.
   - Status: completed for rust dependency security steps.
