# CN-5-002 — Cash Management Engine

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 5 — Universal Engines
> **Status:** For Overseer review.
> **Governing decisions:** D-001 (Universal Checkout — Cash subscribes to checkout.settled.v1); UI-04 (CN-5-102 — tender chain integrity to authorised origin); UI-06 (CN-5-102 — tenant functional currency invariant); UI-08 (CN-5-102 — Obligation bounds preserved via primitive); Law 5 (compliance is configured — variance thresholds, expense categories, tip tax routing in packs); Law 4 (flexibility — one engine serves SME with mixed cash + mobile money + card + safe + multi-currency).
> **CTRs:**
> - **OPEN (this doc depends on / extends):** CTR-029 (Term 1 — pack section gains: `operating_expense_categories`, `cash_reconciliation.variance_thresholds`, multi-currency policy, tip tax routing per jurisdiction); CTR-030 (Term 6 — verticals using layby / advance emit fulfillment events Cash subscribes to); CTR-031 (Term 7 — banking adapter contract analogous to CTR-006 payment adapter); CTR-027 (Term 1/2 — tenant functional currency + site registry); CTR-006 (referenced — payment adapter contract is the pattern CTR-031 follows).
> **Glossary:** See `MASTER-GLOSSARY.md` — Universal-engine Invariant (UI), Site, Scope level, Obligation.
> **Depends on:** CN-4-002 (envelope, causation_id, compensates_event_id); CN-4-004 (command bus + atomic emission + policy rejection); CN-4-011 (Workflow primitive for sessions, Obligation primitive for AR/AP/loans/advances/layby); CN-4-014 (clock protocol — business_date); CN-4-016 (anomaly detection — variance routing); CN-5-009 (Checkout — `checkout.settled.v1` carries tenders + refund compensation pattern); CN-5-001 (Accounting auto-journals on cash umbrella events #3–#5, #7, #11); CN-5-003 (Inventory reservation choreography for layby/advance fulfillment); CN-5-100 (subscription patterns — auto-receipt P3, refund P4 compensation); CN-5-101 (scope policy — Cash is multi-scope per §6: site default + tenant overrides for main_safe, consolidations, inter-site transfers, deposit, equity, payroll deduction); CN-5-102 (UI-04 tender chain, UI-06 currency, UI-08 obligation bounds).
> **Boundaries:** Workflow / Obligation primitive folds → CN-4-011; Checkout settlement mechanics → CN-5-009; Procurement lifecycle producing AP obligations → CN-5-004; HR loan approval logic → CN-5-005; Payroll computation and deduction events → CN-5-005; Accounting auto-journal mappings → CN-5-001; Inventory layby/advance fulfillment side → CN-5-003; payment adapter (mobile money / card) internals → Term 7 via CTR-006; banking adapter internals → Term 7 via CTR-031; statutory cash-flow reporting templates → CN-5-006 + pack content; bank account balance as a tracked till — deferred to v2 (§Open Items).

---

## 1. What This Doc Defines

CN-5-002 is the Cash Management Engine — the **money-position truth layer** for any business. The word "cash" in the engine name follows the Term 5 brief, but the scope is broader: every place money sits is a **till**, regardless of medium. A physical cash drawer, an M-Pesa wallet, a card-merchant balance pre-bank-settlement, a petty cash float at a site, a site safe, a main safe, a multi-currency drawer, a tip pass-through till — all are tills with the same shape, same lifecycle, same events.

The engine records the **money side** of every transaction. The relationships between parties (debts owed, prepayments received, loans extended) live on the **Obligation primitive** (CN-4-011) with appropriate `kind`. Cash and Obligation meet at settlement events. This separation prevents Cash from becoming a monolith spanning AR, AP, loans, checkout, and payment ledger; it is a money-position ledger plus an Obligation interactor.

This doc defines:
- The **till architecture** — 8 till kinds unified under one abstraction (§3).
- The **engine manifest** with multi-scope per-operation overrides (§4).
- The **session lifecycle** via Workflow primitive (§5).
- **Auto-receipt subscriptions** from Checkout and Term 7 adapters (§6).
- The **generic operating-expense pathway** with 14 pack-defined categories (§7).
- **AR / AP integration via Obligation** primitive (§8).
- The **employee loan cross-engine choreography** (HR + Cash + Payroll + Accounting) (§9).
- **Inter-till and inter-site transfers** including drop patterns (§10).
- **Bank deposit handoff** to Term 7 banking adapter (§11 + CTR-031).
- **Reconciliation and variance** with pack thresholds (§12).
- **Refund cash side** as compensation event (§13).
- **Owner equity flows** as Cash-owned commands (§14).
- **Multi-currency tills** with FX coordination (§15).
- **Tips / gratuity** — cash via separate till; digital via Checkout payload metadata (§16).
- **Layby / advance deposits** via Obligation kinds with vertical fulfillment choreography (§17).
- **UI-04 + UI-06 enforcement** structural to every emission (§18).

This doc does NOT define:
- Workflow or Obligation primitive folds (CN-4-011).
- Checkout's settlement mechanics or tender shapes (CN-5-009).
- Procurement's AP creation flow (CN-5-004).
- HR's loan approval logic or payroll computation (CN-5-005).
- Accounting's downstream auto-journal mappings (CN-5-001).
- Inventory's layby/advance fulfillment side (CN-5-003).
- Term 7 adapter internals (CTR-006 / CTR-031).
- Pack content per jurisdiction (Term 1 via CTR-029).

---

## 2. The Seven Laws

| # | Law | Source / why |
|---|-----|--------------|
| **C1** | **Till = money-position abstraction.** Every place money sits — a cash drawer, an M-Pesa wallet, a card-merchant balance, a petty cash float, a safe — is a till. Same shape, same lifecycle, same events. Subscribers operate uniformly on cash position events; till-kind specifics live in `kind` attribute + pack rules. | Universality. BOS does not invent per-medium engines; differences are configuration. |
| **C2** | **Site-scope default; tenant overrides for consolidations + main safe + inter-site movements.** Most tills are per-site (drawer per terminal, mobile money wallet per site, petty cash per site). Main safe is tenant-scope (only one). Inter-site transfers, deposits, equity flows, and consolidated queries operate at tenant. Multi-scope manifest per CN-5-101 §6. | CN-5-101 site-scope default with explicit overrides; complementary with CN-5-001 A4. |
| **C3** | **Every cash event traces to authorised origin (UI-04).** Every `cash.tender.received.v1` walks via `causation_id` to one of: `checkout.settled.v1`, `cash.refund.issued.v1`, `cash.transfer.received.v1`, `cash.collection.received.v1`, `cash.equity.injected.v1`. The set is closed; adding origins is a doctrine extension. | UI-04 (CN-5-102); audit completeness (Charter §1.2). |
| **C4** | **Currency invariant per till (UI-06).** Every till declares a `currency`. Tender / receipt / disbursement currency must equal till currency at command-evaluation time. Multi-currency operations require explicit FX events bridging into functional currency for Accounting. | UI-06 (CN-5-102); supports multi-currency tills (Zanzibar USD) without silent unit mixing. |
| **C5** | **Session lifecycle via Workflow primitive.** Every operational till runs sessions: open with float → in-use → reconcile → close. Sessions are event-sourced via the Workflow primitive (CN-4-011). Continuous-session tills (mobile money, main safe under guard) use one open session per reconciliation period. | CN-4-011 Workflow primitive; audit-ready operational truth. |
| **C6** | **All debts / credits / commitments are Obligation primitive.** AR receivables, AP payables, employee loans, customer layby, advance deposits, delivery owed — every "money owed/promised" relationship is an Obligation (CN-4-011) with appropriate `kind`. Cash records the money side; Obligation records the relationship side. They meet at settlement events. | Unifies cross-engine choreography on one primitive (consistent with CN-5-003 backorder wrap, CN-5-009 credit tender wrap); UI-08 bounds inherit. |
| **C7** | **Reconciliation variance handled per pack threshold.** Pack declares variance bands per till kind. Within `auto_adjust_threshold` → auto-recorded adjustment; within `anomaly_threshold` → CN-4-016 anomaly alert + flagged close requires authorised override; above → hard-block close. Tenant overrides allowed within pack permission (±50% default). | Operational reality (small variances happen); large variances are fraud/error signals; pack governs per jurisdiction. |

---

## 3. Till Architecture — 8 Kinds Unified

| # | `kind` | Per | Reconciliation source | Notes |
|---|--------|-----|-----------------------|-------|
| 1 | `physical_cash_drawer` | terminal × site | Physical count | Default operational till; session per shift |
| 2 | `mobile_money_wallet` | provider × site | Provider statement (digital) | One per provider per site — M-Pesa Mwanza ≠ M-Pesa Mbeya |
| 3 | `card_merchant_balance` | provider × site | Bank settlement statement | Pre-settlement holding; **v2-optional** for default SME setup |
| 4 | `petty_cash_float` | site | Periodic count + receipts | Site-level operational float for small purchases |
| 5 | `site_safe` | site | Daily count after drop reconciliation | End-of-day drop destination |
| 6 | `main_safe` | **tenant** (only one per tenant) | Periodic count | Receives drops from site safes; pre-bank-deposit holding; **tenant scope** (C2) |
| 7 | `multi_currency_drawer` | terminal × site × currency | Physical count per currency | Opt-in per till; currency declared at registration |
| 8 | `tip_passthrough_till` | site | Distribution log | Tracks cash tips; not business revenue (§16); pack governs tax routing |

### Till Registration Record

```yaml
cash_till:
  till_id:                  <unique>
  tenant_id:                <envelope>
  site_id:                  <site> | null    # null for main_safe (tenant scope)
  kind:                     <one of 8 above>
  currency:                 <ISO code; matches tenant functional unless multi-currency till>
  display_name:             "Mwanza Cashier-A Drawer"
  attributes:
    provider:               M-Pesa | Tigo | Airtel | Visa | Mastercard | …    # mobile_money / card tills
    requires_session:       boolean    # true for human-operated; false for continuous (mobile money digital)
    opening_float:          decimal | null
    reconciliation_frequency: per_session | daily | weekly | per_drop
    auto_adjust_threshold:    decimal (pack default; tenant override within ±50%)
    anomaly_threshold:        decimal (pack default; tenant override within ±50%)
```

### Why One Abstraction

Verticals never see till specifics. Subscribers downstream (Accounting, Reporting) operate on cash position events uniformly. Behaviour differences (digital vs physical reconciliation, session vs continuous lifecycle, FX implications of multi-currency drawers) live in the `kind` attribute and pack rules. Adding a new till medium in future (e.g., crypto wallet) is a new `kind` plus pack rules, not new engine code.

---

## 4. Engine Manifest

Cash is a **multi-scope engine** per CN-5-101 §6: `site` default with explicit `tenant`-scope overrides for tenant-wide operations (main safe, consolidations, inter-site movements, deposits, equity, payroll deduction subscription).

```yaml
engine_id:        cash
display_name:     Cash Management Engine
version:          1
scope_policy:     site                                  # C2 default
primitives_used:
  - workflow                                            # session lifecycle (CN-4-011)
  - obligation                                          # AR/AP/loans/layby/advance (CN-4-011)

commands:
  # Till lifecycle (tenant — till is tenant-owned identity)
  - id: cash.till.register.request                      scope_ref: tenant
  - id: cash.till.attributes_update.request             scope_ref: tenant
  - id: cash.till.deactivate.request                    scope_ref: tenant

  # Session lifecycle (site for most; tenant for main_safe sessions)
  - id: cash.session.open.request                       # scope_ref: site (default)
  - id: cash.session.reconcile.request                  # scope_ref: site
  - id: cash.session.close.request                      # scope_ref: site

  # Operational money movements (site)
  - id: cash.expense.record.request                     # generic operating expense
  - id: cash.collection.record.request                  # AR collection (settles receivable)
  - id: cash.payment.record.request                     # AP payment (settles payable)
  - id: cash.transfer.between_tills.request             # within site

  # Cross-site / tenant
  - id: cash.transfer.between_sites.request             scope_ref: tenant
  - id: cash.deposit.initiate.request                   scope_ref: tenant
  - id: cash.deposit.complete.request                   scope_ref: tenant     # system-submitted on adapter outcome

  # Equity (tenant — owner action)
  - id: cash.equity.injection.record.request            scope_ref: tenant
  - id: cash.equity.withdrawal.record.request           scope_ref: tenant

  # Tips & adjustments
  - id: cash.tip.record.request                         # gratuity passthrough (cash tip)
  - id: cash.adjustment.record.request                  # variance within auto-threshold (system-submitted)

  # Layby / advance (creates Obligation)
  - id: cash.advance_received.record.request            # customer prepayment / layby instalment

emits:
  # Till + session
  - cash.till.registered.v1
  - cash.till.attributes_updated.v1
  - cash.till.deactivated.v1
  - cash.session.opened.v1
  - cash.session.closed.v1
  - cash.session.reconciled.v1
  - cash.variance.detected.v1                           # → CN-4-016 anomaly routing
  - cash.variance.adjusted.v1                           # auto within threshold

  # Money flows (the operational primary stream)
  - cash.tender.received.v1                             # UI-04 origin chain
  - cash.tender.disbursed.v1                            # primary outflow
  - cash.expense.recorded.v1                            # umbrella → CN-5-001 #4
  - cash.collection.received.v1                         # AR settlement (Obligation reduces)
  - cash.payment.disbursed.v1                           # AP settlement (Obligation reduces)

  # Transfers & deposits
  - cash.transfer.initiated.v1
  - cash.transfer.received.v1
  - cash.transfer.between_sites.v1                      # CN-5-001 #5 subscribes
  - cash.deposit.requested.v1                           # → Term 7 banking adapter (CTR-031)
  - cash.deposit.completed.v1
  - cash.deposit.failed.v1

  # Equity
  - cash.equity.injected.v1
  - cash.equity.withdrawn.v1

  # Tips
  - cash.tip.recorded.v1
  - cash.tip.distributed.v1                             # cash tip pass-through to staff

  # Layby / advance
  - cash.advance_received.recorded.v1                   # creates Obligation kind: advance_received | layby

  # Refund
  - cash.refund.issued.v1                               # compensation event (compensates_event_id non-null)

  # Adjustments umbrella (for Accounting)
  - cash.adjustment.recorded.v1                         # umbrella for variance + write-off events to Accounting

subscribes_to:
  # Checkout — primary receipt source
  - {event_type: checkout.settled.v1,                  version: 1, handler: record_tender_receipt,        kind: command_emitting, scope_ref: site}
  - {event_type: checkout.refunded.v1,                 version: 1, handler: issue_refund_cash,            kind: compensation,     scope_ref: site}

  # Term 7 payment adapters (CTR-006) — confirm tender outcomes
  - {event_type: <mm-adapter>.tender.outcome.v1,       version: 1, handler: record_mm_tender_outcome,     kind: command_emitting, scope_ref: site}
  - {event_type: <card-adapter>.tender.outcome.v1,     version: 1, handler: record_card_tender_outcome,   kind: command_emitting, scope_ref: site}

  # Term 7 banking adapter (CTR-031) — deposit outcomes
  - {event_type: <bank-adapter>.deposit.outcome.v1,    version: 1, handler: record_deposit_outcome,       kind: command_emitting, scope_ref: tenant}

  # Mobile money provider statements (digital reconciliation)
  - {event_type: <mm-adapter>.statement.received.v1,   version: 1, handler: reconcile_mm_wallet,          kind: command_emitting, scope_ref: site}

  # HR loan approval
  - {event_type: hr.loan.approved.v1,                  version: 1, handler: disburse_employee_loan,       kind: command_emitting, scope_ref: site}

  # Payroll deduction (settles employee loan Obligation)
  - {event_type: payroll.deduction.applied.v1,         version: 1, handler: record_loan_repayment,        kind: command_emitting, scope_ref: tenant}

  # Vertical layby/advance fulfillment (CTR-030 expansion — per-vertical entries grow manifest)
  - {event_type: <vertical>.advance_consumed.v1,       version: 1, handler: reduce_advance_obligation,    kind: command_emitting, scope_ref: site}
  - {event_type: <vertical>.layby_release.v1,          version: 1, handler: close_layby_obligation,       kind: command_emitting, scope_ref: site}

requires:
  - checkout.settled.v1                                 # cannot serve a transacting business without receipt source
```

---

## 5. Session Lifecycle

The Workflow primitive (CN-4-011) backs the session state machine:

```
        ┌────────────────── reopen for adjustment ──────────────────┐
        ▼                                                            │
[opened] ──record movements──► [in_use] ──reconcile──► [reconciled] ──close──► [closed]
```

For human-operated tills (`requires_session: true`): one session per shift; cashier opens with declared float; movements record during shift; reconciles with physical count; closes with variance handling per C7.

For continuous tills (`requires_session: false`, e.g., mobile money wallet): one session per reconciliation period (typically daily); reconciled against provider statement event (`<mm-adapter>.statement.received.v1`); closed automatically on statement arrival.

### Variance Handling at Close (C7)

```
expected = opening_float + sum(receipts) − sum(disbursements)
counted  = physical count (cash tills) | provider statement balance (digital tills)
variance = counted − expected

IF |variance| ≤ auto_adjust_threshold:
  emit cash.variance.adjusted.v1  (auto)
  emit cash.adjustment.recorded.v1 (umbrella → CN-5-001)
  emit cash.session.reconciled.v1; close proceeds normally

ELIF |variance| ≤ anomaly_threshold:
  emit cash.variance.detected.v1  (→ CN-4-016 anomaly routing)
  emit cash.session.reconciled.v1 (flagged); close requires authorised role override

ELSE  (|variance| > anomaly_threshold):
  hard-block close. close.request rejected:
    rejected_by_policy:  C7.variance_exceeds_anomaly_threshold
  Investigation required; only authorised role may force-close with documented reason.
```

Pack defines thresholds per till kind (digital wallets tolerate near-zero variance; physical drawers tolerate more). Tenant override allowed within ±50% of pack default (Q6 ruling).

---

## 6. Auto-Receipt Subscriptions

The primary receipt source is Checkout. Cash subscribes to `checkout.settled.v1` and records one tender receipt per tender in the settlement.

### Handler `record_tender_receipt`

For each tender in `checkout.settled.v1` payload:
- Resolve target till from `(tenant, site, tender.method, tender.currency)` lookup.
- Verify till is in `opened` or `in_use` state.
- Verify currency matches till currency (C4 — UI-06).
- Emit `cash.tender.received.v1`:
  ```
  causation_id:      <event_id of checkout.settled.v1>    (UI-04)
  payload:
    till_id:          <resolved till>
    tender_id:        <from checkout payload>
    amount:           tender.amount
    currency:         tender.currency
    method:           tender.method
    external_ref:     tender.external_ref               # adapter ref for async
    business_date:    checkout.payload.business_date
  ```

For async tenders, the receipt is emitted only after `checkout.tender.confirmed.v1` for that tender — `record_tender_receipt` keys off the **settled** event which by K5 of CN-5-009 means all tenders confirmed.

### Term 7 Payment Adapter Outcomes (Reconciliation Channel)

Adapter outcome events (`<mm-adapter>.tender.outcome.v1`) primarily feed back into Checkout (CN-5-009 Phase 3). For Cash's reconciliation, the **mobile money provider statement** event (`<mm-adapter>.statement.received.v1`) provides the digital reconciliation source: the statement enumerates the day's settled transactions; Cash matches against its recorded receipts; variance is computed automatically.

### Term 7 Banking Adapter Outcomes (Deposit Channel)

`<bank-adapter>.deposit.outcome.v1` feeds back into Cash's deposit-handoff flow (§11).

---

## 7. Generic Operating Expense Pathway

`cash.expense.record.request` is the site-level entry for expenses that do NOT flow through Procurement (rent, utilities, fuel, transport, small repairs, supplies, communications, insurance, marketing, bank charges, staff welfare, professional fees, financial charges, other).

```yaml
cash.expense.record.request:
  scope:           site
  payload:
    till_id:                <funding till>
    expense_category:       <pack-defined>
    amount:                 decimal
    currency:               <matches till currency — UI-06>
    payee_ref:              <party reference> (optional)
    business_date:          <CN-4-014>
    supporting_document_ref: <receipt photo, voucher ID>
    reason_text:            <free-form>
```

### Pack-Defined Expense Categories (N1 — 14 universal)

```yaml
# In pack content (CTR-029 expansion)
operating_expense_categories:
  - {code: "rent",                  account_code_default: "6100", taxable_input: true}
  - {code: "utilities",             account_code_default: "6200"}
  - {code: "fuel",                  account_code_default: "6300"}
  - {code: "transport",             account_code_default: "6310"}
  - {code: "repairs",               account_code_default: "6400"}
  - {code: "supplies",              account_code_default: "6500"}
  - {code: "communications",        account_code_default: "6600"}
  - {code: "insurance",             account_code_default: "6650"}     # N1 added
  - {code: "marketing_advertising", account_code_default: "6700"}     # N1 added
  - {code: "bank_charges",          account_code_default: "6750"}     # N1 added
  - {code: "staff_welfare",         account_code_default: "6800"}
  - {code: "professional_fees",     account_code_default: "6850"}
  - {code: "financial_charges",     account_code_default: "6900"}
  - {code: "other",                 account_code_default: "6999"}
```

Pack may extend per jurisdiction (e.g., a jurisdiction-specific "regulatory_levy" category). Tenants add sub-categories within pack permission.

### Flow

`cash.expense.record.request` accepted → emits `cash.expense.recorded.v1` (umbrella) + `cash.tender.disbursed.v1` (the actual money outflow). CN-5-001 #4 auto-journals: Dr Expense (account per pack mapping); Cr Cash (till's underlying account).

---

## 8. AR / AP Integration via Obligation Primitive

| Direction | Created by | Cash interaction | Obligation `kind` |
|-----------|-----------|------------------|-------------------|
| **AR (customer owes)** | CN-5-009 credit tender → Obligation at settlement | `cash.collection.record.request` settles portion of receivable | `accounts_receivable` |
| **AP (business owes)** | CN-5-004 `procurement.invoice.recorded.v1` → Obligation | `cash.payment.record.request` settles portion of payable | `accounts_payable` |
| **Employee loan (employee owes)** | CN-5-005 `hr.loan.approved.v1` → Obligation | Cash disburses (§9); Payroll deductions settle | `employee_loan` |
| **Delivery owed (business owes goods)** | CN-5-003 `inventory.promise.created.v1` → Obligation | Cash inflow occurred at sale; Inventory `promise.fulfilled` reduces | `delivery` |
| **Advance received (customer prepaid for goods/services)** | `cash.advance_received.record.request` → Obligation | Cash inflow now; vertical fulfillment events reduce | `advance_received` |
| **Layby (customer instalments)** | Vertical `<vertical>.layby.created.v1` → Obligation | Cash records each instalment; obligation reduces; on full → release goods | `layby` |

### Settlement Command Shape (AR / AP)

```yaml
cash.collection.record.request:           # AR collection
  scope:           site
  payload:
    till_id:                <receiving till>
    obligation_ref:         <AR obligation id>
    amount:                 decimal      # partial or full
    currency:               <UI-06>
    payer_party_ref:        <customer>
    business_date:          <CN-4-014>
    payment_method:         cash | mobile_money | card | bank_transfer
    external_ref:           <adapter ref for async methods>
```

Bus validates:
- Obligation exists and `outstanding > 0` (UI-08 bounds).
- `amount ≤ outstanding`.
- Currency matches till (C4).
- For async payment methods, external_ref present.

Emits paired:
- `cash.collection.received.v1` (Cash side — recorded receipt with origin = AR collection per UI-04).
- Obligation primitive: `obligation.settled.v1` (partial or full per amount; CN-4-011 fold).

`cash.payment.record.request` is the AP analogue (outflow + Obligation reduces).

### Why Two Primitives Meet, Not Merge

A receivable and the cash that settles it are conceptually different. The receivable is a relationship (this customer owes us X for that invoice). The cash is a money-position movement (TZS X arrived at till Y at time Z). Coupling them in one primitive would either:
- Force Cash to know about Obligations (lifecycle, kinds, bounds) — overloading the engine.
- Force Obligations to know about money positions — overloading the primitive.

Two primitives meeting at settlement events keeps each concern clean and the audit trail explicit.

---

## 9. Employee Loan Cross-Engine Choreography

Full scope across HR, Cash, Payroll, Accounting. One Obligation owns the truth; events drive its lifecycle.

### Step 1 — HR Approves Loan

```
hr.loan.approved.v1
  payload:
    employee_ref, principal_amount, interest_rate_ref,
    repayment_schedule: [{period, instalment_amount}, …],
    terms_ref
  → Obligation primitive creates:
    obligation_id, kind: employee_loan,
    creditor: tenant, debtor: employee,
    original_amount: principal, outstanding: principal
```

### Step 2 — Cash Subscribes; Disburses

```
Cash handler disburse_employee_loan reacts to hr.loan.approved.v1.
  Submits cash.expense.record.request:
    expense_category: employee_loan_disbursement   (special category — pack defines)
    obligation_ref:   <loan obligation>
    amount:           principal_amount
    currency:         <tenant functional>
    till_id:          <funding till, typically drawer or petty cash>

  Bus accepts; emits:
    cash.expense.recorded.v1   (causation: hr.loan.approved.v1 — UI-01)
    cash.tender.disbursed.v1   (origin chain: employee_loan)
```

### Step 3 — Accounting Auto-Journals (CN-5-001 #4)

```
Dr Employee Loan Receivable     (per pack account mapping)
Cr Cash                          (till's underlying account)
```

### Step 4 — Each Month: Payroll Computes

```
payroll.computed.v1               (gross salary for the employee)
payroll.deduction.applied.v1
  payload:
    employee_ref, obligation_ref, instalment_amount, period
```

### Step 5 — Cash Subscribes Payroll Deduction

```
Cash handler record_loan_repayment reacts to payroll.deduction.applied.v1.
  - Obligation primitive: settle portion (instalment_amount) — UI-08 bounds enforced
  - Emit cash.payment.disbursed.v1 representing the NET cash actually paid to employee
    (gross salary minus deduction; the deducted portion never physically left)
```

### Step 6 — Accounting Auto-Journals Each Period (CN-5-001 #10 + #11)

```
Dr Salary Expense               (gross)
Cr Employee Loan Receivable     (instalment portion — Obligation reduces)
Cr Tax W/H, Pension W/H, etc.   (other deductions)
Cr Cash                          (net actually paid)
```

### Step 7 — Repeat Until Obligation Settled

Final payroll deduction reduces `outstanding` to zero. Obligation primitive fold transitions to `settled` status.

### Cancellation / Write-Off

Employee departure with outstanding loan: HR emits `hr.loan.write_off.requested.v1`; Cash subscribes, emits Obligation compensation (UI-02 symmetry); Accounting writes off the receivable to bad debt expense. The lifecycle remains traceable end-to-end via causation chains.

### UI-08 Bounds Throughout

`outstanding ∈ [0, original_amount]` at every step. Bus rejects any deduction that would breach. Compensation paths preserve symmetry.

---

## 10. Inter-Till and Inter-Site Transfers

### Within Site (Drop Pattern)

End-of-shift drop from drawer to site safe:

```yaml
cash.transfer.between_tills.request:
  scope:           site
  payload:
    from_till_id:           <drawer>
    to_till_id:              <site_safe>
    amount, currency, business_date
```

Emits:
- `cash.transfer.initiated.v1` (from-side reservation conceptually)
- `cash.transfer.received.v1` (to-side, immediate within site)

Single-site transfers are confirmed instantly (no transit physics; the cashier walks five metres). No adapter intermediary.

### Cross-Site (Two-Phase, Per CN-5-101 §6)

```yaml
cash.transfer.between_sites.request:
  scope:           tenant
  payload:
    from_site_id, to_site_id,
    from_till_id, to_till_id,
    amount, currency, business_date
```

Two-phase analogous to CN-5-003 §12 N4 transfer:
1. **Initiate** at from-site → from-till reservation; emit `cash.transfer.initiated.v1`.
2. **Physical transit** (courier, armoured cash-in-transit service). No event during transit; reservation holds the audit position.
3. **Receive** at to-site → `cash.transfer.between_sites.request` confirm phase or paired `inventory.transfer.receive`-analog command → emits `cash.transfer.between_sites.v1` (the umbrella event Accounting #5 subscribes to).

Transit losses or discrepancies emit paired `cash.adjustment.recorded.v1` with reason `transit_loss`.

### Drop Frequency

Per till `attributes.reconciliation_frequency` declares when drops happen. Pack may set defaults (e.g., per_session for drawers; daily for site_safe to main_safe).

---

## 11. Bank Deposit Handoff (CTR-031)

Banking is an **external endpoint** in v1; the bank account itself is not a tracked till (see Open Items for v2). Cash hands off deposits to a Term 7 banking adapter following the CTR-006 payment-adapter pattern.

### Proposed Pattern (CTR-031 Contract)

```
Cash emits:                 cash.deposit.requested.v1
                            payload:
                              from_till_id:           main_safe (typically)
                              amount, currency,
                              bank_account_ref:        <bank account identifier>
                              callback_correlation_id: <Cash uses to route adapter outcome back>
                              business_date

Term 7 banking adapter consumes; invokes bank API / cash-in-transit settlement.

Adapter emits:              <bank-adapter>.deposit.outcome.v1
                            payload:
                              callback_correlation_id,
                              outcome: confirmed | failed,
                              external_ref: <bank transaction id>,
                              reason?

Cash subscribes (handler record_deposit_outcome); submits
                            cash.deposit.complete.request
Cash emits:                 cash.deposit.completed.v1 OR cash.deposit.failed.v1
                            payload includes external_ref, settlement_date, etc.
```

### Failure & Retry

Failed deposits leave main_safe balance unchanged; the failed event is recorded for audit; operator investigates; retry submits a new `cash.deposit.initiate.request` (new correlation, new outcome). No automatic retry by the engine — banking errors typically need investigation.

### Accounting Effect

Successful deposit emits a `cash.transfer.between_sites.v1`-equivalent (or a dedicated `cash.deposit.completed.v1` umbrella reflected in CN-5-001's auto-journal): Dr Bank (external account); Cr Cash (main_safe till's underlying account). The bank account is recorded as an asset account in the chart (per pack); reconciliation against bank statements is a Term 7 + CN-5-006 (Reporting) concern.

---

## 12. Reconciliation & Variance

### Pack Schema (CTR-029 Expansion)

```yaml
# In pack content
cash_reconciliation:
  variance_thresholds:
    physical_cash_drawer:        {auto_adjust: TZS 500,  anomaly: TZS 5000}
    mobile_money_wallet:         {auto_adjust: TZS 100,  anomaly: TZS 1000}
    petty_cash_float:            {auto_adjust: TZS 200,  anomaly: TZS 2000}
    site_safe:                   {auto_adjust: TZS 1000, anomaly: TZS 10000}
    main_safe:                   {auto_adjust: TZS 2000, anomaly: TZS 20000}
    card_merchant_balance:       {auto_adjust: TZS 100,  anomaly: TZS 1000}
    multi_currency_drawer:       {auto_adjust: 5 USD,    anomaly: 50 USD}    # per currency
    tip_passthrough_till:        {auto_adjust: TZS 200,  anomaly: TZS 2000}
  tenant_override_allowed:       true
  tenant_override_bounds:        ±50%       # tenant cannot exceed pack defaults by more than half
  authorised_role_for_override:  manager | owner | …
```

### Variance Routing

Per C7 — see §5 (Variance Handling at Close) for the algorithm.

### Provider Statement Reconciliation (Mobile Money)

Mobile money tills reconcile differently:
- Daily statement event arrives from provider via Term 7 adapter (`<mm-adapter>.statement.received.v1`).
- Engine handler `reconcile_mm_wallet` matches statement against recorded receipts.
- Discrepancies (a recorded receipt with no matching statement entry, or vice versa) emit `cash.variance.detected.v1`.
- Match within tolerance → automatic `cash.session.reconciled.v1` + close.

### Card Settlement Reconciliation (Card Merchant Balance — v2-optional)

Analogous to mobile money: bank settlement statement matches against card receipts; differences are timing (settlement delay) or fee deductions (pack categorises fees as `bank_charges`).

---

## 13. Refund Cash Side

`checkout.refunded.v1` (CN-5-009 §7) triggers Cash's `issue_refund_cash` handler with `kind: compensation`:

```
cash.refund.issued.v1
  compensates_event_id:  <event_id of original cash.tender.received.v1 chain>    (UI-02 / N4 of CN-5-001)
  causation_id:          <event_id of checkout.refunded.v1>                       (UI-01)
  payload:
    till_id:                <refund-out till — symmetric with original receipt direction>
    refund_tenders:         [<reverse-direction tender list>]
    refund_total
    currency
    refund_method:          cash | mobile_money | card | bank_transfer
    business_date:          <CN-4-014 — current open period; UI-05 forbids back-dating>
```

For async refund methods (mobile money refund), Cash emits a `cash.tender.requested.v1` analogue (refund-direction marker) → Term 7 adapter handles → outcome event arrives → Cash records.

UI-04 origin chain: the refund event's causation walks back to the original tender receipt; audit reconstructs the full forward + back chain.

UI-02 compensation symmetry: downstream Accounting and Reporting see the compensation and reverse their derivative effects appropriately.

### Partial Refund (UI-02 Open Item from CN-5-009 §7)

Cash records the partial refund amount; Obligation reductions (if original sale created an Obligation, e.g., credit tender → AR) compensate proportionally per CN-4-011 fold. Per-engine partial-refund mapping is an Open Item carried from CN-5-009 §15.

---

## 14. Owner Equity Flows

Owner equity transactions are business events with operational cash effects. Cash Engine owns the commands (Q9 ruling); the **cash impact is the operational truth** (someone walked in with money, or out with money); Accounting reflects via auto-journal #3 / #4.

```yaml
cash.equity.injection.record.request:
  scope:           tenant
  required_role:   owner_or_authorised
  audit:           dual-actor (owner + recorder)         # CN-4-007 §3
  payload:
    till_id:                <receiving till — typically main_safe or main drawer>
    amount, currency, business_date
    source_text:            free-form (e.g., "owner contribution from personal savings")
    supporting_document_ref: <receipt, transfer slip>
```

Emits `cash.equity.injected.v1` → CN-5-001 #3 auto-journals: Dr Cash; Cr Equity Injection (per pack account mapping).

```yaml
cash.equity.withdrawal.record.request:
  scope:           tenant
  required_role:   owner_or_authorised
  audit:           dual-actor
  payload:
    till_id:                <funding till>
    amount, currency, business_date,
    reason_text
```

Bus validates: tenant equity account permits withdrawal per pack rules (some jurisdictions restrict withdrawals below equity threshold). Emits `cash.equity.withdrawn.v1` → CN-5-001 auto-journal: Dr Owner Drawings; Cr Cash.

### Authorised Role Policy

Per-tenant role policy governs who may submit equity commands. Default: owner role only (multi-owner tenants require dual-actor approval). Open Item — Term 1 may declare a CTR-032 in future for full governance.

---

## 15. Multi-Currency Tills

Per till declaration:

```yaml
cash_till:
  kind:              multi_currency_drawer
  currency:          USD                          # till's working currency (may differ from tenant functional)
```

### Use Cases

- Zanzibar hotel: tenant functional currency TZS; one TZS drawer + one USD drawer (tourist payments).
- Cross-border trader: TZS + USD + KES drawers.

### Coordination with CN-5-001 §11 FX

Cash receipts in non-functional currency:
- The receipt is recorded in the till's currency (preserves operational truth).
- A paired FX event (`<fx-adapter>.fx.rate.recorded.v1` or future FX engine) brings the amount into functional currency for Accounting.
- Pack rules govern FX rate sourcing (CN-5-001 §15 Open Item).

### Cross-Currency Till Transfer

Transferring USD-drawer cash to TZS-drawer cash at end of day (owner conversion):
- `cash.transfer.between_tills.request` with mismatched currencies is **rejected** unless paired with FX event.
- The proper flow: emit FX event; emit two transfers (USD out at USD rate; TZS in at converted amount) with explicit FX bridging.

UI-06 enforced throughout — no silent unit mixing.

---

## 16. Tips / Gratuity

### Cash Tips → Separate Till

`tip_passthrough_till` (kind 8 in §3) receives cash tips. Balance accumulates until distribution to staff.

```yaml
cash.tip.record.request:
  scope:           site
  payload:
    till_id:                <tip till>
    sale_ref:               <checkout.settled event id> (optional — sometimes tip is independent)
    tip_amount, currency,
    receiving_staff_ref:    <employee or pool ref>
```

Emits `cash.tip.recorded.v1`. Later, when tips are distributed: `cash.tip.distributed.v1` (paired with payroll integration if pack treats tips as taxable wages).

### Digital Tips (N2 — Distinct from Cash)

Digital tips (card-add-tip line; mobile money tip line) are **payload metadata on `checkout.settled.v1`**, not a separate till. Checkout's settlement event carries:

```
checkout.settled.v1 payload (extension via vertical):
  tenders:           [<incl. tender with method: card and tip portion>]
  tip_breakdown:     {tip_amount, tip_recipient_ref?}    # optional metadata
```

Pack rules govern routing:
- **Pass-through jurisdiction**: tip portion is recorded as `cash.tip.recorded.v1` (digital), staff payout via payroll without tax.
- **Wages jurisdiction**: tip portion flows through payroll as taxable income; pack's accounting recognition_rules direct the journal.

### Why Distinct Treatment

Cash tips physically pass through staff hands (a customer hands a server a 5,000 TZS note); the till is a passthrough holding pen. Digital tips never have a separate physical reality — they ride on the card/mobile-money receipt and are split downstream. Distinct handling reflects the distinct reality without forcing one model on both.

---

## 17. Layby / Advance Deposits (N4 — Semantic Distinction)

Both wrap the Obligation primitive (CN-4-011) but with different semantics:

| Aspect | **advance_received** | **layby** |
|--------|----------------------|-----------|
| Direction | Single-direction: business owes goods/services worth amount received | Phased: customer accumulates payments to reach a price; goods released only on full payment |
| Reduction | Reduces as fulfillment events arrive (services rendered, goods delivered) | Reduces with each instalment paid in (cash side); no goods released until outstanding = 0 (or per pack rule) |
| Vertical role | Vertical emits `<vertical>.advance_consumed.v1` per fulfillment | Vertical emits `<vertical>.layby_release.v1` only when full payment reached |
| Risk | If business cancels, refund customer the unconsumed portion | If customer abandons, pack rules govern (full refund, partial refund, forfeiture) |

### Advance Received Flow

```
cash.advance_received.record.request:
  scope:           site
  payload:
    till_id:                <receiving till>
    customer_party_ref:     <customer>
    advance_kind:           advance_received
    related_to:             <future order ref / quote ref>
    amount, currency, business_date
```

Emits:
- `cash.advance_received.recorded.v1` (Cash inflow)
- Obligation primitive: `kind: advance_received`; debtor: tenant; creditor: customer; outstanding: amount

As vertical renders work, emits `<vertical>.advance_consumed.v1` → Cash handler `reduce_advance_obligation` reduces Obligation outstanding by the fulfilled portion. UI-08 bounds: outstanding ∈ [0, original_amount].

### Layby Flow

```
Initial — vertical emits <vertical>.layby.created.v1 (or Cash command for layby setup; vertical-side decision)
  → Obligation primitive: kind: layby; debtor: customer; creditor: tenant
    outstanding: total_price (the amount customer must accumulate)

Each instalment:
  cash.advance_received.record.request (advance_kind: layby)
    → cash.advance_received.recorded.v1 (cash inflow)
    → Obligation reduces by instalment amount

On full payment (outstanding = 0):
  Vertical (Term 6 / Retail) detects → emits <vertical>.layby_release.v1
  → Cash subscribes (close_layby_obligation handler) → emits paired:
    - cash.tender.received.v1 reaches its terminal state — accumulated cash has already arrived; no new flow
    - Obligation transitions to settled
  → Vertical releases goods (inventory.movement deduction in coordination per N5)
```

### N5 — Cross-Engine Coordination (Layby + Workshop + Inventory)

Layby or advance frequently couples with Inventory reservations (CN-5-003 §9). Example: workshop accepts customer advance for a custom window; vertical creates Inventory reservation for the aluminium sheet; reservation is held while the customer pays in instalments; on full payment, reservation confirms (inventory deducts) AND advance Obligation settles in lockstep.

The choreography:
1. Customer commits → vertical emits `workshop.quote.accepted.v1` → Inventory reservation + customer Obligation (advance kind) created.
2. Customer pays instalments → Cash records each → advance Obligation reduces; Inventory reservation untouched.
3. Customer pays final instalment → outstanding = 0 → vertical emits `workshop.layby_release.v1` (or analogous) → Inventory reservation confirms (stock deducts) + Cash closes Obligation in lockstep.

Both Obligation reduction (Cash side) and Inventory reservation→consumed (stock side) proceed together; UI-08 bounds + CN-5-003 §9 reservation lifecycle inherit.

---

## 18. UI-04 + UI-06 Enforcement

### UI-04 (Tender Chain to Authorised Origin)

Every `cash.tender.received.v1` walks via `causation_id` chain to one of the closed authorised origins (C3). Cash's `record_tender_receipt` handler refuses to emit a receipt whose causation does not terminate at:
- `checkout.settled.v1` (sale)
- `cash.refund.issued.v1` (refund return to till — uncommon but valid for refund-of-refund)
- `cash.transfer.received.v1` (intra-tenant transfer)
- `cash.collection.received.v1` (AR collection)
- `cash.equity.injected.v1` (owner injection)

Integration test (Phase 1 DC pending CN-5-002 ratification) walks sampled receipts via causation; assert terminal in authorised set.

### UI-06 (Functional Currency)

Per-till currency declared at registration. Every command boundary checks:
- `cash.tender.add.request` (in Checkout — already enforced upstream).
- `cash.expense.record.request`: expense currency = till currency.
- `cash.collection.record.request` / `cash.payment.record.request`: payment currency = till currency = Obligation currency.
- `cash.transfer.between_tills.request`: both till currencies match, OR paired FX event is in the same correlation.
- `cash.advance_received.record.request`: amount currency = till currency.

Mismatches are rejected at the bus:
```
rejected_by_policy:  UI-06.functional_currency_invariant
rejection_reason:    "Tender currency does not match till currency"
```

---

## 19. Example — Mama Amina's Full Day (Mwanza)

*Mama Amina's duka serves as illustrative context per D-004 #4 (peer-technical audience). The example demonstrates the engine contract across till diversity, AR/AP/loan flows, multi-tender refunds, and reconciliation.*

**Tills:**
- `drawer-mwanza-A` (physical_cash_drawer, TZS, opening float TZS 500,000)
- `mpesa-mwanza` (mobile_money_wallet, TZS, M-Pesa provider)
- `petty-cash-mwanza` (petty_cash_float, TZS, float TZS 50,000)
- `site-safe-mwanza` (site_safe, TZS)
- `main-safe-mama-amina` (main_safe, TZS, **tenant scope** — only one for the tenant)
- `tip-till-mwanza` (tip_passthrough_till, TZS)

**08:00 — Open**
- `cash.session.open.request` × 3 (drawer, M-Pesa, petty cash). M-Pesa is continuous-session — opens automatically per day.
- Emits `cash.session.opened.v1` × 3.

**09:30 — Sale (Mixed Tender)**
- Customer buys TZS 45,000 worth; pays TZS 30,000 cash + TZS 15,000 M-Pesa.
- Checkout settles → `checkout.settled.v1`.
- Cash handler emits two `cash.tender.received.v1` events (drawer + M-Pesa), each with `causation_id` to checkout.

**10:00 — Cash Tip**
- Customer leaves TZS 2,000 cash tip → `cash.tip.record.request` → `cash.tip.recorded.v1` to tip-till.

**11:00 — Petty Expense**
- Mzee Hassan buys office paper TZS 5,000 → `cash.expense.record.request {till: petty-cash, category: supplies}` → `cash.expense.recorded.v1` → CN-5-001 #4 auto-journals.

**12:00 — Insurance Premium (Operating Expense)**
- Monthly insurance TZS 75,000 paid from drawer → `cash.expense.record.request {till: drawer, category: insurance}` (N1 — new category) → emits + journals.

**13:00 — Employee Advance (Loan Disbursement)**
- Cashier Asha needs TZS 50,000 advance against salary. HR approves overnight; today disbursement happens.
- `hr.loan.approved.v1` (emitted by HR earlier; Obligation `employee_loan` created).
- Cash handler `disburse_employee_loan` fires now: `cash.expense.record.request {category: employee_loan_disbursement, obligation_ref: <loan>, till: drawer, amount: 50000}`.
- Emits `cash.tender.disbursed.v1` (origin chain: employee_loan).
- CN-5-001 #4 auto-journals: Dr Employee Loan Receivable; Cr Cash.

**14:00 — AR Collection (Customer Pays Outstanding Debt)**
- Customer with TZS 75,000 outstanding from a previous credit sale pays via M-Pesa.
- `cash.collection.record.request {obligation_ref: <AR obligation>, till: mpesa-mwanza, amount: 75000}`.
- Bus validates: Obligation outstanding 75,000 ≥ 75,000 ✓; currency TZS = TZS ✓.
- Emits `cash.collection.received.v1` + Obligation primitive `obligation.settled.v1` (full).
- CN-5-001 auto-journals: Dr Cash (M-Pesa); Cr Accounts Receivable.

**15:30 — AP Payment (Settle Supplier Invoice)**
- Mama Amina pays TZS 200,000 cash to a supplier for a previously-recorded invoice.
- `cash.payment.record.request {obligation_ref: <AP obligation>, till: drawer, amount: 200000}`.
- Emits `cash.payment.disbursed.v1` + Obligation primitive `obligation.settled.v1` (full or partial).
- CN-5-001 auto-journals: Dr Accounts Payable; Cr Cash.

**16:00 — Partial Refund**
- A customer returns part of an earlier purchase. Checkout emits `checkout.refunded.v1` (refund_kind: partial, refund_total: TZS 22,000 in cash).
- Cash handler `issue_refund_cash` (kind: compensation) emits `cash.refund.issued.v1` with `compensates_event_id` to original `cash.tender.received.v1`, causation to refund event.
- Cash outflow from drawer TZS 22,000.

**17:00 — Customer Advance for Custom Order**
- A customer commits to a custom workshop job, pays TZS 100,000 advance.
- `cash.advance_received.record.request {advance_kind: advance_received, customer_party_ref: <cust>, amount: 100000, till: drawer}`.
- Emits `cash.advance_received.recorded.v1` + Obligation primitive: kind=advance_received, outstanding=100000.
- Vertical creates corresponding Inventory reservation per CN-5-003 §9 (N5 cross-engine coordination).

**18:00 — Drop (End-of-Shift)**
- Drawer balance now TZS 588,000 (opening + receipts − expenses − advance − refund). Mzee Hassan drops it to site safe.
- `cash.transfer.between_tills.request {from: drawer, to: site-safe-mwanza, amount: 588000}`.
- Emits `cash.transfer.initiated.v1` + `cash.transfer.received.v1` (instant within site).

**18:30 — Reconcile + Close Drawer**
- Expected drawer = TZS 0 after drop. Counted = TZS 200.
- Variance TZS 200 ≤ auto_adjust_threshold (TZS 500 per pack).
- Bus emits `cash.variance.adjusted.v1` + `cash.adjustment.recorded.v1` (umbrella) + `cash.session.reconciled.v1` + `cash.session.closed.v1`.

**18:30 — Reconcile M-Pesa**
- M-Pesa statement event arrives via Term 7 adapter: `<mm-adapter>.statement.received.v1`.
- Cash handler matches against recorded receipts: TZS 15,000 (sale) + TZS 75,000 (AR collection) = TZS 90,000. Statement shows TZS 90,000.
- Variance 0 → session reconciled + closed automatically.

**Friday Evening — Site Safe → Main Safe Transfer**
- Mzee Hassan drives site safe contents (TZS 4,200,000 accumulated) to main safe.
- `cash.transfer.between_sites.request {from_site, to_site, from_till: site-safe-mwanza, to_till: main-safe, amount: 4200000}`.
- Two-phase: initiated → in-transit → received.

**Friday Late — Bank Deposit**
- Main safe contents to bank: `cash.deposit.initiate.request {till: main-safe, amount: 4200000, bank_account_ref: <bank>}`.
- Emits `cash.deposit.requested.v1` → Term 7 banking adapter handles → emits `<bank-adapter>.deposit.outcome.v1 {confirmed, external_ref: TBANK-DEPOSIT-12345}`.
- Cash handler emits `cash.deposit.completed.v1`.
- CN-5-001 auto-journals: Dr Bank; Cr Cash (main_safe).

**Monthly — Payroll**
- Payroll computes Asha's salary; deducts the TZS 50,000 loan instalment per schedule.
- `payroll.deduction.applied.v1 {employee: Asha, obligation_ref: <loan>, amount: <instalment>}`.
- Cash handler `record_loan_repayment`: Obligation reduces by instalment; emits `cash.payment.disbursed.v1` for net salary.
- CN-5-001 auto-journals: Dr Salary Expense (gross); Cr Loan Receivable (instalment); Cr Cash (net); Cr Tax W/H + Pension W/H.
- Repeat for N months until Obligation outstanding = 0.

### What the Day Demonstrates

- **8 till kinds** in active use across operational scenarios.
- **UI-04 origin chain** maintained for every receipt and disbursement.
- **UI-06 currency** invariant held; no silent mixing.
- **UI-08 obligation bounds** preserved across AR settlement, AP settlement, loan disbursement, loan repayment, advance.
- **Multi-engine choreography** (Checkout → Cash; HR → Cash → Payroll; Cash → Inventory reservation via vertical) operating without orchestrator (CN-5-100 L3 choreography).
- **Reconciliation** with auto-adjust, anomaly detection, and digital reconciliation in one engine.
- **Compensation** (refund) propagating downstream symmetrically.

Mama Amina does not see any of this. She sees her dashboard say "Cash on hand at Mwanza site: TZS X" and "Receivables outstanding: TZS Y." The truth behind that summary is the cohesive event stream above.

---

## 20. Three Whys

### Why does this matter?

Cash is one of the two physical truths every business has (the other is stock). A business that does not know where its money is — across drawers, M-Pesa wallets, safes, customer receivables, supplier payables, employee advances — cannot operate. African SMEs run mixed cash economies (physical + multiple mobile money providers + occasional cards + customer credit + supplier credit + staff loans + customer advances); a single-medium "cash drawer" engine cannot serve them. CN-5-002 makes all those money positions one universal abstraction (till) with one universal session lifecycle, while still recording every business reality — loans choreographed across HR and Payroll, advance deposits coupled with Inventory reservations, multi-currency tills in Zanzibar, tips passed through to staff. The owner's cash truth is emergent from event flow; nothing requires the owner to "remember" to update Cash.

### Why does it belong here (and not in Foundation)?

Foundation provides the Workflow primitive (session lifecycles), the Obligation primitive (debts/credits), the bus (atomic emission), the time authority (business dates), and the anomaly framework (variance detection). Foundation does not say "every business has a main safe and many site safes," "mobile money wallets reconcile via provider statements," "employee loans are HR-approved + Cash-disbursed + Payroll-deducted." Those are universal-engine design choices appropriate for any business that handles money in mixed media. CN-5-002 makes those choices as the universal cash pattern. A specialised engine (e.g., a high-frequency-trading cash engine, a cryptocurrency-only engine) could compose the primitives differently — but that is not what universal cash management is for general business.

### Why this design?

The "till = money position" abstraction collapses what would otherwise be many specialised engines (cash engine, mobile money engine, card engine, safe engine, FX engine) into one cohesive whole, while pack rules + `kind` attribute carry the differentiating semantics. The Cash-records-money-side / Obligation-records-relationship-side split keeps the engine focused; cross-engine choreography handles AR, AP, loans, advances, layby through one primitive. Multi-scope manifest (site default + tenant overrides) matches operational reality without forcing artificial scope partitioning. Pack-driven variance thresholds with tenant override let small operational reality (a 200 TZS drawer variance) be auto-handled while keeping fraud-signal events flagged. The choreography-only architecture (CN-5-100 L3) — no orchestrator of "wait for HR before Cash disburses" — keeps the engine resilient: each engine reacts to events; emergent behaviour is the truth. The whole engine is one cohesive answer to "where is the business's money, in every form, at every site, at every moment?"

---

## 21. Boundaries

| Topic | Lives in |
|-------|----------|
| Workflow primitive (session lifecycle), Obligation primitive (AR/AP/loans/advances) | CN-4-011 |
| Event envelope (causation_id, compensates_event_id) | CN-4-002 |
| Command bus, atomic emission, policy rejection | CN-4-004 |
| Clock protocol (business_date) | CN-4-014 |
| Anomaly detection routing (CN-4-016) for variance escalation | CN-4-016 |
| Checkout settlement (`checkout.settled.v1` carrying tenders; refund compensation pattern) | CN-5-009 |
| Accounting auto-journal mappings (Cash umbrella events to journals) | CN-5-001 |
| Procurement AP creation events (Cash subscribes to settle AP) | CN-5-004 |
| HR loan approval logic and policy | CN-5-005 |
| Payroll computation and deduction events | CN-5-005 |
| Inventory layby/advance fulfillment side (reservation → consumed) | CN-5-003 |
| Subscription patterns + replay semantics + compensation kind | CN-5-100 |
| Scope policy (multi-scope per §6 pattern) | CN-5-101 |
| Cross-engine invariants (UI-04 chain, UI-06 currency, UI-08 obligation bounds) | CN-5-102 |
| Pack content for expense categories, variance thresholds, multi-currency policy, tip tax routing per jurisdiction | Compliance packs authored under CTR-029 (Term 1) |
| Vertical event contracts for layby/advance fulfillment (`<vertical>.advance_consumed.v1`, `<vertical>.layby_release.v1`) | Term 6 via CTR-030 expansion |
| Tenant functional currency + site registries | Term 1 + Term 2 via CTR-027 |
| Payment adapter contract (mobile money, card outcomes) | Term 7 via CTR-006 |
| Banking adapter contract (deposit outcomes) | Term 7 via CTR-031 |
| Reporting templates (cash flow statement, cash position dashboard) | CN-5-006 + pack |
| Statutory cash-related filings (cash declaration, AML thresholds) | Outside BOS (chartered accountant per Charter §1.3); pack content where automatable |
| Bank account balance as a tracked till (in-engine bank mirror) | **v2 future** — see Open Items |

---

## 22. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Pack content for variance thresholds, expense categories, multi-currency, tip tax routing per jurisdiction | Term 1 via CTR-029 expansion | Schema in §7 + §12 + §15 + §16; per-jurisdiction content is Term 1's authorship |
| Vertical event contracts for layby/advance fulfillment | Term 6 via CTR-030 expansion | Per-vertical payload schemas |
| Term 7 banking adapter contract (CTR-031) — full specification | Term 7 via CTR-031 | §11 proposes contract shape; Term 7 finalises |
| `bank_account` till kind for in-engine bank balance tracking | **v2 future** | N3: v1 treats bank as external endpoint via CTR-031 adapter; v2 may add `bank_account` till kind for complete money-position visibility including the bank side. Requires bank-statement-import contract from Term 7. |
| Card-merchant balance till activation (v2-optional for SME default) | Architect + per-tenant policy | Kind 3 in §3; not exercised by default SME deployment |
| Partial refund symmetric mapping per engine (UI-02 carry from CN-5-009 §15) | CN-5-001 + CN-5-003 + Architect | Cash records refund cash side; full UI-02 propagation across Accounting + Inventory is per-engine work |
| Equity flow authorisation policy (who may submit injection/withdrawal) | Term 1 (governance) — possible future CTR-032 | Default: owner role + dual-actor; full governance policy is Term 1's |
| Cross-currency till transfer FX event sourcing | Future FX engine + Term 7 | §15 references; mechanism deferred |
| Mobile money provider statement parsing format (per provider) | Architect + Term 7 | §12 references; each provider has its own statement shape |
| Multi-currency tenant reporting view (showing USD + TZS positions separately) | CN-5-006 | Cash provides data; Reporting provides view |
| Phase 1 doctrine check for UI-04 (cash tender chain integrity) | CN-4-019 + future CTR | Aligns with CN-5-102 §7 Phase 1 ramp-up; CN-5-002 ratification triggers DC addition |
| Phase 1 doctrine check for UI-08 (Obligation bounds preserved across Cash + Procurement + Promotion settlements) | CN-4-019 + future CTR | CN-5-102 §7 Phase 1; arrives with CN-5-004 (Procurement) ratification |
| Cash-flow statement template (per pack reporting_templates) | CN-5-006 + CTR-029 pack content | Cash positions feed the cash-flow statement; template lives in pack |

---

*— End of CN-5-002 —*
