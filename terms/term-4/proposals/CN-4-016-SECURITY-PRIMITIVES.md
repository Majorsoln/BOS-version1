# CN-4-016 — Security Primitives

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-007 #5 (runtime + CI enforcement).
> **Glossary:** See `MASTER-GLOSSARY.md`.
> **Depends on:** CN-4-001 (Doctrine — rate-limit/anomaly events are facts, not replay-deterministic logic), CN-4-004 (Command Bus — rate limits applied at bus boundary), CN-4-006 (Isolation — scope guards), CN-4-007 (Identity — principals are rate-limited/anomaly-flagged), CN-4-008 (Audit Log — rejections and alerts recorded), CN-4-009 (Replay — security layer is not replayed), CN-4-022 (Advisor Framework — advisor scope guard).
> **Boundaries:** Tenant scope guard → CN-4-006; platform scope guard → CN-4-006 §2; advisor scope guard → CN-4-022 §3; engine boundary guard → CN-4-005 §2; authentication (pre-auth) → Term 7/Term 1; anomaly response policy → Term 1; resilience interaction → CN-4-017.

---

## 1. What This Doc Defines

CN-4-016 defines the Kernel's security building blocks beyond isolation (CN-4-006) and identity (CN-4-007): **rate limits, anomaly detection, and scope guards**. These are the defensive mechanisms that protect the system from abuse, attack, and misconfiguration at the authenticated-request layer.

---

## 2. Runtime Layer, Not Replay-Deterministic

Rate limiting and anomaly detection operate as a **runtime/edge layer before the deterministic core**. This is a critical architectural distinction:

| Layer | What it does | Replay behaviour |
|-------|-------------|-----------------|
| **Security layer (this doc)** | Rate-limits commands, detects anomalies, enforces scope guards. Runs at runtime only. | **Not replayed.** Rate-limit counters are ephemeral runtime state, not event-sourced truth. Anomaly detection rules run against live patterns, not historical events. |
| **Deterministic core** (CN-4-001/004/009) | Policy evaluation, event emission, fold logic. | **Replayed.** Same events → same state, always. |
| **Events produced by this layer** | Rejection events (`kernel.command.rejected.v1`) and anomaly alerts (`kernel.anomaly.detected.v1`) are **recorded as facts** in the event store. | **Visible on replay** as historical facts — but the rate-limiter and anomaly detector do not re-run during replay. |

On replay, you see the rejection events and anomaly alerts that were produced at runtime. You do not see the rate-limiter counting or the anomaly detector pattern-matching — those are ephemeral runtime processes.

---

## 3. Authentication Boundary

The Kernel's security primitives operate on **authenticated principals** — the command bus (CN-4-004) sees a valid principal (CN-4-007) on every command. The security layer defined here handles:

| This layer handles | This layer does NOT handle |
|-------------------|--------------------------|
| Authenticated principal exceeds rate limit | Pre-authentication brute-force (login spam, credential stuffing) |
| Authenticated principal triggers a scope mismatch (cross-tenant, platform-read-only) | Authentication failures (wrong password, expired token) |
| Authenticated principal's behaviour pattern is anomalous | Network-layer DDoS |
| Authenticated advisor attempts out-of-scope read | OAuth/SSO/MFA mechanics |

Pre-authentication security (brute-force protection, login rate limiting, credential validation) belongs to the **authentication layer** owned by Term 7 (mechanism) and Term 1 (policy). The Kernel trusts the identity once proven (CN-4-007) — it defends against what authenticated principals do, not how they authenticate.

---

## 4. Rate Limiting

### What It Protects Against

Abuse or misconfiguration that floods the system — too many commands from one authenticated principal, too many API calls from one machine client, too many advisor invocations.

### Rate Limiting at the Kernel Level

| Property | Detail |
|----------|--------|
| **Applied at the command bus boundary** | Rate limits are checked before policy evaluation (CN-4-004). A rate-limited command is rejected before it consumes policy evaluation resources. |
| **Per-principal** | Limits apply per actor (CN-4-007): per human user, per machine client, per system component. One slow cashier does not block another cashier. |
| **Per-tenant aggregate ceiling** | A tenant-level aggregate rate limit exists as a safety ceiling — prevents one tenant from consuming disproportionate system resources. |
| **Counts attempts, not unique commands** | The rate counter increments on every submission attempt (including retries). Legitimate retry with backoff is accommodated by setting limits that allow reasonable retry patterns. |
| **Configurable** | Rate limits are not hard-coded. They are configuration — adjustable per tenant tier, per principal type, per command type. Configuration lives outside the Kernel code. |
| **Rejection is audited** | A rate-limited command produces a rejection event (`kernel.command.rejected.v1`, policy = `security.rate_limit_exceeded`). Visible in the audit trail (CN-4-008). The rejection event is a fact in the store; the counter that produced it is ephemeral. |

---

## 5. Anomaly Detection

### What It Detects

Patterns in authenticated-principal behaviour that indicate something wrong — not necessarily an attack, but activity outside expected bounds.

| Anomaly type | Examples |
|-------------|---------|
| **Volume anomaly** | Sudden spike in commands from one authenticated principal or tenant |
| **Pattern anomaly** | Repeated scope-mismatch rejections, repeated consent rejections from the same actor |
| **Time anomaly** | Activity at unusual hours for a tenant's registered fiscal zone (CN-4-014) |
| **Scope anomaly** | A platform admin accessing an unusual number of tenants in a short period |

### How Anomaly Detection Works

| Property | Detail |
|----------|--------|
| **Reads projections** | Anomaly detection reads from projections (audit log, command history) — it never reads the event store directly. Replay-safe (it does not run during replay). |
| **Advisory, not blocking** | Anomaly detection **alerts** — it does not automatically block actions. Blocking is a rate-limit or policy decision. Anomaly detection informs; humans or policies decide. This avoids false-positive lockouts. |
| **System principal** | Anomaly alerts are emitted by `system:anomaly-detector` (CN-4-007 system principal). Not a human, not an advisor — a Kernel component. Follows the pattern of CN-4-013 §3 B (system principal records observations). |
| **Audited** | Anomaly alerts are recorded as events (`kernel.anomaly.detected.v1`) with: anomaly type, affected actor/tenant, evidence summary, timestamp. |
| **Neutral** | Anomaly rules do not name specific verticals or countries. "Volume spike" is a pattern, not a vertical-specific or jurisdiction-specific check. |

### Rate Limits vs Anomaly Detection

| | Rate limits | Anomaly detection |
|--|-------------|-------------------|
| **Mechanism** | Deterministic threshold (count > limit) | Heuristic pattern matching |
| **Response** | Blocking (reject command) | Advisory (alert) |
| **False positives** | Low (threshold is clear) | Possible (patterns are interpretive) |
| **Runtime state** | Ephemeral counter | Ephemeral pattern state |

---

## 6. Scope Guards

### The Shared Middleware Pattern

Several CN docs define specific scope guards. CN-4-016 defines the **scope guard as a reusable security primitive** — a shared middleware pattern:

| Property | Detail |
|----------|--------|
| **Pattern** | Every scope guard follows the same structure: intercept a request → check the actor's scope against the target → allow or reject. |
| **Non-bypassable** | Scope guards are injected at the Kernel boundary and propagated — like the tenant scope object (CN-4-006). No request can skip a scope check. |
| **Rejection is audited** | A scope guard rejection produces an audit entry with: which guard, which actor, what was attempted, what was denied. |
| **Defence in depth** | Runtime guards catch unexpected paths; CI doctrine checks (CN-4-019: DC-005, DC-006, DC-009, DC-016, DC-019, DC-020) catch design mistakes. Both layers per D-007 #5. |

### Specific Scope Guards (Defined Elsewhere)

| Scope guard | Defined in | What it enforces |
|------------|-----------|-----------------|
| **Tenant scope guard** | CN-4-006 §1 | Tenant A cannot access tenant B's data |
| **Platform scope guard** | CN-4-006 §2 | Platform scope is read-only against tenant data |
| **Advisor scope guard** | CN-4-022 §3 | Advisor cannot read outside granted scope; covers all inputs |
| **Engine boundary guard** | CN-4-005 §2 | No cross-engine direct calls at runtime |

CN-4-016 provides the pattern; each doc above provides the specific guard. All share the same middleware structure, auditability, and non-bypassability.

---

## 7. Three Whys

### Why does this matter?

A multi-tenant system is a target. One tenant's compromised machine client could flood the command bus. A brute-force scope-probing attack produces thousands of scope-mismatch rejections. A curious platform admin could browse tenants' data without justification. Rate limits prevent floods; anomaly detection surfaces suspicious patterns; scope guards prevent unauthorised access. Without these, the system's isolation and integrity guarantees are undermined from the outside.

### Why does it belong in the Kernel?

If security were per-engine, each engine would implement its own rate limits, its own anomaly rules, its own scope checks — inconsistently. One engine's missing rate limit becomes a system-wide vulnerability. The Kernel provides security primitives so every authenticated command, every read, every principal operates under the same defensive layer.

### Why this design?

Rate limits (deterministic, blocking) + anomaly detection (heuristic, advisory) + scope guards (structural, non-bypassable) cover three different threat classes: abuse (volume), attack (pattern), and leakage (scope). Separating blocking (rate limits) from alerting (anomaly detection) avoids false-positive lockouts while still surfacing suspicious activity. The runtime-not-replay distinction keeps the security layer from contaminating determinism. Scope guards as a shared middleware pattern ensure every boundary uses the same enforcement mechanism.

---

## 8. Example — A Compromised Authenticated Machine Client

A tenant's API integration (`machine:erp-sync-042`, authenticated with valid credentials) is compromised. The attacker starts submitting thousands of commands per minute.

1. **Rate limit** triggers: `machine:erp-sync-042` exceeds the per-principal command rate. Commands are rejected with `security.rate_limit_exceeded`. The rejection events are recorded as facts in the store.
2. **Anomaly detection** fires: volume anomaly — this client's command rate is 100x its normal baseline. `kernel.anomaly.detected.v1` is emitted by `system:anomaly-detector` with evidence (command count, time window, baseline comparison).
3. The anomaly alert reaches platform governance (Term 1). A platform admin investigates and disables the compromised credential (authentication layer, Term 7/1).
4. Throughout the flood, the **tenant scope guard** ensured that the compromised client could only target its own tenant's data — no cross-tenant leakage occurred (CN-4-006).

Three layers acted: rate limits slowed the flood; anomaly detection identified it; scope guards contained it. All within the authenticated-principal boundary — pre-auth protection (blocking the attacker's login attempts) belongs to Term 7.

---

## 9. Boundaries

| Topic | Lives in |
|-------|----------|
| Tenant isolation (tenant scope guard) | CN-4-006 |
| Platform scope (platform scope guard) | CN-4-006 §2 |
| Advisor scope enforcement | CN-4-022 §3 |
| Engine isolation (engine boundary guard) | CN-4-005 §2 |
| Identity / principal types | CN-4-007 |
| Command bus (where rate limits are applied) | CN-4-004 |
| Audit log (where rejections and alerts are recorded) | CN-4-008 |
| Doctrine checks (CI enforcement of scope boundaries) | CN-4-019 |
| Authentication mechanisms (pre-auth security) | Term 7 (mechanism) / Term 1 (policy) |
| Anomaly response policy (what happens after an alert) | Term 1 (governance) |
| Resilience mode interaction with rate limits | CN-4-017 |

---

## 10. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Rate limit configuration mechanism (how limits are set and adjusted) | Architect phase | Concept says "configurable, not hard-coded"; Architect designs config |
| Rate limit defaults per principal type / tenant tier | Term 1 + Architect | Governance decides defaults; Architect implements |
| Anomaly detection rules engine (how rules are defined and updated) | Architect phase | Concept defines anomaly types; Architect designs the engine |
| Anomaly alert routing (who gets notified) | Term 1 + Term 3 | Governance decides routing; Term 3 surfaces alerts |
| Adaptive rate limits (anomaly triggers tighter limits automatically) | Architect phase / Future decision | Concept says anomaly = advisory; adaptive response is a possible enhancement |
| DDoS-level / network-layer protection | Architect phase / infrastructure | Concept-level rate limits are at the command bus; network-layer is infrastructure |
| **Resilience mode interaction** | CN-4-017 | Rate limits may tighten in DEGRADED mode (e.g., lower ceilings to protect limited resources). Open item for CN-4-017 to address. |

---

*— End of CN-4-016 —*
