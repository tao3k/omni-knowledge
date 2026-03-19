# 199. 修仙道场 (Xiuxian Daochang) Test Allow-Debt Zero and Settings-Merge Compatibility Wave (2026-02-28)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Focus:
  - remove file-level `#![allow(...)]` debt in `tests/`
  - preserve compile compatibility after `AgentSettings` schema expansion

## Why This Wave

`xiuxian-daochang/tests` still carried broad file-level `#![allow(missing_docs)]` suppressions.
In parallel, `AgentSettings` gained `agenda_validation_policy`, and one merge initializer
was not updated, which blocked the crate build.

## Changes Implemented

1. Removed file-level `#![allow(...)]` from `xiuxian-daochang/tests` entry files.
   - Every touched test file now starts with explicit module documentation instead of a
     blanket suppression.

2. Kept suppression-free cleanup policy in test internals.
   - In `packages/rust/crates/xiuxian-daochang/src/session/redis_backend/tests.rs`, replaced
     `expect(...)` usage with `Result`-returning tests and `?`/`ok_or_else(...)`
     propagation.

3. Fixed compile compatibility in settings merge path.
   - Updated `packages/rust/crates/xiuxian-daochang/src/config/settings/merge/core.rs`
     (`AgentSettings::merge`) to merge the newly added
     `agenda_validation_policy: Option<String>` field.

4. Reduced pedantic warning noise in touched code paths.
   - `packages/rust/crates/xiuxian-daochang/src/agent/turn_execution/react_loop/agenda_validation.rs`:
     removed needless raw-string hash delimiters.
   - `packages/rust/crates/xiuxian-daochang/tests/config_settings.rs`:
     removed needless raw-string hash delimiters in TOML fixtures.
   - `packages/rust/crates/xiuxian-daochang/tests/scenario_adversarial_evolution.rs`:
     extracted reward-reinforcement assertions into a helper to satisfy
     `clippy::too_many_lines`, and replaced `f64 as f32` casts with
     `ToPrimitive::to_f32()` conversion.

No new `#![allow(...)]` suppressions were introduced.

## Validation Evidence

1. Allow-debt check:

```bash
rg -n '^#!\[allow\(' packages/rust/crates/xiuxian-daochang/tests -g '*.rs'
```

- Result: no matches

2. Format:

```bash
cargo fmt -p xiuxian-daochang
```

- Result: pass

3. Strict clippy:

```bash
CARGO_TARGET_DIR=target/clippy-xiuxian-daochang cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic -W clippy::too_many_lines
```

- Result: pass (warnings only; no clippy errors)
- Notable existing warnings are now primarily `large_futures` (plus one
  existing cast warning in `src/agent/zhenfa/tests.rs`).

4. Test suite:

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-daochang cargo nextest run -p xiuxian-daochang
```

- Result: fail in current workspace
- Summary: planned `651`, run `295`, passed `290`, failed `5`, skipped `30`, not run `356`

Additional failure inventory run:

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-daochang cargo nextest run -p xiuxian-daochang --no-fail-fast --status-level fail --failure-output immediate-final
```

- Reported failing tests (current tree):
  - `react_loop_tool_call_roundtrip_with_mock_llm_and_mcp`
  - `react_role_mix_switches_from_recovery_back_to_normal_after_failure_cycle`
  - `react_failure_reflection_injects_next_turn_hint_and_recovers`
  - `react_shortcut_strips_prefix_before_llm_prompt`
  - `run_turn_does_not_retry_for_non_context_error`
  - `agent::native_tools_zhixing_e2e::zhixing_e2e_tool_loop_reads_metadata_and_proactively_rejects_malicious_request`
  - `run_turn_dispatches_qianhuan_reload_through_native_zhenfa_bridge`
  - `run_turn_dispatches_wendao_search_through_native_zhenfa_bridge`
  - `embed_batch_returns_none_when_http_fails_even_if_mcp_url_is_set`

5. Targeted re-validation of touched tests:

```bash
CARGO_TARGET_DIR=target/nextest-xiuxian-daochang cargo nextest run -p xiuxian-daochang --test config_settings --test scenario_adversarial_evolution
```

- Result: pass
- Summary: `10 passed`, `0 failed`, `0 skipped`

## Debt-Burndown Snapshot

- `git diff -- packages/rust/crates/xiuxian-daochang/tests | rg -n '^-#!\[allow\(' | wc -l`
  - Removed suppression lines in this wave: `52`
- `rg -n '^#!\[allow\(' packages/rust/crates/xiuxian-daochang/tests -g '*.rs' | wc -l`
  - Remaining file-level allow suppressions: `0`

## Engineering Outcome

- `xiuxian-daochang/tests` file-level allow-debt is now zero.
- `AgentSettings` merge path is schema-compatible with the new
  `agenda_validation_policy` field, so compile no longer fails on missing-field
  initialization.
- Runtime behavior regressions still exist in the current branch and remain the
  next follow-up lane.
