# CN-4-020 — Extension Points (How to Add a New Engine)

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-007 #11 (registration API concept), D-007 #5 (runtime + CI enforcement), D-007 #9 (resilience modes), D-002B (Developer-AI pipeline).
> **CTRs:** CTR-018 (us → Term 6, registration contract — pending). CTR-022 (us → Term 1, activation & tenant-availability governance — new, see §5).
> **Glossary:** See `MASTER-GLOSSARY.md`.
> **Depends on:** CN-4-001 (Doctrine), CN-4-002 (Event Store Contract), CN-4-005 (Engine Contract Model), CN-4-006 (Isolation), CN-4-007 (Identity), CN-4-010 (Projection Framework), CN-4-017 (Resilience Modes), CN-4-019 (Doctrine Enforcement), CN-4-023 (Kernel Boundary).
> **Boundaries:** What an engine IS → CN-4-005; event shape → CN-4-002; tenant isolation → CN-4-006; doctrine tests → CN-4-019; vertical-specific engine design → Term 6 (CTR-018); activation governance → Term 1 (CTR-022).

---

## 1. What Extension Points Are

CN-4-005 defines what an engine IS (manifest, isolation rules). This doc defines how a new engine **plugs into the Kernel without touching it** — the registration process, compatibility checks, and the contract Term 6 depends on.

The Kernel grows by adding engines, never by modifying itself. This is the extensibility promise from Section 1 (Mission).

---

## 2. Registration Flow

A new engine joins BOS through two phases: a **build-time gate** (CI) followed by **runtime registration** (compatibility → wiring → activation).

### Phase A — Build-Time Gate (CI)

Before an engine can be registered, its code passes the doctrine enforcement gate (CN-4-019 / CN-4-023):

| Check | What it validates |
|-------|-------------------|
| **No cross-engine imports** | Static analysis confirms the engine does not import from another engine's internal modules. |
| **No `datetime.now()`** | All time comes from the clock protocol (CN-4-014). |
| **Primitives used correctly** | Primitives are used through their defined legal operations; no mutation of frozen dataclasses. |
| **Event schemas conform** | Declared event schemas conform to CN-4-002 envelope rules. |
| **Doctrine invariants pass** | All Section 8 criteria that apply to engine code are green. |

For engines proposed through the Developer-AI pipeline (D-002B / CN-4-023), human review is required in addition to CI — the build-time gate is human + test, never test alone.

**If the build-time gate fails, registration cannot proceed.**

### Phase B — Runtime Registration

Once the build-time gate passes, the engine is registered with the Kernel:

| Step | What happens | Who acts |
|------|-------------|---------|
| **1. Manifest submission** | The engine submits its manifest (CN-4-005: engine_id, emits, subscribes_to, requires, commands, primitives_used, scope_policy, capabilities). | Engine developer / deployment pipeline |
| **2. Compatibility check** | The Kernel validates the manifest against the current system state (see §3). | Kernel (automated) |
| **3. Subscription wiring** | The event bus registers the new engine's subscriptions. Emitting engines are NOT notified or modified. | Kernel (automated) |
| **4. Activation** | The engine becomes live — it receives subscribed events and may emit declared events. Activation is **atomic**: fully wired and active, or not at all. If any wiring step fails, the registration rolls back — no orphaned subscriptions, no partial state. | Kernel (automated after checks pass) |

Registration is a **platform-scoped** operation (CN-4-006): it is performed by a platform actor (CN-4-007), recorded in the platform audit trail (CN-4-008), and visible to platform governance.

---

## 3. Compatibility Checks

| Check | What it validates | On failure |
|-------|-------------------|-----------|
| **Unique engine_id** | No active engine uses this identifier. | Rejected. |
| **Event type ownership** | The new engine's `emits` list does not declare an event type already actively emitted by another engine. One emitter per event type at a time; ownership can be transferred during sunset (the old emitter deprecates, the new one takes over — ties to CN-4-005 deprecation). | Rejected — resolve the conflict or coordinate sunset. |
| **Required events exist** | Every event type in `requires` is emitted by at least one active registered engine. | Rejected — a mandatory dependency is unmet. |
| **Primitive availability** | Every primitive in `primitives_used` exists in the Kernel's primitive catalog (CN-4-011). | Rejected — unknown primitive. |
| **Schema compatibility** | Event schemas declared in `emits` conform to CN-4-002 envelope rules. Subscribed event schemas match the versions the new engine expects. | Rejected — schema mismatch. |
| **Scope policy validity** | The declared scope_policy is a recognised scope type (tenant, branch, business). | Rejected — unknown scope policy. |

### What Registration Does NOT Do

- It does **not** modify any existing engine's manifest, code, or subscriptions.
- It does **not** require human approval from existing engine teams (decoupling).
- It does **not** grant the new engine any special privileges — the engine operates under the same isolation rules as every other engine.

---

## 4. Registry as Event-Sourced

The engine registry follows the Kernel's own doctrine (CN-4-001, Law 1). Registration and deregistration are **events**:

| Event type | When emitted |
|-----------|-------------|
| `kernel.engine.registered.v1` | A new engine passes all checks and is activated |
| `kernel.engine.updated.v1` | An existing engine's manifest is re-submitted with changes |
| `kernel.engine.deactivated.v1` | An engine is deactivated (sunset) |

The registry is a **projection** (CN-4-010) rebuilt from these events. This means:

- The registry is **replayable** — the history of which engines were added, updated, and removed is fully reconstructable.
- The registry is **auditable** — every registration action has an actor, timestamp, and full event trail.
- The registry is **deterministic** — replay produces the same registry state from the same events.

This replaces the need for a separate "registry storage mechanism" — the mechanism is the event store itself.

---

## 5. Manifest Update / Re-Registration

An engine evolves over time — it adds new event types, subscribes to new events, bumps its contract version.

| Step | What happens |
|------|-------------|
| **1. Version bump** | The engine bumps its manifest `version` (CN-4-005 contract version). |
| **2. Re-submission** | The updated manifest is submitted to the Kernel. |
| **3. Compatibility re-check** | All compatibility checks (§3) run against the updated manifest and current system state. |
| **4. Additive wiring** | New emits and subscriptions are wired. Existing subscriptions remain. Changes are additive (CN-4-002 additive-only principle). |
| **5. Registry event** | `kernel.engine.updated.v1` is emitted with the old and new manifest versions. |

Re-registration follows the same atomic guarantee as initial registration: fully updated or not at all.

---

## 6. Deregistration / Sunset

An engine may be deactivated when it is no longer needed.

| Step | What happens |
|------|-------------|
| **1. Deprecation** | The engine's manifest marks its emitted events as deprecated (CN-4-005 deprecation marker). Subscribers are warned via doctrine checks. |
| **2. Subscription drain** | Subscribers migrate to alternative event types or remove their subscriptions. |
| **3. Deactivation** | The engine stops receiving events and emitting new events. `kernel.engine.deactivated.v1` is emitted. The registration becomes inactive. |
| **4. Historical events preserved** | All events the engine ever emitted remain in the store (CN-4-001 — no deletion). Replay, audit, and projections continue to work for historical data. |

Deregistration **never deletes events**. The engine goes silent; its history remains forever.

**Tenant notification** of engine deactivation is NOT a Foundation responsibility — it is a hand-off to Term 1 (governance: which tenants are affected) and Term 3 (UX: how tenants are informed).

---

## 7. Resilience-Mode Interaction

Registration and deregistration are **NORMAL-mode operations** (CN-4-017, D-007 #9):

| Mode | Registration/deregistration |
|------|---------------------------|
| **NORMAL** | Allowed — full registration flow. |
| **DEGRADED** | Gated — requires human approval to proceed (safety measure during instability). |
| **READ_ONLY** | Blocked — no state-changing operations, including registration. |

---

## 8. Capability Declaration

The manifest's `capabilities` field (CN-4-005) is **advertise-only**: an engine declares what it can do so the system can discover it (e.g., "this engine provides checkout," "this engine provides inventory tracking").

Capabilities are for **discovery and cataloguing**, not for dependency resolution. Dependencies remain **event-type-based** (`requires` and `subscribes_to`). No engine depends on another engine's capability declaration — only on the event types it emits. This enforces D-007 #11 without reintroducing coupling through a second channel.

---

## 9. CTR-022 — Activation & Tenant-Availability Governance

### CTR-022 — Registered-engine activation & tenant-availability governance
- **From Term:** 4
- **To Term(s):** 1
- **Decision / Topic:** D-007 #11 / Extension Points
- **Boundary Object:** —
- **What is needed:** Platform governance for: (a) whether a newly registered engine activates automatically after checks pass or requires platform admin approval; (b) how a registered engine is made available to tenants (always-on vs catalog item per subscription plan).
- **Why:** Foundation automates registration; Term 1 governs what tenants see and pay for.
- **Proposed contract:** Foundation registers and activates; Term 1 controls tenant-facing availability and may add a human-approval gate before activation.
- **Status:** OPEN
- **Resolution:** —

---

## 10. Three Whys

### Why does this matter?

BOS must absorb verticals not yet imagined — the Mission says "decades of pressure." If adding a new engine requires modifying the Kernel or existing engines, the system's growth is gated by coordination. Every new vertical becomes a negotiation with every existing team. Extension points eliminate this bottleneck: a new engine passes the build-time gate, registers, passes compatibility checks, and is live.

### Why does it belong in the Kernel?

The registration API is a Kernel service because the Kernel owns the engine registry, the event bus, the isolation rules, and the doctrine checks. If registration were ad-hoc (each engine figuring out how to join), there would be no guarantee of isolation, compatibility, or auditability for the new engine.

### Why this design?

A build-gate → manifest-in → compatibility-check → atomic-wire pattern mirrors how well-designed plugin systems work: prove you're safe (CI), declare what you do (manifest), prove you're compatible (checks), get wired in (atomic activation). The alternative — manual integration, shared configuration, human coordination — does not scale and introduces human error.

---

## 11. Example — Adding a Logistics Engine

*Vertical names appear only as illustrations of extensibility — the contract does not know which verticals exist.*

In a future year, BOS adds a logistics engine. It tracks shipments, emits `logistics.shipment.dispatched.v1`, subscribes to `procurement.order.fulfilled.v1`, and uses the Workflow and Obligation primitives.

**Build-time gate (CI):**
- No cross-engine imports ✓
- No `datetime.now()` ✓
- Workflow and Obligation used through legal operations only ✓
- Event schema conforms to CN-4-002 ✓
- All doctrine invariants pass ✓

**Runtime registration:**
1. Manifest submitted. Compatibility checks:
   - `logistics` is a unique engine_id ✓
   - `logistics.shipment.dispatched.v1` is not emitted by any other active engine ✓
   - `procurement.order.fulfilled.v1` exists (emitted by the procurement engine) ✓
   - Workflow and Obligation are in the primitive catalog ✓
   - Schema compatibility ✓
2. Checks pass. Subscription wired atomically.
3. `kernel.engine.registered.v1` emitted — actor: platform admin, timestamp, full manifest recorded.
4. Logistics is live. It begins receiving procurement fulfillment events.

**The procurement engine is unchanged.** It does not know logistics exists. Its manifest, code, and tests are untouched. No Kernel code changed. The registry event records the full history of this addition — replayable, auditable, deterministic.

---

## 12. Boundaries

| Topic | Lives in |
|-------|----------|
| What an engine IS (manifest structure, isolation rules) | CN-4-005 (Engine Contract Model) |
| Event shape / envelope | CN-4-002 (Event Store Contract) |
| Tenant isolation | CN-4-006 (Multi-Tenant Isolation) |
| Identity (who performs registration) | CN-4-007 (Identity Primitive) |
| Audit trail for registration | CN-4-008 (Audit Log Primitive) |
| Projection framework (registry as projection) | CN-4-010 (Projection Framework) |
| Resilience modes (registration gating) | CN-4-017 (Resilience Modes) |
| Doctrine enforcement tests (build-time gate) | CN-4-019 (Doctrine Enforcement Checks) |
| Developer-AI pipeline (human review gate) | CN-4-023 (Kernel Boundary) |
| Vertical-specific engine design | Term 6 (CTR-018) |
| Activation governance and tenant availability | Term 1 (CTR-022) |
| Tenant notification of engine changes | Term 1 (governance) / Term 3 (UX) |

---

## 13. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Global event type registry (queryable catalog of all event types across all engines) | Architect phase | Conceptually emerges from aggregating all manifests; Architect decides the query mechanism. |
| Versioning of the registration API itself | Architect phase | The registration contract may evolve; versioning is Architect's concern. |
| Circular subscription detection (engine A → B → A) | CN-4-019 / Architect | Manifest analysis builds a dependency graph; doctrine check flags cycles. |

---

*— End of CN-4-020 —*
