# 241 - `src` Test Path-Mount Burndown Wave 2 (Infra Crates + Dependency Indexer Lanes) - 2026-03-02

## Scope

This wave continues removal of `src`-side test path mounts by migrating selected
infrastructure and support crates to package-top integration tests.

Target crates in this wave:

- `omni-events`
- `omni-executor`
- `omni-io`
- `omni-security`
- `omni-tui` (socket lane)
- `xiuxian-vector` (fusion kernels lane)
- `xiuxian-memory`
- `xiuxian-wendao` (dependency indexer lanes)

## Structural Changes

### Removed `src` test path-mounts

- `omni-events/src/lib.rs`
- `omni-executor/src/query.rs`
- `omni-io/src/assembler.rs`
- `omni-io/src/discover.rs`
- `omni-io/src/watcher.rs`
- `omni-security/src/sandbox.rs`
- `omni-tui/src/socket.rs`
- `xiuxian-vector/src/keyword/fusion/kernels.rs`
- `xiuxian-memory/src/lib.rs`
- `xiuxian-wendao/src/dependency_indexer/cargo/mod.rs`
- `xiuxian-wendao/src/dependency_indexer/indexer/mod.rs`
- `xiuxian-wendao/src/dependency_indexer/symbols/mod.rs`

### Added package-top integration tests

- `omni-events/tests/lib_unit.rs`
- `omni-executor/tests/query_unit.rs`
- `omni-io/tests/assembler_unit.rs` (`feature = "assembler"`)
- `omni-io/tests/discover_unit.rs`
- `omni-io/tests/watcher_unit.rs` (`feature = "notify"`)
- `omni-security/tests/sandbox_unit.rs`
- `omni-tui/tests/socket_unit.rs`
- `xiuxian-vector/tests/keyword_fusion_kernels.rs`
- `xiuxian-memory/tests/lib_unit.rs`
- `xiuxian-wendao/tests/dependency_indexer_cargo_unit.rs`
- `xiuxian-wendao/tests/dependency_indexer_indexer_unit.rs`
- `xiuxian-wendao/tests/dependency_indexer_symbols_unit.rs`

### Deleted obsolete unit files previously mounted from `src`

- `omni-events/tests/unit/lib_tests.rs`
- `omni-executor/tests/unit/query_tests.rs`
- `omni-io/tests/unit/assembler_tests.rs`
- `omni-io/tests/unit/discover_tests.rs`
- `omni-io/tests/unit/watcher_tests.rs`
- `omni-security/tests/unit/sandbox_tests.rs`
- `omni-tui/tests/unit/socket_tests.rs`
- `xiuxian-vector/tests/unit/keyword/fusion/kernels_tests.rs`
- `xiuxian-memory/tests/unit/lib_tests.rs`
- `xiuxian-wendao/tests/unit/dependency_indexer/cargo/tests.rs`
- `xiuxian-wendao/tests/unit/dependency_indexer/indexer/tests.rs`
- `xiuxian-wendao/tests/unit/dependency_indexer/symbols/tests.rs`

## Validation Evidence

### Mandatory strict clippy for touched crates

```bash
cargo clippy -p omni-events -p omni-executor -p omni-io -p omni-security -p omni-tui -p xiuxian-vector -p xiuxian-memory -p xiuxian-wendao -- -W clippy::too_many_lines
```

Result: success, no warnings/errors.

### Targeted `nextest` proofs

```bash
cargo nextest run -p omni-events --test lib_unit
cargo nextest run -p omni-executor --test query_unit
cargo nextest run -p omni-io --test discover_unit
cargo nextest run -p omni-security --test sandbox_unit
cargo nextest run -p omni-tui --test socket_unit
cargo nextest run -p xiuxian-vector --test keyword_fusion_kernels
cargo nextest run -p xiuxian-memory --test lib_unit
cargo nextest run -p xiuxian-wendao --test dependency_indexer_cargo_unit --test dependency_indexer_indexer_unit --test dependency_indexer_symbols_unit
cargo nextest run -p omni-io --features assembler --test assembler_unit
cargo nextest run -p omni-io --features notify --test watcher_unit
```

Results:

- `omni-events/lib_unit`: `5 passed`, `0 failed`
- `omni-executor/query_unit`: `6 passed`, `0 failed`
- `omni-io/discover_unit`: `4 passed`, `0 failed`
- `omni-security/sandbox_unit`: `3 passed`, `0 failed`
- `omni-tui/socket_unit`: `2 passed`, `0 failed`
- `xiuxian-vector/keyword_fusion_kernels`: `3 passed`, `0 failed`
- `xiuxian-memory/lib_unit`: `1 passed`, `0 failed`
- `xiuxian-wendao` dependency indexer trio: `9 passed`, `0 failed`
- `omni-io/assembler_unit` (`assembler` feature): `3 passed`, `0 failed`
- `omni-io/watcher_unit` (`notify` feature): `1 passed`, `0 failed`

## Burndown Status

Global `src` path-mount scan (all crates):

```bash
rg --line-number --glob 'packages/rust/crates/*/src/**/*.rs' '#\[path\s*=\s*"[^"]*tests/[^"]*"\]' | wc -l
```

Current remaining count: `41`.

The remaining mounts are concentrated in:

- `xiuxian-daochang`
- `omni-ast`
- `omni-tui` (`src/main.rs`)
- `xiuxian-vector` (`match_util`, `search_impl`)
- `xiuxian-skills`
- `xiuxian-wendao` (`fusion`, `xml_lite`)

## Outcome

This wave removed another large block of `src`-side test path-mounts without
introducing lint suppressions, and preserved green strict-clippy/nextest
evidence on all touched crates.
