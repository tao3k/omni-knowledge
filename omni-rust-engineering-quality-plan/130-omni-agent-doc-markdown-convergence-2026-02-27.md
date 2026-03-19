# 修仙道场 (Xiuxian Daochang) Doc-Markdown Convergence (2026-02-27)

## Scope

Continue `xiuxian-daochang/tests` suppression-debt cleanup by removing
`clippy::doc_markdown` file-level allow markers and fixing surfaced doc style
warnings.

## Implemented Changes

1. Removed `clippy::doc_markdown` file-level allow markers across
   `packages/rust/crates/xiuxian-daochang/tests/**/*.rs`.
2. Fixed surfaced markdown doc warning:
   - `packages/rust/crates/xiuxian-daochang/tests/agent_integration.rs`
     - wrapped environment variable names in backticks:
       `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`.
3. Preserved no-suppression-first policy:
   - no new `#[allow(...)]` introduced.

## Verification Evidence

Executed:

```bash
rg -n "clippy::doc_markdown" packages/rust/crates/xiuxian-daochang/tests --glob '*.rs' | wc -l
cargo fmt -p xiuxian-daochang
cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
cargo clippy -p xiuxian-zhenfa -- -W clippy::pedantic
cargo clippy -p xiuxian-zhenfa -- -W clippy::too_many_lines
```

Result:

- `clippy::doc_markdown` marker count in `xiuxian-daochang/tests`: `0`.
- `cargo clippy -p xiuxian-daochang --tests -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines`: pass (`0`).
- `cargo clippy -p xiuxian-zhenfa -- -W clippy::pedantic`: pass (`0`).
- `cargo clippy -p xiuxian-zhenfa -- -W clippy::too_many_lines`: pass (`0`).

## Outcome

`xiuxian-daochang/tests` converged on `clippy::doc_markdown` with strict clippy lanes
clean and no new suppression introduced.
