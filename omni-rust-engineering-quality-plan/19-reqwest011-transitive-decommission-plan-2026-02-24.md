# Reqwest 0.11 Transitive Decommission Plan (2026-02-24)

## Objective

Remove the transitive `reqwest 0.11` chain from `xiuxian-daochang` to eliminate
`RUSTSEC-2025-0134` (`rustls-pemfile 1.0.4`) and reduce related legacy stack risk.

## Current Evidence

1. `xiuxian-daochang` currently depends on:
   - `litellm-rs v0.3.1`
   - `serenity v0.12.5`
2. Dependency tree signal confirms `reqwest v0.11.27` and
   `rustls-pemfile v1.0.4` appear in the `xiuxian-daochang` graph.
3. `cargo info` revalidation shows no newer published versions for
   `litellm-rs` and `serenity` in the current crate line.

## Progress Update (2026-02-24, later pass)

1. Capability split for `litellm-rs` was implemented with real compile-time
   gating (`agent-provider-litellm`) and validated in both:
   - default profile,
   - `--no-default-features` profile.
2. Detailed evidence is recorded in:
   - `21-xiuxian-daochang-feature-gated-profile-validation-2026-02-24.md`.
3. `serenity` isolation was intentionally deferred to a separate slice to keep
   this wave focused on LLM/backend boundary reduction.
4. Dependency-graph guardrail is now wired in CI:
   - `23-xiuxian-daochang-dependency-graph-assertions-gate-2026-02-24.md`.

## Scope

- Primary crate: `packages/rust/crates/xiuxian-daochang`
- Dependencies to pressure:
  - `litellm-rs`
  - `serenity`

## Execution Strategy

1. Split dependency usage by capability in `xiuxian-daochang`:
   - Isolate `litellm-rs` and `serenity` call sites into distinct internal
     modules.
   - Add compile-time feature gates so each integration can be independently
     enabled/disabled in CI experiments.

2. Introduce an HTTP boundary abstraction:
   - Define an internal trait for outbound provider calls used by agent runtime.
   - Add a native implementation using workspace `reqwest 0.12` stack.
   - Keep `litellm-rs` adapter as compatibility backend behind a feature gate.

3. Upstream pressure and fork fallback:
   - Track upstream issues/PRs for `litellm-rs` and `serenity` dependency updates.
   - If upstream stagnates, evaluate a minimal internal patch/fork branch that
     removes `reqwest 0.11` requirement.

4. Security gate tightening:
   - Once `reqwest 0.11` is removed from lock graph, remove
     `RUSTSEC-2025-0134` from:
     - `scripts/rust/cargo_audit_gate.sh`
     - `deny.toml`
     - `18-dependency-security-exception-register-2026-02-24.md`

## Validation Commands

```bash
# verify chain presence/absence
cargo tree -p xiuxian-daochang -e all | rg "reqwest v0\.11\.27|rustls-pemfile v1\.0\.4"

# security gate
just rust-security-gate
# or
devenv tasks run ci:rust-security-gate
```

## Exit Criteria

1. `cargo tree -p xiuxian-daochang -e all` no longer contains `reqwest v0.11.27`.
2. `cargo tree -p xiuxian-daochang -e all` no longer contains `rustls-pemfile v1.0.4`.
3. `RUSTSEC-2025-0134` is removed from temporary exception lists.
4. Rust security gate remains green.

## Owner

- Rust Agent Integrations
