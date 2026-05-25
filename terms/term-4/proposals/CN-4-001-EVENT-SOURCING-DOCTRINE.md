# CN-4-001 — Event Sourcing Doctrine

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-007 #3 (compensating events), D-009 (freeze doctrine).
> **Glossary:** See `MASTER-GLOSSARY.md` — Event, Compensating Event, Fold, Projection, Replay.
> **Boundaries:** This doc is doctrine (why and what it means). Mechanics live in CN-4-002 through CN-4-018.

---

## 1. Doctrine Statement

The BOS Kernel is event-sourced. The following assertions are non-negotiable and apply to every engine, every primitive, and every layer built on the Kernel.

### Assertion 1 — Events are the only source of truth.

There is no database table, cache, or projection that is authoritative. If any derived state conflicts with the event stream, the event stream wins. The stream is complete: everything that happened in BOS is recorded as an event.

### Assertion 2 — State is always derived, never stored as primary.

Every dashboard, report, balance, and read-model is a projection — rebuilt from events. Projections are convenience and performance; they are never the source of truth. If a projection is lost, it is rebuilt. If a projection conflicts with the stream, the projection is wrong.

### Assertion 3 — No hidden mutation anywhere in BOS.

No UPDATE. No DELETE. No silent overwrite. If something changes, a new event records that change. The previous state remains visible in the stream forever. This is structural, not policy — the Kernel does not offer a mechanism to mutate or remove events.

### Assertion 4 — Events are facts, not intentions.

An event records what happened (past tense: `sale.completed.v1`). A command is an intention (present tense: `sale.complete.request`). Commands may be rejected; events cannot. Once emitted, an event is an immutable fact in the historical record.

### Assertion 5 — The event stream is the audit trail.

Legal defensibility flows directly from the stream. When a court, tax authority, or auditor asks "prove what happened," BOS shows the events. No separate audit log is needed to reconstruct the past — the events ARE the past. (The Audit Log Primitive in CN-4-008 adds metadata and query capability on top of this, but the events are the foundation of proof.)

### Assertion 6 — State at any historical point is reconstructible.

Given any moment in the past, the Kernel can derive the exact state that existed at that moment by replaying events up to that point. This "as-of" capability is not optional — it is required for compliance, dispute resolution, and legal proceedings.

---

## 2. Three Whys

### 2.1 Why does this matter? (User pain)

African businesses cannot trust their own numbers because their systems allow silent mutation. A sale is recorded, then quietly edited. A stock count changes overnight with no trace. A cashier adjusts a price "because the customer complained" and the adjustment disappears into the database.

When the owner asks "how much did we really make this month?" — no one can answer with confidence. When the tax officer arrives, documents do not match records. When a dispute arises with a supplier, there is no proof of what was agreed.

This is not a technology problem. It is a trust problem caused by systems that permit hidden change. Event sourcing eliminates this entire class of problem: if it happened, there is an immutable event. If there is no event, it did not happen. The owner can always answer "what really happened?" with proof.

### 2.2 Why does it belong in the Kernel?

Event sourcing is not a feature that can be added to individual engines. It is the architectural decision from which all other Kernel guarantees flow:

- **Legal defensibility** (Section 1) requires an uneditable record → the event stream.
- **Replay** (CN-4-009) requires deterministic state derivation → events as sole source.
- **Engine isolation** (Charter Law 2) requires communication without coupling → engines subscribe to events, never call each other.
- **AI advisory-only** (Charter Law 3) requires a boundary AI cannot cross → AI reads projections derived from events, never writes events.
- **Audit** (CN-4-008) requires a complete trail → the event stream IS the trail.
- **Document verification** (CN-4-012) requires provable history → hash chain over the event stream.

If event sourcing were optional or per-engine, these guarantees would be local and bypassable. The doctrine must be Kernel-level so that every part of BOS inherits it automatically.

### 2.3 Why this design and not alternatives?

| Alternative | What it offers | Why it is rejected |
|-------------|---------------|-------------------|
| **CRUD (mutable state)** | Simple reads; familiar pattern | State can be silently changed. No audit trail without a separate, bypassable log. Cannot replay. Cannot prove integrity to a court. The owner's "what really happened?" is unanswerable. |
| **CRUD + separate audit log** | Mutable state with a bolted-on record | Two sources of truth that can diverge. The audit log can be disabled, fall out of sync, or be incomplete. Replay is impossible because state mutations are lossy — the audit log records actions, not the full before/after. |
| **Event sourcing (append-only, derived state)** | Single source of truth. Audit is built-in. Replay is guaranteed. Legal defensibility is structural. | Trade-off: reads require projections (more engineering); storage grows indefinitely (known, solvable). These are engineering problems with proven solutions. Silent mutation and unprovable state are business-killing problems with no solution. |

The trade-off is accepted deliberately: complexity in reads and storage growth are costs the Kernel pays so that truth, replay, and defensibility are guaranteed for every business that uses BOS, forever.

---

## 3. What This Implies

### 3.1 Compensating Events (D-007 #3)

Events are never deleted. When a wrong event was emitted — a bug, a human error, a system fault — the correction is a **new event** (a compensating event) appended to the stream. The original wrong event remains visible in the stream forever.

**Current truth** after a compensation is the **deterministic fold of all events including compensations**. The fold logic is defined **per primitive** (CN-4-011): each primitive knows how to incorporate a compensation into its running state. "Truth" is not "the latest event." It is not "the raw sum." It is the result of replaying the full stream through the primitive's fold function.

This means:
- The original wrong event is never hidden — it is always auditable.
- The correction is explicit and traceable (who corrected, when, why).
- The current state is always computable from the stream alone.

### 3.2 Freeze Doctrine (D-009)

Events are always interpreted under the rules, rates, and compliance-pack version that were active at the moment of emission. A later rule change — a new tax rate, a regulatory update, a corrected compliance pack — applies only to new events emitted after the change.

This means:
- Replay always reproduces historical state under historical rules — deterministic regardless of when replay runs.
- A document issued under old rules stands under old rules, even after those rules change.
- The Compliance DSL (CN-4-015) must support pack versioning so historical rules remain available.
- Each event references (directly or derivably) the compliance-pack version active at its creation.

### 3.3 Determinism

Given identical events replayed in identical order, the resulting state is **always identical** — regardless of wall-clock time, server, or environment. This is the property that makes replay meaningful and legal defensibility real.

Determinism requires:
- No `datetime.now()` in engine or primitive logic. All time comes from the event stream or the clock protocol (CN-4-014, Time Authority).
- No external state dependency during replay. A replay reads only the event stream and the rules referenced by the events.
- Per-primitive fold functions must be pure: same inputs → same outputs, always.

---

## 4. Example — Duka la Kariakoo

**Scenario: Tax officer visits Mzee Hassan's general store.**

Mzee Hassan runs a general store in Kariakoo, Dar es Salaam. The TRA tax officer arrives for a routine audit and asks: "Show me your sales for March. Prove that what you reported matches what actually happened."

**In a mutable system:** Last week, a cashier corrected a "mistake" by editing an old sale — deleting two line items and changing a price. The March report now shows different numbers than what was originally reported to TRA. Mzee Hassan cannot prove what the original numbers were. The system overwrote them. The tax officer sees a discrepancy between the filed return and the current report. He assumes evasion.

**In BOS (event-sourced):** Every sale in March exists as an immutable event. The cashier's "correction" is also recorded — as a compensating event that references the original sale, states what was wrong, and provides the corrected values. The March report is a projection rebuilt from all events including the compensation.

When the tax officer asks "what really happened," BOS shows:
1. The original sale events — what was sold, when, at what price, by which cashier.
2. The compensating event — what was corrected, when, by whom, with what justification.
3. The hash chain — proving that neither the originals nor the compensation have been tampered with since they were recorded.
4. A deterministic replay of the entire month — producing the exact figures that were reported.

The tax officer gets a complete, provable, tamper-evident history. Mzee Hassan is protected — not by hiding anything, but by the system's structural inability to hide. The compensating event proves the correction was intentional, traceable, and legitimate.

---

## 5. Boundaries

CN-4-001 is doctrine — it states **why** and **what this means**. The following mechanics live elsewhere:

| Topic | Lives in |
|-------|----------|
| The concrete shape of an event (required fields, envelope, metadata) | CN-4-002 (Event Store Contract) |
| How the hash chain proves the stream's integrity | CN-4-003 (Hash Chain & Tamper Detection) |
| How commands are evaluated and events emitted | CN-4-004 (Command Bus & Policy Engine) |
| How replay works mechanically (checkpoints, scoping, partial) | CN-4-009 (Replay Engine) |
| How projections are defined, registered, and rebuilt | CN-4-010 (Projection Framework) |
| The compensating-fold logic for each specific primitive | CN-4-011 (Primitive Catalog) |
| How time is handled deterministically | CN-4-014 (Time Authority) |
| How compliance-pack versions are stored and referenced | CN-4-015 (Compliance DSL Framework) |
| How snapshots are marked as non-truth | CN-4-018 (Snapshot Storage) |

---

## 6. Open Items & Glossary

### Open items left to other CN docs

| Item | Assigned to | Notes |
|------|-------------|-------|
| How is event ordering guaranteed when two commands arrive simultaneously? | CN-4-002 | Sequence numbers, tie-breaking — mechanics, not doctrine |
| What does the compensating fold look like for each primitive? | CN-4-011 | Each primitive defines its own fold logic |
| Freeze under a later-found-buggy compliance pack — how is correction handled? | CN-4-015 + this doc (Section 11, N5) | Principle: freeze holds; correction = new compensating events under corrected pack; CN-4-015 defines the workflow |
| How is "selective replay" (skip bad events) expressed without deletion? | CN-4-009 | Direction: compensating events neutralise the bad events; fold produces correct state |

### Glossary tie-ins (MASTER-GLOSSARY.md)

| Term | As used in this doc |
|------|---------------------|
| **Event** | An immutable record of something that happened — a fact in the past tense, stored in the append-only stream |
| **Compensating Event** | A new event that corrects the effect of a prior event without deleting or modifying it |
| **Fold** | The deterministic function that replays a sequence of events to produce current state, defined per primitive |
| **Projection** | A derived read-model rebuilt from events; convenience, never truth |
| **Replay** | The act of running events through fold logic to reconstruct state at any point in time |

---

*— End of CN-4-001 —*
