# 250. xiuxian-daochang Discord Runtime Harness Warning Reduction Wave (2026-03-02)

## Scope

- Crate: `packages/rust/crates/xiuxian-daochang`
- Targets:
  - `channels_discord_runtime_unit`
  - `agent_memory_recall_state_unit`
- Objective:
  - reduce compile-time warning noise in migrated package-top harness lanes
    without using `#[allow(...)]`,
  - keep behavior and test outcomes unchanged.

## Root-Cause Fixes

1. Removed dead helper in test shim:
- `packages/rust/crates/xiuxian-daochang/tests/agent_memory_recall_state_unit.rs`
  - deleted unused `SessionStore::clear` to remove dead-code warning in that lane.

2. Added harness-local symbol probes (structural, no lint suppression):
- `packages/rust/crates/xiuxian-daochang/tests/channels_discord_runtime_unit.rs`
  - added focused `lint_symbol_probe` call-sites under local harness modules to
    exercise previously unreferenced but intentionally compiled symbols:
    - channel trait message fields and async methods,
    - managed-command type exports and scope constants,
    - managed-runtime parsing/json-summary/session-partition helpers,
    - discord session-partition method symbols,
    - discord runtime internal helpers (`push_background_completion`,
      telemetry function symbols, foreground snapshot construction).
  - this converts warning-producing "compiled but unreachable in this harness"
    items into explicitly referenced harness contracts.

## Validation Evidence

### 1) Targeted warning-focused lane run

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-daochang \
  --test channels_discord_runtime_unit \
  --test agent_memory_recall_state_unit
```

Result:
- `32 passed`, `0 failed`, `2 skipped`
- compile output: no warnings for these two targets

### 2) Wider touched-lane regression

```bash
RUSTC_WRAPPER= cargo nextest run -p xiuxian-daochang \
  --test channels_discord_runtime_unit \
  --test llm \
  --test embedding \
  --test agent_reflection_unit \
  --test agent_memory_recall_unit \
  --test agent_memory_recall_state_unit
```

Result:
- `65 passed`, `0 failed`, `2 skipped`

### 3) Mandatory clippy gate

```bash
RUSTC_WRAPPER= cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

Result:
- success (exit 0)

## Outcome

- The previously noisy Discord runtime harness compile path is now warning-clean
  under the validated target set.
- No broad lint suppression was introduced.
- Test behavior remains stable with full pass on touched lanes.
