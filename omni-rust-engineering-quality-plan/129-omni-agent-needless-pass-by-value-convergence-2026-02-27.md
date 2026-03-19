# 修仙道场 (Xiuxian Daochang) Needless-Pass-By-Value Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::needless_pass_by_value` file-level allow markers and fixing surfaced
ownership-signature warnings.

## Implemented Changes

1. Removed `clippy::needless_pass_by_value` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Fixed surfaced warning in discord runtime test support:
   - `packages/rust/crates/xiuxian-daochang/tests/discord_runtime/support.rs`
     - changed `start_job_manager(agent: Arc<Agent>)` to
       `start_job_manager(agent: &Arc<Agent>)`.
   - updated call sites to pass by reference:
     - `packages/rust/crates/xiuxian-daochang/tests/discord_runtime/authorization.rs`
     - `packages/rust/crates/xiuxian-daochang/tests/discord_runtime/logging.rs`
     - `packages/rust/crates/xiuxian-daochang/tests/discord_runtime/managed_commands.rs`
     - `packages/rust/crates/xiuxian-daochang/tests/discord_runtime/session_preemption.rs`
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::needless_pass_by_value" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::needless_pass_by_value` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::needless_pass_by_value` with strict
pedantic and `too_many_lines` lanes remaining clean.
