# CN-5-105 — Tax-Aware Engines

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 5 — Universal Engines (final cross-cutting doc)
> **Status:** For Overseer review.
> **Governing decisions:** Charter §1.3 (BOS computes, accountant files — strengthened per N8); Law 5 (compliance is configured — tax rules in packs); D-009 (freeze doctrine — events under historical pack/standard); D-004 (neutrality — no tax law in engine code); UI-01 (causation chain on tax events); UI-05 (closed financial period inviolability); UI-07 (trial balance gate); UI-08 (Obligation bounds for tax obligations); **UI-11** (Closed Tax-Period Inviolability — added per CN-5-102 amendment in this commit).
> **CTRs:**
> - **OPEN (referenced):** CTR-043 (Term 5 → Term 1 — Tax pack content governance, opened per main commit `b8bae19`).
> - **OPEN expansions at merge:** CTR-029 (tax pack subsection: rates, exemptions, recovery rules, jurisdiction defaults, return templates, tax_balance_epsilon); CTR-014/015 (tax-advisor cost ceilings + AI Mode dashboard tax-suggestion surface).
> - **No new CTRs filed by this doc** (CTR-043 already opened by Overseer).
> - **Three cosmetic amendments in same commit:**
>   - CN-5-001 — manifest adds 5 tax_period events (`opened`, `close.initiated`, `close.rejected`, `closed`, `tax.assessed`)
>   - CN-5-010 — catalogue grows to 9 advisors (adds `tax-advisor` §4.9 + pack hooks)
>   - CN-5-102 — invariant catalog grows to 11 (adds UI-11 Closed Tax-Period Inviolability)
> **Glossary:** See `MASTER-GLOSSARY.md` — Tax Period, Tax Authority Obligation, `tax_treatment_ref`, `tenant_tax_profile`, Jurisdiction, Withholding Tax (WHT), Reverse-Charge VAT.
> **Depends on:** CN-4-002 (envelope); CN-4-004 (command bus + policy rejection); CN-4-011 (Obligation primitive — tax kinds per TX12); CN-4-014 (clock protocol — tax_period boundaries per pack tax_calendar); CN-4-015 (Compliance DSL — tax rule grammar + sandboxed evaluator); CN-4-021 (Saleable Line — `tax_treatment_ref` input mechanism); CN-5-001 (Accounting Engine — owns `accounting.tax_period.*` + `accounting.tax.*` events; resolves `tax_treatment_ref` at journal time); CN-5-002 (Cash — settles tax authority Obligations); CN-5-004 (Procurement — input VAT recognition + WHT-on-services); CN-5-005 (HR — PAYE computation); CN-5-006 (Reporting — tax return Statement Documents per N3); CN-5-007 (Promotion — discount tax_treatment_ref per PR6); CN-5-009 (Checkout — output VAT at settlement per K3); CN-5-010 (AI Advisors — tax-advisor §4.9 added per amendment); CN-5-100 (subscription patterns); CN-5-101 (scope policy — tenant + jurisdiction); CN-5-102 (UI invariants — UI-01/05/07/08/11); CN-5-103 (event glossary — naming compliance for `accounting.tax_*` events); CN-5-104 (financial period close — distinct parallel mechanism).
> **Boundaries:** Compliance DSL grammar → CN-4-015; tax pack content per jurisdiction (rates, exemptions, recovery, templates) → Term 1 via CTR-043 + CTR-029; `tax_treatment_ref` input field shape → CN-4-021; per-engine `tax_treatment_ref` setting → respective engine doc (CN-5-009 for Checkout, CN-5-004 for Procurement, CN-5-005 for HR, CN-5-007 for Promotion); Statement Document issuance mechanics → CN-5-006 §5/§8 + CN-4-012; tax authority filing/submission → outside BOS (chartered accountant / tax officer per Charter §1.3 + N8); multi-jurisdiction full implementation → v2 future.

---

## 1. Purpose & Boundary

CN-5-105 is the **final cross-cutting doc of Term 5**. It wires tax-aware behaviours into the nine engines previously defined, completing Term 5's coverage of universal business operations. The mechanism rests on `tax_treatment_ref` (CN-4-021 input shape) flowing through engines, resolved by Accounting at journal time via pack-driven rules (CN-4-015), accumulated as tax Obligations (CN-4-011), settled by Cash, and reported as Statement Documents (CN-5-006 N3) at tax-period close.

### BOS DOES vs BOS DOES NOT (per N8 — Charter §1.3 reinforcement)

| BOS DOES | BOS DOES NOT |
|----------|--------------|
| Compute tax obligations per pack rules | File returns with tax authority |
| Generate Statement Documents (VAT return, PAYE certificate) as audit-defensible artefacts | Submit Statements to TRA / KRA / URA / etc. on tenant's behalf |
| Track `tax_authority_payable` / `tax_authority_receivable` Obligations | Pay tax authority directly (cash settlements are the tenant's authorised action via Cash Engine) |
| Send compliance reminders via `tax-advisor` (Phase 1) | Act as authorised tax agent or representative |
| Provide audit-defensible records (hash-frozen Statements, replayable computation) | Represent tenant in tax disputes or audits |
| Aware tenant of changing tax rules via pack updates (D-009 freeze respected) | Interpret tax law — pack content is **chartered-accountant-authored**, NOT BOS-authored |

**The line is operational, not aspirational.** A tenant's chartered accountant downloads Statement Documents from BOS, reviews them, files with the relevant authority outside BOS. BOS never holds itself out as an agent of the tenant before any tax authority.

### Scope (in)

- Cross-engine tax-aware wiring (Checkout / Procurement / HR / Promotion / Cash / Accounting / Reporting / Advisors)
- `tax_treatment_ref` resolution chain (per-engine catalogue)
- Per-tax-kind wiring: Output VAT, Input VAT (recoverable/non-recoverable), Withholding Tax (PAYE + WHT on services), Reverse-Charge VAT, Excise/Levies
- **`tenant_tax_profile`** mechanism (per N7) — informal-sector vs VAT-registered tenant defaults
- Tax-period close choreography (distinct from CN-5-104 financial close)
- Tax Return Statement Documents (per CN-5-006 N3)
- Multi-jurisdiction framework readiness (v1 single-jurisdiction; v2 deferred)
- Mid-period rate change handling (pack version-based deterministic)
- Tax exemptions + special schemes (pack-only)
- Tax-advisor integration (per CN-5-010 amendment)
- UI-11 Closed Tax-Period Inviolability (per CN-5-102 amendment)
- Tax-period failure modes (Q11/Q13)

### Scope (out)

- Compliance DSL grammar (CN-4-015)
- Tax pack content per jurisdiction (Term 1 via CTR-043 + CTR-029)
- Tax law interpretation (chartered accountants — out of BOS)
- Filing / submission to tax authorities (Charter §1.3)
- Multi-jurisdiction full implementation (v2 deferred per Q8)
- Architect-phase performance optimisations

---

## 2. Doctrine — TX1–TX12

| # | Law | Source / why |
|---|-----|--------------|
| **TX1** | **Tax = pack-content + engine-mechanism.** Pack declares rules (rates, thresholds, exemptions, formulas, recognition timing); engines apply mechanically. No tax law in engine code. | Law 5; D-004 neutrality; legal defensibility (chartered-accountant-authored content). |
| **TX2** | **`tax_treatment_ref` is an input mechanism** per CN-4-021. Set by source engine at originating event; resolved by Accounting at journal time; respected by Reporting at Statement time. | CN-4-021; flows like other primitive input refs; consistent pattern. |
| **TX3** | **Tax events follow source-event causation.** Tax journals are derivative events with `causation_id` to originating economic event (sale, purchase, payroll). UI-01 chain auditable. | UI-01 (CN-5-102); full audit trail from tax obligation back to underlying transaction. |
| **TX4** | **Tax rates are pack-versioned per D-009 freeze.** Mid-period rate change = new pack version effective_date; events before use old rate; events after use new. Replay reproduces under correct pack version. | D-009; CN-4-002 `pack_version_ref`; deterministic. |
| **TX5** | **Output VAT** recorded at Checkout settlement (per `checkout.settled.v1` derivative `accounting.journal.posted.v1`); creates `tax_authority_payable_output_vat` Obligation (per TX12). | Economic recognition at sale moment; CN-5-009 K3 (tax at settlement, pack-frozen). |
| **TX6** | **Input VAT** recorded at Procurement (per `procurement.invoice.recorded.v1`); recoverable per pack creates `tax_authority_receivable_input_vat` Obligation OR non-recoverable posts to expense. | CN-5-004 + CN-5-001 #6; per-item recoverability via pack (some items non-recoverable per jurisdiction). |
| **TX7** | **Withholding tax** (PAYE, WHT on services) computed at deduction time; creates distinct `wht_payable` Obligation to tax authority — settlement separate from gross-transaction Obligation. | CN-5-005 §10 PAYE; CN-5-004/5-002 WHT-on-services; per-employee + per-supplier tracking. |
| **TX8** | **Reverse-charge VAT** — pack rule per jurisdiction determines applicability; engine records simultaneous output (to authority) + input (recoverable) per same event; net effect = zero if fully recoverable. | OECD model VAT; cross-border B2B services common pattern; pack governs per jurisdiction. |
| **TX9** | **Tax-period vs financial-period are independent.** Pack declares both via `pack.tax_calendar` + `pack.period_calendar`. Tax-period close = separate Statement bundle + separate 6-phase choreography parallel to CN-5-104 financial close. | Monthly VAT periods often align with monthly financial close, but annual PAYE certificate, quarterly excise, etc. need own close mechanism. |
| **TX10** | **Multi-jurisdiction tenant** — pack declares primary + secondary jurisdictions; transactions tagged `jurisdiction_ref`; tax journals scoped per jurisdiction; reporting aggregates per jurisdiction. Framework hooks in v1; full implementation v2. | Rare for SME (BOS target audience); v1 framework readiness only — engines emit `jurisdiction_ref: <primary>` implicit. |
| **TX11** | **`tax-advisor`** (Phase 1 in CN-5-010 catalogue per amendment) — pre-filing review, anomaly flags, compliance reminders. Suggestions describe what tenant/accountant should DO with BOS-provided artefacts; never act on tenant's behalf (per N8). | Operational support; bounded by Law 3 (advisory-only) + Charter §1.3 (no filing). |
| **TX12** | **Tax obligations use Obligation primitive (CN-4-011).** Four kinds: `tax_authority_payable` (output VAT owed; excise; levies), `tax_authority_receivable` (input VAT net credit), `wht_payable` (PAYE + WHT-on-services), `tax_assessment_payable` (backdated audit adjustment per Q13). UI-08 bounds inherit. | Unifies tax with AR/AP/loans/cost-share/advance/layby — all Obligations. **Footnote on `wht_payable` distinctness from `tax_authority_payable`:** WHT settles on different cadence (per-payroll vs monthly VAT), requires per-employee tracking (for PAYE certificates), and produces separate statutory return distinct from VAT return. Keeping these as separate Obligation kinds keeps reporting + reconciliation clean. |

---

## 3. Precondition — `tenant_tax_profile` (N7)

Before any tax computation, the engine must know the tenant's tax profile. The profile is a tenant-property record set via pack-driven defaults + tenant election (per CTR-043 / CTR-029 expansion):

```yaml
# Pack content + tenant-property event
tenant_tax_profile:
  vat_registered:                   bool             # default false for SME below threshold
  vat_registration_number:          string?          # required if vat_registered: true
  has_formal_employees:             bool             # drives PAYE applicability
  wht_obligated:                    bool             # non-VAT tenants may still have WHT on supplier payments
  tax_exemptions:                   [enum...]        # ngo | agriculture | export_zone | sez | etc.
  informal_sector:                  bool             # legitimate cash-basis; minimal tax obligations
  tax_authority_ref:                string           # TRA | KRA | URA — per primary jurisdiction
  vat_threshold_status:             enum             # below_threshold | approaching | registered
  primary_jurisdiction:             string           # ISO country code
  secondary_jurisdictions:          [string]         # v2 framework readiness
```

### Engine Behaviour Driven by Profile

| Profile field | Engine effect |
|---------------|---------------|
| `vat_registered: false` | Checkout never computes output VAT; receipts show "Bei TZS X (VAT exempt — tenant not registered)"; Procurement never claims input VAT |
| `has_formal_employees: false` | HR PAYE computation skipped; no PAYE Obligations created |
| `wht_obligated: true` (even if not VAT-registered) | Cash + Procurement compute WHT on supplier payments per pack rules |
| `informal_sector: true` | Tax-period close not required; pack provides "informal-sector annual summary" Document only |
| `tax_exemptions: [ngo]` | Specific income categories exempt per pack rule (Checkout's `tax_treatment_ref` lookup resolves to `exempt`) |
| `vat_threshold_status: approaching` | tax-advisor pre-warns at pack-defined turnover thresholds (e.g., 80% of registration threshold) |

### Conservative Default for New Tenant

```yaml
tenant_tax_profile:                              # defaults at tenant registration
  vat_registered:                   false
  has_formal_employees:             false
  wht_obligated:                    false
  informal_sector:                  true
  tax_authority_ref:                <pack.primary_jurisdiction.tax_authority>
  vat_threshold_status:             below_threshold
  primary_jurisdiction:             <pack default>
```

A tenant onboards conservatively — minimal tax engagement until they explicitly upgrade (via tenant-property event when threshold crossed or registration completed). This matches the reality of ~80% of BOS target tenants who are informal-sector SMEs below VAT registration threshold.

### Why N7 Matters

Without `tenant_tax_profile`, the engine would assume full tax compliance for every tenant — generating Obligations and Statement Documents that don't apply to informal-sector operations. The profile is the gate: tax mechanism activates per tenant's actual regulatory status, not blanket assumptions.

---

## 4. `tax_treatment_ref` Resolution Chain

Per TX2: `tax_treatment_ref` is the input mechanism. Each source engine sets it at the originating event; Accounting resolves at journal time; Reporting reads at Statement time.

### 4.1 — Source Engines Set `tax_treatment_ref`

#### Checkout (CN-5-009 K3)

At bill construction (Term 6 vertical → Checkout), per saleable line:

```yaml
line.tax_treatment_ref = pack.tax.lookup(
  item_ref:           <item being sold>
  jurisdiction_ref:   tenant.primary_jurisdiction         # v1 single-jurisdiction
  sale_context:       {customer_type, channel, time}
  tenant_tax_profile:                                      # gates entire resolution
)
```

If `tenant_tax_profile.vat_registered: false` → resolution returns `vat_not_applicable` reference; Checkout K3 computes zero VAT; receipt notes "VAT exempt".

#### Procurement (CN-5-004)

At invoice recording, per line:

```yaml
line.tax_treatment_ref = pack.tax.lookup(
  item_ref:           <item being purchased>
  jurisdiction_ref:   supplier.jurisdiction
  purchase_context:   {supplier_vat_status, import_status}
  tenant_tax_profile:
)
```

Resolves to `input_vat_recoverable` | `input_vat_non_recoverable` | `reverse_charge_applicable` | `vat_not_applicable` per pack rule.

#### HR (CN-5-005)

At payroll computation, per deduction:

```yaml
deduction.tax_treatment_ref = pack.tax.lookup(
  employee_ref:       <employee>
  jurisdiction_ref:   employee.tax_jurisdiction
  comp_kind:          salary | bonus | commission | overtime | termination_payout
  statutory_rules:    pack.hr_statutory_deductions
  tenant_tax_profile:
)
```

If `tenant_tax_profile.has_formal_employees: false` → resolution returns `not_applicable`; no PAYE Obligation created.

#### Promotion (CN-5-007 PR6)

At cost-share + voucher events:

```yaml
discount.tax_treatment_ref = pack.tax.lookup(
  promotion_kind:     campaign_funded | line_discount | bundle | voucher | loyalty
  jurisdiction_ref:   tenant.primary_jurisdiction
  tenant_tax_profile:
)
```

Resolution determines whether discount reduces taxable base (most jurisdictions) or is recorded as separate revenue adjustment (some).

#### Cash (CN-5-002) — N6 Precision

Cash sets `tax_treatment_ref` **ONLY when the payment is the originating event for tax recognition** — specifically WHT-at-payment (where pack rule recognises WHT at settlement, not at invoice recording):

```yaml
# Only for WHT-at-payment patterns
payment.tax_treatment_ref = pack.tax.lookup(
  payment_kind:       supplier_payment | salary_payment | other
  jurisdiction_ref:   tenant.primary_jurisdiction
  supplier_or_employee_ref:  <party>
  tenant_tax_profile:
)
```

For all other payments (paying down AP, settling existing Obligations), Cash **inherits** the `tax_treatment_ref` from the upstream Obligation — never sets new one. This avoids tax-treatment drift between recognition (Procurement / Accounting) and settlement (Cash).

### 4.2 — Accounting Resolves at Journal Time

```yaml
# CN-5-001 auto-journal flow (per #1 / #6 / #10 etc.)
For each incoming event with tax_treatment_ref:
  resolution = pack.tax.resolve(tax_treatment_ref, pack_version_at_event_timestamp):
    account_refs:
      output_vat_account:        <pack-defined per jurisdiction>
      input_vat_account:         <pack-defined>
      wht_account:               <pack-defined per WHT type>
      expense_account:           <pack-defined> (for non-recoverable)
    recovery_rule:              recoverable | non_recoverable | pass_through | exempt
    rate:                        <decimal per pack version at event timestamp>
    obligation_kind:            tax_authority_payable | tax_authority_receivable | wht_payable | (none if exempt)

Accounting emits accounting.journal.posted.v1 with tax-related line entries +
creates Obligation primitive (per TX12) if obligation_kind specified.
```

### 4.3 — Reporting Reads at Statement Time

```yaml
# CN-5-006 tax return Statement generation
At tax-period close OR on-demand:
  VAT return:
    output_VAT_total       = sum(events where tax_treatment_ref resolves to output_VAT in tax_period)
    input_VAT_recoverable  = sum(events where resolution is input_vat_recoverable in tax_period)
    net_position           = output_VAT_total − input_VAT_recoverable
    payable_or_credit      = (positive: payable to authority) | (negative: refund/credit)

  PAYE Summary:
    per_employee:          {gross_paid, paye_withheld, pension_withheld, nhif_withheld}
    total_remittance:      sum(paye_withheld events in tax_period)

  WHT-on-Services Summary:
    per_supplier:          {gross_paid, wht_withheld}
    total_remittance:      sum(wht events in tax_period)

  Multi-jurisdiction breakdowns (v2 framework readiness):
    per_jurisdiction:      {output_VAT, input_VAT, net, returns_required}
```

---

## 5. Output VAT — Sales Tax Collection

### Source

`checkout.settled.v1` per CN-5-009 K3 (tax at settlement, pack-frozen via `pack_version_ref` at event). Each line carries `tax_treatment_ref`; Checkout computes tax amount at settlement.

### Journal

CN-5-001 auto-journal #1 (revenue + tax + COGS) on `checkout.settled.v1`:

```
Dr Cash / AR                <grand_total>
Dr COGS                     <cogs_from_lines>
Cr Revenue                  <pre_tax_total>
Cr Tax Payable (Output VAT) <tax_total>           # creates tax_authority_payable_output_vat
Cr Inventory                <cogs_from_lines>
```

### Obligation

`tax_authority_payable_output_vat` Obligation accumulates per tax period. UI-08 bounds enforced; outstanding grows with each sale; reduces at tax-period close + settlement.

### Settlement

Tenant settles via `cash.payment.record.request` against the Obligation:

```yaml
cash.payment.record.request:
  obligation_ref:        <output_vat Obligation in current tax period>
  amount:                <decimal>
  till_id:               <funding till>
  payee_party_ref:       <tax authority — per tenant_tax_profile.tax_authority_ref>
```

Settles per CN-5-002 AP flow. Obligation reduces; UI-08 holds.

### Special Case: Below Threshold

`tenant_tax_profile.vat_registered: false` → Checkout never creates output VAT Obligation. Receipt simply notes "no VAT applicable; tenant below registration threshold."

---

## 6. Input VAT — Purchase Tax Recovery

### Source

`procurement.invoice.recorded.v1` per CN-5-004 / CN-5-001 #6. Each invoice line carries `tax_treatment_ref` set by Procurement based on item + supplier + jurisdiction.

### Pack-Driven Recoverability (Per Q4)

Pack rules determine per item category (some items non-recoverable per jurisdiction):

```yaml
# Pack content (CTR-043 expansion)
tax_treatment.input_vat:
  recoverable_categories:        [raw_material, inventory_for_resale, capital_equipment, ...]
  non_recoverable_categories:    [entertainment, fuel_for_personal_vehicle, gifts_above_threshold, ...]
  per_jurisdiction_overrides:    {TZ: {...}, KE: {...}}
```

### Journal — Recoverable Path

```
Dr Inventory / Expense         <amount_excl_tax>
Dr Input VAT Receivable        <vat_amount>          # creates tax_authority_receivable_input_vat
Cr Accounts Payable            <amount_incl_tax>
```

### Journal — Non-Recoverable Path

```
Dr Inventory / Expense         <amount_incl_tax>     # full amount expensed; no separate VAT recovery
Cr Accounts Payable            <amount_incl_tax>
```

### Obligation

Recoverable: `tax_authority_receivable_input_vat` Obligation accumulates. Special bound semantics (TX12 footnote): outstanding can grow (input VAT accruing); reduces at tax-period close (offset against output VAT) or via refund (`cash.collection.received.v1`).

Non-recoverable: no tax Obligation created; cost stays in operating expense.

### Special Case: Below Threshold

`tenant_tax_profile.vat_registered: false` → Procurement never claims input VAT regardless of supplier VAT status. Tax amount on supplier invoice is recorded as part of expense (full amount).

---

## 7. Withholding Tax — PAYE + WHT on Services

### PAYE (Pay-As-You-Earn)

Computed at payroll per CN-5-005 §9 statutory deductions:

```yaml
# CN-5-005 payroll computation
For each employee:
  paye_amount = pack.tax_treatment.paye_compute(
    gross_pay,
    employee.tax_jurisdiction,
    pack.hr_statutory_deductions.paye.bands
  )
```

Per N4 of CN-5-005, deduction events emit at `hr.payroll.paid.v1` time:

```yaml
hr.payroll.deduction.applied.v1:
  employee_ref, obligation_ref (paye_obligation), amount: paye_amount, period
```

### PAYE Obligation

`wht_payable_paye` Obligation accumulates per pack-defined remittance period (usually monthly). Per TX12 footnote: tracked per-employee (for PAYE certificate) + aggregate (for remittance to authority).

### WHT on Services

Pack rules determine per supplier/service whether WHT applies:

```yaml
# Pack content (CTR-043 expansion)
tax_treatment.wht_services:
  applicable_services:           [professional_services, consultancy, rent, ...]
  per_jurisdiction_rates:        {TZ: 5%, KE: 5%, UG: 6%, ...}
  exemption_thresholds:          {TZ: TZS 50000_per_invoice, ...}
```

At Procurement invoice recording (if WHT-at-invoice pack pattern) OR Cash payment (if WHT-at-payment pack pattern), engine computes WHT amount + creates `wht_payable_services` Obligation.

### Settlement

PAYE + WHT-on-services Obligations settle via `cash.payment.record.request` to tax authority per pack remittance cycle (often monthly for PAYE; quarterly or monthly for WHT-services).

### Special Case

`tenant_tax_profile.has_formal_employees: false` → no PAYE.
`tenant_tax_profile.wht_obligated: false` → no WHT-on-services (some informal sector tenants exempt).

---

## 8. Reverse-Charge VAT (Q6 / Imports + B2B Cross-Border)

### When Applicable

Pack rule per jurisdiction (default per OECD model):

```yaml
# Pack content (CTR-043 expansion)
tax_treatment.reverse_charge:
  applicable_when:
    - importation_of_services_from_non_resident
    - importation_of_goods_above_threshold (some jurisdictions)
    - B2B_intra_community_supply (EU pattern; not common in EA but documented)
  applicability_conditions:
    - tenant_tax_profile.vat_registered: true        # only VAT-registered tenants reverse-charge
    - supplier.is_non_resident: true                 # or per pack
```

### Mechanism

At `procurement.invoice.recorded.v1` for a reverse-charge invoice:

Accounting auto-journal emits **simultaneous** output VAT + input VAT:

```
Dr Inventory / Expense         <invoice_amount>
Dr Input VAT Receivable        <reverse_charge_vat>     # creates tax_authority_receivable
Cr Accounts Payable            <invoice_amount>         # NB: invoice amount excludes VAT
Cr Output VAT Payable          <reverse_charge_vat>     # creates tax_authority_payable
```

### Net Effect

If fully recoverable: net effect on tax obligations = zero (output and input cancel at next VAT return). Cash flow not affected by VAT line.

If partially recoverable (per pack rule): net positive payable; tenant remits the unrecovered portion.

### Special Case

`tenant_tax_profile.vat_registered: false` → reverse-charge does NOT apply. Tenant cannot claim input VAT; supplier's "no VAT" treatment stands.

---

## 9. Tax-Period Close Choreography (Per TX9 — Six Phases Parallel to CN-5-104)

Tax periods are SEPARATE from financial periods (Q7 ruling). Pack declares both:

```yaml
# Pack content
period_calendar:                  # financial close per CN-5-104
  frequency:                      monthly
  fiscal_year_start:              {month: 1, day: 1}

tax_calendar:                     # tax-period close per this doc
  vat_period_frequency:           monthly                  # VAT return monthly
  vat_return_due:                 "20th of following month"
  paye_period_frequency:          monthly                  # monthly PAYE remittance
  paye_remittance_due:            "9th of following month"
  annual_paye_certificate_due:    "Jan 31"                 # annual cert
  wht_services_period_frequency:  monthly OR quarterly
  excise_period_frequency:        per_excise_kind
```

### Six-Phase Pattern

```
═════════════════════════════════════════════════════════════════════════════
PHASE 1 — TAX-PERIOD READINESS (lighter than financial close; 3 signals)
═════════════════════════════════════════════════════════════════════════════

Engines self-validate tax-relevant operations for the tax period:

  Cash:        cash.tax_period.ready.v1
                Criteria: WHT remittances scheduled in period recorded or
                deferred; tax authority Obligations stable
  Procurement: procurement.tax_period.ready.v1
                Criteria: input VAT events have tax_treatment_ref resolved;
                reverse-charge events recorded
  HR:          hr.tax_period.ready.v1
                Criteria: PAYE computations for period emitted; WHT-on-services
                computations emitted

Inventory + Checkout + Promotion + Reporting do NOT signal — events flow
through Accounting independently.

═════════════════════════════════════════════════════════════════════════════
PHASE 2 — TAX CLOSE REQUEST
═════════════════════════════════════════════════════════════════════════════

Authorised human (tax officer / accountant per pack approval threshold) submits:
  accounting.tax_period.close.request {tax_period_ref, jurisdiction_ref}

Bus validates principal + required signals received per pack.tax_calendar
configuration.

IF missing signals → reject with engine-specific reason
IF all received → emit:
  accounting.tax_period.close.initiated.v1
    payload:
      tax_period_ref
      jurisdiction_ref
      initiated_at, initiated_by
      signals_received_refs

Tax-period freeze-analog activates: bus rejects new commands attempting to
emit tax-relevant events with tax_period_ref ∈ initiated period
(pre-emptive UI-11).

═════════════════════════════════════════════════════════════════════════════
PHASE 3 — TAX COMPUTATION GATE (Q11 — parallel UI-07 pattern)
═════════════════════════════════════════════════════════════════════════════

Accounting computes net tax position for period:

  output_vat_total       = sum(output_VAT events in tax_period)
  input_vat_recoverable  = sum(input_VAT recoverable events in tax_period)
  net_vat_position       = output_vat_total − input_vat_recoverable

  paye_total             = sum(paye events in tax_period)
  wht_services_total     = sum(wht_services events in tax_period)
  excise_total           = sum(excise events in tax_period)

Verification gate (pack-defined epsilon — Q11):
  - Output + Input VAT events balance to recorded journal totals (UI-07 analog)
  - Obligation outstanding amounts match accumulated event totals (UI-08)
  - No required filings missing data per pack tax_return_completeness rules

IF balanced → proceed Phase 4
IF gate fails:
  Bus emits:    accounting.tax_period.close.rejected.v1
    payload:
      tax_period_ref, jurisdiction_ref
      reason:                    tax_computation_unbalanced | missing_filings_data
      detail:                    {specifics}
      pack_version_ref
  Freeze-analog LIFTS.
  Accountant investigates via Reporting + tax-advisor; corrects via
  forward-period events (PC8 / UI-11 — corrections post forward in current
  open tax_period); re-submits close request.

═════════════════════════════════════════════════════════════════════════════
PHASE 4 — TAX CLOSE EMISSION + UI-11 ACTIVATION
═════════════════════════════════════════════════════════════════════════════

Bus atomically emits (CN-4-004 §2):

  accounting.tax_period.closed.v1
    payload:
      tax_period_ref, jurisdiction_ref
      closed_at, closed_by
      summary:
        output_vat_total
        input_vat_recoverable
        net_vat_position           (positive = payable; negative = credit/refund)
        paye_total
        wht_services_total
        excise_totals              (per excise kind)
      signals_received_refs
      pack_version_ref

UI-11 bus policy in-effect: any subsequent command attempting to emit a
tax-relevant event with tax_period_ref ∈ closed_period for the
jurisdiction is rejected per UI-11 — corrections must post forward via
posting_tax_period_ref + references_closed_period (per Q13).

═════════════════════════════════════════════════════════════════════════════
PHASE 5 — TAX RETURN STATEMENT ISSUANCE (per CN-5-006 N3 + PC6)
═════════════════════════════════════════════════════════════════════════════

Reporting subscribes accounting.tax_period.closed.v1 and issues formal
Statement Documents via CN-4-012:

  reporting.statement.issued.v1   × N      (one per tax return type)
    VAT Return Statement:
      template_ref:     <pack-defined per jurisdiction>
      content:          {output, input, net, applicable_offset_or_payable}
      hash, number, template_version_ref, pack_version_ref
    PAYE Remittance Statement:
      content:          {employees, totals, due_date}
    Annual PAYE Certificate (year-end tax period):
      content:          {employees, annual_totals, comparatives}
    WHT-on-Services Return (if applicable):
      content:          {suppliers, totals}
    Per-jurisdiction additional statutory bundles (multi-jurisdiction v2)

Each Document: hash-frozen, numbered, template-versioned, pack-frozen.

═════════════════════════════════════════════════════════════════════════════
PHASE 6 — SETTLEMENT TO AUTHORITY (separate cash flow)
═════════════════════════════════════════════════════════════════════════════

Settlement is the TENANT'S authorised action — BOS provides the Statement
artefacts; tenant's accountant downloads + files; tenant authorises Cash
payment to authority:

  tenant submits (or Cash auto-prepares draft):
    cash.payment.record.request:
      obligation_ref:        <tax_authority_payable Obligation>
      amount:                <decimal>
      payee_party_ref:       <tax authority>

Per CN-5-002 AR/AP flow; Obligation reduces; UI-08 holds.

If net VAT position is CREDIT (input > output) — Q12:
  Per pack rule, either:
    (a) Offset against next period's output VAT (carry-forward)
    (b) Refund request (tenant submits claim to authority offline;
        cash.collection.received.v1 records refund when received)
```

### Key Differences from Financial Period Close (CN-5-104)

| Aspect | Financial Close (CN-5-104) | Tax Close (CN-5-105) |
|--------|----------------------------|------------------------|
| Frequency | Pack-driven (monthly default) | Per tax kind per pack tax_calendar |
| Signals | 4 core (Cash, Inventory, Procurement, HR) | 3 (Cash, Procurement, HR) — tax-specific |
| Gate | UI-07 trial balance | Tax computation balance + UI-08 obligation match |
| Output | P&L, BS, CFS Documents | Tax Return Documents |
| Settlement | None (close is final) | Cash settlement to authority closes cycle |
| Inviolability | UI-05 | UI-11 |
| Scope | tenant | tenant + jurisdiction |

---

## 10. Tax Return Statement Documents (Per CN-5-006 N3 + CTR-029 Expansion)

Pack declares per-jurisdiction Statement templates for each tax return type:

```yaml
# Pack content (CTR-029 + CTR-043 expansion)
tax_return_statements:
  TZ:
    vat_return:
      template_ref:           tz-vat-return-template-v3
      issuance_frequency:     monthly
      due_day_of_month:       20
      number_format:          "TZ-VAT-{TENANT-PREFIX}-{YYYY}-{MM}"
    paye_remittance:
      template_ref:           tz-paye-remittance-template-v2
      issuance_frequency:     monthly
      due_day_of_month:       9
    paye_annual_certificate:
      template_ref:           tz-paye-annual-cert-template-v2
      issuance_frequency:     annual
      due_date:               "Jan 31"
    wht_services_return:
      template_ref:           tz-wht-services-template-v1
      issuance_frequency:     monthly
  KE:                                                 # multi-jurisdiction v2 expansion
    vat_return: {...}
    paye_remittance: {...}
```

Each Statement is a Document per CN-4-012 (terminal fold, hash-frozen, numbered). Issued at Phase 5 of tax-period close.

### Re-issuance Path

Same as CN-5-006 §5 — if a Statement is later found erroneous, new linked Document with `corrects_document_ref` references original. Original Document hash-frozen forever. Closed tax period does NOT reopen.

---

## 11. Multi-Jurisdiction Handling (TX10 — Framework Readiness)

### v1 Single-Jurisdiction

For v1, every tenant operates in their `primary_jurisdiction`. All transactions implicitly tagged. Pack resolution always uses primary.

### Framework Hooks (Documented for v2)

```yaml
# Event payload extension (v2 makes mandatory)
event.payload:
  jurisdiction_ref:     <ISO country code; v1 implicit primary>

# Pack content
pack.jurisdictions:
  primary:              "TZ"
  secondary:            ["KE", "UG"]                  # v2 — tenant operates cross-border

# Tax-period close v2 — per-jurisdiction close decisions
accounting.tax_period.closed.v1:
  jurisdiction_ref:     <which jurisdiction's tax period closed>
  # Multi-jurisdiction tenant has separate tax_period.closed per jurisdiction
```

### Why Defer (Per Q8)

Cross-border tenants are RARE in BOS target audience (SME informal-sector primary). Implementing full multi-jurisdiction in v1 adds complexity that 99% of tenants don't need. Framework hooks ensure v2 doesn't require Foundation amendments — engines emit `jurisdiction_ref` implicit; v2 makes it explicit + adds per-jurisdiction Obligation tracking.

---

## 12. Mid-Period Rate Change (Q2 — Pack Version-Based Deterministic)

When tax authority changes a rate (e.g., VAT goes 18% → 19% effective May 1):

### Mechanism

Pack rotation event records new pack version + effective date:

```yaml
# Pack rotation event
pack.version.activated.v1:
  pack_id:              tz-compliance
  old_version:          tz-compliance-2026.07
  new_version:          tz-compliance-2027.05
  effective_business_date: 2027-05-01
```

### Engine Behaviour

Each event's tax computation uses the pack version active at the event's `business_date`:

- Event with `business_date: 2027-04-30` → resolves under `tz-compliance-2026.07` (old rate 18%)
- Event with `business_date: 2027-05-01` → resolves under `tz-compliance-2027.05` (new rate 19%)

Each event's `pack_version_ref` is recorded permanently (D-009 freeze). Replay deterministic.

### No Compensation for In-Flight Events

Per Q2 ruling: events emitted before the cutover stand under the old pack version; no retroactive recomputation. CN-4-001 immutability + D-009 freeze guarantee deterministic replay.

---

## 13. Tax-Period Failure Modes (Q11 / Q13)

### Q11 — Tax-Period Close Failure (Computation Gate)

If Phase 3 computation gate fails (output/input event sums don't reconcile, Obligation amounts mismatch, required filings data missing):

```
accounting.tax_period.close.rejected.v1 emitted
Freeze-analog lifts
Accountant investigates:
  - Use tax-advisor (post-rejection hook) to identify discrepancy
  - Investigate via Reporting tax-return preview
  - Post corrections via standard accounting.adjustment.post.request in CURRENT
    open period (not the closing period — already in freeze)
  - Resubmit accounting.tax_period.close.request once corrections settled
```

NEVER force-close (parallel CN-5-104 Q10 — financial close).

### Q13 — Backdated Tax Assessment (Audit Finding)

When tax authority audits and assesses additional tax for a prior closed tax period (rare; usually months after filing):

```yaml
accounting.tax.assessed.v1:
  payload:
    tax_period_ref:               <prior closed tax period>
    jurisdiction_ref:             <jurisdiction>
    assessment_amount:            <additional tax owed>
    assessment_kind:              vat | paye | wht | excise
    reason:                       <audit finding text>
    business_date:                <current open period>
    posting_tax_period_ref:       <current open tax_period>     # forward-correction anchor
    references_closed_period:     <closed tax_period_ref>        # backward-reference link
    causation_id:                 <command that recorded the assessment>
```

Creates new Obligation: `kind: tax_assessment_payable`. Settled via `cash.payment.disbursed.v1` to authority.

**Closed tax period truth UNCHANGED.** The original Statement Document (filed return) remains hash-frozen forever. Assessment is a current-period adjustment with backward reference for audit trail.

### Engine Down / Adapter Failure

If Cash banking adapter (CTR-031) fails during tax remittance, the Obligation persists; retry follows standard CN-5-002 §11 deposit retry path.

### Pack Rotation Mid-Tax-Period

Same as financial close — pack rotation mid-tax-period requires re-evaluation. If close.request was issued under old pack version but pack rotated before close.closed.v1 emitted, close re-evaluation uses new pack version (events themselves under their original pack_version_ref per D-009).

---

## 14. Tax Exemptions + Special Schemes (Q10 — Pack-Only)

Per Q10: engine handles exemptions purely through `tax_treatment_ref` resolution; no engine code special-cases.

### Common Schemes (Pack-Configurable)

```yaml
# Pack content (CTR-043 expansion)
tax_exemptions:
  ngo_exemption:
    applicability_when:           tenant_tax_profile.tax_exemptions includes "ngo"
    exempt_income_categories:     [donations, grants]
    output_vat_rate:              0
  agriculture_zero_rate:
    applicable_items:             [unprocessed_produce]
    rate:                         0
  export_zero_rate:
    applicable_sales:             [export_invoices]
    rate:                         0
  sez_special_scheme:
    applicability_when:           tenant_tax_profile.tax_exemptions includes "sez"
    special_treatment:            <pack-defined>
  tourist_vat_refund:
    applicability_when:           customer.tourist_visa_holder: true
    refund_mechanism:             <pack-defined>
```

### Engine Implementation

Engines simply read `tax_treatment_ref` and the resolved category. If resolution returns `exempt`, no tax computation; no Obligation created. Receipts may note exemption per pack template (e.g., "Zero-rated — agricultural produce").

No engine code knows about specific exemption types. New exemption = pack content addition only.

---

## 15. Tax-Advisor Integration (TX11 — Per CN-5-010 Amendment §4.9)

The CN-5-010 amendment (this commit) adds `tax-advisor` as the 9th advisor — Phase 1, with activation thresholds (≥2 closed tax periods with activity + ≥1 prior return filed) ensuring sufficient data for analysis.

### Pre-Tax-Close Hook

```
Schedule:        pack.advisor.tax.schedule_default (default: pre_tax_period_close)
Trigger:         scheduled N days before tax_calendar return_due_date

Output:          - Pre-filing review: completeness check (all expected events recorded)
                 - Anomaly flags: input VAT spike vs trend; missing PAYE for active employees
                 - Compliance reminders: return due date + pack rules

Audience:        accountant, tax_officer, owner
Delivery:        UI feed + optional email (CTR-040 with consent purpose tax_filing_reminders)
```

### Post-Close Hook

```
Trigger:         event_subscription on accounting.tax_period.closed.v1

Output:          - Filing reminders with downloadable Statement Document refs
                 - Net position summary (payable or refund)
                 - Anomalies in closed period for review next period

Wording (per N8 boundary discipline):
  ✅ "Suggested: VAT return for October due Nov 20. Click to download
     VAT Return Statement Document for filing."
  ❌ "Filing now... done."
  ✅ "Net credit position TZS 6,300 from input > output. Pack rule TZ allows
     offset against November OR refund claim. Suggest offset (no claim paperwork)."
  ❌ "I have submitted refund claim on your behalf."
```

### Post-Assessment Hook (Q13)

```
Trigger:         event_subscription on accounting.tax.assessed.v1

Output:          - "Tax authority assessment received for prior period October 2025.
                   Amount TZS 145k. Suggest review of original return + assessment basis."
                 - Evidence drill-down to closed period's Statement + current assessment

Audience:        accountant, owner
```

### Advisor Wording Discipline (N8 — Critical)

Per CN-5-105 §1 boundary table: advisor suggestions describe what the user/accountant should DO with BOS-provided artefacts. Never:
- Suggest BOS filing on tenant's behalf
- Suggest BOS acting as tax agent
- Make statements implying BOS has authority to interact with tax authority directly

All actions are TENANT's — the advisor provides decision support; the tenant (via their accountant) takes action through normal command bus paths.

---

## 16. UI Invariants Touched

| Invariant | Application |
|-----------|-------------|
| **UI-01** (causation chain) | Tax events chain back to source economic events (sale, purchase, payroll); forward to settlement. Full audit walk: settlement → Obligation → tax journal → source event. |
| **UI-05** (closed financial-period inviolability) | Financial close UI-05 honoured for tax-affecting events with effective_date in closed financial period; corrections post forward per PC8 (CN-5-104). |
| **UI-06** (functional currency) | Tax obligations in tenant functional currency; multi-currency tax bridged via FX events. |
| **UI-07** (trial balance) | Tax-related journals included in trial balance; tax close gate per CN-5-105 §9 Phase 3 is parallel UI-07 pattern. |
| **UI-08** (Obligation bounds) | Four tax Obligation kinds (per TX12) follow UI-08 bounds; `tax_authority_receivable_input_vat` has special bound semantics (can grow with accruals). |
| **UI-11** (closed tax-period inviolability — NEW per CN-5-102 amendment) | Closed tax period inviolable per jurisdiction; corrections post forward via posting_tax_period_ref + references_closed_period. |

---

## 17. Worked Example — Dual Scenario (Per N7)

Two examples demonstrating tenant heterogeneity:

### 17.1 Mama Amina — Informal Sector, No VAT

**Setting:** Mama Amina ya duka la mboga. Annual turnover TZS 35M (below TZS 100M VAT registration threshold per TZ pack). Two-person operation (Mama Amina + apprentice paid weekly cash; no formal employment contract). Pack `tz-compliance-2026.07`. `tenant_tax_profile`:

```yaml
vat_registered:                   false
has_formal_employees:             false
wht_obligated:                    false
informal_sector:                  true
tax_authority_ref:                TRA
vat_threshold_status:             below_threshold
primary_jurisdiction:             TZ
tax_exemptions:                   []
```

#### October Operations

**Sales (Output VAT):**
- 10 × Coca-Cola sales × TZS 1,500 = TZS 15,000 total
- Per K3 of CN-5-009 + Mama Amina's `tenant_tax_profile.vat_registered: false`:
  - `pack.tax.lookup()` returns `vat_not_applicable` reference
  - Checkout K3 computes zero VAT
  - Receipt: "Bei TZS 1,500 (VAT exempt — tenant not registered)"
  - NO `tax_authority_payable_output_vat` Obligation created

**Purchases (Input VAT):**
- 1 supplier invoice TZS 50,000 + TZS 9,000 VAT charged (supplier IS VAT-registered)
- Per CN-5-004: invoice line `tax_treatment_ref` resolved per Mama Amina's profile
  - `pack.tax.lookup()` returns `input_vat_non_recoverable` (tenant cannot claim because not VAT-registered)
  - Accounting #6 journals full amount to expense:
    ```
    Dr Inventory                 TZS 59,000        (full amount incl. VAT)
    Cr Accounts Payable          TZS 59,000
    ```
  - NO `tax_authority_receivable_input_vat` Obligation created

**Apprentice Pay:**
- Weekly cash to apprentice TZS 30,000 (informal)
- Per `tenant_tax_profile.has_formal_employees: false`: HR engine does NOT compute PAYE
- Mama Amina records via `cash.expense.record.request` (category: staff_welfare per CN-5-002 §7)
- NO `wht_payable_paye` Obligation

#### Tax-Period Close

- Per `tenant_tax_profile.informal_sector: true`: tax-period close is **skipped**
- No `accounting.tax_period.close.request` submitted
- Pack provides "informal-sector annual summary" Document only — issued at financial year-end via CN-5-006 (different from formal VAT return)

#### Tax-Advisor (If Activated)

Mama Amina activates tax-advisor (rare — most informal-sector tenants don't, but possible):

- Pre-close suggestion (Nov 1): "Turnover at 35% of VAT threshold (TZS 35M of TZS 100M). No action needed; will pre-warn at 80%."
- No filing reminders (no returns due)
- Year-end summary: "Total revenue TZS X; total expenses TZS Y; net profit TZS Z. Suggested: download informal-sector annual summary for personal income tax filing (per pack TZ informal-sector rule)."

#### Charter §1.3 Boundary

Even for informal sector, the boundary holds: BOS provides annual summary; Mama Amina takes it to her bookkeeper / files personal tax return at TRA herself. **BOS did not file.**

### 17.2 Mzee Hassan — VAT-Registered, Has Employees

**Setting:** Mzee Hassan Karakana (Workshop). Annual turnover TZS 145M (above VAT threshold). 4 formal employees (PAYE applies). Pack `tz-compliance-2026.07`. `tenant_tax_profile`:

```yaml
vat_registered:                   true
vat_registration_number:          "TZ-VAT-99887766"
has_formal_employees:             true
wht_obligated:                    true                      # WHT on rent + services
informal_sector:                  false
tax_authority_ref:                TRA
vat_threshold_status:             registered
primary_jurisdiction:             TZ
tax_exemptions:                   []
```

#### October Operations

**Sales (Output VAT):**
- Various sales totalling TZS 12,000,000 base + TZS 2,160,000 output VAT (18% rate)
- Each `checkout.settled.v1` → Accounting journals output VAT line
- `tax_authority_payable_output_vat` Obligation accumulates: TZS 2,160,000 outstanding

**Purchases (Input VAT):**
- Supplier invoices totalling TZS 7,500,000 base + TZS 1,350,000 input VAT
- All items recoverable per pack (no `non_recoverable_categories` matched)
- `tax_authority_receivable_input_vat` Obligation accumulates: TZS 1,350,000 outstanding

**Payroll (PAYE):**
- October payroll for 4 employees; gross TZS 2,400,000; PAYE withheld TZS 230,000
- `hr.payroll.computed.v1` → `hr.payroll.paid.v1` + `hr.payroll.deduction.applied.v1`
- `wht_payable_paye` Obligation: TZS 230,000 outstanding

**WHT on Rent (Q6-Style Service):**
- Mzee Hassan rents workshop for TZS 500,000/month; WHT-on-services rate 10% per TZ pack
- Cash payment for rent: `cash.payment.record.request` with `tax_treatment_ref` set per N6 (Cash sets because WHT-at-payment pack pattern)
- TZS 50,000 WHT withheld; tenant remits to TRA
- `wht_payable_services` Obligation: TZS 50,000

#### Tax-Period Close (Nov 1-3)

**Phase 1 — Readiness Signals:**
```
cash.tax_period.ready.v1         (all WHT remittances recorded or deferred)
procurement.tax_period.ready.v1  (all input VAT events resolved)
hr.tax_period.ready.v1           (PAYE computations for Oct paid)
```

**Phase 2 — Close Request (Nov 4):**
```
Mzee Hassan's accountant (Mzee Tabu, chartered) submits:
  accounting.tax_period.close.request {tax_period_ref: 2026-10, jurisdiction_ref: TZ}

Bus validates principal + all 3 signals received ✓
Emits: accounting.tax_period.close.initiated.v1
Tax-period freeze-analog ACTIVE for 2026-10.
```

**Phase 3 — Computation Gate:**
```
Output VAT total:       TZS 2,160,000
Input VAT recoverable:  TZS 1,350,000
Net VAT position:       TZS  810,000  (payable)
PAYE total:             TZS  230,000  (payable)
WHT-on-services:        TZS   50,000  (payable)

All events reconcile to journal totals ✓
All Obligations bounds hold ✓
Required data complete ✓
```

**Phase 4 — Close Emission:**
```
Bus emits: accounting.tax_period.closed.v1
  payload:
    tax_period_ref:           2026-10
    jurisdiction_ref:         TZ
    closed_at:                2026-11-04T09:00:00 EAT
    closed_by:                human:mzee-tabu-accountant
    summary:
      output_vat_total:       TZS 2,160,000
      input_vat_recoverable:  TZS 1,350,000
      net_vat_position:       TZS   810,000
      paye_total:             TZS   230,000
      wht_services_total:     TZS    50,000

UI-11 PERMANENT for (2026-10, TZ): any future tax-relevant event with
tax_period_ref = 2026-10 + jurisdiction TZ is rejected.
```

**Phase 5 — Statement Documents Issued (per CN-5-006 N3):**
```
reporting.statement.issued.v1 × 3:

  STMT-VAT-MZH-2026-10
    template_version:    tz-vat-return-template-v3
    pack_version_ref:    tz-compliance-2026.07
    content:             {output: 2.16M, input: 1.35M, net_payable: 0.81M}
    hash:                <CN-4-012 hash>

  STMT-PAYE-MZH-2026-10
    template_version:    tz-paye-remittance-template-v2
    content:             {employees: [...], total_remittance: 0.23M}

  STMT-WHT-SVC-MZH-2026-10
    template_version:    tz-wht-services-template-v1
    content:             {suppliers: [rent_landlord], total: 0.05M}
```

**Phase 6 — Settlement to Authority (Tenant's Action):**

Mzee Tabu downloads the Statement Documents. He files at TRA online portal (outside BOS). Mzee Hassan authorises Cash payment to TRA:

```
cash.payment.record.request:
  obligation_ref:        <output_vat Obligation>
  amount:                TZS 810,000
  payee_party_ref:       TRA

cash.payment.record.request:
  obligation_ref:        <paye Obligation>
  amount:                TZS 230,000
  payee_party_ref:       TRA

cash.payment.record.request:
  obligation_ref:        <wht_services Obligation>
  amount:                TZS  50,000
  payee_party_ref:       TRA
```

Each settles per CN-5-002 AP flow; Obligations reduce to zero; UI-08 holds.

**Tax-Advisor (Phase 1, Activated for Mzee Hassan):**

Pre-close suggestion (Nov 1): "VAT return for October due Nov 20. Estimated net payable TZS 810k based on partial period data. Suggested: review entries before closing."

Post-close suggestion (Nov 4 after close): "October tax close complete. VAT Statement STMT-VAT-MZH-2026-10 ready. **Click to download for TRA filing.** PAYE Statement STMT-PAYE-MZH-2026-10 ready. Filing deadline Nov 9 (PAYE) and Nov 20 (VAT)."

Year-end suggestion (Dec 31): "Annual PAYE certificate required by Jan 31 2027. Generate now via reporting.statement.issue.request for kind: annual_paye_certificate."

**Charter §1.3 boundary held throughout.** Mzee Tabu (chartered accountant) downloaded Statements, filed at TRA, instructed Mzee Hassan to settle. BOS did not file.

#### Backdated Assessment Scenario (Q13)

January 2028: TRA audits October 2026 VAT return + finds additional TZS 75,000 owed (alleged under-declared sales).

```
Mzee Tabu records assessment:
  accounting.tax.assessed.v1
    payload:
      tax_period_ref:               2026-10
      jurisdiction_ref:             TZ
      assessment_amount:            TZS 75,000
      assessment_kind:              vat
      reason:                       "TRA audit finding — additional output VAT"
      business_date:                2028-01-15        (current open period)
      posting_tax_period_ref:       2028-01           (current tax period)
      references_closed_period:     2026-10           (backward reference)

Creates new Obligation: kind: tax_assessment_payable; outstanding TZS 75,000
Settled via cash.payment.disbursed.v1 to TRA.

October 2026 tax period truth UNCHANGED.
Original STMT-VAT-MZH-2026-10 remains hash-frozen in store forever.
Audit walks: 2028 assessment → references_closed_period 2026-10 → original return.
```

---

## 18. Boundaries

| Topic | Lives in |
|-------|----------|
| Compliance DSL grammar + sandboxed evaluator | CN-4-015 (Foundation) |
| Tax pack content per jurisdiction (rates, exemptions, recovery, templates) | Term 1 via CTR-043 + CTR-029 |
| `tax_treatment_ref` input field shape | CN-4-021 |
| Output VAT recognition at settlement | CN-5-009 K3 + CN-5-001 #1 |
| Input VAT recognition at invoice | CN-5-004 + CN-5-001 #6 |
| Withholding tax computation | CN-5-005 §9 + CN-5-001 #10/#11 |
| Promotion discount tax treatment | CN-5-007 PR6 |
| Cash tax remittance to authority | CN-5-002 §G AR/AP flow |
| Tax obligation primitive kinds | CN-4-011 + TX12 |
| Tax-period state events | CN-5-001 manifest (this commit amendment) |
| Tax-period close mechanism | This doc (CN-5-105) §9 |
| Tax Return Statement Documents | CN-5-006 N3 + this doc §10 |
| Tax-advisor wiring | CN-5-010 §4.9 (this commit amendment) |
| Closed Tax-Period Inviolability invariant | CN-5-102 UI-11 (this commit amendment) |
| Naming compliance for new events | CN-5-103 G1-G9 |
| Financial period close (distinct from tax close) | CN-5-104 |
| Multi-jurisdiction full implementation | v2 future |
| Statutory filing / submission | Outside BOS (chartered accountant / tax officer per Charter §1.3) |

---

## 19. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Tax pack content per jurisdiction (TZ, KE, UG, etc.) — rates, exemptions, templates, schedules | Term 1 via CTR-043 + CTR-029 expansion | Schema in §§3, 5, 6, 7, 8, 10, 14; per-jurisdiction content is Term 1 + chartered-accountant authorship |
| Multi-jurisdiction full implementation (per-jurisdiction Obligation tracking, per-jurisdiction returns, cross-border services handling) | Future v2 | Framework hooks documented §11; engines emit jurisdiction_ref implicit in v1 |
| Reverse-charge VAT precise pack content per jurisdiction (OECD model defaults + jurisdiction overrides) | Term 1 via CTR-043 | §8 documents mechanism; per-jurisdiction defaults via pack |
| Tax-advisor learning from filing outcomes (post-filing, did suggestions help?) | Future + N5 of CN-5-010 | Pack opt-in `data_scope.includes_decision_journal: true` already supports; advisor v2 may use |
| Statutory return digital submission via authority APIs (TRA online, KRA iTax, etc.) | Future + Term 7 adapters | Charter §1.3 — BOS does not file; if pack permits "submission assistance" (generates submission file, tenant uploads), that's adapter scope; deferred |
| Tax-period freeze-analog window expiry default | Pack content + Term 1 | Parallel to CN-5-104 PC7 freeze window; 24h default reasonable |
| Year-end statutory close composite (annual PAYE + annual VAT summary + corporate income tax — distinct return) | Future per pack | Annual tax period for some kinds; pack declares |
| Multi-currency tax handling (FX revaluation of tax Obligations at tax-period close) | Future + coordination with CN-5-001 §11 | v1 single-currency tenant; FX coordination same pattern |
| Tax-advisor Phase 1 thresholds tuning | Pack content + tenant feedback | Initial defaults §15; pack may adjust per jurisdiction |
| Doctrine check for UI-11 (CN-4-019) | CN-4-019 + future DC | Static or integration test on closed tax_period_ref rejection at bus |
| Doctrine check for tax_treatment_ref presence on tax-relevant events | CN-4-019 + future DC | Schema validation pattern |
| Term 1 governance of `tenant_tax_profile` upgrades (when tenant crosses VAT threshold or registers formal employees) | Term 1 + CTR-043 expansion | Tenant-property event lifecycle; approval thresholds |

---

*— End of CN-5-105 —*

---

## Term 5 — Scope Complete

CN-5-105 is the **final cross-cutting doc of Term 5**. Upon ratification + merge:

- **15/15 docs** in Term 5 scope:
  - **5 cross-cutting:** CN-5-100 (Subscription Wiring Patterns), CN-5-101 (Scope Policy), CN-5-102 (Engine Invariants — now 11 with UI-11), CN-5-103 (Universal Event Glossary), CN-5-104 (Period-Close Choreography), CN-5-105 (Tax-Aware Engines)
  - **10 engine-numbered (incl. CN-5-009/010):** CN-5-001 Accounting, CN-5-002 Cash, CN-5-003 Inventory, CN-5-004 Procurement, CN-5-005 HR/Payroll, CN-5-006 Reporting/BI, CN-5-007 Promotion, CN-5-009 Universal Checkout/Tender, CN-5-010 AI Advisors Wiring (now 9 advisors with tax-advisor)

The universal-engines layer is complete. Term 6 (Vertical Engines) and Term 1/Term 3/Term 7 work proceeds from this foundation.
