# Term 5 — Universal Engines

> **Parent:** [BOS-CONCEPT-CHARTER.md](../BOS-CONCEPT-CHARTER.md) — read first
> **Type:** System Term (Part B)
> **Status:** Concept Phase v1.0

---

## 1. Mission

Universal Engines are the engines every business uses, regardless of what it sells. A retail shop, a restaurant, a hotel, a workshop, a future insurance broker, a future hospital — all of them need to keep their books, count their cash, track their stock, manage purchases, pay their people, and read reports about themselves.

This Term thinks about those shared engines. It defines what makes them "universal" — what they must do for *any* business — and how they remain stable while verticals come and go above them.

Universal Engines are the **middle layer**. Foundation gives them primitives. Vertical Engines build experiences on top of them. They are the workhorses.

---

## 2. Mission Question

> **"What does every business — a tomato seller, a fancy restaurant, a hospital, a marketing agency — actually share in common, beneath the surface of what they sell, and how do we build engines that serve all of them without contorting to any one of them?"**

---

## 3. Scope (In)

This Term owns concepts for these engines:

- **Accounting Engine** — management-truth double-entry bookkeeping; auto-journals from other engines' events
- **Cash Management Engine (CME)** — physical cash tracking, drawers, sessions/shifts, reconciliation, bank/wallet accounts
- **Inventory Engine** — branch-level stock movements, FIFO/LIFO, lot tracking, offcuts, reservations
- **Procurement Engine** — purchase lifecycle (requisition → PO → GRN → invoice match → payment reference)
- **HR & Payroll Engine** — employees, roles, payroll computation, payroll-to-accounting journal flow
- **Reporting & BI Engine** — KPI projections, snapshots, trial balance, P&L, balance sheet (management-grade)
- **Promotion Engine** — discounts, vouchers, loyalty programs (universal because every business may run promos)
- **AI Advisory Layer Application** — how AI advisors (defined by Foundation) are wired into these engines as advisors (Inventory Advisor, Cash Advisor, Procurement Advisor)

Per engine, this Term defines:
- The engine's purpose and non-purpose
- Event types (the verbs the engine emits)
- Command types (the verbs the engine accepts)
- Subscriptions (events from other engines this engine listens to)
- Scope policy (business-scope vs branch-scope rules)
- Key invariants (rules that must always hold)
- Integration with Foundation primitives
- Cross-engine event flows

---

## 4. Scope (Out)

This Term does NOT own:

| What | Owned By |
|------|----------|
| Vertical-specific engines: Retail, Restaurant, Hotel, Workshop, future verticals | Term 6 — Vertical Engines |
| The event store, command bus, primitives, security kernel | Term 4 — Foundation |
| The UI tenants see (cashier screen, owner dashboard) | Term 3 — Tenant Experience |
| Platform admin governance of these engines | Term 1 — Platform Stewards |
| Agent-side workflows | Term 2 — Regional Distribution |
| External integration (M-Pesa for cash transactions) | Term 7 — Integration & Coherence |
| Regional compliance specifics (TZ VAT rates) | Term 1 (rules); Term 4 (DSL); region pack data |
| Statutory accounting (filing taxes) | Out of scope — BOS is management-truth, not statutory |

**Important distinction:** Universal Engines compute the *what* (the truth of the business). UX (Term 3) presents it. Foundation (Term 4) stores it. Vertical Engines (Term 6) extend it for their domain.

---

## 5. Audiences Served

Universal Engines have no direct end-user audience. They serve:

| Audience | What they get from Universal Engines |
|----------|--------------------------------------|
| Term 3 (Tenant Experience) | The numbers tenants see come from these engines |
| Term 6 (Vertical Engines) | These engines consume vertical events (sales, restaurant orders) and produce universal truth |
| Term 1 (Platform Stewards) | Subscription billing relies on usage data emitted by these engines |
| Term 2 (Regional Distribution) | Agents see tenant health via engine outputs |
| Term 7 (Integration & Coherence) | Engines emit events; integrations subscribe |

End-users feel these engines only indirectly — but every "TZS 1.2M revenue this month" answer comes from here.

---

## 6. Key Concepts This Term Will Produce

Concept documents under this Term:

### 6.1 Per-Engine Concepts

| ID | Concept | What it answers |
|----|---------|-----------------|
| CN-5-001 | Accounting Engine | Double-entry, auto-journal, period close, corrections |
| CN-5-002 | Cash Management Engine | Drawers, sessions, reconciliation, branch-scope rules |
| CN-5-003 | Inventory Engine | Movements, FIFO/LIFO, lots, offcuts, reservations |
| CN-5-004 | Procurement Engine | Requisition → PO → GRN → Invoice → Payment reference |
| CN-5-005 | HR & Payroll Engine | Employees, roles, payroll, payroll-to-accounting |
| CN-5-006 | Reporting & BI Engine | KPI projections, snapshots, P&L, balance sheet |
| CN-5-007 | Promotion Engine | Discounts, vouchers, loyalty, cost-share |
| CN-5-008 | AI Advisors Wiring | How Inventory/Cash/Procurement Advisors plug into engines |

### 6.2 Cross-Engine Concepts

| ID | Concept | What it answers |
|----|---------|-----------------|
| CN-5-100 | Subscription Wiring Patterns | How engines listen to each other's events (no direct calls) |
| CN-5-101 | Scope Policy Application | When an engine action requires branch_id vs business_id only |
| CN-5-102 | Engine Invariants | Cross-engine rules (e.g., debit = credit; stock movements always have a source) |
| CN-5-103 | Universal Event Glossary | Catalog of every universal-engine event, with stable contracts |
| CN-5-104 | Period Close Choreography | The dance between Accounting, Cash, Inventory, HR when closing a month |
| CN-5-105 | Tax-Aware Engines (Without Hard-Coding Tax) | How engines respect compliance packs without knowing them |

---

## 7. Edge Cases to Consider

### 7.1 Accounting

- A correction is needed for an entry made 3 months ago. Charter says no overwrites. How? (Compensating entries with clear linkage.)
- A tenant doesn't have a chart of accounts at onboarding. Do we provide a default per vertical? Per country? Both?
- Period close locks a month. A late event for that month arrives (e.g., a delayed bank reconciliation). What happens?
- A multi-currency business (TZS books, but a USD invoice). FX rate at what moment — invoice date, payment date, close date?
- A branch-level P&L is requested. Accounting is business-scope. How do we tag "branch dimension" without making branch a mandatory accounting key?

### 7.2 Cash Management

- A cashier opens a session with TZS 50,000 float but actually has TZS 45,000 in the drawer. When the system catches this on close, what's the workflow?
- A robbery — TZS 200,000 missing. The owner records this. Is it an "adjustment," a "transfer," or a new event type?
- A cash transfer from Branch A to Branch B is in transit (someone is carrying cash). Branch A closed the day; Branch B hasn't received yet. How is this in-transit cash represented?
- An end customer pays partly in cash, partly in M-Pesa, partly on credit. How is the split recorded? (Per-payment-method events.)

### 7.3 Inventory

- A stock count reveals discrepancy — system says 50 units, physical count says 47. Adjustment requires approval. Whose? How fast?
- FIFO/LIFO lots: a lot expires (food). Does inventory automatically write off, or wait for an explicit event?
- Workshop offcut: a 200mm scrap from cutting a window. Is it inventory (re-usable) or waste? How does the tenant decide?
- A stock transfer from Branch A to Branch B is approved but Branch A's stock didn't actually leave (someone forgot to load the truck). What event represents "in transit" before "received"?
- A reservation for stock holds it for a pending order. The order is cancelled hours later. Auto-release, or manual?

### 7.4 Procurement

- A purchase order is approved, GRN comes in, but the supplier delivered the wrong items. Reject GRN? Partial GRN? Return?
- An invoice doesn't match the GRN (supplier billed more). Three-way match logic: who decides?
- A payment is released for an invoice that was matched, but the bank rejects (insufficient funds). What event chain?
- A blanket PO for the year, with releases against it. Is this one PO or many? (Likely many releases against one master — design it.)
- Petty cash purchase — no PO, no GRN, just a receipt. How does this enter Procurement (if at all)?

### 7.5 HR & Payroll

- An employee was overpaid last month. We correct this month. Is the correction visible to the employee, or hidden in a "true-up"?
- A new tax rate kicks in mid-month. Payroll for that month: split rate or apply new rate?
- An employee is paid partly by salary and partly by commission (e.g., a salesperson). How is commission integrated from Retail/Restaurant events?
- An employee is terminated. Their final pay includes outstanding salary + leave payout + minus advances. Choreography?
- Payroll is processed but not yet paid. Are these obligations on the balance sheet (accrued)?

### 7.6 Reporting & BI

- An owner wants to know "profit per branch." Accounting is business-scope. Reporting must derive branch-level views from analytical tags. How?
- A KPI projection broke (bug in the calculator). Rebuild while business is operating: what does the user see during rebuild?
- A "compare to last year" report wants November 2024 data. The compliance pack changed in May 2025. Do we apply 2024's rules or current rules to 2024's data? (Almost certainly 2024's. But say so.)
- A tenant asks "what if I raise prices 10%?" — this is the AI simulation, not a real report. Where does the line between report and simulation live?

### 7.7 Promotions

- A platform-wide promotion (BOS subsidising 50% of a Black Friday discount). Cost is shared between BOS, agent, tenant. How is this recorded — and reversed if cancelled?
- A loyalty card holder gets 5% off everywhere. The discount must be applied at the POS, but is the loyalty record business-scope or branch-scope? (Business.)
- A bogus voucher (someone is abusing it). How is fraud detection wired in?
- A promotion expires mid-cart. The cart was started 30 min ago when promo was valid. Honour it or invalidate?

### 7.8 Cross-Engine

- A retail sale event arrives. Accounting must journal it (revenue + tax + cost of goods). Inventory must reduce stock. Cash must record receipt. **In what order? Are they atomic? What if one fails?** (This is one of the hardest design questions in BOS.)
- A retail refund. Same engines, all in reverse. Are these new events or "undo" markers?
- A cash transfer between branches: Cash Engine handles it. Accounting must journal it (between branch ledgers). Both engines must agree on terminology.
- A procurement payment is released. Cash Engine records cash-out. Accounting journals AP-down/cash-down. Procurement updates the invoice as paid. Three engines, one truth — choreography?

---

## 8. Success Criteria

This Term is doing well when:

1. **Every universal engine can be enabled or disabled per tenant via subscription combo** without affecting other engines.
2. **A new vertical engine in 2027 (e.g., Insurance) can subscribe to existing universal events** without modifying any universal engine.
3. **A tenant's trial balance always balances (debit = credit)** for any historical date — proven by automated invariant tests.
4. **A cash session always reconciles** — or the discrepancy is recorded as an explicit, audit-visible event.
5. **Stock-on-hand reported by Inventory matches the count derived by replaying events** for any branch, any date.
6. **A procurement lifecycle is auditable from requisition to payment** with no missing links.
7. **Payroll matches the sum of payroll-to-accounting journals** for any period.
8. **A regional compliance pack changes a tax rate**, and the next sale reflects it — without any engine code change.
9. **No engine calls another engine directly.** Communication is events-only, and this is enforced by tests.

---

## 9. Architect's Role for This Term

After Concept Team writes documents, the Architect translates them into:

| Concept Output | Architect Output |
|----------------|------------------|
| Accounting (CN-5-001) | Engine module structure, event schemas, journal entry model, period state machine |
| Cash (CN-5-002) | Drawer/session/account models, branch-scope guards, reconciliation flow |
| Inventory (CN-5-003) | Movement event types, lot tracking schema, FIFO/LIFO strategy plugin design |
| Procurement (CN-5-004) | Lifecycle state machine, event/command pairs for each stage |
| HR (CN-5-005) | Employee model, payroll calculator design, accounting hook events |
| Reporting (CN-5-006) | Projection registry, KPI calculator interfaces, snapshot policy |
| Promotion (CN-5-007) | Promotion rule schema, applicability engine, cost-share record model |
| Subscription patterns (CN-5-100) | Subscriber registry usage, handler signatures, idempotency keys |
| Period close (CN-5-104) | Multi-engine close orchestrator design |
| Tax-aware engines (CN-5-105) | How compliance pack data is injected via Foundation, not engine config |

**Architect's directive to Developer for Term 5:**
- Each engine has its own folder under `engines/`
- Each engine emits only its own events; never another engine's
- Each engine subscribes to other engines' events through the bus, never by direct import
- Every command has a `policy_name` on rejection
- Every engine's `_execute_command()` starts with a scope guard
- All engine state is derivable from its events alone
- AI advisors live alongside engines but never modify state

---

## 10. Relationships with Other Terms

### Term 5 depends on:

| Term | Why |
|------|-----|
| Term 4 (Foundation) | Primitives, event store, command bus, scope policy, security, AI guardrails |
| Term 1 (Platform Stewards) | Engine catalog (which engines exist); compliance pack approval |

### Term 5 is depended on by:

| Term | Why |
|------|-----|
| Term 6 (Vertical Engines) | Vertical engines emit events that universal engines consume |
| Term 3 (Tenant Experience) | The dashboard shows numbers computed here |
| Term 1 (Platform Stewards) | Subscription billing reads usage from these engines |
| Term 7 (Integration & Coherence) | External integrations attach to engine events |

### Cross-Term Hand-Offs

- ← Term 4: "Here's the ledger primitive." Term 5 (Accounting) uses it.
- → Term 6: "Retail emits `retail.sale.completed.v1`. Universal engines subscribe." Inventory deducts stock; Cash records receipt; Accounting journals.
- → Term 3: "Cash on hand right now = Cash Engine projection X." Term 3 wires the dashboard.
- ← Term 1: "Subscription billing needs to know monthly transaction count per tenant. Wire it." Term 5 designs the metric event.

---

## 11. Open Questions

1. **Atomic cross-engine writes.** A retail sale triggers updates in Inventory, Cash, Accounting. If Inventory's projection fails, is the sale still valid? (Yes, events are emitted. But projections may lag.) How is this acceptable behaviour communicated to tenants?
2. **Period close authority.** Who initiates period close — the owner manually, or the system on a schedule? Per engine, or platform-wide?
3. **Default chart of accounts.** Ship one? Per vertical? Per country? All three?
4. **Inventory valuation method.** FIFO, LIFO, weighted average — chosen per tenant, per item, or per category? Default?
5. **Promotion cost-share.** A BOS-funded promotion: the tenant gets the discount, BOS absorbs part of the loss. Is this a real cash event (BOS pays back) or an invoice-time adjustment?
6. **Payroll tax provider.** Some countries (Tanzania PAYE) require monthly filings. Does BOS prepare the file, or just provide the data? Both? Out of scope?
7. **Multi-currency reality.** Accounting in TZS, supplier billing in USD. FX gain/loss recognition: per transaction, per period? Who decides?
8. **Promotional pricing precedence.** Item base price + branch override + active promo + loyalty discount + customer-specific deal. What's the precedence stack? Where is it computed (Retail or Promotion engine)?
9. **AI advisor scope.** Inventory Advisor sees this tenant's inventory only? Or anonymous benchmarks ("most workshops your size in TZ reorder this material every 6 weeks")? Privacy implications?
10. **Engine version migration.** Accounting v1 emits `accounting.journal.posted.v1`. Future Accounting v2 needs `accounting.journal.posted.v2`. How is the migration handled — replay-safe?

---

## 12. Notes on Existing Repository

The existing repository has reasonably mature implementations of all universal engines:
- `engines/accounting/` (53 tests passing)
- `engines/cash/`
- `engines/inventory/`
- `engines/procurement/` (41 tests passing)
- `engines/hr/`
- `engines/reporting/` (Phase 5)
- `engines/promotion/` (Phase 7)
- Cross-engine subscriptions wired in `engines/*/subscriptions.py`
- AI advisors at `ai/advisors/`

**Notable from existing code:**
- Each engine has request-suffixed command names (e.g., `retail.sale.complete.request`)
- Event types versioned (`*.v1`)
- `policy_name` enforced on rejections
- Scope guards enforced (GAP-10 closed)

**Concept Phase guidance:** Existing engines are good *reference* for what events and commands look like in practice. But the *purpose, doctrine, and boundaries* of each engine must be written fresh from the user's perspective — because that is often what's missing in code-first development.

**Specifically:** This Term must answer "What does Mama Asha actually need from Accounting?" before describing what Accounting does. The engine exists for Mama Asha, not the other way around.

---

## 13. First Tasks for This Term

When this Term starts concept work, begin in this order:

1. **CN-5-100 first** — subscription wiring patterns, because every engine doc relies on this.
2. **CN-5-101 second** — scope policy application, because every engine doc must respect it.
3. **CN-5-102 third** — engine invariants, the laws every engine must obey.
4. **CN-5-001 fourth** — Accounting, because every other engine emits events that Accounting consumes.
5. **CN-5-003 fifth** — Inventory, because retail/restaurant/workshop all consume from it.
6. **CN-5-002 sixth** — Cash, because every sale produces cash effects.
7. **CN-5-004 seventh** — Procurement.
8. **CN-5-005 eighth** — HR.
9. **CN-5-006 ninth** — Reporting (depends on all above being defined).
10. **CN-5-007 tenth** — Promotion (touches Retail, Accounting, Cash).
11. **CN-5-103, CN-5-104, CN-5-105, CN-5-008** — cross-cutting docs as the picture clarifies.

---

## 14. A Note on Universality

The word "Universal" is doing heavy lifting. It means **"applicable to any business type, present or future."**

Test for universality: if a concept doc mentions a vertical (Retail, Restaurant, Hotel, Workshop) as something the engine *knows about*, that is a leak. The engine should know about events it subscribes to, not verticals.

Example:
- ✅ Inventory Engine subscribes to `procurement.order.received.v1` and reduces stock.
- ❌ Inventory Engine has a special branch for "restaurant inventory" with different logic.

If a vertical needs special behaviour that universal engines can't provide, that behaviour belongs in the vertical engine (Term 6), not here.

---

## Standing Decisions & Cross-Term Work

> Read `DECISION-LOG.md`, `CROSS-TERM-COLLABORATION.md`, and `CROSS-TERM-REQUEST-REGISTER.md` before discussing these. File a CTR for every dependency.

| New Concept | From | This Term must decide |
|-------------|------|------------------------|
| CN-5-009 | D-001 | The **Universal Checkout / Tender Engine**: tender, splits, change, receipt issuance; Accounting + Cash subscribe to its events |
| CN-5-010 | D-002 | The **BI / KPI Advisor** for owners and **engine advisor wiring** (Inventory/Cash/Procurement), advisory-only, on the Advisor Framework |

**CTRs to expect:** files CTR-003 (checkout boundary → 4), CTR-006 (payment adapters → 7), CTR-007 (advisor framework → 4); receives CTR-002 (lines hand-off ← 6), CTR-004 (checkout UI ← 3), CTR-005 (catalog ← 1), CTR-009 (explainability ← 3); confirms CTR-010 (advisory-only).

---

*— End of Term 5 Brief —*
