# 修仙道场 (Xiuxian Daochang) Option-Deref and Raw-String Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang` test-lane suppression-debt reduction by removing:

- `clippy::option_as_ref_deref`
- `clippy::needless_raw_string_hashes`

Then fix all surfaced source warnings and revalidate strict clippy gates.

## Implemented Changes

1. Removed file-level `clippy::option_as_ref_deref` allows across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
   - Baseline before cleanup: `127`.
   - Count after cleanup: `0`.
2. Fixed surfaced `option_as_ref_deref` call sites:
   - `packages/rust/crates/xiuxian-daochang/tests/config_mcp.rs`
   - migrated `args.as_ref().map(Vec::as_slice)` to `args.as_deref()`.
3. Removed file-level `clippy::needless_raw_string_hashes` allows across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
   - Baseline before cleanup: `127`.
   - Count after cleanup: `0`.
4. Fixed surfaced raw-string warnings in concrete tests:
   - `packages/rust/crates/xiuxian-daochang/tests/discover_cache_valkey_precedence.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent_injection.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent/system_prompt_injection_state.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/config_settings.rs`
5. Follow-up cleanup in `config_settings.rs` while removing old markers:
   - dropped `clippy::expect_used` and `clippy::unwrap_used` file-level allows,
   - replaced `expect/unwrap` paths with `require_ok` / `require_some` helpers,
   - fixed an introduced move-borrow regression from by-value `PathBuf` passing,
   - fixed surfaced `collapsible_if` warning in `write_file`.

## Verification Evidence

Executed:

```bash
rg -n "clippy::option_as_ref_deref" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
rg -n "clippy::needless_raw_string_hashes" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
rg -n "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::option_as_ref_deref` allow count in `xiuxian-daochang/tests`: `0`.
- `clippy::needless_raw_string_hashes` allow count in `xiuxian-daochang/tests`: `0`.
- `clippy::expect_used|clippy::unwrap_used` allow count in
  `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic` passes.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines` passes.

## Outcome

`xiuxian-daochang/tests` now converges to zero file-level markers for:

- `option_as_ref_deref`
- `needless_raw_string_hashes`
- `expect_used`
- `unwrap_used`

with strict clippy revalidation preserved after source-level fixes.
