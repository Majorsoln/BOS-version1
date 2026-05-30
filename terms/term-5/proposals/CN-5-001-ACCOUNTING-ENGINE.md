# CN-5-001 — Accounting Engine

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 5 — Universal Engines
> **Status:** For Overseer review.
> **Governing decisions:** Law 5 (compliance is configured, not coded); D-007 #13 (Ledger has no close period — period close is Accounting's); D-009 (freeze doctrine — historical events interpreted under historical pack/standard); D-004 (neutrality — no jurisdiction named in engine code); Charter §1.3 (BOS is **not** a statutory accounting system — management truth only).
> **CTRs:**
> - **OPEN (this doc depends on):** CTR-027 (Term 1/2 — tenant functional currency + site registry); CTR-029 (Term 1 — accounting-standard pack-section schema authorship per jurisdiction); CTR-030 (Term 6 — vertical event payloads carry sufficient data for journal mapping); CTR-016 (Term 1 — platform-scope governance for billing aggregation, reached via Reporting Engine).
> **Glossary:** See `MASTER-GLOSSARY.md` — Universal-engine Invariant (UI), Ledger, Journal, Compensating Event, Site, Scope level.
> **Depends on:** CN-4-002 (envelope incl. `causation_id`, `compensates_event_id`, `pack_version_ref`); CN-4-004 (command bus + atomic emission + policy rejection); CN-4-011 (Ledger primitive — fold, balance invariant); CN-4-014 (clock protocol — business_date, fiscal-zone freeze); CN-4-015 (Compliance DSL — accounting-standard pack section, sandboxed evaluator); CN-5-009 (Checkout — `checkout.settled.v1` is the primary revenue trigger); CN-5-100 (subscription patterns and laws — auto-journal is the canonical P3 / P4 pattern); CN-5-101 (scope policy — Accounting runs at `tenant`); CN-5-102 (engine invariants — UI-01 causation, UI-02 compensation symmetry, UI-05 closed period, UI-06 currency, UI-07 trial balance).
> **Boundaries:** Ledger primitive fold → CN-4-011; Compliance DSL grammar → CN-4-015; specific pack content (rates, accounts per jurisdiction) → compliance packs (Term 1 approval); Checkout settlement mechanics → CN-5-009; Cash/Inventory/Procurement/HR internal logic → CN-5-002/003/004/005; period-close multi-engine choreography → CN-5-104; Reporting templates UX / dual-standard derivation → CN-5-006 (Reporting); statutory filing → outside BOS (chartered accountant per Charter §1.3); FX rate sourcing → Term 7 adapter or future FX engine; depreciation schedule trigger → `system:scheduler` (Architect phase).

---

## 1. What This Doc Defines

CN-5-001 is the first per-engine doc of Term 5 — the Accounting Engine. It implements double-entry bookkeeping as the universal truth layer beneath every business in BOS, regardless of vertical or jurisdiction.

This engine consumes events from every other universal engine and from verticals (Term 6) and produces **journals** — balanced double-entry records — through the bus. It is the **passive truth recorder** of the system: it does not initiate business activity; it observes and journals what other engines emit.

Two non-negotiable framing constraints govern the design:

1. **Standards live in packs, not code** (Law 5). IFRS, TFRS-TZ, US GAAP, and any future country standard are encoded as compliance-pack content (CN-4-015), not as Accounting engine code. The engine reads the pack at command-evaluation time; pack version at emission freezes interpretation forever (D-009).
2. **Management truth only — not statutory filing** (Charter §1.3). BOS Accounting produces audit-ready data complete enough for a chartered accountant to file from. BOS does not file, does not certify, does not replace the accountant.

This doc defines:
- The **engine manifest** (§3).
- The **auto-journal subscription catalog** — every monetary event from elsewhere produces a journal here (§4).
- The **chart of accounts model** — pack-provided base + tenant customisation within standard limits (§5).
- The **accounting-standard pack-section schema** — what each pack must carry for Accounting to operate (§6).
- **Recognition rules** — accrual vs cash-basis as pack-driven configuration (§7).
- **Period close basics** — state, projection, bus policy; the full choreography lives in CN-5-104 (§8).
- **Corrections via compensating journals** — UI-05 + UI-02 application (§9).
- **Opening balances** — one-time historical seeding pattern (§10).
- **Currency** — UI-06 application and FX revaluation pattern (§11).
- The Charter §1.3 **management-truth boundary** — explicit (§12).

This doc does NOT define:
- Ledger primitive fold mechanics (CN-4-011).
- Compliance DSL grammar or sandboxed evaluator (CN-4-015).
- Pack content per jurisdiction (Term 1 authors via CTR-029).
- Other engines' internal logic.
- Multi-engine period-close choreography (CN-5-104).
- Reporting templates UX or dual-standard derivation (CN-5-006).
- Statutory filing or tax submission (outside BOS scope).

---

## 2. The Five Laws

These laws govern every Accounting operation.

| # | Law | Source / why |
|---|-----|--------------|
| **A1** | **Standards in packs, not code.** Chart-of-accounts templates, recognition rules (accrual vs cash-basis, timing), reporting templates (P&L, balance sheet, cash-flow), depreciation conventions, FX treatment, and period calendar all live in the compliance pack (CN-4-015). A new country = a new pack; never an engine code change. | Law 5; D-004 neutrality; D-009 freeze (historical events interpreted under historical standard pack). |
| **A2** | **Auto-journal completeness.** Every event with monetary business effect produces exactly one journal entry (or, when recognition is deferred per the standard, an accrual journal at the appropriate moment). There is no path by which a monetary event lands in the store without being reflected in a journal. | Charter §1.2 (legal defensibility — audit completeness); UI-07 trial balance only holds if no monetary event is missed. |
| **A3** | **Balanced at emission.** Every `accounting.journal.posted.v1` satisfies `sum(debits) = sum(credits)` at emission. The bus rejects unbalanced posts. Per-journal balance inherits from the Ledger primitive's fold (CN-4-011); UI-07 (tenant trial balance) is the system-wide aggregate that follows. | CN-4-011 Ledger; UI-07 inheritance. |
| **A4** | **Tenant scope; site as analytical dimension only.** Accounting runs at `tenant` scope (CN-5-101). When an upstream event carries `site_id`, the journal records it as an **analytical tag**, never as a ledger partition key. The trial balance is tenant-wide; per-site P&L is a Reporting view derived from the tag, not a separate ledger. | D-007 #13 (Ledger has no partitioning beyond what its primitive supports); A1 (standards rarely require per-site ledgers; when they do, the pack expresses it). |
| **A5** | **Corrections via compensating journals; never modify, never re-open.** Errors in posted journals are corrected by appending compensating journal events. Closed periods are inviolable (UI-05); corrections post to the current open period with causation back to the original. There is no `period.reopen` operation. | CN-4-001 immutability; D-009 freeze; UI-05 closed-period inviolability; Charter §1.2. |

---

## 3. Engine Manifest

```yaml
engine_id:        accounting
display_name:     Accounting Engine
version:          1
scope_policy:     tenant                  # A4 — ledger is tenant-wide
primitives_used:
  - ledger                                # CN-4-011 double-entry primitive

commands:
  - id: accounting.chart_of_accounts.configure.request
    scope_ref: tenant
    description: One-time chart configuration at tenant onboarding under jurisdiction's pack
  - id: accounting.account.add.request
    scope_ref: tenant
    description: Tenant adds a customised sub-account within pack-defined limits
  - id: accounting.account.deactivate.request
    scope_ref: tenant
    description: Deactivate a tenant-added sub-account (standard-required accounts cannot be deactivated)
  - id: accounting.opening_balances.set.request
    scope_ref: tenant
    description: One-time historical opening-balance seeding (N3); dual-actor audit; pack defines validation
  - id: accounting.adjustment.post.request
    scope_ref: tenant
    description: Authorised manual journal (current period only — UI-05); restricted role; audited; free-form + optional structured reason_code
  - id: accounting.depreciation.run.request
    scope_ref: tenant
    description: Periodic depreciation run; submitted by system:scheduler at pack-defined frequency
  - id: accounting.period.close.request
    scope_ref: tenant
    description: Initiate period close (CN-5-001 §8 basics; full multi-engine choreography in CN-5-104)

# Note: there is NO direct accounting.journal.post.request command.
# Journals are produced only through auto-journal handlers (system:checkout-accounting-handler
# and similar) or through accounting.adjustment.post.request (authorised manual path).
# This is enforced by the bus (CN-4-004 §3 engine policy).

emits:
  - accounting.chart_of_accounts.configured.v1
  - accounting.account.added.v1
  - accounting.account.deactivated.v1
  - accounting.opening_balances.set.v1
  - accounting.journal.posted.v1            # the primary truth event
  - accounting.journal.reversed.v1          # compensation event (compensates_event_id non-null)
  - accounting.depreciation.posted.v1       # journal kind = depreciation (subset of journal.posted)
  - accounting.period.opened.v1
  - accounting.period.closed.v1             # consumed by UI-05 bus policy across engines

subscribes_to:                              # per CN-4-005 §7, named events only — manifest grows by addition
  # Revenue
  - {event_type: checkout.settled.v1,             version: 1, handler: post_sale_journal,        kind: command_emitting, scope_ref: tenant}
  - {event_type: checkout.refunded.v1,            version: 1, handler: reverse_sale_journal,     kind: compensation,     scope_ref: tenant}
  # Cash flows (non-sale)
  - {event_type: cash.tender.received.v1,         version: 1, handler: post_cash_receipt,        kind: command_emitting, scope_ref: tenant}
  - {event_type: cash.tender.disbursed.v1,        version: 1, handler: post_cash_disbursement,   kind: command_emitting, scope_ref: tenant}
  - {event_type: cash.transfer.between_sites.v1,  version: 1, handler: post_inter_site_transfer, kind: command_emitting, scope_ref: tenant}
  # Procurement (AP cycle)
  - {event_type: procurement.invoice.recorded.v1, version: 1, handler: post_purchase_journal,    kind: command_emitting, scope_ref: tenant}
  - {event_type: procurement.invoice.paid.v1,     version: 1, handler: post_ap_settlement,       kind: command_emitting, scope_ref: tenant}
  - {event_type: procurement.grn.received.v1,     version: 1, handler: post_grn_accrual,         kind: command_emitting, scope_ref: tenant}
  # Inventory
  - {event_type: inventory.adjustment.recorded.v1, version: 1, handler: post_inventory_adjustment, kind: command_emitting, scope_ref: tenant}
  # HR / Payroll
  - {event_type: payroll.computed.v1,             version: 1, handler: post_payroll_accrual,     kind: command_emitting, scope_ref: tenant}
  - {event_type: payroll.paid.v1,                 version: 1, handler: post_payroll_settlement,  kind: command_emitting, scope_ref: tenant}
  # FX (from Term 7 adapter or future FX engine — N1)
  - {event_type: <fx-adapter>.fx.rate.recorded.v1, version: 1, handler: post_fx_revaluation,     kind: command_emitting, scope_ref: tenant}
  # Promotion cost-share
  - {event_type: promotion.cost_share.recorded.v1, version: 1, handler: post_promotion_share,    kind: command_emitting, scope_ref: tenant}

requires:                                   # mandatory subset
  - checkout.settled.v1                     # primary revenue source; engine cannot serve a transacting business without it
```

---

## 4. Auto-Journal Subscription Catalog

Each subscription handler produces exactly one journal entry (A2) by submitting `accounting.adjustment.post.request`'s **system-only variant** — the bus's auto-journal path that bypasses the authorised-role restriction (system actor = the handler's principal) but applies all other policies including A3 balance check and UI-05 / UI-06 / pack-recognition rules.

The account mappings below are **illustrative**. Actual mappings live in the compliance pack's `recognition_rules` (§6 / §7) and are resolved by the sandboxed evaluator (CN-4-015 §4).

| # | Event subscribed | Journal kind | Dr (illustrative) | Cr (illustrative) |
|---|------------------|--------------|-------------------|-------------------|
| 1 | `checkout.settled.v1` | Revenue + tax + COGS | Cash / AR (per tender mix); COGS | Revenue; Tax Payable; Inventory |
| 2 | `checkout.refunded.v1` (P4 compensation) | Revenue reversal (partial or full) | Reverse of #1 per `refund_kind` | Reverse of #1 |
| 3 | `cash.tender.received.v1` (non-sale, e.g., owner injection) | Cash receipt | Cash | Equity Injection / Other Income |
| 4 | `cash.tender.disbursed.v1` (non-AP, e.g., petty cash spend) | Cash disbursement | Expense (per category) | Cash |
| 5 | `cash.transfer.between_sites.v1` | Inter-site cash transfer | Cash (analytical tag: to-site) | Cash (analytical tag: from-site) |
| 6 | `procurement.invoice.recorded.v1` | AP creation | Inventory / Expense; Input Tax | AP |
| 7 | `procurement.invoice.paid.v1` | AP settlement | AP | Cash |
| 8 | `procurement.grn.received.v1` (when invoice not yet) | GRN accrual | Inventory | GR/IR (Goods-Received-Not-Invoiced) |
| 9 | `inventory.adjustment.recorded.v1` | Stock write-up / write-down | Inventory or Adjustment | Adjustment or Inventory |
| 10 | `payroll.computed.v1` | Payroll accrual | Salary Expense | Salary Payable; Tax W/H; Pension W/H |
| 11 | `payroll.paid.v1` | Payroll settlement | Salary Payable | Cash |
| 12 | `accounting.depreciation.run.request` (system:scheduler) → `accounting.depreciation.posted.v1` | Periodic depreciation | Depreciation Expense | Accumulated Depreciation |
| 13 | `<fx-adapter>.fx.rate.recorded.v1` | FX revaluation (pack-defined frequency) | FX Loss or FX Asset | FX Gain or FX Liability |

### Inventory Transfer Between Sites — No Journal in v1

`inventory.movement.transferred.v1` (between sites within the same legal entity) produces **no journal** in v1 — both sides of the transfer are the same Inventory account; only analytical tags change. Standards that require inter-branch P&L attribution can express this in the pack via `recognition_rules` later; CN-5-001 v1 does not subscribe to inventory transfers for journal purposes.

### Adding New Auto-Journal Subscriptions

When a Term 6 vertical or a new universal engine produces a new monetary event type, CN-5-001's manifest grows by one subscription entry (per CN-4-005 §7 — manifests grow by addition). The handler is registered; the pack must declare the recognition mapping for the new event type before the subscription becomes effective. Pack rotation governance lives with Term 1.

### Causation Linkage on Every Journal (UI-01)

Every emitted `accounting.journal.posted.v1` carries `causation_id = <event_id of the triggering source event>` (CN-5-100 §5). This makes any journal traceable back to the business event that produced it — a regulator or auditor walking from a tax filing → P&L → journal → source event reconstructs the full provenance chain.

---

## 5. Chart of Accounts Model

### Account Record

| Field | Source |
|-------|--------|
| `account_code` | Pack-defined for standard-required accounts; tenant-defined (within pack-defined ranges) for customisations |
| `account_name` | Pack-defined for required; tenant-defined for customisations |
| `account_type` | Enum: `asset` / `liability` / `equity` / `revenue` / `expense` (the five universal categories) |
| `parent_account_code` | For hierarchical roll-ups (e.g., `1110 Cash on Hand` parent = `1100 Cash & Equivalents`) |
| `is_standard_required` | Boolean — true if from pack; cannot be deactivated by tenant |
| `tenant_added` | Boolean — true if added by tenant within pack limits |
| `active_status` | `active` / `deactivated` |
| `analytical_dimensions` | Optional list — e.g., `[site_id, cost_centre]` — A4 dimensional analysis without ledger partitioning |
| `currency_constraint` | Optional — defaults to tenant functional currency (UI-06); pack may declare specific accounts as multi-currency |

### Configuration Lifecycle

| Phase | What happens |
|-------|--------------|
| **Onboarding** | Term 2 onboarding binds tenant to jurisdiction (CTR-027). Pack for that jurisdiction supplies the base chart. Cashier or onboarding role submits `accounting.chart_of_accounts.configure.request`; bus validates against pack; engine emits `accounting.chart_of_accounts.configured.v1` carrying the base chart frozen at this pack version. |
| **Customisation** | Tenant adds sub-accounts via `accounting.account.add.request` within pack-defined limits (which parent ranges, what types, naming patterns). Bus rejects accounts outside limits. |
| **Standard upgrade** | When pack version changes (e.g., TFRS-TZ updated for new fiscal year), new required accounts arrive **additively** — existing accounts persist with all historical data. Removed-from-standard accounts become `deactivated` on the chart (data retained per CN-4-001 immutability); their balances are frozen at the moment of deactivation. |
| **Deactivation** | Tenant-added sub-accounts may be deactivated via `accounting.account.deactivate.request` (standard-required accounts cannot). Deactivated accounts retain their balance and history; new journals cannot post to them. |

### Why Configurable Within Limits

A tenant adding accounts freely produces a chart that does not roll up into the standard's reporting templates (P&L / balance sheet won't aggregate correctly). A tenant restricted to pack-prescribed accounts cannot model their business (e.g., "rent expense by branch"). The middle ground: the pack defines the parent structure, account types, and ranges; the tenant adds sub-accounts within those constraints. The reporting templates remain valid because every tenant account rolls up to a pack-defined parent.

---

## 6. Accounting-Standard Pack-Section Schema

CN-5-001 defines the **schema** of the accounting section that every jurisdiction pack must carry. CN-4-015 provides the DSL grammar; Term 1 authors and approves the content per jurisdiction (CTR-029).

```yaml
# Section in compliance pack
accounting:
  standard_id:           IFRS | TFRS-TZ | US-GAAP | ...     # the named accounting standard
  standard_version:      <standard-specific version reference>

  # 6.1 — Base chart of accounts (§5)
  chart_of_accounts_base:
    - {code: "1100", name: "Cash & Equivalents",  type: asset,     parent: null, required: true,
        tenant_subaccount_range: "1110-1199"}
    - {code: "1200", name: "Accounts Receivable", type: asset,     parent: null, required: true}
    - {code: "1300", name: "Inventory",           type: asset,     parent: null, required: true}
    - {code: "2100", name: "Accounts Payable",    type: liability, parent: null, required: true}
    - {code: "2300", name: "Tax Payable",         type: liability, parent: null, required: true}
    - {code: "3100", name: "Owners Equity",       type: equity,    parent: null, required: true}
    - {code: "4000", name: "Revenue",             type: revenue,   parent: null, required: true}
    - {code: "5000", name: "Cost of Sales",       type: expense,   parent: null, required: true}
    # … hundreds more per standard

  # 6.2 — Recognition rules (§7)
  recognition_rules:
    revenue:
      method:            accrual | cash_basis | hybrid
      tenant_election:   {allowed: true, eligibility: "annual_revenue < threshold"}   # if pack allows tenant to elect
      timing:            at_invoice | at_delivery | at_settlement | at_completion
      mapping:
        - {event_type: "checkout.settled.v1",
            triggers_journal: true,
            entries:
              - {dr: "1100", basis: "tenders_total_cash_portion"}
              - {dr: "1200", basis: "tenders_total_credit_portion"}
              - {dr: "5000", basis: "cogs_from_line_items"}
              - {cr: "4000", basis: "pre_tax_total"}
              - {cr: "2300", basis: "tax_total"}
              - {cr: "1300", basis: "cogs_from_line_items"}}
    expense:
      method:            accrual | cash_basis
      mapping:
        - {event_type: "procurement.invoice.recorded.v1",
            triggers_journal: true,
            entries:
              - {dr: "1300", basis: "amount_excl_tax_inventory_portion"}
              - {dr: "1310", basis: "input_tax_amount"}
              - {cr: "2100", basis: "amount_incl_tax"}}
    payroll:
      mapping:
        - {event_type: "payroll.computed.v1",
            triggers_journal: true,
            entries: [...]}

  # 6.3 — Reporting templates (consumed by CN-5-006 Reporting Engine)
  reporting_templates:
    profit_and_loss:
      sections:
        - {name: "Revenue",           account_groupings: ["4000-4999"]}
        - {name: "Cost of Sales",     account_groupings: ["5000-5099"]}
        - {name: "Gross Profit",      formula: "Revenue - Cost of Sales"}
        - {name: "Operating Expense", account_groupings: ["6000-6999"]}
        - {name: "Operating Profit",  formula: "Gross Profit - Operating Expense"}
    balance_sheet:
      sections: [...]
    cash_flow_statement:
      sections: [...]

  # 6.4 — Depreciation
  depreciation:
    methods_allowed:   [straight_line, declining_balance, units_of_production]
    asset_class_defaults:
      buildings:  {method: straight_line,    useful_life_years: 40}
      vehicles:   {method: declining_balance, rate: 25}
      machinery:  {method: straight_line,    useful_life_years: 10}
    run_frequency:   monthly | quarterly | annual

  # 6.5 — Forex
  forex:
    gain_loss_recognition:   profit_and_loss | other_comprehensive_income
    revaluation_frequency:   monthly | quarterly | annual
    rate_source:             <adapter-ref or fixing-method>

  # 6.6 — Period calendar (N2)
  period_calendar:
    fiscal_year_start:       {month: 7, day: 1}             # e.g., July 1 for Tanzania, January 1 for IFRS-default
    reporting_frequencies:   [monthly, quarterly, annual]   # which closes are mandatory
    close_lag_days:          15                              # days after period end by which close must complete
    period_naming_format:    "YYYY-MM"                       # how a period is identified in event payloads
```

CN-5-001 owns the field names, types, and semantics. Per-jurisdiction content (the actual accounts, rates, templates) is authored and approved by Term 1 under CTR-029. Pack version at every event emission (D-009) freezes which version of these rules apply.

---

## 7. Recognition Rules — Accrual vs Cash-Basis

The pack declares the default recognition method per standard. Most modern standards default to **accrual** (revenue recognised when earned, expense when incurred). Many jurisdictions allow small businesses to elect **cash-basis** (revenue/expense recognised when cash moves).

| Method | Revenue journal at | Expense journal at |
|--------|--------------------|--------------------|
| **Accrual** | `checkout.settled.v1` (delivery/completion per pack timing) | `procurement.invoice.recorded.v1` (invoice receipt) |
| **Cash-basis** | `cash.tender.received.v1` (cash actually received) | `procurement.invoice.paid.v1` (cash actually paid) |
| **Hybrid** | Per pack `mapping.timing` per event type | Per pack `mapping.timing` per event type |

### Tenant Election

Some packs allow tenants to elect cash-basis if eligible (e.g., annual revenue below a threshold). The election is recorded as a tenant property (similar to fiscal zone, CN-4-014) and is itself a doctrine-bound choice — once elected for a tax year, it cannot be changed mid-year without standards-defined re-statement. The pack declares eligibility criteria; Term 1 governance approves elections.

### Why It Lives in the Pack, Not Code

Two tenants in the same country may use different methods (one accrual, one cash-basis). The pack expresses the menu of allowed methods; the tenant's election is a property; the recognition rules are evaluated by the sandboxed evaluator at every event. CN-5-001 code never knows whether "this tenant is accrual" — it asks the pack at every journal handler invocation. This is what Law 5 ("compliance is configured, not coded") means in practice.

---

## 8. Period Close — Basics

This section establishes the per-tenant **state** of accounting periods and the **bus policy** that enforces closed-period inviolability (UI-05). The multi-engine **choreography** of close (Cash session reconciliation, Inventory cut-off, HR payroll cut-off, etc.) lives in CN-5-104.

### Period State Events

| Event | Carries | Effect |
|-------|---------|--------|
| `accounting.period.opened.v1` | `{period, opened_at, opened_by}` | Period state projection records the period as `open`. Effective-date-bearing events with `effective_date ∈ period` are accepted. |
| `accounting.period.closed.v1` | `{period, closed_at, closed_by, summary: {pre-close trial balance, totals}}` | Period state projection records the period as `closed`. UI-05 bus policy now rejects any command attempting to emit an effective-date-bearing event in this period. |

Periods are identified per the pack's `period_naming_format` (typically `YYYY-MM` for monthly).

### Period State Projection

A tenant-scoped projection (CN-4-010) tracks each period's status: `open` / `closing` (during the CN-5-104 choreography) / `closed`. The projection is rebuilt from `accounting.period.opened.v1` and `accounting.period.closed.v1` events. UI-05's bus policy reads this projection at command time.

### UI-05 Enforcement at the Bus

When any command across BOS (a sale, a payroll run, an inventory adjustment) carries an `effective_date`, the bus consults Accounting's period state projection:
- If `effective_date` falls in an `open` period for the tenant → proceed.
- If `effective_date` falls in a `closed` period → reject:
  ```
  rejected_by_policy:  UI-05.closed_period_inviolable
  rejection_reason:    "Period <period> is closed for this tenant; corrections must post to the current open period"
  ```

This is a bus policy that **every effective-date-bearing engine** inherits — not Accounting-specific code in other engines. Accounting publishes the projection; the bus reads it (CN-4-004 §3 engine policy).

### Close Initiation

`accounting.period.close.request` is the entry point. CN-5-001 handles the basics: validate that the period is currently `open`; transition to `closing`; emit a "close started" marker; await the multi-engine choreography (CN-5-104) to complete; then emit `accounting.period.closed.v1`. The choreography itself — which engines must signal ready, how partial closes are handled, what blocks a close — is CN-5-104's concern.

### No Re-Open

There is no `accounting.period.reopen.request`. Once `accounting.period.closed.v1` is emitted, that period is closed forever. Any correction post-dates to the current open period with causation back to the closed-period event (A5). This is the operational meaning of D-009 freeze at the period boundary.

---

## 9. Corrections via Compensating Journals

### Pattern

When an erroneous journal is identified after emission, the correction is a **compensating journal event** (CN-4-001 §3.1):

```
event_type:            accounting.journal.reversed.v1
compensates_event_id:  <event_id of the erroneous journal — UI-02 link>    (CN-4-002)
causation_id:          <event_id of the triggering correction command>     (UI-01 link — N4)
payload:
  reversed_journal_ref: <reference to original journal>
  reversal_entries:     [<symmetric reverse of original entries; Dr↔Cr>]
  reason_ref:           <free-form text + optional structured reason_code>
  business_date:        <current open period — UI-05>
```

### N4 — Two Distinct Envelope Links

The compensation event carries **two different references** that must not be conflated:

| Field | Refers to | Why it exists |
|-------|-----------|----------------|
| `causation_id` | The triggering correction command's derivative event | UI-01 — every derivative event names its trigger so audit (CN-4-008) can walk the choreography forward. |
| `compensates_event_id` | The original erroneous journal being reversed | UI-02 — structurally identifies this as a compensation (vs a forward journal), routes it to `kind: compensation` subscribers in downstream engines (Reporting, Cash analytics), and enables compensation-symmetry analysis. |

A journal that **causes** a follow-up journal carries causation forward. A journal that **reverses** a prior journal carries compensation back. Both fields are populated independently and have different audit semantics.

### Closed-Period Correction Flow

When the original journal is in a closed period:
- The compensating journal's `business_date` is in the **current open period** (A5; UI-05).
- The compensating journal's `compensates_event_id` walks back to the closed-period original.
- Reporting reads both events and shows "Period X figures: as originally closed" plus "Subsequent correction posted in Period Y referring back to Period X."
- The original period's trial balance, as it stood at close, **does not change**. The correction is a forward-period event with an explicit backward reference.

### Why Not Re-Open

A re-opened period would mean past statements (P&L, balance sheet, tax filings derived from them) become retroactively wrong. The forward-correction-with-causation pattern preserves the historical truth (what was closed is what was closed) while making the correction fully traceable. This is the freeze doctrine at the operational layer.

---

## 10. Opening Balances (N3)

When a tenant onboards to BOS from a prior system, they have historical balances — accounts payable owed to suppliers, inventory on shelves, cash on hand, equity. These are not the result of business events that BOS observed; they are **historical state** that must be seeded into the chart at onboarding.

### The Command

```
accounting.opening_balances.set.request
  scope:           tenant
  required_role:   onboarding_accountant (per Term 2 onboarding flow)
  audit:           dual-actor (the onboarding actor + the tenant accountant)
  payload:
    as_of_date:               <opening date, typically tenant's fiscal year start>
    balances:                 [{account_code, debit_or_credit, amount, currency}]
    supporting_document_refs: [<references to source statements>]
```

### Validation

The bus invokes the pack's validation rules:
- Balances must satisfy `sum(debit_balances) = sum(credit_balances)` (A3 inherited at the seeding moment).
- All `account_code`s must exist in the configured chart.
- Currency must match tenant functional currency (UI-06).
- Pack-specific constraints (e.g., "Equity opening must equal Assets − Liabilities") are evaluated.

### Why Not Through `adjustment.post.request`

Manual adjustment is for **current-period corrections** (A5 / UI-05). Opening balances are not corrections — they are historical seed data. Treating them as adjustments would obscure provenance and conflate two different audit categories. The dedicated command makes the seeding visible as a one-time onboarding event with its own audit trail.

### One-Time

After `accounting.opening_balances.set.v1` is emitted for a tenant, the command is rejected on subsequent submissions. Historical re-seeding (rare, e.g., onboarding error discovered post-launch) is a governance escalation, not a routine operation.

---

## 11. Currency & UI-06

UI-06 (CN-5-102) requires every primary financial event to use the tenant's functional currency. Accounting enforces this at:

| Boundary | Check |
|----------|-------|
| `accounting.account.add.request` | New account's `currency_constraint` (if specified) must equal tenant functional currency, unless the pack declares the account as multi-currency permitted. |
| Every auto-journal handler invocation | The source event's currency is read; if it does not match tenant functional currency, the handler does **not** emit a forward journal — it requires a paired FX event from the same source choreography (the source engine must emit `<adapter>.fx.rate.recorded.v1` or equivalent FX bridging event before the journal can post in functional currency). |
| `accounting.adjustment.post.request` | Adjustment currency must equal functional unless paired with explicit FX context. |
| `accounting.opening_balances.set.request` | All balances must be in functional currency at the seeding moment. |

### FX Revaluation Pattern

When the pack's `forex.revaluation_frequency` triggers (e.g., monthly), the system scheduler emits an instruction that brings the latest FX rate (sourced via Term 7 adapter or future FX engine) into Accounting's auto-journal flow. The `post_fx_revaluation` handler computes gain/loss on multi-currency accounts (rare in v1 — most BOS tenants are single-currency) and emits the revaluation journal per pack's `forex.gain_loss_recognition` directive (P&L or OCI).

Single-currency tenants experience no FX revaluation activity. Multi-currency support is an Open Item (§15) — the v1 model supports the bridging pattern in principle; full multi-currency reporting is deferred.

---

## 12. Charter §1.3 — Management Truth, Not Statutory Filing

This boundary is explicit. CN-5-001 produces:

| What BOS Accounting does | What it does NOT do |
|--------------------------|---------------------|
| Records every business event as a journal (A2 completeness) | File statutory returns to tax authorities |
| Maintains a balanced trial balance per tenant (A3 + UI-07) | Certify financial statements |
| Produces audit-ready data with full causation lineage (UI-01) | Replace the chartered accountant |
| Computes tax amounts on transactions per pack rules | Submit those amounts to tax APIs (Term 7 may expose an export adapter; submission is the accountant's act) |
| Generates P&L, balance sheet, cash flow per pack templates (via CN-5-006 Reporting) | Sign or stamp those statements |
| Honours freeze doctrine — historical events stay under historical rules (D-009) | Re-open closed periods or amend filed statements |

The chartered accountant — a real human professional engaged by the tenant — uses BOS data for filings, audits, and management reporting. BOS guarantees the data is complete, balanced, auditable, and standard-compliant **as configured by the approved pack**. The accountant relies on this; BOS does not replace them.

No `accounting.return.filed.v1` event exists. No tax-authority API submission happens from this engine. If a tenant or their accountant requires an export in a specific format for filing, that is a Reporting Engine (CN-5-006) export or a Term 7 adapter — never a CN-5-001 responsibility.

---

## 13. Example — Mama Amina's Year (FY 2026-07 to 2027-06)

*Mama Amina's duka serves as illustrative context, per D-004 #4. The example shows the engine contract operating across a year; it is not a UX description.*

Mama Amina operates `mama-amina-duka`, jurisdiction Tanzania, fiscal year per pack `tz-compliance-2026.07` (TFRS-TZ standard, fiscal year starts July 1). Functional currency: TZS. Two sites: Mwanza (`mama-amina-mwanza-001`) and Mbeya (`mama-amina-mbeya-001`).

### Onboarding (June 2026)

Term 2 onboarding completes. The pack supplies the TFRS-TZ base chart (147 required accounts). Mama Amina's accountant adds 8 sub-accounts (per-site rent expense, per-site utilities). Onboarding accountant submits `accounting.opening_balances.set.request` with TZS 2.4M cash, TZS 1.8M inventory, TZS 0.6M AP, TZS 3.6M equity — balanced. Bus accepts; `accounting.opening_balances.set.v1` emitted with dual-actor audit. Pack version: `tz-compliance-2026.07`.

### Operational Year (July 2026 – June 2027)

Daily events flow through the auto-journal handlers:

- **Every sale**: `checkout.settled.v1` → `post_sale_journal` → `accounting.journal.posted.v1` (Dr Cash/AR/Mobile Money receivable, Dr COGS, Cr Revenue, Cr Tax Payable 18% per TFRS-TZ pack, Cr Inventory). Per-site analytical tag on each line. Causation back to the checkout event.
- **Supplier invoices**: `procurement.invoice.recorded.v1` → `post_purchase_journal` → Dr Inventory + Dr Input Tax + Cr AP. Causation chain to procurement event.
- **Monthly payroll**: `payroll.computed.v1` → `post_payroll_accrual` → Dr Salary Expense + Cr Payable + Cr WHT + Cr Pension.
- **Monthly depreciation**: `system:scheduler` submits `accounting.depreciation.run.request` on the first business day of each month per pack frequency → `accounting.depreciation.posted.v1` for fixed assets per asset class defaults.
- **Inter-site cash transfer**: `cash.transfer.between_sites.v1` → `post_inter_site_transfer` → Dr Cash (analytical tag: to-site) + Cr Cash (analytical tag: from-site). Single Cash account; site-level disaggregation via tags (A4).

### Monthly Close (e.g., October 31, 2026)

Mzee Hassan (Mama Amina's bookkeeper) submits `accounting.period.close.request {period: 2026-10}`. CN-5-104 choreography fan-in waits for `cash.period.ready.v1`, `inventory.period.ready.v1`, `payroll.period.ready.v1`. When all signals received, CN-5-001 verifies UI-07 (trial balance balanced), then emits `accounting.period.closed.v1 {period: 2026-10, summary: {...}}`. UI-05 bus policy now rejects any further events with `effective_date ∈ 2026-10`.

### Correction in November

On November 5, Mama Amina discovers an October 28 sale was double-counted. The correction cannot post in October (UI-05). Mzee Hassan submits `accounting.adjustment.post.request {business_date: 2026-11-05, reason_ref: "duplicate sale removal", reason_code: "ERR-DUP-001"}` with the reversing entries. Bus validates: November is open; role has authorisation; entries balance. Emits `accounting.journal.reversed.v1` with `compensates_event_id = <October duplicate sale journal id>` and `causation_id = <Mzee Hassan's adjustment command derivative id>`. UI-02 propagation: downstream `kind: compensation` subscribers (Reporting, perhaps Cash analytics) update their views. **October's trial balance, as closed, does not change.** A November-period entry corrects the cumulative position with full causation back to October.

### Year-End (June 30, 2027)

The CN-5-104 annual-close choreography runs (heavier than monthly — includes FX revaluation per pack frequency, asset class depreciation true-ups). `accounting.period.closed.v1 {period: 2026-07-to-2027-06, annual: true}` is emitted. Reporting Engine (CN-5-006) produces the annual P&L and balance sheet from the TFRS-TZ reporting templates.

### The Statutory Filing — Outside BOS

Mzee Hassan exports the annual statements (CN-5-006 export). He prepares Tanzania's statutory return offline using his professional certification. He files with TRA. **BOS did not file.** Charter §1.3 boundary held.

### A Pack Upgrade Mid-Year

In April 2027, Tanzania publishes an updated tax rate (VAT changes 18% → 19% effective May 1). Term 1 approves pack `tz-compliance-2027.05`. Pack rotation event is emitted. Events emitted **before** May 1 reference `tz-compliance-2026.07` and compute VAT at 18%; events emitted **on or after** May 1 reference `tz-compliance-2027.05` and compute at 19% (D-009 freeze). The chart of accounts remains the same — only the rate value in `recognition_rules` changed. Mama Amina's bookkeeper sees no operational change; the system simply uses the new rate for new transactions.

---

## 14. Three Whys

### Why does this matter?

Every business is, at the end of the day, its books. A business that does not know what it owns, owes, earned, and spent does not know whether it is alive. CN-5-001 makes the books a structural emergent property of every business event BOS observes — not a separate manual process the owner has to remember. Auto-journal completeness (A2) means a sale, a purchase, a payroll run, a refund all land in the ledger automatically with full audit lineage. Standards-in-packs (A1) means a business in Tanzania uses TFRS-TZ, a business in the US uses US GAAP, and adding a new country never touches Accounting code. Closed-period inviolability (A5 / UI-05) means past truth stays past truth — the books a tenant filed taxes against do not silently change.

### Why does it belong here (and not in Foundation)?

Foundation provides the Ledger primitive (CN-4-011), the bus (CN-4-004), the compliance DSL (CN-4-015), the time authority (CN-4-014). Foundation does not say "every sale produces a journal" or "VAT is computed at the rate in the active pack" — those are universal-engine design decisions appropriate for any business of any size in any jurisdiction. CN-5-001 makes those decisions as the universal Accounting pattern. A different engine (a specialised tax engine, a forecasting engine) could compose Ledger differently — but that is not what universal management-truth accounting is.

### Why this design?

Auto-journal subscription (P3 command-emitting) inherits the system's existing event flow — Accounting did not need a new orchestration layer; it consumes what other engines already emit. Standards in packs operationalise Law 5 — adding IFRS, TFRS-TZ, US GAAP is content authorship, not engineering. Single standard per tenant (with dual-reporting via CN-5-006 for tenants that need it) avoids dual-ledger complexity while supporting the realistic case. Forward-correction-with-causation preserves historical truth (no re-open) while allowing operational fixes. Charter §1.3 explicit boundary keeps Accounting honest — BOS is the data layer for the accountant, not a replacement for them. The whole engine is one cohesive answer to "how does any business, in any country, keep its books?"

---

## 15. Boundaries

| Topic | Lives in |
|-------|----------|
| Ledger primitive (fold, balance, scope-agnostic) | CN-4-011 |
| Compliance DSL grammar + sandboxed evaluator | CN-4-015 |
| Pack content per jurisdiction (the actual TFRS-TZ / IFRS / GAAP rules) | Compliance packs authored under CTR-029 (Term 1) |
| Event envelope (`causation_id`, `compensates_event_id`, `pack_version_ref`) | CN-4-002 |
| Command bus, atomic emission, rejection policy | CN-4-004 |
| Clock protocol (business_date, fiscal-zone freeze) | CN-4-014 |
| Checkout settlement event (the primary revenue trigger) | CN-5-009 |
| Subscription patterns + replay semantics + causation linkage | CN-5-100 |
| Scope policy (Accounting runs at tenant) | CN-5-101 |
| Cross-engine invariants (UI-01/02/05/06/07) | CN-5-102 |
| Multi-engine period-close choreography | CN-5-104 |
| Cash Engine internal logic (sessions, drawers) | CN-5-002 |
| Inventory Engine internal logic (movements, FIFO/LIFO, lots) | CN-5-003 |
| Procurement Engine internal logic (PO/GRN/invoice/payment) | CN-5-004 |
| HR / Payroll Engine internal logic | CN-5-005 |
| Reporting templates UX, dual-standard derivation, exports | CN-5-006 |
| Vertical bill construction (Term 6) | per-vertical docs + CTR-030 |
| FX rate sourcing (Term 7 adapter or future FX engine) | Term 7 (CTR-006 vicinity) or a future engine doc |
| Depreciation scheduler trigger | `system:scheduler` (Architect phase) |
| Tenant functional currency + site registries | Term 1 (governance) + Term 2 (population) via CTR-027 |
| Statutory filing / tax submission / certification | Outside BOS (chartered accountant per Charter §1.3) |
| Document export (P&L, balance sheet) for accountant use | CN-5-006 + Term 7 export adapter |

---

## 16. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Per-jurisdiction pack content (TFRS-TZ, IFRS, US GAAP, …) | Term 1 via CTR-029 | Schema is fixed here (§6); content is per-country authorship |
| Vertical payload conformance for journal mapping (cost basis, item refs, etc.) | Term 6 via CTR-030 | Auto-journal can only work if upstream events carry the needed fields |
| FX rate sourcing engine vs Term 7 adapter | Future decision | v1 references `<fx-adapter>.fx.rate.recorded.v1` generically; the source mechanism is for Architect + Term 7 |
| Full multi-currency reporting (multi-currency tenants) | Future — Term 5 v2 + CN-5-006 | v1 supports the bridging pattern in principle; full multi-currency P&L deferred |
| Manual adjustment role authorisation (which tenant roles may post adjustments) | Term 1 (governance) + Architect | Per-tenant role policy |
| Opening-balances re-seeding governance (rare, post-onboarding correction) | Term 1 + future decision | One-time by default; re-seeding is a governance escalation |
| Standard tenant-election lifecycle (e.g., cash-basis election; restating on change) | Term 1 + pack content | Pack declares eligibility; lifecycle (annual lock, restatement on change) is Term 1 |
| Doctrine check for A2 auto-journal completeness | CN-4-019 + future CTR | Phase 1 invariant per CN-5-102 §7; CTR to add a DC arrives with CN-5-001 ratification |
| Dual reporting (IFRS view + statutory view for the same tenant) | CN-5-006 (Reporting) + Term 1 | v1 = single ledger; Reporting derives alternative views; full dual-ledger is a future v2 question |
| Audit-completeness DC ("every state-changing action produces a journal where monetary") | CN-4-019 living catalog | Already noted in CN-4-019 §9 open items; CN-5-001 ratification triggers the addition |

---

*— End of CN-5-001 —*
