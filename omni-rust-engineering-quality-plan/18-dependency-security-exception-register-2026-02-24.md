# Dependency Security Exception Register (2026-02-24)

## Purpose

Track temporary Rust dependency security exceptions with explicit ownership,
removal conditions, and review cadence.

This register is authoritative for temporary exceptions used by:

- `scripts/rust/cargo_audit_gate.sh`
- `deny.toml` (`[advisories].ignore`)

## Exception Register

| Advisory ID | Affected Crate | Root Chain | Scope | Owner | Removal Condition | Review Date |
| --- | --- | --- | --- | --- | --- | --- |
| `RUSTSEC-2023-0071` | `rsa` 0.9.10 | `sqlx-mysql -> sea-orm -> litellm-rs -> xiuxian-daochang` | `cargo audit` ignore | Rust Agent Integrations | `litellm-rs/sea-orm/sqlx` path no longer resolves vulnerable `rsa` line | 2026-03-10 |
| `RUSTSEC-2025-0141` | `bincode` 1.3.3 | `litellm-rs -> xiuxian-daochang` | `cargo audit` ignore + `cargo deny` ignore | Rust Agent Integrations | `litellm-rs` removes `bincode` 1.x or upstream replacement is adopted | 2026-03-10 |
| `RUSTSEC-2024-0436` | `paste` 1.0.15 | `lance/datafusion -> xiuxian-vector` | `cargo audit` ignore + `cargo deny` ignore | Rust Vector Platform | `lance/datafusion` path no longer resolves `paste` 1.0.x | 2026-03-10 |
| `RUSTSEC-2025-0134` | `rustls-pemfile` 1.0.4 | `reqwest 0.11 <- litellm-rs/serenity -> xiuxian-daochang` | `cargo audit` ignore + `cargo deny` ignore | Rust Agent Integrations | `litellm-rs` and `serenity` stack migrates off `reqwest` 0.11 pem chain | 2026-03-10 |
| `RUSTSEC-2026-0002` | `lru` 0.12.5 | `tantivy/lance -> xiuxian-vector` | `cargo audit` ignore | Rust Vector Platform | `tantivy/lance` chain resolves to non-affected `lru` version | 2026-03-10 |

## Operating Rules

1. Each exception must have one accountable owner (team or person).
2. Exceptions are temporary and must be reviewed at least every two weeks.
3. Any exception older than one quarter requires escalation and explicit re-approval.
4. When an exception is removed, update:
   - `scripts/rust/cargo_audit_gate.sh`
   - `deny.toml`
   - this register
   - `17-dependency-security-lane-bootstrap-2026-02-24.md`
