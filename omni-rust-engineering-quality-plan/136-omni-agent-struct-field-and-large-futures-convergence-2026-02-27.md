# 修仙道场 (Xiuxian Daochang) Struct-Field and Large-Futures Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang` quality convergence after the `struct_field_names` cleanup
wave, remove follow-up pedantic warnings without adding new suppressions, and
record reproducible evidence.

## Implemented Changes

1. Confirmed `clippy::struct_field_names` marker convergence in
   `packages/rust/crates/xiuxian-daochang/tests`:
   - marker count remains zero (`rg` verification).
2. Fixed follow-up pedantic warning in gateway HTTP tests:
   - replaced manual match extraction with `let ... else` in
     `packages/rust/crates/xiuxian-daochang/tests/gateway/http/runtime.rs`.
3. Fixed cross-crate pedantic warning in Wendao native tool setup:
   - made `Arc<LinkGraphIndex>` input truly consumed in
     `packages/rust/crates/xiuxian-wendao/src/zhenfa_router/native.rs` to
     resolve `clippy::needless_pass_by_value`.
4. Removed all remaining `xiuxian-daochang` `large_futures` pedantic warnings in the
   CLI/runtime entry flow by explicitly boxing large futures:
   - `packages/rust/crates/xiuxian-daochang/src/nodes/repl.rs`
   - `packages/rust/crates/xiuxian-daochang/src/nodes/stdio.rs`
   - `packages/rust/crates/xiuxian-daochang/src/main.rs`
5. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::struct_field_names" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs'
cargo fmt -p xiuxian-wendao -p xiuxian-daochang
cargo clippy -p xiuxian-wendao -- -W clippy::pedantic
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::struct_field_names` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-wendao -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines`: pass (`0`).
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang` is now clean on the targeted pedantic follow-up items from this
wave (`manual_let_else`, `large_futures`) while keeping test-marker convergence
(`struct_field_names = 0`) and cross-crate wendao compatibility intact.
