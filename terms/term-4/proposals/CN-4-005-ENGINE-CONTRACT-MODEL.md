# CN-4-005 — Engine Contract Model

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-007 #5 (engine isolation: runtime + CI).
> **Glossary:** See `MASTER-GLOSSARY.md`.
> **Depends on:** CN-4-001 (Doctrine), CN-4-002 (Event Store Contract), CN-4-006 (Isolation).
> **Boundaries:** New-engine onboarding → CN-4-020/CTR-018; event shape → CN-4-002; commands → CN-4-004; tenant scope → CN-4-006; payloads → CN-4-011; doctrine tests → CN-4-019.

---

## 1. Engine Manifest

An engine is a bounded module that owns one business domain. It declares itself to the Kernel through a **manifest** — a structured, machine-readable declaration of what the engine is, what it emits, what it consumes, and what it is allowed to do.

### Manifest Fields

| Field | Description |
|-------|-------------|
| **engine_id** | Unique identifier for this engine |
| **display_name** | Human-readable name |
| **version** | Engine contract version (the interface version, not the code/release version) |
| **emits** | List of event types this engine emits, with schema version. Each entry may carry a **deprecation marker** (deprecated since version X, sunset target) to support graceful event-type sunset. |
| **subscribes_to** | List of event types this engine subscribes to |
| **requires** | The mandatory subset of `subscribes_to` — event types the engine **cannot function without**. `requires` ⊆ `subscribes_to` always. Optional subscriptions (nice-to-have, enrichment) are in `subscribes_to` but not in `requires`. |
| **commands** | List of commands this engine accepts |
| **primitives_used** | Which Foundation primitives this engine builds on (ledger, obligation, item, etc.) |
| **scope_policy** | Whether the engine operates at tenant-level, branch-level, or business-level scope |
| **capabilities** | What this engine offers to the system (declared, not assumed) |

### Design Principles

- The manifest is **declarative** — the engine says what it does, not how.
- The manifest is **the Kernel's view of the engine** — the Kernel reads manifests to validate compatibility, route events, and enforce isolation.
- The manifest **never references another engine by name** in `requires` or `subscribes_to` — it references event types. This preserves decoupling: the engine knows "I need `checkout.settled.v1`" but not "I need the Checkout engine." If the event type is emitted by a different engine tomorrow, the subscription still works.
- **Deprecation** is expressed within the manifest's `emits` list, not as a separate mechanism. This keeps the lifecycle of an event type visible in the declaration. Ties to CN-4-002 schema versioning (additive only; breaking change = new event type).

---

## 2. Isolation Guarantee

Charter Law 2: no engine calls another engine directly. Communication is events only. The Kernel enforces this at two layers.

### Runtime Guards

- The Kernel middleware intercepts any direct call from one engine to another's internal API, function, or module and **rejects it**.
- An engine can only interact with another engine by: (a) emitting events that the other subscribes to, or (b) subscribing to events the other emits.
- The command bus is the sole entry point for state changes — no engine submits commands on behalf of another engine directly.

### CI Doctrine Checks (CN-4-019)

- Static analysis detects cross-engine imports or direct function calls across engine boundaries.
- Any code that imports from another engine's internal modules is blocked before it reaches production.
- The manifest's `emits` and `subscribes_to` are cross-checked: an engine cannot subscribe to an event type no engine declares it emits (orphan subscription warning).

### Why Both Layers

CI catches design mistakes early — developers get feedback before code leaves their machine. Runtime catches unexpected runtime paths that static analysis may miss — defensive safety in production. Neither alone is sufficient; both together make isolation structural rather than aspirational.

---

## 3. Subscription Framework

### How Subscription Works

An engine subscribes to events by declaring them in its manifest (`subscribes_to`). The Kernel's event bus routes matching events to all declared subscribers.

| Principle | What it means |
|-----------|---------------|
| **Emitter does not know subscribers** | When an engine emits an event, it does not know (or care) which other engines subscribe. No coupling exists from emitter to subscriber. |
| **Subscriber does not import from emitter** | The subscriber receives the event through the bus. It never calls the emitter's code, reads the emitter's database, or imports the emitter's modules. |
| **New subscriber = manifest update only** | A new engine subscribes to an existing event type by declaring it in its manifest. No change to the emitting engine, the bus, or the Kernel. |
| **Subscription is by event type, not engine** | The subscriber declares interest in an event type (e.g., `retail.sale.completed.v1`), not "events from the Retail engine." |

### Delivery Guarantees

| Guarantee | Scope |
|-----------|-------|
| **At-least-once delivery** | Every declared subscriber receives every matching event at least once. |
| **Ordered within a stream** | Events for one aggregate arrive in sequence order. |
| **Idempotent handling expected** | Subscribers must handle re-delivery gracefully. The Projection Framework (CN-4-010) provides patterns for idempotent processing. |

Exactly-once semantics are an Architect-phase concern (mechanism). The concept guarantees: no event is silently lost; duplicates are expected and handled.

### Request-Reply via Events

Request-reply over events (engine A emits a request event; engine B emits a response event) is **possible but discouraged**. It reintroduces temporal coupling — engine A waits for engine B's response, creating a hidden dependency on B's availability and latency. The canonical pattern is the **status-event + local-projection** pattern (§4 below). If request-reply is used, it remains event-only (no direct call), but it should be flagged in the manifest's `requires` as a mandatory dependency so the coupling is visible.

---

## 4. The Status-Event Pattern

**Problem (O7):** Engine A needs to know a fact from Engine B — "is the current accounting period closed?" Law 2 says no direct calls. How?

**Solution:** Engine B emits a status event when a relevant state changes. Engine A subscribes and maintains its own local projection.

| Step | What happens |
|------|-------------|
| 1 | Accounting closes March → emits `accounting.period.closed.v1 { period: "2026-03" }` |
| 2 | Engine A (subscribed) receives the event, updates its local projection: `period_status["2026-03"] = closed` |
| 3 | When Engine A needs to check, it reads its own projection — no call to Accounting |

### Rules

- The status event is a **fact about what happened**, not a query response.
- Engine A's projection may be slightly stale (eventual consistency). If Engine A needs to block on the status before proceeding, it checks its projection — if the status hasn't arrived, the operation waits or defers (never calls the emitter).
- Accounting does not know Engine A subscribes. The coupling is one-directional: subscriber depends on the event type, never on the emitter.
- This pattern scales to any number of subscribers. If ten engines need to know the period status, all ten subscribe independently. Accounting's manifest and code remain unchanged.

---

## 5. Malformed-Event Handling

**Problem (O9/O10):** An engine emits an event that violates the contract. How is damage contained?

### Validation at Emission

| Check | When | On failure |
|-------|------|------------|
| **Event type declared in manifest** | Command Bus, before emission | Rejected — the engine is not declared to emit this event type. The command fails; no event enters the store. |
| **Schema validation** | Command Bus, before emission | Rejected — the payload does not match the declared schema for this event type and version. The command fails. |
| **Envelope completeness** | Event Store, at write | Rejected — missing required envelope fields (tenant_id, actor, timestamp, etc., per CN-4-002). |

### Damage Containment

- A malformed event **never enters the store**. Validation happens before write. The stream remains clean.
- The failing engine receives a rejection through the command bus rejection mechanism (CN-4-004). Other engines are unaffected — their subscriptions receive no malformed event because none was written.
- As a backstop, the hash chain (CN-4-003) detects any anomaly in the stream on verification. But the primary defense is pre-write validation.

### Structural Violation vs Business-Logic Error

An event that is schema-valid but logically wrong (e.g., wrong quantity, wrong price) is **not** a malformed event. It is a business error. Correction follows the compensating-event pattern (CN-4-001, D-007 #3). The engine contract model prevents structural violations; it does not prevent business-logic bugs — that is the engine's own responsibility.

---

## 6. Three Whys

### Why does this matter?

Without engine contracts, BOS becomes a monolith. Engines call each other's internal functions, share database tables, break when one changes. Adding a new vertical means modifying every existing engine. This is how systems become unrewritable — the very outcome the Kernel exists to prevent.

### Why does it belong in the Kernel?

Engine isolation is Charter Law 2 — a non-negotiable doctrine. If isolation were engine-self-discipline ("please don't call other engines"), one developer's shortcut would create a coupling that takes months to untangle. The Kernel enforces isolation structurally: the manifest declares what's allowed; runtime + CI guards reject what isn't.

### Why this design?

A manifest-based, event-only contract gives each engine a clear boundary: "here is what I emit, here is what I consume, here is what I can do." The Kernel reads the manifest to validate, route, and enforce. The alternative — shared libraries, direct calls, shared databases — creates invisible dependencies that make the system brittle and impossible to extend safely.

---

## 7. Example — A Future Engine Subscribes Without the Emitter Knowing

*Vertical names appear here only as illustrations of extensibility — the contract does not know which verticals exist.*

In a future year, BOS adds an engine for a new vertical. This new engine needs to know whenever a high-value sale occurs (to suggest coverage to the customer).

1. The new engine's manifest declares: `subscribes_to: [retail.sale.completed.v1]`.
2. The Kernel's event bus begins routing matching events to the new engine.
3. The existing engine that emits `retail.sale.completed.v1` is unchanged — its manifest is unchanged, its code is unchanged, its tests are unchanged.
4. The new engine receives the events, processes them through its own logic, and emits its own events (advisory, per Law 3).

**Zero lines of code changed in the emitting engine.** Zero coordination required between the two engineering teams. The contract held. The Kernel routed. The new engine plugged in.

---

## 8. Boundaries

| Topic | Lives in |
|-------|----------|
| How a NEW engine is registered/onboarded (registration API, compatibility check, onboarding checklist) | CN-4-020 (Extension Points) — CTR-018 boundary |
| Event shape / envelope (the 15 fields) | CN-4-002 (Event Store Contract) |
| How commands are evaluated and events emitted | CN-4-004 (Command Bus & Policy Engine) |
| Tenant isolation scope | CN-4-006 (Multi-Tenant Isolation) |
| Per-primitive payload schemas and fold logic | CN-4-011 (Primitive Catalog) |
| Doctrine enforcement tests (CI guards, orphan-subscription detection) | CN-4-019 (Doctrine Enforcement Checks) |

**CN-4-005 / CN-4-020 boundary:** CN-4-005 defines what an engine IS (the contract, the manifest, the isolation rules). CN-4-020 defines how a new engine is ADDED (the registration process, compatibility check, the onboarding checklist). CTR-018 (pending Term 6) covers the registration API — that is CN-4-020's concern, not this doc's.

---

## 9. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Event type registry: where are all declared event types catalogued across all engines? | CN-4-020 / Architect | The manifest declares per-engine; a global registry emerges from aggregating all manifests. Architect decides the registry mechanism. |
| Schema evolution notification: when a shared event type bumps a version, how are downstream subscribers notified? | CN-4-010 + CN-4-019 | Projection Framework handles multi-version handlers; doctrine checks flag stale subscriptions. |
| Delivery mechanism (message broker, in-process bus, hybrid) | Architect phase | The concept guarantees at-least-once, ordered-within-stream. Mechanism is Architect's. |
| Engine dependency graph: can the Kernel detect circular subscription chains? | CN-4-019 / Architect | Manifest analysis can build a dependency graph; doctrine check flags cycles. |
| Scope policy detail: what do "branch-level" and "business-level" scope mean for engine operations? | CN-4-006 + CN-4-011 | Scope policies interact with tenant isolation and primitive shapes. |

---

*— End of CN-4-005 —*
