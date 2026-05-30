# CN-5-102 — Engine Invariants

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 5 — Universal Engines
> **Status:** For Overseer review.
> **Governing decisions:** D-004 (neutrality — invariants name no vertical); D-007 #3 (compensating fold); D-009 (freeze doctrine — closed-period semantics).
> **CTRs:** CTR-025 (Term 5 → Term 4 — DC-038..041 added to CN-4-019 living catalog, OPEN); CTR-026 (Term 5 → Term 6 — verticals respect UI-03 source ref + UI-09 site_id, OPEN); CTR-027 (Term 5 → Term 1/2 — tenant site registry + tenant functional currency registry, OPEN).
> **Glossary:** See `MASTER-GLOSSARY.md` — Universal-engine Invariant (UI), Doctrine Check (DC).
> **Depends on:** CN-4-001 (events are truth); CN-4-002 (envelope incl. `causation_id`, `pack_version_ref`); CN-4-004 (bus rejection policy); CN-4-011 (primitive folds — Obligation, Inventory Movement, Ledger); CN-4-014 (tenant fiscal zone as tenant-property pattern); CN-4-016 (anomaly detection); CN-4-017 (resilience mode escalation); CN-4-019 (doctrine-check living catalog); CN-5-100 (subscription laws — derivative events, causation linkage); CN-5-101 (scope laws — site_id payload, site-scope dispatch).
> **Boundaries:** Doctrine check mechanics (gate, attestation, tooling) → CN-4-019 + CN-4-023; primitive folds → CN-4-011; subscription patterns/laws → CN-5-100; scope policy → CN-5-101; per-engine business rules → per-engine docs (CN-5-001 … CN-5-007, CN-5-009, CN-5-010); period-close choreography mechanics → CN-5-104; tax-aware engine wiring → CN-5-105; tenant-property registries (site, functional currency) → Term 1 (governance) + Term 2 (population).

---

## 1. What This Doc Defines

CN-5-102 is the **cross-engine doctrine of state** for universal engines. It states the truths that must hold about state once events have been emitted by two or more universal engines working in choreography. These invariants are the cross-engine analogue of CN-4-019's Kernel doctrine checks: they are testable, mechanizable, and authoritative.

This doc defines:
- A **catalog of cross-engine invariants** (initial: UI-01 … UI-10) (§3).
- The **per-invariant format** every entry uses (§4).
- An **enforcement strategy** that distributes each invariant across runtime, CI-integration, and CI-schema layers (§5).
- A **violation-handling model** — hard-fail, anomaly alert, mode escalation (§6).
- A **DC ramp-up plan** — which invariants become CN-4-019 doctrine checks now (Phase 0, CTR-025) and which wait for per-engine docs (Phase 1) (§7).
- The **living-catalog discipline** by which CN-5-102 grows (§8).

This doc does NOT define:
- Subscription laws or replay semantics (CN-5-100).
- Scope-level laws or runtime resolution (CN-5-101).
- Primitive folds — e.g., that a single Ledger journal balances (CN-4-011).
- Per-engine business rules — e.g., "a sale must have at least one line" (per-engine docs).
- The doctrine-gate mechanism itself (CN-4-019 + CN-4-023).
- Tenant-property registries that some invariants reference (Term 1 + Term 2, via CTR-027).

---

## 2. Doctrine — The Three Boundary Rules

These rules govern what is and is not an "engine invariant" in CN-5-102.

| # | Rule | Why |
|---|------|-----|
| **B1** | **Cross-engine only.** A statement is a CN-5-102 invariant only when verifying it requires state from two or more universal engines, OR when it constrains a primitive choreography that crosses engines. Per-engine truths belong to per-engine docs. | CN-4-019 §8 separates "Kernel invariants" from "engine business logic"; CN-5-102 is the cross-engine analogue. Per-engine truths in CN-5-102 would duplicate per-engine docs and blur ownership. |
| **B2** | **State-shape, not behavior.** An invariant is a statement of truth about state ("debits equal credits per tenant," "every derivative event has causation"), not a statement of action ("Cash must record receipts"). Behavior belongs to subscription patterns (CN-5-100) and engine docs. | Invariants are post-conditions across the system. They survive engine reorganisation, handler renaming, and replay. Statements of behaviour rot with implementation; statements of state do not. |
| **B3** | **Inheritance, not re-statement.** Primitive folds (CN-4-011), subscription laws (CN-5-100 L1–L5), and scope laws (CN-5-101 L1–L5) are inherited as background truths. CN-5-102 builds on them; it does not restate them. | Avoids doctrine sprawl. Each invariant must add a constraint not already established below; if it does not, it is removed. |

---

## 3. The Initial Universal-Engine Invariant Catalog

Ten invariants are recorded below. Each follows the per-invariant format (§4). The catalog is **living** (§8): new universal-engine docs may add invariants through the normal collaboration process.

---

### UI-01 — Causation Integrity on Derivative Events

| Field | Value |
|-------|-------|
| **Statement** | Every event whose handler ran as the result of subscribing to another event carries `causation_id` pointing to that source event. Command-origin events (events emitted by a handler that was invoked by a command, not by a subscription) carry `causation_id = null`. |
| **Scope** | tenant (the chain belongs to one tenant; CN-4-006 isolation) |
| **Spans engines** | All universal engines |
| **Enforcement** | CI schema validation (causation_id field shape) + runtime (bus refuses emission from a `command_emitting` subscription handler whose causing-event reference is missing) |
| **DC mapping** | DC-038 (Phase 0, CTR-025) |
| **Violation** | Hard fail at emission |
| **Reasoning** | Without causation, the audit log (CN-4-008) cannot reconstruct the lineage of any cross-engine outcome. The choreography substitutes for an orchestrator's run-log (CN-5-100 §5); the substitute fails if links are missing. The distinction between command-origin (`causation_id = null`) and subscription-derivative (`causation_id = <source>`) is rooted in CN-5-100 §5. |

---

### UI-02 — Compensation Symmetry Across Choreography

| Field | Value |
|-------|-------|
| **Statement** | When a compensating event is emitted for an event E, every downstream derivative event of E must be compensated by a corresponding compensating effect within bounded propagation. No orphan derivative survives a compensation. |
| **Scope** | tenant |
| **Spans engines** | All universal engines participating in the original choreography of E |
| **Enforcement** | CI integration test (replay: emit E → derivatives → compensate E → assert downstream compensations present) |
| **DC mapping** | Phase 1 — pending CN-5-001 Accounting + CN-5-104 Period-Close (compensation choreography pattern) |
| **Violation** | Post-hoc anomaly alert (CN-4-016) + investigation; mode escalation if widespread |
| **Reasoning** | CN-4-001 Assertion 3 forbids deletion; truth = fold of all events including compensations. If a compensating event "undoes" the source but leaves derivatives standing, the system's running state is incoherent: Accounting has reversed but Inventory still shows the reduction. The choreography that propagated forward must propagate back. |

---

### UI-03 — Inventory Movement Source-Ref Required

| Field | Value |
|-------|-------|
| **Statement** | Every inventory movement event payload carries a typed `source_ref` identifying its origin: `procurement.grn`, `retail.sale`, `restaurant.consume`, `workshop.consume`, `transfer`, `adjustment`, or `count`. No naked stock change is accepted. |
| **Scope** | site (the movement is per-site; CN-5-101) |
| **Spans engines** | Inventory + Procurement + every vertical engine that affects stock + Cash for adjustment-via-compensation |
| **Enforcement** | CI schema validation (payload contract — required field with enum constraint) + runtime (bus rejects emission missing source_ref) |
| **DC mapping** | DC-039 (Phase 0, CTR-025) |
| **Violation** | Hard fail at command |
| **Reasoning** | A stock change without a source is the audit equivalent of a missing receipt — the system cannot explain why the number moved. Procurement disputes, vertical sales reconciliation, theft investigation, and tax audit all rely on every movement being traceable. Pairs with CTR-026 (verticals must populate the source). |

---

### UI-04 — Cash Tender → Authorised Origin Chain

| Field | Value |
|-------|-------|
| **Statement** | Every `cash.tender.received.v1` event traces causally to an authorised origin: `checkout.settled.v1`, `cash.refund.issued.v1`, or `cash.transfer.received.v1`. |
| **Scope** | tenant (causation chain may cross site events but stays within tenant) |
| **Spans engines** | Cash + Checkout + Promotion (refund paths) + Cash compensation of itself (transfer) |
| **Enforcement** | CI integration test (causation walk: for a sampled set of `cash.tender.received` events, walk `causation_id` until terminal; assert terminal is in the authorised set) |
| **DC mapping** | Phase 1 — pending CN-5-002 Cash + CN-5-009 Checkout |
| **Violation** | Post-hoc anomaly alert; mode escalation if pattern detected |
| **Reasoning** | A tender record with no authorised origin is either a bug, a misrouted event, or — in the worst case — fraud. The system must be able to answer "where did this cash come from?" by walking events alone (CN-4-008 audit lineage). The authorised-origin set is closed: if a new origin emerges (e.g., direct cash injection by owner), it joins the set explicitly. |

---

### UI-05 — Closed-Period Inviolability

| Field | Value |
|-------|-------|
| **Statement** | Once `accounting.period.closed.v1` is emitted for period P (tenant T), no event with `effective_date ∈ P` is accepted by the bus for tenant T — **including compensating events**. Corrections for closed-period truths are posted in the **current open period** as new events whose `causation_id` references the closed-period event being corrected. |
| **Scope** | tenant |
| **Spans engines** | Accounting (period state) + every universal engine that emits effective-date-bearing events |
| **Enforcement** | Runtime (bus engine-policy on every effective-date-bearing command — CN-4-004 §3) + CI integration test |
| **DC mapping** | Phase 1 — pending CN-5-001 Accounting + CN-5-104 Period-Close Choreography |
| **Violation** | Hard fail at command |
| **Reasoning** | A closed period is a tenant's authoritative snapshot — taxes filed, statements issued, decisions made. Allowing late events (even compensations) into the closed window invalidates the snapshot retroactively. The doctrine: closed-period correctness is fixed forever; corrections move forward with explicit causation back, preserving both the historical truth and the corrective trail (D-009 freeze + CN-4-001 immutability). |

---

### UI-06 — Tenant Functional Currency Invariant

| Field | Value |
|-------|-------|
| **Statement** | Every primary financial event for tenant T uses T's **declared functional currency**. Multi-currency operations emit explicit `*.fx.recorded.v1` events that bridge non-functional currency amounts into the functional currency. |
| **Scope** | tenant |
| **Spans engines** | Accounting + Cash + Procurement + Promotion + HR (payroll) + Checkout |
| **Enforcement** | CI schema validation (currency field on event payload validated against tenant's functional currency registry) + runtime (bus reject if mismatch with no FX event) |
| **DC mapping** | DC-040 (Phase 0, CTR-025) — extends DC-037 |
| **Violation** | Hard fail at command |
| **Reasoning** | Functional currency is a **tenant property** (a fact about the tenant, like its fiscal zone in CN-4-014). Without a single functional currency, the trial balance (UI-07) is undefined — debits in one currency cannot directly balance credits in another. Multi-currency reality is honoured through explicit FX events, not by silently mixing units. CTR-027 establishes the registry; this invariant uses it. **Distinction from DC-037:** DC-037 checks single-currency per settlement (a single tender event); UI-06 checks consistency across every primary financial event tenant-wide. |

---

### UI-07 — Tenant Trial Balance

| Field | Value |
|-------|-------|
| **Statement** | For any tenant T at any `global_position` G, the sum of debits equals the sum of credits across all Accounting journal events for T at G. |
| **Scope** | tenant |
| **Spans engines** | Accounting (carrier) + every engine whose events trigger journals (Cash, Inventory, Procurement, HR, Promotion, Checkout) |
| **Enforcement** | CI integration test (replay tenant events to randomly-sampled `global_position` values; compute trial balance; assert balanced) |
| **DC mapping** | Phase 1 — pending CN-5-001 Accounting |
| **Violation** | Hard fail in CI; mode escalation (NORMAL → DEGRADED, CN-4-017) if detected at runtime — a runtime imbalance indicates projection drift or worse |
| **Reasoning** | Trial balance is the mechanical proof that double-entry holds. Each individual journal balancing (a primitive-level guarantee from CN-4-011 Ledger) does not by itself prove that *all* journals together balance at *every* point — replay could expose a missing handler invocation, a dropped subscription, a compensation gap. UI-07 is the system-wide assertion; per-journal balance is the primitive guarantee that makes it tractable. |

---

### UI-08 — Obligation Bounds Preserved

| Field | Value |
|-------|-------|
| **Statement** | No engine bypasses the Obligation primitive's bounds (`outstanding_amount ∈ [0, original_amount]` unless `obligation.kind = credit_refund`) through raw mutation. Engines affect Obligation state only through the primitive's legal operations (create, partially settle, fully settle, void) emitted as engine-namespaced events. |
| **Scope** | tenant |
| **Spans engines** | Procurement (PO obligations) + Cash (settlement) + Promotion (refund obligations) + Accounting (AP/AR projections) |
| **Enforcement** | Runtime (Obligation primitive fold in CN-4-011 enforces bounds; bus rejects commands that would emit out-of-bound events) + CI integration test (replay scenarios with concurrent settlements) |
| **DC mapping** | Phase 1 — pending CN-5-004 Procurement + CN-5-007 Promotion |
| **Violation** | Hard fail at settlement command |
| **Reasoning** | The bounds themselves are a **primitive guarantee** (CN-4-011); the cross-engine aspect is that **settlement events arrive from multiple engines** (Cash settles, Promotion writes off, Accounting projects). The invariant is that no engine takes a shortcut — every engine affecting an obligation does so through declared events the primitive recognises. Without this, one engine's "creative" settlement could leave outstanding amounts negative or exceeding original, breaking AP/AR truth across the tenant. |

---

### UI-09 — Site ID Registry Resolution

| Field | Value |
|-------|-------|
| **Statement** | Every site-scope event references a `site_id` that exists in the tenant's **site registry**. Unknown site_ids are rejected at command or event dispatch. |
| **Scope** | site (dispatch) / tenant (registry lookup) |
| **Spans engines** | All universal engines with site-scope subscriptions; all vertical engines emitting site-scope events |
| **Enforcement** | CI schema validation (registry lookup as part of payload validation) + runtime (CN-5-101 §5 site-scope dispatch rejects on missing or unknown site_id) |
| **DC mapping** | DC-041 (Phase 0, CTR-025) |
| **Violation** | Hard fail at command/event emission |
| **Reasoning** | The site registry is a **tenant property** — set by the regional agent at onboarding (Term 2, Law 6), governed by Term 1 (CTR-027). Without registry resolution, a typo or stale reference would silently create a "ghost site" with its own per-site projections, splintering tenant truth. The registry makes site identity authoritative; UI-09 ensures every site-scope operation respects it. Pairs with CN-5-101 L2/L4 (site_id payload + dispatch tuple) and CTR-024 (verticals carry site_id). |

---

### UI-10 — Promotion Cost-Share Reconciliation

| Field | Value |
|-------|-------|
| **Statement** | When a promotion has cost split between platform, agent, and tenant (per D-001 cost-share), the associated cash-out events and accounting journal entries sum to the declared split: `bos_share + agent_share + tenant_share = total_discount` per promotion activation. |
| **Scope** | tenant (for the tenant-share leg) and platform (for the BOS/agent legs — Term 1 governance) |
| **Spans engines** | Promotion + Cash + Accounting + Platform billing (Term 1) |
| **Enforcement** | CI integration test (synthetic cost-share scenarios: emit promotion → settle → assert sums) |
| **DC mapping** | Phase 1 — pending CN-5-007 Promotion + Term 1 billing wiring (CTR-016) |
| **Violation** | Post-hoc anomaly alert; rebuild affected projections; investigation |
| **Reasoning** | A promotion whose declared cost split does not match its realised cash and accounting effects is one of three things: a bug (handler emitted wrong amounts), a misconfiguration (cost-share definition diverged from realisation), or fraud (a party adjusted figures after the fact). All three need detection; none can be silently absorbed. The invariant makes the three-way reconciliation a first-class system property. |

---

## 4. Per-Invariant Format

Every entry in §3 follows this format. Future invariants added via the living-catalog process (§8) MUST use the same shape:

| Section | Meaning |
|---------|---------|
| **Statement** | The truth as a single sentence (or short paragraph). What must be true about state. |
| **Scope** | The CN-5-101 level at which the invariant applies — `site` / `tenant` / `platform`. |
| **Spans engines** | The universal engines (and named other Terms' engines, if any) whose events or state participate. |
| **Enforcement** | The CN-4-019 enforcement layer(s) — runtime bus policy, CI integration test, CI schema validation, or a combination. |
| **DC mapping** | The CN-4-019 doctrine check ID if mechanized, or "Phase 1 — pending <CN-doc>" if waiting for an upstream concept doc. |
| **Violation** | The handling pattern (§6) — hard fail, anomaly alert, or mode escalation. Multi-pattern is allowed when an invariant behaves differently at different stages. |
| **Reasoning** | Two to four sentences: why the invariant must hold; what breaks if it doesn't; what it costs the system to maintain. |

---

## 5. Enforcement Strategy

CN-5-102 invariants enforce through the three CN-4-019 §3 check types. The choice is per-invariant and reflects how the violation manifests.

| Layer | Best for | Examples from §3 |
|-------|----------|------------------|
| **Runtime (bus policy)** | Invariants preventable at command time. The bus rejects emission of any event that would violate the invariant. | UI-01 (missing causation_id), UI-03 (missing source_ref), UI-05 (closed period), UI-06 (currency mismatch), UI-08 (obligation bounds), UI-09 (unknown site_id) |
| **CI integration test** | Invariants detectable only by replay or scenario walk — they involve state derived across engines or causation chains. | UI-02 (compensation symmetry), UI-04 (tender chain), UI-07 (trial balance), UI-10 (cost-share reconciliation) |
| **CI schema validation** | Invariants enforced by payload structure — required fields, registry lookups, enum constraints. | UI-01 (causation_id field presence), UI-03 (source_ref enum), UI-06 (currency code), UI-09 (registry lookup as schema check) |

Most invariants combine layers: UI-01 is schema (field present) + runtime (bus refuses derivative without causation); UI-05 is runtime (bus rejects late-effective-date command) + integration test (replay confirms no event landed in closed window).

The Architect phase (CN-4-019 §9) decides the tooling. CN-5-102 specifies what each invariant must verify; CN-4-019 records the DCs once accepted.

---

## 6. Violation Handling

Every invariant declares a violation pattern. CN-5-102 recognises three; an invariant may map to one or more depending on stage and detection layer.

| Pattern | When applied | Mechanism |
|---------|--------------|-----------|
| **Hard fail** | Preventable at command/emission time | The bus emits `kernel.command.rejected.v1` (CN-4-004 §4) with `rejected_by_policy = <UI-NN>` and a generic rejection reason. No violating event enters the store. The caller sees the rejection and corrects. |
| **Post-hoc anomaly alert** | Detectable only after emission — requires walking choreography, comparing balances, or detecting drift | Security primitives (CN-4-016) emit an anomaly alert with the invariant ID and the events implicated. Operators investigate. Correction follows the compensating-event pattern (CN-4-001 §3.1). |
| **Mode escalation** | Systemic violation suggesting structural data-integrity problem | Health-check dimension (CN-4-017 §5) escalates NORMAL → DEGRADED. The impaired dimension is the invariant ID. Operations may be deferred or restricted until the integrity issue is resolved. |

### Multi-Pattern Invariants

An invariant may behave differently at different stages:
- **UI-03** is hard-fail at emission for new events; for legacy stock-movement events present in the store before the invariant was added, it surfaces as anomaly alerts that operators address with compensating adjustments.
- **UI-07** is hard-fail at CI build time (catching new code that breaks balance); at runtime it escalates the system to DEGRADED if drift is detected during routine integration tests.

The invariant entry in §3 lists all applicable patterns.

### Why Not Silent Tolerance

Universal-engine state is the system's truth. A silently-tolerated violation accumulates: one orphan stock movement becomes ten, then a hundred, until reconciliation is impossible. Every violation either prevents the emission or surfaces as an alert that operators must close. There is no "accept and ignore" path.

---

## 7. DC Ramp-Up Plan

CN-5-102 introduces ten invariants; not all are mechanizable today. Four are mechanizable now without waiting for further concept docs; six wait for the per-engine docs that populate the choreography details they constrain.

### Phase 0 — Mechanizable Now (CTR-025 → Term 4)

| DC | Invariant | Category in CN-4-019 |
|----|-----------|----------------------|
| **DC-038** | UI-01 causation_id integrity on derivative events | Event Doctrine |
| **DC-039** | UI-03 inventory source-ref presence | Engine Contracts |
| **DC-040** | UI-06 tenant functional-currency invariant (extends DC-037) | Freeze & Documents |
| **DC-041** | UI-09 site_id registry lookup | Isolation / Scope |

CTR-025 is OPEN with Term 4. Once accepted and merged, the CN-4-019 catalog increments from 37 to 41 doctrine checks, with appropriate updates to the Type and Law-mapping tables.

### Phase 1 — Pending Per-Engine Docs

| Invariant | Waits for |
|-----------|-----------|
| **UI-02** Compensation symmetry | CN-5-001 (Accounting) + CN-5-104 (Period Close) |
| **UI-04** Tender → origin chain | CN-5-002 (Cash) + CN-5-009 (Checkout/Tender) |
| **UI-05** Closed-period inviolability | CN-5-001 (Accounting) + CN-5-104 (Period Close) |
| **UI-07** Trial balance | CN-5-001 (Accounting) |
| **UI-08** Obligation bounds | CN-5-004 (Procurement) + CN-5-007 (Promotion) |
| **UI-10** Promotion cost-share reconciliation | CN-5-007 (Promotion) + CTR-016 (Term 1 billing) |

Each Phase 1 invariant will be mechanized through its own CTR to Term 4 once the upstream engine doc is ratified.

### Phase 2 — Future (Open)

If a Term 5 engine doc identifies a cross-engine invariant not in this catalog, the living-catalog process (§8) adds it. Each addition follows the same per-invariant format and earns or defers a DC accordingly.

---

## 8. Living-Catalog Discipline

CN-5-102 mirrors CN-4-019 §5 for the universal-engine layer:

| Rule | Meaning |
|------|---------|
| **Adding an invariant** | A new invariant is added to §3 through normal collaboration (discuss-then-write; Concept Lead and Overseer review). Once accepted, a CTR to Term 4 may follow to mechanize as a DC. |
| **Modifying or removing an invariant** | Requires Concept Lead or governance authority approval. Audited as a doctrine-affecting change. Universal-engine state truths do not weaken silently. |
| **Invariant IDs are stable** | UI-NN IDs are not reused. A retired invariant is marked deprecated, not reassigned. |

---

## 9. Example — Period Close Across Three Engines

*Mama Amina's duka serves as illustrative context, per D-004 #4. The example shows the invariants holding together; it is not a UX description.*

Mama Amina's accountant Mzee Hassan closes April 2026. Accounting emits `accounting.period.closed.v1 { period: 2026-04, tenant: mama-amina-duka }`.

### A Late Sale Attempt (UI-05 + UI-09)

The morning of May 3, Cashier A at the Mwanza site tries to record a late sale with `effective_date: 2026-04-30`. The Checkout engine submits `checkout.settle.request`:

- The bus runs engine policies. The Accounting engine policy implementing **UI-05** queries Accounting's period-state projection: April 2026 is closed for `mama-amina-duka`. The command is rejected:
  ```
  event_type:           kernel.command.rejected.v1
  rejected_by_policy:   UI-05.closed_period_inviolable
  rejection_reason:     "Period 2026-04 is closed for this tenant"
  correlation_id:       <command id>
  ```
- Cashier A sees the rejection. He resubmits with `effective_date: 2026-05-03`. The bus, with no UI-05 violation, evaluates the rest of the policies. **UI-09** verifies the payload `site_id = mama-amina-mwanza-001` is in the tenant's site registry — yes (registered via Term 2 onboarding). The sale settles.

### Trial-Balance Audit (UI-07)

Mzee Hassan asks BOS to confirm the April books are sound. A point-in-time replay (CN-4-009) reconstructs Accounting's state at the close-of-period `global_position`. **UI-07**'s integration test sums all journal debits and credits for `mama-amina-duka`: TZS 4,230,000 = TZS 4,230,000. Balanced. The report includes the assertion; the hash chain (CN-4-003) proves the events were not tampered with. Mzee Hassan signs off.

### A Correction Pattern (UI-05 + UI-02)

Mzee Hassan discovers that an April sale recorded TZS 12,000 when it should have been TZS 15,000. The correction cannot land in April (UI-05). Instead:

- A new event is emitted in the current open period (May): `accounting.correction.recorded.v1 { causation_id: <april-sale-event-id>, original_amount: 12000, correct_amount: 15000, correction_journal: ... }`. The May journal posts the +TZS 3,000 difference with full causal lineage back to the April event being corrected.
- **UI-02** (compensation symmetry): because the original April sale propagated to Cash and Inventory, the correction's effects propagate too — but only into May, never retroactively into April. Cash's May projection reflects the +TZS 3,000; Inventory is unaffected (the items moved correctly; only the price was wrong). The choreography of correction is symmetrical with respect to the engines that participated, but **forward-only with respect to time**.

### A Multi-Currency Procurement (UI-06)

Mama Amina imports a small consignment from a Kenyan supplier billed in KES. Procurement emits `procurement.invoice.recorded.v1 { amount: 50000, currency: KES }`. Tenant functional currency is TZS:

- **UI-06**'s bus policy detects the non-functional currency. The command is allowed because Procurement also emits a paired `procurement.fx.recorded.v1 { source: 50000 KES, target: 850000 TZS, fx_rate: 17.0, fx_event_date: 2026-05-12 }` as part of the same submission. The functional-currency invariant holds: every primary financial effect lands in TZS; the KES amount is bridged through an explicit FX event with its own pack-version reference (D-009).
- Without the paired FX event, UI-06 would have rejected the command. Multi-currency is honoured; silent unit mixing is not.

Across all four scenarios, no invariant required Mama Amina or Mzee Hassan to know about it. The doctrine watches the state shape; the operators run the business.

---

## 10. Three Whys

### Why does this matter?

A universal-engine system without cross-engine invariants is a collection of well-behaved engines that drift apart in aggregate. Each engine's own tests pass; the system as a whole produces incoherent state — books that don't balance, stock that has no source, cash that arrived from nowhere, promotions that cost more than they were funded for. Universal-engine invariants are how seven engines stay one system. They are the cross-engine analogue of CN-4-019's Kernel guarantees.

### Why does it belong here (and not in Foundation)?

Foundation (CN-4-019) defines Kernel invariants — isolation, immutability, advisory-only, replay determinism. Those are engine-agnostic. The invariants in CN-5-102 require knowing what universal engines do together: that Accounting subscribes to Checkout, that Cash subscribes to Checkout, that promotions affect both. Foundation does not know that and should not. CN-5-102 is the Term 5 doctrine layer; Foundation provides the mechanism (CN-4-019 catalog, runtime + CI enforcement, living-catalog rules) and CN-5-102 populates the universal-engine portion.

### Why this design?

A small initial catalog (ten) with a clear per-invariant format and a phased ramp-up via CTRs keeps the work tractable. Phase 0 mechanizes what is mechanizable now without waiting; Phase 1 waits for engine docs that supply the choreography details; the living-catalog process ensures the catalog grows alongside the system rather than being frozen at concept time. Three violation patterns (hard fail, anomaly, mode escalation) cover the spectrum of detectability without forcing every invariant into a single shape. Inheritance from CN-4-019, CN-4-011, CN-5-100, and CN-5-101 prevents doctrine sprawl — CN-5-102 only adds what the others do not already say.

---

## 11. Boundaries

| Topic | Lives in |
|-------|----------|
| Doctrine-check mechanics (gate, attestation, no-override) | CN-4-023 |
| Doctrine-check catalog (the DCs themselves) | CN-4-019 |
| Primitive folds (per-primitive bounds and compensation) | CN-4-011 |
| Subscription patterns and laws | CN-5-100 |
| Scope policy (levels, runtime resolution) | CN-5-101 |
| Per-engine business rules (sale has items > 0, etc.) | per-engine docs (CN-5-001 … CN-5-007, CN-5-009, CN-5-010) |
| Period-close choreography mechanics (state machine, event sequence) | CN-5-104 |
| Tax-aware engine wiring (compliance pack lookup at command time) | CN-5-105 |
| Tenant site registry (definition + governance) | Term 1 (governance), Term 2 (population), CTR-027 |
| Tenant functional currency registry | Term 1, CTR-027 |
| Anomaly detection mechanism | CN-4-016 |
| Mode escalation mechanism | CN-4-017 |

---

## 12. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Phase 0 doctrine checks accepted into CN-4-019 | Term 4 via CTR-025 | OPEN — four DCs (DC-038..041) await Term 4 acceptance |
| Verticals respect UI-03 source_ref + UI-09 site_id discipline | Term 6 via CTR-026 | OPEN — universal-engine invariants depend on emission-boundary compliance |
| Tenant site registry + functional currency registry | Term 1 (governance) + Term 2 (population) via CTR-027 | OPEN — UI-06 and UI-09 require these registries to be authoritative |
| Phase 1 mechanization CTRs (UI-02, UI-04, UI-05, UI-07, UI-08, UI-10) | Future — per engine doc | Each will be filed as its upstream engine doc (CN-5-001, CN-5-002, etc.) is ratified |
| Anomaly-alert routing for post-hoc invariants | Architect phase + CN-4-016 | Concept (§6) specifies the violation pattern; CN-4-016 defines the alert mechanism |
| Test fixtures for cross-engine integration checks (synthetic multi-engine scenarios) | Architect phase | Per CN-4-019 §9; concept requires the checks, Architect produces the data |
| Catalog naming for future invariants spanning Term 6 verticals (do they become UI-NN or a separate series?) | Concept Lead + Overseer | Pending — current catalog is universal-engine-only; vertical-spanning invariants may need their own prefix |

---

*— End of CN-5-102 —*
