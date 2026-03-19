# 修仙道场 (Xiuxian Daochang) Match-Wildcard-for-Single-Variants Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::match_wildcard_for_single_variants` file-level allow markers and
revalidating strict clippy gates.

## Implemented Changes

1. Removed `clippy::match_wildcard_for_single_variants` file-level allow
   markers across `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Revalidated strict test + crate lanes after marker removal:
   - no source warnings were surfaced for this category.
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::match_wildcard_for_single_variants" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::match_wildcard_for_single_variants` marker count in
  `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on
`clippy::match_wildcard_for_single_variants` with strict pedantic and
`too_many_lines` validation still green.
