# 修仙道场 (Xiuxian Daochang) Redundant-Closure Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang` test-lane suppression-debt reduction by removing
`clippy::redundant_closure_for_method_calls` file-level allows and fixing all
newly surfaced closure warnings with method-reference rewrites.

## Implemented Changes

1. Removed `clippy::redundant_closure_for_method_calls` file-level allows
   across `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
   - Baseline count before cleanup: `128`.
   - Count after cleanup: `0`.
2. Fixed all surfaced warnings from strict `--tests` pedantic revalidation
   (10 warnings across 8 test files), including:
   - `map_or(..., |items| items.len())` -> `map_or(..., Vec::len)`
   - `.map(|a| a.as_slice())` -> `.map(Vec::as_slice)`
   - `.and_then(|value| value.as_u64())` -> `.and_then(serde_json::Value::as_u64)`
   - `.iter().map(|value| value.to_string())` ->
     `.iter().map(std::string::ToString::to_string)`
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` added,
   - all warnings fixed at source.

## Verification Evidence

Executed:

```bash
rg -n "clippy::redundant_closure_for_method_calls" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:

- `clippy::redundant_closure_for_method_calls` allow count in
  `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic` passes.
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines` passes.

## Outcome

`xiuxian-daochang/tests` no longer relies on file-level suppression for
`redundant_closure_for_method_calls`, and strict clippy verification remains
green after source-level remediation.
