# CN-5-104 — Period-Close Choreography

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 5 — Universal Engines
> **Status:** For Overseer review.
> **Governing decisions:** D-009 (freeze doctrine — closed-period events under historical pack); Charter §1.2 (legal defensibility — auditable, replayable close); Law 2 (engine isolation — choreography, not orchestration); Law 5 (compliance is configured — period calendar, statutory bundles, approval thresholds pack-driven); UI-01 (causation chain across close events); UI-05 (closed-period inviolability — enforced via bus policy from close emission); UI-07 (tenant trial balance — gate condition).
> **CTRs:**
> - **OPEN (this doc references for resolution):** CTR-042 (Term 5 → Term 1 — period calendar governance: gregorian/fiscal/lunar variants per jurisdiction; close approval gates per jurisdiction).
> - **OPEN expansions at merge:** CTR-029 (Statement templates per period kind; statutory summary report bundles per period; FX revaluation rules per period); CTR-014 (per-advisor pre-close/post-close hooks); CTR-015 (AI Mode dashboard pre-close suggestion surfaces); CTR-016 (platform billing aggregation post-close subscription).
> - **No new CTRs filed by this doc** (CTR-042 already opened per Overseer commit).
> - **CN-5-001 cosmetic amendment** (single-pass with CN-5-104 ratification): adds `accounting.period.close.initiated.v1` + `accounting.period.close.rejected.v1` to manifest emits.
> **Glossary:** See `MASTER-GLOSSARY.md` — Period, Period-Close, Statement, UI-NN, Scope level.
> **Depends on:** CN-4-002 (envelope); CN-4-004 (command bus + atomic multi-event emission + policy rejection); CN-4-007 (principal types — accountant, owner authorisation); CN-4-008 (audit log — close decisions auditable); CN-4-010 (Projection Framework — period-state projection rebuilds); CN-4-011 (Workflow primitive — period lifecycle; Approval primitive — close authorisation); CN-4-012 (Document Engine — Statement issuance per N3 of CN-5-006); CN-4-014 (clock protocol — fiscal zone + business_date); CN-4-015 (Compliance DSL — pack-driven period_calendar + Statement templates); CN-5-001 (Accounting Engine — owns `accounting.period.*` events; UI-05 bus policy mechanism); CN-5-002 (Cash — emits `cash.period.ready.v1`); CN-5-003 (Inventory — emits `inventory.period.ready.v1`); CN-5-004 (Procurement — emits `procurement.period.ready.v1`); CN-5-005 (HR — emits `hr.payroll.period.ready.v1`); CN-5-006 (Reporting — issues Statement Documents per N3); CN-5-010 (AI Advisors — accounting-advisor + bi-advisor pre/post-close hooks); CN-5-100 (subscription patterns — P5 choreography sequence); CN-5-101 (scope policy — tenant-level close); CN-5-102 (UI-01/05/07 enforcement); CN-5-103 (event glossary — naming compliance for `*.period.ready.v1` family + new `accounting.period.close.*` events).
> **Boundaries:** Period-state events + UI-05 bus policy mechanism → CN-5-001 §8; Statement Document templates + issuance mechanics → CN-5-006 §5/§8 + CN-4-012; UI-05/07 invariants themselves → CN-5-102; period calendar primitive (gregorian/fiscal/lunar) → pack content via CTR-042 (Term 1); subscription pattern definition → CN-5-100 §2 P5; site-level operational close (end-of-shift reconciliation) → CN-5-002 §5 (NOT this doc per PC10); cross-tenant aggregation (Term 1 platform billing) → CTR-016 + CN-5-101 §7; advisor pre-close/post-close suggestions → CN-5-010.

---

## 1. Purpose & Boundary

CN-5-104 is the **cross-cutting choreography doc** for tenant-level period-close. Nine engines exist; Accounting owns the period state; CN-5-006 issues formal Statement Documents post-close. CN-5-104 wires these into a deterministic, replayable, auditable choreography.

The mechanism: four core engines self-validate and signal `<engine>.period.ready.v1`; an authorised human submits `accounting.period.close.request`; the bus verifies all signals received + UI-07 trial balance balanced; on success, the bus atomically emits `accounting.period.closed.v1` (activating UI-05 forever for that period) and triggers Statement Document issuance per CN-5-006 N3. From close emission onward, the closed period is inviolable — corrections post forward with explicit causation back (per CN-5-103 E5/N1 `posting_period_ref`).

### Scope (in)

- Tenant-level period-close choreography (fan-in P5 per CN-5-100)
- Engine readiness criteria for 4 core signalling engines (Cash, Inventory, Procurement, HR)
- Freeze window doctrine (PC7 — pre-close UI-05 reinforcement)
- UI-07 trial-balance gate at Phase 3
- Statement Document issuance per CN-5-006 N3 + pack templates
- Forward-period correction-of-closed-period semantics (PC8 per CN-5-103 E5/N1)
- Multi-currency close (single close, FX at close moment)
- Year-end close (same mechanism, statutory bundle scope)
- Period independence (PC11 — no cascade close)
- Advisor pre/post-close hooks (operational support per CN-5-010)

### Scope (out)

- The `accounting.period.*` events themselves (CN-5-001 owns; CN-5-104 references)
- Statement Document templates + issuance mechanics (CN-5-006 + CN-4-012)
- UI-05 / UI-07 invariants themselves (CN-5-102)
- Period calendar primitive (gregorian/fiscal/lunar — pack content via CTR-042)
- Site-level operational close (CN-5-002 §5 session lifecycle — per PC10)
- Cross-tenant billing aggregation (CTR-016 + Term 1)
- Reopen mechanism — **does not exist** (forward-correction only per PC8)
- Performance optimisations (snapshot-at-close, parallel-fold) — Architect phase

---

## 2. Doctrine — PC1–PC11

| # | Law | Source / why |
|---|-----|--------------|
| **PC1** | **Period-close is choreography (P5 per CN-5-100), not orchestration.** No engine commands another. Engines self-emit `<engine>.period.ready.v1` independently. Accounting is the **coordinator for Documents** (Statement issuance via CN-5-006 + CN-4-012) but **not the orchestrator** of other engines. | CN-5-100 L3; engine isolation (Law 2); resilience without central failure point. |
| **PC2** | **Period is a tenant property; calendar is pack-driven.** Pack `period_calendar` declares gregorian/fiscal/lunar variant + frequency + cutoff days. Per jurisdiction; tenant elects within pack permission. | Law 5; D-004 neutrality; CTR-042 governs content per jurisdiction. |
| **PC3** | **Close trigger is human-initiated.** Authorised principal (accountant/owner per Approval primitive + pack threshold) submits `accounting.period.close.request`. No auto-scheduled close — books must be operationally complete first. | CN-4-007 identity; CN-4-011 Approval primitive; operational reality. |
| **PC4** | **Engines with financial state declare readiness criteria; engines without don't signal.** Four core signalling engines (Cash, Inventory, Procurement, HR). Promotion does NOT signal (events flow individually; cost-share obligations are part of trial balance). Reporting does NOT signal (read-side per R1). Checkout does NOT signal (settlement event-driven). | §3 catalogue; minimises fan-in to engines that have aggregation requiring finalisation. |
| **PC5** | **Accounting coordinates close.** Accounting subscribes to required signals + verifies UI-07 trial balance + emits `accounting.period.closed.v1`. UI-05 bus policy activates from emission. | CN-5-001 §8; CN-5-102 UI-05/07. |
| **PC6** | **Statement Documents emit POST-close.** Per CN-5-006 N3 doctrine — formal hash-frozen Documents via CN-4-012 (P&L, Balance Sheet, Cash Flow, Statutory Summaries). Issuance is post-close, atomic with close per CN-4-004 §2. | CN-5-006 N3; D-009 freeze; CN-4-012 terminal fold. |
| **PC7** | **Freeze window doctrine.** Once `accounting.period.close.request` is accepted, the bus emits `accounting.period.close.initiated.v1` — from this moment, the bus rejects new commands attempting to emit events with `effective_date ∈ period` (pre-emptive UI-05). Window typically minutes; pack-defined maximum (default 24 hours) before close.request expires and must be re-submitted. | Prevents race conditions during close evaluation; ensures Phase 3 sees a stable trial balance. |
| **PC8** | **Forward-period correction-of-closed-period.** Per CN-5-103 E5/N1: events post-close affecting closed-period truth are emitted in the current open period (`posting_period_ref: current_period`) with `compensates_event_id` walking back to the closed-period event. The closed period's truth, as it stood at close, does NOT change. Corrections are auditable forward-period events. | CN-5-103 E5/N1; D-009 freeze; UI-05; replay determinism. |
| **PC9** | **Period-close is replayable.** All events forming the close are in the event store: signal events, `close.initiated`, journal events (for the trial-balance fold), `close.closed` (or `close.rejected`), Statement Document issuance events. Replay reproduces close exactly under the recorded `pack_version_ref` (D-009 freeze). | CN-4-001 §6; replay determinism; legal defensibility. |
| **PC10** | **Tenant-level close only in v1.** Site-level operational close (end-of-shift reconciliation per CN-5-002 §5) is operational, not financial; lives in Cash engine, not here. | CN-5-101 multi-scope; clear separation of operational vs financial close. |
| **PC11** | **Periods are independent — no cascade close.** Closing Q4 2026 does NOT auto-close October/November/December 2026; each monthly period closes separately. Quarterly/annual **reports** are aggregations over closed monthly periods, not separate close decisions on parent periods. Period close is a per-period decision; aggregation is read-side. | Avoids hierarchical-close complexity; statutory reporting still works via aggregation over closed monthlies; matches accounting practice. |

---

## 3. Engine Readiness Catalogue — 4 Core Signalling Engines

Per PC4: four engines have financial-state aggregation requiring finalisation before close. Pack declares per-tenant which signals are required.

### Pack-Driven Inclusion (per Q1 — CTR-042 expansion)

```yaml
# Pack content (CTR-042 expansion)
period_close:
  required_signals:
    default:
      - cash.period.ready.v1
      - inventory.period.ready.v1
      - procurement.period.ready.v1
      - hr.payroll.period.ready.v1
  tenant_overrides_allowed:
    - condition:    "tenant.has_employees == false"
      omit:         hr.payroll.period.ready.v1
    - condition:    "tenant.inventory_item_count == 0"
      omit:         inventory.period.ready.v1
    - condition:    "tenant.has_procurement == false"
      omit:         procurement.period.ready.v1
  promotion_signal:           not_required                     # PC4 — events flow individually
  reporting_signal:           not_required                     # R1 read-side
  checkout_signal:            not_required                     # settlement event-driven
```

### Signalling Engines (Summary)

| Engine | Signal event | What "ready" means |
|--------|--------------|--------------------|
| Cash (CN-5-002) | `cash.period.ready.v1` | Sessions reconciled or carried-forward; variances resolved; MM statements matched; in-transit resolved; Obligations in period recorded |
| Inventory (CN-5-003) | `inventory.period.ready.v1` | All movements processed; lot expiry runs done; cycle counts complete; UI-03 source-ref holds for all events |
| Procurement (CN-5-004) | `procurement.period.ready.v1` | GRN-invoice matches complete or flagged pending; AP Obligations created; supplier terms recorded |
| HR (CN-5-005) | `hr.payroll.period.ready.v1` | Payrolls with `payment_date ∈ period` are `hr.payroll.paid.v1`; statutory deductions computed; leave accruals recorded |

### Non-Signalling Engines (Why)

| Engine | Why no signal |
|--------|---------------|
| Promotion (CN-5-007) | Events flow individually (`promotion.rule.applied.v1`, `promotion.cost_share.recorded.v1`, voucher/loyalty events). Cost-share Obligations are part of trial balance. No internal aggregation needs finalisation. |
| Reporting (CN-5-006) | Read-side per R1; reads what close produces, never emits to close fan-in. |
| Checkout (CN-5-009) | Settlement is event-driven (per sale). No period-level state to finalise. Downstream engines (Cash, Inventory, Accounting) hold the period reflections. |
| Accounting (CN-5-001) | Accounting is the **coordinator** — receives signals + emits close decision. Doesn't signal to itself. |

---

## 4. Per-Engine Readiness Criteria (Definitive)

### 4.1 — Cash (`cash.period.ready.v1`)

Criteria (all must hold):

| # | Criterion | Verification |
|---|-----------|--------------|
| a | Every `cash.session.opened.v1` in period has matching `cash.session.closed.v1` OR is explicitly carried-forward to next period (per pack `cash_reconciliation.session_carryforward_allowed`) | Cash projection: open_sessions_count_in_period − closed_sessions_count_in_period = carried_forward_count |
| b | Variances per session resolved: auto-adjusted within threshold (`cash.variance.adjusted.v1`) OR escalated decision recorded | No pending `cash.variance.detected.v1` without paired resolution |
| c | Mobile money provider statements for period received + matched against `cash.tender.received.v1` events | Per `<mm-adapter>.statement.received.v1` events + reconciliation projection |
| d | In-transit deposits resolved | No open `cash.deposit.requested.v1` without paired outcome (`completed` or `failed`) |
| e | AR collections + AP payments in period recorded; Obligation primitives reflect period activity | UI-08 bounds hold across all Obligations affected by Cash in period |

Emission: when all criteria met, Cash emits `cash.period.ready.v1 {period_ref, evaluated_at, criteria_summary}`. Idempotent — re-evaluation produces same signal if state unchanged.

### 4.2 — Inventory (`inventory.period.ready.v1`)

| # | Criterion | Verification |
|---|-----------|--------------|
| a | All `inventory.movement.*.v1` events in period are emitted (no pending command queue) | Pending command queue empty for inventory-scoped commands |
| b | Lot expiry scheduler runs scheduled in period completed | All `inventory.lot.expiry_run.request` events for period have produced their `inventory.lot.expired.v1` emissions |
| c | Cycle counts (per N5 of CN-5-003) scheduled in period completed | All `inventory.stock.count.request` events with `business_date ∈ period` have produced their adjustment events |
| d | `inventory.adjustment.recorded.v1` events for write-offs/counts/adjustments settled (no pending) | Adjustment queue empty |
| e | UI-03 source-ref invariant holds across all period events (no naked stock changes) | Validation pass on all inventory events in period |

### 4.3 — Procurement (`procurement.period.ready.v1`)

| # | Criterion | Verification |
|---|-----------|--------------|
| a | All `procurement.grn.received.v1` in period have either matching `procurement.invoice.recorded.v1` OR `pending_invoice_at_close: true` flag (rolls forward via GR/IR accrual per CN-5-001 #8 — does NOT block close) | GRN-invoice match projection |
| b | `procurement.invoice.recorded.v1` events have completed three-way match OR are in dispute state (disputes don't block; they're disclosed in close summary) | Invoice match-state projection |
| c | AP Obligations created in period recorded | Obligation primitive bounds (UI-08) hold |
| d | Supplier credit-term schedules recorded for invoices that fall due in/after period | Supplier credit-terms projection |

### 4.4 — HR / Payroll (`hr.payroll.period.ready.v1`)

| # | Criterion | Verification |
|---|-----------|--------------|
| a | All payrolls whose `payment_date` falls in accounting period are `hr.payroll.paid.v1` (handles week/month frequency mismatch — pack period_frequency may differ from accounting period_frequency) | Payroll projection: payrolls_with_payment_date_in_period_count == payrolls_paid_in_period_count |
| b | Statutory deductions computed per pack rules (PAYE, pension, NHIF, levies) | `hr.payroll.computed.v1` payload includes all required deduction breakdowns |
| c | Employee loan deductions applied (per N4 of CN-5-005 — atomic with paid event) | `hr.payroll.deduction.applied.v1` events present for all active loans |
| d | Leave accruals per `hr.accrual.run.request` completed for period | `hr.leave.accrued.v1` events present per pack frequency |
| e | Statutory filings due in period prepared as Documents (for year-end close — annual PAYE cert, etc.) | Per CTR-029 expansion year-end bundle |

---

## 5. Choreography Flow — Six Phases

```
═════════════════════════════════════════════════════════════════════════════════
PHASE 1 — ENGINES COMPLETE BOOKS (continuous; no orchestration)
═════════════════════════════════════════════════════════════════════════════════

Engines independently self-validate as events arrive. When all criteria met
per §4, engine emits its <engine>.period.ready.v1 signal. Order doesn't
matter; idempotent re-evaluation OK.

  Cash:           cash.period.ready.v1            (when §4.1 criteria met)
  Inventory:      inventory.period.ready.v1       (when §4.2 criteria met)
  Procurement:    procurement.period.ready.v1     (when §4.3 criteria met)
  HR:             hr.payroll.period.ready.v1      (when §4.4 criteria met)

Accountant monitors via Reporting projection of period_close_status.
Each ready signal is causation-chained to the trigger event that brought
its engine into ready state (UI-01).

═════════════════════════════════════════════════════════════════════════════════
PHASE 2 — CLOSE REQUEST + FREEZE WINDOW (per PC7 / Q6)
═════════════════════════════════════════════════════════════════════════════════

Authorised human (accountant/owner per Approval primitive + pack threshold):

  accounting.period.close.request {period_ref, requested_by}

Bus validates:
  - Principal authorised per Approval primitive
  - Period currently 'open' state (per Accounting period-state projection)
  - All required signals (per pack.period_close.required_signals for this
    tenant) have been received

IF missing signals:
  rejected_by_policy:     PC4.required_signals_unmet
  rejection_reason:       "Missing: [<engine>.period.ready.v1, ...]"
  Accountant addresses missing engines (operationally); engines emit
  signals; accountant re-submits close.request.

IF all signals received:
  Bus emits:              accounting.period.close.initiated.v1
    payload:
      period_ref
      initiated_at
      initiated_by
      signals_received_refs: [<list of period.ready event_ids>]
      pack_version_ref
  FREEZE WINDOW ACTIVE: bus rejects any subsequent command attempting to
  emit an event with effective_date ∈ period (pre-emptive UI-05).
  Window duration: pack-defined maximum (default 24h) before close.request
  expires and must be re-submitted.

═════════════════════════════════════════════════════════════════════════════════
PHASE 3 — UI-07 TRIAL BALANCE VERIFICATION
═════════════════════════════════════════════════════════════════════════════════

Accounting reads its trial-balance projection at the global_position of
the close.initiated event:

  trial_balance = sum(debits) − sum(credits) for all
                  accounting.journal.posted.v1 events in this tenant ≤
                  close.initiated.global_position
                  (minus all accounting.journal.reversed.v1 compensations)

Pack defines epsilon (rounding tolerance):
  pack.accounting.trial_balance_epsilon: TZS 1 (default)

IF |trial_balance| ≤ epsilon → proceed Phase 4

IF |trial_balance| > epsilon → REJECT:
  Bus emits:              accounting.period.close.rejected.v1
    payload:
      period_ref
      rejected_at
      reason:              trial_balance_unbalanced
      imbalance_amount     <decimal>
      suspect_accounts:    [<account refs ranked by largest delta>]
      pack_version_ref
  Freeze window LIFTS (close attempt abandoned).
  Accountant investigates via Reporting + accounting-advisor (CN-5-010
  §4.5 — post-close-rejection suggestion hook); posts adjustment via
  accounting.adjustment.post.request; re-submits close.request.

═════════════════════════════════════════════════════════════════════════════════
PHASE 4 — CLOSE EMISSION + UI-05 ACTIVATION (atomic per CN-4-004 §2)
═════════════════════════════════════════════════════════════════════════════════

Bus atomically emits:

  accounting.period.closed.v1
    payload:
      period_ref
      closed_at
      closed_by             <principal>
      summary:
        trial_balance:       {total_debits, total_credits, balanced: true, epsilon_used}
        accounts_count
        journal_events_count
        period_revenue
        period_expense
        period_profit_or_loss
        fx_gain_loss_total   (if multi-currency tenant — per §7)
      signals_received_refs: [<event_ids of all <engine>.period.ready.v1>]
      pack_version_ref

UI-05 bus policy in-effect: any subsequent command attempting to emit an
event with effective_date ∈ period is rejected per UI-05 with PC8 guidance
("post forward via posting_period_ref + compensates_event_id").

Freeze window lifts (UI-05 supersedes — permanent lock).

═════════════════════════════════════════════════════════════════════════════════
PHASE 5 — STATEMENT DOCUMENT ISSUANCE (per CN-5-006 N3 + PC6)
═════════════════════════════════════════════════════════════════════════════════

Reporting subscribes to accounting.period.closed.v1. Per CN-5-006 §5/§8
+ N3, formal Statement Documents are issued via CN-4-012:

  reporting.statement.issued.v1   × N        (one event per Statement)
    Documents issued:
      - P&L Statement                    (per pack reporting_templates.profit_and_loss)
      - Balance Sheet Statement          (per pack reporting_templates.balance_sheet)
      - Cash Flow Statement              (per pack reporting_templates.cash_flow_statement)
    Year-end (PC9 + Q9 + §7):
      - Annual PAYE Summary Statement   (per CTR-029 statutory bundle)
      - Annual VAT Summary Statement
      - Annual Pension Summary Statement
      - Per jurisdiction additional bundles

Each Document: hash-frozen, numbered per pack format, template_version_ref +
pack_version_ref recorded. Verifiable years later (D-009 freeze).

═════════════════════════════════════════════════════════════════════════════════
PHASE 6 — POST-CLOSE OBSERVATION (operational, advisor-supported)
═════════════════════════════════════════════════════════════════════════════════

  - Reporting projections finalize period-end snapshots (CN-4-018 non-truth
    cache; complementary to Statement Documents which are formal truth)
  - Advisors emit post-close suggestions (CN-5-010 §4.5/§4.1):
      accounting-advisor: account ageing flags, unusual entry suggestions
      bi-advisor:         period-over-period, year-over-year analysis if year-end
  - Term 1 platform aggregator subscribes accounting.period.closed.v1 +
    reporting.metrics.published.v1 for billing roll-up (per CTR-016 +
    CN-5-101 §7 Chaguo C)
  - Forward-period corrections (PC8) become available — any errors
    discovered later post forward.
```

---

## 6. UI-07 Verification (Trial Balance Gate)

### Computation

The trial balance is computed as a deterministic fold over Accounting journal events:

```
For tenant T at close.initiated global_position G:

  debits  = sum(j.debit_amount for j in accounting.journal.posted.v1
                                  where j.tenant = T AND j.global_position ≤ G)
  credits = sum(j.credit_amount for j in accounting.journal.posted.v1
                                  where j.tenant = T AND j.global_position ≤ G)

Compensations included:
  Each accounting.journal.reversed.v1 contributes symmetric Dr↔Cr entries
  per its reversal_entries payload.

trial_balance = |debits − credits|
```

Pack defines epsilon for rounding:
```yaml
pack.accounting.trial_balance_epsilon: TZS 1     # default; pack may override per jurisdiction
```

### Pass/Fail Outcomes

| Outcome | Action |
|---------|--------|
| `trial_balance ≤ epsilon` | Phase 4 proceeds (close emitted) |
| `trial_balance > epsilon` | Phase 3 rejection → `accounting.period.close.rejected.v1` emitted with `imbalance_amount` + `suspect_accounts` ranked by largest absolute delta |

### Suspect Accounts Ranking

The rejection event includes a ranked list of accounts contributing most to the imbalance, computed by per-account `|sum(debits) − sum(credits)|` descending. This guides the accountant's investigation; advisors (`accounting-advisor` per CN-5-010 §4.5) may emit follow-up suggestions referencing these accounts.

### Never Force-Close

There is no `accounting.period.force_close.request`. UI-07 imbalance is a hard gate; the accountant must investigate and correct via standard `accounting.adjustment.post.request` (current period — UI-05 honoured if previous periods involved) before re-submitting close. (Q10 ruling.)

---

## 7. Statement Document Issuance (per CN-5-006 N3 + PC6)

### Mechanism

Per CN-5-006 N3 doctrine: Statements are formal Documents via CN-4-012 — hashed, numbered, template-versioned, pack-frozen. Issued **post-close** as atomic continuation of Phase 4 / Phase 5.

### Pack-Driven Templates

Pack `reporting_templates` declares which Statements issue per period kind:

```yaml
# Pack content (CTR-029 expansion)
period_close_statements:
  monthly:
    - profit_and_loss_statement
    - balance_sheet_statement
    - cash_flow_statement
  quarterly:
    - (inherits monthly + quarterly_summary)
  annual:
    - (inherits monthly + quarterly + annual_paye_certificate +
       annual_vat_return + annual_pension_summary + annual_nhif_summary +
       per jurisdiction additional statutory bundles)
```

### Reissuance Path (Q8)

If a Statement Document is later found to contain an error (rare; usually caught at close), correction follows CN-4-012 §6 + CN-5-006 §5 amendment mechanism:

- Original Statement Document remains hash-frozen forever (CN-4-011 Document terminal fold)
- `reporting.statement.amend.request` triggers issuance of a NEW Document with `corrects_document_ref` to original
- Both Documents are independently verifiable; new explains what changed and why

The closed period itself does NOT reopen — only the Statement artefact is re-issued.

### Per CN-5-006 N3 (Statements ≠ Snapshots)

Statements are **truth artefacts** (verifiable years later by hash). Snapshots (CN-4-018) are **non-truth performance cache**. CN-5-104 issues Statements at close. Snapshots are an independent mechanism — they may be created in Phase 6 for fast post-close queries, but they are not the close output.

---

## 8. UI-05 Reinforcement + Freeze Window

Two periods of UI-05-style enforcement:

### Freeze Window (Phase 2 → Phase 3/4 — Temporary)

From `accounting.period.close.initiated.v1` emission until either:
- `accounting.period.closed.v1` emitted (success path), OR
- `accounting.period.close.rejected.v1` emitted (failure path), OR
- Pack-defined window expiry (default 24h) — close.request expires

During the freeze window, the bus rejects new commands attempting to emit events with `effective_date ∈ period`. Reason: a stable trial balance must be evaluated; new events landing mid-evaluation create races.

### Permanent UI-05 (Phase 4 onward)

From `accounting.period.closed.v1` emission onward, UI-05 (CN-5-102) is in-effect **forever** for that period. Any subsequent command with `effective_date ∈ period` is rejected:

```
rejected_by_policy:    UI-05.closed_period_inviolable
rejection_reason:      "Period <period_ref> is closed for this tenant; corrections must post to current open period with posting_period_ref + compensates_event_id (per CN-5-103 E5/N1)"
```

The freeze window is **superseded** by permanent UI-05 once close emits; no longer relevant.

---

## 9. Forward-Period Correction-of-Closed-Period (PC8)

When an error in closed-period truth is discovered, the correction is **forward-period** — emitted in the current open period, with explicit references back.

### Mechanism (per CN-5-103 E5/N1)

```yaml
accounting.journal.reversed.v1
  payload:
    reversed_journal_ref:  <event_id of the erroneous closed-period journal>
    reversal_entries:      [<symmetric reverse Dr↔Cr>]
    reason_ref:            <code + free text>
    business_date:         <current open period>          # UI-05 allows because effective_date ∈ open period
    posting_period_ref:    <current open period>          # N1 — explicit forward-period anchor
    compensates_event_id:  <event_id of the erroneous closed-period journal>     # UI-02 link
    causation_id:          <triggering correction command derivative>
```

### Closed-Period Truth Stays Unchanged

The closed period's trial balance, P&L, balance sheet — as they stood at close — **do not change**. The closed Statement Documents remain hash-verifiable forever. The correction is a forward-period event with a backward-references — both events coexist in the store:

- Original closed-period journal: truth as known at close
- Forward correction: truth as discovered later, posted in current period

Reporting shows both: "Period October 2026 closed at TZS 5,300,000 profit (Statement STMT-PL-2026-10); a November correction posted TZS 22,000 adjustment referencing October's duplicate sale (Statement STMT-AMEND-2026-11)."

### UI-02 Cross-Engine Propagation

The compensating journal triggers downstream subscribers via `kind: compensation`:
- Cash: compensates any associated cash flow
- Inventory: restores any associated stock (if applicable)
- Reporting: updates current-period projections
- Promotion: reverses any promotion-related side effects

All compensations are themselves forward-period events with their own `posting_period_ref`.

---

## 10. Pre-Close Validations (Q5 — Defence-in-Depth)

Two validation layers:

### Layer 1 — Engine Self-Validation

Each engine validates its own state per §4 criteria **before** emitting its `<engine>.period.ready.v1`. Failure = engine simply doesn't emit; the engine's books are not yet complete for the period.

The accountant sees in Phase 2 which engines have not yet signalled (via Reporting projection of `period_close_status`); chases the engines (operationally — talk to cashier, stock keeper, HR officer); engines complete their work; signals emerge; accountant resubmits.

### Layer 2 — Accounting Cross-Validation (Phase 3)

Accounting performs cross-engine integrity checks before emitting `closed`:

- **UI-07** — trial balance balanced (mandatory; Phase 3 hard gate)
- **UI-08** — all Obligations in period within bounds (cost-share receivables, AR, AP, employee loans, advances)
- **UI-06** — currency consistency (all financial events in tenant functional currency unless paired with FX)
- **UI-10** — cost-share reconciliation balanced for all promotion cost-share events in period (if Promotion active)

UI-07 is the primary gate; others are sanity checks. Failure on any → rejection event with reason.

### Layer 3 — Advisor Pre-Close Suggestions (Optional, Operational)

Per CN-5-010 §4.5 + §4.1: tenant-configurable pre-close advisor runs that suggest review items **before** the accountant submits close.request:

- `accounting-advisor`: account ageing review, unusual journal patterns, period-comparison anomalies
- `bi-advisor`: period revenue/expense trend vs prior periods

This is **observational**, not gate. The accountant decides whether to act on suggestions before submitting close. Suggestions land in Decision Journal per CN-4-013 (audit-ready).

---

## 11. Multi-Currency Period-Close (Q3)

For multi-currency tenants (e.g., Zanzibar hotel with TZS functional + USD multi-currency drawer per CN-5-002 §15):

### Single Close per Period

There is **one** close per period in the tenant's functional currency. Multi-currency balances are translated **at close moment** per pack `forex.revaluation_frequency`.

### FX Translation at Close

Phase 1 readiness (Cash):
- Cash readiness criterion §4.1 (e) extends: "FX revaluation of non-functional-currency tills completed per pack `forex.revaluation_frequency`"

If revaluation frequency = monthly and period = monthly: revaluation runs **before** Cash emits `cash.period.ready.v1`. Revaluation events (`<fx-adapter>.fx.rate.recorded.v1` + Accounting auto-journals via CN-5-001 N1 namespace fix) are part of the period's journal record. UI-07 trial-balance check at Phase 3 includes all FX gain/loss journals.

### Phase 4 Summary Includes FX

`accounting.period.closed.v1` summary payload includes `fx_gain_loss_total` aggregated for the period.

### Statement Document Treatment

Statements present figures in tenant functional currency by default. Multi-currency view (USD positions side-by-side with TZS) may be derived via CN-5-006 §11 pack mapping rules — but this is a Reporting concern, not a separate close.

### No Per-Currency Close

A tenant does NOT have separate "TZS close" and "USD close". One close; one set of Statement Documents in functional currency; alternative views derived if pack supports.

---

## 12. Failure Modes

| Failure | Phase | Behaviour | Recovery |
|---------|-------|-----------|----------|
| Engine fails to emit ready signal | 2 | Bus rejects close.request with missing signals listed | Engine completes its books; emits signal; re-request |
| UI-07 trial balance unbalanced | 3 | `accounting.period.close.rejected.v1` emitted with reason + imbalance + suspect accounts; freeze lifts | Accountant investigates via Reporting + advisor; posts adjustment; re-requests |
| UI-08 obligation bound exceeded | 3 | Rejection with `obligation_bound_violation`; specifies offending Obligation | Investigate primitive integrity (Architect-level); correct via compensation; re-request |
| UI-06 currency mismatch (FX event missing for non-functional balance) | 3 | Rejection with `fx_revaluation_incomplete` | Run pack-scheduled FX revaluation; emit FX events; re-request |
| UI-10 cost-share imbalance | 3 | Rejection with `cost_share_reconciliation_failed` | Promotion engine investigation; correction via compensating cost_share event; re-request |
| Approval denied at Phase 2 | 2 | Approval primitive rejects | Authorised approver reviews; resubmits if appropriate |
| Freeze window expires (>24h default) | 2-3 | `accounting.period.close.initiated.v1` expires; window lifts implicitly; close.request invalidated | New `accounting.period.close.request` submitted; cycle restarts |
| In-flight event after Phase 2 freeze | 2-3 | Bus rejects new command per PC7 freeze | Engine command rejected; events with effective_date in period must wait until close completes (then post forward) OR be re-effective-dated to next period |
| Statement Document issuance fails | 5 | `reporting.statement.issued.v1` not emitted for that Statement; close event STILL VALID — Statement is separate artefact | Reporting retries; if persistent, Architect-level investigation; close stays in effect |
| Multi-currency FX revaluation incomplete | 1-2 | Cash doesn't emit ready until complete | Pack-scheduled FX revaluation runs; completes; Cash emits ready |
| Year-end statutory bundle generation fails | 5 | Some statutory Statements absent at close time; close event STILL VALID; Statements can be issued later via `reporting.statement.issue.request` | Reporting retries or Architect investigates; statutory Statements eventually issued (post-close issuance OK for Statements — Phase 5 doesn't have to be atomic with Phase 4 for ALL Statements) |

### Recovery Doctrine

| Principle | Implication |
|-----------|-------------|
| Never force-close | UI-07 failure means investigation, not override |
| Never reopen | PC8 forward-correction only |
| Always log rejection | `accounting.period.close.rejected.v1` is permanent audit record |
| Re-request is normal | Multiple close attempts per period are expected; each is recorded |

---

## 13. Year-End Close (Q9)

Year-end close uses the **same six-phase mechanism** as monthly close. The only difference is the **Statement bundle scope**.

### Same Mechanism

- Phase 1: same engine readiness criteria
- Phase 2: same close.request (authorised principal, freeze window activation)
- Phase 3: same UI-07 verification
- Phase 4: `accounting.period.closed.v1` with `payload.annual: true` flag
- Phase 5: expanded Statement bundle per pack `period_close_statements.annual`
- Phase 6: same advisor post-close suggestions; bi-advisor likely emits year-over-year analysis

### Expanded Statement Bundle

Pack declares year-end statutory bundles per jurisdiction (CTR-029 expansion):

```yaml
# Pack content
period_close_statements:
  annual:
    inherits_from:    [monthly_statements, quarterly_statements]
    additional:
      - {code: annual_paye_certificate,        template_ref: <jurisdiction-specific>}
      - {code: annual_vat_return,              template_ref: <jurisdiction-specific>}
      - {code: annual_pension_summary,         template_ref: <jurisdiction-specific>}
      - {code: annual_nhif_summary,            template_ref: <jurisdiction-specific>}
      - {code: annual_employer_levies_summary, template_ref: <jurisdiction-specific>}
      - {code: annual_financial_statements,    template_ref: <jurisdiction-specific>}      # statutory-format full FS
```

Each statutory Statement is a separate Document per CN-4-012 — hash-frozen, numbered, template-versioned. The tenant's accountant uses these as the data for statutory filings (per Charter §1.3 — BOS does not file; the accountant does).

### Period Independence Reaffirmed (PC11)

Year-end close is a **separate close decision** on the annual period. It does NOT auto-close the contained monthly periods (those should already be closed monthly throughout the year). If a tenant operates only annual closes (no monthly), the year-end close is their primary close — but PC11 still holds: year-end is its own close decision, not a cascade.

---

## 14. Period Independence and Aggregation (PC11)

### No Cascade Close

Closing one period does not auto-close adjacent periods, nor does it auto-close parent periods (quarterly containing months, annual containing quarters).

```
Oct 2026 (closed)        Nov 2026 (open)        Dec 2026 (open)
   ↓                         ↓                       ↓
   ↓                         ↓                       ↓
   └──────────────── Q4 2026 (separate close decision) ─────────────┘
                                  ↓
                                  ↓
                  Annual 2026 (separate close decision)
```

Each close is an independent decision by the accountant (or their designated process per pack approval thresholds). The Phase 2 close.request specifies which `period_ref` is being closed.

### Aggregation as Reporting Concern

Quarterly and annual reports are **aggregations over closed monthly periods** — they read from the Reporting projection of closed-period data, not from a separate close decision.

- Quarterly P&L = sum of Oct/Nov/Dec closed-period figures
- Annual P&L = sum of all 12 closed monthly figures (assuming monthly-close tenant) OR direct annual close (tenant operating only annually)

This avoids hierarchical-close complexity (what if Oct closed but Q4 not yet closed? What if Q4 re-aggregated after a forward-period correction to Oct?). PC11 + forward-correction (PC8) handle this cleanly: closed monthlies stay closed; corrections post forward; aggregations recalculate from current projection state.

### Statutory Reporting Still Works

Statutory filings often require annual figures. Per PC11:
- Tenant closes Jan, Feb, ..., Dec monthly (12 closes per year)
- Tenant closes annual period at year-end (1 additional close)
- Annual P&L Statement aggregates over the 12 closed monthlies + any forward corrections posted within the annual period
- Statutory bundle (PAYE certificate, etc.) issued at annual close

Some packs may permit "annual-only" tenants (no monthly closes); annual close then is the only close decision in the year.

---

## 15. Advisor Integration (Q11 — CN-5-010 Hooks)

Per CN-5-010 §4.5 (accounting-advisor) + §4.1 (bi-advisor): both are wired with scheduled triggers that align with period-close cadence.

### Pre-Close Hook

```
Schedule:        pack.advisor.accounting.schedule (default: pre-period-close)
                  pack.advisor.bi.schedule (default: pre-period-close summary)

Trigger:         scheduled execution N days before pack.period_calendar.cutoff_day

Output:          accounting-advisor suggestions about:
                   - Account ageing flags
                   - Unusual journal patterns vs prior periods
                   - Obligation balance anomalies
                   - Pending events that should post before close

                 bi-advisor suggestions about:
                   - Period revenue trend
                   - Period expense trend
                   - KPI variance vs prior period
                   - Anomalies worth investigating

Audience:        accountant, owner, manager (per CN-5-010 advisor audiences)

Delivery:        UI feed (CTR-039) + optional email digest (CTR-040)
```

The accountant reviews suggestions; decides whether to investigate before submitting close.request. **Not a gate** — close.request can proceed even if advisor suggests review. Operational helper only.

### Phase 3 Rejection Hook

If `accounting.period.close.rejected.v1` is emitted (UI-07 failure), CN-5-010 `accounting-advisor` may emit a post-rejection suggestion analysing the imbalance:

```
Trigger:         event_subscription on accounting.period.close.rejected.v1

Output:          "Trial balance imbalance TZS X. Suspect accounts: [list]. Likely
                  causes given period activity: [pattern analysis]. Suggest
                  investigation of account Y between dates A-B."

Audience:        accountant, owner

Delivery:        UI feed + optional alert (CTR-040 with consent)
```

Helps the accountant find the root cause faster.

### Post-Close Hook

```
Trigger:         event_subscription on accounting.period.closed.v1

Output:          accounting-advisor: account ageing post-close (e.g., "AP outstanding > 60d totalling TZS Y")
                 bi-advisor: period-over-period summary; year-over-year if year-end

Audience:        owner, manager, accountant

Delivery:        UI feed (default) + email digest (opt-in)
```

Suggestions feed into next period's decisions.

---

## 16. Worked Example — Karakana ya Mzee Hassan October 2026 Monthly Close

*Karakana ya Mzee Hassan serves as illustrative context per D-004 #4 (peer-technical audience).*

**Setting:** Karakana operates 2 sites (Mwanza + Mbeya). Pack `tz-compliance-2026.07` (TFRS-TZ; monthly close per `pack.period_calendar.frequency: monthly`; cutoff day 31; FX revaluation monthly). Functional currency TZS. Mzee Hassan is owner; Mzee Hassan (accountant role, dual-actor when required) holds close-authorisation per pack approval threshold.

### Phase 1 — October Throughout + Early November

```
Oct 31 23:59:   Last sale of October settled (checkout.settled.v1)
                Closes October sales window per business_date logic.

Nov 1-3:        Operational completion:
                 - Pending GRNs from late October matched to invoices recorded
                 - AP Obligations confirmed for all October purchases
                 - Cash session reconciliations for Oct 30/31 closed; variances within auto-adjust threshold; auto-adjustments emitted
                 - M-Pesa October provider statement received via <mm-adapter>.statement.received.v1; matched to recorded cash.tender.received.v1 events; reconciliation complete
                 - October payroll (computed Oct 25, paid Oct 30) — all hr.payroll.paid.v1 events emitted; deductions applied per N4 of CN-5-005
                 - Inventory month-end cycle count complete (Karakana counts all aluminium SKUs Oct 31 evening + Nov 1 morning); inventory.stock.adjusted.v1 events recorded for discrepancies
                 - Lot expiry runs Nov 1 (per pack daily schedule) clear
                 - FX revaluation Nov 1 (pack monthly) — no FX positions for this tenant (single-currency TZS); skipped

Nov 3 morning:  Engines self-validate per §4 criteria:
                 - Cash:        all sessions reconciled, MM matched, no in-transit → emit cash.period.ready.v1
                 - Inventory:   movements done, lot expiry done, counts done → emit inventory.period.ready.v1
                 - Procurement: GRN-invoice matches complete; one invoice flagged pending_invoice_at_close (rolls forward) → emit procurement.period.ready.v1
                 - HR:          October payroll paid; deductions applied; leave accrual run done → emit hr.payroll.period.ready.v1

Pre-close advisor suggestions (CN-5-010 hooks per §15):
  accounting-advisor: "Account 2100 AP has TZS 145k overdue > 30 days from supplier `cement-tz`. Consider clearance before close."
  bi-advisor: "October revenue +12% vs September; aluminium-window category driving."
  Mzee Hassan reviews; decides AP can post next period (supplier on extended terms); proceeds with close.
```

### Phase 2 — Close Request (Nov 4 Morning)

```
Mzee Hassan (accountant role + owner — dual-actor satisfied):
  Submits: accounting.period.close.request {period_ref: 2026-10}

Bus validates:
  - Principal: Mzee Hassan, accountant role → Approval primitive gates pass (pack tenant_size: small → owner role suffices)
  - Period state: 2026-10 currently 'open' ✓
  - Required signals per pack: [cash, inventory, procurement, hr.payroll] all received ✓
  - All four signals' event_refs present in Accounting's period-readiness projection

Bus emits:
  accounting.period.close.initiated.v1
    payload:
      period_ref:                     2026-10
      initiated_at:                   2026-11-04T08:30:00 EAT
      initiated_by:                   human:mzee-hassan
      signals_received_refs:          [
        <cash.period.ready.v1 event_id>,
        <inventory.period.ready.v1 event_id>,
        <procurement.period.ready.v1 event_id>,
        <hr.payroll.period.ready.v1 event_id>
      ]
      pack_version_ref:               tz-compliance-2026.07
      freeze_window_expires_at:       2026-11-05T08:30:00 EAT  (default 24h)

FREEZE WINDOW ACTIVE:
  Any subsequent command attempting to emit event with effective_date ∈ 2026-10
  is rejected immediately (PC7 pre-emptive UI-05).
```

### Phase 3 — UI-07 Verification (Same Moment)

```
Accounting reads trial_balance projection at global_position of close.initiated:

  Total debits across all 1,847 accounting.journal.posted.v1 in 2026-10: TZS 142,000,000
  Total credits across same:                                              TZS 142,000,000
  Compensations included:                                                 12 accounting.journal.reversed.v1 events (sym Dr↔Cr)

  trial_balance = |142,000,000 − 142,000,000| = 0
  Pack epsilon: TZS 1
  0 ≤ 1 → BALANCED ✓

UI-08 sanity:  All Obligations (loyalty, AP, AR, employee loans, advance_received) within bounds ✓
UI-06 sanity:  All financial events in TZS; no FX events needed (single-currency) ✓
UI-10 sanity:  No active campaigns in Oct → trivially balanced ✓

Proceed Phase 4.
```

### Phase 4 — Close Emission (Atomic per CN-4-004 §2)

```
Bus atomically emits:

  accounting.period.closed.v1
    payload:
      period_ref:                     2026-10
      closed_at:                      2026-11-04T08:30:01 EAT
      closed_by:                      human:mzee-hassan
      summary:
        trial_balance:
          total_debits:               TZS 142,000,000
          total_credits:              TZS 142,000,000
          balanced:                   true
          epsilon_used:               TZS 1
        accounts_count:               156
        journal_events_count:         1,847
        period_revenue:               TZS 14,500,000
        period_expense:               TZS 9,200,000
        period_profit_or_loss:        TZS +5,300,000
        fx_gain_loss_total:           TZS 0       (single-currency tenant)
      signals_received_refs:          [<as in close.initiated>]
      pack_version_ref:               tz-compliance-2026.07

UI-05 PERMANENT LOCK for 2026-10. Freeze window supersedes by UI-05.
```

### Phase 5 — Statement Document Issuance

```
Reporting subscribes accounting.period.closed.v1. Per pack period_close_statements.monthly:

  Issues 3 Statement Documents via CN-4-012:

  reporting.statement.issued.v1 (P&L):
    document_id:          STMT-PL-MZH-2026-10
    template_version_ref: tz-pl-template-v3
    pack_version_ref:     tz-compliance-2026.07
    document_hash:        <hash>
    content:              {revenue: 14.5M, COGS: 8.2M, gross_profit: 6.3M, opex: 1.0M, profit: 5.3M, sections per template}

  reporting.statement.issued.v1 (Balance Sheet):
    document_id:          STMT-BS-MZH-2026-10
    template_version_ref: tz-bs-template-v2
    ...

  reporting.statement.issued.v1 (Cash Flow):
    document_id:          STMT-CFS-MZH-2026-10
    template_version_ref: tz-cfs-template-v1
    ...

Each Document: hash-frozen, numbered per pack (RCT-MZH-STMT-2026-10-A, etc.),
verifiable years later via hash recomputation + replay if needed.
```

### Phase 6 — Post-Close (Days After)

```
Reporting projection updates for closed-period queries:
  - period_close_2026-10 snapshot recorded per CN-4-018 (non-truth cache — for fast queries)
  - Statement Documents permanent (truth)

CN-5-010 advisor hooks fire (event_subscription on accounting.period.closed.v1):

  accounting-advisor:
    "October closed at TZS 5.3M profit. AP outstanding > 60d totalling TZS 850k from suppliers
     [aluminium-tz, cement-tz]. Suggest payment authorization next 14 days."
    → kernel.advisor.suggestion.recorded.v1

  bi-advisor:
    "October revenue +12% vs September (TZS 14.5M vs 12.9M). Aluminium-window category drove
     +18%; offset by -3% in installation labour. Margins held at 35%."
    → kernel.advisor.suggestion.recorded.v1

Term 1 platform aggregator subscribes accounting.period.closed.v1 (per CTR-016 + CN-5-101 §7):
  - Reads tenant's October summary for billing roll-up
  - Aggregates BOS-share cost-share receivables (if any active)
  - Applies November subscription invoice credit
```

### Forward-Correction Scenario (Nov 12)

```
Nov 12: Mzee Hassan discovers October sale recorded twice for customer Mama Asha
        (cashier Asha accidentally double-tapped Oct 28; TZS 22,000 double-counted).

Cannot post correction to October (UI-05 PERMANENT for 2026-10).

Submits: accounting.adjustment.post.request
  payload:
    business_date:        2026-11-12  (current open period — November)
    posting_period_ref:   2026-11
    compensates_event_id: <event_id of original Oct 28 sale journal>
    entries:              [Cr Revenue 18,644, Cr Tax Payable 3,356, Dr Cash 22,000]   (reverse symmetric)
    reason_ref:           "duplicate sale correction — Oct 28 customer Mama Asha"
    reason_code:          ERR-DUP-001

Bus validates:
  - November 2026 is currently 'open' ✓
  - Principal: Mzee Hassan (accountant role) ✓
  - Entries balance ✓

Bus emits: accounting.journal.reversed.v1
  payload:
    reversed_journal_ref:  <Oct 28 event_id>
    reversal_entries:      [as above]
    reason_ref:            "duplicate sale correction"
    business_date:         2026-11-12
    posting_period_ref:    2026-11
    compensates_event_id:  <Oct 28 event_id>
    causation_id:          <adjustment.post.request derivative>

UI-02 cross-engine propagation (compensation):
  - Cash:  cash.refund.issued.v1 (TZS 22,000 returned via store credit OR refund channel)
  - Inventory:  inventory.stock.adjusted.v1 (1 unit restored if items were deducted)
  - Reporting:  November period projections updated; October Statement UNCHANGED

October 2026 trial balance (Statement STMT-PL-MZH-2026-10): UNCHANGED at TZS 5.3M profit.
November 2026 trial balance reflects the -TZS 22,000 revenue correction.

Audit trail walks: Nov 12 reversal → causation_id to Mzee Hassan command →
                   compensates_event_id to Oct 28 duplicate → fully traceable.

The original October Statement remains hash-verifiable in 2046 — the correction is
a forward-period event with a backward reference, both events in the store forever.
```

### What the Example Demonstrates

- **PC1**: Choreography — engines self-emit; Accounting coordinates Documents; no orchestrator
- **PC2**: Period defined per pack `tz-compliance-2026.07` (gregorian monthly)
- **PC3**: Human-initiated close.request from authorised accountant
- **PC4**: 4 core signals (Cash, Inventory, Procurement, HR); Promotion no signal (no campaigns active)
- **PC5**: Accounting coordinates via UI-07 + emission
- **PC6**: Statements as Documents post-close (CN-5-006 N3)
- **PC7**: Freeze window active Phase 2 → Phase 4
- **PC8**: Forward-correction (Nov 12 scenario)
- **PC9**: Every event in store; replayable
- **PC10**: Tenant-level close
- **PC11**: October closed independently; no cascade
- **UI-05** + **UI-07**: Phase 3 gate + Phase 4 permanent lock
- **N3 doctrine (CN-5-006)**: Statements as Documents, not snapshots
- **N1 doctrine (CN-5-103)**: `posting_period_ref` on forward correction
- **CN-5-010 hooks**: pre-close + post-close advisor suggestions

---

## 17. Open Items + Boundaries

| Item | Assigned to | Notes |
|------|-------------|-------|
| Pack content for `period_calendar`, `period_close.required_signals`, `period_close_statements`, `trial_balance_epsilon`, freeze window expiry per jurisdiction | Term 1 via CTR-042 + CTR-029 expansions | Schema in §3 + §6 + §7 + §8 + §13; per-jurisdiction content is Term 1's authorship |
| Statement template content (P&L, Balance Sheet, Cash Flow, statutory summaries) per jurisdiction | Term 1 via CTR-029 expansion | CN-5-006 §6 schema; pack content authored per jurisdiction |
| `accounting.period.close.initiated.v1` + `accounting.period.close.rejected.v1` event definitions | CN-5-001 cosmetic amendment (same commit as CN-5-104) | Per Q6/§5; added to CN-5-001 manifest |
| Advisor pre/post-close hooks (CN-5-010 §4.5/§4.1) | CN-5-010 + CTR-014 expansion | Schedule + cost ceilings per advisor |
| Platform billing aggregation post-close subscription | Term 1 via CTR-016 + CN-5-101 §7 | Subscribes `accounting.period.closed.v1` + `reporting.statement.issued.v1` |
| Cross-tenant aggregation governance | CTR-016 + Term 1 | Platform scope reads post-close events; doesn't participate in close decision |
| Snapshot-at-close performance optimisation (parallel-fold trial balance, etc.) | Architect phase | Concept guarantees correctness; performance is Architect's |
| Multi-currency reporting view at close (USD positions side-by-side with TZS) | CN-5-006 §11 + Future | v1 single functional currency in Statements; multi-currency view derived if pack supports |
| Doctrine check for PC7 freeze window (CN-4-019) | CN-4-019 + future DC | Static or integration test ensuring bus rejects in-period events during freeze window |
| Doctrine check for UI-05 forward-correction discipline (`posting_period_ref` mandatory on correction events) | CN-4-019 + future DC | Schema validation on `accounting.journal.reversed.v1` |
| Tenant-customer visibility of close summary (e.g., "your invoice from October has been finalised") | Term 3 future | D-DISC-001 pattern — engine provides; UX surfaces if pack permits |
| Period-close failure analytics over time (recurring imbalances, common causes) | CN-5-006 + CN-5-010 future | Reporting projection on `accounting.period.close.rejected.v1` events; advisor pattern detection |
| Statement amendment workflow detail (when Statement found erroneous post-close) | CN-5-006 §5 amendment mechanism (already specified) + future operational detail | Reissuance via new linked Document per CN-4-012 §6 |
| Annual-only close tenants (no monthly close in their pack) | Pack content + Term 1 | Pack permits; close mechanism identical; only annual close decision per year |

### Boundaries (Cross-References)

| Topic | Lives in |
|-------|----------|
| Period-state events + UI-05 bus policy mechanism | CN-5-001 §8 |
| Statement Document templates + issuance mechanics | CN-5-006 §5/§8 + CN-4-012 |
| UI-05 + UI-07 invariants | CN-5-102 |
| Period calendar primitive | Pack via CTR-042 + CN-4-014 |
| Subscription pattern P5 | CN-5-100 §2 |
| Scope policy (tenant-level close) | CN-5-101 |
| Event naming + glossary | CN-5-103 (G1-G9; close events in CN-5-001 manifest catalogue §8) |
| Per-engine readiness criteria detail | Per-engine docs (CN-5-002/003/004/005) — readiness criteria reference §4 here |
| Site-level operational close | CN-5-002 §5 (NOT this doc) |
| Cross-tenant billing aggregation | CTR-016 + Term 1 |
| Advisor pre/post-close hooks | CN-5-010 §4.5 + §4.1 |
| Multi-currency FX revaluation | CN-5-001 §11 + CN-5-002 §15 |
| Forward-correction pattern (compensates_event_id + posting_period_ref) | CN-5-103 E5/N1 + CN-5-001 §9 |

---

*— End of CN-5-104 —*
