# CN-5-010 — AI Advisors Wiring

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 5 — Universal Engines
> **Status:** For Overseer review.
> **Governing decisions:** D-002A (Runtime Advisor Framework); D-005 (AI Mode dashboard pattern per role); D-009 (freeze doctrine — advisor versioning); Law 3 (AI is advisory only; never executes); Charter §1.3 (management truth boundary); UI-01 (causation chain on suggestions).
> **CTRs:**
> - **OPEN (this doc references for resolution):** CTR-039 (Term 5 → Term 3 — UI surface for advisor suggestions); CTR-040 (Term 5 → Term 7 — channel routing for advisor digest/alerts); CTR-041 (Term 5 → Term 4 — advisor_id registry mechanism parallel to CN-4-020).
> - **OPEN expansions at merge:** CTR-014 (cost governance + per-advisor ceilings + Phase 1 activation thresholds); CTR-015 (AI Mode dashboard pattern advisor-suggestion-feed primitive).
> - **Foundation extension (Term 4 living-catalog authority — not CTR):** `kernel.advisor.suggestion.dropped.v1` per N4 (real-time latency-exceeded path) — Term 4 confirms via CN-4-013 living catalog upon CN-5-010 ratification.
> - **No new CTRs filed by this doc.**
> **Glossary:** See `MASTER-GLOSSARY.md` — Advisor, Audience, Scope level, Decision Journal, UI-NN.
> **Depends on:** CN-4-006 (tenant + site scope); CN-4-007 (principal types incl. advisor); CN-4-013 (Decision Journal — `kernel.advisor.suggestion.recorded.v1`, `kernel.advisor.decision.recorded.v1`); CN-4-018 (Snapshot Storage); CN-4-020 (Engine registry — pattern for CTR-041 advisor registry); CN-4-022 (Advisor Framework — `{audience, scope, model}` contract); CN-5-001–CN-5-007, CN-5-009 (per-engine projections and Documents Advisors read); CN-5-006 (Reporting projections — primary advisor read surface, N3 catalogue states); CN-5-100 (subscription patterns); CN-5-101 (scope policy); CN-5-102 (UI invariants — UI-01 suggestion causation; UI-04 if checkout-advisor real-time); CN-5-103 (event glossary — `kernel.advisor.*` per G2 legitimate Foundation).
> **Boundaries:** Advisor Framework contract (audience / scope / model plug-in) → CN-4-022; Decision Journal fields and mechanism → CN-4-013; engine registry pattern → CN-4-020 (CTR-041 analog); UI for advisor suggestions → Term 3 (CTR-039); channel delivery → Term 7 (CTR-040 + CTR-021 consent); per-advisor cost ceiling + model-tier elections → Term 1 (CTR-014 expansion); pack content for advisor cutoffs, thresholds, schedules → Term 1 (CTR-014 + new pack hooks); Developer-AI (D-002B; build-time, outside kernel) → CN-4-023 (NOT this doc).

---

## 1. Purpose & Boundary

CN-5-010 **wires concrete Advisors** onto the CN-4-022 Foundation Advisor Framework, completing the Term 5 runtime-AI scope (D-002A). Nine advisors are catalogued — one per universal engine plus a cross-engine BI advisor plus a cross-engine tax-advisor (added by CN-5-105 amendment). Each advisor is defined by the CN-4-022 3-tuple `{audience, data_scope, model}` and routes its suggestions through CN-4-013 Decision Journal events. Law 3 (advisory only) is structural: advisors never emit business events; they propose, and humans act.

This doc finalises the runtime-AI layer of Term 5. After it, every engine has a concrete advisor wired; suggestions land in the Decision Journal; humans act through normal command-bus paths; the audit chain (UI-01) connects original observation → suggestion → human decision → command → resulting events.

### Scope (in)

- 8 concrete Advisors (5 Phase 0 immediately activatable; 3 Phase 1 conditioned on pack thresholds)
- Per-advisor wiring (audience, data scope, trigger, suggestion shape, audience routing, activation, model)
- Decision Journal integration per CN-4-013 (suggestion + decision events; N1 — `recommended_command_draft` is data, never autonomous command)
- Activation governance (pack-driven + tenant opt-in + cost ceiling per CTR-014)
- Suggestion delivery (UI per CTR-039 + channels per CTR-040 + consent per CTR-021)
- Evidence + confidence conventions
- Advisor versioning + freeze (replay determinism)
- Failure modes including real-time latency-exceeded path (N4 — `kernel.advisor.suggestion.dropped.v1`)

### Scope (out)

- The Advisor Framework itself (CN-4-022)
- Decision Journal mechanism (CN-4-013)
- Model registry / cost governance specifics (CTR-014 owner = Term 1)
- Tenant-facing UX (Term 3 via CTR-039)
- Channel delivery medium (Term 7 via CTR-040)
- Developer-AI (D-002B — out-of-kernel; CN-4-023)
- Cross-tenant benchmarking (deferred v2 per Q5)
- Learning/feedback (advisor stateless per A10; Architect future)

---

## 2. Doctrine — A1–A10

| # | Law | Source / why |
|---|-----|--------------|
| **A1** | **Advisor = 3-tuple `{audience, data_scope, model}`** per D-002A + CN-4-022 §3. Audience = role(s); data_scope = projections/Documents read; model = pack-driven plug-in (CTR-014 governance). | CN-4-022 framework contract. |
| **A2** | **Advisors NEVER emit business events (Law 3).** Suggestions land in CN-4-013 Decision Journal only. **(Strengthened by N1)** — even `recommended_command_draft` carried on a suggestion is **data**, not a submission. Humans build and submit the command via the normal bus path; no advisor-mediated command path exists. | Law 3; CN-4-013 §2; N1 — prevents "soft autonomy" via UI shortcut. |
| **A3** | **Read scope = projections (CN-5-006 read surface) + Documents (CN-4-012).** Advisors NEVER read raw event store directly. | Read/write separation; replay safety. |
| **A4** | **Suggestion journalled as `kernel.advisor.suggestion.recorded.v1`** per CN-4-013. Required fields: `advisor_id`, `advisor_version`, `audience`, `tenant_ref`, `site_ref?`, `data_scope_summary`, `suggestion_text`, `evidence_refs[]`, `confidence_score`, `confidence_tier` (derived per N2), `recommended_command_draft?` (data only — A2), `model_id`, `model_version`, `pack_version_ref`. | CN-4-013 §3 schema; CN-5-103 G2 (`kernel.*` legitimate Foundation). |
| **A5** | **Human decision journalled as `kernel.advisor.decision.recorded.v1`** per CN-4-013. Captures: `suggestion_ref`, `decision ∈ {acted, dismissed, deferred}`, `human_actor`, `timestamp`, `command_emitted_ref?` (if acted). | CN-4-013 §3; closes the advisor-to-action audit chain. |
| **A6** | **Async fire-and-forget; bus does not wait.** Advisor is a projection consumer triggered by event/schedule/on-demand. Suggestions arrive asynchronously to the audience surface. **(Strengthened by N4)** — for real-time advisors (checkout-advisor), if pack-defined latency budget is exceeded, the suggestion is **dropped**, recorded as `kernel.advisor.suggestion.dropped.v1`, and the originating settlement completes normally. The advisor never blocks the bus. | Operational guarantee; settlement integrity > advisor opinion; N4 graceful degradation. |
| **A7** | **Per-tenant + per-advisor activation pack/tenant governed.** Conservative default: no advisor enabled until tenant opts in. Pack provides "recommended defaults" + cost ceilings per CTR-014. | CTR-014 governance; privacy posture; cost control. |
| **A8** | **Tenant + site scope; cross-tenant via platform-scope gated.** v1 advisors operate within tenant boundary only. Cross-tenant benchmarking (Q5) deferred to v2 with explicit platform-scope governance + opt-in Consent. | CN-4-006 isolation; CN-4-022 scope; Charter §1.2. |
| **A9** | **Advisor versioning + freeze (D-009 analog).** Each advisor carries `advisor_version`; suggestion records version at emission. Replay reproduces under historical version. Breaking change = new `advisor_version`; compatibility window pack-driven (parallel CN-5-103 G6). | Replay determinism; legal defensibility ("what would advisor have said?"). |
| **A10** | **Stateless within framework (per N5 clarification).** Advisor maintains no engine-internal learning state machine in v1. Each evaluation is deterministic given (data scope read + model_version + advisor_version + pack_version). **N5 clarification:** this does NOT prohibit advisor reading the Decision Journal as **input** projection — pack may opt in per-advisor via `data_scope.includes_decision_journal: bool` (default `false`). Reading past decisions is observation; building learned state from them is prohibited in v1. | Q12 ruling; deterministic replay; framework simplicity; N5 distinguishes observation from learning. |

---

## 3. Advisor Catalogue — 9 Advisors with Phase Markers

Nine advisors total (8 in v1 baseline + tax-advisor added by CN-5-105 amendment). Phase 0 = immediately activatable at v1 ratification (5). Phase 1 = activatable only when pack-defined thresholds (N3) are met (4 — procurement, promotion, checkout, tax). All 9 are present in the v1 manifest; activation gates are explicit.

### Phase 0 — Immediately Activatable (5)

| ID | Audience | Primary data scope | Trigger | Default model tier (pack) |
|----|----------|---------------------|---------|----------------------------|
| `bi-advisor` | owner, manager, accountant | Reporting (CN-5-006) cross-engine projections + Statements (CN-4-012) | Scheduled (daily/weekly/monthly) + on-demand | medium |
| `inventory-advisor` | manager, stock_keeper | Inventory (CN-5-003) projections — stock-on-hand, ROP, slow-movers, lot expiry | Scheduled (daily) + event-triggered (post-grn, post-deduct) | low (mostly deterministic) |
| `cash-advisor` | manager, cashier_supervisor | Cash (CN-5-002) — variance trends, AR/AP ageing, deposit pipeline, MM-statement reconciliation | Scheduled (daily) + event-triggered (post-reconcile, post-variance) | low |
| `hr-advisor` | manager, hr_officer | HR (CN-5-005) — leave balances, payroll history, employee loans, attendance, statutory compliance | Scheduled (monthly pre-payroll + quarterly leave review) | low |
| `accounting-advisor` | accountant, bookkeeper | Accounting (CN-5-001) — trial balance, period state, account ageing, journal-flow patterns | Scheduled (pre-period-close + post-close audit) + on-demand | medium |

### Phase 1 — Activatable When Pack-Defined Thresholds Met (4)

Each Phase 1 advisor has explicit pack thresholds (N3). Activation request is **rejected** at command time if thresholds not met:

```yaml
# Pack content (CTR-014 expansion)
pack.advisor.procurement:
  min_supplier_count:                    5          # Phase 1 requires ≥5 registered suppliers
  min_invoice_history_per_supplier:      3          # ≥3 invoices per supplier
  min_invoice_history_window_days:       90
pack.advisor.promotion:
  min_campaign_count_settled:            2          # ≥2 campaigns fully settled
  min_cost_share_cycles_completed:       1          # ≥1 cost-share settlement cycle complete
pack.advisor.checkout:
  real_time_mechanism_ratified:          false      # Architect-phase confirmation required
pack.advisor.tax:                                    # added by CN-5-105 amendment per §4.9
  min_tax_periods_with_activity:         2          # ≥2 closed tax periods of activity
  min_tax_returns_filed:                 1          # ≥1 prior tax return filed
  cost_ceiling:                           <pack-defined>
  schedule_default:                       pre_tax_period_close + post_filing_review
  tier_range:                             [medium, high]
  default_tier:                           medium
```

| ID | Audience | Primary data scope | Trigger | Phase 1 condition |
|----|----------|---------------------|---------|---------------------|
| `procurement-advisor` | procurement_officer, manager | Procurement (CN-5-004) — supplier performance, 3-way-match flags, payment timing for discounts | Scheduled + event-triggered (post-grn, post-invoice) | Pack thresholds (suppliers, history) |
| `promotion-advisor` | marketing, manager | Promotion (CN-5-007) — campaign ROI, cost-share collection, loyalty engagement | Scheduled (post-campaign) | Pack thresholds (settled campaigns, cost-share cycles) |
| `checkout-advisor` | cashier (real-time UI) | Checkout (CN-5-009) — tender mix, refund frequency, customer anomalies | **Real-time post-settlement** (A6 / N4 latency budget) | Real-time mechanism Architect-ratified |
| `tax-advisor` | accountant, tax_officer, owner | Accounting tax-period state + Reporting tax-return summary + Cash tax-authority Obligations + Procurement input-VAT recoverable + HR statutory deductions | Scheduled (pre-tax-period-close + post-filing review) + event-triggered (post-tax-close + post-assessment) | Pack thresholds (≥2 closed tax periods with activity + ≥1 prior return filed) |

### Activation Gate Behaviour (N3)

When a tenant submits a request to enable a Phase 1 advisor, the bus consults the pack thresholds:

```
Tenant submits: advisor.activation.request {advisor_id: procurement-advisor}

Bus validates Phase 1 conditions:
  - Read pack.advisor.procurement.min_supplier_count (5)
  - Query Procurement projection: registered_suppliers_count = 3
  - 3 < 5 → REJECT

rejected_by_policy:    A7.phase1_activation_thresholds_unmet
rejection_reason:      "Advisor procurement-advisor requires ≥5 registered suppliers (current: 3)"
```

Once thresholds are met (e.g., tenant registers more suppliers, accumulates invoice history), activation request succeeds. No grace period; conditions are evaluated at command time.

---

## 4. Per-Advisor Wiring

Each subsection uses the template per §E of the mjadala discussion (rendered as YAML for compactness).

### 4.1 — `bi-advisor`

```yaml
advisor_id:           bi-advisor
advisor_version:      v1
audience:             [owner, manager, accountant]
data_scope:
  primary_projections:
    - reporting.trial_balance
    - reporting.p_and_l_running
    - reporting.balance_sheet_running
    - reporting.sales_analytics
    - reporting.ar_ap_ageing
    - reporting.cash_flow_summary
  documents:           [period_end_statements]
  tenant_scope:        tenant
  includes_decision_journal: false                 # N5 default
trigger:
  kind:                scheduled + on_demand
  schedule_ref:        pack.advisor.bi.schedule (default: daily summary, weekly deep, monthly Statement-driven)
suggestion_shape:
  template_ref:        pack.advisor.bi.suggestion_template
  required_fields:     [evidence_refs, confidence_score, confidence_tier, suggestion_text, recommended_command_draft?]
audience_routing:
  ui:                  Term 3 dashboard (CTR-039)
  channels:            [email digest, sms_high_priority] (opt-in; CTR-040 + CTR-021)
activation:
  pack_default:        recommended (not auto-enabled per Q2)
  tenant_opt_in:       required
  cost_ceiling_ref:    pack.advisor.bi.cost_ceiling
model:
  tier_range:          [low, high]
  default_tier:        medium
phase:                 0
```

**Sample suggestion:** "Mauzo ya Coca-Cola wiki hii TZS 142k vs avg-4-wks TZS 182k (-22%). Causes possible: stock-out (item available 4/7 days at Mwanza), pricing untouched, no seasonality flag. Evidence: [reporting.sales.by_item, inventory.stock.deducted week-trend, inventory.stock_on_hand]. Confidence 0.78 (medium)."

### 4.2 — `inventory-advisor`

```yaml
advisor_id:           inventory-advisor
advisor_version:      v1
audience:             [manager, stock_keeper]
data_scope:
  primary_projections:
    - inventory.stock_on_hand
    - inventory.reorder_point
    - inventory.deduct_trend
    - inventory.lot_expiry_watch
    - inventory.slow_movers
    - procurement.supplier_lead_time
  documents:           []
  tenant_scope:        tenant (queries per site)
  includes_decision_journal: false
trigger:
  kind:                scheduled + event_subscription
  schedule_ref:        pack.advisor.inventory.schedule (default: daily expiry watch)
  subscribed_events:   [inventory.stock.deducted.v1, procurement.grn.received.v1]
suggestion_shape:
  template_ref:        pack.advisor.inventory.suggestion_template
  required_fields:     [evidence_refs, confidence_score, confidence_tier, suggestion_text, recommended_command_draft]
audience_routing:
  ui:                  Term 3 inventory dashboard (CTR-039)
  channels:            [sms reorder_critical] (opt-in)
activation:
  pack_default:        recommended
  tenant_opt_in:       required
  cost_ceiling_ref:    pack.advisor.inventory.cost_ceiling
model:
  tier_range:          [low, medium]
  default_tier:        low
phase:                 0
```

**Sample suggestion:** "Item `coca-cola-500ml` at site `karakana-mwanza-001`: on-hand 4, ROP 12, avg-daily-deduct 2.3, supplier lead time 5d. Suggest reorder qty 50 (covers ~3 wks). Evidence: [inventory.stock_on_hand_<ts>, inventory.deduct_trend_<ts>, procurement.supplier_lead_time_<ts>]. Confidence 0.85 (high)."

### 4.3 — `cash-advisor`

```yaml
advisor_id:           cash-advisor
advisor_version:      v1
audience:             [manager, cashier_supervisor]
data_scope:
  primary_projections:
    - cash.till_position
    - cash.variance_trends
    - cash.ar_ageing
    - cash.ap_ageing
    - cash.deposit_pipeline
    - cash.mm_statement_reconciliation
  documents:           []
  tenant_scope:        tenant
  includes_decision_journal: false
trigger:
  kind:                scheduled + event_subscription
  schedule_ref:        pack.advisor.cash.schedule (default: daily)
  subscribed_events:   [cash.session.reconciled.v1, cash.variance.detected.v1]
suggestion_shape:
  template_ref:        pack.advisor.cash.suggestion_template
  required_fields:     [evidence_refs, confidence_score, confidence_tier, suggestion_text]
audience_routing:
  ui:                  Term 3 cash dashboard (CTR-039)
  channels:            [sms anomaly_alerts] (opt-in)
activation:
  pack_default:        recommended
  tenant_opt_in:       required
  cost_ceiling_ref:    pack.advisor.cash.cost_ceiling
model:
  tier_range:          [low, medium]
  default_tier:        low
phase:                 0
```

**Sample suggestion:** "Till `till-mwanza-2` (cashier human:asha): variance ≥ ±TZS 3000 in 7 of last 14 sessions. Pattern: variance always at session close (no in-session pattern). Suggest physical recount + cashier review meeting. Evidence: [cash.session.reconciled last 14d, cash.variance.detected last 14d]. Confidence 0.72 (medium)."

### 4.4 — `hr-advisor`

```yaml
advisor_id:           hr-advisor
advisor_version:      v1
audience:             [manager, hr_officer]
data_scope:
  primary_projections:
    - hr.employee_leave_balance
    - hr.payroll_period_history
    - hr.employee_loan_outstanding
    - hr.attendance_compliance
    - hr.statutory_filing_due
  documents:           [payroll_period_summary_statements]
  tenant_scope:        tenant
  includes_decision_journal: false
trigger:
  kind:                scheduled
  schedule_ref:        pack.advisor.hr.schedule (default: monthly pre-payroll + quarterly leave + annual statutory)
suggestion_shape:
  template_ref:        pack.advisor.hr.suggestion_template
  required_fields:     [evidence_refs, confidence_score, confidence_tier, suggestion_text, recommended_command_draft?]
audience_routing:
  ui:                  Term 3 HR dashboard (CTR-039)
  channels:            [email monthly_digest] (opt-in)
activation:
  pack_default:        recommended
  tenant_opt_in:       required
  cost_ceiling_ref:    pack.advisor.hr.cost_ceiling
model:
  tier_range:          [low, medium]
  default_tier:        low
phase:                 0
```

**Sample suggestion:** "Employee `asha` annual leave balance 24 days (max 24; pack expiry policy: forfeit unused at fiscal year end Dec 31). Suggest scheduled leave proposal in next 60 days. Evidence: [hr.leave.accrued history, hr.leave.balance.set, pack.hr_leave.annual.max_balance]. Confidence 0.95 (high)."

### 4.5 — `accounting-advisor`

```yaml
advisor_id:           accounting-advisor
advisor_version:      v1
audience:             [accountant, bookkeeper]
data_scope:
  primary_projections:
    - accounting.trial_balance
    - accounting.period_state
    - accounting.account_ageing
    - accounting.journal_flow_pattern
  documents:           [period_end_statements]
  tenant_scope:        tenant
  includes_decision_journal: false
trigger:
  kind:                scheduled + on_demand
  schedule_ref:        pack.advisor.accounting.schedule (default: pre-close + post-close + on_demand)
suggestion_shape:
  template_ref:        pack.advisor.accounting.suggestion_template
  required_fields:     [evidence_refs, confidence_score, confidence_tier, suggestion_text, recommended_command_draft?]
audience_routing:
  ui:                  Term 3 accounting dashboard (CTR-039)
  channels:            [email pre_close_digest] (opt-in)
activation:
  pack_default:        recommended
  tenant_opt_in:       required
  cost_ceiling_ref:    pack.advisor.accounting.cost_ceiling
model:
  tier_range:          [medium, high]
  default_tier:        medium
phase:                 0
```

**Sample suggestion:** "Account `2100 Accounts Payable` has 3 invoices outstanding > 60 days totalling TZS 850,000. Two are with active supplier `aluminium-tz`; one with deactivated supplier `old-cement-supplier`. Suggest payment authorization for active; investigation for deactivated. Evidence: [procurement.invoice.recorded, cash.payment.disbursed cross-walk; procurement.supplier.deactivated]. Confidence 0.88 (high)."

### 4.6 — `procurement-advisor` (Phase 1)

```yaml
advisor_id:           procurement-advisor
advisor_version:      v1
audience:             [procurement_officer, manager]
data_scope:
  primary_projections:
    - procurement.supplier_performance
    - procurement.three_way_match_rate
    - procurement.payment_timing_vs_discount_window
  documents:           []
  tenant_scope:        tenant
  includes_decision_journal: false
trigger:
  kind:                scheduled + event_subscription
  schedule_ref:        pack.advisor.procurement.schedule (default: weekly)
  subscribed_events:   [procurement.grn.received.v1, procurement.invoice.recorded.v1]
suggestion_shape:
  required_fields:     [evidence_refs, confidence_score, confidence_tier, suggestion_text, recommended_command_draft?]
audience_routing:
  ui:                  Term 3 procurement dashboard (CTR-039)
  channels:            [email weekly_digest] (opt-in)
activation:
  pack_default:        recommended
  tenant_opt_in:       required
  phase_1_thresholds:
    min_supplier_count:                 pack.advisor.procurement.min_supplier_count (5)
    min_invoice_history_per_supplier:   pack.advisor.procurement.min_invoice_history_per_supplier (3)
    min_invoice_history_window_days:    pack.advisor.procurement.min_invoice_history_window_days (90)
  cost_ceiling_ref:    pack.advisor.procurement.cost_ceiling
model:
  tier_range:          [low, medium]
  default_tier:        low
phase:                 1
```

### 4.7 — `promotion-advisor` (Phase 1)

```yaml
advisor_id:           promotion-advisor
advisor_version:      v1
audience:             [marketing, manager]
data_scope:
  primary_projections:
    - promotion.campaign_roi
    - promotion.cost_share_collection_status
    - promotion.loyalty_engagement_metrics
    - promotion.voucher_redemption_rate
  documents:           []
  tenant_scope:        tenant
  includes_decision_journal: false
trigger:
  kind:                scheduled
  schedule_ref:        pack.advisor.promotion.schedule (default: post-campaign + monthly summary)
suggestion_shape:
  required_fields:     [evidence_refs, confidence_score, confidence_tier, suggestion_text]
audience_routing:
  ui:                  Term 3 promotion dashboard (CTR-039)
  channels:            [email monthly_digest] (opt-in)
activation:
  pack_default:        recommended
  tenant_opt_in:       required
  phase_1_thresholds:
    min_campaign_count_settled:          pack.advisor.promotion.min_campaign_count_settled (2)
    min_cost_share_cycles_completed:     pack.advisor.promotion.min_cost_share_cycles_completed (1)
  cost_ceiling_ref:    pack.advisor.promotion.cost_ceiling
model:
  tier_range:          [low, medium]
  default_tier:        low
phase:                 1
```

### 4.8 — `checkout-advisor` (Phase 1, Real-Time)

```yaml
advisor_id:           checkout-advisor
advisor_version:      v1
audience:             [cashier]                              # real-time, point-of-checkout
data_scope:
  primary_projections:
    - checkout.tender_mix_per_customer
    - checkout.refund_frequency_per_customer
    - checkout.anomaly_patterns
  documents:           []
  tenant_scope:        tenant (per-customer reads via Party projection)
  includes_decision_journal: false
trigger:
  kind:                event_subscription (POST-SETTLEMENT; per N4 never pre-emption)
  subscribed_events:   [checkout.settled.v1]                 # advisor runs AFTER settlement event recorded
  latency_budget_ref:  pack.advisor.checkout.latency_budget_ms (default: 2000ms)
suggestion_shape:
  required_fields:     [evidence_refs, confidence_score, confidence_tier, suggestion_text]
audience_routing:
  ui:                  Term 3 cashier real-time UI (CTR-039)                    # post-settlement card
  channels:            []                                                       # real-time only; no async channels
activation:
  pack_default:        recommended (but Phase 1 gated)
  tenant_opt_in:       required
  phase_1_thresholds:
    real_time_mechanism_ratified:        pack.advisor.checkout.real_time_mechanism_ratified (false default)
  cost_ceiling_ref:    pack.advisor.checkout.cost_ceiling
model:
  tier_range:          [low, low]                            # tight cost cap; real-time
  default_tier:        low
phase:                 1
```

**Critical N4 behaviour:** if the advisor's evaluation takes longer than `latency_budget_ms` after the settlement event, the suggestion is **dropped** and recorded as `kernel.advisor.suggestion.dropped.v1`. Settlement is unaffected; cashier sees no suggestion for this transaction. See §12.

### 4.9 — `tax-advisor` (Phase 1; CN-5-105 amendment)

```yaml
advisor_id:           tax-advisor
advisor_version:      v1
audience:             [accountant, tax_officer, owner]
data_scope:
  primary_projections:
    - accounting.tax_period_state
    - reporting.tax_return_summary
    - cash.tax_authority_obligations
    - procurement.input_vat_recoverable
    - hr.statutory_deductions_summary
  documents:           [tax_return_statements (current + historical)]
  tenant_scope:        tenant
  includes_decision_journal: true                  # N5 opt-in — review past audit findings + filing history
trigger:
  kind:                scheduled + event_subscription
  schedule_ref:        pack.advisor.tax.schedule_default
                        (default: pre_tax_period_close + post_filing_review)
  subscribed_events:   [accounting.tax_period.closed.v1, accounting.tax.assessed.v1]
suggestion_shape:
  template_ref:        pack.advisor.tax.suggestion_template
  required_fields:     [evidence_refs, confidence_score, confidence_tier, suggestion_text, recommended_command_draft?]
audience_routing:
  ui:                  Term 3 tax dashboard (CTR-039)
  channels:            [email tax_filing_reminders, sms compliance_alerts] (opt-in; CTR-040 + CTR-021)
activation:
  pack_default:        recommended
  tenant_opt_in:       required
  phase_1_thresholds:
    min_tax_periods_with_activity:    pack.advisor.tax.min_tax_periods_with_activity (2)
    min_tax_returns_filed:            pack.advisor.tax.min_tax_returns_filed (1)
  cost_ceiling_ref:    pack.advisor.tax.cost_ceiling
model:
  tier_range:          [medium, high]                # tax is regulatory — higher confidence requirement
  default_tier:        medium
phase:                 1
```

**Sample suggestions (note wording precision per N8 of CN-5-105 — boundary-clean):**

- "Suggested: VAT return for October due 2026-11-20 (per pack.tax_calendar). Net payable TZS 145,000. Click to download VAT Return Statement Document for filing." (NOT "Filing now..." — BOS does not file; accountant downloads + files)
- "Anomaly: Input VAT claimed September TZS 220k vs 12-month avg TZS 95k. Items contributing: [item_refs]. Suggested: review entries before period close."
- "Reminder: Annual PAYE certificate due 2027-01-31. Employees with deduction mismatches: [employee_refs]. Suggested: review and resolve before filing."
- "Tenant turnover at 35% of VAT registration threshold (per tenant_tax_profile.vat_threshold_status). No action needed; will pre-warn at 80%."

**Critical N8 doctrine for tax-advisor:** suggestions describe what the user/accountant should DO with BOS-provided artefacts (download, review, file outside BOS); they NEVER suggest BOS filing or acting as agent. Per Charter §1.3 + CN-5-105 §1 boundary table.

---

## 5. Decision Journal Integration

CN-4-013 owns the journal. CN-5-010 wires concrete usage.

### Suggestion Recording (A4)

```yaml
kernel.advisor.suggestion.recorded.v1
  payload:
    advisor_id:            <one of 8>
    advisor_version:       v1
    audience:              [role(s)]
    tenant_ref:            <tenant>
    site_ref:              <site if applicable>
    data_scope_summary:    <short text summarising what was read>
    suggestion_text:       <human-readable prose>
    evidence_refs:         [<projection_ref>, <document_ref>, …]    # mandatory per A4
    confidence_score:      0.0–1.0
    confidence_tier:       low | medium | high                      # derived per N2
    recommended_command_draft:                                       # OPTIONAL; data only per N1
      kind:                <command name>
      payload_template:    <data shape for human to confirm/edit before submission>
    model_id:              <model identifier>
    model_version:         <model version>
    pack_version_ref:      <pack at suggestion moment — D-009>
    timestamp:             <CN-4-014>
```

### Decision Recording (A5)

```yaml
kernel.advisor.decision.recorded.v1
  payload:
    suggestion_ref:        <suggestion event_id>
    decision:              acted | dismissed | deferred
    human_actor:           <CN-4-007 principal>
    deferral_reason:       <optional, if deferred>
    dismissal_reason:      <optional, if dismissed>
    command_emitted_ref:   <event_id of the command the human submitted, if acted>
    timestamp:             <CN-4-014>
```

### N1 — `recommended_command_draft` Is Data, Not a Command

The `recommended_command_draft` field on the suggestion is **data describing a command a human might choose to submit**. It is NOT a command submission. When the human clicks `[Act]` in the Term 3 UI (CTR-039), Term 3:

1. Reads the draft from the suggestion event
2. Constructs a real command from the draft + any human edits (e.g., qty adjustment, supplier choice)
3. Submits the command to the bus via the **normal command path** (with the human's principal)
4. Bus evaluates standard policies (approval gates, validation, scope, consent, etc.)
5. If accepted, business event(s) emitted normally
6. The `kernel.advisor.decision.recorded.v1` event records the `command_emitted_ref` linking back

**No advisor-mediated command path exists.** The advisor proposes; the human disposes; the bus enforces. Even if the human takes the suggestion verbatim, the command goes through the bus's full policy stack — the suggestion does not bypass any check.

### Audit Chain

For any business event downstream of an advisor suggestion, the causation walk reaches back:

```
business_event → command → kernel.advisor.decision.recorded.v1 → kernel.advisor.suggestion.recorded.v1 → triggering_event_or_schedule
```

UI-01 is preserved throughout. Auditors can ask "why did this PO get created?" and walk back: PO → requisition → human command → decision (acted) → advisor suggestion → original low-stock observation.

---

## 6. Cross-Tenant Boundary (A8)

### v1 — Single-Tenant Only

All v1 advisors operate within a single tenant's data scope. No advisor reads across tenants. Site-scope reads stay within the tenant's site registry (UI-09 / CTR-027).

### v2 Future — Cross-Tenant Benchmarking (Q5)

Cross-tenant "benchmarking" advisors ("biashara nyingine za aina yako kawaida zinapata X") are explicitly **deferred to v2**. Requires:

- Platform-scope advisor pattern (CN-4-006 §2) — separate audit trail
- Opt-in Consent per tenant per benchmarking purpose (CN-4-011 Consent)
- Anonymisation guarantees (CN-4-022 §scope — aggregate-only, no identifiable per-tenant data)
- Term 1 governance approval for the benchmarking advisor

Open Item §18. No v1 advisor declares cross-tenant data scope.

---

## 7. Activation Governance

### Activation Flow

```
1. Tenant requests:    advisor.activation.request {advisor_id, model_tier?, cost_cap?}
2. Bus validates:
   a. advisor_id exists in registry (CTR-041)
   b. Phase 1 thresholds met if Phase 1 advisor (N3)
   c. Cost ceiling within pack.advisor.<id>.cost_ceiling
   d. Model tier within pack.advisor.<id>.tier_range
3. Bus emits:          advisor.activation.recorded.v1                # under advisor.* namespace? OR kernel.advisor.* per CTR-041 ruling
4. Suggestions begin flowing per advisor's trigger
```

### Default Posture (Q2 — Conservative)

- **Pack default:** advisors marked `recommended` (informational; tenant decides)
- **Tenant must explicitly enable** each advisor; no auto-enable on pack rollout
- Activation is reversible: `advisor.deactivation.request` ceases suggestions; existing recorded suggestions remain (CN-4-001 immutability)

### CTR-014 Expansion (Cost Governance)

Pack hooks added by CN-5-010:

```yaml
pack.advisor.<advisor_id>:
  cost_ceiling:                   <max monthly cost per tenant>
  tier_range:                     [<min>, <max>]                   # allowed model tiers
  recommended_default:            true | false
  phase_1_thresholds:             {...}                            # per-advisor N3 (Phase 1 only)
  confidence_tier_cutoffs:        {low: 0.5, medium: 0.8}          # N2 — pack-overridable
  schedule_ref:                   <schedule reference>
  latency_budget_ms:              <real-time advisors only>
```

Term 1 governs per-tenant election within pack bounds; bills usage per CTR-014.

### CTR-041 — Advisor Registry

Per CTR-041, Term 4 defines an advisor registry mechanism parallel to CN-4-020 engine registry. Each advisor_id is registered as part of pack content; tenant activation references registered advisor_ids only. Unknown advisor_ids are rejected at command time.

---

## 8. Suggestion Delivery

### UI Primary (CTR-039)

Per Q3 + Q9 + CTR-039: Term 3 surfaces advisor suggestions in role-specific feeds. Each suggestion card shows:

- Suggestion text (with Kiswahili-first option per pack)
- Confidence tier + score
- Evidence drill-down (links to underlying projections/Documents)
- Action buttons: `[Act]` / `[Dismiss]` / `[Defer]`
- If `recommended_command_draft` present: pre-filled form for human to confirm/edit before submission (N1)

Term 3 records the human's choice via `kernel.advisor.decision.recorded.v1`.

### Channels (CTR-040, Consent-Gated)

Per CTR-040 + CTR-021: optional digest/alert delivery via SMS / WhatsApp / email per tenant + per advisor:

- Consent purpose = `advisor_digest` (new purpose added per CTR-021 expansion)
- Hard-block at send if consent absent (parallel CN-5-007 PR5)
- Per-advisor channel preference (e.g., bi-advisor weekly summary via email; cash-advisor anomaly alerts via SMS)

Delivery mechanism is Term 7's (per CN-5-006 N5 + CN-5-007 §13 alert delivery boundary). Advisor emits suggestion event; Term 7 routes per pack rules.

### Audience Routing

Each advisor declares `audience` (CN-4-022 §3 + A1). Term 3 ensures only audience members see suggestions in their feed; channels respect per-customer consent + audience scope.

---

## 9. Evidence + Confidence Conventions

### Mandatory `evidence_refs[]` (Q8)

Every suggestion **must** carry `evidence_refs[]` — non-empty list of pointers to projections, Documents, or events that support the claim. Pack may enforce minimum count (default: ≥1). Suggestions without evidence_refs are rejected at emission.

Reasons:
- **Explainability** — human can verify the basis
- **Audit** — regulator or accountant can walk evidence
- **Debugging** — when suggestion is wrong, evidence shows why

### Confidence Score + Derived Tier (Q11 / N2)

- **`confidence_score`** = decimal 0.0–1.0 — the **source of truth**
- **`confidence_tier`** = `low | medium | high` — **derived** from score per pack cutoffs:

```yaml
# Pack content (CTR-014 expansion)
pack.advisor.confidence_tier_cutoffs:
  low:    <  0.5            # default
  medium: 0.5 ≤ x < 0.8     # default
  high:   ≥ 0.8             # default
# Tenants may override within pack permission
```

Pack-overridable; per-tenant override permitted within bounds.

Tier exists for human readability (UI badges, channel digest summaries); score exists for analytics, sorting, threshold filters.

### Display Conventions (Term 3 — CTR-039)

- UI shows tier badge prominently (colour-coded per pack)
- Score visible on detail expansion
- Sub-low-tier suggestions may be hidden from default feed (pack-driven)

---

## 10. Feedback Loop & Stateless Property (A10 + N5)

### Stateless Default (A10)

Advisors maintain no learning state in v1. Each evaluation is deterministic given:

- Data scope read at evaluation time
- `model_id` + `model_version`
- `advisor_version`
- `pack_version_ref`

Replay reproduces the same suggestion under the same inputs + frozen versions. This is required for replay determinism (CN-4-001 §6).

### N5 — Decision Journal as Input (Opt-In)

The stateless rule prohibits **advisor-owned learning state machines**, NOT advisor reading the Decision Journal as input data.

- **Default v1:** `data_scope.includes_decision_journal: false` for all 8 advisors
- **Opt-in per advisor:** pack may enable for specific advisors (e.g., `cash-advisor` may read past variance dismissals to avoid re-suggesting same pattern)

When opted in, the advisor reads the Decision Journal projection like any other projection (CN-5-006 read surface). The advisor still does NOT maintain its own derived state — each evaluation reads fresh from the journal projection.

This distinguishes:
- ❌ Prohibited: advisor maintains "memory of past suggestions and outcomes" influencing future model behaviour
- ✅ Permitted: advisor reads journal as observation input each time

### Learning / Feedback Future

Architect-phase concern. May arrive in CN-5-010 v2 if real demand emerges. Not in v1 scope.

---

## 11. Advisor Versioning + Freeze (A9)

Per A9 (D-009 analog; CN-5-103 G6):

| Aspect | Rule |
|--------|------|
| Baseline | `advisor_version: v1` at first ratification of each advisor |
| Recorded on suggestion | Every `kernel.advisor.suggestion.recorded.v1` carries `advisor_version` |
| Additive change | Within `v1`, suggestion-text style refinements + new evidence categories permitted (no breaking schema change) |
| Breaking change | Requires `v2` (new logic, new prompt template, new model class). `v1` and `v2` coexist during pack-driven compatibility window |
| Replay | Suggestion replayed under recorded `advisor_version` + `model_version` + `pack_version_ref` + data scope at original moment → reproduces original suggestion deterministically |

### Replay Determinism Test

A 2026 suggestion replayed in 2046:
1. Load original `kernel.advisor.suggestion.recorded.v1` event
2. Read recorded `advisor_version`, `model_version`, `pack_version_ref`
3. Replay underlying projections to original `global_position`
4. Re-evaluate advisor with frozen versions + replayed inputs
5. Result must match original `suggestion_text` (modulo model-determinism guarantees)

This is what makes advisor suggestions legally defensible — "what would the advisor have said?" reproducible decades later.

---

## 12. Failure Modes

| Failure | Effect | Recovery |
|---------|--------|----------|
| **Advisor model unavailable** | Suggestion not emitted; failure event logged | Retry per pack policy; alternate model tier if pack permits |
| **Model timeout (non-real-time)** | Suggestion deferred to next scheduled run | No state change; audit records timeout |
| **Real-time advisor latency-exceeded (N4)** | `kernel.advisor.suggestion.dropped.v1` emitted; settlement completes normally | No suggestion delivered for that transaction; cashier UI shows no card; doctrine: settlement integrity > advisor opinion |
| **Data scope incomplete** (projection lag, missing source) | Suggestion suppressed; reason logged | Advisor re-evaluates on next trigger when projection catches up |
| **Cost ceiling exceeded** | Subsequent suggestions blocked until next cycle; tenant notified | Tenant adjusts ceiling (CTR-014) or waits |
| **Consent absent for channel delivery** | Suggestion remains in UI feed; channel delivery blocked (PR5 pattern) | Suggestion still discoverable in UI |
| **Activation request fails Phase 1 thresholds** | Bus rejects activation per N3 | Tenant accumulates required history; re-requests later |
| **Unknown advisor_id at activation** | Rejected per CTR-041 registry | Registry update via Term 4 process |
| **Advisor reads projection with version mismatch** (e.g., projection rebuilt under newer handler) | Per CN-4-010 §7 multi-version handling; advisor must opt into compatible version | Pack/Architect manages handler versions |

### N4 Detail — Real-Time Drop

```yaml
kernel.advisor.suggestion.dropped.v1                              # Foundation event per CN-4-013 living-catalog extension
  payload:
    advisor_id:            checkout-advisor                       # typically real-time advisor
    advisor_version:       v1
    tenant_ref:            <tenant>
    site_ref:              <site>
    trigger_event_ref:     <event_id of checkout.settled.v1>
    drop_reason:           latency_budget_exceeded
    latency_budget_ms:     2000
    elapsed_ms:            2417
    timestamp:             <CN-4-014>
```

This is a **Foundation event** under the `kernel.advisor.*` namespace (per CN-5-103 G2 — `kernel.*` is Foundation's). Term 4 confirms via existing living-catalog authority on CN-4-013 — not a separate CTR. CN-5-010 ratification triggers the CN-4-013 extension.

---

## 13. UI Invariants Touched

| Invariant | Application in CN-5-010 |
|-----------|-------------------------|
| **UI-01** (causation chain) | Suggestion events carry causation to triggering event/schedule; decision events carry causation to suggestion; downstream business events carry causation to commands the human submitted. Full chain walks: business event → command → decision → suggestion → trigger. |
| **UI-04** (tender chain) | Only `checkout-advisor` (Phase 1, real-time) reads checkout settlement data. When advisor reads tender data, it does NOT participate in tender chain — it observes post-settlement. UI-04 backward walk from `cash.tender.received.v1` is unaffected; advisor reads are sideways observations. |
| **UI-05** (closed-period inviolability) | Advisor may read closed-period data (Statements, historical projections) for analytical suggestions; advisor never emits events into closed periods (A2 — no business events at all). |
| **UI-06** (functional currency) | Advisor presents financial figures in tenant functional currency unless tenant explicitly opts in to multi-currency view (advanced; future). |
| **UI-08** (Obligation bounds) | Read-only — advisor observes Obligation outstanding for suggestions (loan repayment, AR collection, cost-share follow-up); no state mutation. |

---

## 14. Adapter / Subscription Map

Per-advisor cross-engine subscriptions (read surface):

| Advisor | Primary subscriptions / projection reads |
|---------|-------------------------------------------|
| `bi-advisor` | Reporting projections (cross-engine fan-in surfaces); Statements |
| `accounting-advisor` | Accounting trial-balance, period-state projections; Statements |
| `cash-advisor` | Cash variance + ageing projections; subscribes `cash.session.reconciled.v1`, `cash.variance.detected.v1` for event-triggered runs |
| `inventory-advisor` | Inventory stock + expiry projections; subscribes `inventory.stock.deducted.v1`, `procurement.grn.received.v1` |
| `procurement-advisor` (Phase 1) | Procurement supplier-performance + match-rate projections; subscribes `procurement.grn.received.v1`, `procurement.invoice.recorded.v1` |
| `hr-advisor` | HR leave + payroll + statutory projections; scheduled trigger only |
| `promotion-advisor` (Phase 1) | Promotion ROI + cost-share + loyalty projections; scheduled |
| `checkout-advisor` (Phase 1, real-time) | Checkout tender-mix + refund + customer-anomaly projections; subscribes `checkout.settled.v1` (post-settlement, per N4) |

All subscriptions are **`kind: projection`** (CN-5-100 §2 P2) — advisor reads and stops; no command emission from advisor (A2). The "command emission" from advisor-suggestion chain happens via the human's separate command, not from the advisor's subscription handler.

---

## 15. Worked Example — Karakana Inventory-Advisor Full Lifecycle + Contrast Scenario

*Karakana ya Mzee Hassan serves as illustrative context per D-004 #4 (peer-technical audience).*

### 15.1 Happy Path — `inventory-advisor` Reorder Suggestion (8 Steps)

**Setting:** Karakana Mwanza, item `coca-cola-500ml`, ROP 12, on-hand 4 after recent sales, avg-daily-deduct 2.3, supplier `coca-cola-tz` lead time 5d.

**Step 1 — Trigger (event-subscription per §4.2)**
```
inventory.stock.deducted.v1 emitted (latest sale, on-hand drops to 4)
inventory-advisor subscribed (event_subscription trigger)
Advisor evaluation runs async
```

**Step 2 — Data scope query (A3)**
```
Read projections:
  inventory.stock_on_hand[karakana-mwanza-001, coca-cola-500ml] = 4
  inventory.reorder_point[coca-cola-500ml] = 12
  inventory.deduct_trend[coca-cola-500ml, last_30d] = 2.3 avg/day
  procurement.supplier_lead_time[coca-cola-tz] = 5d avg
```

**Step 3 — Compute suggestion**
```
on_hand (4) < ROP (12) → REORDER NEEDED
suggested_qty = avg_daily × (lead_time + safety_buffer)
             = 2.3 × (5 + 7) ≈ 28
Round up to typical pack size: 50
Confidence: 0.85 (high — data sufficient + clear signal)
```

**Step 4 — Emit suggestion event (A4)**
```yaml
kernel.advisor.suggestion.recorded.v1:
  advisor_id:           inventory-advisor
  advisor_version:      v1
  audience:             [manager, stock_keeper]
  tenant_ref:           mzee-hassan-karakana
  site_ref:             karakana-mwanza-001
  data_scope_summary:   "stock on-hand, ROP, 30d deduct trend, supplier lead time"
  suggestion_text:      "Item coca-cola-500ml: on-hand 4, ROP 12. Suggest reorder qty 50 (covers ~3 wks with 5d lead time)."
  evidence_refs:
    - reporting.inventory.stock_on_hand<ts>
    - reporting.inventory.deduct_trend<ts>
    - reporting.procurement.supplier_lead_time<ts>
  confidence_score:     0.85
  confidence_tier:      high                              # derived per N2 cutoffs
  recommended_command_draft:                              # N1 — data only
    kind:               procurement.requisition.create.request
    payload_template:
      supplier_ref:     coca-cola-tz
      items:            [{item: coca-cola-500ml, qty: 50}]
  model_id:             pack.advisor.inventory.model
  model_version:        <version>
  pack_version_ref:     tz-compliance-2026.07
  timestamp:            <CN-4-014>
```

**Step 5 — UI surfaces suggestion (CTR-039)**

Manager Mzee Hassan sees in Inventory Advisor feed:

```
┌──────────────────────────────────────────────────────────────────┐
│  📦  Coca-Cola 500ml — Reorder Needed                            │
│      Mwanza · On-hand 4 / ROP 12                                 │
│      Suggest: order 50 from Coca-Cola TZ                         │
│      Confidence: HIGH (0.85)                                     │
│      [View Evidence ↗]  [Act]  [Dismiss]  [Defer]               │
└──────────────────────────────────────────────────────────────────┘
```

**Step 6 — Human decision (A5)**

Mzee Hassan reviews evidence (clicks View Evidence; sees underlying projection values); clicks `[Act]`. Term 3:

1. Reads `recommended_command_draft` from suggestion event
2. Presents pre-filled requisition form (qty 50, supplier coca-cola-tz)
3. Mzee Hassan confirms (no edits); clicks Submit
4. Term 3 emits both:

```yaml
kernel.advisor.decision.recorded.v1:
  suggestion_ref:        <step 4 event_id>
  decision:              acted
  human_actor:           human:mzee-hassan
  timestamp:             <CN-4-014>
  command_emitted_ref:   <step 7 event_id, filled below>
```

**Step 7 — Command submitted (N1 — normal bus path)**

`human:mzee-hassan` submits `procurement.requisition.create.request` via normal bus. Bus evaluates per CN-5-004 §3:

- Manager approval gate (TZS 4.5M — under owner band; manager approval auto-granted given role)
- Validation passes
- Emits: `procurement.requisition.created.v1`, then `procurement.requisition.approved.v1`, then `procurement.po.created.v1` (Document issued)

**Step 8 — Audit chain (UI-01)**

Walking from `procurement.po.created.v1` backward via causation:

```
procurement.po.created.v1
  causation_id → procurement.requisition.approved.v1
  causation_id → procurement.requisition.created.v1
  causation_id → command derivative
                 (correlated with kernel.advisor.decision.recorded.v1)
                   suggestion_ref → kernel.advisor.suggestion.recorded.v1
                                    triggered by inventory.stock.deducted.v1
                                                  causation_id → original sale
```

Auditor question: "Why did this purchase order exist?" — answer reconstructed: a low-stock observation by inventory-advisor; manager Mzee Hassan agreed and acted. Every link is in the store.

### 15.2 Contrast Scenario — `checkout-advisor` Latency Exceeded (N4)

**Setting:** Cashier completes a sale at Mwanza. `checkout.settled.v1` emitted at t=0. `checkout-advisor` subscribed; evaluation begins.

**Step 1 — Settlement completes normally (A6 — advisor never blocks)**
```
checkout.settled.v1 emitted at t=0
Receipt issued, Accounting / Cash / Inventory subscriptions begin handling
Cashier UI shows receipt printed; transaction complete
```

**Step 2 — Advisor evaluation begins**
```
checkout-advisor subscription handler picks up event at t≈100ms
Begins reading projections + invoking model
```

**Step 3 — Latency budget exceeded (N4)**
```
pack.advisor.checkout.latency_budget_ms = 2000
At t=2000ms, evaluation still running (model timeout / projection lag / network)
Framework triggers drop
```

**Step 4 — Suggestion dropped**
```yaml
kernel.advisor.suggestion.dropped.v1:                     # Foundation event per CN-4-013 extension
  advisor_id:           checkout-advisor
  advisor_version:      v1
  tenant_ref:           mzee-hassan-karakana
  site_ref:             karakana-mwanza-001
  trigger_event_ref:    <checkout.settled.v1 event_id>
  drop_reason:          latency_budget_exceeded
  latency_budget_ms:    2000
  elapsed_ms:           2417
  timestamp:            <CN-4-014>
```

**Step 5 — Cashier UI shows no suggestion**

No suggestion card appears for this transaction. The cashier moves on to the next customer. Settlement is complete; nothing is broken.

**Step 6 — Audit visibility**

Operations team can query `kernel.advisor.suggestion.dropped.v1` events to see drop frequency. If checkout-advisor drops > pack-defined threshold % of evaluations, operations adjusts (increase latency budget, optimise model, or deactivate advisor temporarily).

### What the Two Scenarios Demonstrate Together

| Doctrine | Happy path | Contrast |
|----------|------------|----------|
| **A2** — no business events from advisor | Advisor proposed; human submitted command | Advisor dropped silently; settlement unaffected |
| **A4 + A5** — full journal chain | Both suggestion + decision recorded | Drop recorded; no decision (nothing to decide on) |
| **A6** — async, non-blocking | Async post-event evaluation | Async drop; bus never waited |
| **N1** — `recommended_command_draft` is data | Term 3 built command from draft; human confirmed; bus enforced | (no command — dropped before suggestion) |
| **N4** — real-time graceful degradation | (not applicable — non-real-time advisor) | Drop preserves settlement integrity |
| **UI-01** — causation | Full chain walks PO ← requisition ← command ← decision ← suggestion ← trigger | Drop event causation walks to trigger only (no decision/command) |
| **A9** — versioning | `advisor_version: v1` on suggestion | `advisor_version: v1` on drop |

---

## 16. Three Whys

### Why does this matter?

Tenants have engines that record what happened; they need a layer that helps them **notice patterns** in what happened and **decide what to do next**. Without advisors, every tenant has to invent their own analytics + intuition — and most SMEs do not have a data team. CN-5-010 makes a small, focused set of advisors available out-of-the-box: stock running low, cash variance pattern, leave balance about to expire, period close pre-check, supplier slowdown, promotion ROI. Each advisor is bounded, auditable, and replay-deterministic. Mama Asha runs her shop with a quiet inventory-advisor watching her stock; Mzee Hassan plans his cash with a cash-advisor flagging variances. The system suggests; the human decides; the bus enforces. Law 3 is not an aspiration — it is the structural reality.

### Why does it belong here (and not in Foundation)?

Foundation provides the Advisor Framework (CN-4-022 — audience / scope / model), the Decision Journal (CN-4-013 — recording mechanism), and the advisory-only doctrine guarantees. Foundation does not say "every business needs an inventory advisor that suggests reorders when stock is below ROP" or "the cash-advisor should flag variance patterns ≥3 occurrences in 14 days." Those are universal-engine wiring choices appropriate for any business with stock and cash. CN-5-010 makes those choices as the universal advisor pattern. A specialised advisor set (e.g., trading-floor-only) could be wired differently — but that is not universal commerce.

### Why this design?

The 8-advisor catalogue maps 1-to-1 to engines plus a cross-engine BI advisor — symmetrical and easy to extend. Phase 0 / Phase 1 split lets immediately-mechanizable advisors activate at v1 ratification while deferring those that need data maturity or Architect-phase mechanism (N3 thresholds). The `recommended_command_draft` as data (N1) preserves Law 3 absolutely — even a "highly confident" advisor cannot bypass the bus. Stateless default (A10) with optional Decision Journal read (N5) accommodates both the simple case (most advisors) and the future case (advisor that learns from dismissals) without committing to learning machinery now. Versioning (A9) makes suggestions replay-deterministic — a 2026 advisor warning that turned out to be right (or wrong) can be reproduced in 2046. The real-time drop pattern (N4) keeps the bus untouchable — settlement integrity wins over advisor opinion, every time. The whole design is one cohesive answer to "how can BOS help tenants think better without ever taking decisions away from them?"

---

## 17. Boundaries

| Topic | Lives in |
|-------|----------|
| Advisor Framework contract (3-tuple, advisor lifecycle, model plug-in) | CN-4-022 |
| Decision Journal events + schema | CN-4-013 |
| `kernel.advisor.suggestion.dropped.v1` extension (N4) | CN-4-013 (Term 4 living-catalog authority — confirmed on CN-5-010 ratification, not separate CTR) |
| Identity / principal model (advisor as principal type) | CN-4-007 |
| Tenant + site scope; platform scope for v2 benchmarking | CN-4-006 |
| Snapshot/projection storage Reporting reads from | CN-4-018 + CN-5-006 |
| Engine registry pattern → advisor registry (CTR-041) | CN-4-020 |
| Subscription patterns | CN-5-100 |
| Scope policy | CN-5-101 |
| UI invariants (UI-01 + UI-04 + UI-05 + UI-08) | CN-5-102 |
| Event naming (`kernel.advisor.*` legitimate Foundation per G2) | CN-5-103 |
| Per-engine projections advisors read | CN-5-001 / 002 / 003 / 004 / 005 / 006 / 007 / 009 |
| BI/KPI projection foundation Reporting provides | CN-5-006 (per R7 of CN-5-006) |
| Tenant-customer-facing advisor UX | Term 3 future (D-DISC-001) |
| Term 3 advisor suggestion UI (feed, action, evidence drill-down) | Term 3 via CTR-039 |
| Channel delivery (SMS / WhatsApp / email digest) | Term 7 via CTR-040 + CTR-021 consent purpose `advisor_digest` |
| Per-advisor cost ceiling + model-tier elections + Phase 1 thresholds | Term 1 via CTR-014 expansion |
| AI Mode dashboard pattern (advisor-suggestion-feed primitive) | Term 7 / Term 3 via CTR-015 expansion |
| Developer-AI (build-time, outside kernel) | CN-4-023 (D-002B — NOT this doc) |

---

## 18. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| **D-DISC-001** — Tenant-customer-facing advisor surfaces | Term 3 future | CN-5-010 generates per Q9; Term 3 surfaces when customer-facing UX work begins |
| Phase 1 activation thresholds in pack content per jurisdiction | Term 1 via CTR-014 expansion | Schema in §3; per-jurisdiction tuning is Term 1's authorship |
| Cross-tenant benchmarking (v2 — Q5) | Future + CN-4-006 platform-scope + Consent opt-in | Requires platform-scope advisor pattern + anonymisation guarantees + Term 1 governance |
| Learning / feedback loop (Q12, future) | Architect phase + future v2 | A10 stateless v1; learning machinery deferred |
| Decision Journal projection opt-in per advisor (N5) | Pack content + per-advisor decision | Default false; opt-in mechanism is per-advisor `data_scope.includes_decision_journal` field |
| **`kernel.advisor.suggestion.dropped.v1`** Foundation extension | Term 4 living-catalog authority on CN-4-013 | Pending Term 4 confirmation upon CN-5-010 ratification (not separate CTR) |
| `checkout-advisor` real-time mechanism ratification (Phase 1 condition) | Architect phase | `pack.advisor.checkout.real_time_mechanism_ratified` flag defaults false until Architect ratifies cache + latency strategy |
| CTR-039 (Term 3 UI surface) — full specification | Term 3 | Suggestion feed, action buttons, evidence drill-down, dismissal/deferral capture |
| CTR-040 (Term 7 channel routing) — adapter contract | Term 7 | Digest formats per channel; consent purpose `advisor_digest` |
| CTR-041 (Term 4 advisor_id registry) — mechanism | Term 4 | Parallel CN-4-020 engine registry; per-tenant activation references registered ids |
| CTR-014 expansion finalised | Term 1 | Per-advisor cost ceilings + model-tier elections + Phase 1 thresholds |
| CTR-015 expansion finalised | Term 7 / Term 3 | AI Mode dashboard advisor-suggestion-feed primitive |
| Advisor performance SLO at scale (suggestion latency, false-positive rate per advisor) | Architect phase | Concept guarantees correctness; performance is Architect's |
| Multi-currency advisor presentation (multi-currency tenants — rare in v1) | Future | UI-06 single-currency default; multi-currency view deferred |
| Doctrine check for A2 (no business events from advisor code path) | CN-4-019 + future DC | Static analysis can detect; arrives with CN-5-010 ratification |
| Doctrine check for A4 mandatory `evidence_refs[]` | CN-4-019 + future DC | Schema validation can enforce |

---

*— End of CN-5-010 —*
