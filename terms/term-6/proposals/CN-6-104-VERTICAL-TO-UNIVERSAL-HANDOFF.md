# CN-6-104 — Vertical-to-Universal Hand-Off Pattern

> **Term:** 6 — Vertical Engines
> **Status:** Concept Phase v1.0 — Draft for Overseer review
> **Branch:** `claude/brave-johnson-S7Lky`
> **Reading order:** CN-5-009 (Universal Checkout) → CN-5-100 (Subscription Wiring) → CN-5-103 (G1–G9) → CN-6-100 §3 + §6 (Recipe + payload conformance) → CN-6-101 BD5 → CN-6-102 NC3 + NC9 → CN-6-103 SP1-SP8 → **this doc**
> **Ordering authority:** TERM-6 Brief §13 — fifth Term 6 deliverable; **last cross-cutting framework doc before concrete verticals CN-6-001..004**.

---

## 1. Purpose & Boundary

### 1.1 Mission

CN-6-104 defines the **handshake**: how a vertical emits events that universal engines consume, and how universal engines emit signals that verticals consume back. Brief §6.2 calls it "the vertical-to-universal hand-off" — the pattern by which a sale at Salma's till, a check-in at Kilimanjaro Lodge Moshi, a cut on Mzee Hassan's workshop floor, or a dispensing at Faraja's pharmacy counter **becomes** authoritative books-of-record across Accounting, Cash, Inventory, Procurement, HR, Reporting, Promotion, and Checkout.

CN-5-100 owns the universal-side subscription wiring. CN-6-104 owns the **vertical-side discipline at the boundary** — what payload to emit, when to expect settlement back, how to handle rejection, how UI-01 causation flows, and how Foundation primitives (Obligation, Party, Document) carry cross-vertical relationships that BD7 prevents from becoming bridge engines.

This is the closing doc of the cross-cutting cluster. After merge, Term 6 begins CN-6-001 Retail with the framework complete.

### 1.2 DOES vs DOES NOT

| CN-6-104 DOES | CN-6-104 DOES NOT |
|---------------|-------------------|
| Define HO1–HO9 vertical-side handoff doctrine | Author universal-side subscription wiring (CN-5-100) |
| Specify the canonical `<vertical>.bill.ready.v1` → Universal Checkout handshake | Specify the Saleable Line / Tender value shapes (CN-4-021) |
| Catalog the eight universal engines + Advisor wiring per vertical handoff | Author concrete per-vertical event vocabularies (CN-6-001..004) |
| Specify UI-01 causation chain discipline across handoffs | Define UI-01 invariant mechanism (CN-5-102) |
| Specify failure-mode recovery patterns (rejection, walk-out, orphan settlement) | Specify pack-rule content (Term 1 + jurisdiction packs) |
| Document the canonical settlement-back signal (HO9) + situational universal-to-vertical signals | Document specific cross-vertical bridges (CN-6-005) |
| Establish Obligation-as-universal-handoff doctrine for cross-vertical relationships | Enumerate cross-vertical bridge instances (CN-6-005 future) |
| Hand off the cross-cutting cluster to concrete verticals (§13) | Author CN-6-001..004 — those are next, separately |

### 1.3 Audience

Term 6 authors writing CN-6-001..004 (immediate consumers); Architects implementing vertical engines; Term 7 reviewing handoff-relevant PRs; Term 5 confirming subscription contracts honour vertical-side discipline; future vertical contributors needing the canonical template.

### 1.4 Charter Compliance

| Law | How CN-6-104 honours it |
|-----|--------------------------|
| Law 1 — State from events only | HO3 + HO6 — handoff is event emission, no side channel; Law 1 immutability per Q6 (no post-acceptance retry) |
| Law 2 — Engines isolated | HO8 + BD7 + §10 — Obligation primitive carries cross-vertical, no direct subscription |
| Law 3 — AI advisory only | §5 Advisor row — advisors subscribe to read-only projections; never autonomous (CTR-010) |
| Law 4 — Flexibility first-class | HO5 fan-out — any future universal engine subscribing to existing bill.ready works without vertical changes |
| Law 5 — Compliance configured | N3 walk-out timeout per pack; Q3 Pattern A/B per pack hook |
| Law 6 — Distribution regional | Pack-driven handoff parameters; jurisdiction binding via CTR-027 |

### 1.5 Parsimony — handoff is where business becomes truth

When Salma scans a tin of cooking oil at Mama Amina's till and presses the M-Pesa receipt, the handoff happens silently: `retail.bill.ready.v1` emits, Universal Checkout settles via the M-Pesa adapter, and within seconds — Accounting has projected the revenue, Cash has logged the till deposit, Inventory has deducted the stock, Promotion has updated Mama Amina's loyalty book, Reporting has fed the day's dashboard. Salma sees a receipt print. Mama Amina sees "today's revenue" tick upward on her phone. None of them see the handoff. **That is the point.**

The handoff is the deepest plumbing in BOS — the place where a business event becomes books-of-record. If the handoff misfires, none of the upstream good (Mama Amina's trust in her numbers, Faraja's TFDA compliance, Mzee Hassan's project audit) holds. If the handoff is parsimonious, every layer works.

HO1–HO9 below are the rules that keep the plumbing tight. Each is justified against the question: "does this match how the real cashier, real pharmacist, real fundi, real receptionist, real freight dispatcher experiences the moment of transaction?"

---

## 2. Inputs and Relationship

### 2.1 Parent doctrine

- **CN-5-009** Universal Checkout — the canonical handoff consumer; K1/K2 idempotency; tender flow post-handoff; receipt issuance
- **CN-5-100** Subscription Wiring Patterns A/B/C — universal-side mechanism CN-6-104 hands into
- **CN-5-101** Scope Policy — handoff respects vertical scope (CN-6-103 SP1-SP8)
- **CN-5-102 UI-01** causation chain integrity; UI-09 site_id resolution
- **CN-5-103 G1-G9** naming compliance at handoff event
- **CN-4-021** Saleable Line + Tender value shapes
- **CN-4-011** Foundation primitives (Obligation for cross-vertical; Party for customer; Workflow for state)

### 2.2 Term 6 upstream

- **CN-6-100 §3 + §6 + VE4** — recipe step "emit bill.ready"; mandatory emission; CTR-024/026/030 payload conformance
- **CN-6-101 BD5** — universal mechanism + vertical context (HO2 = handoff-layer split-it)
- **CN-6-101 BD7** — no bridge engines; Obligation-mediated cross-vertical (§10)
- **CN-6-102 NC3** — bill.ready uniformity (`<vertical>.bill.ready.v1` exact 3-segment)
- **CN-6-102 NC9** — compensation pair declaration at registration
- **CN-6-103 SP1-SP8** — scope at handoff boundary; SP5 site_id mandatory; SP7 forward-only migration

### 2.3 CTRs cited (no new)

- **CTR-018, CTR-002** (registration; verticals feed lines pattern)
- **CTR-024** (site_id payload); **CTR-026** (UI-03 + UI-09); **CTR-030** (Accounting payload sufficiency)
- **CTR-038** (namespace reservation)
- **CTR-044, CTR-045** pending upstream context
- **CTR-046** — DC-NN-f (HO9 settlement-back-subscription) queued for next amendment alongside DC-NN-e (SP8 from CN-6-103)

### 2.4 Sibling boundary

CN-6-005 (Vertical Bridges, future) catalogues specific cross-vertical bridge patterns — CN-6-104 §10 establishes the Obligation-as-universal-handoff doctrine; CN-6-005 enumerates instances (charge-to-room, sell-via-POS, in-stay dining). Light, intentional overlap.

CN-6-001..004 (concrete verticals, next) consume this doc directly — each declares its handoff manifest per HO1-HO9.

---

## 3. Doctrine — HO1–HO9

**HO1 — The canonical handoff is `<vertical>.bill.ready.v1` → Universal Checkout.** Every vertical that sells, charges, or invoices emits this exact event. No exceptions. Insurance (premium charge), Workshop (advance + balance via Obligation), recurring Pharmacy auto-refill (per-cycle Workflow), Logistics (trip-close bill) — all emit bill.ready, varying only `saleable_lines` and timing.

**HO2 — Vertical owns bill content; universal owns tender.** The vertical decides *what is payable* (`saleable_lines[]`, `discount_refs[]`, `tax_treatment_ref` per line). Universal Checkout decides *how it is paid* (cash, M-Pesa, card, split-tender, credit-via-Obligation), computes change, applies pack-driven tax computation, issues the receipt. A vertical that knows "M-Pesa" or "tender_method" inside its code is misdesigned (VE4 violation).

**HO3 — Handoff is one-way command emission; downstream consumption is asynchronous.** The vertical emits via the command bus (CN-4-004) and receives a **synchronous command-bus accept-or-reject** result (UI-09 site_id validation, payload contract checks per CTR-024/026/030 happen at acceptance). The vertical does **not** await downstream universal subscription outcomes (Accounting projecting, Inventory deducting, Promotion adjudicating) — those fire independently per CN-5-100.

**HO4 — UI-01 causation chain spans every handoff.** The universal event's `causation_id` is the vertical handoff event's `event_id`; the vertical handoff event's `causation_id` chains back to the originating workflow event; the workflow event chains back to the originating customer command. Audit walks backward from any artifact (receipt, journal entry, inventory movement) to the originating human action.

**HO5 — Multiple universal engines may consume the same vertical event (fan-out per CN-5-100 P1).** A single `restaurant.bill.ready.v1` is consumed by Checkout (tender), Accounting (revenue projection), Inventory (Pattern A consumption derivation OR Pattern B trigger), Promotion (ROI + cost-share + loyalty), Reporting (KPI feed), and Advisor (read-only projection feed). Vertical emits once; universals subscribe independently per their own manifests.

**HO6 — Bidirectional handoff has one canonical pattern + situational patterns.**
- **Canonical (universal-to-vertical):** `checkout.settled.v1` → vertical Workflow finalization. Mandatory subscription per HO9 for every billable Workflow.
- **Situational:** `accounting.period.closed.v1` (vertical may freeze Workflow per pack rule); `pack.effective.v1` (vertical may re-pin per D-009 freeze); `inventory.stock.depleted.v1` (vertical may transition Workflow to `blocked` state per CN-6-100 §3.13).

Vertical never *initiates* dialogue with universals; only subscribes to declared signals.

**HO7 — Compensation patterns at handoff failure are explicit per failure class.** Universal-rejects-payload (UI-09 unknown site_id, missing tax_treatment_ref): `<vertical>.bill.recalled.v1` + Workflow rollback to pre-bill state. Customer-walks-out post-emission, pre-settlement: pack-rule timeout (§8 N3 defaults) → recall OR convert to Obligation (`kind: unpaid_walkout`). Vertical Workflow cancels pre-billing: no handoff needed; terminal `cancelled` state. NC9 compensation_pair declarations cover each.

**HO8 — Vertical never reads other engines' state; reads only Foundation primitive projections and its own engine projections.** Permitted reads:
- Foundation primitive projections (Party, Document, Obligation, Workflow, Consent, Identity)
- The vertical's own engine projections (its own events, replayed)

Prohibited reads:
- Other universal engines' state (Accounting balances, Cash totals, Inventory levels — those flow through events the vertical subscribes to, not direct reads)
- Other verticals' state (VE2 isolation)
- Cross-tenant state (Law 2 isolation)

If a vertical thinks it needs to "look at Inventory's current stock level," it subscribes to `inventory.stock.depleted.v1` per HO6 situational signals — not direct state read.

**HO9 — Settlement signal subscription is mandatory for billable workflows.** Every vertical Workflow that emits `<vertical>.bill.ready.v1` MUST declare a settlement-back subscription in its Workflow primitive instance:

```
workflow_instances:
  - workflow_id: <vertical>.<workflow_name>
    lifecycle_states: [...]
    billable: true                                  # if false, HO9 N/A
    settlement_subscription:
      event_type: checkout.settled.v1
      filter: originating_workflow_ref == <self>
      transitions_to: <terminal_success_state>      # e.g., closed, archived, served
```

The doctrine gate at CN-4-020 registration verifies: any manifest declaring `billable: true` Workflow without `settlement_subscription` is rejected. Without HO9, billable workflows stall at "bill_emitted" indefinitely — the recurring root cause of the gaps Brief §12 cites. Mechanizable: DC-NN-f queued for CTR-046 next amendment cycle alongside DC-NN-e (SP8 subscription-side from CN-6-103).

---

## 4. The Canonical Handoff — `<vertical>.bill.ready.v1` → Universal Checkout

### 4.1 Why this is canonical

Every business sells, charges, or invoices. The bill is the moment a business event becomes claim against a customer. Universal Checkout converts that claim into money + receipt. This is the unique cross-vertical universal mechanism that every existing and future vertical needs — exactly once per Workflow, exactly the same shape.

### 4.2 The event shape (CN-4-021 + CN-6-102 NC3)

```
<vertical>.bill.ready.v1
{
  bill_id,                          # vertical-internal identifier
  site_id,                          # SP5 + CTR-024 (resolves UI-09 registry)
  saleable_lines: [                 # array of CN-4-021 Saleable Lines
    {
      line_id,
      item_ref,
      description,
      quantity,
      unit_price,                   # pre-tax
      tax_treatment_ref,            # pack lookup result; never inline rate (CN-5-105)
      discount_refs: [],            # promotion_intent OR promotion_ref (HO2 + Q2)
      source_tag,                   # opaque vertical tag (audit only)
      line_total                    # non-authoritative cache; K2 recomputes
    },
    ...
  ],
  originating_workflow_ref,         # VE7 Workflow primitive ID — filter target for HO9
  business_date,                    # CN-5-105 tax period attribution
  payer_party_ref?,                 # Foundation Party primitive ID (optional)
  obligation_refs: [],              # if bill resolves outstanding Obligations (§10)
  expansion_mode_per_line: [        # Pattern A vs B per line (Q3)
    { line_id, mode: "auto" | "vertical_managed" }
  ]
}
```

### 4.3 The flow

1. Vertical Workflow reaches its bill-ready state (sale completed, table session closed, folio ready, project completed, prescription dispensed, trip arrived).
2. Vertical emits `<vertical>.bill.ready.v1` via command bus.
3. Command bus validates payload (site_id resolution UI-09, tax_treatment_ref presence CN-5-105, payload sufficiency CTR-030). Validation pass → event into store.
4. Universal Checkout subscribes by canonical pattern `*.bill.ready.v1`; receives the event.
5. Checkout K2 recomputes line totals + tax per pack (authoritative); presents tender choices to cashier.
6. Tender resolves (cash, M-Pesa, card, split, credit-via-Obligation per HO6 cross-vertical).
7. `checkout.settled.v1` emits carrying `originating_workflow_ref`.
8. Vertical's HO9 subscription fires; Workflow transitions to terminal success state.
9. Fan-out (HO5): Accounting projects revenue; Cash logs receipt; Inventory deducts (Pattern A) or awaits vertical Pattern B emission; Promotion records loyalty; Reporting updates KPI; Advisor projections refresh.

Steps 3–9 happen within seconds. Salma, Mama Amina, Faraja, Mzee Hassan, the Lodge Serengeti GM — none of them see the choreography. They see the receipt and the updated dashboard.

---

## 5. Per-Universal-Engine Handoff Catalog

The reference table CN-6-001..004 authors copy from. Each row specifies what vertical emits + how universal consumes + payload requirements.

| Universal Engine | Vertical Emits | Handoff Pattern | Payload Requirements |
|------------------|-----------------|-----------------|----------------------|
| **Checkout (CN-5-009)** | `<vertical>.bill.ready.v1` | Canonical; one-way emit + HO9 settlement-back | saleable_lines + site_id + originating_workflow_ref + payer_party_ref + business_date + tax_treatment_ref per line |
| **Accounting (CN-5-001)** | `<vertical>.bill.ready.v1` + workflow lifecycle events | Projection consumer per CTR-030 | Recognition timing + tax_treatment_ref + chart_of_accounts_hint + cogs_basis per line |
| **Cash (CN-5-002)** | **INDIRECT** — no direct vertical handoff | Cash subscribes to `checkout.settled.v1` + tender events; vertical does NOT emit to Cash directly | (no direct vertical handoff — verticals must not emit to Cash) |
| **Inventory (CN-5-003)** | Pattern A: derived from Checkout settlement; Pattern B: `<vertical>.<consumed_kind>.consumed.v1` direct | Per pack hook `pack.<vertical>.inventory_expansion_mode.<item_category>` ∈ {auto, vertical_managed} (Q3) | Pattern B payload: source_ref + qty + site_id + item_ref + consumed_at |
| **Procurement (CN-5-004)** | `<vertical>.purchase_need.recorded.v1` (optional per CTR-030) | Trigger event Procurement may subscribe to | item_ref(s) + qty + target_delivery_date + justification + originating_workflow_ref |
| **HR/Payroll (CN-5-005)** | `<vertical>.commission_earned.v1`, `<vertical>.piece_completed.v1` | Trigger events for payroll computation per CTR-030 | employee_ref + amount/count + period + source_ref |
| **Reporting (CN-5-006)** | Any vertical event | Massive fan-in; vertical never tailors emission for Reporting (CN-5-006 N3 — read-side only) | (no special Reporting fields; existing payloads sufficient) |
| **Promotion (CN-5-007)** | `<vertical>.bill.ready.v1` + customer-interaction events | Subscribes for ROI + cost-share + loyalty per CN-5-007 §16 expansion | party_ref + saleable_lines + promotion_refs (if pre-emitted) |
| **Advisor (CN-4-022 + CN-5-010)** | Vertical emits normal events; advisors subscribe to read-only projections | One-way emission; advisor subscriptions per CN-5-010 audience contract; **NO autonomous action** (Law 3 + CTR-010) | (no special advisor fields; advisors read from projections, not from vertical-tailored payloads) |

### 5.1 Cash indirection — important

Cash NEVER receives a direct vertical handoff. A vertical that emits `<vertical>.cash.deposit_requested.v1` (or any event Cash would consume directly) violates VE4 + HO2 — the vertical is trying to own tender. Cash receives:

- `checkout.settled.v1` from Universal Checkout (per tender resolution)
- Tender outcome events from Term 7 adapters (per CTR-006 + CTR-031)

Verticals never participate in this flow. If a vertical author thinks "my vertical needs to tell Cash X," the answer is always: emit at the bill.ready boundary; Cash subscribes to Checkout downstream.

### 5.2 Advisor indirection — important (Law 3 affirmed)

Advisors subscribe to vertical event projections in **read-only** mode per CN-5-010. They never receive command-bus events directly; they observe via projection (CN-4-010). They never write events; they only produce suggestion entries in the Decision Journal (CN-4-013). Their output is human-mediated per Law 3 + CTR-010.

A vertical that "calls" an advisor synchronously is misdesigned — advisors are projection consumers, not RPC endpoints. A vertical that subscribes to `kernel.advisor.*` events to act on advisor suggestions is also misdesigned — humans act on suggestions, not engines.

---

## 6. UI-01 Causation Chain Across Handoff

### 6.1 The chain

Every event carries `event_id` + `causation_id`. The causation_id points to the immediate predecessor that triggered this event. Walking the chain backwards reconstructs the full audit path.

### 6.2 Example chain — Mama Amina retail sale

```
event_id: evt-7732    (customer-command: retail.sale.complete.request)
                      | actor: salma@kariakoo; party_ref: <customer>
                      ▼
event_id: evt-7733    (retail.sale.completed.v1)
causation_id: evt-7732
                      | source_workflow_ref: <sale-workflow>; site_id, total
                      ▼
event_id: evt-7734    (retail.bill.ready.v1)
causation_id: evt-7733
                      | saleable_lines, site_id, originating_workflow_ref, business_date
                      ▼  [Universal Checkout subscribes]
event_id: evt-7735    (checkout.tender.requested.v1)
causation_id: evt-7734
                      | tender_method: m_pesa, callback_correlation_id
                      ▼  [Term 7 M-Pesa adapter]
event_id: evt-7736    (m_pesa.tender.outcome.v1)
causation_id: evt-7735
                      | outcome: confirmed, external_ref: <mpesa-txn-id>
                      ▼
event_id: evt-7737    (checkout.settled.v1)
causation_id: evt-7736
                      | originating_workflow_ref → triggers HO9 subscription
                      ▼  [Vertical's HO9 subscription]
event_id: evt-7738    (retail.sale.archived.v1)
causation_id: evt-7737
                      | Workflow terminal success
```

Six events, all chained, all replayable. Salma's command at evt-7732 is the originating action. Mama Amina's auditor can walk from receipt back to till. This is Charter §1.2 legal-defensibility in concrete form.

### 6.3 Fan-out branches

Fan-out (HO5) creates parallel branches from a single event. From evt-7734 (the bill.ready), multiple universal events branch:

```
evt-7734 (retail.bill.ready.v1)
├── evt-7735 (checkout.tender.requested.v1)        [causation: 7734]
├── evt-7740 (accounting.journal.posted.v1)         [causation: 7734]
├── evt-7741 (inventory.stock.deducted.v1)          [causation: 7734]  (Pattern A)
├── evt-7742 (promotion.loyalty.earned.v1)          [causation: 7734]
└── evt-7743 (reporting.kpi.updated.v1)             [causation: 7734]
```

Each downstream branch is independently auditable. Reporting's evt-7743 may have its own children (KPI projections, dashboard refreshes); each carries causation back to evt-7734 which carries back to evt-7733 → evt-7732. The tree is a forest of audit paths, all rooted at the originating command.

---

## 7. Multi-Universal Fan-Out (HO5)

### 7.1 The pattern

A single vertical event triggers independent processing in multiple universal engines. Each universal subscribes per its own manifest; vertical does not know who subscribes. New universal engines (or new subscriptions) can be added without vertical changes (VE6 + Law 4).

### 7.2 Fan-out at Lodge Serengeti restaurant bill.ready (preview WP1)

```
restaurant.bill.ready.v1
├── Checkout (CN-5-009):       tender flow → checkout.settled.v1
├── Accounting (CN-5-001):     revenue projection → accounting.journal.posted.v1
├── Inventory (CN-5-003):      Pattern B awaiting restaurant.ingredient.consumed.v1
├── Promotion (CN-5-007):      ROI + happy-hour cost-share → promotion.cost_share.recorded.v1
├── Reporting (CN-5-006):      KPI update → reporting.kpi.updated.v1 (table-turnover, RevPAR-equivalent)
└── Advisor (CN-5-010):        restaurant-floor-advisor projection refresh; suggestion eligibility re-check
```

Six independent processing paths from one vertical emission. Each runs at its own latency. The vertical does not block on any.

### 7.3 Adding a new universal engine

If, after CN-6-104 is in production, a new universal engine — say, a Sustainability Reporting engine — wants to subscribe to all bill.ready events to compute carbon footprint per transaction, it does so by adding its subscription to its own manifest. **Verticals do not change.** This is VE6 (closed for modification, open for extension) realised at the handoff layer.

---

## 8. Failure Modes + Compensation Pattern

### 8.1 The catalog

| Failure | Cause | Recovery |
|---------|-------|----------|
| Universal rejects bill.ready (UI-09 unknown site_id, missing tax_treatment_ref, payload contract violation) | Vertical emitted nonconformant payload | Command bus emits `kernel.command.rejected.v1` to vertical's subscription on its own commands; vertical Workflow transitions to `blocked` state per CN-6-100 §3.13; remediation by author/operator; re-emission with corrected payload uses new command_id (no retry of rejected command_id) |
| Downstream consumer fails (Accounting projection error, Inventory deduction logic bug) | Universal engine internal bug | Universal engine handles internally (dead-letter, retry, doctrine event); vertical is not involved. The bill.ready emission is already accepted; downstream consumer issues do not roll back the vertical. |
| Vertical emits malformed payload (CTR-024/026/030 violation) | Vertical author bug | Bus rejects at command-time; same as row 1; Workflow blocked |
| Vertical Workflow cancels pre-billing (customer leaves before order placed; reservation cancelled before check-in) | Business event | No handoff needed; Workflow → cancelled terminal state per NC9 compensation_pair |
| **Customer walks out post-bill.ready, pre-settlement** (Brief §7.2 restaurant walk-out; Brief §7.3 hotel no-pay-and-leave) | Business event; not system failure | Vertical Workflow waits for settlement signal up to `pack.<vertical>.bill_settlement_timeout_hours` (N3 defaults below); on timeout, vertical emits `<vertical>.bill.recalled.v1` OR converts outstanding to Obligation per `pack.<vertical>.bill_settlement_timeout_action` ∈ {recall, convert_to_obligation} |
| **Orphan settlement** (checkout.settled.v1 with no preceding bill.ready) | Bus K1 idempotency should prevent; if observed = bug | Universal Checkout K1 idempotency rejects at acceptance; if it slips through, doctrine-violation event emits per CN-4-019 monitoring; should never occur in production |

### 8.2 N3 walk-out timeout pack defaults

Pack rule `pack.<vertical>.bill_settlement_timeout_hours` per vertical, default values reflecting business reality:

| Vertical | Default timeout | Rationale |
|----------|------------------|-----------|
| Restaurant | 0.5 hours | Diner pays at table or walks out; immediate resolution |
| Hotel | 24 hours | Post-checkout settlement window; folio disputes resolve same day |
| Workshop | 168 hours (1 week) | Project handoff to customer for review/acceptance; longer pickup window |
| Pharmacy | 0 hours (immediate) | Dispensing is point-of-transaction; no walk-out window |
| Logistics | 720 hours (30 days) | B2B freight customers settle on invoice net-terms |
| Retail | 0 hours (immediate) | POS is point-of-transaction |

Pack rule `pack.<vertical>.bill_settlement_timeout_action` ∈ {recall, convert_to_obligation} determines recovery branch:

- **recall** (Restaurant default, Retail default): emit `<vertical>.bill.recalled.v1`; Workflow → `voided` terminal state; original bill.ready is event-store record but downstream universals treat as recalled
- **convert_to_obligation** (Workshop default, Hotel default for post-checkout disputes, Logistics default): outstanding balance becomes Obligation primitive instance (`kind: unpaid_walkout`, debtor party_ref, creditor vertical); collection mechanism per pack

Per-tenant overrides allowed within pack-declared range.

### 8.3 The walk-out scenario at Lodge Serengeti restaurant

A group dines, the bill emits, mid-settlement someone steps outside to "make a call" and never returns. The waiter waits 30 minutes (Restaurant default). At 30 minutes, the Workflow times out. Per pack default `recall`:

```
restaurant.bill.recalled.v1 {
  bill_id: <original>,
  reason: "walk_out_timeout",
  business_date,
  actor: system:scheduler
}
restaurant.table_session.voided.v1 {
  session_id,
  reason: "walk_out",
  outstanding_amount_recovered: 0,
  ...
}
```

Accounting sees the recall and reverses its provisional revenue projection. The table is freed for the next service. Lodge Serengeti's monthly P&L shows the loss in a "walk-outs" line per pack chart of accounts. No data is lost; the original bill.ready remains in the event store with its recall compensation chained backwards.

---

## 9. Bidirectional Handoff — Canonical Settlement-Back + Situational Signals (HO6 + HO9)

### 9.1 HO9 canonical settlement-back

Per HO9 (§3), every billable Workflow declares its settlement subscription. The Workflow primitive instance carries:

```
workflow_instances:
  - workflow_id: restaurant.table_session
    lifecycle_states: [opened, ordering, kitchen, served, billed, closed, voided]
    billable: true
    settlement_subscription:
      event_type: checkout.settled.v1
      filter: originating_workflow_ref == <self>
      transitions_to: closed
```

When `checkout.settled.v1` arrives with matching `originating_workflow_ref`, the Workflow primitive automatically transitions to the declared terminal state. Vertical author does not write handler code — the primitive handles the transition declaratively.

### 9.2 Situational signals

Beyond settlement, verticals may subscribe to other universal signals per pack rule:

| Universal Signal | Vertical Reaction | When |
|------------------|--------------------|------|
| `accounting.period.closed.v1` | Vertical may freeze Workflows in `audit_locked` state | Pack rule for verticals with multi-period workflows (Workshop long projects, Hotel multi-month reservations) |
| `pack.effective.v1` | Vertical may re-pin Workflows to new pack version | Per D-009 freeze — new tax rate, new chart-of-accounts |
| `inventory.stock.depleted.v1` | Vertical may transition Workflow to `blocked` state per CN-6-100 §3.13 | Workshop awaiting material; Restaurant 86'd ingredient |
| `pack.advisor.activation.changed.v1` | Vertical may emit refreshed projections for new advisor scope | Per CN-5-010 advisor activation governance |

Each subscription is declared in the vertical manifest with explicit filter + reaction. The doctrine gate verifies (CN-4-020).

### 9.3 The constraint — verticals never initiate

Critically, verticals never *initiate* dialogue with universals beyond their declared subscriptions. There is no "vertical asks Accounting for current balance"; there is no "vertical asks Promotion which discounts apply." Verticals subscribe to declared signals + emit declared events. The dialogue is one-shot, asynchronous, audited.

A vertical that thinks it needs to query a universal engine in real time is misdesigned. The right pattern is: vertical reads its own Foundation primitive projections (Party, Obligation, Document, Workflow) per HO8; universal engines push signals via subscription.

---

## 10. Cross-Vertical via Obligation — Doctrine (Q5 Closure)

### 10.1 BD7 + VE2 reaffirmed at handoff

CN-6-101 BD7 prohibits bridge engines. CN-6-100 VE2 prohibits direct vertical-to-vertical subscription. Cross-vertical relationships flow through **Foundation primitives** — specifically the Obligation primitive (CN-4-011).

This is the doctrine. CN-6-005 (future Bridges doc) will enumerate specific cross-vertical bridge patterns with full event chains; CN-6-104 establishes the doctrine that all such bridges use the Obligation primitive (or Party/Document primitives where appropriate) — never direct vertical-to-vertical events.

### 10.2 The five-step Obligation-as-universal-handoff pattern (N5)

For any cross-vertical relationship (vertical-A's event must influence vertical-B's Workflow), the five-step pattern is:

**Step 1 — Vertical-A emits Obligation creation.** Vertical-A's bill.ready (or other workflow event) includes an Obligation primitive emission:

```
obligation.created.v1 {
  obligation_id,
  kind: <relationship_type>,                # e.g., "hospitality_charge", "delivery_request", "layby_balance"
  debtor_party_ref,                         # who owes
  creditor_engine: <vertical-A>,            # who is owed (the originating vertical)
  amount?,                                  # monetary, if applicable
  payload_ref: <vertical-A-event-id>,       # back-ref for audit
  counterparty_workflow_ref?,               # if cross-vertical workflow link known
  business_date
}
```

**Step 2 — Vertical-B subscribes to Foundation Obligation events.** Vertical-B's manifest declares a subscription:

```
subscribes_to:
  - event_type: obligation.created.v1
    filter: kind == "<relationship_type>" && counterparty_workflow_ref == <vertical-B-workflow>
```

The filter matches obligations targeting vertical-B's own workflows. Vertical-B never subscribes to `<vertical-A>.*` events directly (VE2).

**Step 3 — Vertical-B incorporates Obligation into its Workflow.** Vertical-B's Workflow tracks the Obligation as part of its state (e.g., Hotel folio aggregates hospitality charges). The vertical-B Workflow continues its lifecycle, accruing the Obligation as a folio/folio-line/aggregated charge.

**Step 4 — Vertical-B aggregates at its bill.ready.** When vertical-B's Workflow reaches bill-ready, the aggregated Obligations are included in `saleable_lines[]` with a reference back to each Obligation:

```
vertical-B.bill.ready.v1 {
  bill_id,
  saleable_lines: [
    {line_id, item_ref: "vertical-A-charge", obligation_ref: <obl-1>, amount, ...},
    {line_id, item_ref: "vertical-B-room-night", amount, ...},
    ...
  ],
  obligation_refs: [<obl-1>, <obl-2>, ...]  # all obligations being resolved
}
```

**Step 5 — Settlement resolves Obligations; vertical-A finalizes.** Universal Checkout consumes vertical-B's bill.ready, settles tender, emits `checkout.settled.v1`. The Obligation primitive sees the settlement (via subscription to `checkout.settled.v1` filtered by `obligation_refs` in the payload) and emits `obligation.settled.v1` for each. Vertical-A subscribes to `obligation.settled.v1` filtered by its own obligations and finalizes its Workflow (e.g., restaurant.table_session.closed.v1).

### 10.3 Why this honours BD7 + VE2

- Vertical-A and vertical-B **never reference each other's events**. They reference the Foundation Obligation primitive.
- Either vertical can be decommissioned without breaking the other. Decommission Restaurant tomorrow → Hotel folios that had restaurant charges still have valid Obligation references; Hotel can fall back to manual data entry.
- New cross-vertical relationships (Workshop fabricates window, Logistics delivers it) reuse the exact same pattern with `kind: "delivery_request"` — no new engine, no new direct subscription.
- The Obligation primitive is one universal mechanism; per-relationship `kind` is the vertical context per BD5 split-it.

CN-6-005 catalogues the specific instances (charge-to-room, sell-via-POS, in-stay dining, delivery-from-workshop). CN-6-104 establishes that all instances follow the five-step pattern above.

---

## 11. Worked Patterns — Four Vertical Applications

### 11.1 WP1 — Lodge Serengeti restaurant: fan-out + settlement-back + full UI-01 audit chain (N4)

**Anchor:** Lodge Serengeti restaurant (established CN-6-101 §11.4, CN-6-102 WP3, CN-6-103 WP2). A party of four orders dinner; bill emits; M-Pesa settlement.

**Forward chain — fan-out from bill.ready:**

```
restaurant.table_session.opened.v1                    [evt-A]
                       |
restaurant.kitchen_ticket.fired.v1                    [evt-B; cause: A]
restaurant.kitchen_ticket.cooked.v1                   [evt-C; cause: B]
restaurant.kitchen_ticket.served.v1                   [evt-D; cause: C]
                       |
restaurant.bill.ready.v1                              [evt-E; cause: D]
                       ▼  fan-out (HO5)
       ┌───────────────┼──────────────┬──────────────┬─────────────────┐
       ▼               ▼              ▼              ▼                 ▼
checkout.tender.        accounting.    promotion.     reporting.       advisor (read-only
requested.v1            journal.       cost_share.    kpi.updated.v1   projection refresh —
[evt-F; cause: E]       posted.v1      recorded.v1    [evt-J; cause:E] no event)
       ▼                [evt-H;        [evt-I;
m_pesa.tender.          cause: E]      cause: E]
outcome.v1
[evt-G; cause: F]
       ▼
checkout.settled.v1                                   [evt-K; cause: G]
                       ▼  HO9 settlement-back subscription
restaurant.table_session.closed.v1                    [evt-L; cause: K]
```

**Backward audit walk-back (7 hops from receipt to originating command):**

```
1. Customer receipt printed         (Document evt from checkout.settled)
2. checkout.settled.v1               evt-K  →  cause evt-G
3. m_pesa.tender.outcome.v1          evt-G  →  cause evt-F
4. checkout.tender.requested.v1      evt-F  →  cause evt-E
5. restaurant.bill.ready.v1          evt-E  →  cause evt-D
6. restaurant.kitchen_ticket.served  evt-D  →  cause evt-C (chained back to evt-B then evt-A)
7. restaurant.table_session.opened   evt-A  →  cause: <waiter-command> "session_open.request"
                                                actor: waiter@lodge-serengeti
                                                party_ref: <guest_party>
```

Every event in the chain is replayable. Lodge Serengeti's auditor (or TRA officer auditing the period close) can reconstruct from receipt back to the waiter's first action. **This is what UI-01 buys: the chain is the legal defense.**

### 11.2 WP2 — Mzee Hassan workshop: dual-channel handoff (bill.ready + Pattern B consumption)

**Anchor:** Karakana ya Mzee Hassan, Arusha. A customer orders a window frame; project completes; bill emits + consumption events emit simultaneously.

**Manifest excerpt:**

```
emits:
  - event_type: workshop.bill.ready.v1
    # canonical handoff per HO1
  - event_type: workshop.material.consumed.v1
    # Pattern B consumption per Q3 ruling
  - event_type: workshop.cut.executed.v1
    # vertical-internal cut tracking
```

**Pack hook:**

```
pack.workshop.inventory_expansion_mode.glass = "vertical_managed"     # Pattern B (cuts emit explicit)
pack.workshop.inventory_expansion_mode.fittings = "auto"              # Pattern A (Inventory derives from sale)
```

**Event chain:**

```
workshop.project.accepted.v1                          [evt-A]
                       |
workshop.cut.executed.v1                              [evt-B; cause: A]   # glass 1.2m × 0.9m
workshop.cut.executed.v1                              [evt-C; cause: A]   # glass 1.2m × 0.5m
workshop.material.consumed.v1                         [evt-D; cause: B]   # Pattern B — Inventory deducts
workshop.material.consumed.v1                         [evt-E; cause: C]   # Pattern B
                       |
workshop.project.completed.v1                         [evt-F; cause: A]
                       |
workshop.bill.ready.v1                                [evt-G; cause: F]   # saleable_lines: window + fittings
                       ▼  fan-out
                       Inventory: Pattern A fittings deduction (auto from Checkout K2)
                       Accounting: revenue + COGS projection
                       Checkout: tender → settled → ...
                       ▼
checkout.settled.v1                                   [evt-H]
                       ▼  HO9 settlement-back
workshop.project.archived.v1                          [evt-I; cause: H]
```

**What this demonstrates:**

- Dual emission paths in one Workflow: cuts + bill
- Pattern A vs B coexist per pack hook (Q3 closure)
- HO5 fan-out from bill.ready
- HO9 settlement-back closes Workflow
- UI-01 chain traceable end-to-end

### 11.3 WP3 — Lodge Serengeti hotel + restaurant charge-to-room via Obligation (Q5 closure)

**Anchor:** Lodge Serengeti hotel + restaurant (same tenant, same site) — Mama Halima, in town for the week per CN-6-102 §11.4, dines at the restaurant and charges to her room.

**Cross-vertical event chain via Obligation primitive:**

```
restaurant.table_session.opened.v1                    [evt-A]
                       |
restaurant.kitchen_ticket.served.v1                   [evt-B; cause: A]
                       |
restaurant.bill.ready.v1                              [evt-C; cause: B]
                       | payment_method_hint: "charge_to_room"
                       | payer_party_ref: <halima>
                       ▼  Step 1 — Vertical-A emits Obligation
obligation.created.v1                                 [evt-D; cause: C]
                       | kind: "hospitality_charge"
                       | debtor_party_ref: <halima>
                       | creditor_engine: restaurant
                       | amount: <bill_total>
                       | counterparty_workflow_ref: <halima_folio>
                       ▼  Step 2 — Hotel folio Workflow subscribes
                       (Hotel manifest: subscribes_to obligation.created.v1
                        filter: kind == "hospitality_charge"
                                && counterparty_workflow_ref == <self>)
                       ▼  Step 3 — Vertical-B incorporates
hotel.folio.charge_added.v1                           [evt-E; cause: D]
                       | folio_id: <halima_folio>
                       | obligation_ref: <obl-from-D>
                       | line_amount, line_description
                       ▼  [days pass; more charges accumulate]
hotel.reservation.checked_out.v1                      [evt-F]
                       ▼  Step 4 — Vertical-B aggregates at bill.ready
hotel.folio.ready.v1                                  [evt-G; cause: F]
                       | saleable_lines: [room-night, room-night, hospitality_charge (← obl), ...]
                       | obligation_refs: [<obl-from-D>, ...]
                       ▼  Universal Checkout
checkout.settled.v1                                   [evt-H]
                       ▼  Step 5 — Obligation resolves; vertical-A finalizes
obligation.settled.v1                                 [evt-I; cause: H]
                       | obligation_id: <obl-from-D>
                       ▼  Restaurant subscribes to its own obligations
restaurant.table_session.closed.v1                    [evt-J; cause: I]
                       (restaurant.bill.ready evt-C was held in "billed_pending" state
                        until obligation resolved)
```

**What this proves:**

- Restaurant and Hotel never reference each other's events (VE2 honoured)
- The Obligation primitive carries the cross-vertical relationship (BD7 honoured)
- Decommissioning Hotel does not break Restaurant (isolation preserved)
- Accounting auto-journals correctly: restaurant revenue recognized only at obligation resolution (CN-5-001 + N3 of CN-5-105 tax timing)
- CN-6-005 will catalog this and other bridge patterns; CN-6-104 establishes the doctrine

### 11.4 WP4 — Mama Halima Logistics trip: multi-site-by-nature + fuel Pattern B

**Anchor:** Mama Halima Dar→Mwanza trip (CN-6-100 §12 + CN-6-102 §12.5 + CN-6-103 WP4).

**Event chain:**

```
logistics.trip.planned.v1                             [evt-A]
                       | origin_site_id: dar-depot
                       | destination_site_id: mwanza-receiving
                       |
logistics.trip.dispatched.v1                          [evt-B; cause: A]
                       |
logistics.fuel.consumed.v1                            [evt-C; cause: B]   # Pattern B — Inventory deducts at origin
                       | site_id: dar-depot
                       | source_ref: <trip-A>
                       |
logistics.trip.in_transit_started.v1                  [evt-D; cause: B]
logistics.trip.arrived.v1                             [evt-E; cause: D]
logistics.trip.closed.v1                              [evt-F; cause: E]
                       |
logistics.bill.ready.v1                               [evt-G; cause: F]
                       | origin_site_id, destination_site_id   (multi-site-by-nature per SP4)
                       | saleable_lines: [freight, fuel_surcharge, demurrage]
                       ▼  fan-out
                       Accounting: revenue by origin-destination per pack
                       Checkout: tender → settled
                       ▼
checkout.settled.v1                                   [evt-H]
                       ▼  HO9 settlement-back
logistics.trip.archived.v1                            [evt-I; cause: H]
```

**What this demonstrates:**

- Multi-site-by-nature payload (SP4 + CN-6-103 §7) flows through to Accounting and Reporting
- Fuel as Pattern B consumption (Q3 closure) — site-scope (origin depot)
- Bill emitted at trip close — single bill.ready per trip Workflow (Q1 closure)
- HO9 settlement-back closes the trip

---

## 12. Boundaries + Open Items + Cross-Term Hooks

### 12.1 CN-6-104's place in the corpus

| Concern | Owned by | CN-6-104 role |
|---------|----------|----------------|
| Vertical-to-universal handoff doctrine | **CN-6-104** (this doc) | Authoritative |
| Universal Checkout consumption | CN-5-009 | Consumer — CN-6-104 specifies what gets handed off |
| Subscription wiring patterns | CN-5-100 | Sibling — CN-6-104 specifies vertical-emission discipline; CN-5-100 specifies subscription patterns |
| Saleable Line / Tender value shapes | CN-4-021 | Foundation contract CN-6-104 builds on |
| Concrete per-vertical handoff manifests | CN-6-001..004 | Consumers — each vertical declares its handoffs per HO1-HO9 |
| Cross-vertical bridges (specific instances) | CN-6-005 (future) | CN-6-104 §10 establishes doctrine; CN-6-005 catalogs instances |
| Mixed-Vertical Tenant handoff coexistence | CN-6-105 (future) | CN-6-104 §10 + WP3 demonstrate; CN-6-105 generalises |
| Advisor wiring | CN-5-010 + CN-4-022 | CN-6-104 §5 catalog row |

### 12.2 CTRs cited (no new)

- **CTR-018, CTR-002** (registration; lines feed pattern) — §3 + §4
- **CTR-024, CTR-026, CTR-030** (payload conformance) — §3 HO3, §5 catalog
- **CTR-038** (namespace) — §3 + §6
- **CTR-044, CTR-045** — pending upstream context
- **CTR-046** — DC-NN-f (HO9 settlement-back-subscription) **queued for next amendment cycle alongside DC-NN-e (SP8 from CN-6-103)**

**Cumulative CTR-046 expansion queue after CN-6-104:**

| Queued check | Source | Spec |
|--------------|--------|------|
| DC-NN-e | CN-6-103 §5.3 | For any engine with `engine_kind: vertical`, no `subscribes_to[*].scope_ref: platform` allowed |
| DC-NN-f | CN-6-104 §3 (HO9) | For any engine with `engine_kind: vertical` declaring a `workflow_instances[]` with `billable: true`, the workflow MUST declare `settlement_subscription` with event `checkout.settled.v1` |

Both checks queue together for Term 4's next CTR-046 amendment cycle.

### 12.3 Open items inside Term 6 scope

- **CN-6-005 future** — CN-6-104 §10 establishes the Obligation-as-universal-handoff doctrine; CN-6-005 will enumerate specific bridge patterns (charge-to-room, sell-via-POS, in-stay dining, workshop-delivery via Logistics)
- **CN-6-105 future** — Mixed-Vertical Tenants generalising the Mama Amina + Faraja pattern from CN-6-102 §11.4
- **CN-6-901..904 stress-test sketches** — apply CN-6-100..104 to Insurance, Healthcare, Education, Marketing Agency; CN-6-905 to salon + light services cluster
- **Multi-jurisdiction handoff** — deferred to CN-5-105 §11 v2; CN-6-104 v1 single-jurisdiction per CTR-027

### 12.4 D-DISC cross-references

- **D-DISC-001 — tenant-customer promotion UX**: when customer-facing surface shows "your loyalty balance" at vertical-customer-interaction event (e.g., `restaurant.customer.seated.v1` per CN-6-102 §9), handoff to Promotion provides the projection. Term 3 designs the surface; CN-6-104 §5 + §9 wire the data.
- **D-DISC-002 — POS self-service expansion**: self-service kiosk emits the same vertical events as cashier-operated POS — handoff doctrine unchanged; identity (CN-4-007 customer-as-actor) handles the actor distinction. CN-6-104 patterns absorb without modification.

---

## 13. Closing the Cross-Cutting Cluster

### 13.1 The cluster is complete

With CN-6-104 merged, the Term 6 cross-cutting framework cluster is complete:

| Doc | Mission | Status |
|-----|---------|--------|
| CN-6-100 | The recipe — how to add a new vertical | ✅ |
| CN-6-101 | The boundary doctrine — what belongs in vertical vs universal vs Foundation | ✅ |
| CN-6-102 | The naming conventions — event vocabulary at the vertical layer | ✅ |
| CN-6-103 | The scope policy — site / tenant / multi-site-by-nature for verticals | ✅ |
| CN-6-104 | The hand-off pattern — how verticals communicate with universals | **✅ (this doc)** |

Anyone authoring a new vertical now has the complete operational + doctrinal reference. Brief §14 Flexibility Test was pre-emptively demonstrated via Logistics (CN-6-100 §12) and the Salon framing (CN-6-101 §11.2). The framework is ready.

### 13.2 Character map handoff to CN-6-001..004 (N6)

The Term 6 character corpus carries forward into concrete-vertical docs. Each vertical author inherits the established cast and may extend with consistent additions:

| Vertical doc | Primary anchor | Established characters in doc |
|--------------|----------------|---------------------------------|
| **CN-6-001 Retail** | Duka la Mama Amina, Kariakoo | Mama Amina (owner), Salma (cashier daughter), customers including Mama Halima |
| **CN-6-002 Restaurant** | Lodge Serengeti restaurant | Floor staff, kitchen, guests (Mama na Bwana Mwema among them) |
| **CN-6-003 Hotel** | Kilimanjaro Lodge Moshi + Lodge Serengeti (chain) | Receptionists, GMs, guests, housekeeping; chain-level guest profile across both |
| **CN-6-004 Workshop** | Karakana ya Mzee Hassan, Arusha | Mzee Hassan (owner-fundi), apprentice fundis, customers |

Logistics (Mama Halima), Pharmacy (Faraja at Mama Amina's expansion), and the salon/light-services cluster carry into future stress-test sketches (CN-6-901..905). The corpus stays cohesive — readers across docs encounter the same people, the same Tanzanian towns, the same business expansions.

### 13.3 Per-Brief §13 ordering, what comes next

- **CN-6-001 Retail** — first concrete vertical. Mama Amina's Kariakoo duka. The simplest vertical baseline; tests the framework on its easiest case.
- **CN-6-002 Restaurant** — Lodge Serengeti. Adds kitchen workflow + table conflict + recipe consumption (Pattern B).
- **CN-6-003 Hotel** — Kilimanjaro Lodge Moshi + Lodge Serengeti chain. Adds reservation lifecycle + folio + chain-scope guest profile + cross-vertical Obligation (with Restaurant).
- **CN-6-004 Workshop** — Mzee Hassan. Most complex existing vertical; tests parametric cuts + project lifecycle + mixed-scope (cuts site, styles tenant).
- **CN-6-005 Bridges** — catalogues cross-vertical bridge patterns per CN-6-104 §10 doctrine.
- **CN-6-105 Mixed-Vertical Tenants** — generalises Mama Amina + Faraja pattern.
- **CN-6-901..904** — Insurance, Healthcare, Education, Marketing Agency stress-test sketches.
- **CN-6-905** — Salon + light services cluster placement decision (per CN-6-101 §11.2 elevation).

### 13.4 The bar — final restatement before concrete work

Salma scans a tin of cooking oil. The receipt prints. Mama Amina's phone updates. Within seconds, six universal engines have done their work — none of which Salma or Mama Amina ever sees.

That is what the cross-cutting cluster delivers. Five documents — recipe, boundary, naming, scope, handoff — encode the discipline that keeps the plumbing tight while every real business interaction stays human-paced and human-meaningful.

The framework is done. The verticals begin.

---

*— End of CN-6-104 Vertical-to-Universal Hand-Off Pattern v1 —*
*— End of Term 6 cross-cutting cluster —*
