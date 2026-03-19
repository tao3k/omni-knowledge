# 231. `xiuxian-daochang` All-Target Clippy Unblock via Valkey-Hooks and Wendao-Refresh Import Fix (2026-03-01)

## Scope

- Unblock `cargo clippy -p xiuxian-daochang --all-targets` hard failures encountered
  during Telegram media test-support convergence.
- Remove `expect_err` usage in touched test code path.
- Repair cross-crate import drift in `xiuxian-qianji` that blocked dependency
  compilation under the same clippy run.

## Changes

1. Removed `expect_err` hard error in valkey live hook test
- File: `packages/rust/crates/xiuxian-daochang/tests/unit/agent/zhenfa/valkey_hooks_tests.rs`
- Replaced:
  - `expect_err("...")` on mutation-lock contention path.
- With:
  - explicit `match` that panics on unexpected `Ok(...)` and captures `Err(...)`
    for normal assertion flow.
- Also normalized literal-bound signatures in helper tools:
  - `fn id(&self) -> &'static str` for static IDs.

2. Fixed LinkGraph refresh enum import path drift
- File: `packages/rust/crates/xiuxian-qianji/src/executors/wendao_refresh.rs`
- Updated refresh-mode import to current public location:
  - `use xiuxian_wendao::link_graph::LinkGraphRefreshMode;`
  - `use xiuxian_wendao::LinkGraphIndex;`
- This removed the unresolved import error that blocked transitive compilation
  while running `xiuxian-daochang` all-target clippy.

## Validation Evidence

1. All-target strict clippy for `xiuxian-daochang`

```bash
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::too_many_lines
```

- Exit code: `0` (unblocked)
- Notes:
  - Remaining output is warning-level (for example `too_many_lines`,
    `unnecessary_wraps`, `missing_docs`) and no hard failures.

2. Telegram media target-lane regression validation (from adjacent wave)

```bash
cargo nextest run -p xiuxian-daochang \
  --test channels_telegram_media \
  --test channels_telegram_media_markdown \
  --test channels_telegram_media_caption \
  --test channels_telegram_media_caption_fallback \
  --test channels_telegram_media_upload \
  --test channels_telegram_media_markdown_upload \
  --test channels_telegram_media_delivery
```

- Exit code: `0`
- Result: `25 passed`, `0 skipped`

## Outcome

- `xiuxian-daochang` all-target strict clippy run no longer fails at compile/error
  stage.
- `expect_err` denial path in touched valkey hook test has been replaced with
  explicit error handling.
- Cross-crate `xiuxian-qianji` -> `xiuxian-wendao` refresh import alignment is
  restored for this lane.
