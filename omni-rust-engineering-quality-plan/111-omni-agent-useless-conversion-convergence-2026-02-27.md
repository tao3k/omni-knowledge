# 修仙道场 (Xiuxian Daochang) Useless-Conversion Convergence (2026-02-27)

## Scope

Continue suppression-debt reduction in `xiuxian-daochang/tests` by removing
`clippy::useless_conversion` file-level allows and fixing all surfaced source
warnings without adding new suppression attributes.

## Implemented Changes

1. Removed `clippy::useless_conversion` file-level allow entries across:
   - `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`
2. Revalidated strict test lane and fixed surfaced real warning:
   - `packages/rust/crates/xiuxian-daochang/tests/agent_injection.rs`
   - removed a useless same-type conversion in tool metadata title handling.
   - preserved required `Into<Cow<'static, str>>` conversions for fields whose
     target type is `Cow<'static, str>` (`name` and `description`).
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::useless_conversion" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::useless_conversion` allow count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic` passes.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines` passes.

## Outcome

`xiuxian-daochang/tests` converged to zero file-level suppression markers for
`clippy::useless_conversion` with strict clippy validation still green.
