# 203. 修仙道场 (Xiuxian Daochang) Env-Free Bootstrap Tests and Zhenfa Reload-Args Compat Wave (2026-02-28)

## Scope

This wave continued modern Rust engineering convergence in `xiuxian-daochang`,
focused on two root-cause directions:

1. Remove test-side `unsafe_code` allowances tied to process-global env mutation.
2. Fix native zhenfa reload argument compatibility without suppression.

## Changes

1. Reworked config tests to avoid env mutation and rely on explicit base paths:
   - `packages/rust/crates/xiuxian-daochang/src/config/tests.rs`
   - `packages/rust/crates/xiuxian-daochang/src/config/xiuxian.rs`

2. Added explicit-input helpers in bootstrap modules so tests can validate
   precedence logic without `std::env::set_var/remove_var`:
   - `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/builder.rs`
   - `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/zhixing.rs`
   - `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/qianhuan.rs`
   - `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/tests.rs`

3. Hardened bootstrap template test expectations for current embedded-resource
   topology (semantic link counts may be zero in some snapshots while still
   remaining valid):
   - `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap/tests.rs`

4. Native zhenfa compatibility fix:
   - Added empty-args null-retry path for empty-object dispatch payloads in
     bridge execution.
   - `packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/bridge.rs`

5. Aligned qianhuan native reload argument type with orchestrator object-arg
   contract:
   - `packages/rust/crates/xiuxian-qianhuan/src/zhenfa_router/native.rs`
   - `QianhuanReloadArgs` changed from unit struct to empty struct.

6. Updated corresponding native zhenfa test invocation:
   - `packages/rust/crates/xiuxian-daochang/src/agent/zhenfa/tests.rs`

## Validation Evidence

Commands executed:

1. `cargo fmt -p xiuxian-daochang`
2. `cargo fmt -p xiuxian-qianhuan -p xiuxian-daochang`
3. `CARGO_TARGET_DIR=target/clippy-xiuxian-daochang cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic -W clippy::too_many_lines`
4. `CARGO_TARGET_DIR=target/nextest-xiuxian-daochang cargo nextest run -p xiuxian-daochang`
5. `CARGO_TARGET_DIR=target/clippy-xiuxian-qianhuan cargo clippy -p xiuxian-qianhuan --all-targets -- -W clippy::too_many_lines`

Outcomes:

- Strict clippy command passed.
- `xiuxian-qianhuan` clippy command passed.
- Full nextest passed: `653 passed`, `0 failed`, `30 skipped`.

## Result

`xiuxian-daochang/src` remaining `#[allow(...)]` usage stayed constrained to one
targeted numeric-cast lane:

- `packages/rust/crates/xiuxian-daochang/src/agent/embedding_dimension.rs`

No remaining `unsafe_code` allowances are required in `xiuxian-daochang/src`
test modules touched by this wave.
