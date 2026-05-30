# CN-5-009 — Universal Checkout / Tender Engine

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 5 — Universal Engines
> **Status:** For Overseer review.
> **Governing decisions:** D-001 (Universal Checkout — split tender from verticals); D-007 #1 (value shapes in Foundation; Checkout is a Term 5 engine); D-007 #12 (DEGRADED block-reserved numbering); D-009 (freeze doctrine — pack version at emission); Law 2 (engine isolation); Law 5 (compliance is configured).
> **CTRs:**
> - **CLOSED upon merge:** CTR-001 (Saleable Line/Tender shape — built on); CTR-003 (Checkout is engine, not primitive — demonstrated); CTR-005 (always-on vs catalog — always-on for every tenant).
> - **NEGOTIATING (proposed contract refined here):** CTR-006 (payment adapter contract at tender boundary — request/reply via events with `callback_correlation_id`).
> - **OPEN (this doc depends on):** CTR-004 (Term 3 — one checkout UX); CTR-028 (Term 7 + Term 1 — tender method registry vocabulary + governance); CTR-016 (Term 1 — platform-scope governance, billing aggregation); CTR-027 (Term 1/2 — tenant functional currency registry, used by UI-06); CTR-024 (Term 6 — verticals carry `site_id`).
> **Glossary:** See `MASTER-GLOSSARY.md` — Universal Checkout, Saleable Line / Charge, Tender, Site, Scope level, UI-NN.
> **Depends on:** CN-4-002 (envelope, ordering, atomic emission references); CN-4-004 (command bus, atomic multi-event emission, policy rejection); CN-4-011 (Obligation primitive — created on credit tender; Document primitive — receipt); CN-4-012 (Document Engine — receipt, numbering, hash, DEGRADED block-reserved); CN-4-014 (clock protocol — record time and business date); CN-4-015 (Compliance DSL — tax treatment at settlement, pack-frozen); CN-4-021 (Saleable Line & Tender value shapes); CN-5-100 (subscription patterns and laws); CN-5-101 (scope policy — site default for Checkout); CN-5-102 (engine invariants — UI-01 causation, UI-04 tender chain, UI-06 currency, UI-09 site_id).
> **Boundaries:** Saleable Line / Tender shape → CN-4-021; document hash and numbering → CN-4-012; tax rules → compliance packs via CN-4-015; vertical bill production → Term 6 (per-vertical events); checkout UX → Term 3 (CTR-004); payment adapter internals → Term 7 (CTR-006); tender method vocabulary → Term 7 + Term 1 (CTR-028); promotion rule definition and bill-level discounts → CN-5-007; downstream universal-engine subscriptions (Accounting/Cash/Inventory effects) → CN-5-001/002/003.

---

## 1. What This Doc Defines

CN-5-009 is the first concrete engine of Term 5. It implements **D-001 Universal Checkout** on top of the value shapes provided by CN-4-021 (Saleable Line, Tender) and the doctrine layers established by CN-5-100 (subscriptions), CN-5-101 (scope), and CN-5-102 (invariants).

Universal Checkout is the **revenue hinge** of BOS. Every vertical — Retail, Restaurant, Hotel, Workshop, and any future vertical — emits its own `bill.ready` event. Checkout consumes those events, settles payment, issues a receipt, and emits `checkout.settled.v1`. Every downstream universal engine — Accounting, Cash, Inventory, Promotion, Reporting — observes that one event to do its work.

This doc defines:
- The **engine manifest** (engine_id, scope policy, emits, subscribes_to, commands, requires).
- The **settlement workflow** — a four-phase event-sourced sequence from `<vertical>.bill.ready.v1` to `checkout.settled.v1`.
- **Multi-tender semantics** — how cash (synchronous) and async tenders (mobile money, card) cohere.
- **Refund and cancellation** — refund as a command that produces a compensating event.
- **Tax computation at settlement** — pack-frozen, deterministic.
- **Receipt issuance** — via CN-4-012 Document Engine, including DEGRADED block-reserved numbering.
- **Credit tender → Obligation** creation.
- **Currency enforcement** — UI-06 at every command boundary.

This doc does NOT define:
- The shapes of Saleable Line and Tender (CN-4-021).
- The mechanics of the Document Engine — templates, hashes, numbering (CN-4-012).
- The Compliance DSL grammar (CN-4-015).
- Specific tax rates (compliance packs).
- How verticals construct their bills (Term 6 per-vertical docs).
- The cashier-facing UI (Term 3, CTR-004).
- Payment adapter internals (Term 7, CTR-006).
- Promotion rule definitions or bill-level discounts (CN-5-007).
- Accounting, Cash, Inventory effects downstream — each owns its own subscription doc.

---

## 2. The Five Laws

These laws apply to every operation of the Checkout engine. Violations are doctrine violations.

| # | Law | Source / why |
|---|-----|--------------|
| **K1** | **Vertical-blind.** Checkout never reads or branches on the identity of the vertical that produced a bill. The `source_tag` (CN-4-021 §2) is opaque — Checkout carries it for traceability but never inspects its content. A doctrine check (CN-4-019 DC-022) verifies via static analysis that no Checkout code path inspects `source_tag`. | Charter Law 2; D-001 universal-checkout intent; CN-4-021 §2 non-branching guarantee. |
| **K2** | **Authoritative inputs only.** Checkout recomputes every line total from authoritative inputs (`quantity × unit_price`, then applying `discount_refs`). The `line_total` cache on the incoming line (CN-4-021 §2) is never trusted. If recomputation disagrees with the cache, the recomputed value wins; the cache is ignored. | CN-4-021 §2 is explicit: `line_total` is "non-authoritative cache." Trusting it would make the engine credulous of upstream input. |
| **K3** | **Tax at settlement, pack-frozen.** Tax amounts are computed by Checkout — never received from the vertical. The compliance pack version active **at settlement emission time** is the version used (D-009 freeze; CN-4-015). The resulting `pack_version_ref` is recorded on `checkout.settled.v1`. | Law 5; D-009; CN-4-021 §2 (vertical declares `tax_treatment_ref`, not the amount). |
| **K4** | **Tender lifecycle = events, not mutable state.** The Tender value shape (CN-4-021 §3) is lifecycle-free. Pending / confirmed / failed are expressed as separate Checkout events, not as fields on the shape. Async tender outcomes (mobile money, card) arrive as events from Term 7 adapters. | CN-4-021 §3 explicit ("no status field"); CN-5-100 L4 (subscribers produce events only through commands). |
| **K5** | **Settlement is explicitly triggered and atomic.** `checkout.settled.v1` is emitted **only** in response to an explicit `checkout.settle.request` command from the cashier, **and only when** all tenders are confirmed and `sum(confirmed tender amounts) ≥ grand_total`. The settled event and the receipt-issuance event are emitted atomically (CN-4-004 §2). The engine does not auto-settle. | CN-4-004 atomic multi-event; UI-04 tender chain integrity; prevents premature emission while async tenders are still pending. |

---

## 3. Engine Manifest

This is the Checkout engine's declaration to the Kernel, conforming to CN-4-005 §1 with per-operation `scope_ref` overrides per CN-4-005 §1.1 (post-CTR-023) and CN-5-100 §4.

```yaml
engine_id:        checkout
display_name:     Universal Checkout / Tender Engine
version:          1
scope_policy:     site                    # default — sales occur per site (CN-5-101)
primitives_used:
  - obligation                            # credit tender → Obligation (CN-4-011)
  - document                              # receipt issuance (CN-4-011, CN-4-012)
```

### Commands

| Command | Scope | Description |
|---------|-------|-------------|
| `checkout.ingest_bill.request` | `site` | System-submitted on receipt of `<vertical>.bill.ready.v1` — starts a settlement workflow |
| `checkout.tender.add.request` | `site` | Cashier adds a tender (cash, mobile money, card, voucher, credit) to in-flight workflow |
| `checkout.tender.confirm.request` | `site` | System-submitted on receipt of adapter outcome (async tenders) |
| `checkout.tender.fail.request` | `site` | System-submitted on adapter failure |
| `checkout.settle.request` | `site` | Cashier explicitly triggers settlement (K5) |
| `checkout.cancel.request` | `site` | Cashier cancels in-flight workflow before settlement |
| `checkout.refund.request` | `site` | Cashier submits a refund against a settled checkout — produces a compensating event |

System-submitted commands (`ingest_bill`, `tender.confirm`, `tender.fail`) carry `system:checkout-orchestrator` as actor (CN-4-007); cashier-submitted commands carry `human:cashier-<id>`. All commands are bus-policed per CN-4-004 §3.

### Emitted Events

| Event | Purpose |
|-------|---------|
| `checkout.started.v1` | Workflow opened. Carries ingested lines, recomputed pre-tax totals, computed tax breakdown, grand total, currency, site_id, cashier_session_id. |
| `checkout.tender.requested.v1` | Async tender forwarded to Term 7 adapter (synchronous tenders skip this). |
| `checkout.tender.confirmed.v1` | Tender confirmed (cash: immediately on add; async: on adapter outcome). |
| `checkout.tender.failed.v1` | Tender failed; workflow waits for replacement. |
| `checkout.settled.v1` | **The hinge.** Final settlement record consumed by Accounting / Cash / Inventory / Reporting / Promotion. |
| `checkout.cancelled.v1` | Workflow cancelled before settlement. |
| `checkout.refunded.v1` | Compensation event with non-null `compensates_event_id` pointing to a prior `checkout.settled.v1`. |
| `checkout.receipt.issued.v1` | Document primitive event (CN-4-011 terminal fold) — atomic with `checkout.settled.v1`. |

### Subscriptions

Per CN-4-005 §1, the manifest references event types, never engines by name. Per K1, the entries are per-vertical — adding a new vertical means adding one entry to this manifest, not modifying Checkout's logic.

| event_type | version | kind | handler | scope_ref |
|------------|---------|------|---------|-----------|
| `<vertical>.bill.ready.v1` (one entry per vertical: `retail.bill.ready.v1`, `restaurant.bill.ready.v1`, `hotel.bill.ready.v1`, `workshop.bill.ready.v1`, …) | 1 | `command_emitting` | `ingest_bill` | `site` |
| `<adapter>.tender.outcome.v1` (one entry per registered async adapter — e.g., `mpesa.tender.outcome.v1`, `airtelmoney.tender.outcome.v1`, `stripe.tender.outcome.v1`) | 1 | `command_emitting` | `record_tender_outcome` | `site` |

The manifest **grows by addition only** as new verticals (Term 6) and new adapters (Term 7, CTR-028) come online. Existing entries never change.

### `requires`

The mandatory subset of `subscribes_to`:
- At least one `<vertical>.bill.ready.v1` subscription must be present (otherwise the engine has nothing to settle).
- For every async tender method declared in the tender registry, the corresponding `<adapter>.tender.outcome.v1` subscription must be present (otherwise async settlement cannot complete).

Synchronous-only deployments (cash only) require only the vertical subscription.

---

## 4. Settlement Workflow

Checkout's central process is an event-sourced workflow with four phases. The workflow is **per settlement session** — typically per customer transaction at a site.

### Phase 1 — Ingest

A vertical emits `<vertical>.bill.ready.v1` carrying Saleable Lines (CN-4-021 §2). Checkout subscribes (kind: `command_emitting`); the `ingest_bill` handler submits `checkout.ingest_bill.request`. The bus:

1. **Validates** the vertical's payload against the Saleable Line shape.
2. **Recomputes** each line total from authoritative inputs (K2). Discards the cached `line_total`.
3. **Resolves the compliance pack** active at this moment (CN-4-015 §5 tenant↔jurisdiction binding) and **computes tax** per line (K3): looks up the `tax_treatment_ref`, applies the rate from the pack, produces a per-line tax amount.
4. **Aggregates** pre-tax total, tax total (with per-treatment breakdown), and grand total.
5. **Emits** `checkout.started.v1` carrying the workflow snapshot (see §5 for shape).

The workflow is now open and identified by its `stream_id` (Checkout assigns one per settlement session).

### Phase 2 — Collect Tenders

The cashier submits one `checkout.tender.add.request` per tender. The bus:

1. **Validates** currency against the settlement currency (M1 — currency check; UI-06). A tender with non-matching currency is rejected:
   ```
   rejected_by_policy:  UI-06.functional_currency_invariant
   rejection_reason:    "Tender currency does not match settlement currency"
   ```
2. **Validates** the method against the tender method registry (Term 7 + Term 1, CTR-028). Unknown method → rejected.
3. **Branches** on method category:
   - **Synchronous** (`cash`, `voucher`, `credit`): immediately emits `checkout.tender.confirmed.v1`.
   - **Async** (`mobile_money`, `card`, `bank_transfer`): emits `checkout.tender.requested.v1` carrying `{tender_id, method, amount, callback_correlation_id}`. The Term 7 adapter for that method subscribes and proceeds to Phase 3.

The cashier may add multiple tenders sequentially. Each is independent and event-sourced; the workflow tolerates partial states (e.g., one confirmed cash tender plus one pending mobile money tender).

### Phase 3 — Adapter Round-Trip (async only)

For each `checkout.tender.requested.v1`, the matching Term 7 adapter:

1. Consumes the event (Term 7 owns this subscription).
2. Invokes the external payment system (M-Pesa, Stripe, etc.) — Term 7 internals.
3. Emits `<adapter>.tender.outcome.v1 { tender_id, callback_correlation_id, outcome: confirmed | failed, external_ref, reason? }`.

Checkout subscribes to the adapter's outcome event (handler `record_tender_outcome`); the handler submits either `checkout.tender.confirm.request` or `checkout.tender.fail.request`. The bus emits `checkout.tender.confirmed.v1` or `checkout.tender.failed.v1`. The proposed contract for CTR-006 is precisely this request/reply via events.

If a tender fails, the workflow waits for a replacement tender (cashier adds another via Phase 2). The failed tender remains in the workflow record for audit; it does not block other tenders' progression.

### Phase 4 — Settle

When the cashier judges the workflow ready (all needed tenders confirmed), they submit `checkout.settle.request` (K5 — explicit trigger). The bus:

1. **Verifies** `sum(confirmed tender amounts) ≥ grand_total`. If not, rejects:
   ```
   rejected_by_policy:  K5.settlement_preconditions_unmet
   rejection_reason:    "Confirmed tenders (<sum>) do not cover grand total (<total>)"
   ```
2. **Verifies** no tender is still pending (no outstanding `checkout.tender.requested.v1` without a matching `confirmed` or `failed`). If pending, rejects: "Async tender(s) still pending; wait or cancel."
3. **Computes** change = `sum(confirmed) − grand_total` (zero or positive).
4. **Determines** credit tender presence — if any confirmed tender has `method = credit`, an Obligation will be created (§10).
5. **Issues** the receipt via the Document Engine (CN-4-012) — assembled content, template selection, hash computation, deterministic numbering (per-site sequence in NORMAL; block-reserved in DEGRADED, see §9).
6. **Atomically emits** (CN-4-004 §2):
   - `checkout.settled.v1` (the hinge)
   - `checkout.receipt.issued.v1` (Document primitive)
   - `obligation.created.v1` if credit tender present

All three events share a `correlation_id` (the settle request) and carry causation back through the workflow per CN-5-100 §5 / UI-01.

### Cancellation

`checkout.cancel.request` is valid in Phases 1–3 (before `checkout.settled.v1` exists). The bus emits `checkout.cancelled.v1`. Any tender still in `requested` state is signalled to its adapter via a cancellation event (Term 7's contract surface); the adapter is responsible for reversing whatever it has done.

### DEGRADED Mode Behaviour (M3)

When the system is in DEGRADED mode (CN-4-017):

- **Receipt numbering** falls back to per-site block-reserved sequences (CN-4-012 §3). Receipts issued in DEGRADED carry numbers from the site's pre-reserved block; reconciliation happens on recovery per CN-4-012 §3.
- **Async tenders** may stall if the Term 7 adapter is impaired. The workflow's tender remains in `requested` state; the cashier sees the freshness indicator and may choose to cancel and re-tender via a synchronous method.
- **Synchronous settlements** (cash-only) proceed normally — they require only Checkout, Document Engine, and per-site numbering.
- **Subscribers downstream** (Accounting / Cash / Inventory) may lag (stale-but-available per CN-4-010 §6); freshness is surfaced (CN-4-022 §7) to dashboards reading their projections.

The settlement workflow itself does not change; the operational envelope around it does.

---

## 5. Event Shapes

### `checkout.started.v1`

```
event_type:        checkout.started.v1
stream_id:         checkout-<session-id>
tenant_id:         <envelope>
actor:             system:checkout-orchestrator
causation_id:      <event_id of <vertical>.bill.ready.v1>
correlation_id:    <inherited from ingest_bill command>
pack_version_ref:  <pack version resolved at ingest>
timestamp:         <record time, CN-4-014>
payload:
  site_id:                <CN-5-101 site-scope>
  cashier_session_id:     <optional, links to Cash session>
  business_date:          <CN-4-014 business-time>
  currency:                <tenant functional currency, UI-06>
  lines:                  [<recomputed Saleable Lines, K2>]
  pre_tax_total:          <decimal>
  tax_breakdown:          [{treatment_ref, taxable_amount, tax_amount}]
  tax_total:              <decimal>
  grand_total:            <decimal>
  source_tag:             <opaque, K1 — Checkout does not read this>
```

### `checkout.tender.requested.v1` (async only)

```
event_type:        checkout.tender.requested.v1
stream_id:         checkout-<session-id>
causation_id:      <event_id of checkout.tender.add.request derivative>
payload:
  tender_id:                 <unique within session>
  method:                    <e.g., mobile_money>
  amount:                    <decimal>
  currency:                  <matches settlement currency, UI-06>
  callback_correlation_id:   <Checkout uses this to route the adapter's outcome back to this session>
```

### `checkout.tender.confirmed.v1` / `checkout.tender.failed.v1`

```
event_type:        checkout.tender.confirmed.v1
causation_id:      <for cash: the add.request derivative; for async: the adapter outcome event>
payload:
  tender_id:        <as above>
  method:           <as above>
  amount:           <decimal>
  external_ref:     <adapter-supplied for async; null for cash>
  (for .failed): reason: <human reason>
```

### `checkout.settled.v1` (the hinge)

```
event_type:        checkout.settled.v1
stream_id:         checkout-<session-id>
tenant_id:         <envelope>
actor:             human:cashier-<id>     (the settle command actor)
causation_id:      <event_id of the corresponding <vertical>.bill.ready.v1 — root of choreography>
correlation_id:    <inherited from settle command>
pack_version_ref:  <pack version active at this moment; D-009>
timestamp:         <record time>
payload:
  site_id:                <CN-5-101>
  cashier_session_id:     <optional, links to Cash session>
  business_date:          <CN-4-014>
  currency:                <tenant functional currency>
  lines:                  [<final Saleable Lines as settled>]
  pre_tax_total:          <decimal>
  tax_breakdown:          [{treatment_ref, taxable_amount, tax_amount}]
  tax_total:              <decimal>
  grand_total:            <decimal>
  tenders:                [<confirmed tender list with external_refs>]
  tenders_total:          <decimal>
  change:                  <decimal — zero or positive>
  obligation_ref:         <obligation_id if credit tender present, else null>
  receipt_ref:            <document_id of the issued receipt>
  source_tag:             <opaque, carried for traceability — K1>
```

### `checkout.refunded.v1` (compensation)

```
event_type:            checkout.refunded.v1
compensates_event_id:  <event_id of the original checkout.settled.v1>     (CN-4-002)
causation_id:          <event_id of the refund command derivative>
payload:
  site_id:                  <same as original>
  cashier_session_id:       <optional, current session>
  business_date:            <CN-4-014 — current open period; UI-05 forbids back-dating>
  currency:                  <same as original>
  refund_kind:              full | partial
  lines_refunded:           [<subset of original lines (partial) or all (full)>]
  pre_tax_refund:           <decimal>
  tax_reversed_breakdown:   [{treatment_ref, refund_taxable, refund_tax}]
  tax_total_reversed:       <decimal>
  refund_total:             <decimal>
  refund_tenders:           [<how money flows back — symmetric to original tenders by default>]
  obligation_adjustment:    <if original had credit tender, how the Obligation is adjusted>
  refund_receipt_ref:       <document_id of credit-note document>
  reason_ref:               <code or text>
```

The compensating event triggers downstream engines via `kind: compensation` subscriptions (CN-5-100 P4). Accounting reverses its journal, Cash records the outflow, Inventory restores stock — each in symmetry with what it did originally (UI-02).

### `checkout.cancelled.v1`

```
event_type:        checkout.cancelled.v1
stream_id:         checkout-<session-id>
causation_id:      <event_id of the cancel command derivative>
payload:
  session_state_at_cancel: <which tenders were pending/confirmed/failed>
  reason_ref:               <code or text>
```

### `checkout.receipt.issued.v1`

The Document primitive's terminal-fold event (CN-4-011 Document; CN-4-012 §2). Co-emitted atomically with `checkout.settled.v1` per K5.

```
event_type:        checkout.receipt.issued.v1
stream_id:         document-<receipt-id>
payload:
  document_id:           <receipt-id>
  template_version_ref:  <template at issuance — D-009>
  pack_version_ref:      <pack at issuance — same as settled.v1>
  document_number:       <per CN-4-012 §3; deterministic, scoped tenant + receipt-type + site>
  document_hash:         <CN-4-012 §5>
  content:               <line items, totals, tender summary, business_date>
```

---

## 6. Multi-Tender Model & Adapter Async

A real settlement frequently combines methods: TZS 100,000 cash + TZS 82,220 mobile money. CN-5-009 supports this by event-sourcing the tender lifecycle (K4), not by carrying status on the value shape.

### Synchronous vs Async Methods

| Category | Examples | Confirmation timing |
|----------|----------|---------------------|
| Synchronous | `cash`, `voucher`, `credit` | Immediate — `checkout.tender.confirmed.v1` emitted on `checkout.tender.add.request` acceptance |
| Async | `mobile_money`, `card`, `bank_transfer` | Deferred — `checkout.tender.requested.v1` emitted; Term 7 adapter's outcome event triggers `checkout.tender.confirmed.v1` or `.failed.v1` |

The cashier-facing flow is identical for both: add a tender, wait for confirmation, proceed. The engine internally distinguishes by method category; the registry (CTR-028) declares which methods are async.

### The `callback_correlation_id` Pattern

To route an adapter outcome back to the correct settlement session, Checkout supplies a `callback_correlation_id` in `checkout.tender.requested.v1`. The Term 7 adapter must echo this exact value back in its `<adapter>.tender.outcome.v1`. When the adapter outcome arrives, Checkout's `record_tender_outcome` handler uses the callback_correlation_id to identify the originating session and submit the appropriate `tender.confirm.request` or `tender.fail.request`.

This is the proposed CTR-006 contract: events flow Checkout → adapter → Checkout, with no direct calls.

### Atomicity Under Multi-Tender

K5 makes settlement explicit and conditional. The bus emits `checkout.settled.v1` only when:
- All tenders that were ever requested are now in a terminal state (confirmed or failed).
- `sum(confirmed amounts) ≥ grand_total`.

If a settle request arrives prematurely, the bus rejects it with a clear reason; the workflow remains open.

### Change

`change = sum(confirmed amounts) − grand_total`. Always non-negative (overpayment by the customer). Change does not produce its own tender event — it is recorded on `checkout.settled.v1` as a payload field. Cash drawer adjustments for change are downstream Cash Engine concerns (CN-5-002), not Checkout's.

---

## 7. Refund as Command + Compensation Event

Refund follows the dual pattern: cashier-initiated **command** that, on acceptance, emits a **compensating event**.

### The Command

`checkout.refund.request` is submitted by an authorised cashier (a refund-authorised role; specific role policy is per-tenant configuration). The command names the original `checkout.settled.v1` event ID and indicates whether the refund is full or partial (with a list of refunded lines for partial).

The bus evaluates standard policies:
- **K3 / CN-4-015:** the pack version at the refund moment is the version used for tax-reversal computation (the system honours the freeze: the tax that was charged at the original settlement is what is reversed, not whatever the current rate is — the original settlement's `pack_version_ref` is the authoritative reference).
- **UI-05 (closed-period):** the refund event posts to the **current open period**, with `causation_id` walking back to the closed-period settled event. The refund does not retroactively re-open a closed period.
- **UI-06 (currency):** refund_currency must equal original currency.
- **UI-04 (tender chain):** refund_tenders form a valid chain — Cash refunds out to a cash drawer, mobile-money refunds reverse-through Term 7 adapter, etc.

### The Compensating Event

`checkout.refunded.v1` carries a non-null `compensates_event_id` (CN-4-002) pointing to the original `checkout.settled.v1`. This makes it structurally identifiable as a compensation (per CN-4-002 §1) and routes it through the compensation subscription kind (CN-5-100 P4) at downstream engines.

### Refund Tender Outflow

Refunds typically reverse-through the original tender path (cash refund out for a cash payment, M-Pesa refund for an M-Pesa payment). The refund_tenders list expresses this. For async outflows, Checkout emits `checkout.tender.requested.v1` with a refund-direction marker (or equivalent) and the Term 7 adapter handles the reverse transaction. The adapter's outcome events confirm the refund tender just as it confirms forward tenders.

### Partial Refund Symmetry (M2)

A partial refund (refunding only some lines from the original settlement) raises a UI-02 question: each downstream engine must compensate its portion symmetrically. Accounting reverses the portion of the journal that corresponds to the refunded lines; Inventory restores the refunded items; Cash records the partial outflow.

The per-engine symmetric-compensation logic for partial refunds is **Open Item** for the Architect phase and per-engine docs (CN-5-001/002/003). CN-5-009 defines the trigger event (`checkout.refunded.v1` with `refund_kind: partial` and `lines_refunded` subset); the propagating engines define how they map a partial subset to their state changes.

### Refund Receipt

A refund issues a **new document** (credit note / refund receipt), per CN-4-012 §6. It is a distinct document with its own number (in its document-type's sequence), its own hash, and a reference to the original document. The original receipt is never modified.

---

## 8. Tax Computation at Settlement

CN-5-009 implements Law 5 + D-009 at the settlement boundary. Tax is computed by Checkout, never by the vertical.

### Resolution

| Step | What happens |
|------|--------------|
| 1 | The pack version active at ingest is resolved (`tenant → jurisdiction → pack at moment`, CN-4-015 §5). |
| 2 | For each Saleable Line, the `tax_treatment_ref` is looked up in the resolved pack via the sandboxed evaluator (CN-4-015 §4). The evaluator returns the applicable rate(s) and any special rules. |
| 3 | Per-line tax amount = `(quantity × unit_price − applicable discounts) × rate`. The exact formula is in the pack (it may include nested rules — e.g., "round each line tax to nearest cent"); Checkout invokes the evaluator and trusts the result. |
| 4 | Per-treatment aggregates are computed for the breakdown displayed on the receipt and recorded in the event payload. |
| 5 | At settlement emission, the pack version is recorded once more (the freeze applies at the moment of `checkout.settled.v1` per D-009; in practice the ingest and settle moments are the same pack version — the engine confirms this and rejects if the pack changed mid-workflow). |

### What If the Pack Changes Mid-Workflow

A pack rotation between ingest and settle is a rare event. Checkout's bus policy treats this as a doctrine-relevant state change: if the active pack version at settle differs from the pack version recorded on `checkout.started.v1`, the settle request is rejected with a clear reason ("compliance pack rotated during workflow; cancel and re-ingest"). The workflow must be cancelled and re-started under the new pack — no silent re-computation.

### Why Not Pre-Compute and Cache

A naive implementation would compute tax once at ingest and cache the values. CN-5-009 instead recomputes at every event emission that records tax (ingest emits with computed tax; settle re-confirms). This is by design: the pack version at emission (D-009) is the authoritative interpretation reference, and the engine demonstrates it computed under the current pack rather than relying on a stale computation.

---

## 9. Receipt Issuance

Checkout uses the Document Engine (CN-4-012) as a service primitive — it composes, does not re-implement.

### Template

Each tenant + receipt type uses a versioned template (CN-4-012 §4). The template is selected at issuance; its version reference is recorded on `checkout.receipt.issued.v1`. Old receipts continue to reference their original template version forever; template upgrades affect only future issuances.

### Numbering

CN-4-012 §3 provides deterministic, fiscal-compliant, branch/terminal-scoped numbering. Checkout receipts are numbered per (tenant, receipt-type, site) — matching the `scope_policy: site` declaration of the engine. Format is configured by the compliance pack (e.g., `RCT-KRK01-{SEQ:5}` for Karakana site 01 with five-digit sequence).

### Hash

CN-4-012 §5 specifies the hash. Checkout supplies the assembled content; the Document Engine computes the hash over `content + template_version_ref + pack_version_ref + document_number` and stores it on the event. The hash is independently verifiable (offline, via QR scan) per Term 7's external surface (CTR-017 / Term 7).

### DEGRADED — Block-Reserved Numbering (M3, D-007 #12)

When the central numbering allocator is unreachable (DEGRADED, CN-4-017), each site uses its pre-reserved block (CN-4-012 §3). Receipts issued in DEGRADED are numbered sequentially within the block; on recovery, the block is reconciled per CN-4-012 §3 (gap-free or gap-tolerant per jurisdiction). The settlement workflow itself is unaffected; only the numbering source changes.

---

## 10. Credit Tender → Obligation

A credit tender represents amount owed by the customer — payment on terms.

When `checkout.settle.request` is processed and one or more confirmed tenders have `method = credit`, the atomic emission at Phase 4 includes `obligation.created.v1` (Obligation primitive, CN-4-011). The Obligation:

| Field | Value |
|-------|-------|
| `obligation_id` | Newly assigned |
| `kind` | `accounts_receivable` (default) or `credit_refund` (rare — refund of a previous credit overpayment) |
| `debtor` | The customer party (from the bill or settlement context) |
| `creditor` | The tenant |
| `amount` | The credit-tender amount |
| `currency` | Settlement currency |
| `original_amount` | Equals `amount` at creation |
| `outstanding` | Equals `amount` (UI-08 bounds: in [0, original_amount]) |
| `linked_to` | The `checkout.settled.v1` event ID |

Subsequent settlement of this obligation — when the customer later pays — happens through other engines (Cash Engine records the settlement payment; the obligation primitive's fold reduces outstanding; Accounting journals the receipt). Checkout is not involved beyond creating the Obligation.

The `checkout.settled.v1` payload's `obligation_ref` carries the new obligation_id so subscribers can join settlement → obligation cleanly.

---

## 11. Currency & UI-06 Enforcement

UI-06 (CN-5-102) requires every primary financial event for a tenant to use the tenant's declared functional currency, unless an explicit FX event bridges. Checkout enforces this at multiple points:

| Boundary | Check |
|----------|-------|
| `checkout.ingest_bill.request` | All line `unit_price.currency` values must equal the tenant's functional currency (resolved from CTR-027 registry). |
| `checkout.tender.add.request` | The tender's currency must equal the settlement's currency (M1). Mismatched tender is rejected. |
| `checkout.settle.request` | The bus re-verifies the payload's `currency` field; emission with a non-functional currency is rejected unless paired with explicit FX events (CN-5-102 UI-06 allowance, defined per per-engine docs as needed). |

Multi-currency businesses are a future concern (CN-4-021 §8 open item; CN-5-009 §14 open item). Currently, single-currency settlements are the only supported case.

---

## 12. Example — Karakana Settles a Window

*Karakana ya Mzee Hassan in Mwanza serves as illustrative context, per D-004 #4 (peer-technical audience). The example shows the engine contract operating; it is not a UX description.*

A workshop completes a custom aluminium window for a customer. The workshop's vertical engine produces three Saleable Lines (matching CN-4-021 §6):

| line_id | description | qty | unit_price | tax_treatment_ref | line_total (cache) |
|---------|-------------|-----|------------|-------------------|---------------------|
| L1 | Aluminium frame 1200×900mm | 1 | TZS 85,000 | tz-vat-standard | 85,000 |
| L2 | Glass panel 5mm clear | 2 | TZS 22,000 | tz-vat-standard | 44,000 |
| L3 | Installation labour | 1 | TZS 30,000 | tz-vat-exempt | 30,000 |

### Phase 1 — Ingest

```
Workshop emits:    workshop.bill.ready.v1
                    { lines: [L1, L2, L3], site_id: karakana-mwanza-001, ... }

Checkout subscribes (subscribes_to entry: workshop.bill.ready.v1, site scope)
  → handler ingest_bill submits checkout.ingest_bill.request

Bus accepts; recomputes (K2):
  L1 line_total = 1 × 85000 = 85000 ✓
  L2 line_total = 2 × 22000 = 44000 ✓
  L3 line_total = 1 × 30000 = 30000 ✓

Bus resolves pack: tz-compliance-2026.05 (tenant: mzee-hassan-karakana, jurisdiction: TZ)
Bus computes tax via sandboxed evaluator (CN-4-015):
  L1 tax: 85000 × 18% = 15300
  L2 tax: 44000 × 18% = 7920
  L3 tax: 0 (exempt)
Pre-tax total: 159000. Tax total: 23220. Grand total: 182220.

Bus emits: checkout.started.v1
  { stream_id: checkout-2026-05-15-karakana-mwanza-001-0087,
    site_id: karakana-mwanza-001, business_date: 2026-05-15,
    currency: TZS, lines: [...], pre_tax_total: 159000,
    tax_breakdown: [...], tax_total: 23220, grand_total: 182220,
    cashier_session_id: krk-session-2026-05-15-A,
    source_tag: workshop-quote-0087 (opaque, K1) }
```

### Phase 2 — Cash Tender

```
Cashier:           checkout.tender.add.request
                    { tender_id: T1, method: cash, amount: 100000, currency: TZS }

Bus validates currency (M1): TZS = TZS ✓
Bus validates method (CTR-028 registry): cash is synchronous ✓
Bus emits:         checkout.tender.confirmed.v1
                    { tender_id: T1, method: cash, amount: 100000 }
```

### Phase 3 — Mobile Money Tender (async)

```
Cashier:           checkout.tender.add.request
                    { tender_id: T2, method: mobile_money, amount: 82220, currency: TZS,
                      external_ref: <customer-supplied phone> }

Bus validates; method is async.
Bus emits:         checkout.tender.requested.v1
                    { tender_id: T2, method: mobile_money, amount: 82220,
                      callback_correlation_id: cck-T2-44821 }

M-Pesa adapter (Term 7) subscribes; invokes M-Pesa; receives confirmation.
M-Pesa adapter emits:  mpesa.tender.outcome.v1
                        { tender_id: T2, callback_correlation_id: cck-T2-44821,
                          outcome: confirmed, external_ref: MPESA-TXN-44821 }

Checkout subscribes (record_tender_outcome handler):
  → submits checkout.tender.confirm.request

Bus emits:         checkout.tender.confirmed.v1
                    { tender_id: T2, method: mobile_money, amount: 82220,
                      external_ref: MPESA-TXN-44821 }
```

### Phase 4 — Settle

```
Cashier:           checkout.settle.request (explicit per K5)

Bus verifies:
  - All tenders terminal? T1 confirmed, T2 confirmed — yes ✓
  - sum(confirmed) ≥ grand_total? 100000 + 82220 = 182220 ≥ 182220 ✓
  - No outstanding tender.requested? Yes ✓
  - Pack version unchanged since started.v1? Yes ✓

Bus computes change = 0; no credit tender present, so no Obligation.
Bus invokes Document Engine for receipt:
  Template: workshop-receipt-v3
  Pack: tz-compliance-2026.05
  Number: RCT-KRK01-00187 (per-site sequence)
  Hash: computed over content + template_ref + pack_ref + number

Bus atomically emits (CN-4-004 §2):
  1. checkout.settled.v1                  (the hinge)
  2. checkout.receipt.issued.v1           (document terminal fold)

checkout.settled.v1 payload:
  site_id: karakana-mwanza-001, business_date: 2026-05-15, currency: TZS,
  lines: [L1, L2, L3], pre_tax_total: 159000, tax_breakdown: [...], tax_total: 23220,
  grand_total: 182220, tenders: [T1 cash 100000, T2 mobile_money 82220 MPESA-TXN-44821],
  tenders_total: 182220, change: 0, obligation_ref: null,
  receipt_ref: RCT-KRK01-00187, source_tag: workshop-quote-0087, cashier_session_id: krk-session-2026-05-15-A
```

### Downstream

Per CN-5-100 §8 fan-out: Accounting (`scope_ref: tenant`), Cash (`scope_ref: site`), and Inventory (`scope_ref: site`) each subscribe to `checkout.settled.v1` independently. Each derivative event (`accounting.journal.posted.v1`, `cash.tender.received.v1` ×2, `inventory.stock.deducted.v1` ×N) carries `causation_id = <event_id of checkout.settled.v1>` (UI-01). Reporting subscribes too, updating its month-to-date projection.

### Refund Variant

Suppose three days later the customer reports a defect; Mzee Hassan offers a partial refund of TZS 22,000 (one of the glass panels).

```
Cashier:           checkout.refund.request
                    { original: <event_id of checkout.settled.v1>,
                      refund_kind: partial,
                      lines_refunded: [L2 with qty 1, refund_amount 22000 + tax 3960 = 25960],
                      refund_tenders: [{method: cash, amount: 25960}],
                      reason_ref: defect-claim }

Bus validates:
  - UI-05: refund.business_date is 2026-05-18 (current open period; not back-dating into April) ✓
  - UI-06: currency TZS = TZS ✓
  - Pack at original settlement: tz-compliance-2026.05 — used for tax reversal computation ✓

Bus emits:         checkout.refunded.v1
                    { compensates_event_id: <original settled event_id>,
                      refund_kind: partial,
                      lines_refunded: [L2 partial],
                      pre_tax_refund: 22000, tax_total_reversed: 3960, refund_total: 25960,
                      refund_tenders: [...], refund_receipt_ref: CR-KRK01-00012,
                      obligation_adjustment: null }
                   checkout.receipt.issued.v1 (credit note CR-KRK01-00012)

Downstream engines (Accounting, Cash, Inventory) consume the compensation event
via their kind: compensation subscriptions (CN-5-100 P4):
  - Accounting reverses TZS 22000 revenue + TZS 3960 tax journal entries
  - Cash records TZS 25960 outflow at karakana-mwanza-001
  - Inventory restores 1 glass panel to stock at karakana-mwanza-001
```

UI-02 compensation symmetry holds: each engine that participated in the original `checkout.settled.v1` choreography produces a symmetric reverse effect for the refunded portion. The partial-refund mapping logic for each engine is per-engine doc territory (CN-5-001 / CN-5-002 / CN-5-003).

---

## 13. Three Whys

### Why does this matter?

Checkout is the moment money changes hands. Every other universal engine — Accounting, Cash, Inventory, Reporting, Promotion — derives its truth from what happens here. If Checkout is even slightly wrong — a mismatched currency, a stale tax rate, an unverified line total, a premature settlement emission while async tenders are still pending — the error propagates everywhere. CN-5-009 makes the moment of payment a strictly-disciplined, event-sourced, deterministically-priced, pack-frozen transaction. The hinge holds.

### Why does it belong here (and not in Foundation)?

Foundation provides the materials: the shapes (CN-4-021), the bus (CN-4-004), the document engine (CN-4-012), the compliance evaluator (CN-4-015), the obligation primitive (CN-4-011), the time authority (CN-4-014). Foundation does not say "tenders are collected sequentially; settle is explicitly cashier-triggered; credit tenders create obligations; refunds are compensating events." Those are universal-engine design choices appropriate for any business that takes payment. CN-5-009 makes those choices for the universal Checkout pattern. A different engine (a hypothetical "auction-checkout" with batch settlement) could be built differently using the same Foundation materials — but that is not what universal checkout is.

### Why this design?

A four-phase event-sourced workflow with K5's explicit settlement trigger handles three realities simultaneously: synchronous tenders (cash), asynchronous tenders (mobile money, card), and multi-tender combinations. The Document Engine integration centralises receipt issuance and inherits the hash/number/freeze guarantees Foundation already provides. The compensating-event refund pattern propagates correctness through downstream engines automatically via CN-5-100 P4 — no separate refund choreography to maintain. The per-vertical subscription pattern (one manifest entry per Term 6 vertical) keeps Checkout vertical-blind while supporting arbitrary new verticals. The currency check at every command boundary makes UI-06 enforcement structural, not aspirational. The whole engine is a small composition over strong primitives.

---

## 14. Boundaries

| Topic | Lives in |
|-------|----------|
| Saleable Line and Tender value shapes | CN-4-021 |
| Document hash, numbering, template versioning, terminal fold | CN-4-012 |
| Obligation primitive (creation, fold, bounds) | CN-4-011 (Obligation) |
| Tax rate values, tax treatment rules, pack content | Compliance packs via CN-4-015 |
| Sandboxed evaluator that runs tax rules | CN-4-015 §4 |
| Time authority (record time vs business date; fiscal zone) | CN-4-014 |
| Command bus, atomic multi-event emission, rejection policy | CN-4-004 |
| Event envelope (causation_id, compensates_event_id, pack_version_ref) | CN-4-002 |
| Subscription patterns, replay semantics, causation linkage | CN-5-100 |
| Scope policy (site default for Checkout) | CN-5-101 |
| Cross-engine invariants (UI-01 causation, UI-04 tender chain, UI-05 closed period, UI-06 currency, UI-09 site_id) | CN-5-102 |
| Vertical bill construction (basket, table bill, folio, quote) | Term 6 (per-vertical docs) |
| Checkout UX (cashier screen) | Term 3 (CTR-004) |
| Payment adapter internals (M-Pesa, card processors) | Term 7 (CTR-006) |
| Tender method vocabulary + registry governance | Term 7 + Term 1 (CTR-028) |
| Tenant functional currency registry | Term 1 + Term 2 (CTR-027) |
| Tenant site registry (site_id resolution) | Term 1 + Term 2 (CTR-027) |
| Promotion rule definitions and bill-level discounts | CN-5-007 |
| Downstream engine effects (Accounting journals, Cash receipts, Inventory deductions) | CN-5-001 / CN-5-002 / CN-5-003 |
| Period close interaction (UI-05) | CN-5-001 + CN-5-104 |
| Document verification public surface (QR, offline portal, authenticity) | Term 7 (CTR-017) |

---

## 15. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Partial-refund symmetric compensation mapping per engine (M2 / UI-02) | CN-5-001 (Accounting) + CN-5-002 (Cash) + CN-5-003 (Inventory) + Architect | CN-5-009 produces the trigger event; downstream engines map their compensation logic |
| Cashier authorisation roles for refund (who may submit `checkout.refund.request`) | Term 1 (governance) + Architect | Per-tenant role policy; not in scope here |
| Adapter cancellation contract (when Checkout cancels mid-workflow, how does Term 7 reverse a pending adapter transaction?) | CTR-006 / Term 7 | Touched here but full contract is Term 7's |
| Tender method registry mechanism (where it lives, how it's queried at bus-policy time) | CTR-028 / Term 7 + Term 1 / Architect | Vocabulary governance + lookup mechanism |
| Multi-currency settlements (single-currency is the only supported case in v1) | Term 5 + Term 7 (future) | CN-4-021 §8 open item; CN-5-009 v1 enforces single-currency |
| Receipt template versioning per tenant (who maintains templates, how versions roll out) | Architect + Term 1 (governance) | CN-4-012 §4 leaves this for Architect/Term 1 |
| Bill-level promotion integration (vs line-level discount_refs) | CN-5-007 (Promotion) | Phase 1 supports line-level only; bill-level deferred |
| Settlement under DEGRADED with async tender adapter impaired (cashier UX for stalled tenders) | Term 3 (CTR-004) + Architect | CN-5-009 stalls cleanly; the UX response is Term 3's |
| Performance budget for full workflow round-trip (ingest → settle) under load | Architect | Concept guarantees correctness; performance is Architect's |
| Discovery: doctrine check (CN-4-019) for K1 source_tag non-branching — confirm DC-022 covers Checkout specifically | CN-4-019 (already DC-022) | M4 verification — DC-022 already exists; CN-5-009 confirms it applies here |

---

*— End of CN-5-009 —*
