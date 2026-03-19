# Xiuxian-Wendao Map-Unwrap-Or Convergence and Enhancer Test Repair (2026-02-26)

## Scope

Continue post-marker cleanup by removing `clippy::map_unwrap_or` suppression
debt in `xiuxian-wendao` tests and repairing a surfaced test import mismatch
that blocked `--all-targets` verification.

## Implemented Changes

1. Removed `clippy::map_unwrap_or` file-level allows from
   `xiuxian-wendao/tests` and replaced concrete call sites with
   `Option::is_some_and(...)` patterns.
2. Fixed newly surfaced warnings/errors:
   - `tests/test_kg_cache.rs`: converted `VALKEY_URL` probe to
     `.ok().is_some_and(...)`.
   - `tests/test_graph/mod.rs`: same `VALKEY_URL` probe cleanup.
   - `src/enhancer/markdown_config.rs`: replaced redundant closure
     `.map(|token| token.to_lowercase())` with `.map(str::to_lowercase)`.
3. Repaired test import path after stricter target coverage:
   - `tests/test_enhancer.rs`
   - switched `infer_relations` import from crate root to
     `xiuxian_wendao::enhancer::infer_relations`.
   - added crate-level test doc comment to satisfy missing-doc warning.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-wendao
cargo clippy -p xiuxian-wendao --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-wendao --all-targets -- -W clippy::pedantic
cargo clippy -p xiuxian-wendao -- -W clippy::too_many_lines
rg -n "clippy::map_unwrap_or" packages/rust/crates/xiuxian-wendao/tests --glob '*.rs' | wc -l
rg -nUP "\\.map\\((?s:.*?)\\)\\.unwrap_or\\(" packages/rust/crates/xiuxian-wendao/tests --glob '*.rs' | wc -l
```

Result:

- `xiuxian-wendao` tests and all targets pass pedantic checks.
- `xiuxian-wendao` passes `too_many_lines` policy verification.
- `clippy::map_unwrap_or` allow count in `xiuxian-wendao/tests`: `0`.
- `map(...).unwrap_or(...)` occurrences in `xiuxian-wendao/tests`: `0`.

## Outcome

`xiuxian-wendao` progressed from marker-zero baseline to additional
suppression-debt reduction in `map_unwrap_or`, while preserving full clippy
verification coverage.
