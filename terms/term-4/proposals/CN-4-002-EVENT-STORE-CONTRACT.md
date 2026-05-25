# CN-4-002 — Event Store Contract

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-009 (freeze doctrine).
> **Glossary:** See `MASTER-GLOSSARY.md` — Event, Compensating Event, Fold, Projection, Replay.
> **Depends on:** CN-4-001 (Event Sourcing Doctrine).
> **Boundaries:** This doc defines the event shape, ordering, and store contract. Mechanics of hashing → CN-4-003; emission → CN-4-004; replay → CN-4-009; projections → CN-4-010; payloads/fold → CN-4-011; time → CN-4-014; scaling → Architect phase.

---

## 1. Event Envelope

Every event in BOS is wrapped in a universal envelope. The envelope is engine-agnostic — no field names a vertical, country, or integration. The payload is engine-specific; the envelope is the Kernel's territory.

### Required Fields

| Field | Type | Description | Constraint |
|-------|------|-------------|------------|
| **event_id** | Unique identifier | Globally unique, immutable. Identifies this specific event forever. | Generated at emission; write-once. |
| **event_type** | String | `<engine>.<noun>.<verb>.v<n>` — the event's semantic identity. | Charter §8.1, BO-5. |
| **version** | Integer | The schema version number (the `n` in `.v<n>`). Denormalised from event_type for indexing and routing convenience. | Must match the version in event_type. |
| **stream_id** | Identifier | Which aggregate/entity stream this event belongs to. Scopes replay and ordering. | Per-engine, per-aggregate. |
| **sequence** | Monotonic integer | Position within the stream. Strictly ordered; no two events in the same stream share a sequence number. | Assigned at write; permanent. |
| **global_position** | Monotonic integer | Position within the entire store. Total order across all streams. | Assigned by the store at write time; permanent. |
| **tenant_id** | Identifier | Which tenant this event belongs to. The primary isolation key. | CN-4-006. Never null for business events. |
| **actor** | Actor reference | Who caused this event — principal type (human, advisor, system, machine) + identity. | CN-4-007, D-007 #4. |
| **timestamp** | Clock-protocol time | When this event was recorded. From the clock protocol (CN-4-014), never `datetime.now()`. This is **record time** — business/effective dates (e.g., "sale date," "invoice date") live in the payload, not the envelope. | CN-4-014. |
| **pack_version_ref** | Version reference | The compliance-pack version active at emission. Enables freeze doctrine (D-009). Always present in the envelope; value may be null for events with no compliance interpretation (system/infrastructure events). | D-009. |
| **correlation_id** | Identifier | Links this event to the command that caused it. Enables tracing request → event. | CN-4-004. |
| **causation_id** | Identifier | The event_id of the event that triggered this event (if any). Enables causal chains in event-driven cascades. | Null if the event was caused by a command, not another event. |
| **compensates_event_id** | Identifier | The event_id of the event this event compensates. Null for non-compensating events. | Dedicated envelope field — makes compensating events structurally identifiable without parsing engine-specific payloads. D-007 #3. |
| **payload** | Structured data | The engine-specific content. Opaque to the store; meaningful to the engine and its primitives. | CN-4-011 defines per-primitive payload shapes. |
| **metadata** | Registered key-value | Engine-agnostic operational data (e.g., source IP, request origin). Schema-constrained: metadata fields are registered (like event types), not ad-hoc. Not part of business truth — never used in fold logic. | Extensible via registration; prevents "hidden state" leaking into metadata. |

**Total: 16 fields** (14 original + compensates_event_id + metadata registered constraint).

---

## 2. Ordering Guarantees

### Per-Stream Ordering

Within a single stream (one aggregate/entity), events are strictly ordered by `sequence` number. The sequence is monotonically increasing with no gaps within a stream. No two events in the same stream share a sequence number. This is the **primary ordering guarantee** — it determines what happened to this aggregate and in what order.

### Global Ordering

Across all streams in the entire store, every event receives a `global_position` — a monotonically increasing number assigned by the store at write time. This provides a **total order** across the entire store. It is deterministic: replay in global_position order always produces the same result.

### Same-Instant Resolution (O1)

Two commands arriving at the exact same nanosecond are resolved by the ordering mechanism:

- **Different streams** (e.g., two cashiers completing two different sales): Each event lands in its own stream with `sequence=1`. The store assigns different `global_position` values — one is always first. The ordering is deterministic but not semantically meaningful (neither sale is "more important"). Replay honours this order.

- **Same stream** (e.g., two commands targeting the same aggregate): Resolved by **optimistic concurrency** — the writer specifies the expected `sequence` number. If the stream has advanced past that number (another event was written first), the write is rejected and the command must retry or fail. This prevents conflicting events within a single stream.

### Ordering at Scale

How the store assigns global_position (database sequence, logical clock, distributed counter) is an Architect decision. The concept guarantees: a monotonically increasing total order exists, is assigned at write time, is permanent, and is deterministic on replay.

---

## 3. Schema Versioning

Event types evolve over time. `retail.sale.completed.v1` and `retail.sale.completed.v2` coexist in the store forever — events are immutable.

### The Versioning Contract

| Rule | Meaning |
|------|---------|
| **Old events are never migrated.** | A v1 event stored today is still v1 in a decade. The store never rewrites events to a new schema. |
| **New versions are additive.** | v2 may add fields. It never removes or renames fields present in v1. A breaking change requires a new event type (`sale.completed_v2` or `sale.settled`), not a version bump on the existing type. |
| **Consumers handle all live versions.** | A projection that reads `sale.completed` must handle both v1 and v2. The Projection Framework (CN-4-010) manages handler registration per version. |
| **Upcasting is optional and read-time only.** | An engine MAY upcast a v1 event to v2 shape at read time for convenience. The stored event remains v1 forever. Upcasting is a projection/consumer concern, not a store concern. |
| **No hard limit on live versions.** | The store accepts any valid version number. Doctrine enforcement (CN-4-019) may flag excessive version sprawl as a code-health concern, but the store does not reject on version count. |

---

## 4. Append-Only Guarantee

The store enforces immutability at the contract level. These guarantees hold always, with no override mechanism.

| Guarantee | Meaning |
|-----------|---------|
| **No UPDATE** | Once written, an event's fields — envelope and payload — cannot be changed. |
| **No DELETE** | Events cannot be removed from the store. Ever. Under any circumstance. |
| **No reorder** | An event's `sequence` and `global_position` are permanent once assigned. |
| **Write-once** | An `event_id` appears exactly once. Duplicate writes are rejected. |

How the store enforces these guarantees (database triggers, immutable storage, write-ahead logs) is an Architect decision. The concept states: these four guarantees hold, with no mechanism to bypass them at any layer.

---

## 5. Three Whys

### Why does this matter?

Without a defined event shape, every engine invents its own structure. Replay, projections, audit, isolation, and hash chaining all break because there is no common format to process. The contract is what makes the event stream machine-readable, replayable, and legally trustworthy across every engine.

### Why does it belong in the Kernel?

The event envelope is the Kernel's data contract — the surface every engine writes to and every Kernel service reads from. If the shape were per-engine, the Kernel could not guarantee replay, integrity, or cross-engine event subscription. A universal envelope is the mechanical foundation of engine isolation (Law 2): engines publish events in a common shape; subscribers consume without knowing the publisher.

### Why this design?

An envelope-plus-payload pattern separates universal concerns (identity, ordering, isolation, tracing, compliance-pack reference) from engine-specific content. The envelope belongs to the Kernel; the payload belongs to the engine. This keeps the Kernel engine-agnostic while still processing, ordering, hashing, and replaying every event.

---

## 6. Example — Duka la Mama Amina

**Two cashiers, same instant:**

Mama Amina's duka in Mwanza has two cashiers working the Friday rush. Cashier A completes a sale of 3 items (TZS 45,000). Cashier B completes a sale of 1 item (TZS 12,000). Both press "Complete" at 14:32:07.

The store assigns:

| | Cashier A | Cashier B |
|--|-----------|-----------|
| stream_id | sale-0042 | sale-0043 |
| sequence | 1 | 1 |
| global_position | 98301 | 98302 |
| actor | human:cashier-a | human:cashier-b |
| tenant_id | mama-amina-duka | mama-amina-duka |
| pack_version_ref | tz-compliance-2026.03 | tz-compliance-2026.03 |
| compensates_event_id | null | null |

Different streams, different global positions. No conflict. Replay always processes A's event before B's (global position order). Both are immutable, both carry the actor, both reference the same compliance-pack version.

Later, Cashier A realises the price was wrong on one item. A compensating event is appended:

| | Compensation |
|--|--------------|
| stream_id | sale-0042 |
| sequence | 2 |
| global_position | 98350 |
| compensates_event_id | (A's original event_id) |
| actor | human:cashier-a |

The fold for stream `sale-0042` now produces the corrected total. The original sale event at sequence 1 remains forever — visible, auditable, immutable. The correction is explicit, traceable, and structurally identifiable by the non-null `compensates_event_id`.

---

## 7. Boundaries

| Topic | Lives in |
|-------|----------|
| Why events are the sole source of truth (doctrine) | CN-4-001 |
| How events are linked into a tamper-proof hash chain | CN-4-003 |
| How commands are evaluated and events emitted through the bus | CN-4-004 |
| How replay processes events to rebuild state | CN-4-009 |
| How projections register handlers per event type/version | CN-4-010 |
| What each primitive's payload contains and how its fold works | CN-4-011 |
| How time is assigned from the clock protocol | CN-4-014 |
| How the store scales (partitioning, archiving, sharding) | Architect phase |

---

## 8. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Hash chain: how events are linked using event_id and global_position | CN-4-003 | This contract provides the fields; hash chain uses them |
| Per-primitive payload schemas and fold logic | CN-4-011 | Each primitive defines what the payload contains |
| Projection handler registration per event version | CN-4-010 | The versioning contract here enables multi-version handlers there |
| Store scaling (partitioning, archiving) | Architect phase | Contract guarantees append-only, immutable, replayable at any scale; mechanism is Architect's |

---

*— End of CN-4-002 —*
