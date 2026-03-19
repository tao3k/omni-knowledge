# Xiuxian-MCP And LLM Test Pedantic Convergence Wave (2026-02-26)

## Scope

Converge `xiuxian-mcp` and `xiuxian-llm` test lanes on strict clippy
pedantic, with no suppression additions and no legacy fallback shortcuts.

## Why

Pedantic runs showed hard failures in both crates caused by test-level
`expect`/`unwrap` usage, plus style warnings (`doc_markdown`,
`match_wildcard_for_single_variants`, `uninlined_format_args`).

These failures blocked high-quality workspace enforcement.

## Implemented Changes

### `xiuxian-mcp` tests

1. Migrated `expect`/`unwrap_err` to `Result`-based flow in:
   - `tests/config.rs`
   - `tests/client.rs`
   - `tests/streamable_http_integration.rs`
2. Removed redundant single-component import and wildcard single-variant
   match arms in config tests.
3. Fixed doc markdown and format style in integration/client tests.
4. Refactored shared assertion helper to return `Result<()>` and propagate
   failures with `?`.

### `xiuxian-llm` tests

1. Migrated `expect`/`expect_err` to `Result` + explicit `Err` branch checks in:
   - `tests/llm_openai_client.rs`
2. Refactored mock server bootstrap to return `Result<String>` and use `?`
   for bind/address failure propagation.

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-mcp -p xiuxian-llm
cargo clippy -p xiuxian-mcp --all-targets -- -W clippy::pedantic
cargo clippy -p xiuxian-llm --all-targets -- -W clippy::pedantic
cargo test -p xiuxian-mcp --test config
cargo test -p xiuxian-mcp --test client
cargo test -p xiuxian-mcp --test streamable_http_integration
cargo test -p xiuxian-llm --test llm_openai_client
```

Result:

- Both pedantic clippy lanes passed.
- `xiuxian-mcp` tests passed (`4 + 4 + 1`, with 1 ignored real-server test).
- `xiuxian-llm` OpenAI client tests passed (`3` passed).

## Outcome

`xiuxian-mcp` and `xiuxian-llm` now comply with strict pedantic test gates for
the covered lanes without suppression-based bypasses, strengthening workspace
quality-gate reliability.
