# 修仙道场 (Xiuxian Daochang) Single-Match-Else Marker Zero and Zhenfa Follow-Up (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::single_match_else` file-level allow markers, then revalidating strict
clippy gates and fixing any newly surfaced real warnings.

## Implemented Changes

1. Removed `clippy::single_match_else` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Follow-up fixes surfaced by strict revalidation:
   - `packages/rust/crates/xiuxian-daochang/src/agent/mcp.rs`
     - collapsed nested `if` (`clippy::collapsible_if`).
   - `packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/bridge.rs`
     - removed unnecessary `Result` wrapper from
       `resolve_zhenfa_base_url` (`clippy::unnecessary_wraps`),
       and adjusted caller path accordingly.
   - `packages/rust/crates/xiuxian-daochang/tests/zhenfa_tool_bridge.rs`
     - added crate-level docs comment to satisfy `missing_docs`.
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::single_match_else" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo check -p xiuxian-daochang --tests
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::single_match_else` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo check -p xiuxian-daochang --tests`: pass.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` kept strict-lint convergence after
`clippy::single_match_else` marker removal, with source follow-up fixes in the
agent/zhenfa paths and all quality gates green.
