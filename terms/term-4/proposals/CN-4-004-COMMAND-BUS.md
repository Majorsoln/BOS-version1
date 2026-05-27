# CN-4-004 — Command Bus & Policy Engine

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-007 #8 (sandboxed compliance evaluator), D-009 (freeze doctrine — pack version at command-processing time).
> **Glossary:** See `MASTER-GLOSSARY.md`.
> **Depends on:** CN-4-001 (Doctrine), CN-4-002 (Event Store Contract), CN-4-003 (Hash Chain), CN-4-005 (Engine Contract Model), CN-4-006 (Isolation), CN-4-007 (Identity), CN-4-013 (AI Guardrails — advisor cannot submit), CN-4-015 (Compliance DSL — compliance policies), CN-4-017 (Resilience Modes — write-mode enforcement).
> **Boundaries:** Event shape → CN-4-002; hash chain → CN-4-003; engine manifest → CN-4-005; tenant scope → CN-4-006; identity → CN-4-007; AI guardrails → CN-4-013; compliance evaluator → CN-4-015; resilience modes → CN-4-017; rate limiting → CN-4-016; engine business rules → Terms 5/6.

---

## 1. What This Doc Defines

The command bus is the **sole entry point for state changes** in BOS. Every command — whether from a human, a system component, or a machine client — passes through the bus. The bus evaluates the command against policies, and either emits events (accepted) or emits a rejection event (denied). No state change happens outside the bus.

---

## 2. The Command Flow

| Step | What happens | Actor |
|------|-------------|-------|
| **1. Command submitted** | A principal submits a command through the bus. The command names the action requested (Charter §8.1: `<engine>.<noun>.<action>.request`). | Any principal except advisor (Law 3, CN-4-013 §2) |
| **2. Envelope validation** | Bus verifies: actor has valid principal + scope (CN-4-007), tenant_scope present (CN-4-006), command type is declared in the target engine's manifest (CN-4-005). | Kernel (automated) |
| **3. Policy evaluation** | The bus runs the command against all applicable policies (§3). Policies are evaluated in order: kernel/doctrine first, then engine, then compliance. Fail-fast: if a kernel policy rejects, later policies are not evaluated. | Kernel (automated) |
| **4a. Accept → event emission** | All policies pass. The bus emits the resulting event(s) into the event store (CN-4-002). Events are hash-chained (CN-4-003). All events from one accepted command are written **atomically** — all or nothing, no partial state. Each emitted event carries the command's `correlation_id` and the appropriate `causation_id` (CN-4-002). | Kernel |
| **4b. Reject → rejection event** | A policy fails. The bus emits a `kernel.command.rejected.v1` event (§4). No business event is emitted. The rejection event carries the command's `correlation_id`. | Kernel |

### Resilience-Mode Gate

Before policy evaluation, the bus checks the current resilience mode (CN-4-017):

| Mode | Bus behaviour |
|------|---------------|
| **NORMAL** | Full flow — all commands accepted for evaluation. |
| **DEGRADED** | State-changing commands are gated per CN-4-017's degraded-mode rules (some commands may be deferred or restricted). |
| **READ_ONLY** | All state-changing commands are **rejected** by a kernel-level mode-gate policy. Only read operations proceed. |

This is where CN-4-017's write-mode enforcement is physically applied.

### Commands That Never Enter the Bus

- **Advisor suggestions:** Advisors produce draft commands (CN-4-022 §5) but cannot submit them. A draft becomes a command only when a human submits it — the human is the acting principal.
- **Direct engine-to-engine calls:** Engines communicate via events, never via commands to each other's bus endpoints (CN-4-005, Law 2).

### System Principals and the Bus

System principals (`system:scheduler`, `system:replay-engine`, etc., CN-4-007 §4) submit commands through the bus and are subject to **all policies** — kernel, doctrine, engine, and compliance. System principals are **not privileged over policy** and are **not a backdoor**. A scheduled month-end trigger submits a command; the bus evaluates it; the command is accepted or rejected on the same basis as any human-submitted command.

---

## 3. Policy Engine

### What a Policy Is

A policy is a **named, pure evaluator** that inspects a command and returns accept or reject. Every rejection names the policy that caused it — rejections are always policy-traced (Section 9 directive).

Policies are:
- **Side-effect-free:** A policy reads state but never modifies it. It produces a decision, nothing else.
- **Deterministic:** Given the same command + state + pack version, a policy always returns the same result. This ensures replay and audit consistency.

A policy with side effects would break replay (re-evaluating a command during replay could produce different results) and audit (the evaluation cannot be independently verified).

### Policy Categories

| Category | Examples | Enforced by | Overridable |
|----------|---------|-------------|------------|
| **Kernel policies** | Tenant scope match, advisor cannot submit, command type declared in manifest, resilience-mode gate | Kernel | Never |
| **Doctrine policies** | Consent required for outreach (D-008), platform scope is read-only for writes (CN-4-006) | Kernel | Never |
| **Engine policies** | Business rules specific to an engine (e.g., "cannot complete sale with zero items") | Engine (declared in manifest, run by bus) | By engine update only |
| **Compliance policies** | Rules from compliance packs applied to commands (e.g., "invoice requires tax treatment") | Compliance DSL sandboxed evaluator (CN-4-015, D-007 #8) | By pack update only |

### Policy Ordering and Fail-Fast

All kernel and doctrine policies run **before** engine and compliance policies. If a kernel or doctrine policy rejects, engine and compliance policies are never evaluated. This prevents engine logic from overriding Kernel guarantees.

### Compliance Policy and Freeze Doctrine

Compliance policies are evaluated using the **compliance-pack version active at the time of command processing** (D-009). The resulting events record this `pack_version_ref` (CN-4-002). This links the bus to the freeze doctrine: the pack version at processing time is the version under which the event is interpreted forever.

---

## 4. Rejection Events

Every rejection produces an event — rejections are not silent. The rejection event is stored in the event store (append-only, hash-chained, auditable).

### Rejection Event Shape

| Field | Description |
|-------|-------------|
| **event_type** | `kernel.command.rejected.v1` |
| **original_command_ref** | Reference to the command that was rejected |
| **rejected_by_policy** | The `policy_name` that caused the rejection |
| **rejection_reason** | Human-readable explanation |
| **actor** | The principal who attempted the command |
| **tenant_scope** | Which tenant the command targeted |
| **correlation_id** | Links to the original command's correlation_id (lineage) |
| **timestamp** | Clock protocol (CN-4-014) |

### Cross-Tenant Rejection Privacy

When a command targets a tenant the actor has no access to, the rejection must be recorded carefully: the rejection event confirms the attempt was denied but does **not** reveal whether the target tenant exists. The rejection reason is generic ("scope mismatch") rather than specific ("tenant X exists but you lack access"). This prevents information leakage about tenant existence.

### Why Rejections Are Events

- **Auditability:** A tenant or auditor can see every attempted action, not just successful ones.
- **Security analysis:** Repeated rejections from the same actor may indicate an attack or misconfiguration.
- **Debugging:** Engine developers can see why their commands were rejected and which policy stopped them.
- **Lineage:** The `correlation_id` links the rejection to the original command, preserving the full request→outcome trace.

---

## 5. Idempotency

The command bus supports **idempotent command submission**. If the same command (identified by `correlation_id`) is submitted twice:

- If the first submission was accepted and the event(s) already emitted, the bus returns success without emitting duplicates (write-once guarantee, CN-4-002 §4).
- If the first submission was rejected, the bus re-evaluates (policies or state may have changed since the first attempt).

This is important for retry scenarios (network failures, DEGRADED mode recovery).

---

## 6. Three Whys

### Why does this matter?

Without a command bus, state changes happen through direct writes scattered across engines. There is no single point where policy is evaluated, no audit trail of attempts, no consistent rejection mechanism. A developer forgets a permission check, and an unauthorised action goes through silently. The bus is the single chokepoint through which every state change must pass — and where every doctrine guarantee is enforced.

### Why does it belong in the Kernel?

The bus is the Kernel's gatekeeper. Every state change in BOS passes through it. If the bus were per-engine, each engine would enforce its own policies — inconsistently. Kernel policies (isolation, advisory-only, consent, resilience-mode) would be bypassable. The bus is where "the Laws become mechanical" (Section 1).

### Why this design?

A centralised bus with named pure-evaluator policies and rejection events gives: (a) a single enforcement point for all doctrine, (b) a traceable rejection trail with full lineage, (c) a clear separation between kernel/doctrine policies (non-negotiable) and engine/compliance policies (configurable), (d) deterministic replay of command evaluation (policies are pure, side-effect-free). The alternative — per-endpoint or per-engine policy — scatters enforcement and makes it impossible to prove that every command was evaluated consistently.

---

## 7. Example — Consent Rejection for SMS Receipt

A cashier tries to send a receipt via SMS without customer consent.

1. Cashier submits `retail.sale.complete.request` with `receipt_delivery=sms`.
2. **Resilience-mode gate:** NORMAL mode — proceed.
3. **Envelope validation:** Actor is `human:cashier-c`, tenant scope = `mama-amina-duka`, command type is in Retail engine manifest. ✓
4. **Policy evaluation (kernel/doctrine first):**
   - `isolation.tenant_scope_match` — cashier's scope matches target tenant. ✓
   - `consent.outreach_requires_consent` — the customer has no SMS consent record (CN-4-011 Consent primitive). **REJECT.**
5. `kernel.command.rejected.v1` is emitted:
   - rejected_by_policy: `consent.outreach_requires_consent`
   - rejection_reason: "Customer has no SMS consent; cannot send receipt via SMS"
   - actor: `human:cashier-c`
   - correlation_id: (links to the original command)
6. The cashier sees the rejection and offers to print the receipt instead.

Engine policies (e.g., "sale must have at least one item") were never reached — the doctrine-level consent policy rejected first (fail-fast). The rejection is recorded, auditable, and traceable.

---

## 8. Boundaries

| Topic | Lives in |
|-------|----------|
| Event shape / envelope (what the bus produces) | CN-4-002 |
| Hash chain (linking emitted events) | CN-4-003 |
| Engine manifest (which commands/events are declared) | CN-4-005 |
| Tenant isolation (scope validation) | CN-4-006 |
| Identity / principal types (who submits) | CN-4-007 |
| AI guardrails (advisor cannot submit) | CN-4-013 |
| Compliance DSL evaluator (compliance policies, sandboxed) | CN-4-015 |
| Resilience modes (write-mode rules the bus enforces) | CN-4-017 |
| Rate limiting on command submission | CN-4-016 (Security Primitives) |
| Specific engine business rules (engine policies) | Terms 5/6 |

---

## 9. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Command schema registry (global catalog of all command types) | CN-4-005 + Architect | Commands are declared in engine manifests; global catalog = Architect |
| Async vs sync command evaluation (does the submitter wait?) | Architect phase | Concept says "command → accept/reject"; timing is Architect's |
| Policy composition (all must pass = AND; is OR ever needed?) | Architect phase | Concept says all applicable policies run; Architect decides composition |
| Rate limiting on command submission | CN-4-016 | Rate limits are a security concern, applied at the bus boundary |
| Compliance policy hot-reload (pack update → policy update without restart) | CN-4-015 + Architect | Pack versioning (D-009) governs; hot-reload is Architect's |

---

*— End of CN-4-004 —*
