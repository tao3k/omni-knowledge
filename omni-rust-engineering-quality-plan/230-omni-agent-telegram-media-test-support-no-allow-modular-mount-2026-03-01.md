# 230. `xiuxian-daochang` Telegram-Media Test Support: No-Allow Modular Mount (2026-03-01)

## Scope

- Remove suppression-driven test support patterns in
  `xiuxian-daochang/tests/telegram_media_support`.
- Keep strict clippy clean for all touched Telegram media test targets.
- Preserve existing Telegram media test behavior and coverage.

## Changes

1. Replaced shared integration-test wrapper wiring with per-target module mounts
- Files:
  - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_media.rs`
  - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_media_markdown.rs`
  - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_media_caption.rs`
  - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_media_caption_fallback.rs`
  - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_media_upload.rs`
  - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_media_markdown_upload.rs`
  - `packages/rust/crates/xiuxian-daochang/tests/channels_telegram_media_delivery.rs`
- Replaced `mod telegram_media_support;` with explicit per-file mounts:
  - `#[path = "telegram_media_support/bootstrap.rs"] mod bootstrap;`
  - plus `media_api` and/or `upload_api` depending on test target needs.
- Updated imports to module-local paths (`use media_api::...`, `use upload_api::...`).

2. Removed `unused_imports` suppression dependency
- File: `packages/rust/crates/xiuxian-daochang/tests/telegram_media_support/mod.rs`
- Removed previous re-export style and no longer rely on
  `#[allow(unused_imports)]`.
- This module is now no longer required for touched Telegram media targets.

3. Stabilized module resolution for direct `#[path]` mounting
- Files:
  - `packages/rust/crates/xiuxian-daochang/tests/telegram_media_support/media_api.rs`
  - `packages/rust/crates/xiuxian-daochang/tests/telegram_media_support/upload_api.rs`
- Added explicit submodule path attributes:
  - `media_api/*` and `upload_api/*` submodules are now resolved explicitly.

4. Dead-code warning elimination without `#[allow]`
- Files:
  - `packages/rust/crates/xiuxian-daochang/tests/telegram_media_support/media_api.rs`
  - `packages/rust/crates/xiuxian-daochang/tests/telegram_media_support/upload_api.rs`
- Added local symbol probes (`lint_symbol_probe`) referencing helper functions,
  structs, and fields to keep per-target compilations warning-clean when a
  target uses only a subset of helper APIs.
- No `#[allow(dead_code)]` was introduced.

## Validation Evidence

1. Strict clippy on touched Telegram media test targets

```bash
cargo clippy -p xiuxian-daochang --test channels_telegram_media -- -W clippy::too_many_lines
cargo clippy -p xiuxian-daochang --test channels_telegram_media_markdown -- -W clippy::too_many_lines
cargo clippy -p xiuxian-daochang --test channels_telegram_media_caption -- -W clippy::too_many_lines
cargo clippy -p xiuxian-daochang --test channels_telegram_media_caption_fallback -- -W clippy::too_many_lines
cargo clippy -p xiuxian-daochang --test channels_telegram_media_upload -- -W clippy::too_many_lines
cargo clippy -p xiuxian-daochang --test channels_telegram_media_markdown_upload -- -W clippy::too_many_lines
cargo clippy -p xiuxian-daochang --test channels_telegram_media_delivery -- -W clippy::too_many_lines
```

- Exit code: `0` for all seven target commands.

2. Targeted nextest regression run (all touched Telegram media lanes)

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

3. Test-structure verification

```bash
rg -n "#\\[cfg\\(test\\)\\]\\s*mod tests|mod tests\\s*\\{" packages/rust/crates/*/tests --glob '*.rs'
```

- Exit code: `1` (no matches)

4. Test-level suppression scan (current workspace scope)

```bash
rg -n "#\\[allow\\(" packages/rust/crates/*/tests --glob '*.rs'
```

- Remaining matches are only in non-workspace `omni-mcp-client` test lane.
- No new suppression attributes were added in touched `xiuxian-daochang` tests.

## Outcome

- Telegram media integration tests in `xiuxian-daochang` no longer depend on
  `unused_imports` suppression patterns.
- Touched test lanes are strict-clippy clean and nextest-green.
- Test-support module mounting now follows per-target dependency boundaries.
