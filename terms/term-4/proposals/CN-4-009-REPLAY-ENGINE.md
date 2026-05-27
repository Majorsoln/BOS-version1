# CN-4-009 — Replay Engine

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-009 (freeze doctrine — events interpreted under historical pack versions).
> **Glossary:** See `MASTER-GLOSSARY.md` — Replay, Fold, Projection.
> **Depends on:** CN-4-001 (Doctrine — events are truth, state is derived), CN-4-002 (Event Store Contract — global_position, versioning), CN-4-003 (Hash Chain — integrity verification during replay), CN-4-010 (Projection Framework — rebuild mechanics), CN-4-011 (Primitive Catalog — per-primitive fold logic), CN-4-014 (Time Authority — deterministic time).
> **References:** CN-4-017 (Resilience — replay availability per mode), CN-4-018 (Snapshots — checkpoint storage, non-truth).
> **Boundaries:** Projection rebuild mechanics → CN-4-010; hash chain verification → CN-4-003; per-primitive fold → CN-4-011; time protocol → CN-4-014; snapshots → CN-4-018; resilience modes → CN-4-017.

---

## 1. What This Doc Defines

CN-4-001 established the doctrine: events are truth, state is derived. CN-4-010 defined projections: the read-models replay builds. This doc defines the **replay engine** itself: how state is rebuilt from events, what scoping options exist, how determinism is guaranteed, and how edge cases are handled.

Replay is the mechanical proof of CN-4-001 Assertion 6: "state at any historical point is reconstructible."

---

## 2. Replay Types

| Replay type | What it does | Used for |
|-------------|-------------|---------|
| **Full replay** | Processes all events from genesis (or checkpoint) to produce current state. | Definitive reconstruction; system-wide verification. |
| **Scoped replay** | Processes events for one tenant, one aggregate, or one stream. | Tenant export verification (CN-4-006 §3), per-tenant projection rebuild. |
| **Point-in-time replay** | Processes events up to a specific global_position. | "What was the state at moment X?" — audit, dispute resolution, historical reporting. |
| **Projection rebuild** | A specific form of full/scoped replay that rebuilds one projection (CN-4-010 §8). | Handler fix, corruption recovery, new projection deployment. |

---

## 3. Determinism Guarantee

Replay must produce **identical state** from identical events — regardless of when replay runs, on which server, or in which environment. This is the property that makes replay legally defensible.

### Five Requirements

| Requirement | How it holds |
|-------------|-------------|
| **Same events** | The event store is append-only (CN-4-001/002). Events don't change. |
| **Same order** | Events are processed in global_position order (CN-4-010 §6). Order is permanent (CN-4-002). |
| **Same rules (data)** | Freeze doctrine (D-009): events are interpreted under the compliance-pack version recorded at emission. Replay uses historical pack versions, not current ones. |
| **Same rules (code)** | Fold/handler code remains semantically stable through **additive versioning** (CN-4-002 §3). If fold logic must change, a new event-type version is introduced with a version-aware handler (CN-4-010 §7). Replay of old events uses the handler matching the event's version; replay of new events uses the new handler. Determinism holds through version-aware handlers, not through "code freeze." |
| **Same time** | No `datetime.now()` in fold/handler logic (CN-4-014). All time comes from event timestamps or the clock protocol. |

If any of these is violated, replay produces different state — and the system's legal defensibility is broken. Doctrine checks DC-004, DC-017, DC-018 (CN-4-019) verify these properties.

---

## 4. Point-in-Time Cutoff

The canonical cutoff for point-in-time replay is **global_position** — not timestamp.

A request expressed as a timestamp ("state at 2026-03-31 23:59:59") is resolved to: "the global_position of the last event at or before that timestamp." This resolution happens once, before replay begins. Replay then processes events up to that global_position.

This eliminates ambiguity from timestamp ties (multiple events at the same timestamp). global_position is a total order with no ties (CN-4-002 §2).

---

## 5. Replay with Hash-Chain Verification

### Audit/Defensible Replay

When replay is used for legal defensibility, audit, or dispute resolution, it includes (or is accompanied by) **hash-chain verification** (CN-4-003) for the scope being replayed.

The replay output states whether the chain was verified:
- **Chain verified, intact:** The events replayed are proven untampered. The state produced is legally defensible.
- **Chain verification failed at position N:** The events up to N-1 are verified; events at N and after are unverifiable (CN-4-003 §4). The replay output notes this.

### Routine Replay

Replay for projection rebuilds or developer debugging does not require chain verification (it would be too slow for routine operations). Chain verification is required for **defensible** replay — when the output will be used as evidence.

---

## 6. Checkpoints

Full replay from genesis becomes expensive as the event store grows. Checkpoints accelerate replay without compromising determinism.

### What a Checkpoint Is

A checkpoint is a **verified snapshot of projection state at a known global_position**. It is created by:

1. Running replay up to position N.
2. Verifying the hash chain up to N (CN-4-003) — confirming the checkpoint is based on untampered events.
3. Storing the resulting projection state + position N.

### How Checkpoints Accelerate Replay

Instead of replaying from genesis, replay starts from the most recent valid checkpoint and processes only events after that checkpoint's position. The result is identical to a full replay (determinism guarantee) because:
- The checkpoint was produced by a verified replay
- Events after the checkpoint are applied in the same global_position order
- Fold logic is pure and deterministic (version-aware handlers match each event's version)

### Checkpoint Trust and Invalidation

Checkpoints are **non-truth** (like snapshots, CN-4-018). They are performance accelerators, not sources of state.

A checkpoint is **invalidated** if a hash-chain breach (CN-4-003) is detected at or before its position. When a checkpoint is invalid:
- Replay falls back to the previous valid checkpoint
- If no valid checkpoint exists, replay falls back to genesis
- The invalidated checkpoint is marked (not deleted — events are truth, checkpoints are derived)

---

## 7. Selective Replay / Bad-Event Handling (Section 7, O9)

A bug emitted bad events for a period. Charter forbids deletion (CN-4-001 Assertion 3). How does replay produce correct state?

### Resolution: Compensating Events

Bad events are corrected by appending compensating events (CN-4-001 §3.1, D-007 #3). Replay processes **all** events including the bad ones and their compensations. The per-primitive fold (CN-4-011) produces the correct current state because the compensating events neutralise the bad events' effects.

There is no "skip" mechanism. Every event in the stream is processed. Correctness comes from the fold, not from filtering.

### Why Not Selective Skip?

A skip mechanism would mean: "these events are in the store but we pretend they don't exist." This violates immutability (the events exist) and breaks determinism (different skip lists → different state). Compensating events are the only doctrine-compliant correction mechanism.

---

## 8. Replay and Resilience Modes

| Mode | Replay behaviour |
|------|-----------------|
| **NORMAL** | Full replay capabilities available. Projection rebuilds run with stale-but-available + catch-up + switchover (CN-4-010 §8). |
| **DEGRADED** | Replay is available but may be deprioritised. Long-running rebuilds may be deferred. Point-in-time queries still work. |
| **READ_ONLY** | Replay for read purposes (point-in-time queries, export verification) is fully available. |

**Replay never emits events.** Replay is pure state reconstruction — it reads events and produces derived state. It does not write to the event store. In READ_ONLY mode, projection rebuilds that need to persist derived state (writing to the projection store) may be deferred, but this is a "derived write" (persisting a projection), not "emitting events." The distinction matters: replay never creates new facts in the event stream under any mode.

---

## 9. Three Whys

### Why does this matter?

Replay is the proof that BOS's truth is real. A court asks: "prove the state was X at time T." Replay answers: here are the events (verified by hash chain), here are the historical rules (frozen by D-009), here are the version-aware handlers — and here is the result: deterministic, verifiable, reproducible. Without replay, the event store is just an append-only log with no way to prove what it implies.

### Why does it belong in the Kernel?

Replay operates on the event store, uses per-primitive folds, respects tenant isolation, honours the freeze doctrine, and verifies the hash chain. These are all Kernel-level concepts. If replay were per-engine, each engine would implement its own replay with its own (possibly inconsistent) determinism guarantees. The Kernel provides one replay engine so the guarantee is uniform.

### Why this design?

Full/scoped/point-in-time replay with checkpoints and compensating-event handling covers every use case: audit, export, rebuild, dispute resolution, debugging. The compensating-event-only correction model (no skip, no delete) preserves immutability while allowing error recovery. Checkpoints accelerate replay without weakening determinism. Hash-chain verification during defensible replay makes the output legally admissible — not just "we replayed it" but "we replayed it and proved the inputs were untampered."

---

## 10. Example — Mzee Hassan Disputes His March Figures

Mzee Hassan's accountant says: "The March closing figures don't match my hand calculation."

1. BOS runs a **tenant-scoped, point-in-time replay**: tenant = `mzee-hassan-duka`, cutoff = global_position of the last event at or before March 31 23:59:59 (fiscal zone, CN-4-014).
2. **Hash-chain verification** runs for the tenant's chain within the scope. Result: chain intact, no tampering.
3. The replay engine loads the most recent valid checkpoint (say, March 1, position 12,401) and replays events from that point.
4. Events are processed in global_position order. March's compliance-pack version is used for each event (D-009 freeze). Version-aware handlers match each event's type version. Per-primitive folds produce the running state.
5. At the cutoff, replay produces: total sales = TZS 4,230,000, total expenses = TZS 2,870,000, closing inventory = 847 items.
6. The accountant compares with their hand calculation. A discrepancy is found in one sale — a compensating event was recorded on March 28 that the accountant missed.
7. The replay proves: the figures are correct given the events. The compensating event (visible in the stream) explains the discrepancy. The hash chain proves no event was tampered with.

**Replay output:** State at position 14,892 (March 31). Chain verified: intact. Pack version: tz-compliance-2026.03. Deterministic: reproducible.

The dispute is resolved with mathematical proof, not with trust.

---

## 11. Boundaries

| Topic | Lives in |
|-------|----------|
| Event store (source of events) | CN-4-002 |
| Event doctrine (immutability, compensating events) | CN-4-001 |
| Hash chain (integrity verification) | CN-4-003 |
| Projection framework (rebuild mechanics, handler contract) | CN-4-010 |
| Per-primitive fold logic | CN-4-011 |
| Tenant isolation (scoped replay) | CN-4-006 |
| Time authority (deterministic time) | CN-4-014 |
| Freeze doctrine (historical rules) | D-009 |
| Snapshot/checkpoint storage (non-truth) | CN-4-018 |
| Resilience modes (replay availability) | CN-4-017 |

---

## 12. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Checkpoint storage mechanism (format, location) | Architect phase | Concept says "verified snapshot at known position, non-truth"; Architect decides storage |
| Checkpoint creation policy (frequency, triggers) | Architect phase + Term 1 | Frequency is a performance/governance trade-off |
| Replay performance targets (how fast must a full replay run?) | Architect phase | Concept guarantees determinism and correctness; speed is Architect's |
| Parallel replay (multiple scoped replays simultaneously) | Architect phase | Scoped replays are independent; parallelism is Architect's |
| Replay monitoring (progress tracking, ETA for long rebuilds) | Architect phase | Useful for long-running operations; implementation is Architect's |

---

*— End of CN-4-009 —*
