# 修仙道场 (Xiuxian Daochang) Struct-Field-Names Compatibility Migration Plan (2026-02-26)

## Scope

This shard is a design-only migration plan (no production code changes in this
 shard) for the final `xiuxian-daochang` suppression category:

- `clippy::struct_field_names` (3 remaining sites)

Remaining sites:

- `packages/rust/crates/xiuxian-daochang/src/config/settings/types.rs` (`McpSettings`)
- `packages/rust/crates/xiuxian-daochang/src/channels/telegram/channel/policy.rs` (`TelegramSlashCommandPolicy`)
- `packages/rust/crates/xiuxian-daochang/src/channels/discord/channel/policy.rs` (`DiscordSlashCommandPolicy`)

## Why This Needs a Compatibility Plan

These fields are externally meaningful:

- config keys are user-facing YAML contract,
- slash policy fields are wired into ACL precedence and command authorization,
- tests and runtime merge logic depend on current names.

A direct rename would break existing settings and call sites. The migration must
be additive-first and contract-safe.

## Target Design

### 1) MCP settings: nested agent section as canonical internal model

Current shape (legacy style):

- `agent_pool_size`
- `agent_handshake_timeout_secs`
- `agent_connect_retries`
- ...

Target shape (canonical):

- `mcp.agent.pool_size`
- `mcp.agent.handshake_timeout_secs`
- `mcp.agent.connect_retries`
- ...

Compatibility rule:

- legacy flat keys remain accepted for at least one full migration cycle,
- canonical nested keys are preferred in documentation and runtime normalization,
- merge logic resolves to one canonical in-memory structure.

### 2) Slash command policy: scope-oriented structure

Current style repeats `*_allow_from` in every field.

Target canonical model:

- `global_allow_from`
- `scopes.session_status`
- `scopes.session_budget`
- `scopes.session_memory`
- `scopes.session_feedback`
- `scopes.job_status`
- `scopes.jobs_summary`
- `scopes.background_submit`

Compatibility rule:

- keep old field names as compatibility aliases/wrappers during transition,
- keep authorization precedence unchanged:
  1. global override
  2. command-specific allowlist
  3. admin fallback

## Migration Waves

### Wave 0: Contract Inventory and Freeze

- Freeze current behavior with focused tests:
  - MCP settings deserialization + merge precedence.
  - Telegram/Discord slash authorization precedence.
- Add explicit “legacy-key compatibility” test table before refactor.

Exit criteria:

- baseline tests capture current behavior exactly.

### Wave 1: Add Canonical Models (No Breaking Changes)

- Introduce canonical nested structs for MCP and slash policy.
- Keep legacy fields accepted through serde aliases/flatten bridge.
- Add normalization adapters:
  - `from_legacy_or_canonical(...) -> CanonicalType`.

Exit criteria:

- old config and old call sites still pass unchanged,
- canonical inputs also pass.

### Wave 2: Internal Call-Site Cutover

- Switch internal code to consume only canonical fields/types.
- Keep legacy compatibility layer at boundaries (config parse / constructors).
- Keep public constructors stable; add new canonical constructors where needed.

Exit criteria:

- internal references to legacy duplicated names are removed,
- behavior parity maintained in integration tests.

### Wave 3: Deprecation Window

- Mark legacy fields/helpers as deprecated with migration notes.
- Update docs/examples to canonical shape only.
- Optional warning logs when legacy keys are loaded.

Exit criteria:

- users have migration guidance and non-breaking transition path.

### Wave 4: Legacy Removal (Future, explicit decision)

- Remove legacy aliases only after agreed deprecation horizon and release note.

Exit criteria:

- no legacy usage in repo configs/tests,
- compatibility sunset approved.

## Validation Matrix

For each wave, require:

- `cargo fmt -p xiuxian-daochang`
- `cargo clippy -p xiuxian-daochang --all-targets -- -W clippy::pedantic`
- `cargo test -p xiuxian-daochang --lib`
- focused suites:
  - `cargo test -p xiuxian-daochang --test config_settings`
  - `cargo test -p xiuxian-daochang --test channels_telegram_slash_authorization`
  - `cargo test -p xiuxian-daochang --test channels_discord_slash_authorization`
  - `cargo test -p xiuxian-daochang --test telegram_acl_overrides`
  - `cargo test -p xiuxian-daochang --test discord_acl_overrides`

## Risks and Controls

Risk: silent precedence drift in ACL rules.

Control:

- keep a dedicated precedence snapshot test set,
- compare canonical-vs-legacy inputs with paired assertions.

Risk: user config breakage.

Control:

- additive parser first, hard removal last,
- migration guide with old/new key mapping table.

Risk: partial refactor leaves dual sources of truth.

Control:

- enforce one canonical in-memory type immediately in Wave 2,
- keep legacy only at ingestion boundaries.

## Immediate Next Action (Recommended)

Execute Wave 0 and Wave 1 together in one PR-sized slice:

- add canonical structs + normalization adapters,
- add compatibility tests (old and new forms),
- do not remove any legacy key in this slice.

This gives measurable progress while keeping rollout risk low.
