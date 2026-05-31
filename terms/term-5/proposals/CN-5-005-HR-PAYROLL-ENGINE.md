# CN-5-005 — HR & Payroll Engine

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 5 — Universal Engines
> **Status:** For Overseer review.
> **Governing decisions:** Law 5 (compliance is configured — statutory deductions, leave rules, payroll periods, termination computation in packs); D-009 (freeze doctrine — historical payrolls under historical pack/standard); D-004 (neutrality — no jurisdiction named in engine code); Charter §1.3 (BOS is not a statutory HR/payroll filing system — management truth only); UI-01 (causation chain); UI-05 (closed-period inviolability); UI-06 (functional currency); UI-08 (Obligation bounds preserved across loan lifecycles).
> **CTRs:**
> - **OPEN (this doc extends with expansion notes at merge):** CTR-029 (Term 1 — pack section gains: `hr_leave.types`, `hr_payroll.period_frequency`, `hr_statutory_deductions.*` per jurisdiction, termination computation rules); CTR-030 (Term 6 — verticals emit `<vertical>.commission_earned.v1`, `<vertical>.piece_completed.v1` per vertical's compensation pattern).
> - **OPEN (this doc depends on):** CTR-027 (Term 1/2 — tenant functional currency + site registry).
> - **Open Item (future):** CTR-034 (Term 7 — payroll filing-format adapters: PAYE file, pension file, NHIF file generation). Per Charter §1.3 BOS does not file; filing-format generation is filing-prep and is deferred until clear tenant demand.
> **Glossary:** See `MASTER-GLOSSARY.md` — Universal-engine Invariant (UI), Site, Scope level, Obligation, Party.
> **Depends on:** CN-4-002 (envelope, causation_id, compensates_event_id); CN-4-004 (command bus + atomic emission + policy rejection); CN-4-011 (Party primitive for employee identity, Workflow primitive for employment/leave/payroll lifecycles, Approval primitive for leave/loan/payroll/termination gates, Obligation primitive for employee loans); CN-4-014 (clock protocol — business_date, period boundaries, leave-accrual schedule); CN-4-015 (Compliance DSL — statutory deductions, leave accrual, payroll period rules, termination rules); CN-4-007 (Identity — system:scheduler principal for accrual runs); CN-5-001 (Accounting auto-journals on payroll events #10 / #11); CN-5-002 (Cash disburses payroll and loans per §9 employee loan choreography; subscribes payroll deduction events for loan repayment); CN-5-100 (subscription patterns — auto-journal P3; compensation P4); CN-5-101 (scope policy — HR is multi-scope per §6: tenant default + site overrides for attendance, site-anchored assignments, site-initiated loan requests); CN-5-102 (UI-01, UI-05, UI-06, UI-08).
> **Boundaries:** Party / Workflow / Approval / Obligation primitive folds → CN-4-011; pack content per jurisdiction → Term 1 (CTR-029); Compliance DSL grammar → CN-4-015; payroll journal mappings → CN-5-001 §4 (#10 accrual, #11 settlement); loan disbursement / repayment cash mechanics → CN-5-002 §9; tip pass-through routing → CN-5-002 §16; vertical commission/piece-rate event contracts → Term 6 via CTR-030; statutory tax/pension/insurance submission → outside BOS (chartered HR officer or accountant per Charter §1.3); filing-format generation → Term 7 future (CTR-034 Open Item).

---

## 1. What This Doc Defines

CN-5-005 is the HR & Payroll Engine — the **employee lifecycle and payroll truth layer** for any business. The engine handles the full employment lifecycle from registration to termination, computes payroll periodically with full statutory deductions, manages leave balances, and orchestrates employee loans across Cash and Accounting.

Two non-negotiable framing constraints govern the design (parallel to CN-5-001):

1. **Statutory rules live in packs, not code** (Law 5). PAYE tax bands, pension contribution rates, NHIF / health insurance, training levies, leave accrual rules, termination computation — all in compliance pack. Engine code is jurisdiction-agnostic.
2. **Management truth only — not statutory filing** (Charter §1.3). BOS computes payroll, tracks loans, accrues leave, produces payslips, generates statutory-summary reports. **The tenant's HR officer or chartered accountant files** with tax authorities, pension funds, and insurance bodies.

This doc defines:
- **Engine manifest** (§3) — multi-scope per CN-5-101 §6.
- **Employee management** via Party primitive + HR projection (§4).
- **Roles, assignments, compensation** — versioned (§5).
- **Leave management** — pack-driven accrual + balance lifecycle (§6).
- **Attendance** — site-scope (§7).
- **Payroll periods and computation flow** (§8).
- **Statutory deductions** — pack-driven (§9).
- **Employee loan choreography** — recap from CN-5-002 §9 plus write-off path (§10).
- **Commission / piece-rate / tip integration** — vertical Pattern B (§11).
- **Termination and final settlement** — workflow with loan resolution (§12).
- **Charter §1.3 boundary** — explicit (§13).
- **Cross-engine subscription map** (§14).
- **UI invariants enforcement** (§15).

This doc does NOT define:
- Primitive folds (CN-4-011).
- Compliance DSL grammar (CN-4-015).
- Pack content per jurisdiction (Term 1 via CTR-029).
- Accounting payroll journal mappings (CN-5-001 §4 #10 / #11).
- Cash loan/payroll disbursement mechanics (CN-5-002 §9).
- Vertical commission/piece-rate event semantics (Term 6 via CTR-030).
- Statutory filing (outside BOS per Charter §1.3).

---

## 2. The Seven Laws

| # | Law | Source / why |
|---|-----|--------------|
| **H1** | **Employee = Party primitive + HR projection.** Employee identity lives in Party primitive (CN-4-011); HR-specific state (roles, assignments, compensation, leave balance, employment history, loan status) lives in HR engine projection. | CN-4-011 Party primitive is identity-only; employment relationship is engine-specific state that evolves over time. Same separation as supplier in CN-5-004 §4. |
| **H2** | **Assignments and compensation are versioned.** Role changes, salary revisions, transfers between sites — all event-sourced; truth at any moment = fold of events. Full employment history reconstructable for audit, dispute, regulatory inspection. | Charter §1.2 audit completeness; employment law often requires multi-year retention; replay determinism (CN-4-001). |
| **H3** | **Payroll is a per-period event.** Each payroll period (monthly default; pack-configurable) is a discrete `hr.payroll.computed.v1` event capturing gross pay, all deductions (statutory + voluntary + loan), and net pay per employee. Re-computation post-emission requires compensating event (no edits per CN-4-001 immutability + UI-05). | D-009 freeze applies: a closed payroll period uses the pack/standard active at run time; later rate changes do not retroactively re-compute. |
| **H4** | **Statutory deductions are pack-driven (Law 5).** PAYE tax bands, pension contribution rates, NHIF / health insurance, training levies, leave accrual rules, termination computation — all live in compliance pack. Engine code never names a jurisdiction-specific deduction; pack content does. | Law 5; D-004 neutrality; D-009 freeze. Adding new country = new pack content; engine code unchanged. |
| **H5** | **Charter §1.3 boundary — management truth, not statutory filing.** BOS computes payroll, tracks loans, accrues leave, produces payslips and statutory-summary reports. The tenant's HR officer or chartered accountant files with tax authorities, pension funds, insurance bodies. | Charter §1.3 (BOS is not a statutory accounting/HR system); parallel to CN-5-001 §12 Accounting boundary. |
| **H6** | **Loans/advances via Obligation primitive (UI-08).** Employee loans are Obligation primitive (CN-4-011, kind: `employee_loan`) per CN-5-002 §9. HR creates the Obligation at approval; Cash disburses; HR payroll emits deduction events at payment time; Obligation reduces; Cash records repayment. UI-08 bounds enforced throughout. | Unified across engines (AR/AP/loans/advances/layby all Obligation). One primitive; multi-engine choreography. |
| **H7** | **Multi-engine choreography, no orchestrator.** HR → Cash → Accounting → (monthly) HR Payroll → Cash → Accounting. Each step is a P3 command_emitting subscription (CN-5-100); emergent behaviour produces the truth. No central "loan orchestrator" or "payroll coordinator" exists. | CN-5-100 L3 choreography-only; CN-5-002 §9 demonstrated pattern; resilience without central failure point. |

---

## 3. Engine Manifest

HR is a **multi-scope engine** per CN-5-101 §6: `tenant` default with explicit `site`-scope overrides for site-anchored operations (attendance, leave requests at site, site-initiated loan requests).

```yaml
engine_id:        hr
display_name:     HR & Payroll Engine
version:          1
scope_policy:     tenant                                # H1 — employees are tenant identity
primitives_used:
  - party                                               # employee records (CN-4-011)
  - workflow                                            # employment / leave / payroll lifecycles
  - approval                                            # leave / loan / payroll / termination gates
  - obligation                                          # employee loans (kind: employee_loan)

commands:
  # Employee lifecycle (tenant)
  - id: hr.employee.register.request                    scope_ref: tenant
  - id: hr.employee.attributes_update.request           scope_ref: tenant
  - id: hr.employee.deactivate.request                  scope_ref: tenant

  # Roles & assignments
  - id: hr.role.define.request                          scope_ref: tenant
  - id: hr.role.update.request                          scope_ref: tenant
  - id: hr.role.deactivate.request                      scope_ref: tenant
  - id: hr.assignment.create.request                    # scope_ref: site (assignment is site-anchored)
  - id: hr.assignment.end.request                       # scope_ref: site
  - id: hr.assignment.transfer.request                  scope_ref: tenant     # cross-site transfer

  # Compensation
  - id: hr.compensation.set.request                     scope_ref: tenant
  - id: hr.compensation.revise.request                  scope_ref: tenant

  # Leave
  - id: hr.leave.balance.set.request                    scope_ref: tenant     # opening balance at onboarding
  - id: hr.leave.request.request                        # scope_ref: site (employee requests at site)
  - id: hr.leave.approve.request                        # scope_ref: site (or tenant per threshold)
  - id: hr.leave.cancel.request                         # scope_ref: site

  # Attendance
  - id: hr.attendance.record.request                    # scope_ref: site
  - id: hr.overtime.record.request                      # scope_ref: site

  # Periodic accrual (N2 — HR-owned namespace; system:scheduler is actor)
  - id: hr.accrual.run.request                          scope_ref: tenant     # system:scheduler submits

  # Payroll
  - id: hr.payroll.run.request                          scope_ref: tenant
  - id: hr.payroll.approve.request                      scope_ref: tenant     # authorised role; pre-disbursement gate
  - id: hr.payroll.mark_paid.request                    scope_ref: tenant     # system-submitted after Cash confirmation

  # Loans (HR-side; Cash actually disburses)
  - id: hr.loan.request.request                         # scope_ref: site
  - id: hr.loan.approve.request                         scope_ref: tenant
  - id: hr.loan.reject.request                          scope_ref: tenant
  - id: hr.loan.write_off.request                       scope_ref: tenant     # on termination or default

  # Termination
  - id: hr.termination.initiate.request                 scope_ref: tenant
  - id: hr.termination.finalize.request                 scope_ref: tenant

emits:
  # Employee
  - hr.employee.registered.v1
  - hr.employee.attributes_updated.v1
  - hr.employee.deactivated.v1

  # Roles & assignments
  - hr.role.defined.v1
  - hr.role.updated.v1
  - hr.role.deactivated.v1
  - hr.assignment.created.v1
  - hr.assignment.ended.v1
  - hr.assignment.transferred.v1

  # Compensation
  - hr.compensation.set.v1
  - hr.compensation.revised.v1

  # Leave
  - hr.leave.balance.set.v1
  - hr.leave.requested.v1
  - hr.leave.approved.v1
  - hr.leave.rejected.v1
  - hr.leave.cancelled.v1
  - hr.leave.accrued.v1                                 # periodic accrual

  # Attendance
  - hr.attendance.recorded.v1
  - hr.overtime.recorded.v1

  # Payroll
  - hr.payroll.computed.v1                              # → CN-5-001 #10 Accounting (accrual journal)
  - hr.payroll.approved.v1                              # gate passed; → CN-5-002 subscribes (N3) to schedule disbursement
  - hr.payroll.paid.v1                                  # → CN-5-001 #11 Accounting (settlement); workflow status

  # Per-employee deduction events — emitted AT .paid (N4)
  - hr.payroll.deduction.applied.v1                     # → CN-5-002 §9 subscribes (loan repayment) — N1 namespace fix

  # Loans
  - hr.loan.requested.v1
  - hr.loan.approved.v1                                 # → CN-5-002 §9 subscribes (disburse_employee_loan)
  - hr.loan.rejected.v1
  - hr.loan.disbursed.v1                                # workflow status after Cash confirmation
  - hr.loan.write_off.requested.v1                      # → CN-5-001 + CN-5-002 compensation paths

  # Termination
  - hr.termination.initiated.v1
  - hr.termination.finalized.v1

subscribes_to:
  # System scheduler for periodic accrual (N2 — HR command, system actor)
  # The scheduler submits hr.accrual.run.request directly; not a separate subscription.

  # Cash payment confirmations (HR-initiated outflows — consolidated handler N1 pattern)
  - {event_type: cash.tender.disbursed.v1,              version: 1, handler: handle_cash_outflow_confirmation, kind: command_emitting, scope_ref: tenant}
    # Internal routing on payload.origin:
    #   employee_loan_disbursement → mark loan disbursed → emit hr.loan.disbursed.v1
    #   payroll_payment            → submit hr.payroll.mark_paid.request → emit hr.payroll.paid.v1

  # Vertical work events feeding compensation inputs (CTR-030 expansion)
  - {event_type: <vertical>.commission_earned.v1,       version: 1, handler: record_compensation_input, kind: command_emitting, scope_ref: tenant}
  - {event_type: <vertical>.piece_completed.v1,         version: 1, handler: record_compensation_input, kind: command_emitting, scope_ref: tenant}
  - {event_type: cash.tip.distributed.v1,               version: 1, handler: record_tip_distribution,   kind: command_emitting, scope_ref: tenant}

requires:
  - cash.tender.disbursed.v1                            # cannot confirm payroll/loan disbursement without Cash confirmation
```

### N1 — Event Namespace Fix

The deduction event is `hr.payroll.deduction.applied.v1` (Charter §8.1 — `<engine>.<noun>.<verb>.v<n>`, engine = `hr`). CN-5-002 §9 references the older `payroll.deduction.applied.v1` name; that wording will be aligned during Term 5's broader coherence pass (e.g., when CN-5-103 Universal Event Glossary catalogs all events). No formal amendment to CN-5-002; cosmetic naming alignment.

### N2 — Scheduler Namespace Fix

Periodic accrual uses **`hr.accrual.run.request`** submitted by `system:scheduler` actor (CN-4-007). HR owns the command and emits the resulting `hr.leave.accrued.v1` events. There is no `system.scheduler.*` event namespace (parallel to CN-5-001 N1 for depreciation).

### N3 — Cash Subscribes Approved, Not Computed

Per the disbursement-gate principle: Cash schedules payroll disbursement on `hr.payroll.approved.v1` (post-gate), not on `hr.payroll.computed.v1`. Computation establishes the numbers; approval authorises the cash outflow; only then does Cash act. This prevents disbursement of a computed-but-unapproved payroll.

### N4 — Deduction-Obligation Reduction Ordering

`hr.payroll.deduction.applied.v1` events are emitted **at `hr.payroll.paid.v1` time** — atomically with the paid event via CN-4-004 §2 multi-event emission. Reducing the Obligation outstanding **before** Cash disbursement confirmation would silently over-reduce if disbursement subsequently fails. Tying deduction events to payment-confirmation keeps the Obligation reduction safe.

---

## 4. Employee Management

### Employee Record (H1)

Party primitive (CN-4-011) for identity; HR engine projection for employment state.

```yaml
employee:
  party_ref:               <Party primitive ID>
  employee_number:         <tenant-unique identifier; pack-formatted>
  national_id_or_tin:      <jurisdiction-specific>
  contact:                 {phone, email, physical_address_ref}
  emergency_contact:       <party_ref>
  active_status:           active | on_leave | terminated | deactivated
  hire_date:               <CN-4-014 business_date>
  termination_date:        <null until terminated>
  attributes:
    employee_type:         permanent | contract | casual | intern   # pack-defined
    grade_level:           <pack-defined or tenant-defined>
    tax_id_local:          <jurisdiction-specific>
    pension_member_id:     <jurisdiction-specific>
    nhif_member_id:        <jurisdiction-specific>
    bank_account_ref:      <for direct salary payments via Cash + CTR-031 banking adapter>
```

### Lifecycle

```
[registered] → [active] ↔ [on_leave] → [active] → [terminated]
                                                 → [deactivated]
```

`on_leave` is a transient state during approved leave; reverts to `active` automatically on leave end date or explicit return.

---

## 5. Roles, Assignments, Compensation (H2)

### Role Definition

```yaml
role:
  role_id:           <unique>
  role_name:         "Cashier" | "Manager" | "Accountant" | …
  authority_tags:    [<permission tags used by other engines for role-based command authorisation>]
  pack_required:     boolean       # some roles are pack-mandated (e.g., compliance officer in regulated industries)
```

Roles are tenant-wide (same role names across all sites).

### Assignment (Site-Anchored)

```yaml
hr.assignment.created.v1
  payload:
    assignment_id:       <unique>
    employee_ref:        <employee>
    role_ref:            <role>
    site_id:             <primary site>
    effective_from:      <business_date>
    effective_until:     <null for open-ended>
    fte_fraction:        1.0
    additional_sites:    [<site_id>]      # Q9 — multi-site staff (e.g., regional manager)
```

Versioned: changes to role / FTE / sites emit new assignment events; effective dates anchor to business calendar.

### Compensation

```yaml
hr.compensation.set.v1
  payload:
    employee_ref:        <employee>
    compensation_kind:   salary | hourly | commission | piece_rate | hybrid
    base_amount:         decimal
    base_period:         monthly | weekly | hourly
    currency:            <tenant functional currency — UI-06>
    commission_rules:    {percent, basis, scope: site | tenant}     # for commission
    piece_rate_rules:    {amount_per_piece, item_categories: […]}   # for piece_rate
    effective_from:      <business_date>
    pack_version_ref:    <D-009>
```

Revisions emit new `hr.compensation.revised.v1`; full history retained for replay and dispute.

---

## 6. Leave Management (H4)

Pack-driven per jurisdiction.

### Leave Types & Accrual Rules (Pack-Defined, CTR-029 Expansion)

```yaml
# In pack content
hr_leave:
  types:
    - {code: annual,    accrual_per_month: 1.75, max_balance: 30,    carryover: 5}
    - {code: sick,      accrual_per_month: 1.0,  max_balance: 14,    carryover: 0,  evidence_required: medical_cert}
    - {code: maternity, fixed_entitlement: 84,   conditions: {gender: female, hire_tenure_days: 365}}
    - {code: paternity, fixed_entitlement: 7}
    - {code: bereavement, fixed_entitlement: 5}
  encashment_on_termination:  true     # whether unused annual leave is paid out at termination
  accrual_frequency:           monthly  # scheduler trigger frequency
```

### Per-Employee Balance Projection

Keyed `(employee_ref, leave_type) → current_balance`.

### Monthly Accrual via Scheduler (N2)

```
system:scheduler submits hr.accrual.run.request {tenant_id, accrual_kind: leave, period}
  → HR engine handler:
    For each active employee × each leave_type:
      compute accrual_amount per pack rule
      respect max_balance (excess capped or carried forward per pack)
    Emit hr.leave.accrued.v1 per (employee, leave_type, period)
  → balance projection updates
```

### Leave Request Lifecycle (Workflow Primitive)

```
[requested] ──approve──► [approved] ──leave_starts──► [active] ──leave_ends──► [consumed]
        │
        ├──reject──► [rejected]
        │
        └──cancel──► [cancelled]
```

Balance reduces only when the leave is **consumed** (employee returns from leave) — not at approval. This handles cancellations cleanly.

---

## 7. Attendance

Site-scope. Employees clock in / out at the site they're assigned to (or one of their `additional_sites` if multi-site).

```yaml
hr.attendance.record.request:
  scope:             site
  payload:
    employee_ref, site_id, clock_event: in | out, timestamp, business_date

hr.overtime.record.request:
  scope:             site
  payload:
    employee_ref, site_id, hours, overtime_kind: regular | weekend | holiday | night, business_date
```

For **salaried** employees, attendance is informational (payroll computes per salary regardless). For **hourly / piece-rate** employees, attendance and work events drive payroll computation.

Pack rules govern overtime multipliers (e.g., 1.5x regular OT, 2.0x holiday).

---

## 8. Payroll Periods & Computation (H3)

### Period Definition (Pack-Configurable, CTR-029 Expansion)

```yaml
# In pack content
hr_payroll:
  period_frequency:        monthly                # pack may set weekly | bi-weekly per jurisdiction
  period_cutoff_day:       25                     # of the month; activities after this go to next period
  payment_date_target:     last_business_day_of_month
  payroll_cycle_open_window_days: 7               # how long the run stays open for corrections before approval
```

### Payroll Run Flow

```
Step 1 — Initiate
  hr.payroll.run.request {period}
  → workflow state: payroll_computing
  → engine aggregates inputs per employee:
      base_pay              (per compensation_kind)
      hours / pieces        (from attendance + work events)
      commission accumulator (from <vertical>.commission_earned.v1)
      piece-rate accumulator (from <vertical>.piece_completed.v1)
      tip distribution      (from cash.tip.distributed.v1; per pack tax treatment)
      overtime              (per pack multipliers)
      leave payments        (if any due in this period)
      gross_pay = sum

Step 2 — Compute Deductions (pack-driven, §9)
  For each employee, run pack's deduction rules:
    paye_tax
    pension_employee
    nhif_or_health
    loan_instalment       (per outstanding Obligation, kind: employee_loan; planned amount per schedule)
    voluntary_deductions  (tenant-configured: union dues, savings, etc.)
    other_statutory       (training levy, etc.)
  Compute employer contributions:
    pension_employer
    employer_levies

Step 3 — Net Pay
  net_pay = gross_pay - sum(employee_deductions)

Step 4 — Emit
  hr.payroll.computed.v1 {period, employees: [{employee_ref, gross, deduction_breakdown, net, employer_contributions}]}
  → CN-5-001 #10 auto-journals: Dr Salary Expense (gross); Cr Salary Payable; Cr Tax W/H; Cr Pension W/H; etc.

Step 5 — Authorise (gate before disbursement)
  hr.payroll.approve.request (authorised role; pack-driven amount-band thresholds)
  → hr.payroll.approved.v1
  → CN-5-002 subscribes (N3 — post-gate) → schedules per-employee disbursement

Step 6 — Cash Disbursement
  Cash per employee: cash.tender.disbursed.v1 (origin: payroll_payment)
  (or aggregated bank file via Term 7 banking adapter — CTR-031)

Step 7 — HR Confirms (Atomic with Deduction Emission, N4)
  HR handle_cash_outflow_confirmation subscribes cash.tender.disbursed.v1 (origin: payroll_payment)
  Once all employees' disbursements are confirmed for the period:
    hr.payroll.mark_paid.request submitted
    Bus atomically emits (CN-4-004 §2):
      1. hr.payroll.paid.v1
      2. hr.payroll.deduction.applied.v1  per employee per loan deduction
    → CN-5-001 #11 auto-journals: Dr Salary Payable; Cr Cash (per employee)
    → CN-5-002 §9 subscribes hr.payroll.deduction.applied.v1 → Obligation reduces (UI-08); records loan repayment side
  → workflow state: paid
```

### Why N4 Ordering Matters

If `hr.payroll.deduction.applied.v1` were emitted at computation time (Step 4), the Obligation would reduce before Cash disbursement was confirmed. If subsequent disbursement failed (network, adapter error, insufficient funds), the loan would silently show as paid down without the cash having moved. Tying deduction emission to confirmed payment (Step 7, atomic with `paid.v1`) keeps the Obligation reduction safe — it happens only when the money actually moved.

### Period Close + UI-05

Once `hr.payroll.paid.v1` emitted and the corresponding accounting period closes (via CN-5-104 choreography), UI-05 inviolability locks: no further payroll events with `effective_date` in that period. Corrections post forward (current period) with causation back.

---

## 9. Statutory Deductions (H4 — Pack-Driven)

Pack schema (CTR-029 expansion):

```yaml
# In pack content
hr_statutory_deductions:
  paye:
    name:               "PAYE Tax"
    employer_remittance: monthly
    bands:                                       # progressive
      - {min: 0,         max: 270000,    rate: 0,    fixed_amount: 0}
      - {min: 270001,    max: 520000,    rate: 8,    fixed_amount: 0}
      - {min: 520001,    max: 760000,    rate: 20,   fixed_amount: 20000}
      - {min: 760001,    max: 1000000,   rate: 25,   fixed_amount: 68000}
      - {min: 1000001,   max: null,      rate: 30,   fixed_amount: 128000}
    dependent_allowances: {…}
    account_code:        "2310"

  pension_employee:
    name:               "Employee Pension"
    rate_percent:        10
    cap_amount:          null
    employer_remittance: monthly
    account_code:        "2330"

  pension_employer:
    name:               "Employer Pension"
    rate_percent:        10
    account_code:        "5180"                  # employer pension expense

  nhif_or_health:
    name:               "NHIF"
    bands_or_flat:       bands
    bands:               [{min: 0, max: 5999, fixed_amount: 150}, …]
    account_code:        "2340"

  training_levy:
    name:               "Skills Development Levy"
    rate_percent:        4.5
    paid_by:             employer
    account_code:        "5190"
```

Pack rotation governs rate changes (D-009 freeze): historical payrolls remain under their original pack version; new periods use current rates.

---

## 10. Employee Loan Choreography (H6, H7)

Recap of the multi-engine flow from CN-5-002 §9, from HR's perspective. The full mechanism is documented in CN-5-002 §9; CN-5-005 documents HR's role and the write-off path.

### Active Loan Flow

```
1. Employee requests via hr.loan.request.request → hr.loan.requested.v1 (workflow: loan_pending_approval)
2. Authorised role: hr.loan.approve.request → hr.loan.approved.v1
   - Obligation primitive creates: kind: employee_loan, original_amount: principal, outstanding: principal
3. CN-5-002 subscribes hr.loan.approved.v1 → disburses (CN-5-002 §9 Step 2)
4. Cash emits cash.tender.disbursed.v1 (origin: employee_loan_disbursement)
5. HR handle_cash_outflow_confirmation (internal routing on origin) → emits hr.loan.disbursed.v1
   - workflow: loan_active
6. Each payroll period:
   - Step 2 includes loan instalment in deduction computation
   - Step 7 emits hr.payroll.deduction.applied.v1 atomically with hr.payroll.paid.v1 (N4)
   - CN-5-002 §9 subscribes → records loan repayment; Obligation reduces (UI-08 bounds)
7. When outstanding = 0: Obligation transitions to settled; HR projection updates; workflow: loan_repaid
```

### Write-Off on Termination or Default (UI-02 Compensation)

```
hr.loan.write_off.request (authorised role, dual-actor)
  payload: obligation_ref, reason_ref, remaining_outstanding, business_date
  → hr.loan.write_off.requested.v1 emitted with:
      compensates_event_id: <event_id of hr.loan.approved.v1>   (UI-02)
      causation_id:         <write-off command derivative>        (UI-01)
  → Obligation primitive: outstanding adjusted via primitive compensation; transitions to written_off
  → CN-5-001 auto-journals: Dr Bad Debt Expense; Cr Employee Loan Receivable
  → CN-5-002 records the compensation symmetrically
```

The write-off is auditable, traceable, and irrevocable — same compensation pattern as every other UI-02 application across BOS.

---

## 11. Commission / Piece-Rate / Tip Integration (Pattern B Analog)

Vertical engines that produce work-driven compensation events feed HR via subscriptions, analogous to CN-5-003 Pattern B for Inventory.

### Subscribed Events and Effects

| Vertical event | Effect at HR | Pack tax treatment |
|----------------|--------------|---------------------|
| `<retail>.commission_earned.v1 {employee_ref, sale_ref, commission_amount}` | Aggregates into employee commission accumulator for the current period | Pack declares: folded into PAYE base, or taxed at special rate |
| `<workshop>.piece_completed.v1 {employee_ref, piece_ref, piece_count, unit_rate}` | Aggregates into piece-rate accumulator | Folded into gross pay; PAYE applies |
| `cash.tip.distributed.v1 {staff_ref, amount}` (from CN-5-002 §16) | Aggregates into tip income for that period | Pack-driven: pass-through (non-taxable) OR taxable wages |

### Aggregation

The handler `record_compensation_input` and `record_tip_distribution` update employee-period accumulators in HR's projection. At payroll computation (Step 1), these accumulators feed `gross_pay` per the compensation_kind and pack tax-treatment rules.

### Period Boundary

Inputs received after the payroll period cutoff (per pack `period_cutoff_day`) are deferred to the next period. The event's `business_date` (CN-4-014) determines which period it belongs to; events received late but business-dated within an open period are accepted; events business-dated within a closed period are rejected per UI-05.

---

## 12. Termination & Final Settlement (Q6)

```
Step 1 — Initiate
  hr.termination.initiate.request {employee_ref, termination_date, reason_ref, last_working_day}
  → hr.termination.initiated.v1 emitted
  → workflow: termination_in_progress
  → employee status: remains active until finalize (allows partial-period work)

Step 2 — Compute Final Pay (within finalize)
  hr.termination.finalize.request (authorised role, dual-actor audit)
  Engine computes per pack termination rules:
    Positives:
      outstanding_salary:        prorated to last_working_day
      leave_payout:              unused leave × daily_rate per pack encashment rules
      severance:                 per pack / contract rules
      bonus_owed:                if any
    Negatives:
      outstanding_loan:          remaining Obligation outstanding (per H6)
      other_owed_to_business:    advances not yet recovered
    Final net = sum(positives) − sum(negatives)

Step 3 — Outstanding Loan Resolution (Q7)
  IF outstanding_loan ≤ sum(positives):
    Auto-deduct from final pay
    Emit hr.payroll.deduction.applied.v1 (atomic with finalize) → Obligation settles fully
  ELIF outstanding_loan > sum(positives) but partial possible:
    Auto-deduct what positive pay covers; emit hr.payroll.deduction.applied.v1 for partial
    Remaining outstanding triggers hr.loan.write_off.request (authorised role decision required)
    Workflow holds until write-off decision made
  ELSE (no positive pay):
    Full loan amount → escalate to authorised role for write-off decision

Step 4 — Emit
  hr.termination.finalized.v1
    payload:
      outstanding_salary, leave_payout, severance, bonus,
      loan_settled, loan_written_off, net_final_pay,
      business_date
  → workflow: terminated
  → employee status: terminated
  → CN-5-001 auto-journals via existing payroll mappings
  → CN-5-002 subscribes for cash disbursement of net_final_pay
```

### Authorised Role Hard Gate

Final pay disbursement is gated on loan-resolution-decision-recorded. The workflow holds in `termination_pending_loan_decision` state until an authorised role records the decision (auto-deduct, partial + write-off, full write-off). This prevents silent loss to the business.

---

## 13. Charter §1.3 Boundary — Management Truth, Not Statutory Filing (H5)

Explicit table (parallel to CN-5-001 §12):

| What BOS HR/Payroll does | What it does NOT do |
|--------------------------|---------------------|
| Computes payroll per pack-driven statutory rules | Files PAYE returns with tax authority |
| Tracks loans, accrues leave, produces payslips | Submits pension contributions to pension fund |
| Produces statutory-summary reports (PAYE summary, NHIF, pension) | Files those reports with regulators |
| Generates payroll journal events for Accounting | Replaces the HR officer or chartered accountant |
| Honours freeze doctrine — historical payrolls under historical rules | Re-compute past periods retroactively |
| Maintains employment history for audit + dispute | Issue legally-binding employment certificates (HR officer signs) |

The tenant's HR officer or chartered accountant uses BOS data for filings. BOS guarantees the data is complete, balanced, statutory-compliant **as configured by the approved pack**. The officer/accountant files; BOS does not.

No `hr.statutory_filing.submitted.v1` event exists. Filing-format generation (PAYE file, pension file, NHIF file) is **CTR-034 Open Item** — possibly a Term 7 export adapter when tenants demand it; not in v1.

---

## 14. Cross-Engine Subscription Map

| HR event | Subscribed by | Effect |
|----------|---------------|--------|
| `hr.payroll.computed.v1` | CN-5-001 #10 Accounting | Auto-journal accrual: Dr Salary Expense; Cr Salary Payable; Cr Tax W/H; Cr Pension W/H; etc. |
| `hr.payroll.approved.v1` (N3) | CN-5-002 | Schedules per-employee cash disbursement |
| `hr.payroll.paid.v1` | CN-5-001 #11 Accounting | Auto-journal settlement: Dr Salary Payable; Cr Cash (per employee) |
| `hr.payroll.deduction.applied.v1` (N1, N4) | CN-5-002 §9 | Records loan repayment; Obligation reduces (UI-08) |
| `hr.loan.approved.v1` | CN-5-002 §9 | Disburses loan from configured till |
| `hr.loan.write_off.requested.v1` | CN-5-001 | Bad debt journal: Dr Bad Debt Expense; Cr Employee Loan Receivable |
| `hr.loan.write_off.requested.v1` | CN-5-002 §9 | Compensation propagation (UI-02 symmetry) |
| `hr.termination.finalized.v1` | CN-5-002 | Final pay disbursement |
| `hr.attendance.recorded.v1`, `hr.overtime.recorded.v1` | CN-5-006 Reporting | Workforce metrics |
| `hr.leave.accrued.v1`, `hr.leave.approved.v1` | CN-5-006 Reporting | Leave-balance reporting |

| HR subscribes to | Handler | Effect |
|------------------|---------|--------|
| `cash.tender.disbursed.v1` (filtered by origin) | `handle_cash_outflow_confirmation` (consolidated) | Internal routing: `employee_loan_disbursement` → emit `hr.loan.disbursed.v1` (workflow: loan_active); `payroll_payment` → once all employees confirmed for period, submit `hr.payroll.mark_paid.request` → atomic emission of `hr.payroll.paid.v1` + `hr.payroll.deduction.applied.v1` per loan (N4) |
| `<vertical>.commission_earned.v1` | `record_compensation_input` | Aggregate into employee commission accumulator |
| `<vertical>.piece_completed.v1` | `record_compensation_input` | Aggregate into piece-rate accumulator |
| `cash.tip.distributed.v1` | `record_tip_distribution` | Aggregate per pack tax treatment |

Periodic accrual is **not** a subscription — `system:scheduler` directly submits `hr.accrual.run.request` to HR's command bus.

---

## 15. UI Invariants Enforcement

| Invariant | Application in CN-5-005 |
|-----------|------------------------|
| **UI-01** (causation chain) | Every payroll deduction event chains back to the payroll run + paid event; every loan repayment chains to the payroll deduction; every loan disbursement chains to the loan approval. Audit walks unambiguously. |
| **UI-05** (closed-period inviolability) | Once a payroll period is closed (via CN-5-104 choreography), no events with `effective_date` in that period accepted. Corrections post forward (current period) with causation back. |
| **UI-06** (functional currency) | Payroll runs in tenant functional currency; multi-currency employees (rare — expatriate staff paid in USD) require pack opt-in + FX bridging per CN-5-001 §11. |
| **UI-08** (obligation bounds) | Employee-loan Obligation outstanding ∈ [0, original_amount] at every step. Bus rejects deductions that would breach. Compensation paths for write-off preserve symmetry. N4 ordering prevents over-reduction on failed disbursement. |

---

## 16. Example — Karakana Wafanyikazi (Year-Long Flow)

*Karakana ya Mzee Hassan in Mwanza/Mbeya serves as illustrative context per D-004 #4 (peer-technical audience). The example walks employment lifecycle, payroll cycle with statutory deductions, loan choreography, and termination across a fiscal year.*

**Employees** (4 active mid-year):
- Mzee Hassan — owner (not on payroll)
- Asha — Cashier, primary site Mwanza, salary TZS 450,000/mo
- Juma — Carpenter, primary site Mwanza, salary TZS 600,000/mo
- Said — Carpenter, primary site Mbeya (with additional_sites: [Mwanza]), salary TZS 550,000/mo

**Pack:** `tz-compliance-2026.07` (TFRS-TZ, fiscal year starts July).

### Onboarding (June 2026)

- Asha registered: `hr.employee.register.request` → Party + HR record.
- `hr.role.define.request` for Cashier (if not already).
- `hr.assignment.create.request {employee: Asha, role: Cashier, site: karakana-mwanza-001, fte: 1.0}` → `hr.assignment.created.v1`.
- `hr.compensation.set.request {kind: salary, base: 450000 TZS, period: monthly}` → `hr.compensation.set.v1`.
- `hr.leave.balance.set.request` for opening annual leave (5 carryover days).
- Similar for Juma and Said.

### August Loan — Asha Requests Advance

```
Asha submits via cashier UI:
  hr.loan.request.request {employee: Asha, principal: 50000, repayment_schedule: 5×10000 monthly}
  → hr.loan.requested.v1 (workflow: loan_pending_approval)

Manager approves:
  hr.loan.approve.request
  → hr.loan.approved.v1
  → Obligation primitive: kind: employee_loan, outstanding: 50000 TZS

CN-5-002 §9 subscribes:
  Cash submits cash.expense.record.request (kind: employee_loan_disbursement)
  → cash.tender.disbursed.v1 (origin: employee_loan_disbursement, amount: 50000)
  → CN-5-001 #4 journal: Dr Employee Loan Receivable; Cr Cash

HR's handle_cash_outflow_confirmation (origin: employee_loan_disbursement):
  → submits internal command; emits hr.loan.disbursed.v1
  → workflow: loan_active
```

### August Payroll Run

```
End of August — payroll cycle:
  hr.payroll.run.request {period: 2026-08} → workflow: payroll_computing

Per-employee computation (Asha, illustrative):
  base_pay:           450,000
  commission:         0 (cashier — no commission events)
  overtime:           0
  gross_pay:          450,000

  Pack deduction rules (tz-compliance-2026.07):
    paye_tax:          450000 in 270001-520000 band → (450000-270000) × 8% = 14,400
    pension_employee:  450000 × 10% = 45,000
    nhif:              per bands, ≈ 1,500
    loan_instalment:   10,000 (per Asha's loan schedule)
    voluntary:         0
  total_deductions:    70,900
  net_pay:             379,100

  Employer contributions:
    pension_employer:  45,000
    training_levy:     20,250 (4.5%)

(Similar for Juma and Said.)

hr.payroll.computed.v1 emitted (full employee breakdown)
  → CN-5-001 #10 journals: Dr Salary Expense 1,600,000 (gross sum);
                            Cr Salary Payable, Cr PAYE Payable, Cr Pension W/H, etc.

Manager approves:
  hr.payroll.approve.request
  → hr.payroll.approved.v1
  → CN-5-002 subscribes (N3); schedules per-employee disbursement

Cash disburses each employee:
  cash.tender.disbursed.v1 (origin: payroll_payment) × 3 (Asha, Juma, Said)
  HR handle_cash_outflow_confirmation collects all confirmations.
  When all 3 confirmed (typically same day):
    hr.payroll.mark_paid.request submitted
    Bus atomically emits (N4):
      1. hr.payroll.paid.v1
      2. hr.payroll.deduction.applied.v1 {employee: Asha, obligation_ref: <loan>, amount: 10000, period: 2026-08}
  → CN-5-001 #11 journals: Dr Salary Payable; Cr Cash (per employee net)
  → CN-5-002 §9 subscribes hr.payroll.deduction.applied.v1:
      Obligation outstanding: 50000 − 10000 = 40000 (UI-08 bound ✓)
      Records loan repayment side
```

### Months 2–5

Same pattern. Asha's loan reduces TZS 10,000/month. By January 2027, outstanding = 0. Obligation primitive transitions: `settled`. HR workflow → `loan_repaid`.

### April 2027 — Asha Resigns

```
hr.termination.initiate.request {employee: Asha, termination_date: 2027-04-15, reason_ref: voluntary_resignation, last_working_day: 2027-04-15}
  → hr.termination.initiated.v1
  → workflow: termination_in_progress

Two weeks notice; Asha works until April 15.

hr.termination.finalize.request (Mzee Hassan + HR officer, dual-actor):
  Engine computes (per pack termination rules):
    outstanding_salary:   15 days prorated = 15/30 × 450000 = 225,000
    leave_payout:         unused 7 days × daily_rate (15,000) = 105,000
    severance:            per contract — none for voluntary resignation
    bonus_owed:           none
    Positives total:      330,000

    outstanding_loan:     0 (fully repaid by January)
    Negatives total:      0

    Final net (gross):    330,000
    Less statutory deductions on the final pay: PAYE, NHIF, pension
    Final net (after):    ≈ 290,000

hr.termination.finalized.v1 emitted (full breakdown)
  → CN-5-001 journals via existing payroll mappings
  → CN-5-002 disburses final net pay
  → HR projection: Asha.active_status = terminated
```

### Year-End — Statutory Reports

Reporting Engine (CN-5-006) produces:
- PAYE summary per employee for the fiscal year
- Pension contribution summary (employee + employer)
- NHIF summary
- Skills development levy summary

Mzee Hassan exports → his HR officer files with TRA (PAYE), pension fund, NHIF. **BOS did not file** any of these. Charter §1.3 boundary held.

### Pack Upgrade Mid-Year

In April 2027, Tanzania publishes updated PAYE band thresholds effective May 1. Pack `tz-compliance-2027.05` approved. Payrolls before May 1 reference `tz-compliance-2026.07` and use old bands; payrolls May 1 and after reference new pack and use new bands (D-009 freeze). HR engine code unchanged; only pack content changed. Mzee Hassan's HR officer sees the new bands take effect automatically; old payrolls retain their original computation forever.

---

## 17. Three Whys

### Why does this matter?

Wafanyikazi (employees) are the workforce; payroll is the operational truth of how labour is compensated. Every business with staff has a payroll story — who was hired, what they earn, what taxes are withheld, what advances they received and repaid, when they took leave, what was their final settlement when they left. Without a unified engine, payroll is spreadsheets + memory + risk of error + risk of dispute + risk of regulatory penalty. CN-5-005 makes the full lifecycle event-sourced: every hire, every salary revision, every leave day, every payroll computation, every deduction, every loan, every termination — all auditable, replayable, and statutory-rule-compliant under the pack active at each moment. The HR officer's job becomes filing what BOS computed, not computing what to file.

### Why does it belong here (and not in Foundation)?

Foundation provides Party (identity), Workflow (lifecycle), Approval (gates), Obligation (loans), Document (payslips), and the Compliance evaluator (statutory rules). Foundation does not say "every employee has roles + compensation + leave balance," "payroll runs per period with pack-driven deductions," "loans are HR-approved + Cash-disbursed + Payroll-deducted in choreographed steps." Those are universal HR/payroll choices appropriate for any business with staff in any jurisdiction. CN-5-005 makes those choices. A specialised engine (gig-economy payments-per-task, military pay grades) could compose primitives differently — but that is not what universal HR/payroll is for general business.

### Why this design?

Party-plus-projection (H1) keeps identity clean while letting employment state evolve. Per-period payroll events (H3) make replay and freeze-doctrine compliance straightforward. Statutory-in-packs (H4) lets one engine serve TZ + KE + UG + etc. as packs evolve. Charter §1.3 explicit boundary (H5) keeps BOS honest — chartered HR officer files; BOS provides the data. Obligation-primitive loans (H6) reuse the unified primitive for AR/AP/loans/delivery — one mechanism across engines. Multi-engine choreography (H7) — HR → Cash → Accounting → next period — produces emergent truth without orchestrator. N3 + N4 ordering (Cash subscribes approved-not-computed; deductions emitted at paid-not-computed) prevents silent over-reduction on failures. Pattern B integration for commissions/tips lets vertical engines drive compensation inputs without HR knowing verticals exist. The whole engine is one cohesive answer to "how does any business, in any jurisdiction, manage its workforce and pay them honestly?"

---

## 18. Boundaries

| Topic | Lives in |
|-------|----------|
| Party, Workflow, Approval, Obligation primitive folds | CN-4-011 |
| Compliance DSL grammar + sandboxed evaluator | CN-4-015 |
| Pack content per jurisdiction (TFRS-TZ PAYE, IFRS pension rules, etc.) | Compliance packs authored under CTR-029 (Term 1) |
| Event envelope (causation_id, compensates_event_id, pack_version_ref) | CN-4-002 |
| Command bus + atomic multi-event emission | CN-4-004 |
| Clock protocol (business_date, period boundaries, accrual schedule) | CN-4-014 |
| Identity (system:scheduler principal) | CN-4-007 |
| Accounting payroll journal mappings | CN-5-001 §4 (#10 accrual, #11 settlement) |
| Cash loan disbursement and repayment mechanics; payroll cash disbursement | CN-5-002 §9 + §G |
| Tip distribution from passthrough till | CN-5-002 §16 |
| Subscription patterns + replay semantics + compensation kind | CN-5-100 |
| Scope policy (multi-scope per §6 pattern) | CN-5-101 |
| Cross-engine invariants (UI-01, UI-05, UI-06, UI-08) | CN-5-102 |
| Vertical commission / piece-rate / work event contracts | Term 6 via CTR-030 expansion |
| Tenant functional currency + site registries | Term 1 + Term 2 via CTR-027 |
| Filing-format generation (PAYE / pension / NHIF files) | Term 7 future (CTR-034 Open Item) |
| Reporting templates (payslip, statutory summaries, headcount, workforce metrics) | CN-5-006 + pack content |
| Statutory tax filing, pension submission, insurance submission | Outside BOS (chartered HR officer or accountant per Charter §1.3) |
| Legally-binding employment certificates (signed letters) | Outside BOS (HR officer signs) |
| Universal event glossary (eventual naming-alignment pass for hr.payroll.deduction.applied.v1, etc.) | CN-5-103 (future) |

---

## 19. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Pack content for hr_leave, hr_payroll, hr_statutory_deductions, termination_computation per jurisdiction | Term 1 via CTR-029 expansion | Schema in §6 + §8 + §9 + §12; per-jurisdiction content is Term 1's authorship |
| Vertical event contracts for commission / piece-rate / work events | Term 6 via CTR-030 expansion | Per-vertical payload schemas |
| Filing-format adapter (PAYE file, pension file, NHIF file generation) | Term 7 future via CTR-034 | Open Item per Charter §1.3 — BOS does not file; defer until clear demand |
| Naming alignment of `hr.payroll.deduction.applied.v1` references in CN-5-002 §9 | CN-5-103 (Universal Event Glossary) | N1 — cosmetic alignment in broader coherence pass |
| Authorised role policy (who may approve loans, payrolls, terminations) | Term 1 (governance) + per-tenant policy | Default: manager/owner roles; full governance per tenant |
| Payroll re-computation policy (when corrections are needed for an already-paid period) | Per-engine + Architect | Compensating events post forward (current period); re-running a past period requires explicit doctrine support |
| Multi-currency employee payment (expatriate staff paid in non-functional currency) | CN-5-001 + future FX engine | v1: tenant functional currency default; multi-currency deferred |
| Year-end statutory closing choreography (annual PAYE certificate, annual pension statement) | CN-5-104 (Period-Close Choreography) + Term 1 | Multi-engine close mechanics |
| Self-service payslip access (employee views own payslip) | Term 3 (UX) | CN-5-005 produces the data; Term 3 designs the surface |
| Performance budget for payroll computation at scale (large workforce, complex compensation) | Architect phase | Concept guarantees correctness; performance is Architect's |
| Phase 1 doctrine check for UI-08 across HR + Cash loan settlements | CN-4-019 + future CTR | Aligns with CN-5-102 §7 Phase 1 ramp-up; arrives with CN-5-005 ratification |
| Bank-file payroll disbursement (single bank transaction paying all employees) via Term 7 banking adapter | Future + CTR-031 expansion | Currently per-employee disbursement; bulk-file pattern is operational optimisation |

---

*— End of CN-5-005 —*
