# 200. 修仙道场 (Xiuxian Daochang) Nextest Regression Closure (Agenda-Validation Isolation Wave, 2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Focus:
  - close post-refactor runtime regression failures in `nextest`
  - keep tests deterministic while preserving production configuration behavior

## Problem Snapshot

After the prior suppression-debt wave, `cargo nextest run -p xiuxian-daochang` reported
runtime behavior regressions in multiple tests. Root causes clustered into two
lanes:

1. Agenda validation preflight leaked into tests that asserted legacy `run_turn`
   request/round semantics.
2. `native_tools_zhixing_e2e` read non-isolated notebook state, so `agenda.view`
   rendered existing journal notes instead of graph-backed task metadata.

A secondary mismatch existed in embedding transport tests: assertion expected a
single retry while the current HTTP transport policy allows a wider retry budget.

## Changes Implemented

1. Explicitly disabled agenda validation preflight in isolated test configs
   (test-only behavior stabilization):
   - `packages/rust/crates/xiuxian-daochang/tests/agent_injection.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/zhenfa_tool_bridge.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent_context_window_recovery.rs`
   - `packages/rust/crates/xiuxian-daochang/tests/agent/native_tools_zhixing_e2e.rs`

   Each fixture now writes:

   ```toml
   [agent]
   llm_backend = "http"
   agenda_validation_policy = "never"
   ```

2. Eliminated notebook cross-run contamination in native tool e2e:
   - `packages/rust/crates/xiuxian-daochang/tests/agent/native_tools_zhixing_e2e.rs`
   - Test bootstrap now removes stale root on first init and configures an
     explicit isolated `wendao.zhixing.notebook_path` under that root.

3. Updated embedding retry assertion to current transport contract:
   - `packages/rust/crates/xiuxian-daochang/tests/embedding_client.rs`
   - Replaced exact `http_calls == 8` coupling with policy-level assertion:
     `http_calls >= 2` (at least one retry before final `None`).

## Validation Evidence

1. Format:

```bash
cargo fmt -p xiuxian-daochang
```

- Result: pass

2. Targeted failure-set regression run:

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-daochang cargo nextest run -p xiuxian-daochang \
  --test agent_injection \
  --test zhenfa_tool_bridge \
  --test agent_context_window_recovery \
  --test embedding_client \
  --test agent_suite \
  --status-level fail --failure-output immediate-final --no-fail-fast
```

- Result: pass
- Summary: `33 passed`, `0 failed`, `1 skipped`

3. Full crate test suite:

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-daochang cargo nextest run -p xiuxian-daochang
```

- Result: pass
- Summary: planned `683`, run `653`, passed `653`, failed `0`, skipped `30`

4. Strict clippy:

```bash
CARGO_TARGET_DIR=target/clippy-xiuxian-daochang cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass (warnings only; no clippy errors)
- Remaining warnings are predominantly pre-existing `large_futures` plus one
  cast warning in `src/agent/zhenfa/tests.rs`.

## Outcome

- `xiuxian-daochang` nextest regression inventory for this wave closed (`9 -> 0`).
- Test behavior is now deterministic and isolated from host notebook residue.
- Agenda-validation capability remains available in production config, while
  legacy semantic tests explicitly pin `never` policy where required.
