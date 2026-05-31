# CN-5-006 — Reporting & BI Engine

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 5 — Universal Engines
> **Status:** For Overseer review.
> **Governing decisions:** Law 2 (engine isolation — Reporting reads events only; never writes business state; never sends notifications directly); Law 5 (compliance is configured — report templates, statutory summaries, KPI base library, export formats in packs); D-007 #10 (snapshots are non-truth, marker-tagged); D-009 (freeze doctrine — period-end statements interpreted under pack/template at issuance); Charter §1.3 (BOS is management truth + audit-ready data; the chartered accountant files, not BOS); UI-01 (causation chain on meta-events); UI-05 (closed-period inviolability); UI-06 (functional currency); UI-07 (trial balance).
> **CTRs:**
> - **OPEN (this doc extends with expansion notes at merge):** CTR-029 (Term 1 — pack gains: `statutory_summary_reports`, `reporting_snapshots` frequency policy, `export_templates` with authorised roles, `alternative_standard_views` mapping rules, KPI base library, alert delivery routing rules); CTR-030 (Term 6 — verticals may emit vertical-specific dimensional report events as catalogued per vertical).
> - **OPEN (this doc depends on):** CTR-027 (Term 1/2 — tenant functional currency + site registry); CTR-015 (Term 7 — AI Mode dashboard pattern; CN-5-006 surfaces projections; Term 7 records pattern).
> - **Open Item (future):** CTR-035 (Term 7 — filing-format export adapter generating machine-readable files matching jurisdiction submission specs; analogous to CTR-031 banking and CTR-034 payroll-filing).
> **Glossary:** See `MASTER-GLOSSARY.md` — Universal-engine Invariant (UI), Site, Scope level, Projection, Snapshot, Document.
> **Depends on:** CN-4-002 (envelope); CN-4-004 (command bus); CN-4-008 (audit log — Reporting may consume audit projections); CN-4-010 (Projection Framework — Reporting is the projection-heaviest engine); CN-4-011 (Workflow primitive for report lifecycle; Approval primitive for sensitive export gates; Document primitive for period-end statements — N3); CN-4-012 (Document Engine — issuance, hash, numbering, template versioning, freeze for period-end statements per N3); CN-4-013 (Decision Journal — Advisor decisions consumed); CN-4-014 (clock protocol — period boundaries, business_date); CN-4-018 (Snapshot Storage — non-truth performance cache); CN-4-022 (Advisor Framework — Reporting projections fed to advisors; CN-5-010 wires); CN-5-001/002/003/004/005/009 (all source engines whose events feed Reporting projections); CN-5-100 (subscription patterns — kind: projection); CN-5-101 (scope policy — Reporting is multi-scope per §6); CN-5-102 (UI-01/05/06/07 enforcement).
> **Boundaries:** Projection Framework mechanics → CN-4-010; Snapshot storage primitives → CN-4-018; Advisor Framework contract → CN-4-022; Document Engine mechanics → CN-4-012; business truth events → each owning engine; pack content per jurisdiction → Term 1 (CTR-029); BI/KPI Advisor wiring → CN-5-010; tenant-facing dashboard UX → Term 3 (CTR-004, CTR-015 AI Mode pattern); notification/alert delivery medium → Term 3 (in-dashboard) + Term 7 (channels per D-008); filing submission to tax authorities / pension funds → outside BOS per Charter §1.3.

---

## 1. What This Doc Defines

CN-5-006 is the Reporting & BI Engine — the **read-side / analytics truth layer** of BOS. It consumes events from every other universal engine plus verticals plus Foundation, and produces projections, KPIs, snapshots, formal period-end statements, statutory summaries, and exports.

Critical framing: Reporting **does not generate business truth**. Every Reporting projection is a derived view of events emitted by the engines that own those domains. Reporting rearranges, aggregates, and presents; it does not invent. Its emitted events are **meta-events** about reports themselves — never business state changes.

Two doctrine-critical distinctions govern the design:

1. **Snapshots ≠ Statements.** A projection snapshot (CN-4-018) is a non-truth performance cache; it may be discarded and rebuilt at any time. A period-end financial statement (CN-4-012 Document) is a formal hashed/numbered/template-versioned/pack-frozen artefact — verifiable years later by hash chain. **N3 (this doc)** establishes this as core doctrine.
2. **Reporting computes; HR/accountant files.** Per Charter §1.3, BOS produces audit-ready data and standard-compliant exports. The tenant's chartered accountant or HR officer files with tax authorities. BOS does not submit (parallel to CN-5-001 §12 and CN-5-005 §13).

This doc defines:
- The **engine manifest** with multi-scope per-operation overrides (§3).
- **Projection architecture** inheriting CN-4-010 (§4).
- **Live reports vs period-locked Documents** distinction (§5 / N1 / N3).
- **Financial reports** — pack-templated per CN-5-001 §6 (§6).
- **Operational reports** — sales, AR/AP ageing, inventory, procurement, workforce, cash (§7).
- **Statutory summary reports** — pack-defined, issued as Documents (§8 / N3).
- **KPI definitions and alerts** with delivery boundary (§9 / N5).
- **Snapshots** — CN-4-018 integration, performance cache only (§10).
- **Multi-standard reporting** — single ledger; alternative views via pack mapping (§11).
- **Export and filing-format generation** — Term 7 future per CTR-035 (§12).
- **BI Advisor integration boundary** — CN-5-010 owns; CN-5-006 provides (§13).
- **Real-time vs scheduled projections** (§14).
- **UI invariants compliance** (§15).

This doc does NOT define:
- Projection Framework mechanics (CN-4-010).
- Snapshot storage primitives (CN-4-018).
- Document Engine mechanics (CN-4-012).
- Advisor Framework contract (CN-4-022).
- Business truth events (each owning engine).
- Pack content per jurisdiction (Term 1 via CTR-029).
- BI/KPI Advisor wiring (CN-5-010).
- Dashboard / notification UX (Term 3).
- Filing submission (outside BOS).

---

## 2. The Seven Laws

| # | Law | Source / why |
|---|-----|--------------|
| **R1** | **Read-side only — no business truth emission.** Reporting emits ONLY meta-events about reports / snapshots / Documents / exports / alerts (`reporting.snapshot.created.v1`, `reporting.statement.issued.v1`, `reporting.export.generated.v1`, `reporting.alert.triggered.v1`, etc.). It NEVER creates a journal, never moves cash, never adjusts stock, never sends notifications directly (N5). | CN-4-001 Assertion 2 — projections are derived, never truth. Reporting is the analytics layer; other engines own truth; Term 3 + Term 7 own delivery. |
| **R2** | **All projections derive from event store.** Every Reporting projection is rebuildable from events alone per CN-4-010 §2. Rebuild is deterministic. Snapshots accelerate rebuild (CN-4-018) but are non-truth (D-007 #10). | Charter §1.2 (auditability); CN-4-001 Assertion 6 (state at any historical point reconstructable). |
| **R3** | **Pack-driven templates (Law 5).** Report sections, formula logic, statutory summary formats, export schemas, KPI base library — all in compliance pack `reporting_templates` section (extending CN-5-001 §6) + related new sections. Engine code is template-agnostic; pack content per jurisdiction. | Law 5; D-009 freeze — historical reports reproduce under historical templates. |
| **R4** | **Multi-scope: tenant default; site operational; platform extremely rare.** Most reports are tenant-wide (P&L, balance sheet, trial balance — A4 of CN-5-001). Operational dashboards may be site-scope (per-site sales, per-site inventory ageing). Platform scope is reserved for Term 1 billing aggregation per CN-5-101 §7 (aggregation happens at Term 1, not here — Reporting emits per-tenant). | CN-5-101 multi-scope; CN-5-101 §7 Chaguo C platform-aggregation pattern. |
| **R5** | **Single ledger; dual standard views via Reporting.** Per CN-5-001 Q5 ruling — tenant has ONE accounting standard (single ledger, single trial balance). When dual reporting needed (IFRS view derived from TFRS-TZ data, etc.), Reporting derives via **pack mapping rules** — never via parallel ledger. | CN-5-001 Q5 (single standard per tenant); CTR-029 pack mapping rules; avoids dual-ledger complexity. |
| **R6** | **Snapshots ≠ Statements (N3 — doctrine-critical).** A snapshot is a non-truth performance cache (CN-4-018); discardable; rebuildable. A statement (period-end P&L, balance sheet, statutory summary) is a formal Document via CN-4-012 — hashed, numbered, template-versioned, pack-frozen, terminal-fold, verifiable years later. These are **distinct primitives serving distinct purposes**; the engine uses both. | D-007 #10 (snapshots non-truth); CN-4-012 Document Engine; D-009 freeze; Charter §1.2 legal defensibility. |
| **R7** | **BI Advisor integration is read-only (Law 3).** When BI Advisor (CN-5-010, per D-005) reads Reporting projections to provide insights, the Advisor never writes events or commands (CN-4-022 §5). CN-5-006 provides the read surface; CN-5-010 wires the Advisor; Law 3 advisory-only enforced. | CN-4-022 Advisor Framework; D-005 AI Mode; Law 3 advisory-only. |

---

## 3. Engine Manifest

Reporting is a **multi-scope engine** per CN-5-101 §6: `tenant` default with explicit `site`-scope overrides for site operational reports.

```yaml
engine_id:        reporting
display_name:     Reporting & BI Engine
version:          1
scope_policy:     tenant                                # R4 default
primitives_used:
  - workflow                                            # report lifecycle (ad-hoc / scheduled)
  - approval                                            # sensitive export gates (statutory submissions, year-end statements)
  - document                                            # period-end statements as Documents (N3)

commands:
  # Snapshot management (R6 — performance cache only)
  - id: reporting.snapshot.create.request               scope_ref: tenant     # scheduled or manual
  - id: reporting.snapshot.invalidate.request           scope_ref: tenant     # for known-bad snapshots

  # Period-end Statements (N3 — formal Documents via CN-4-012)
  - id: reporting.statement.issue.request               scope_ref: tenant     # period-end P&L, BS, CFS, statutory summaries — authorised role required
  - id: reporting.statement.amend.request               scope_ref: tenant     # issues a NEW statement Document referencing original (CN-4-012 §6 correction pattern)

  # Ad-hoc reports (live, transient)
  - id: reporting.report.run.request                    # scope_ref: tenant (default; site override for site reports)

  # KPIs
  - id: reporting.kpi.define.request                    scope_ref: tenant
  - id: reporting.kpi.update.request                    scope_ref: tenant
  - id: reporting.kpi.deactivate.request                scope_ref: tenant

  # Alerts
  - id: reporting.alert.set.request                     scope_ref: tenant
  - id: reporting.alert.deactivate.request              scope_ref: tenant

  # Exports
  - id: reporting.export.generate.request               scope_ref: tenant     # pack-defined templates; authorised role per template

  # Projection management
  - id: reporting.projection.rebuild.request            scope_ref: tenant

emits:
  # Meta-events about reports (R1 — never business truth)
  - reporting.snapshot.created.v1
  - reporting.snapshot.invalidated.v1
  - reporting.statement.issued.v1                       # N3 — Document terminal-fold event
  - reporting.statement.amended.v1                      # N3 — new Document referencing original
  - reporting.report.run.v1                             # live report meta-event
  - reporting.kpi.defined.v1
  - reporting.kpi.updated.v1
  - reporting.kpi.deactivated.v1
  - reporting.kpi.recomputed.v1                         # when projection refresh changes KPI value
  - reporting.alert.set.v1
  - reporting.alert.triggered.v1                        # threshold breach (N5 — meta only; Term 3/7 deliver)
  - reporting.alert.cleared.v1
  - reporting.export.generated.v1                       # export file ready for download
  - reporting.projection.rebuilt.v1                     # system event after rebuild

subscribes_to:
  # ── FINANCIAL LAYER (primary truth from Accounting) ──
  - {event_type: accounting.journal.posted.v1,            version: 1, handler: update_financial_projections,    kind: projection,   scope_ref: tenant}
  - {event_type: accounting.journal.reversed.v1,          version: 1, handler: update_financial_projections,    kind: compensation, scope_ref: tenant}
  - {event_type: accounting.period.closed.v1,             version: 1, handler: finalize_period_snapshot,        kind: projection,   scope_ref: tenant}
  - {event_type: accounting.depreciation.posted.v1,       version: 1, handler: update_financial_projections,    kind: projection,   scope_ref: tenant}

  # ── CASH LAYER ──
  - {event_type: cash.tender.received.v1,                 version: 1, handler: update_cash_projections,         kind: projection,   scope_ref: tenant}
  - {event_type: cash.tender.disbursed.v1,                version: 1, handler: update_cash_projections,         kind: projection,   scope_ref: tenant}
  - {event_type: cash.expense.recorded.v1,                version: 1, handler: update_cash_projections,         kind: projection,   scope_ref: tenant}
  - {event_type: cash.collection.received.v1,             version: 1, handler: update_ar_ageing,                kind: projection,   scope_ref: tenant}
  - {event_type: cash.payment.disbursed.v1,               version: 1, handler: update_ap_ageing,                kind: projection,   scope_ref: tenant}
  - {event_type: cash.session.reconciled.v1,              version: 1, handler: update_cash_operations,          kind: projection,   scope_ref: tenant}
  - {event_type: cash.variance.detected.v1,               version: 1, handler: update_cash_operations,          kind: projection,   scope_ref: tenant}
  - {event_type: cash.deposit.completed.v1,               version: 1, handler: update_cash_projections,         kind: projection,   scope_ref: tenant}

  # ── INVENTORY LAYER ──
  - {event_type: inventory.stock.received.v1,             version: 1, handler: update_inventory_projections,    kind: projection,   scope_ref: tenant}
  - {event_type: inventory.stock.deducted.v1,             version: 1, handler: update_inventory_projections,    kind: projection,   scope_ref: tenant}
  - {event_type: inventory.stock.adjusted.v1,             version: 1, handler: update_inventory_projections,    kind: projection,   scope_ref: tenant}
  - {event_type: inventory.lot.expired.v1,                version: 1, handler: update_inventory_expiry,         kind: projection,   scope_ref: tenant}
  - {event_type: inventory.adjustment.recorded.v1,        version: 1, handler: update_inventory_projections,    kind: projection,   scope_ref: tenant}
  - {event_type: inventory.reservation.created.v1,        version: 1, handler: update_inventory_reservations,   kind: projection,   scope_ref: tenant}
  - {event_type: inventory.reservation.confirmed.v1,      version: 1, handler: update_inventory_reservations,   kind: projection,   scope_ref: tenant}
  - {event_type: inventory.reservation.released.v1,       version: 1, handler: update_inventory_reservations,   kind: projection,   scope_ref: tenant}

  # ── PROCUREMENT LAYER ──
  - {event_type: procurement.po.created.v1,               version: 1, handler: update_procurement_projections,  kind: projection,   scope_ref: tenant}
  - {event_type: procurement.grn.received.v1,             version: 1, handler: update_procurement_projections,  kind: projection,   scope_ref: tenant}
  - {event_type: procurement.invoice.recorded.v1,         version: 1, handler: update_procurement_projections,  kind: projection,   scope_ref: tenant}
  - {event_type: procurement.invoice.matched.v1,          version: 1, handler: update_procurement_match_rates,  kind: projection,   scope_ref: tenant}
  - {event_type: procurement.invoice.dispute_raised.v1,   version: 1, handler: update_procurement_disputes,     kind: projection,   scope_ref: tenant}
  - {event_type: procurement.invoice.paid.v1,             version: 1, handler: update_procurement_projections,  kind: projection,   scope_ref: tenant}
  - {event_type: procurement.credit_note.recorded.v1,     version: 1, handler: update_procurement_projections,  kind: compensation, scope_ref: tenant}

  # ── HR / PAYROLL LAYER ──
  - {event_type: hr.payroll.computed.v1,                  version: 1, handler: update_payroll_projections,      kind: projection,   scope_ref: tenant}
  - {event_type: hr.payroll.paid.v1,                      version: 1, handler: update_payroll_projections,      kind: projection,   scope_ref: tenant}
  - {event_type: hr.payroll.deduction.applied.v1,         version: 1, handler: update_payroll_projections,      kind: projection,   scope_ref: tenant}
  - {event_type: hr.leave.accrued.v1,                     version: 1, handler: update_leave_balances,           kind: projection,   scope_ref: tenant}
  - {event_type: hr.leave.approved.v1,                    version: 1, handler: update_leave_balances,           kind: projection,   scope_ref: tenant}
  - {event_type: hr.attendance.recorded.v1,               version: 1, handler: update_attendance_metrics,       kind: projection,   scope_ref: tenant}
  - {event_type: hr.loan.approved.v1,                     version: 1, handler: update_loan_projections,         kind: projection,   scope_ref: tenant}
  - {event_type: hr.termination.finalized.v1,             version: 1, handler: update_headcount,                kind: projection,   scope_ref: tenant}

  # ── CHECKOUT / SALES LAYER ──
  - {event_type: checkout.settled.v1,                     version: 1, handler: update_sales_projections,        kind: projection,   scope_ref: tenant}
  - {event_type: checkout.refunded.v1,                    version: 1, handler: update_sales_projections,        kind: compensation, scope_ref: tenant}

  # ── VERTICAL-DIMENSIONAL EVENTS (CTR-030 expansion — N4) ──
  # Per-vertical entries grow manifest as Term 6 catalogues verticals.
  # Examples (future):
  #   - <workshop>.project.profitability.recorded.v1
  #   - <hotel>.occupancy.recorded.v1
  #   - <pharmacy>.dispense.recorded.v1
  # v1 catalogue is empty; populated per CTR-030 vertical contracts.

requires:
  - accounting.journal.posted.v1                          # cannot produce financial reports without journals
```

### Why So Many Subscriptions

Reporting is the natural fan-in point of the system. The manifest grows by addition as engines emit new events. The subscription `kind` is overwhelmingly `projection` — Reporting updates its read-models and stops; it does not submit follow-up commands (R1).

---

## 4. Projection Architecture

Reporting inherits CN-4-010 Projection Framework. Per-projection patterns:

| Projection family | Type | Update | Source events |
|-------------------|------|--------|---------------|
| **Trial balance** | Tenant-wide running totals per account | Real-time | `accounting.journal.posted.v1`, `.reversed.v1` |
| **P&L by period** | Period-aggregated revenue/expense per pack template | Real-time + period-close snapshot/Document | Accounting journals + pack templates |
| **Balance sheet** | Point-in-time asset/liability/equity per pack template | Real-time + period-close snapshot/Document | Accounting journals + pack templates |
| **Cash flow statement** | Period-aggregated cash flows per pack categorisation | Period-aggregated + period-close Document | Cash events + Accounting categorisation |
| **AR / AP ageing** | Outstanding by age bucket | Real-time | Obligation events + Cash settlement events |
| **Inventory valuation** | Stock value per item / site / lot | Real-time | Inventory movements + costing |
| **Inventory ageing / expiry** | Stock by age + expiry watch | Real-time + scheduled rebuild | Inventory lots + receive dates + expiry |
| **Sales analytics** | Revenue by site / hour / item / channel | Real-time | `checkout.settled.v1` + dimensional tags |
| **Procurement spend** | Spend by supplier / category / period | Real-time | Procurement events |
| **Workforce metrics** | Headcount / leave / attendance / overtime | Real-time | HR events |
| **KPI projections** | Tenant-defined + pack base library | Real-time or scheduled per KPI definition | Multiple per KPI |

Each projection follows CN-4-010: `processed_position` tracked, idempotent on re-delivery, deterministic on rebuild.

---

## 5. Live Reports vs Period-Locked Documents (N1, N3)

A central doctrine distinction in CN-5-006:

| Aspect | **Live Report** | **Period-Locked Statement** |
|--------|-----------------|------------------------------|
| Definition | Read-time evaluation against current projection | Formal Document issued at period close (CN-4-012) |
| Truth status | Transient view; not audit-reproducible without separate artefact | Immutable formal artefact; hash-verified; replay-reproducible under historical pack/template |
| Mechanism | `reporting.report.run.request` → handler evaluates → returns result | `reporting.statement.issue.request` → CN-4-012 Document issued → `reporting.statement.issued.v1` |
| Persistence | Result not stored as Document; may be cached via snapshot | Document is permanent terminal-fold record (CN-4-011 Document fold) |
| Number / hash | None inherent | Has document number + content hash + template_version_ref + pack_version_ref |
| Years-later verification | Requires replay + reconstruction from events | Self-contained — hash check proves integrity |
| Use case | Management dashboard, exploratory analytics, what-if | Filing-grade evidence, audit trail, board package |

### When to Use Which

| Need | Use |
|------|-----|
| "What's my cash position right now?" | Live report |
| "What was my Q3 2026 P&L for filing?" | Period-locked Statement (Document) |
| "Show me top-selling items today" | Live report |
| "Year-end balance sheet for auditor" | Period-locked Statement |
| "Monthly P&L for board pack" | Period-locked Statement (issued each month at close) |
| "Custom KPI dashboard" | Live report (with KPI projection updated real-time) |

### N3 Doctrine — Statements ARE Documents

A period-end financial statement (P&L, balance sheet, cash flow, statutory summary) is **not** "a snapshot of the projection." It is a **formal Document via CN-4-012** with:
- Document number (per pack numbering rules)
- Content hash (over the assembled report content + template_version_ref + pack_version_ref + number)
- Template version (the pack's `reporting_templates` version active at issuance — D-009 freeze)
- Terminal fold (CN-4-011 Document — issued once, never modified)
- Optional cryptographic signature (CTR-017 future, for external authenticity)

A P&L for 2026 can be verified in 2046 by:
1. Retrieve the `reporting.statement.issued.v1` event from the store.
2. Recompute the hash from stored content + references.
3. Compare with stored hash.
4. Optionally replay the underlying events under the historical pack version to reproduce the content.

Snapshots are an **independent mechanism** (§10) — performance cache for rebuild; non-truth; discardable. The two co-exist; they serve different needs.

### Amendments

If a closed period's statement is later found to contain an error (rare; usually caught at close), the correction follows CN-4-012 §6: **a new statement Document** is issued (e.g., "P&L 2026 — Amendment A") referencing the original. Both Documents remain hash-verifiable. The original is never modified.

---

## 6. Financial Reports (Pack-Templated per CN-5-001 §6)

Reporting consumes the `reporting_templates` section of the accounting pack:

```yaml
# In pack content (CN-5-001 §6.3)
reporting_templates:
  profit_and_loss:
    template_version:    v3
    sections:
      - {name: "Revenue",           account_groupings: ["4000-4999"]}
      - {name: "Cost of Sales",     account_groupings: ["5000-5099"]}
      - {name: "Gross Profit",      formula: "Revenue - Cost of Sales"}
      - {name: "Operating Expense", account_groupings: ["6000-6999"]}
      - {name: "Operating Profit",  formula: "Gross Profit - Operating Expense"}
  balance_sheet:
    template_version:    v2
    sections: [...]
  cash_flow_statement:
    template_version:    v1
    sections: [...]
```

### Live View

`reporting.report.run.request {kind: profit_and_loss, period: 2026-Q4}` returns the result as a transient response. No Document issued.

### Period-Locked Issuance (N3)

`reporting.statement.issue.request {kind: profit_and_loss, period: 2026-Q4}` is the formal-issuance command. Bus:
- Verifies authorised role (per pack `export_templates.authorised_role`)
- Verifies period is closed (UI-05 — a P&L issued for an open period is provisional and clearly marked; pack governs whether provisional issuance is permitted)
- Evaluates template against trial balance at period-close `global_position`
- Issues Document via CN-4-012: hash + number + template_version + pack_version_ref
- Emits `reporting.statement.issued.v1` with document_ref

### Template Versioning + Freeze (D-009)

Each statement Document records the template version + pack version at issuance. Future template upgrades (e.g., new P&L section ordering) apply only to new statements. Old statements remain under their original template forever (CN-4-012 §4).

---

## 7. Operational Reports

Beyond financial reports, Reporting produces operational analytics (mostly live; some scheduled with snapshots).

### Sales Analytics

- Revenue by site / item / category / hour / day-of-week / channel
- Tender mix (cash vs mobile money vs card per period)
- Refund rate by site / item / cashier
- Average transaction value, units per transaction
- Top-N items, top-N customers

### AR / AP Ageing

- Outstanding receivables by customer + age bucket (0-30 / 31-60 / 61-90 / 90+)
- Outstanding payables by supplier + age bucket
- Top debtors, top creditors
- Overdue items past due_date

### Inventory Reports

- Stock-on-hand by item / site / lot
- Stock valuation (per current costing method)
- Slow-moving items (no movement in N days, pack-defined)
- Expiry watch (lots expiring within X days)
- Stockout incidents (deductions blocked per CN-5-003 I3)

### Procurement Reports

- Spend by supplier / category / period
- Three-way-match auto-pass rate
- Average lead time per supplier
- Supplier performance (on-time delivery, quality)
- Open POs (not yet fully received)

### Workforce Metrics

- Headcount by site / role
- Leave balance by employee / leave type
- Attendance compliance
- Overtime trends
- Outstanding employee loans by employee + period

### Cash Operations

- Cash position by till (real-time)
- Variance trends per till
- Drop frequency and amounts
- Deposit pipeline (main safe → bank)
- Cash flow by site by day

### Vertical-Dimensional Reports (N4 — Future via CTR-030)

Vertical-specific reports — workshop project profitability, hotel occupancy by room type, pharmacy dispensing patterns, restaurant table turnover — depend on vertical event subscriptions. **v1 catalogue is empty here**; reports are added as Term 6 catalogues verticals via CTR-030 expansion. No vertical-specific report is hard-coded in v1.

---

## 8. Statutory Summary Reports (Pack-Defined; Issued as Documents per N3)

Statutory summaries are produced per pack templates that match jurisdiction submission needs. Per N3, these are issued as **Documents** (not snapshots) for legal defensibility.

```yaml
# In pack content (CTR-029 expansion)
statutory_summary_reports:
  - {code: "paye_summary_monthly",
      period:               monthly,
      template_ref:         "tz-paye-summary-template-v3",
      account_groupings:    ["2310"],
      grouping_dimensions:  [employee_ref],
      issued_as_document:   true,                       # N3 — formal Document
      authorised_role:      [hr_officer, owner],
      number_scope:         "tenant + statutory_paye + period"}

  - {code: "pension_summary_monthly",
      period:               monthly,
      template_ref:         "tz-pension-summary-v2",
      account_groupings:    ["2330", "5180"],
      issued_as_document:   true,
      authorised_role:      [hr_officer, accountant]}

  - {code: "nhif_summary_monthly",
      period:               monthly,
      template_ref:         "tz-nhif-summary-v1",
      issued_as_document:   true,
      authorised_role:      [hr_officer]}

  - {code: "vat_summary_monthly",
      period:               monthly,
      template_ref:         "tz-vat-summary-v4",
      account_groupings:    ["2300", "1310"],
      issued_as_document:   true,
      authorised_role:      [accountant, owner]}

  - {code: "annual_paye_certificate",
      period:               annual,
      template_ref:         "tz-annual-paye-cert-v2",
      grouping_dimensions:  [employee_ref],
      issued_as_document:   true,
      authorised_role:      [hr_officer, owner]}
```

### Issuance Flow

```
reporting.statement.issue.request {kind: statutory_paye_summary, period: 2026-08}
  → bus verifies authorised role per pack
  → Reporting evaluates template against payroll projections
  → CN-4-012 Document issued: number, hash, template_version, pack_version_ref
  → reporting.statement.issued.v1 emitted with document_ref
```

These produce **report Document artefacts**, not filings. The tenant's HR officer / accountant uses the Document for filing. Per Charter §1.3, BOS does not submit. Filing-format machine-readable files (matching tax authority API specs) → **CTR-035 future** Term 7 adapter.

---

## 9. KPI Definitions + Alerts

### KPI Definition

Pack declares a base KPI library:

```yaml
# In pack content (CTR-029 expansion)
kpi_base_library:
  - {code: "sales_growth_mom",        name: "Sales Growth Month-on-Month",
      computation: {basis: "monthly_revenue", formula: "(current - previous) / previous * 100"}}
  - {code: "gross_margin",             name: "Gross Margin %",
      computation: {basis: "gross_profit / revenue * 100"}}
  - {code: "inventory_turnover",       name: "Inventory Turnover",
      computation: {basis: "cogs_period / avg_inventory_value"}}
  - {code: "cash_runway_days",         name: "Cash Runway Days",
      computation: {basis: "current_cash / avg_daily_burn"}}
  - {code: "ar_dso",                   name: "Days Sales Outstanding",
      computation: {basis: "ar_outstanding / (revenue / days_in_period)"}}
```

Tenants may define custom KPIs within pack-permitted scope:

```yaml
reporting.kpi.define.request:
  scope:           tenant
  payload:
    kpi_id:                <tenant-unique>
    kpi_name:              "Mwanza Site Revenue Share"
    computation:           {numerator_source, denominator_source, formula}
    dimension_filters:     [{site_id: mwanza-001}]
    refresh_frequency:     real_time | hourly | daily
```

### Alerts (Threshold-Based)

```yaml
reporting.alert.set.request:
  scope:           tenant
  payload:
    kpi_ref, threshold_condition, severity: info | warning | critical
    notification_routing_ref:   <pack-defined routing rule — N5>
```

When KPI breaches threshold:
- `reporting.alert.triggered.v1` meta-event emitted
- Per pack `notification_routing_ref`, the alert is routed to delivery via Term 3 (in-dashboard surface) or Term 7 (channels per D-008)
- **Reporting does NOT send the notification itself** (N5)

### N5 — Alert Delivery Boundary

Per Law 2 (engine isolation) + the channel-as-integration ruling (D-008): Reporting does not own the medium of delivery. Reporting **emits the meta-event**; Term 3 (dashboard surface) or Term 7 (SMS / email / WhatsApp / Telegram channel adapters) handle delivery.

The pack declares `notification_routing_ref` which Term 7 resolves; Reporting's responsibility ends at meta-event emission. This is the same boundary used in CN-5-002 (variance detection emits anomaly meta-event; Term 7 routes), CN-5-003 (lot expiry meta-event), etc.

---

## 10. Snapshots (CN-4-018 Integration — Non-Truth Cache)

Snapshots accelerate projection rebuild without compromising determinism. Per R6 / N3, they are **distinct from Statements** (which are formal Documents).

### Scheduled Snapshots

Pack declares snapshot frequency per projection family:

```yaml
# In pack content (CTR-029 expansion)
reporting_snapshots:
  trial_balance:               daily
  p_and_l_running:             daily
  balance_sheet_running:       daily
  inventory_valuation:         daily
  ar_ap_ageing:                hourly
  sales_analytics:             hourly
```

`system:scheduler` submits `reporting.snapshot.create.request` per frequency.

### Manual Snapshots

For audit / dispute / point-in-time queries, authorised role submits `reporting.snapshot.create.request` with target `global_position` or `business_date`.

### Non-Truth Marker (D-007 #10)

Every snapshot carries CN-4-018 non-truth marker. DC-025 (CN-4-019) enforces presence; DC-026 enforces no-code-treats-snapshot-as-source. A snapshot disagreeing with full replay → replay wins; snapshot invalidated.

### Snapshots vs Statements — Operational Use

| Aspect | Snapshot (CN-4-018) | Statement (CN-4-012 Document) |
|--------|---------------------|-------------------------------|
| Purpose | Accelerate projection rebuild | Formal period-end record |
| Lifecycle | Created / invalidated / replaced anytime | Issued once; never modified |
| Hash | None | Yes (content + refs hash) |
| Number | None | Yes (per pack numbering) |
| Discardable | Yes (any time) | No (immutable per CN-4-001) |
| Audit value | None standalone (must be combined with events) | Self-contained, hash-verifiable |
| Typical user | System (performance) | Auditor, regulator, board |

Both exist in CN-5-006; they serve different needs and never conflate (R6).

---

## 11. Multi-Standard Reporting (R5)

Per CN-5-001 Q5: tenant has ONE accounting standard (single ledger). When dual reporting is needed (IFRS view + statutory view for the same tenant), Reporting derives the alternative view via **pack mapping rules**:

```yaml
# In pack content
alternative_standard_views:
  - {view_id:                "ifrs_view_from_tfrs_tz",
      base_standard:          "TFRS-TZ",
      derived_standard:       "IFRS",
      account_remapping:      [{from: "5100", to: "5050"}, ...],
      section_remapping:      [{from: "OperatingExpense.Rent", to: "OperatingExpense.LeaseExpense"}, ...],
      formula_overrides:      [{section: "GrossProfit", formula: "<IFRS-formula>"}],
      reporting_templates:    {profit_and_loss: <IFRS-templated sections>, balance_sheet: ...}}
```

Reporting evaluates the alternative view against the same trial balance projection but with the pack's remapping rules applied. The auditor sees one truth in two presentations.

Both the base-standard Statement and the alternative-standard Statement can be issued as separate Documents per N3 (each with its own number, hash, template/standard reference). Both are valid; they present the same underlying truth.

---

## 12. Export & Filing-Format Generation

### Export Categories

| Category | Purpose | Format |
|----------|---------|--------|
| **Dashboard data exports** | Tenant downloads chart data for spreadsheets | CSV / JSON / Excel |
| **Statutory summary exports** | HR officer / accountant downloads filing data | Pack-defined CSV with specific columns |
| **Filing-format files** (machine-readable for tax authority APIs) | PAYE file, VAT file, pension file | **CTR-035 future** — Term 7 adapter per jurisdiction |

### Pack-Defined Export Templates

```yaml
# In pack content (CTR-029 expansion)
export_templates:
  - {code: "paye_filing_export",   template_ref: "tz-paye-export-v3",   format: csv,
      authorised_role: [hr_officer, owner]}
  - {code: "vat_filing_export",    template_ref: "tz-vat-export-v2",    format: csv,
      authorised_role: [accountant, owner]}
  - {code: "p_and_l_excel",        template_ref: "tz-pl-excel-v1",      format: excel,
      authorised_role: [owner, manager]}
```

Authorised roles for sensitive exports declared per template. Bus policy enforces.

### Export vs Statement

| | Export | Statement (Document) |
|--|--------|----------------------|
| Hashable | Optional (file can be hashed for transit integrity) | Yes (CN-4-012 hash) |
| Numbered | No (filename, optionally with date) | Yes (per pack numbering) |
| Permanent record | The export file is downloaded; not necessarily kept | Document is permanent in event store |
| Use case | Hand-off to external system / human filing | Audit trail, formal record |

Exports may be **derived from** Statements (e.g., year-end P&L Statement → Excel export for board pack) or directly from projections (e.g., today's sales CSV for marketing analysis).

---

## 13. BI Advisor Integration Boundary (R7)

CN-5-006 provides the read surface; **CN-5-010 owns the BI Advisor wiring** (per D-002A + D-005). The split:

| Concern | Lives in |
|---------|----------|
| Projections, KPIs, snapshots, Statements (Documents), exports | CN-5-006 |
| Advisor instance definition (audience, scope, model, prompt assembly) | CN-5-010 + CN-4-022 Advisor Framework |
| Advisor reading Reporting projections | CN-5-010 invokes; CN-5-006 serves |
| Advisor suggestion → Decision Journal | CN-4-013 (Foundation) |
| Advisor commits to action | **Never** — Law 3 advisory only (CN-4-022 §5) |

Reporting projections are **read-only data** to the Advisor. The Advisor produces inert draft commands per CN-4-022 §5 that a human submits through the bus — never autonomously.

---

## 14. Real-Time vs Scheduled Projections

| Pattern | When applies | Examples |
|---------|--------------|----------|
| **Real-time** | KPIs feeding live dashboards (cash position, current sales) | Trial balance, AR/AP ageing, current cash position, today's sales |
| **Period-aggregated** | P&L per month, cash flow per quarter — computed on demand | Live P&L view; periodic Statements at close |
| **Scheduled rebuild** | Heavy aggregations (year-over-year comparisons, top-N analyses) snapshotted at scheduled intervals | Historical revenue trends, supplier performance reports |
| **Manual rebuild** | Following doctrine change, bug fix, or corruption | Engineer-initiated full rebuild |

Pack declares frequency per projection family. Tenant override allowed within pack permission.

---

## 15. UI Invariants Compliance

| Invariant | Application in CN-5-006 |
|-----------|------------------------|
| **UI-01** (causation chain) | Meta-events (`reporting.snapshot.created.v1`, `reporting.statement.issued.v1`, etc.) carry causation_id to triggering command. Audit walks the issuance trail. |
| **UI-05** (closed-period inviolability) | Period-end Statements locked at close per CN-4-011 Document terminal fold; subsequent corrections via amendment Documents (N3) referencing original. Closed-period figures in original Statement never change. |
| **UI-06** (functional currency) | All reports in tenant functional currency by default; multi-currency reports require explicit FX events from Accounting. |
| **UI-07** (trial balance) | Reporting's trial-balance projection MUST balance for any tenant at any `global_position`. Integration test asserts at every rebuild; runtime drift escalates per CN-4-017 to DEGRADED. |

---

## 16. Example — Karakana Year-End + Mzee Hassan Dashboard

*Karakana ya Mzee Hassan serves as illustrative context per D-004 #4 (peer-technical audience). The example demonstrates live reports + period-locked Statements (Documents) + statutory summary exports + Charter §1.3 boundary.*

**Setting:** End of December 2026. Karakana operates two sites (Mwanza + Mbeya), tenant functional currency TZS, pack `tz-compliance-2026.07` (TFRS-TZ).

### Mzee Hassan's Live Dashboard (Today)

Per CN-5-006's real-time projections:

- **Cash position**: TZS 8.2M across 6 tills (Cash projections updating on every `cash.tender.*` event)
- **Today's sales**: TZS 1.8M across 2 sites (sales projection on `checkout.settled.v1`)
- **AR outstanding**: TZS 850K (AR ageing projection from Obligation primitive + `cash.collection.received.v1`)
- **AP outstanding**: TZS 2.1M (AP ageing projection from Procurement + `cash.payment.disbursed.v1`)
- **Stock value**: TZS 14.5M across 2 sites (Inventory valuation projection)
- **Top-selling item this week**: Aluminium windows (sales analytics by item)
- **Variance alert**: M-Pesa wallet Mbeya TZS 5,200 over expected — `reporting.alert.triggered.v1` emitted; routed via Term 3 in-dashboard surface (N5)

All of this is **live reports** — read-time evaluation against current projections. Not Documents; transient.

### Year-End P&L Statement (December 31)

Mzee Hassan's accountant submits:

```
reporting.statement.issue.request
  scope: tenant
  payload:
    kind:        profit_and_loss
    period:      2026 (annual)
    pack_version_ref: tz-compliance-2026.07
  required_role: accountant
```

Bus:
- Verifies authorised role ✓
- Verifies period closed via Accounting period state projection (UI-05) — yes, December 31 closed via CN-5-104 choreography
- Evaluates pack template `tz-compliance-2026.07.reporting_templates.profit_and_loss` against trial balance projection at year-end `global_position`
- Result: Revenue TZS 142M, COGS TZS 89M, Gross Profit TZS 53M, OpEx TZS 38M, Operating Profit TZS 15M
- **Issues Document** via CN-4-012: number `STMT-PL-KRK-2026-A`, content hash, template_version_ref v3, pack_version_ref `tz-compliance-2026.07`
- Emits `reporting.statement.issued.v1 {document_ref, kind: profit_and_loss, period: 2026}`

The Statement is **permanent** — verifiable in 2046 by recomputing the hash from stored content + references, or by replaying events under the historical pack to reproduce.

Same flow for `balance_sheet` and `cash_flow_statement` Statements.

### Statutory Summary Statements (December)

For the monthly PAYE summary:

```
reporting.statement.issue.request
  payload:
    kind:        statutory_paye_summary_monthly
    period:      2026-12
  required_role: hr_officer
```

Bus:
- Authorised role ✓
- Evaluates pack template `tz-paye-summary-template-v3` against payroll projections
- Issues Document `STMT-PAYE-KRK-2026-12`, hashed, numbered, template-frozen
- Emits `reporting.statement.issued.v1`

Same for pension_summary, nhif_summary, vat_summary.

### Filing Export (CSV for Tax Authority)

Mzee Hassan's HR officer submits:

```
reporting.export.generate.request
  payload:
    template:    paye_filing_export
    period:      2026
  required_role: hr_officer
```

Bus produces CSV file matching TRA submission spec; emits `reporting.export.generated.v1` with file reference.

HR officer downloads the file → submits to TRA's online portal. **BOS did not file.** Charter §1.3 boundary held.

### Live Year-Over-Year Comparison (Dashboard)

Mzee Hassan asks his dashboard: "Compare 2026 revenue to 2025."

`reporting.report.run.request {kind: yoy_revenue_comparison, current_year: 2026, prior_year: 2025}` evaluates against current trial-balance projection + 2025 period-close Statement (loaded from Document). Returns transient view; not a Document.

### BI Advisor Suggestion (CN-5-010, Future)

The BI Advisor (CN-5-010, wired per D-005 AI Mode) reads Reporting projections; observes inventory turnover dropped 12% in Q4 vs Q3.

Advisor produces draft suggestion: "Consider reorder review for slow-moving items." Decision Journal records (CN-4-013): model identity, prompt ref, data ref, recommendation.

Mzee Hassan reviews via dashboard; if approved, he initiates a procurement reorder review himself (no autonomous action — R7 / Law 3 holds).

### Pack Upgrade Mid-Period

In April 2027, Tanzania publishes updated VAT rates effective May 1. New pack version `tz-compliance-2027.05`.

The **December 2026 P&L Statement** issued earlier remains under `tz-compliance-2026.07` — D-009 freeze. Replaying or re-verifying that Statement uses the historical pack/template versions. The new pack applies only to new Statements issued after May 1, 2027.

This is exactly the value of N3 (Statements as Documents): a board pack from 2026 stays interpretable under 2026 rules forever, regardless of subsequent pack rotations.

---

## 17. Three Whys

### Why does this matter?

A business's truth lives in many engines — sales in Checkout, journals in Accounting, stock in Inventory, money in Cash, supplier obligations in Procurement, payroll in HR. The owner cannot run a business by querying ten engines separately; the regulator cannot audit by stitching ten different report formats; the board cannot make decisions on data they cannot trust. CN-5-006 makes the read-side first-class: every engine's events feed projections; every projection serves dashboards, KPIs, and statutory summaries; every period-end produces formal Statements (Documents) that hash-verify decades later. Mzee Hassan sees one coherent picture in his dashboard; his auditor verifies one P&L Statement document; his tax officer files from one statutory summary. The truth was always in the events; CN-5-006 makes it visible.

### Why does it belong here (and not in Foundation)?

Foundation provides the Projection Framework (CN-4-010), Snapshot Storage (CN-4-018), Document Engine (CN-4-012), Advisor Framework (CN-4-022), and the audit log (CN-4-008). Foundation does not say "every business needs a P&L per pack template," "statutory summaries are issued as Documents," "alert delivery goes to Term 3/7 not Reporting," or "tenants define KPIs within pack-permitted scope." Those are universal Reporting choices appropriate for any business. CN-5-006 makes those choices as the universal Reporting pattern. A specialised analytics engine (e.g., fraud-detection-only) could compose primitives differently — but that is not what universal Reporting is.

### Why this design?

The fan-in subscription model (R1 + huge subscription list) keeps Reporting the natural integration point — engines emit; Reporting consumes; truth converges. The live-vs-Statement distinction (N3) honours the two distinct needs: management agility (live dashboards) + legal defensibility (formal Statements as Documents). Snapshots as non-truth performance cache (R6) keeps the cache layer honest — never confused with truth. Pack-driven everything (R3) operationalises Law 5 for the read side: adding a new country's report formats = pack content, not engine code. Multi-standard via mapping (R5) avoids dual-ledger; preserves single source of truth. BI Advisor boundary (R7) keeps Reporting's read surface clean while honouring Law 3 (the Advisor cannot commit). Alert delivery boundary (N5) keeps Reporting from owning channels it should not own. The whole engine is one cohesive answer to "how does any business, in any jurisdiction, see its own truth and prove it to others?"

---

## 18. Boundaries

| Topic | Lives in |
|-------|----------|
| Projection Framework mechanics (processed_position, idempotency, rebuild) | CN-4-010 |
| Snapshot Storage primitives (non-truth marker, validation) | CN-4-018 |
| Document Engine mechanics (hash, numbering, template versioning, terminal fold) | CN-4-012 |
| Advisor Framework contract (audience, scope, model, decision journal fields) | CN-4-022 |
| Audit log (Reporting may consume audit projections) | CN-4-008 |
| Event envelope | CN-4-002 |
| Command bus | CN-4-004 |
| Clock protocol (period boundaries, business_date) | CN-4-014 |
| Pack content per jurisdiction (templates, KPI library, export formats, statutory summary specs) | Compliance packs authored under CTR-029 (Term 1) |
| Business truth events (revenue, expense, cash, stock, AR/AP, payroll) | Each owning engine (CN-5-001/002/003/004/005/009) |
| Subscription patterns + replay semantics + kind: projection / compensation | CN-5-100 |
| Scope policy (multi-scope per §6 pattern) | CN-5-101 |
| Cross-engine invariants (UI-01, UI-05, UI-06, UI-07) | CN-5-102 |
| Vertical-dimensional report event contracts | Term 6 via CTR-030 expansion (N4) |
| Tenant functional currency + site registries | Term 1 + Term 2 via CTR-027 |
| BI / KPI Advisor wiring (per D-005 AI Mode) | CN-5-010 |
| Dashboard / notification UX (per-role surfaces, AI Mode) | Term 3 (CTR-004, CTR-015) |
| Channel adapters for alert delivery (SMS, email, WhatsApp, Telegram per D-008) | Term 7 (CTR-019/020) |
| Filing-format export adapter (PAYE file, VAT file matching jurisdiction submission specs) | Term 7 future (CTR-035 Open Item) |
| Filing submission to tax authorities, pension funds | Outside BOS (chartered accountant / HR officer per Charter §1.3) |
| Document verification public surface (QR, offline portal, signature) | Term 7 (CTR-017) |
| Statutory disclosures requiring board / auditor signature | Outside BOS (signed by responsible human) |

---

## 19. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Pack content for statutory_summary_reports, reporting_snapshots, export_templates, alternative_standard_views, KPI base library, alert delivery routing per jurisdiction | Term 1 via CTR-029 expansion | Schema across §§5–11; per-jurisdiction content is Term 1's authorship |
| Vertical-dimensional report event contracts (workshop project profitability, hotel occupancy, etc.) | Term 6 via CTR-030 expansion | N4 — v1 catalogue empty; added per vertical |
| Filing-format adapter (PAYE file, VAT file, pension file generation matching jurisdiction specs) | Term 7 future via CTR-035 | Open Item; defer per Charter §1.3 until clear tenant demand |
| AI Mode dashboard pattern adoption per role | Term 3 via CTR-015 + CN-5-010 | CN-5-006 provides projections; UX surface lives at Term 3 |
| BI Advisor wiring (engine + KPI advisors) | CN-5-010 | Subsequent Term 5 doc per D-005 |
| Authorised role policy for Statement issuance and sensitive exports | Term 1 (governance) + per-tenant policy | Pack provides defaults; tenant policy may extend |
| Statement amendment workflow detail (when error in closed-period Statement is discovered) | Architect phase + future decision | §5 mentions amendment-as-new-Document; full workflow detail deferred |
| Performance budget for fan-in subscription processing at scale | Architect phase | Concept guarantees correctness; performance is Architect's |
| Multi-currency reporting view (USD + TZS positions side-by-side) | Future + CN-5-001 FX | v1 single-currency reports; multi-currency deferred |
| Doctrine check for R1 read-only emission (CI verifies no Reporting code emits business events) | CN-4-019 + future CTR | Static analysis can detect; arrives with CN-5-006 ratification |
| Doctrine check for R6 — snapshot and Statement remain distinct types | CN-4-019 | Schema validation can enforce |
| Standard-view derivation rule completeness (which views work; pack mapping limits) | Architect + Term 1 | Some standards' nuances may not map cleanly via remapping alone |
| Long-term Statement archive (immutable storage for decade-plus document retention) | Architect phase | CN-4-012 Documents are in event store; physical archive policy is Architect/governance |

---

*— End of CN-5-006 —*
