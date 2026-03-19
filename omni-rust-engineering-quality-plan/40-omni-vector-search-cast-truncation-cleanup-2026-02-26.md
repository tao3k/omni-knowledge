# Omni Vector Search Cast-Truncation Cleanup (2026-02-26)

## Objective

Remove remaining `cast_possible_truncation` suppression in `xiuxian-vector`
search implementation by introducing an explicit, reusable numeric conversion
path from `f64` to `f32`.

## Scope

### Changed files

- `packages/rust/crates/xiuxian-vector/Cargo.toml`
- `packages/rust/crates/xiuxian-vector/src/search/search_impl/mod.rs`
- `packages/rust/crates/xiuxian-vector/src/search/search_impl/hybrid_ops.rs`
- `packages/rust/crates/xiuxian-vector/src/search/search_impl/rows.rs`

### What changed

1. Added direct dependency `num-traits = "0.2"` in `xiuxian-vector`.
2. Introduced `f64_to_f32_saturating` in `search_impl/mod.rs`:
   - returns `0.0` for non-finite values,
   - uses `ToPrimitive::to_f32()` for conversion without `as` cast,
   - saturates to `f32::MIN`/`f32::MAX` on out-of-range finite values.
3. Replaced `1.0 - r.distance as f32` in hybrid search fusion with
   `f64_to_f32_saturating(1.0 - r.distance)`.
4. Replaced FTS `Float64Array` score conversion `arr.value(index) as f32` with
   `f64_to_f32_saturating(arr.value(index))`.
5. Removed both `#[allow(clippy::cast_possible_truncation)]` suppressions from:
   - `search/search_impl/hybrid_ops.rs`
   - `search/search_impl/rows.rs`

## Verification Evidence

- `cargo fmt -p xiuxian-vector`
- `cargo clippy -p xiuxian-vector --all-targets -- -W clippy::pedantic`
- `cargo test -p xiuxian-vector --tests`

Result: all passed.

## Outcome

`xiuxian-vector` search path no longer relies on cast-truncation suppression in
these two production hot paths, and the conversion behavior is now explicit and
reusable for future search-layer code.
