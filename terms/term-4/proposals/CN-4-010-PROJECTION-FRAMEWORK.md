# CN-4-010 — Projection Framework

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-007 #9 (resilience modes), D-007 #10 (snapshots as non-truth).
> **Glossary:** See `MASTER-GLOSSARY.md` — Projection, Replay, Fold.
> **Depends on:** CN-4-001 (Doctrine — projections are derived, never truth), CN-4-002 (Event Store Contract — global_position, versioning), CN-4-005 (Engine Contract — at-least-once delivery), CN-4-006 (Isolation — tenant scoping), CN-4-009 (Replay Engine — rebuild mechanics).
> **References:** CN-4-017 (Resilience — rebuild impact), CN-4-018 (Snapshots — rebuild optimisation), CN-4-022 (Advisor Framework — freshness_indicator source).
> **Boundaries:** Replay mechanics → CN-4-009; snapshot storage → CN-4-018; tenant isolation → CN-4-006; resilience modes → CN-4-017; engine-specific projections → Terms 5/6.

---

## 1. What This Doc Defines

CN-4-010 defines how read-models are built from events. A projection is the Kernel's mechanism for turning the event stream into queryable state. Multiple Foundation components are already projections: the audit log (CN-4-008), the Decision Journal (CN-4-013), the engine registry (CN-4-020). This doc formalises the framework they all share.

---

## 2. What a Projection Is

A projection is a **derived read-model** rebuilt from events. It is never a source of truth (CN-4-001 Assertion 2).

| Property | What it means |
|----------|---------------|
| **Derived** | Built by processing events through handlers — not stored independently |
| **Rebuildable** | Can be reconstructed from scratch by replaying events |
| **Deterministic** | Same events, processed in the same order → same projection state, always |
| **Disposable** | Can be dropped and rebuilt without data loss (events are the truth) |
| **Queryable** | Optimised for reads — structured for the queries it serves |
| **Position-aware** | Exposes its current processed position and freshness (see §4) |

---

## 3. Projection Lifecycle

| Phase | What happens |
|-------|-------------|
| **Registration** | A projection is registered with the framework: its name, which event types it subscribes to, and its handler per event type/version. |
| **Build (initial)** | The framework replays all relevant events through the handlers, in global_position order, to produce the initial state. |
| **Live update** | As new events arrive, the framework routes them to the projection's handler. The projection updates incrementally, advancing its processed position. |
| **Rebuild** | The projection is dropped and rebuilt from events (e.g., after a handler change, a bug fix, or corruption). Stale-but-available projections serve reads during rebuild (§6). |

---

## 4. Position Tracking & Freshness

Every projection exposes:

| Field | Description |
|-------|-------------|
| **processed_position** | The `global_position` (CN-4-002) of the last event this projection has processed |
| **processed_timestamp** | The timestamp of that event (clock protocol, CN-4-014) |

This serves two purposes:

**Freshness for consumers.** Any component reading a projection knows how current the data is. The Advisor Framework's `freshness_indicator` (CN-4-022 §7, N4) is derived from this: if the projection's processed_position is behind the store's latest position, the data is stale. In DEGRADED mode (CN-4-017), consumers are informed that advice or dashboard data may be based on stale projections.

**Idempotency for delivery.** The processed_position acts as a **high-water mark**. When the framework receives an event with `global_position ≤ processed_position`, it is a re-delivery and is skipped. This eliminates the need for individual handlers to implement their own idempotency logic — the framework handles it via position tracking (CN-4-005 §3: at-least-once delivery, idempotent handling expected).

---

## 5. Handler Contract

Each projection registers **one handler per event type per version** it processes.

| Handler property | What it means |
|-----------------|---------------|
| **Side-effect-free against the event store** | A handler reads events and updates the projection. It never writes new events. |
| **Version-aware** | A handler for `sale.completed.v1` and a handler for `sale.completed.v2` can coexist. The framework routes by type + version. (CN-4-002 §3 versioning contract.) |
| **Deterministic** | Same event + same prior projection state → same updated projection state. Required for rebuild consistency. |

Idempotency is handled by the framework's position tracking (§4), not by individual handlers.

---

## 6. Deterministic Processing Order

Events are applied to a projection in **global_position order** (CN-4-002). This is the deterministic processing order that guarantees "same events → same projection state." Without a defined order, two rebuilds of the same projection could produce different results.

For tenant-scoped projections, the framework processes that tenant's events in global_position order (the subset of the global order that belongs to this tenant).

---

## 7. Multi-Version Handling

When an event type bumps a version (CN-4-002 §3), the projection framework handles coexistence:

| Scenario | How it works |
|----------|-------------|
| **Projection has handlers for v1 and v2** | Framework routes each event to the matching handler by version. Both versions processed correctly. |
| **Projection has handler for v1 only, v2 events arrive** | The event **must** be upcasted to v1 shape (CN-4-002 §3 read-time upcasting) and processed through the v1 handler. If upcasting is not defined for this version transition, the projection **fails** — this is a doctrine violation (the projection is incomplete). Silent skipping is never permitted. |
| **Handler upgrade (new handler for v2 added)** | The projection is rebuilt — replaying all events through the updated handler set. Old v1 events go through the v1 handler; new v2 events through the v2 handler. |

An unhandled version is **never silently skipped**. A projection that skips events is an incomplete view of reality — it violates the "derived from events" guarantee (CN-4-001 Assertion 2). The options are upcast or fail; silence is not an option.

---

## 8. Rebuild During Operation (Section 8 #4)

A rebuild must not take the system offline.

### The Rebuild Process

| Step | What happens |
|------|-------------|
| **1. Initiate** | Rebuild is triggered (handler fix, corruption, manual). A new projection instance begins building alongside the old one. |
| **2. Historical replay** | The new instance replays all relevant events from the beginning (or from a snapshot/checkpoint, CN-4-018/CN-4-009) in global_position order. |
| **3. Stale-but-available** | During replay, the **old projection** continues serving reads. Queries return data that may be slightly behind, but the system is operational. |
| **4. Catch-up** | The new instance catches up to the live event tail — processing events from the end of historical replay (position N) through to the current position. There is no gap between rebuild-end and switchover. |
| **5. Atomic switchover** | When the new instance is fully caught up (current), the framework atomically switches reads from the old projection to the new one. |
| **6. Cleanup** | The old projection instance is disposed. |

At no point does a query fail because "projection is rebuilding." The tenant always gets an answer — possibly stale, never absent.

If a rebuild is too long-running or disruptive, resilience mode transitions (CN-4-017) may apply. Snapshots (CN-4-018) and replay checkpoints (CN-4-009) may be used to accelerate the historical replay phase.

---

## 9. Tenant-Scoped Projections

Projections respect tenant isolation (CN-4-006). A projection for tenant A processes **only** events where `tenant_id = A`. No cross-tenant data leaks into a projection — this is enforced by the **framework**, not by individual handlers. Handlers never see events outside their projection's tenant scope.

Platform-scoped projections (e.g., aggregate metrics across tenants) are separate and operate under platform scope rules (CN-4-006 §2).

---

## 10. Known Projections in Foundation

Several Foundation components are already defined as projections:

| Projection | Defined in | Built from |
|-----------|-----------|-----------|
| Audit log (tenant trail) | CN-4-008 | All events scoped to a tenant |
| Audit log (platform trail) | CN-4-008 | All platform-scope events |
| Decision Journal | CN-4-013 | `kernel.advisor.suggestion.recorded.v1`, `kernel.advisor.decision.recorded.v1` |
| Engine registry | CN-4-020 | `kernel.engine.registered/updated/deactivated.v1` |
| Per-primitive state | CN-4-011 | Engine-namespaced events using each primitive's fold |

Engine-specific projections (dashboards, reports, KPIs) are defined by Terms 5/6 using this framework.

---

## 11. Three Whys

### Why does this matter?

Without a projection framework, every engine builds its own read-model mechanism — inconsistent rebuild strategies, no idempotency guarantee, no multi-version handling, no freshness tracking. One engine's projection can't be rebuilt; another's silently drops events. The framework makes projections a first-class Kernel concept with uniform guarantees.

### Why does it belong in the Kernel?

The Kernel itself uses projections (audit log, journal, registry). If the framework were per-engine, the Kernel's own read-models would lack the guarantees it promises to others. The framework is Kernel-level because projections are fundamental to how BOS serves data — every dashboard, every query, every report reads from a projection.

### Why this design?

Handler-per-event-type with position-tracked idempotency, deterministic global_position ordering, and mandatory version handling (upcast or fail, never skip) gives the strongest consistency guarantee possible for derived state. The rebuild-during-operation guarantee (stale-but-available + catch-up + atomic switchover) ensures projections never cause downtime. These are concept-level commitments that every projection — Foundation or engine — inherits.

---

## 12. Example — Rebuilding the Audit Log After a Handler Fix

A bug is found in the audit log's handler for `kernel.command.rejected.v1` — it was not recording the `rejection_reason` field. The handler is fixed.

1. The framework initiates a rebuild of the tenant audit log.
2. A new projection instance begins replaying events from the beginning (or from a snapshot + replay-since, CN-4-018/CN-4-009).
3. The **old (buggy) projection** continues serving reads — tenants see their audit logs, but rejection reasons are missing.
4. The new instance replays all events through the **fixed handler set**. Rejection events now produce audit entries with the reason field populated. Events are processed in global_position order.
5. The new instance catches up to the live event tail — no gap.
6. The framework **atomically switches** reads to the new projection.
7. Tenants now see rejection reasons in their audit log. No downtime occurred. The rebuild is logged as a system action (`system:projection-framework`, CN-4-007).

---

## 13. Boundaries

| Topic | Lives in |
|-------|----------|
| Event store (source of events) | CN-4-002 |
| Event doctrine (projections are derived, never truth) | CN-4-001 |
| Replay engine (replay mechanics used during rebuild) | CN-4-009 |
| Tenant isolation (projection scoping) | CN-4-006 |
| Snapshot storage (cached state for rebuild acceleration, non-truth) | CN-4-018 |
| Resilience modes (rebuild impact on system mode) | CN-4-017 |
| Advisor freshness indicator (derived from processed_position) | CN-4-022 |
| Engine-specific projections (dashboards, reports) | Terms 5/6 |

---

## 14. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Projection storage mechanism (database tables, materialised views, in-memory) | Architect phase | Concept says "queryable, rebuildable"; Architect decides storage |
| Rebuild orchestration (parallel per-tenant, sequential, prioritised) | Architect phase | Concept guarantees stale-but-available + catch-up + atomic switchover; orchestration is Architect's |
| Projection monitoring (detecting stale projections, alerting on lag) | Architect phase | processed_position enables monitoring; implementation is Architect/infrastructure |
| Handler testing patterns (how to verify idempotency and determinism) | Architect phase | Concept requires these properties; Architect designs test patterns |
| Checkpoint/resume during long rebuilds | CN-4-009 + CN-4-018 | Replay engine may provide checkpoints; snapshots may provide starting points |

---

*— End of CN-4-010 —*
