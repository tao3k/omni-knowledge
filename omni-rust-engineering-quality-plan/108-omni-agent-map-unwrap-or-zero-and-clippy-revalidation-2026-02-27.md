# 修仙道场 (Xiuxian Daochang) Map-Unwrap-Or Zero and Clippy Revalidation (2026-02-27)

## Scope

Continue the second-pass Rust quality convergence by removing
`clippy::map_unwrap_or` suppression debt in `xiuxian-daochang/tests`, fixing newly
surfaced real warnings, and revalidating the crate with strict clippy gates.

## Implemented Changes

1. Removed `clippy::map_unwrap_or` file-level allow entries across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
   - Baseline count before cleanup: `125`.
   - Count after cleanup: `0`.
2. Fixed real `map_unwrap_or` call sites exposed by strict revalidation:
   - `packages/rust/crates/xiuxian-daochang/tests/discord_runtime/support.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/mcp_discover_cache.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/embedding_client_cache.rs`
3. Fixed one production warning surfaced during `--tests` pedantic run:
   - `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/session_admin_persistence.rs`
   - replaced redundant closure with `Table::is_empty`.
4. Fixed one compile blocker surfaced during revalidation:
   - `packages/rust/crates/xiuxian-daochang/src/config/settings/mod.rs`
   - re-exported `MistralSettings` so
     `config/settings/loader.rs` imports resolve correctly.

## Verification Evidence

Executed:

```bash
rg -n "clippy::map_unwrap_or" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
rg -nUP "\\.map\\([^)]*\\)\\s*\\.unwrap_or(?:_else)?\\(" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --test agent_injection -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang --test agent_suite -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::map_unwrap_or` allow count in `xiuxian-daochang/tests`: `0`.
- `map(...).unwrap_or(...)` pattern count in `xiuxian-daochang/tests`: `0`.
- `xiuxian-daochang` test targets pass pedantic clippy checks.
- `xiuxian-daochang` crate passes `too_many_lines` policy check.

## Outcome

`xiuxian-daochang` moved from broad suppression-based handling of
`map_unwrap_or` to source-level convergence with zero suppression markers in
tests and a clean strict-clippy revalidation loop for this wave.
