# 修仙道场 (Xiuxian Daochang) Field-Reassign-With-Default Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::field_reassign_with_default` file-level allow markers and fixing
surfaced `AgentConfig::default()` reassignment patterns.

## Implemented Changes

1. Removed `clippy::field_reassign_with_default` file-level allow markers
   across `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Refactored `AgentConfig` construction from
   `let mut config = AgentConfig::default(); config.field = ...;`
   to struct initialization with `..AgentConfig::default()` in:
   - `packages/rust/crates/xiuxian-daochang/tests/agent/memory_recall_feedback_state.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent/memory_recall_state.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent_session_context.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent/system_prompt_injection_state.rs`
3. Resolved cross-crate pedantic spillover that surfaced during xiuxian-daochang
   validation:
   - `packages/rust/crates/xiuxian-qianhuan/src/zhenfa_router/native.rs`
   - `packages/rust/crates/xiuxian-wendao/src/zhenfa_router/native.rs`
   - changed `fn id(&self) -> &str` to `fn id(&self) -> &'static str` to fix
     `clippy::unnecessary_literal_bound`.
4. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::field_reassign_with_default" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang -p xiuxian-qianhuan -p xiuxian-wendao
cargo clippy -p xiuxian-qianhuan -- -W clippy::pedantic
cargo clippy -p xiuxian-qianhuan -- -W clippy::too_many_lines
cargo clippy -p xiuxian-wendao -- -W clippy::pedantic
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::field_reassign_with_default` marker count in `xiuxian-daochang/tests`: `0`.
- `xiuxian-qianhuan` pedantic + `too_many_lines`: pass (`0`).
- `xiuxian-wendao` pedantic + `too_many_lines`: pass (`0`).
- `xiuxian-daochang` pedantic (`--tests`) + `too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::field_reassign_with_default` with
cross-crate pedantic spillover resolved and strict validation lanes clean.
