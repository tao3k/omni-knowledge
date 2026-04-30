# 239. `xiuxian-daochang` Bootstrap Tests Top-Level Harness Migration and `src` Path-Mount Zero (2026-03-01)

## Scope

- Remove the final `src`-side `#[cfg(test)] #[path = "../../tests/..."]` mount in
  `xiuxian-daochang`.
- Keep bootstrap helper tests executable from package-top `tests/`.
- Revalidate strict clippy and targeted nextest after migration.

## Changes

1. Removed final `src`-side bootstrap test mount

- File:
  - `packages/rust/crates/xiuxian-daochang/src/agent/bootstrap.rs`
- Removed:
  - `#[cfg(test)] #[path = "../../tests/unit/agent/bootstrap_tests.rs"] mod tests;`

2. Added top-level bootstrap harness

- File:
  - `packages/rust/crates/xiuxian-daochang/tests/agent_bootstrap.rs`
- Added a dedicated package-top harness that reuses:
  - `packages/rust/crates/xiuxian-daochang/tests/unit/agent/bootstrap_tests.rs`
- The harness hosts bootstrap helper modules (`service_mount`, `hot_reload`,
  `memory`, `qianhuan`, `zhenfa`, `zhixing`) needed by existing tests.

3. Stabilized one fixture assertion against current embedded steward persona text

- File:
  - `packages/rust/crates/xiuxian-daochang/tests/unit/agent/bootstrap_tests.rs`
- Updated the semantic-content assertion to accept either legacy phrase
  (`Agenda Steward Persona`) or current embedded heading (`Clockwork Guardian`)
  while preserving the same contract (embedded steward persona content is loaded).

## Validation Evidence

1. Migrated bootstrap lane strict clippy

```bash
cargo clippy -p xiuxian-daochang --test agent_bootstrap -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: warning-zero.

2. Migrated bootstrap lane nextest

```bash
cargo nextest run -p xiuxian-daochang --test agent_bootstrap
```

- Exit code: `0`
- Result: `18 passed`, `0 failed`.

3. Aggregated migrated-lane revalidation

```bash
cargo clippy -p xiuxian-daochang --test agent_bootstrap --test agent_admission --test agent_memory_recall_feedback --test agent_memory_recall_metrics -- -W clippy::too_many_lines
cargo nextest run -p xiuxian-daochang --test agent_bootstrap --test agent_admission --test agent_memory_recall_feedback --test agent_memory_recall_metrics
```

- Exit code: `0`
- Result: clippy warning-zero; nextest `40 passed`, `0 failed`.

4. Extended migrated-lane matrix revalidation

```bash
cargo clippy -p xiuxian-daochang --test llm --test embedding --test nodes_warmup --test config_xiuxian --test agent_mcp_startup --test agent_memory_recall_feedback --test agent_memory_recall_metrics --test agent_admission --test agent_bootstrap -- -W clippy::too_many_lines
cargo nextest run -p xiuxian-daochang --test llm --test embedding --test nodes_warmup --test config_xiuxian --test agent_mcp_startup --test agent_memory_recall_feedback --test agent_memory_recall_metrics --test agent_admission --test agent_bootstrap
```

- Exit code: `0`
- Result: clippy warning-zero; nextest `69 passed`, `0 failed`.

5. Mandatory touched-crate strict clippy

```bash
cargo clippy -p xiuxian-daochang -- -W clippy::too_many_lines
```

- Exit code: `0`
- Result: warning-zero.

6. Path-mount zero verification

```bash
rg --line-number --glob 'packages/rust/crates/xiuxian-daochang/src/**/*.rs' '#\\[path\\s*=\\s*\"\\.\\./\\.\\./tests/.*\"\\]'
```

- Exit code: `1`
- Result: no matches (expected), indicating `xiuxian-daochang/src` is now clear of
  `#[path = "../../tests/..."]` hooks.

## Outcome

- The final `xiuxian-daochang` `src` path-mounted test hook has been removed.
- Bootstrap helper tests now run through a package-top test target.
- `xiuxian-daochang` reaches `src` path-mount zero for this migration line, with
  strict clippy and targeted nextest evidence.
