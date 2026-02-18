# Evidence Metrics Snapshot (2026-02-18)

This snapshot records codex-only metrics used by this reference set.

## 1. Structural Metrics

| Metric | Value |
| --- | ---: |
| Codex crate directories with `Cargo.toml` | 45 |
| Codex crates with `[lints] workspace = true` | 38 |
| Codex Rust files >500 lines | 197 |
| Codex Rust files >1000 lines | 77 |
| Codex top-level workflow files | 18 |
| Codex crate roots with `deny(print_stdout/print_stderr)` | 7 |
| Dedicated `cargo-deny` workflow | present |
| Dedicated `cargo-audit` workflow | present |
| `clippy -D warnings` lane in Rust CI | present |
| `cargo nextest` lane in Rust CI | present |

## 2. Representative Large Modules

- `.cache/researcher/openai/codex/codex-rs/tui/src/bottom_pane/chat_composer.rs` (8403)
- `.cache/researcher/openai/codex/codex-rs/core/src/codex.rs` (8394)
- `.cache/researcher/openai/codex/codex-rs/tui/src/chatwidget/tests.rs` (7342)
- `.cache/researcher/openai/codex/codex-rs/tui/src/chatwidget.rs` (7330)
- `.cache/researcher/openai/codex/codex-rs/app-server/src/codex_message_processor.rs` (6970)

## 3. Governance Evidence Anchors

- Workspace lint contract:
  - `.cache/researcher/openai/codex/codex-rs/Cargo.toml` (`[workspace.lints.clippy]`)
- Rust CI quality lanes:
  - `.cache/researcher/openai/codex/.github/workflows/rust-ci.yml`
- Supply-chain security:
  - `.cache/researcher/openai/codex/.github/workflows/cargo-deny.yml`
  - `.cache/researcher/openai/codex/codex-rs/.github/workflows/cargo-audit.yml`
  - `.cache/researcher/openai/codex/codex-rs/deny.toml`
- Local developer parity:
  - `.cache/researcher/openai/codex/justfile` (`clippy`, `test`/`nextest`)

## 4. Reproduction Command Fragments

```bash
# Count codex crates and lint inheritance
ls .cache/researcher/openai/codex/codex-rs/*/Cargo.toml | wc -l
rg -l '^\[lints\]' .cache/researcher/openai/codex/codex-rs/*/Cargo.toml | wc -l

# Count large codex Rust files
find .cache/researcher/openai/codex/codex-rs -name '*.rs' -type f -print0 | xargs -0 wc -l | awk '$1>500{c++} END{print c+0}'
find .cache/researcher/openai/codex/codex-rs -name '*.rs' -type f -print0 | xargs -0 wc -l | awk '$1>1000{c++} END{print c+0}'

# Count codex workflow files
find .cache/researcher/openai/codex/.github/workflows -maxdepth 1 -type f | wc -l

# Count codex crate roots denying stdout/stderr prints
rg -l 'deny\(clippy::print_stdout|deny\(clippy::print_stderr' \
  .cache/researcher/openai/codex/codex-rs/*/src/lib.rs | wc -l

# Verify key CI lanes
rg -n 'clippy|nextest' .cache/researcher/openai/codex/.github/workflows/rust-ci.yml
rg -n 'cargo-deny' .cache/researcher/openai/codex/.github/workflows/cargo-deny.yml
find .cache/researcher/openai/codex/codex-rs/.github/workflows -maxdepth 1 -name '*audit*'
```

## 5. Interpretation Note

These metrics are codex internal observations.
Project-specific gap analysis should live outside this directory.
