# CN-4-018 — Snapshot Storage (Non-Truth)

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review. Final Foundation concept document (23/23).
> **Governing decisions:** D-007 #10 (non-truth marker type + CI check + runtime guard).
> **Glossary:** See `MASTER-GLOSSARY.md`.
> **Depends on:** CN-4-001 (Doctrine — events are sole truth), CN-4-002 (Event Store Contract — global_position), CN-4-003 (Hash Chain — breach invalidates snapshots), CN-4-006 (Isolation — tenant-scoped), CN-4-009 (Replay — checkpoints as specialised snapshots), CN-4-010 (Projection Framework — projections that snapshots cache), CN-4-017 (Resilience — snapshot behaviour per mode), CN-4-019 (Doctrine Checks — DC-025, DC-026).
> **Boundaries:** Event store → CN-4-002; replay checkpoints → CN-4-009; projection framework → CN-4-010; hash chain → CN-4-003; tenant isolation → CN-4-006; resilience modes → CN-4-017; doctrine checks → CN-4-019.

---

## 1. What This Doc Defines

CN-4-018 defines how performance caches (snapshots) work within an event-sourced system that forbids treating anything but events as truth. Snapshots accelerate reads and replay without undermining the doctrine. This doc specifies the non-truth marker, the snapshot lifecycle, and the boundary between snapshots and event-sourced truth.

---

## 2. What a Snapshot Is

A snapshot is a **cached copy of derived state at a known point** — a frozen projection state or a replay checkpoint. It is a performance optimisation, never a source of truth.

| Property | What it means |
|----------|---------------|
| **Derived** | Built from events via replay or projection. Not stored independently of events. |
| **Non-truth** | If a snapshot conflicts with a replay from events, the snapshot is wrong. Events always win. |
| **Disposable** | Can be deleted and rebuilt without data loss. Events are the authority. |
| **Point-in-time** | A snapshot is valid "as of" a specific global_position (CN-4-002). Captured atomically at one consistent position — no torn reads. |
| **Accelerator** | Exists to make reads faster and replays shorter — not to add new information. |
| **Version-tagged** | Records the handler/projection version (or code version) it was built with (see §5). |

---

## 3. The Non-Truth Marker (D-007 #10)

Every snapshot carries a **non-truth marker** — a structural indicator that this data is derived, not authoritative.

### Three-Layer Enforcement

| Layer | Enforcement |
|-------|-------------|
| **Type-level** | The snapshot data structure carries an explicit marker that is structurally incompatible with truth-bearing types. A developer cannot accidentally pass a snapshot where an event or authoritative projection is expected. |
| **CI doctrine check (DC-025, DC-026)** | DC-025: the marker is present on all snapshots. DC-026: no code path treats a snapshot as source of state. |
| **Runtime guard (where feasible)** | A read path that expects authoritative data rejects snapshot-typed inputs at runtime. Defence-in-depth — CI catches most cases; runtime catches the rest. |

### Marker Durability Through Serialisation/Storage

The non-truth marker **persists through serialisation and storage**. When a snapshot is written to disk, database, or cache and later read back, it is deserialised as a non-truth-marked type — never as a plain authoritative structure. The type-level defence does not evaporate at the storage boundary.

This means: no code path that loads a snapshot from storage can silently receive it as unmarked data. The serialisation format encodes the marker; the deserialisation logic enforces it. The Architect designs the specific serialisation mechanism; the concept requires the marker to survive the round trip.

### Why Three Layers + Storage Durability

A developer under time pressure takes a shortcut: "just read the snapshot from the cache." Without a durable marker, the cached data looks like any other data structure — silently serving stale data as truth. The type-level marker prevents this in code; storage durability prevents it across the serialisation boundary; CI catches design mistakes; runtime catches unexpected paths.

---

## 4. Snapshot Operations Are Cache Operations, Not Events

Snapshot creation, refresh, and disposal are **cache operations** — they do not produce events in the event store. This is a critical distinction:

| Operation type | Produces events? | Examples |
|---------------|-----------------|---------|
| **Event-sourced truth** | Yes — events record facts | Registry (CN-4-020), Decision Journal (CN-4-013), Audit Log (CN-4-008), business events |
| **Snapshot operations** | **No** — cache management, not truth | Snapshot creation, refresh, invalidation, disposal |

Snapshot operations do not bloat the event store with cache churn. A snapshot created, refreshed ten times, and disposed produces zero events. The events that the snapshot derives from are the truth; the snapshot itself is ephemeral cache state.

If snapshot operations need to be monitored or audited (e.g., "when was this snapshot last refreshed?"), this is operational metadata managed outside the event store — infrastructure-level logging, not event-sourced truth.

---

## 5. Snapshot Lifecycle

| Phase | What happens |
|-------|-------------|
| **Creation** | A snapshot is captured by freezing the current state of a projection (CN-4-010) or a replay checkpoint (CN-4-009 §6) at a specific global_position. The capture is **atomic** — state at exactly one position, no torn reads (CN-4-010 deterministic processing order). The snapshot records: the global_position, the handler/projection version it was built with, and the non-truth marker. |
| **Use** | The snapshot accelerates: (a) projection rebuilds (start from snapshot, replay only events after its position — CN-4-010 §8), (b) point-in-time reads (serve the snapshot if freshness is acceptable), (c) replay checkpoints (CN-4-009 §6). |
| **Staleness** | A snapshot becomes stale as new events arrive past its position. Staleness is measurable: snapshot position vs store's current position. |
| **Refresh** | A stale snapshot is refreshed by replaying events from its position to the current position and storing a new snapshot. The old snapshot may be retained or disposed. |
| **Invalidation** | A snapshot is invalidated if: (a) a hash-chain breach (CN-4-003) is detected at or before its position, or (b) the handler/projection version it was built with no longer matches the current handler set (detectable by comparing the recorded version against the active version — not manual). An invalidated snapshot is not used; replay falls back to the previous valid snapshot or genesis. |
| **Disposal** | An invalidated or superseded snapshot is discarded. No data is lost — events are the authority. Disposal is a cache operation, not an event. |

---

## 6. Snapshot vs Projection vs Checkpoint

| Concept | What it is | Source of truth? | Produces events? | Defined in |
|---------|-----------|-----------------|-----------------|-----------|
| **Event** | An immutable fact in the store | **Yes** — the only source | Yes (it IS an event) | CN-4-002 |
| **Projection** | A live, incrementally-updated read-model derived from events | No — derived, rebuildable | No (derived state) | CN-4-010 |
| **Snapshot** | A frozen copy of projection/replay state at a point | No — derived, disposable, non-truth-marked | **No — cache operation** | This doc |
| **Checkpoint** | A verified snapshot used to accelerate replay | No — non-truth, hash-verified at creation | **No — cache operation** | CN-4-009 §6 |

Checkpoints (CN-4-009) are a specialised form of snapshot with additional verification (hash chain validated at creation, invalidated on breach). This doc governs the general snapshot concept; CN-4-009 adds checkpoint-specific rules.

---

## 7. Tenant-Scoped Snapshots

Snapshots respect tenant isolation (CN-4-006). A snapshot for tenant A contains **zero data from tenant B** (DC-008, CN-4-019). Snapshots are created and consumed within a tenant's scope. Platform-scoped snapshots (for aggregate projections) are separate and operate under platform scope rules.

---

## 8. Snapshots and Resilience Modes

| Mode | Snapshot behaviour |
|------|-------------------|
| **NORMAL** | Snapshots created and refreshed normally. |
| **DEGRADED** | Stale snapshots may serve reads when projections are lagging. Freshness is disclosed via the projection's processed_position (CN-4-010 §4). Snapshot refresh may be deprioritised. |
| **READ_ONLY** | Existing snapshots serve reads. No new snapshots created (no derived writes). |

---

## 9. Three Whys

### Why does this matter?

An event store grows forever (CN-4-001 — no deletion). A full replay of years of events to serve a single dashboard query is unacceptable. Snapshots make reads fast without compromising the event-sourced model. But the danger is real: a developer treats a snapshot as truth, serves stale data, and the system's guarantees silently erode. The non-truth marker — durable through storage, enforced by type system, CI, and runtime — prevents this structurally.

### Why does it belong in the Kernel?

If snapshot handling were per-engine, one engine's snapshot could be treated as truth by another engine that doesn't know it's a cache. The Kernel defines one snapshot concept with a non-truth marker so that every component — Foundation and engine — treats snapshots the same way: as disposable accelerators, never as authority.

### Why this design?

A durable marker-type + CI + runtime guard is defence-in-depth against the most likely failure mode: a developer using a snapshot as truth. Making the marker structural (type-level, surviving serialisation) means the defence operates at every boundary — code, storage, and runtime. The lifecycle (create → use → stale → refresh → invalidate → dispose) is simple and maps directly to the event-sourced model. Snapshot operations as cache operations (not events) keeps the truth store clean of cache churn.

---

## 10. Example — Stale Snapshot Refreshed, Invalidated Snapshot Falls Back

**Refresh scenario:**

Mama Amina's inventory projection has a snapshot at global_position 50,000 (handler version v3). The store's current position is 51,200.

1. A projection rebuild is triggered (handler fix, CN-4-010 §8). The rebuild starts from the snapshot at position 50,000 instead of genesis.
2. The rebuild replays 1,200 events (50,001–51,200) through the fixed handlers. Seconds instead of hours.
3. A new snapshot is created at position 51,200 (handler version v4). The old snapshot is superseded and disposed.
4. No events were produced by the snapshot operations — only cache was managed.

**Invalidation scenario:**

Later, a hash-chain breach is detected at position 49,990 (CN-4-003). The snapshot at 50,000 was built from events that include the breached position — it is invalidated (position 50,000 > breach position 49,990).

Also, the handler version recorded on the snapshot (v3) no longer matches the current handler set (v4) — double invalidation (handler mismatch + breach).

The rebuild falls back to the previous valid snapshot (say, position 40,000, handler v4, no breach in its range) or to genesis. The rebuild is slower but correct. Events are the authority; the snapshot was just a shortcut.

---

## 11. Boundaries

| Topic | Lives in |
|-------|----------|
| Event store (the authoritative source snapshots derive from) | CN-4-002 |
| Event doctrine (snapshots are never truth) | CN-4-001 |
| Projection framework (projections that snapshots cache) | CN-4-010 |
| Replay engine (checkpoints as specialised snapshots) | CN-4-009 |
| Hash chain (breach invalidates snapshots) | CN-4-003 |
| Tenant isolation (snapshot scoping) | CN-4-006 |
| Resilience modes (snapshot behaviour per mode) | CN-4-017 |
| Doctrine checks DC-025, DC-026 | CN-4-019 |

---

## 12. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Snapshot storage mechanism (same store as projections, separate cache, in-memory) | Architect phase | Concept says "derived, disposable, non-truth-marked, durable marker through serialisation" |
| Snapshot refresh policy (frequency, triggers — time-based, event-count, on-demand) | Architect phase + Term 1 | Performance/governance trade-off |
| Non-truth marker type design (distinct type, wrapper, serialisation format) | Architect phase | Concept says "structural, type-level, CI-checkable, survives serialisation" |
| Snapshot size management (compression, pruning old snapshots) | Architect phase | Performance concern |
| Snapshot monitoring (staleness tracking, alerting) | Architect phase | Operational — infrastructure-level, not event-sourced |

---

*— End of CN-4-018 —*
