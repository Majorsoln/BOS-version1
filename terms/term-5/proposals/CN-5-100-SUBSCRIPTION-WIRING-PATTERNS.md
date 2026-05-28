# CN-5-100 — Subscription Wiring Patterns

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 5 — Universal Engines
> **Status:** For Overseer review.
> **Governing decisions:** D-007 #5 (engine isolation — runtime + CI); D-009 (freeze doctrine — historical pack version per event).
> **Glossary:** See `MASTER-GLOSSARY.md` — Event, Subscription, Projection, Fold, Replay.
> **Depends on:** CN-4-001 (Doctrine — events are truth, state is derived), CN-4-002 (Event Store Contract — envelope, ordering, causation_id), CN-4-004 (Command Bus — sole entry for state changes), CN-4-005 (Engine Contract — manifest, `subscribes_to`, at-least-once, ordered-within-stream, status-event pattern), CN-4-009 (Replay — "replay never emits events"), CN-4-010 (Projection Framework — processed_position, idempotency, stale-but-available), CN-4-017 (Resilience Modes — health-check lag escalation).
> **References:** CN-4-008 (Audit Log — causation lineage), CN-4-011 (Primitive Catalog — per-primitive folds), CN-4-022 (Advisor Framework — read-only via projections).
> **Boundaries:** Event envelope and ordering → CN-4-002; manifest shape → CN-4-005; bus and policy → CN-4-004; projection lifecycle and rebuild → CN-4-010; replay mechanics → CN-4-009; resilience-mode behaviour → CN-4-017; scope policy details → CN-5-101; engine invariants → CN-5-102; cross-engine choreography of period close → CN-5-104; engine business rules → per-engine docs (CN-5-001 … CN-5-007, CN-5-009, CN-5-010).

---

## 1. What This Doc Defines

CN-5-100 is the **contract between universal engines** for how one engine reacts to another engine's events. Every other Term 5 doc relies on this — Accounting subscribes to checkout events, Inventory subscribes to procurement events, Reporting subscribes to almost everything. Without a settled wiring contract, each engine doc would re-invent the rules of choreography.

This doc defines:
- The **five named subscription patterns** that universal engines may use (§2).
- The **five laws** that govern every subscription (§3).
- The shape of a **subscription declaration** in the engine manifest (§4), extending CN-4-005.
- **Replay semantics** for subscribers — when re-execution happens and when it does not (§5).
- **Failure and freshness semantics** when a subscriber falls behind or fails (§6).
- How **scope** is inherited from event to subscriber (§7, pointer to CN-5-101).

This doc does NOT define:
- The event envelope or ordering (CN-4-002).
- The manifest's general fields (CN-4-005 §1).
- The mechanics of projections — handlers, position tracking, rebuilds (CN-4-010).
- Scope policy itself (branch vs business vs platform) — that is CN-5-101.
- Engine-specific business rules — each engine's doc owns those.

---

## 2. The Five Subscription Patterns

Every cross-engine reaction in Term 5 fits one of five named patterns. Naming them makes downstream engine docs short: an engine declares the pattern by name, and the rules of that pattern follow from this doc.

| # | Pattern | What it does | Subscriber output |
|---|---------|-------------|-------------------|
| **P1** | **Fan-out subscription** | One event is consumed by multiple independent subscribers. No ordering is implied between them. | Each subscriber acts on its own (mix of P2/P3/P4). |
| **P2** | **Projection-only subscription** | Subscriber updates its own read-model and stops. | Read-model update; no new events. |
| **P3** | **Command-emitting subscription** | Subscriber reacts by submitting a new command through the bus; the bus evaluates and may accept or reject. | A new command into CN-4-004; possibly new events from that command. |
| **P4** | **Compensation subscription** | Subscriber reacts to a compensating event (CN-4-002 `compensates_event_id` is non-null) by reversing or adjusting its own state. | Read-model adjustment or a compensating command (P3 sub-case). |
| **P5** | **Cross-engine choreography sequence** | A declarative chain of P2/P3 reactions across engines, with no central orchestrator. Each step subscribes to the previous step's event. | Whatever each step's handler produces. |

### Ordering Between Subscribers (Ruling Addition 1)

For a fan-out subscription (P1), **there is no ordering guarantee between subscribers**. If Accounting, Cash, and Inventory all subscribe to `checkout.settled.v1`, the order in which each catches up is undefined and may differ from one moment to the next. Each subscriber processes its own input in global_position order (CN-4-010 §6) for **its own stream of subscribed events**, but no cross-subscriber sequencing exists.

This is intentional. Choreography is emergent (L3, §3). If any Term 5 engine needs to **wait for** another engine's effect before proceeding, it does so by subscribing to that engine's **status event** (CN-4-005 §4 status-event pattern) and reading its own local projection — never by waiting on another subscriber's progress.

### Fan-In Is Not a New Pattern

A subscriber that must observe several events before acting (e.g., "period close completes only when Accounting, Cash, Inventory, and HR have each emitted their `*.period.ready.v1`") implements **status-event + local projection** (CN-4-005 §4) plus a guarded check. This is two P2 subscriptions plus a local readiness test — not a new pattern. CN-5-104 (Period-Close Choreography) instantiates this for the period-close case.

---

## 3. The Five Laws

These laws are non-negotiable. Every Term 5 subscription must satisfy all five. Violations are doctrine violations (CN-4-019 enforces them in CI).

| # | Law | Source / why |
|---|-----|--------------|
| **L1** | **Events-only.** A Term 5 engine never calls another engine's function, RPC, or internal API. The sole means of cross-engine reaction is event subscription through the Kernel's bus. | Charter Law 2; CN-4-005 §2 (isolation guarantee). |
| **L2** | **Subscriptions are declarative.** Every subscription is declared in the engine's manifest (`subscribes_to`, CN-4-005 §1) as a per-entry record: `{event_type, version, handler, kind, scope_ref}`. There is no runtime registration. | CN-4-005 §1 already establishes `subscribes_to`; CN-5-100 extends each entry with `kind` and `scope_ref`. Doctrine enforcement (CN-4-019) reads manifests. |
| **L3** | **Choreography only — no central orchestrator.** No Term 5 engine and no Kernel component sequences other engines' actions. Each engine reacts to events it subscribes to; the system-level behaviour is the emergent sum of those reactions. | Foundation provides no orchestrator primitive; central sequencers create coupling and single points of failure. The bus and manifest are sufficient. |
| **L4** | **Subscribers produce events only through commands.** A subscription handler may (a) update its own projection state (P2), (b) submit a new command through the bus (P3/P4), or (c) both. It **never writes events directly to the store** and never bypasses the bus. | CN-4-004 §2: the bus is the sole entry point for state changes. Direct event emission from a handler would bypass policy evaluation, identity, and freeze doctrine. |
| **L5** | **Idempotency is a framework property, not a handler property.** Re-delivery of an event is handled by `processed_position` (CN-4-010 §4). Handlers do not implement per-event de-duplication. | CN-4-010 §4 is already the framework guarantee. CN-5-100 imports it without restating mechanism. |

---

## 4. Subscription Declaration

CN-4-005 §1 already lists `subscribes_to` as a manifest field. CN-5-100 specifies the **shape of each entry**.

| Field | Type | Meaning |
|-------|------|---------|
| **event_type** | String | `<engine>.<noun>.<verb>.v<n>` per Charter §8.1. The subscriber names an event type, never an engine (CN-4-005 §1: "the manifest never references another engine by name"). |
| **version** | Integer | The schema version the handler accepts. If multiple versions are accepted, declare each as its own entry. Multi-version handling otherwise follows CN-4-010 §7 (upcast or fail; never silent skip). |
| **handler** | Handler reference | The named handler within the subscribing engine that processes this event-type/version. |
| **kind** | Enum: `projection` \| `command_emitting` \| `compensation` | The subscription kind. Determines replay behaviour (§5). |
| **scope_ref** | Scope policy reference | The scope under which this subscription operates. Resolved per CN-5-101 (not redefined here). Default: inherit the event's `tenant_id` per CN-4-010 §9. |

### Required vs Optional

CN-4-005 §1 distinguishes `requires` (mandatory subset of `subscribes_to`) from optional subscriptions. CN-5-100 honours that: a `requires` subscription is one the engine cannot function without (e.g., Accounting requires `checkout.settled.v1`); an optional subscription enriches behaviour but is not load-bearing.

### Cross-Engine Contract Markers (Ruling Q4)

Every event emitted by a Term 5 engine is potentially subscribable (no "private events," Ruling Q4). However, the manifest's `emits` list (CN-4-005 §1) is where the engine declares which event types it considers **cross-engine contracts** — meaning: additive-only forever (CN-4-002 §3), no breaking change without a new event type. Engines that need internal-only state use **their own projections** (CN-4-010), not events. This preserves "events are truth" while bounding the universe of contract events that must be supported in perpetuity.

### Orphan Detection

CN-4-005 §2 says: "the manifest's `emits` and `subscribes_to` are cross-checked: an engine cannot subscribe to an event type no engine declares it emits (orphan subscription warning)." CN-5-100 inherits this guarantee — no new mechanism needed.

---

## 5. Replay & Derivative Events

This section resolves Ruling Q2.

### Principle

CN-4-009 §8 establishes: **"Replay never emits events."** A subscriber's reaction that emits new events (a `command_emitting` or `compensation` subscription) is therefore a **live-only** behaviour. During replay, the events those reactions produced **are already in the store** — replay processes them as the events they are, not as outcomes to be re-derived.

### How Each Kind Behaves on Replay

| Kind | Live behaviour | Replay behaviour |
|------|----------------|------------------|
| **projection** | Handler updates the subscriber's read-model. | Handler runs again to rebuild the read-model. Determinism per CN-4-010 §6. |
| **command_emitting** | Handler submits a command through the bus; the bus evaluates and emits the resulting event(s). | Handler does **not** re-execute. The events it caused live-time are replayed as themselves. The read-model effects of those derivative events come from their **own** projection handlers, in global_position order. |
| **compensation** | Handler reacts to a compensating event by updating state and/or submitting a corrective command. | Same as command_emitting if the handler emitted a command. If it only updated a projection, replay re-runs the projection update. |

### Derivative Events Are First-Class Store Events

A derivative event — Accounting's `accounting.journal.posted.v1` triggered by `checkout.settled.v1` — is not a "side effect." It is a fully-formed event in the store with:
- Its own `event_id`, `stream_id`, `sequence`, `global_position` (CN-4-002).
- Its own `pack_version_ref` recorded at emission (CN-4-002 / D-009).
- Its own subscribers downstream.

Replay treats it identically to any other event. The chain `checkout.settled.v1 → accounting.journal.post.request → accounting.journal.posted.v1` is reconstructed by replaying all three records in order, not by re-running the handler that produced the middle one.

### Causation Linkage (Ruling Addition 2)

Every derivative event records the triggering event in its envelope's `causation_id` field (CN-4-002). This makes choreography **traceable end-to-end without an orchestrator**:

- `checkout.settled.v1` has `causation_id = null` (caused by a command, not another event — CN-4-002 §1).
- The follow-up command Accounting submits carries the same `correlation_id` as the original (CN-4-004 §5) and references the checkout event in its causation.
- `accounting.journal.posted.v1` carries `causation_id = <event_id of checkout.settled.v1>`.

The audit log (CN-4-008) walks the `causation_id` chain to reconstruct the full lineage of any outcome — which event triggered which command, which command produced which downstream event. This is the audit substitute for orchestration: there is no orchestrator log because there is no orchestrator; the events themselves are the log.

### Implication for Engine Authors

When writing a Term 5 engine doc, declare each subscription's `kind` explicitly. A handler that updates the engine's own projection is `projection`. A handler that submits a command is `command_emitting`. A handler that recognises a compensating event and reverses state is `compensation`. The kind is not a hint — it determines replay-time semantics enforced by the framework.

---

## 6. Failure & Freshness

This section resolves Ruling Q3.

### Per-Subscriber Stale-But-Available

When a subscriber falls behind or fails:

| Stage | Behaviour |
|-------|-----------|
| **Subscriber lag (normal)** | The subscriber's `processed_position` (CN-4-010 §4) trails the store's latest position. Reads against its projection are stale-but-available (CN-4-010 §6) — queries return data; the freshness indicator (CN-4-022 §7) shows the lag. Other subscribers are unaffected — they each have their own processed_position. |
| **Subscriber handler error** | At-least-once delivery (CN-4-005 §3) re-attempts the event. The subscriber resumes from its last processed_position; no events are skipped. Other subscribers continue independently (L3 — no ordering between subscribers). |
| **Subscriber lag exceeds health-check threshold** | The Kernel's health-check dimension `projection_lag` (CN-4-017 §5) escalates the system mode. If only one subscriber is impaired, the resilience response is proportionate (CN-4-017 §5 impaired-dimension context) — typically NORMAL→DEGRADED. |
| **Catastrophic subscriber failure (handler broken)** | The projection is **rebuilt** per CN-4-010 §8 — stale-but-available + catch-up + atomic switchover. The system never goes offline because one Term 5 subscriber's handler has a bug. |

### Why No Hard Fail on the Source Event

A retail sale that emits `checkout.settled.v1` is **complete** the moment that event is in the store. If Cash Engine's subscription fails to process it, the sale itself is not undone — the event remains; Cash's projection is stale; the sale is still legally and operationally true. This is enforced by Law 1 (CN-4-001): **events are truth; projections are derived**. A failing subscriber cannot retroactively invalidate truth.

Hard-fail rejection is the bus's job at command time (CN-4-004 §3), not a subscriber's job at event time. By the time an event is in the store, the truth has been recorded; downstream behaviour is observation, not gating.

### Subscribers Communicate Freshness, Not Truth

A subscriber's projection always exposes its `processed_position` and `processed_timestamp` (CN-4-010 §4). Consumers — UI (Term 3), advisors (CN-4-022), reports (CN-5-006) — read freshness alongside the data. The Advisor Framework's `freshness_indicator` is derived from this and disclosed to users (CN-4-022 §7).

---

## 7. Scope Inheritance

CN-5-100 does not define scope policy. It defines only the **link** between an event and a subscription's scope:

- Tenant-scoped events deliver to tenant-scoped subscribers per CN-4-010 §9. No subscriber sees events outside its tenant.
- Platform-scoped events (CN-4-006) are subscribed to under the platform scope (CN-4-007) — used by Term 1 components, not by Term 5 universal engines under normal operation. If a universal engine needs platform-aggregate data (e.g., billing aggregation), it is via a platform-scoped subscription audited per platform-scope rules.
- Branch- vs business-scope distinction for universal engines is the subject of **CN-5-101**. CN-5-100 references it via the manifest's `scope_ref` field (§4) but does not define the policy itself.

---

## 8. Example — Retail Sale at Duka la Mama Amina

*Mama Amina's duka serves as illustrative context, per D-004 #4 (peer-technical audience). The example shows the engine contract operating; it is not a UX description.*

Cashier A completes a sale of three items for TZS 45,000 (paid: TZS 30,000 cash + TZS 15,000 M-Pesa).

### Step 1 — Vertical produces lines and hand-off

The Retail engine (Term 6) emits `retail.bill.ready.v1` carrying the saleable lines (per CTR-002, pending). This is not in scope for CN-5-100; it is a Term 6 emission.

### Step 2 — Checkout settles

The Universal Checkout / Tender Engine (CN-5-009, forthcoming) consumes the lines, accepts the tender, computes change (zero in this case), issues a receipt, and emits:

```
event_type:       checkout.settled.v1
stream_id:        checkout-2026-05-28-mwanza-001-0042
tenant_id:        mama-amina-duka
actor:            human:cashier-a
causation_id:     null            (command-driven, not event-driven)
correlation_id:   <original command>
pack_version_ref: tz-compliance-2026.05
payload:          {lines: [...], tenders: [{method: cash, ...}, {method: mpesa, ...}], receipt_ref: ...}
```

### Step 3 — Three universal engines subscribe (fan-out, P1)

Each engine's manifest declares one entry in `subscribes_to`:

| Engine | event_type | version | kind | handler | scope_ref |
|--------|------------|---------|------|---------|-----------|
| Accounting (CN-5-001) | `checkout.settled.v1` | 1 | `command_emitting` | `post_sale_journal` | business |
| Cash (CN-5-002) | `checkout.settled.v1` | 1 | `command_emitting` | `record_tender_receipt` | branch |
| Inventory (CN-5-003) | `checkout.settled.v1` | 1 | `command_emitting` | `deduct_stock_for_lines` | branch |

Each subscriber receives the event independently. No ordering exists between them (§2 ruling).

### Step 4 — Each subscriber reacts (P3, command-emitting)

**Accounting handler** computes the journal (revenue + tax + cost-of-goods) and submits `accounting.journal.post.request` through the bus (L4). Bus evaluates policies, accepts, and emits:

```
event_type:       accounting.journal.posted.v1
causation_id:     <event_id of checkout.settled.v1>
correlation_id:   <inherited from the post.request command>
pack_version_ref: tz-compliance-2026.05    (D-009 — same pack as the trigger)
payload:          {debits: [...], credits: [...], journal_ref: ...}
```

**Cash handler** submits `cash.tender.receipt.request` for each tender. Bus emits `cash.tender.received.v1` (×2 — one cash, one M-Pesa), each carrying `causation_id` back to the checkout event.

**Inventory handler** submits `inventory.stock.deduct.request` for each line. Bus emits `inventory.stock.deducted.v1` per line, each with `causation_id` linking back.

### Step 5 — Downstream observation

The audit log (CN-4-008) reconstructs the full lineage from the causation chain: one settled checkout produced one accounting journal, two cash receipts, and three inventory deductions. The chain is verifiable, ordered (per stream), and reproducible on replay.

### Step 6 — Replay

A replay later (e.g., a tax audit for May 2026) processes all events in global_position order. The handlers in Accounting/Cash/Inventory **do not re-execute** for the original checkout event — their derivative events are already in the store and replay them as themselves (§5). The pack version `tz-compliance-2026.05` recorded on each event ensures historical interpretation (D-009).

### What If Cash's Handler Has a Bug?

Suppose `record_tender_receipt` crashes on M-Pesa tenders. The bus has already emitted `checkout.settled.v1` — the sale is truth. At-least-once delivery (CN-4-005 §3) re-tries; if the handler is broken, the projection lag grows; the health check (CN-4-017 §5) escalates; a platform admin investigates; the handler is fixed; the projection is rebuilt (CN-4-010 §8). Accounting and Inventory are unaffected throughout — they each have their own processed_position. The sale stays valid; only Cash's read-model is stale until rebuild completes.

---

## 9. Three Whys

### Why does this matter?

Without a defined subscription contract, each Term 5 engine would invent its own answer to "how do I react to another engine's event?" — and the answers would diverge. One engine would import another's modules; one would call another's database; one would write events directly to the store from a handler. Each shortcut is one tenant-impacting outage waiting to happen. CN-5-100 makes wiring explicit, declarative, and uniform — so the seven universal engines behave as one system, not seven coupled silos.

### Why does it belong here (and not in Foundation)?

Foundation provides the **mechanism**: the event store (CN-4-002), the bus (CN-4-004), the projection framework (CN-4-010), the manifest (CN-4-005). Foundation does not say "Accounting subscribes to Checkout." That is a Term 5 design decision. CN-5-100 is the Term 5 **policy** on top of Foundation's mechanism: which patterns are allowed, what each subscription declares, what replay does. Foundation is engine-agnostic; CN-5-100 is universal-engine-specific.

### Why this design?

Declarative subscriptions + five named patterns + five laws give every Term 5 engine doc a short common vocabulary. A new vertical (Term 6) added in 2027 can wire into existing universal engines by emitting events those engines already subscribe to — zero code change in Term 5 (CN-4-005 §7 demonstrates this for a future engine). Choreography-only (L3) eliminates the central-orchestrator failure mode. Position-tracked idempotency (L5) inherits CN-4-010's guarantees without restating mechanism. Live-only re-execution semantics (§5) make replay deterministic without ambiguity about what gets re-run. The whole design is "lean concept on top of strong primitives" — Foundation does the heavy lifting; CN-5-100 names the patterns that use it.

---

## 10. Boundaries

| Topic | Lives in |
|-------|----------|
| Event envelope (15 fields, including `causation_id`, `pack_version_ref`) | CN-4-002 |
| Event-sourcing doctrine (events are truth) | CN-4-001 |
| Command bus as sole state-change entry | CN-4-004 |
| Engine manifest (general field list, isolation guarantees, status-event pattern) | CN-4-005 |
| Projection framework (handlers, processed_position, rebuilds) | CN-4-010 |
| Replay mechanics ("replay never emits events") | CN-4-009 |
| Resilience modes (lag escalation, DEGRADED behaviour) | CN-4-017 |
| Audit log (lineage via causation chain) | CN-4-008 |
| Advisor freshness disclosure | CN-4-022 |
| Scope policy (branch vs business vs platform) for universal engines | CN-5-101 |
| Cross-engine invariants (debit=credit, stock has source, etc.) | CN-5-102 |
| Period-close fan-in choreography | CN-5-104 |
| Engine-specific subscriptions (which Accounting/Cash/Inventory/… actually wires) | per-engine docs (CN-5-001 … CN-5-007, CN-5-009, CN-5-010) |

---

## 11. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Static analysis rule: every `subscribes_to` entry has a declared `kind` and `scope_ref` | CN-4-019 + Architect | Doctrine enforcement extension; mechanism is Architect's |
| Per-engine catalog of `command_emitting` vs `projection` subscriptions | per-engine docs (CN-5-001 …) | Each Term 5 engine doc declares its own subscriptions and kinds |
| Mechanism for declaring "this subscription is required vs optional" | CN-4-005 §1 already covers via `requires` | No new mechanism needed |
| Cycle detection in subscription graphs (engine A → B → A) | CN-4-019 + Architect | CN-4-005 §9 already flags it; CN-5-100 confirms it applies to Term 5 |
| Performance budget for fan-out delivery (how fast must subscribers catch up?) | Architect phase + CN-4-017 thresholds | CN-5-100 says "eventually consistent"; thresholds are config (D-004) |
| Subscription naming convention (handler reference syntax) | Architect phase | Concept says "named handler within the subscribing engine"; format is Architect's |

---

*— End of CN-5-100 —*
