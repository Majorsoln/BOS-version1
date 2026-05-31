# CN-5-004 — Procurement Engine

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 5 — Universal Engines
> **Status:** For Overseer review.
> **Governing decisions:** Law 2 (engine isolation — Procurement interacts with Inventory/Accounting/Cash via events only); Law 5 (compliance is configured — approval thresholds, three-way-match variance bands, supplier credit-term defaults, input VAT rates per pack); D-009 (freeze doctrine — documents and AP interpreted under historical pack at issuance); UI-01 (causation chain — every lifecycle transition links to its trigger); UI-02 (compensation symmetry — Credit Note propagates symmetrically through Inventory + Accounting + Cash); UI-08 (Obligation bounds preserved across multi-engine settlements).
> **CTRs:**
> - **OPEN (this doc extends with expansion notes at merge):** CTR-029 (Term 1 — pack section gains: `procurement_approvals.thresholds`, `three_way_match.variance_thresholds`, supplier credit-term defaults, multi-currency procurement FX rules); CTR-030 (Term 6 — verticals may emit `<vertical>.purchase_need.recorded.v1` events for vertical-driven procurement).
> - **OPEN (this doc depends on):** CTR-027 (Term 1/2 — tenant functional currency + site registry).
> - **Open Item (future):** CTR-033 (Term 7 — supplier-onboarding integration: TIN verification, import compliance) — to be filed when pack content explicitly requires external verification.
> **Glossary:** See `MASTER-GLOSSARY.md` — Universal-engine Invariant (UI), Site, Scope level, Obligation, Document.
> **Depends on:** CN-4-002 (envelope, causation_id, compensates_event_id); CN-4-004 (command bus + atomic emission + policy rejection); CN-4-011 (Workflow primitive for lifecycle, Approval primitive for single-step gates, Obligation primitive for AP, Document primitive for PO/GRN/Invoice/Voucher/Credit Note, Party primitive for supplier records); CN-4-012 (Document Engine — issuance, hash, numbering, freeze); CN-4-014 (clock protocol — business_date, due_date computation); CN-4-015 (Compliance DSL — tax, approval thresholds, supplier-type policies); CN-5-009 (Checkout context — payment medium for supplier settlement aligns with tender taxonomy); CN-5-001 (Accounting auto-journals on procurement events #6 / #7 / #8); CN-5-002 (Cash settles AP Obligations; pays suppliers); CN-5-003 (Inventory subscribes `procurement.grn.received.v1` and consumes return events); CN-5-100 (subscription patterns — auto-journal P3, compensation P4); CN-5-101 (scope policy — Procurement is multi-scope per §6: site default + tenant overrides for supplier records, credit terms, high-value approval, payment scheduling); CN-5-102 (UI-01 causation, UI-02 compensation symmetry, UI-08 obligation bounds).
> **Boundaries:** Workflow / Approval / Obligation / Document / Party primitive folds → CN-4-011; Document hash + numbering + freeze → CN-4-012; tax computation grammar → CN-4-015 + compliance packs; stock storage / lots / costing → CN-5-003; AP journal mappings → CN-5-001 §4; payment medium and AP cash settlement → CN-5-002 §8 / §G; petty / informal cash-and-carry expense pathway (no PO, no supplier invoice) → CN-5-002 §7 (see §11 boundary note); vertical-specific procurement logic → Term 6; supplier verification services (TIN check, import compliance) → Term 7 future (CTR-033 Open Item).

---

## 1. What This Doc Defines

CN-5-004 is the Procurement Engine — the **buying lifecycle truth layer** for any business. Karakana buys aluminium, Café buys beans, Pharmacy buys medicines, Retail buys produce — every business with suppliers passes through the same lifecycle: Requisition → PO → GRN → Invoice → Payment. CN-5-004 implements that lifecycle universally, supporting partial flows, multi-currency, supplier credit terms, three-way match, approval workflows, and returns.

The engine **orchestrates** rather than owns the downstream effects. Procurement creates AP Obligations (CN-4-011); Cash settles them. Procurement emits GRN events; Inventory receives stock. Procurement emits invoice events; Accounting auto-journals. The engine is the buying intent + document chain + workflow status; the financial and physical effects belong to the engines that own those domains.

This doc defines:
- The **engine manifest** with multi-scope per-operation overrides (§3).
- **Supplier management** via the Party primitive and credit-terms projection (§4).
- The **lifecycle workflow** — 5 core phases plus optional Phase 0 RFQ (§5).
- The **document chain** — PO, GRN, Invoice Recording, Payment Voucher, Credit Note (§6).
- **Three-way match logic** with pack-driven variance thresholds (§7).
- The **AP Obligation lifecycle** — creation at invoice; partial settlement; UI-08 bounds (§8).
- **Approval workflows** — Approval primitive for single-step, Workflow primitive for multi-step (§9).
- **Partial flows** — multiple GRNs per PO, multiple invoices per GRN portion, multiple payments per invoice (§10).
- **Direct expense vs inventory-tracked procurement** — one lifecycle, two effects (§11).
- **Returns to supplier** — Credit Note compensation propagating across engines (§12).
- **Multi-currency procurement** — AP in supplier currency; FX coordination (§13).
- **Tax / input VAT** — pack-driven recognition (§14).
- **Cross-engine subscription map** (§15).
- **UI-01 + UI-02 + UI-08 enforcement** (§16).

This doc does NOT define:
- Primitive folds (CN-4-011).
- Document mechanics (CN-4-012).
- Compliance DSL grammar (CN-4-015).
- Stock storage, lots, costing (CN-5-003).
- AP journal mappings (CN-5-001).
- Payment medium or AP cash settlement (CN-5-002 §8).
- Petty cash-and-carry expense without PO (CN-5-002 §7 — see §11 boundary note).
- Vertical-specific procurement logic (Term 6).
- Supplier verification services (CTR-033 future).

---

## 2. The Seven Laws

| # | Law | Source / why |
|---|-----|--------------|
| **P1** | **Lifecycle is the truth.** Every purchase passes through deterministic Workflow primitive (CN-4-011) states. Phases cannot be skipped; transitions are append-only event-sourced. State at any moment = fold of transition events. | Charter §1.2 audit completeness; supplier dispute resolution requires complete trail; CN-4-011 Workflow guarantee. |
| **P2** | **Document at every binding transition.** Each phase transition that creates a legally-binding artefact (PO issued, GRN signed, Invoice recorded, Payment authorised, Credit Note issued) emits a Document primitive event (CN-4-012) — hashed, numbered, frozen at pack/template version. | D-009 freeze; legal defensibility (Charter §1.2); supplier/auditor verification. |
| **P3** | **Three-way match before payment authorisation.** PO ↔ GRN ↔ Invoice are matched on quantity, price, and item. Pack-defined variance thresholds determine auto-pass / flag / hard-block per dimension. Unmatched invoices cannot proceed to payment authorisation. | Audit-proof — payment without match indicates fraud, error, or scope creep. Pack-driven thresholds honour Law 5. |
| **P4** | **AP via Obligation primitive (UI-08).** Accounts Payable is **not Procurement engine state** — it lives in the Obligation primitive (CN-4-011, kind: `accounts_payable`). Procurement creates the Obligation at invoice recording; Cash settles via `cash.payment.disbursed.v1`. UI-08 bounds enforced (`outstanding ∈ [0, original_amount]`). | Unified across engines (AR in CN-5-002, delivery in CN-5-003, advance/layby in CN-5-002, AP here). One primitive for "owed" relationships; engines record money/document sides. |
| **P5** | **Compliance via pack (approvals, tax, supplier types, credit-term defaults).** Approval thresholds per amount band, input VAT rates per supplier type, supplier credit-term defaults per jurisdiction, three-way-match variance bands — all pack-driven. Tenant overrides within pack-defined permission (typically ±50%). | Law 5; D-009 freeze; CTR-029 expansion governs content per jurisdiction. |
| **P6** | **Returns via compensating documents.** Returning goods to supplier issues a Credit Note — a new Document with terminal fold (CN-4-012 §6) — referencing the original Invoice. The Credit Note event carries `compensates_event_id` to the original invoice; downstream engines (Inventory, Accounting, Cash) react via `kind: compensation` subscriptions. UI-02 symmetry enforced. | CN-4-011 Document terminal fold; CN-5-100 P4 compensation pattern; UI-02 propagation. |
| **P7** | **Direct expense vs inventory-tracked: one lifecycle, two effects.** A PO for office paper (consumed at receipt) and a PO for goods-for-resale follow the **same lifecycle**. The difference is at GRN: the receipt event carries an `inventory_effect` flag (`tracked` or `expense_at_receipt`). Inventory subscribes only when tracked; Accounting auto-journals to Inventory or Expense per flag. | Universality — one engine for all formal procurement. Differences live in payload, not in lifecycle. (Note: informal cash-and-carry expenses without PO use CN-5-002 §7 — see §11 boundary.) |

---

## 3. Engine Manifest

Procurement is a **multi-scope engine** per CN-5-101 §6: `site` default with explicit `tenant`-scope overrides for tenant-wide identity and high-value operations.

```yaml
engine_id:        procurement
display_name:     Procurement Engine
version:          1
scope_policy:     site                                  # default — most SME order per-site
primitives_used:
  - workflow                                            # lifecycle state machine (CN-4-011)
  - approval                                            # single-step approval gates (CN-4-011)
  - obligation                                          # AP creation (CN-4-011)
  - document                                            # PO/GRN/Invoice/Voucher/Credit Note (CN-4-011 + CN-4-012)
  - party                                               # supplier records (CN-4-011)

commands:
  # Supplier management (tenant — supplier identity is tenant-wide)
  - id: procurement.supplier.register.request           scope_ref: tenant
  - id: procurement.supplier.attributes_update.request  scope_ref: tenant
  - id: procurement.supplier.deactivate.request         scope_ref: tenant
  - id: procurement.supplier.credit_terms_set.request   scope_ref: tenant

  # Optional Phase 0 — RFQ / Quote
  - id: procurement.rfq.issue.request                   # scope_ref: site (BOS-issued Document)
  - id: procurement.quote.record.request                # scope_ref: site (record supplier's quote)

  # Phase 1 — Requisition
  - id: procurement.requisition.create.request          # scope_ref: site
  - id: procurement.requisition.approve.request         # scope_ref: site (or tenant for high-value bands)
  - id: procurement.requisition.reject.request          # scope_ref: site

  # Phase 2 — PO
  - id: procurement.po.create.request                   # scope_ref: site — from approved requisition
  - id: procurement.po.amend.request                    # scope_ref: site
  - id: procurement.po.cancel.request                   # scope_ref: site

  # Phase 3 — GRN
  - id: procurement.grn.record.request                  # scope_ref: site

  # Phase 4 — Invoice
  - id: procurement.invoice.record.request              # scope_ref: site — creates AP Obligation
  - id: procurement.invoice.dispute.request             # scope_ref: site
  - id: procurement.invoice.resolve_dispute.request     # scope_ref: site

  # Phase 5 — Payment (Procurement schedules / authorises; Cash actually pays)
  - id: procurement.payment.schedule.request            # scope_ref: tenant
  - id: procurement.payment.authorize.request           # scope_ref: tenant

  # Returns (Compensation)
  - id: procurement.return.initiate.request             # scope_ref: site
  - id: procurement.credit_note.record.request          # scope_ref: site

emits:
  # Supplier
  - procurement.supplier.registered.v1
  - procurement.supplier.attributes_updated.v1
  - procurement.supplier.deactivated.v1
  - procurement.supplier.credit_terms_set.v1

  # Quote / RFQ (optional Phase 0)
  - procurement.rfq.issued.v1                           # BOS-issued Document (CN-4-012)
  - procurement.quote.recorded.v1                       # supplier-owned, recorded by BOS (reference)

  # Requisition
  - procurement.requisition.created.v1
  - procurement.requisition.approved.v1
  - procurement.requisition.rejected.v1

  # PO
  - procurement.po.created.v1                           # Document issued (CN-4-012)
  - procurement.po.amended.v1
  - procurement.po.cancelled.v1

  # GRN
  - procurement.grn.received.v1                         # → CN-5-003 Inventory (P3)
  - procurement.grn.discrepancy.detected.v1
  - procurement.grn.compensated.v1                      # for returns (CN-5-003 subscribes compensation)

  # Invoice
  - procurement.invoice.recorded.v1                     # → CN-5-001 Accounting (#6); creates AP Obligation
  - procurement.invoice.matched.v1                      # three-way match passed
  - procurement.invoice.dispute_raised.v1
  - procurement.invoice.dispute_resolved.v1

  # Payment lifecycle (Procurement-side view; Cash does actual cash flow)
  - procurement.payment.scheduled.v1
  - procurement.payment.authorized.v1
  - procurement.invoice.paid.v1                         # workflow status update on Cash settlement
  - procurement.invoice.partially_paid.v1               # partial settlement status

  # Returns
  - procurement.return.initiated.v1
  - procurement.credit_note.recorded.v1                 # compensating Document (CN-4-012 §6)
  - procurement.return.completed.v1

subscribes_to:
  # Cash settlement events (N1 — consolidated handler with internal routing on obligation.kind)
  - {event_type: cash.payment.disbursed.v1,             version: 1, handler: handle_cash_payment, kind: command_emitting, scope_ref: tenant}
    # Internal routing inside handler:
    #   IF payload.obligation_ref.kind == accounts_payable → mark_invoice_paid path
    #   IF payload.obligation_ref.kind == credit_refund (supplier refund) → mark_credit_note_received path
    # Single subscription, single handler; routing is engine-internal logic, not separate subscribers.

  # Future: vertical purchase needs (CTR-030 expansion — per-vertical entries grow manifest)
  - {event_type: <vertical>.purchase_need.recorded.v1,  version: 1, handler: ingest_purchase_need,  kind: command_emitting, scope_ref: site}

  # Future: inventory reorder triggers (deferred until CN-5-003 v2 adds reorder-point events)
  # - {event_type: inventory.reorder_point.reached.v1, version: 1, handler: ingest_reorder_need, ...}

requires:
  - cash.payment.disbursed.v1                           # cannot complete lifecycle without payment confirmation
```

### Why Consolidate the Cash-Payment Subscription (N1)

`cash.payment.disbursed.v1` is the same event whether it settles an AP or processes a supplier refund (cash inflow against a credit-refund obligation). Declaring two separate subscriptions with different handlers conflates manifest declaration with internal routing logic. The cleaner pattern: **one subscription, one handler**, with engine-internal routing on `obligation_ref.kind`. The manifest stays focused on event types; the routing — which is engine business logic — lives inside the handler.

---

## 4. Supplier Management

Suppliers are recorded as Party primitive instances (CN-4-011) with Procurement-specific projection state.

### Supplier Record

```yaml
supplier:
  party_ref:               <Party primitive ID>
  tin_or_business_id:      <jurisdiction-specific identifier>
  contact:                 {phone, email, physical_address_ref}
  supplier_type_ref:       <pack-defined: domestic_supplier | import_supplier | service_provider | …>
  default_currency:        <ISO code; supplier's billing currency>
  active_status:           active | suspended | deactivated
  attributes:
    delivery_lead_time_days: integer | null
    preferred_payment_methods: [cash, mobile_money, bank_transfer]
```

### Credit Terms

Credit terms live in a Procurement engine projection separate from the Party identity:

```yaml
procurement.supplier.credit_terms_set.v1
  payload:
    supplier_ref:              <Party>
    terms_kind:                cod | net | discount_net
    days_to_pay:               30                    # for net 30
    early_payment_discount:                          # for 2/10 net 30
      percent:                 2.0
      days_for_discount:       10
    pack_version_ref:          <D-009 — terms governed by pack rules>
    effective_from:            <business_date>
```

Multiple terms records may exist for one supplier (over time); the active terms at invoice recording determine the Obligation's `due_date` and discount eligibility. Historical terms remain for replay.

### Why Credit Terms in Projection, Not Party Primitive

Party primitive is identity-only (CN-4-011) — name, contact, type tags. Credit terms are commercial relationship state that evolves over time and varies per tenant-supplier relationship. Keeping them in Procurement's projection avoids unbounded Party primitive extension and lets the engine version terms cleanly.

---

## 5. Lifecycle Workflow Phases (Q1)

5 core phases plus optional Phase 0. Each phase is a Workflow primitive (CN-4-011) state.

```
[Phase 0: rfq_issued → quote_recorded* → quote_selected]    (OPTIONAL — per-purchase opt-in)
            │
            ▼
[Phase 1: requisition_drafted → requisition_approved | requisition_rejected]
            │ (approved)
            ▼
[Phase 2: po_drafted → po_issued → po_amended* → po_cancelled?
                                 → po_partially_received → po_fully_received]
            │ (po_issued)
            ▼
[Phase 3: grn_partial* → grn_fully_received | grn_with_discrepancy]
            │
            ▼
[Phase 4: invoice_recorded → invoice_matched | invoice_disputed → invoice_resolved]
            │ (matched)
            ▼
[Phase 5: payment_scheduled → payment_authorized → invoice_partially_paid* → invoice_paid]
            │
            ▼
[completed]
```

(`*` indicates a state that may repeat.)

### Optional Phase 0 — RFQ / Quote

For larger or competitive sourcing purchases:
- `procurement.rfq.issue.request` — BOS issues an RFQ document (per N2 — this is a BOS-issued document, hashed and numbered per CN-4-012, sent to one or more suppliers).
- `procurement.quote.record.request` — record each supplier's response (the quote is owned by the supplier; BOS records the reference, analogous to invoice recording §6).

Comparison and selection happen at the UX layer (Term 3); the chosen quote triggers requisition creation referencing the quote ID. This phase is **per-purchase opt-in** — small POs skip directly to Phase 1.

### Approval Timing in RFQ Case (N3)

RFQ itself does not bind the business commercially — it is a market scan. Default behaviour: **no pre-RFQ approval gate**; approval happens at requisition stage (post-quote, when the business is about to commit). Some jurisdictions (e.g., government procurement) may require pre-RFQ authorisation; the pack may opt in to a `procurement.rfq.approve.request` gate under that jurisdiction's rules. Default packs do not include this gate.

### Partial States

Phases 2, 3, and 5 admit partial states reflecting operational reality:
- **PO** stays `partially_received` while some line quantities remain outstanding.
- **GRN** sequence (`grn_partial → grn_partial → … → grn_fully_received`) supports deliveries arriving in tranches.
- **Invoice** payment cycles through `partially_paid` as Cash settles in instalments; advances to `paid` only when AP outstanding = 0.

The state machine accommodates these via Workflow primitive's transition history; truth = fold over events.

---

## 6. Document Chain (CN-4-012 Integration)

Every legally-binding phase transition issues a Document via CN-4-012:

| Document | Issued at event | Issued by BOS or recorded? | Number scope |
|----------|-----------------|----------------------------|--------------|
| **RFQ** (optional, N2) | `procurement.rfq.issued.v1` | **Issued by BOS** (sent to suppliers) | tenant + RFQ type + site |
| **Quote** (optional, N2) | `procurement.quote.recorded.v1` | **Recorded** (owned by supplier; BOS records reference + content) | tenant + Quote type + site |
| **PO** | `procurement.po.created.v1` | **Issued by BOS** (sent to supplier) | tenant + PO type + site |
| **GRN** | `procurement.grn.received.v1` | **Issued by BOS** (proof of receipt) | tenant + GRN type + site |
| **Invoice Recording** | `procurement.invoice.recorded.v1` | **Recorded** (owned by supplier; BOS records reference + content) | tenant + Invoice type + site |
| **Payment Voucher** | `procurement.payment.authorized.v1` | **Issued by BOS** (internal authorisation document) | tenant + Voucher type + site |
| **Credit Note** | `procurement.credit_note.recorded.v1` | **Recorded if supplier-issued** OR **Issued by BOS** if return-driven — depends on which side initiates | tenant + Credit Note type + site (terminal fold, references original Invoice) |

### Issued vs Recorded Distinction (N2)

- **Issued by BOS**: BOS controls the content, computes the hash, assigns the number. RFQ, PO, GRN, Payment Voucher fall here. The hash guarantees BOS-side integrity.
- **Recorded**: an external party (the supplier) owns the document; BOS records a reference document containing the supplier's metadata (their invoice/quote/credit-note number, date, content). The hash BOS computes covers BOS's record, not the supplier's underlying document. Quote and Invoice Recording fall here. Credit Notes may go either way depending on initiator.

This distinction matters for verification: a BOS-issued document is fully verifiable from BOS's hash chain alone; a recorded document is verifiable for tampering of BOS's record, but the supplier's underlying document's authenticity depends on the supplier (signature, supplier's own systems).

### Document Versioning + Freeze (D-009)

Each document references the pack version + template version at issuance. Template upgrades apply only to new documents; old documents remain under their original template forever (CN-4-012 §4).

---

## 7. Three-Way Match Logic (P3, Q3)

When `procurement.invoice.record.request` is processed, the bus invokes the three-way match engine policy (CN-4-004 §3).

### Match Algorithm

```
Inputs:
  PO           = referenced procurement.po.created.v1
  GRNs         = all procurement.grn.received.v1 events linked to this PO
  Invoice      = the candidate invoice being recorded

Per-line match (for each invoice line):
  Item match:     invoice.item_ref ∈ PO.items                       (exact at pack default;
                                                                     "substitute_with_evidence" permitted at pack discretion)
  Quantity match: invoice.qty ≤ Σ(GRN.qty for this item)
                  AND invoice.qty conforms to GRN portion being invoiced
  Price match:    invoice.unit_price within pack-defined variance of PO.unit_price

Pack-defined variance thresholds (per dimension):
  quantity_variance: {auto_pass: 0%,   flag: 5%,   hard_block: 10%}
  price_variance:    {auto_pass: 2%,   flag: 5%,   hard_block: 15%}
  item_match:        {auto_pass: exact, flag: substitute_with_evidence, hard_block: unknown_item}

Tenant override on thresholds: ±50% of pack default permitted (consistent with CN-5-002 §12).

Result:
  - All lines auto-pass → emit procurement.invoice.matched.v1; workflow advances to ready_for_payment
  - Any line flagged   → emit procurement.invoice.recorded.v1 with status: needs_review;
                          authorised role submits procurement.invoice.match.confirm.request
                          (overrides flag with documented reason)
  - Any line hard-blocked → emit procurement.invoice.dispute_raised.v1 automatically;
                            workflow → invoice_disputed; payment cannot proceed until resolved
```

### Disputes

Hard-blocked or escalated invoices enter dispute state.
- `procurement.invoice.dispute.request` carries reason and supporting evidence; emits `procurement.invoice.dispute_raised.v1`.
- `procurement.invoice.resolve_dispute.request` (authorised role) records the resolution — accept variance with documented reason, reject invoice (supplier must issue replacement), request supplier credit note for difference. Emits `procurement.invoice.dispute_resolved.v1`. Workflow either advances (accepted) or stays disputed pending further action.

Disputes are audited fully — every state, every decision, every actor in the trail.

---

## 8. AP Obligation Lifecycle (P4)

```
Invoice recorded:
  procurement.invoice.recorded.v1 emitted
  → Obligation primitive created:
      kind:              accounts_payable
      creditor:          supplier party
      debtor:            tenant
      original_amount:   invoice total
      outstanding:       invoice total
      currency:          invoice currency
      due_date:          derived from supplier credit terms (§4) + invoice business_date

Partial payment(s):
  cash.payment.record.request (one or many — CN-5-002 §G)
  → Each emits Obligation primitive's partial-settle event;
    outstanding reduces by paid amount (UI-08 bounds)
  → Procurement's handle_cash_payment handler (N1) subscribes cash.payment.disbursed.v1
    with obligation.kind = accounts_payable:
      IF outstanding > 0 after settlement → emit procurement.invoice.partially_paid.v1
      IF outstanding = 0 after settlement → emit procurement.invoice.paid.v1
      Workflow updates accordingly

Full payment:
  outstanding = 0
  → Obligation transitions to settled status (primitive fold)
  → Workflow → completed
  → procurement.invoice.paid.v1 emitted
```

### UI-08 Bounds Throughout

`outstanding ∈ [0, original_amount]` at every step (unless `kind: credit_refund` exception for supplier-side refunds). Bus rejects any settlement command that would breach.

### Why AP Is Not Procurement State

A receivable and the cash that settles it are operationally distinct concepts. The same applies to payables. Procurement records **what was bought and what is owed by document chain**; the Obligation primitive records **the relationship** ("we owe this supplier this amount until settled"); Cash records **the money side** of settlement events. Three concerns, three engines, one Obligation primitive coordinating. This is the same pattern CN-5-002 §8 uses for AR and CN-5-003 §13 uses for delivery obligations.

---

## 9. Approval Workflows (Q4)

Approval thresholds per amount band, pack-defined (CTR-029 expansion):

```yaml
# In pack content
procurement_approvals:
  thresholds:
    - {max_amount: 100000,    required_approvers: [cashier],          mechanism: approval_primitive}
    - {max_amount: 1000000,   required_approvers: [manager],           mechanism: approval_primitive}
    - {max_amount: 10000000,  required_approvers: [owner],             mechanism: approval_primitive}
    - {max_amount: null,      required_approvers: [owner, board],      mechanism: workflow_primitive,
                              multi_approver_logic: all_required}
  tenant_override_allowed:    true
  tenant_override_bounds:     ±50%
```

### Mechanism Selection

- **Single-approver bands** use the **Approval primitive** (CN-4-011): one request → one decision (approve/reject). Simple, audit-clean.
- **Multi-approver bands** use the **Workflow primitive** (CN-4-011): a defined sequence of approver stages, each emitting transition events.

Tenant role policy declares which roles fill the approver slots.

### Sequencing

Approval happens at **requisition stage (Phase 1)**, not at PO stage (Q4 ruling). The PO is the **outcome** of an approved requisition; it never needs separate approval. This avoids double-approval and keeps the lifecycle linear.

If a PO needs amendment that increases value above the originally-approved band, the `procurement.po.amend.request` triggers re-approval at the new band before the amendment takes effect.

### Pre-RFQ Approval (Optional, N3)

Default: no approval needed to issue an RFQ (RFQ is a market scan, not a commitment). Some jurisdictions (government procurement) require pre-RFQ approval; the pack opts in to a `procurement.rfq.approve.request` gate under those rules. Default packs do not include this.

---

## 10. Partial Flows

The lifecycle accommodates partial reality at multiple points (Q5):

| Scenario | Mechanism |
|----------|-----------|
| **Partial GRN** | One PO → multiple GRN events. Each GRN records a portion. PO state = `po_partially_received` until cumulative qty = PO qty. |
| **Multiple Invoices per PO** | One PO → one or more invoices, each invoicing a subset of received qty. Each invoice runs three-way match against its corresponding GRN portion. Each creates its own AP Obligation. |
| **Partial Payment** | One AP Obligation → multiple `cash.payment.record.request` events. Obligation primitive's fold reduces outstanding per settlement (UI-08 bounds). Procurement workflow status advances `invoice_partially_paid` → `invoice_paid` only at outstanding = 0. |
| **Mixed currencies** | An invoice in USD against a multi-currency till: payment in USD reduces USD Obligation directly. Payment via FX bridging from TZS till: paired FX event + USD payment to supplier. See §13. |
| **Amendments** | PO amendment changing qty/price emits `procurement.po.amended.v1` with new totals; if increases approval band, re-approval triggered. Existing GRN events retained; future GRNs match against amended PO. |

The state machine accommodates all these via Workflow primitive transition events; truth = fold over the events at any moment.

---

## 11. Direct Expense vs Inventory-Tracked Procurement (P7, Q7)

Same lifecycle for both. The difference appears at GRN:

```yaml
procurement.grn.received.v1
  payload:
    site_id, supplier_ref, po_ref, grn_doc_ref,
    items:
      - {line_ref, item_ref, qty, unit_cost,
          inventory_effect: tracked | expense_at_receipt,
          expense_category: <pack code if expense_at_receipt>}
```

| `inventory_effect` | Inventory subscribes? | Accounting auto-journal (CN-5-001 #8 / #6) |
|--------------------|-----------------------|---------------------------------------------|
| `tracked` | Yes — emit `inventory.stock.received.v1`; create lot if lot-tracked | Dr Inventory; Cr GR/IR (at GRN if invoice not yet) or Cr AP (at invoice recording) |
| `expense_at_receipt` | No — Inventory ignores; item never enters stock projection | Dr Expense (per `expense_category` pack mapping); Cr GR/IR or Cr AP |

### Boundary Note — Formal Procurement vs Petty Cash-and-Carry (N4)

**This engine handles formal procurement-with-supplier** — purchases with a PO + supplier invoice (even for direct expenses like office paper bought from an office supplies supplier).

**Petty / informal expenses** (cashier walks to corner shop for tea, no PO, no invoice, just a receipt) use **CN-5-002 §7 `cash.expense.record.request`** — site-level cash expense without procurement lifecycle.

**Decision rule:**
- Has PO + supplier invoice → CN-5-004 (formal procurement, even if `expense_at_receipt`).
- Cash-and-carry, no PO, no formal invoice → CN-5-002 §7 (generic operating expense pathway).

The same expense category vocabulary (CN-5-002 §7's 14 categories per N1) applies in both paths; the path differs by the document trail. This boundary prevents Procurement from being overloaded with informal small purchases that don't need its lifecycle apparatus, while keeping all formal supplier-relationship purchases (even direct expenses) under one auditable lifecycle.

---

## 12. Returns to Supplier — Credit Note Compensation (P6, Q8)

When goods received need returning (defective, wrong items, surplus to need):

### Flow

```
Step 1 — Initiate
  procurement.return.initiate.request
    payload: {grn_ref, items_to_return: [{line_ref, qty, reason_ref}], supporting_evidence_ref}
  → procurement.return.initiated.v1 emitted (workflow state: return_in_progress)

Step 2 — Credit Note Recording (supplier-issued) or Issuance (BOS-driven)
  procurement.credit_note.record.request
    payload:
      return_ref:                  <return_ref>
      supplier_credit_note_ref:    <supplier's document number, if supplier issued>
      items_credited:              [{line_ref, qty, unit_credit, total_credit}]
      total_credited
      currency
  → procurement.credit_note.recorded.v1 emitted with:
      compensates_event_id:  <original procurement.invoice.recorded.v1>      (UI-02 link)
      causation_id:          <credit note command derivative>                  (UI-01 link)
    A Credit Note Document is issued (terminal fold, references original invoice — CN-4-012 §6)

Step 3 — Cross-engine compensation (UI-02 symmetry)
  - Inventory subscribes procurement.credit_note.recorded.v1 (or procurement.grn.compensated.v1)
    → emits inventory.stock.adjusted.v1 (reduce stock, source_ref: return_to_supplier)
  - Accounting subscribes procurement.credit_note.recorded.v1 (kind: compensation)
    → Dr AP (reduce); Cr Inventory (reduce stock value) OR Cr Expense (reverse if direct expense)
  - If supplier refunds in cash (vs reducing AP):
    Cash records inflow via cash.collection.record.request with origin: supplier_refund
    (UI-04 chain extends: pack rule adds "supplier_refund" to authorised-origin set)

Step 4 — Completion
  procurement.return.completed.v1 emitted; workflow state: return_closed.
  Original invoice's AP Obligation either reduces (offset against future invoices) or
  is fully credited (and supplier refunds via cash).
```

### Variant — Return Caught at GRN (Pre-Invoice)

If discrepancy is caught at GRN inspection (e.g., 5 of 50 sheets warped on arrival), GRN records only the accepted qty + emits `procurement.grn.discrepancy.detected.v1` with notes. The invoice (when received) reflects only the accepted qty; no separate Credit Note needed unless supplier already invoiced full qty.

### Variant — Return Post-Payment

The Credit Note may produce a cash refund (supplier wires money back; Cash records inflow with `supplier_refund` origin per pack-extended authorised-origin set) or be applied against the next invoice (offset). Both flows are supported; pack rules govern.

---

## 13. Multi-Currency Procurement (Q9)

Supplier invoices may be in supplier's currency (USD) while tenant functional currency is TZS.

### AP in Supplier Currency

The AP Obligation is created in the **supplier's currency**:

```yaml
procurement.invoice.recorded.v1
  payload:
    invoice_amount:        50000
    invoice_currency:      USD
    ...
  → Obligation primitive:
      original_amount:     50000 USD
      currency:            USD
      outstanding:         50000 USD
```

### FX Recognition — Pack-Driven (CN-5-001 §6 / §11)

Pack `recognition_rules` declare when FX is recognised:
- **At-record FX**: paired FX event emitted at invoice recording; Accounting journals in TZS at recorded rate; subsequent payment-time rate difference = FX gain/loss.
- **At-payment FX**: invoice journaled in supplier currency initially (with FX rate placeholder); FX gain/loss recognised at payment moment when actual conversion rate is known.

Per CN-5-001 §11 `forex.gain_loss_recognition` directs P&L or OCI routing.

### Payment in Foreign Currency

Cash can pay from a multi-currency till (CN-5-002 §15) — direct payment in USD; Obligation reduces in USD.

Cash can pay from a TZS till + FX bridging — paired FX event + USD-equivalent payment to supplier; Obligation reduces in USD; FX gain/loss event records the rate difference.

UI-06 enforced throughout — no silent unit mixing; FX events are explicit bridges.

---

## 14. Tax / Input VAT

Input VAT (the tax paid on purchases that becomes recoverable against output VAT) is computed per pack at invoice recording:

```yaml
procurement.invoice.recorded.v1
  payload:
    invoice_lines:
      - {line_ref, qty, unit_price, tax_treatment_ref}    # tax_treatment_ref → pack
    pre_tax_total
    tax_breakdown:                                         # computed by pack evaluator
      - {treatment_ref, taxable_amount, tax_amount, recoverable: true/false}
    tax_total
    grand_total
```

The compliance evaluator (CN-4-015 §4) reads `tax_treatment_ref` and computes the tax amount + recoverability flag. Recoverable input tax journals to Input Tax (asset); non-recoverable journals to Expense.

CN-5-001 #6 auto-journals consume this breakdown:

```
Dr Inventory / Expense    (pre-tax amount per line)
Dr Input Tax              (recoverable tax)
Dr Expense (non-recov)    (non-recoverable tax, if any)
Cr AP                      (grand total)
```

Pack rotation governs rate changes (D-009 freeze): historical invoices remain under their original pack version; new invoices use current rates.

---

## 15. Cross-Engine Subscription Map

| Procurement event | Subscribed by | Effect |
|-------------------|---------------|--------|
| `procurement.grn.received.v1` | CN-5-003 Inventory (P3) | Receive stock if `inventory_effect: tracked`; create lot if lot-tracked; emit `inventory.stock.received.v1` |
| `procurement.grn.received.v1` | CN-5-001 Accounting (P3 — #8) | If invoice not yet recorded: GRN accrual (Dr Inventory/Expense; Cr GR/IR) |
| `procurement.grn.compensated.v1` | CN-5-003 Inventory (P4) | Reverse receipt for returns (kind: compensation) |
| `procurement.invoice.recorded.v1` | CN-5-001 Accounting (P3 — #6) | Auto-journal: Dr Inventory/Expense + Dr Input Tax; Cr AP (Obligation) |
| `procurement.invoice.matched.v1` | CN-5-006 Reporting | Workflow status update for AP reports |
| `procurement.invoice.paid.v1` | CN-5-006 Reporting | Workflow status update |
| `procurement.credit_note.recorded.v1` | CN-5-001 Accounting (P4 — compensation) | Reverse AP + reverse Inventory/Expense per pack rules |
| `procurement.credit_note.recorded.v1` | CN-5-003 Inventory (P4) | Stock reduction (returned items leave stock) if `inventory_effect: tracked` |

| Procurement subscribes to | Handler | Effect |
|--------------------------|---------|--------|
| `cash.payment.disbursed.v1` | `handle_cash_payment` (N1 consolidated) | Internal routing on `obligation_ref.kind`: `accounts_payable` → update workflow + emit `procurement.invoice.partially_paid.v1` or `.paid.v1`; `credit_refund` (supplier refund) → update return workflow + emit `procurement.return.completed.v1` if final |
| Future: `<vertical>.purchase_need.recorded.v1` | `ingest_purchase_need` | Create requisition from vertical-emitted need (CTR-030 expansion) |

---

## 16. UI-01 + UI-02 + UI-08 Enforcement

| Invariant | Application in CN-5-004 |
|-----------|------------------------|
| **UI-01** (causation chain) | Every emitted event carries `causation_id` to its trigger: requisition.approved.v1 → causation: approval command; po.created.v1 → causation: requisition.approved.v1; grn.received.v1 → causation: grn.record.request derivative; invoice.recorded.v1 → causation: invoice.record.request derivative; invoice.paid.v1 → causation: cash.payment.disbursed.v1. Audit walks the full lineage of any payment back to its originating need. |
| **UI-02** (compensation symmetry) | Credit Note (`procurement.credit_note.recorded.v1`) carries `compensates_event_id` to original `procurement.invoice.recorded.v1`. Downstream Inventory + Accounting + Cash react via `kind: compensation` subscriptions, producing symmetric reverse effects. Partial-credit symmetry follows partial-refund pattern from CN-5-009 §15 Open Item. |
| **UI-08** (Obligation bounds) | AP Obligation enforced at every settlement: `outstanding ∈ [0, original_amount]`. Bus rejects payment commands that would breach. Credit Note path reduces `original_amount` via primitive fold (not by going negative on outstanding). |

DC mechanization: Phase 1 doctrine checks for UI-08 across Cash + Procurement settlements arrive with CN-5-004 ratification per CN-5-102 §7 Phase 1 plan.

---

## 17. Example — Karakana Buys Aluminium

*Karakana ya Mzee Hassan in Mwanza serves as illustrative context per D-004 #4 (peer-technical audience). The example walks the full lifecycle including partial GRN, supplier credit terms with early-payment discount, and a defective-return variant.*

**Supplier:** Aluminium Tanzania Ltd (`supplier-alutz`), Net 30 with 2/10 early-payment discount, domestic, TZS.

**Purchase need:** 50 aluminium sheets × TZS 85,000 = TZS 4,250,000.

### Phase 1 — Requisition

```
Manager submits procurement.requisition.create.request:
  site_id: karakana-mwanza-001
  supplier_ref: supplier-alutz
  items: [{item: aluminium-sheet-2400x1500, qty: 50, unit_price_estimate: 85000}]
  total_estimate: 4,250,000 TZS
  reason: "stock replenishment for window manufacturing"
  → procurement.requisition.created.v1 emitted (workflow: requisition_drafted)

Bus consults pack approval thresholds:
  TZS 4.25M > TZS 1M (manager band) → required_approvers: [manager]
  Approval primitive request emitted; manager approves
  → procurement.requisition.approved.v1 emitted (workflow: requisition_approved)
```

### Phase 2 — PO

```
Procurement submits (or system-auto-submits from approved requisition):
  procurement.po.create.request → procurement.po.created.v1 + PO Document
  PO Number: PO-KRK-2026-0087
  PO Document Hash: <CN-4-012>
  pack_version_ref: tz-compliance-2026.05
  Workflow: po_issued
```

### Phase 3 — GRN (Partial)

```
Initial delivery — 30 sheets:
  procurement.grn.record.request {po_ref, items: [{aluminium-sheet, qty: 30, unit_cost: 85000, inventory_effect: tracked}]}
  → procurement.grn.received.v1 (GRN-A, qty: 30, source_ref: procurement.grn, site_id: karakana-mwanza-001)
  → GRN Document issued (GRN-KRK-2026-0142)
  → Workflow: po_partially_received

Downstream:
  CN-5-003 Inventory subscribes → inventory.stock.received.v1 (30 sheets); creates lot
  CN-5-001 Accounting (#8 GRN accrual): Dr Inventory 2,550,000; Cr GR/IR 2,550,000

A week later — remaining 20 sheets:
  procurement.grn.record.request {po_ref, items: [{aluminium-sheet, qty: 20, ...}]}
  → procurement.grn.received.v1 (GRN-B, qty: 20)
  → GRN Document GRN-KRK-2026-0143
  → Workflow: po_fully_received

  CN-5-003 → +20 sheets, second lot
  CN-5-001 #8 GRN accrual: Dr Inventory 1,700,000; Cr GR/IR 1,700,000
```

### Phase 4 — Invoice Recording + Three-Way Match

```
Supplier invoice arrives:
  Total: 50 sheets × 85,000 + 18% VAT = TZS 5,015,000

procurement.invoice.record.request:
  po_ref:               PO-KRK-2026-0087
  supplier_invoice_ref: ALUTZ-INV-9871
  invoice_lines:        [{item: aluminium-sheet, qty: 50, unit_price: 85000, tax_treatment_ref: tz-vat-standard}]
  pre_tax_total:        4,250,000
  tax_total:            765,000
  grand_total:          5,015,000
  currency:             TZS

Bus runs three-way match (§7):
  PO.qty: 50; GRN.qty (A+B): 50; Invoice.qty: 50 → quantity match ✓ (auto-pass)
  PO.price: 85000; Invoice.price: 85000 → price match ✓ (auto-pass, 0% variance)
  Items match exactly ✓

→ procurement.invoice.matched.v1 + procurement.invoice.recorded.v1 emitted
→ Invoice Recording Document INV-REC-KRK-2026-0058
→ AP Obligation created:
    obligation_id: ap-alutz-2026-0058
    kind: accounts_payable
    original_amount: 5,015,000 TZS
    outstanding: 5,015,000 TZS
    due_date: invoice_business_date + 30 days (Net 30 per credit terms)

CN-5-001 #6 auto-journal:
  Dr Inventory       4,250,000
  Dr Input Tax         765,000
  Cr AP-Alutz        5,015,000
  (Note: this offsets the GR/IR accruals from Phase 3 GRN events;
   net effect on Inventory account = full 4,250,000; GR/IR returns to zero)
```

### Phase 5 — Payment (Within Discount Period)

```
Karakana pays within 10 days to capture 2% early-payment discount.

Eligible discount: 2% × 5,015,000 = 100,300
Payment amount: 5,015,000 - 100,300 = 4,914,700

Mzee Hassan submits cash.payment.record.request (CN-5-002 §G):
  obligation_ref: ap-alutz-2026-0058
  till_id:        drawer-mwanza-A
  amount:         5,015,000 TZS   (the obligation reduces by full amount; discount recorded separately)
  currency:       TZS

Actually, the pack rule for early-payment discount works like this:
  - Cash physically pays 4,914,700
  - Obligation reduces by 5,015,000 (full)
  - The 100,300 difference is recorded as a credit to the AP via cash.adjustment.recorded.v1
    with reason: early_payment_discount; OR a paired Cash discount entry

Per pack rule (this varies; one common pattern):
  Cash emits:    cash.payment.disbursed.v1 (amount 4,914,700)
  Cash emits:    cash.adjustment.recorded.v1 (reason: early_payment_discount, amount 100,300)
  Obligation:    settled in full (outstanding = 0)

Procurement subscribes cash.payment.disbursed.v1:
  handle_cash_payment routes on obligation.kind = accounts_payable:
    → AP Obligation outstanding now = 0
    → emit procurement.invoice.paid.v1
    → Workflow: completed

CN-5-001 #7 auto-journal (AP settlement):
  Dr AP-Alutz                5,015,000
  Cr Cash                    4,914,700
  Cr Purchase Discount Earned  100,300
```

### Causation Chain Complete

Audit walks: `procurement.invoice.paid.v1 → cash.payment.disbursed.v1 → cash.payment.record.request → procurement.invoice.matched.v1 → procurement.invoice.recorded.v1 → procurement.grn.received.v1 (×2) → procurement.po.created.v1 → procurement.requisition.approved.v1 → procurement.requisition.created.v1`. Every step UI-01-linked. UI-08 bounds preserved throughout.

### Variant — Defective Return Mid-Lifecycle

Suppose 5 of the 50 sheets received are warped (caught during GRN-A inspection of the 30):

```
GRN-A records: items_accepted: 25, items_rejected: 5 (warped)
procurement.grn.discrepancy.detected.v1 emitted with discrepancy details

Supplier issues credit note for the 5 rejected sheets (offline).
Karakana records:
  procurement.credit_note.record.request:
    return_ref:               <return-ref>
    supplier_credit_note_ref: ALUTZ-CN-2026-0091
    items_credited:           [{aluminium-sheet, qty: 5, unit_credit: 85000, total_credit: 425000}]
    total_credited:           500,500 TZS (425,000 + 75,500 VAT)
    currency:                 TZS

  → procurement.credit_note.recorded.v1
      compensates_event_id:   <if applied against the recorded invoice — links to that invoice>
      causation_id:           <credit note command derivative>
    Credit Note Document CN-KRK-2026-0034 issued (terminal fold)

Cross-engine compensation:
  CN-5-003 Inventory subscribes (kind: compensation) → inventory.stock.adjusted.v1
    (5 sheets removed from stock at karakana-mwanza-001, source_ref: return_to_supplier)
  CN-5-001 Accounting subscribes (kind: compensation):
    Dr AP-Alutz                500,500     (AP reduces by credit amount)
    Cr Inventory               425,000
    Cr Input Tax                75,500

  AP Obligation outstanding: 5,015,000 - 500,500 = 4,514,500
  (Karakana now owes net 4,514,500; payment proceeds against this reduced amount)
```

UI-02 symmetry holds: every engine that posted on the original invoice posts a symmetric reverse for the credited portion.

---

## 18. Three Whys

### Why does this matter?

Every business that has suppliers has a procurement story to tell: what was needed, who approved it, what was ordered, what was actually received (and when), what was invoiced (and was the invoice correct), what was paid (and when). Without a unified lifecycle engine, each engine reconstructs procurement state from its slice (Inventory knows what arrived; Accounting knows what was journaled; Cash knows what was paid) — incomplete and inconsistent. Disputes with suppliers, three-way audits, regulatory inspections, fraud investigations all require the full chain. CN-5-004 makes the chain a first-class event-sourced reality: every transition emits an event; truth = fold; the audit walks the trail unambiguously.

### Why does it belong here (and not in Foundation)?

Foundation provides the Workflow primitive (lifecycle), the Approval primitive (gates), the Obligation primitive (AP), the Document primitive (PO/GRN/Invoice/Voucher/Credit Note), the Party primitive (supplier records), and the compliance evaluator (tax). Foundation does not say "every purchase passes Requisition → PO → GRN → Invoice → Payment," "three-way match is the integrity check," "AP is created at invoice recording," or "Credit Note is a compensating Document." Those are universal procurement choices appropriate for any business with suppliers. CN-5-004 makes those choices. A specialised engine (high-frequency commodity trading, JIT manufacturing) could compose primitives differently — but that is not what universal procurement is for general business.

### Why this design?

Lifecycle-as-Workflow makes phases auditable and replay-deterministic. AP-as-Obligation (P4) unifies the "owed" concept across engines (matching AR in Cash, delivery in Inventory, advance/layby in Cash). One-lifecycle-two-effects (P7) avoids splitting Procurement into "purchasing engine + expense engine" while keeping the inventory/expense distinction clean via a single flag at GRN. Three-way match (P3) makes payment integrity structural, not aspirational. Credit-Note-as-compensation (P6) uses the same CN-4-002 envelope mechanism that handles every other compensation in BOS — no new machinery. Pack-driven approval thresholds + variance bands operationalise Law 5 — adding a new country's rules is content authorship, not engineering. The boundary with CN-5-002 §7 (formal procurement vs petty cash-and-carry) prevents Procurement from being overloaded with informal transactions while keeping all supplier-relationship purchases under one auditable trail. The whole engine is one cohesive answer to "how does any business, with any supplier, buy anything?"

---

## 19. Boundaries

| Topic | Lives in |
|-------|----------|
| Workflow, Approval, Obligation, Document, Party primitive folds | CN-4-011 |
| Document hash, numbering, freeze, terminal fold | CN-4-012 |
| Compliance DSL grammar + sandboxed evaluator (tax computation, approval thresholds) | CN-4-015 |
| Pack content per jurisdiction (TFRS-TZ thresholds, IFRS audit requirements, etc.) | Compliance packs authored under CTR-029 (Term 1) |
| Event envelope (causation_id, compensates_event_id, pack_version_ref) | CN-4-002 |
| Command bus, atomic emission, policy rejection | CN-4-004 |
| Clock protocol (business_date, due_date computation) | CN-4-014 |
| Stock storage, lots, costing, expiry | CN-5-003 |
| AP journal mappings | CN-5-001 §4 (#6 invoice, #7 settlement, #8 GRN accrual) |
| Payment medium, AP cash settlement, cash refund handling on supplier returns | CN-5-002 §G + §13 |
| Informal cash-and-carry expense (no PO, no supplier invoice) | CN-5-002 §7 |
| Subscription patterns + replay semantics + compensation kind | CN-5-100 |
| Scope policy (multi-scope per §6 pattern) | CN-5-101 |
| Cross-engine invariants (UI-01 causation, UI-02 compensation symmetry, UI-08 obligation bounds) | CN-5-102 |
| Vertical-specific purchase need events | Term 6 via CTR-030 expansion |
| Tenant functional currency + site registries | Term 1 + Term 2 via CTR-027 |
| Supplier onboarding integration (TIN verification, import compliance) | Term 7 future (CTR-033 Open Item) |
| Reporting templates (AP ageing, supplier performance, procurement spend by category) | CN-5-006 + pack content |
| Statutory procurement reporting (large-contract disclosures, public-sector procurement transparency) | Pack content via CTR-029 |

---

## 20. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Pack content for approval thresholds, three-way-match variance bands, supplier credit-term defaults, multi-currency procurement FX rules per jurisdiction | Term 1 via CTR-029 expansion | Schema in §7 + §9 + §13; per-jurisdiction content is Term 1's authorship |
| Vertical purchase-need event contracts (`<vertical>.purchase_need.recorded.v1` per vertical) | Term 6 via CTR-030 expansion | CN-5-004 manifest grows by addition per vertical |
| Supplier-onboarding external verification (TIN check, import compliance) | Term 7 future via CTR-033 | Open Item; file CTR-033 when pack content explicitly requires external verification |
| Pre-RFQ approval gate enablement per jurisdiction (government procurement) | Pack content + Term 1 | N3 — default off; pack opts in for jurisdictions requiring it |
| Partial-credit symmetric compensation mapping per engine (extends CN-5-009 §15 Open Item) | CN-5-001 + CN-5-002 + CN-5-003 + Architect | Per-engine mapping for partial Credit Note effects |
| Reorder-point triggered procurement (subscription to inventory.reorder_point.reached.v1) | Future — CN-5-003 v2 + CN-5-004 v2 | Currently human-initiated; future automation |
| Master agreement / blanket PO pattern (one PO, multiple release calls over period) | Future v2 | v1 supports single POs; blanket POs deferred |
| Petty cash advance for site purchases (advance funds to site manager for ad-hoc procurement) | Pending CN-5-002 + future decision | Crosses boundary §11 — may need its own pattern |
| Phase 1 doctrine check for UI-08 across Cash + Procurement settlements | CN-4-019 + future CTR | Aligns with CN-5-102 §7 Phase 1 ramp-up; CN-5-004 ratification triggers DC addition |
| Performance budget for three-way match at scale (large POs with many lines and multiple GRNs) | Architect phase | Concept guarantees correctness; performance is Architect's |
| Multi-currency FX gain/loss recognition: at-record vs at-payment per pack — implementation specifics | Architect + CN-5-001 + pack | Pack declares rule; engine implements |
| Approval delegation patterns (manager away, deputy approves on their behalf) | Term 1 governance + Architect | Per-tenant role policy; not in scope for engine concept |
| Supplier performance metrics (on-time delivery rate, quality score) | CN-5-006 + pack | Reporting derives from Procurement events; metrics live in Reporting |

---

*— End of CN-5-004 —*
