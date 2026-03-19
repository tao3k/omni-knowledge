# Cross-Crate Baseline and Wendao Test Pedantic Convergence (2026-02-27)

## Scope

Second-pass convergence beyond `xiuxian-daochang`, focused on:

1. Cross-crate strict-clippy baseline validation for `xiuxian-qianji` and
   `xiuxian-zhixing`.
2. `xiuxian-wendao` pedantic cleanup without suppression.
3. Targeted test revalidation with `cargo nextest`.

## Implemented Changes

1. Fixed pedantic warnings in:
   - `packages/rust/crates/xiuxian-wendao/tests/test_wendao_resource_registry.rs`
2. Replaced `match` patterns flagged by `clippy::manual_let_else` with explicit
   `let ... else` bindings.
3. Replaced wildcard-on-single-variant error matching with explicit variant
   destructuring (`WendaoResourceRegistryError::MissingLinkedResources`).
4. Kept all fixes suppression-free (no new `allow` attributes).

## Verification Evidence

Executed:

```bash
cargo clippy -p xiuxian-qianji --all-targets -- \
  -W clippy::pedantic -W clippy::too_many_lines
cargo clippy -p xiuxian-zhixing --all-targets -- \
  -W clippy::pedantic -W clippy::too_many_lines
cargo fmt -p xiuxian-wendao
cargo clippy -p xiuxian-wendao --all-targets -- \
  -W clippy::pedantic -W clippy::too_many_lines
cargo nextest run -p xiuxian-wendao --test test_wendao_resource_registry
```

Results:

- `xiuxian-qianji`: strict-clippy command completed with no warnings.
- `xiuxian-zhixing`: strict-clippy command completed with no warnings.
- `xiuxian-wendao`: original pedantic warnings in
  `test_wendao_resource_registry.rs` were resolved; re-run completed cleanly.
- `cargo nextest` targeted integration test passed:
  `2 passed, 0 skipped`.

## Outcome

This wave extends warning-zero confidence from `xiuxian-daochang` into adjacent Rust
crates and removes concrete pedantic debt in `xiuxian-wendao` through
structural test-code improvements, preserving the project's no-suppression
quality policy.
