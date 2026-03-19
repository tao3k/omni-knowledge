# 修仙道场 (Xiuxian Daochang) Embedding Test Allow-Debt Reduction Wave (2026-02-26)

## Scope

Reduce stale `expect/unwrap` suppression debt in embedding-focused
`xiuxian-daochang` test modules that no longer use panic-style `expect/unwrap`.

## Why

Embedding transport and backend tests are part of the core regression surface
for provider compatibility (`OpenAI`/`LiteLLM`/Ollama path behaviors). Keeping
obsolete suppressions in these files weakens strict-lint signal quality.

## Implemented Changes

Removed file-level `clippy::expect_used` / `clippy::unwrap_used` from:

1. `tests/embedding/backend.rs`
2. `tests/embedding/transport_litellm.rs`
3. `tests/embedding/transport_openai.rs`
4. `tests/embedding_role_perf_smoke.rs`

## Verification Evidence

Executed:

```bash
cargo fmt -p xiuxian-daochang
cargo test -p xiuxian-daochang --test embedding_client
cargo test -p xiuxian-daochang --test embedding_role_perf_smoke
cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic
rg -n "clippy::expect_used|clippy::unwrap_used" packages/rust/crates/xiuxian-daochang/tests | cut -d: -f1 | sort -u | wc -l
```

Result:

- `embedding_client` target passed (`12 passed`).
- `embedding_role_perf_smoke` remains intentionally ignored by default.
- `xiuxian-daochang` remained green under pedantic clippy.
- `xiuxian-daochang/tests` allow-marker file count dropped from `96` to `92`.

## Outcome

Embedding test lanes now carry less stale suppression debt, preserving behavior
while tightening strict-gate credibility for transport/backend evolution.
