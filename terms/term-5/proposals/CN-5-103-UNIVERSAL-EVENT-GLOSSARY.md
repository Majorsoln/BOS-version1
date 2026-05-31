# CN-5-103 — Universal Event Glossary

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 5 — Universal Engines
> **Status:** For Overseer review.
> **Governing decisions:** Charter §8.1 (event-name format `<engine>.<noun>.<verb>.v<n>`); D-009 (freeze doctrine — events interpreted under historical pack); Law 2 (engine isolation — events are the contract); UI-01 / UI-02 / UI-04 / UI-05 / UI-06 / UI-07 / UI-08 / UI-10 (cross-engine invariants from CN-5-102 — referenced per event in master catalogue).
> **CTRs:**
> - **OPEN (this doc references for resolution):** CTR-036 (Term 5 → Term 4 — `kernel.*` namespace reservation; resolution lands DC-042 in CN-4-019); CTR-037 (Term 5 → Term 4 — `pack.*` event namespace clarification — owner, producer, event list); CTR-038 (Term 5 → Term 6 — vertical namespace pre-allocation per Q6 (c) hybrid).
> - **No new CTRs.**
> **Glossary:** See `MASTER-GLOSSARY.md` — Universal-engine Invariant (UI), Site, Scope level, Event, Command.
> **Depends on:** Charter §8.1 (naming format); CN-4-002 (event envelope — citation only); CN-4-004 (command bus — citation only); CN-4-005 (engine manifest — subscriptions reference event types); CN-4-011 (Foundation primitives — citation only); CN-4-012 (Document Engine — citation only); CN-4-013 (Decision Journal — citation only); CN-4-018 (Snapshot Storage — citation only); CN-5-001 / CN-5-002 / CN-5-003 / CN-5-004 / CN-5-005 / CN-5-006 / CN-5-007 / CN-5-009 (all event sources catalogued here); CN-5-100 / CN-5-101 / CN-5-102 (doctrine references).
> **Boundaries:** Foundation events → cited only, owned by Term 4; vertical event names → deferred to Term 6 per CTR-038; pack-emitted events → pending CTR-037; adapter-internal event semantics → Term 7; new event addition post-ratification → CTR or amendment per G7.

---

## 1. Purpose & Boundary

CN-5-103 is the **Universal Event Glossary** — a cross-cutting registry (not an engine) that:

1. **Catalogs** every command and event declared across the 11 ratified Term 5 documents (~190 distinct names).
2. **Establishes naming doctrine** (G1–G9) and **resolves naming inconsistencies** (E1–E10) discovered during the audit.
3. **Governs future event addition** via CTR/amendment process (G7) — no silent additions.

This document has **no engine manifest, no own commands, no own events**. It is a registry + governance layer. The events it catalogs are emitted by the engines that own them; CN-5-103 records the contract.

### What This Doc Does NOT Define

- Foundation events (CN-4-002/004/013/018 etc.) — cited only; ownership Term 4.
- Vertical event names (`restaurant.*`, `hotel.*`, `workshop.*`, `retail.*`, `pharmacy.*`, `clinic.*`) — namespace ownership pre-allocated here per CTR-038; specific event names deferred to Term 6.
- Pack-emitted events (`pack.*` or `kernel.pack.*`) — pending CTR-037 Term 4 resolution.
- Adapter-internal event semantics — Term 7 owns; this doc records the angle-bracket placeholder convention.
- New event semantics — events added post-ratification follow G7.

---

## 2. Doctrine (G1–G9)

| # | Law | Source / why |
|---|-----|--------------|
| **G1** | **Event-name format = `<engine>.<noun>.<verb>.v<n>`** per Charter §8.1. `<engine>` ∈ registered engine list (accounting, cash, inventory, procurement, hr, reporting, promotion, checkout — plus Term 6 verticals when registered per CTR-038). | Charter §8.1; uniformity enables bus routing, doctrine checks (CN-4-019), replay determinism (CN-4-001). |
| **G2** | **`kernel.*` namespace is RESERVED for Kernel meta-events only** (Term 4 — CN-4-002/004/008/013/018, etc.). Universal engines NEVER emit `kernel.*` events. Per CTR-036, resolution lands DC-042 in CN-4-019 (CI-enforced). | Term 4 owns kernel events; engine isolation (Law 2); Term 5 engines emit only under their own namespace. |
| **G3** | **Adapter events use `<adapter-id>.<noun>.<verb>.v<n>`** — `<adapter-id>` is the neutral placeholder used in engine docs (e.g., `<mm-adapter>`, `<bank-adapter>`, `<fx-adapter>`, `<channel-adapter>`, `<card-adapter>`). Concrete adapter-id resolves at deployment time per Term 7 adapter registry. | Term 7 owns adapters; engine docs reference placeholder; deployment maps to concrete adapter (e.g., `<mm-adapter>` → `mpesa-adapter`, `tigopesa-adapter`). |
| **G4** | **`system.*` namespace IS NOT ALLOWED.** Scheduler-triggered work uses engine-owned commands with `system:scheduler` actor (CN-4-007). Engines emit their own scheduled-work events (e.g., `accounting.depreciation.posted.v1`, `inventory.lot.expired.v1`, `promotion.voucher.expired.v1`). | Parallel to CN-5-001 N1 (depreciation), CN-5-005 N2 (accrual), CN-5-007 (voucher expiry). Avoids parallel namespace for scheduler. |
| **G5** | **Compensating events use verb-pair convention.** Established pairs include: `applied`/`reversed`; `issued`/`burned` (or `cancelled`/`expired`); `received`/`refunded`; `posted`/`reversed`; `accrued`/`expired` (with compensation propagating per UI-02). | UI-02 (CN-5-102); audit-discoverability of compensations; replay analysis. |
| **G6** | **Event versioning: `.v1` is baseline; `.v2` introduced only for breaking schema change.** Additive changes within `.v1` are permitted (CN-4-002 §3). Producer may emit both `.v1` and `.v2` during a compatibility window (window length pack-driven). Deprecated versions marked per N4 lifecycle (§4). | CN-4-002 §3 additive-only doctrine; freeze (D-009) — historical events stay under their original schema forever. |
| **G7** | **Event-name addition post-ratification requires CTR or amendment.** Engine owner Term may not silently introduce new events between glossary updates. Process: file CTR or amend engine doc → Overseer ratifies → glossary updated → producer emits. | Charter §1.2 (auditability); CN-4-005 manifest declarative discipline; prevents "snuck in" events that consumers cannot discover. |
| **G8** | **Cross-engine references use producer's authoritative name (no aliasing).** When engine A subscribes to engine B's event, A's manifest and docs reference B's exact event name as registered here. No renaming, no abbreviation. | Engine isolation (Law 2); CN-4-005 §1 (subscriptions reference event types, not engines by name); single source of truth. |
| **G9** | **Verb tense = past or past-participle.** Events describe what happened (`paid`, `received`, `posted`, `redeemed`, `expired`). Commands use imperative `.request` form (`pay`, `receive`, `post`, `redeem`, `expire`). Present-progressive (`paying`, `receiving`) NOT allowed. | Consistency; semantic clarity (events are facts; commands are intents); doctrine-check enforceable. |

---

## 3. Naming Conventions

### Command vs Event Form

| Aspect | Command (intent) | Event (fact) |
|--------|------------------|--------------|
| Form | `<engine>.<noun>.<verb>.request` | `<engine>.<noun>.<verb_past>.v<n>` |
| Tense | Imperative (e.g., `record`, `submit`, `issue`) | Past or past-participle (e.g., `recorded`, `submitted`, `issued`) |
| Suffix | `.request` (commands are not versioned) | `.v<n>` (events versioned per CN-4-002 §3) |
| Example | `cash.expense.record.request` | `cash.expense.recorded.v1` |

### Singular Nouns

All nouns are singular. `inventory.movement.recorded.v1` (singular `movement`), not `inventory.movements.recorded.v1`. Multi-line semantics carried in payload, not name.

### Compound Nouns

Multi-word nouns use underscore: `cash.advance_received.recorded.v1`, `promotion.cost_share.recorded.v1`, `hr.payroll.deduction.applied.v1`, `procurement.credit_note.recorded.v1`.

### Sub-Namespace (4-Segment Names)

When an engine has logically-grouped sub-domains, 4-segment names are allowed: `promotion.loyalty.points.accrued.v1`, `promotion.outreach.requested.v1`, `procurement.invoice.dispute_raised.v1`. Engine = first segment; sub-domain = second; noun = third; verb = fourth.

### Adapter Names

Per G3: `<adapter-id>.<noun>.<verb>.v<n>`. Concrete examples in deployment:
- `<mm-adapter>` → `mpesa-adapter`, `tigopesa-adapter`, `airtelmoney-adapter`
- `<card-adapter>` → `visa-adapter`, `mastercard-adapter`, `stripe-adapter`
- `<bank-adapter>` → `crdb-adapter`, `nmb-adapter`
- `<fx-adapter>` → `boi-fx-adapter`, `reuters-fx-adapter`
- `<channel-adapter>` → `sms-adapter`, `whatsapp-adapter`, `telegram-adapter`, `email-adapter`

---

## 4. Catalogue State Classification (N4 — 4 States)

Each event in the master catalogue (§§5–14) carries a **state** indicating its lifecycle position:

| State | Meaning | Permissible transitions |
|-------|---------|--------------------------|
| **emitted** | Currently in engine manifest; producer emits in production | → deprecated (when superseded) |
| **pending** | Engine doc declares emission but downstream wiring not yet complete (e.g., a subscriber engine not yet ratified) | → emitted (when wiring completes) |
| **future** | Placeholder reserved for documents not yet written (CN-5-104 / CN-5-105 / Term 6 / Term 7) — name reserved but no producer yet | → emitted (when producer-doc ratifies) |
| **deprecated** | Replaced by a newer version (`.v2`+) or retired; producer no longer emits new occurrences; existing events remain in store per CN-4-001 immutability | terminal (no further transitions) |

### State Transition Governance (G7)

| Transition | Authority |
|------------|-----------|
| pending → emitted | Engine owner Term updates manifest; Overseer ratifies via glossary update |
| future → emitted | Producer doc ratification triggers glossary update |
| emitted → deprecated | Engine owner Term + Overseer agreement; CTR or amendment; compatibility window pack-driven (G6) |

State changes are themselves audit events recorded in CHANGELOG (no separate Foundation event needed — glossary versioning carries them).

---

## 5. Master Catalogue — Foundation Reference (Citation Only)

Foundation owns these events; CN-5-103 references for context but does NOT re-catalogue. Per Q7 ruling.

| Event | Owner doc | Purpose | UI invariants |
|-------|-----------|---------|---------------|
| `kernel.command.rejected.v1` | CN-4-004 §4 | Bus-level rejection record | UI-01 (causation back to original command) |
| `kernel.event.*` (envelope ops) | CN-4-002 | Event store envelope ops | — |
| `kernel.advisor.suggestion.recorded.v1` | CN-4-013 | Advisor Decision Journal | (Foundation) |
| `kernel.advisor.decision.recorded.v1` | CN-4-013 | Advisor Decision Journal | (Foundation) |
| `kernel.audit.entry.recorded.v1` | CN-4-008 | Audit log | (Foundation) |
| `kernel.snapshot.*` | CN-4-018 | Snapshot ops (non-truth marker) | DC-025/026 |
| `kernel.replay.*` | CN-4-009 | Replay engine ops | DC-004/017/018 |

**Pack events** (`pack.*` or `kernel.pack.*`) — **pending CTR-037 resolution**; CN-5-103 §17 updates upon Term 4 ruling.

---

## 6. Master Catalogue — Cross-Cutting (CN-5-100 / 101 / 102)

These docs are doctrine references; they do NOT emit events. References to events appear as examples drawn from per-engine docs (catalogued in §§7–14). No additional entries here.

---

## 7. Master Catalogue — Universal Checkout (CN-5-009)

Format: `name | producer | consumers | doc-ref | payload-ref | state | UI inv | notes`.

### Commands

| Command | Producer | Doc ref | State | Notes |
|---------|----------|---------|-------|-------|
| `checkout.ingest_bill.request` | system:checkout-orchestrator | CN-5-009 §3, §4 | emitted | System-submitted on vertical bill |
| `checkout.tender.add.request` | human:cashier | CN-5-009 §3, §4 | emitted | Per tender |
| `checkout.tender.confirm.request` | system:checkout-orchestrator | CN-5-009 §3 | emitted | System on async outcome |
| `checkout.tender.fail.request` | system:checkout-orchestrator | CN-5-009 §3 | emitted | System on async failure |
| `checkout.settle.request` | human:cashier | CN-5-009 §3, §4 | emitted | Explicit settle trigger (K5) |
| `checkout.cancel.request` | human:cashier | CN-5-009 §3 | emitted | Pre-settle cancel |
| `checkout.refund.request` | human:cashier (refund-authorised) | CN-5-009 §3, §7 | emitted | Compensation initiator |

### Events

| Event | Producer | Consumers | Doc ref | State | UI inv | Notes |
|-------|----------|-----------|---------|-------|--------|-------|
| `checkout.started.v1` | Checkout | (none required) | CN-5-009 §3, §5 | emitted | UI-01 | Workflow open |
| `checkout.tender.requested.v1` | Checkout | Term 7 adapters per method | CN-5-009 §3, §5 | emitted | UI-01 | Async tender to adapter |
| `checkout.tender.confirmed.v1` | Checkout | (Cash via settled) | CN-5-009 §3, §5 | emitted | UI-01, UI-06 | Per tender confirmation |
| `checkout.tender.failed.v1` | Checkout | (Cash via settled) | CN-5-009 §3, §5 | emitted | UI-01 | Per tender failure |
| **`checkout.settled.v1`** | Checkout | **Accounting (#1), Cash, Inventory, Reporting, Promotion** | CN-5-009 §3, §5 | emitted | UI-01, UI-04, UI-06 | **THE HINGE** |
| `checkout.cancelled.v1` | Checkout | (audit) | CN-5-009 §3, §5 | emitted | UI-01 | Pre-settle cancel |
| `checkout.refunded.v1` | Checkout | **Accounting (#2 compensation), Cash, Inventory, Promotion** | CN-5-009 §3, §5, §7 | emitted | UI-01, UI-02 | Compensates_event_id to original settled; partial supported |
| `checkout.receipt.issued.v1` | Checkout (via Document Engine) | Reporting | CN-5-009 §3, §5 | emitted | UI-01 | Document terminal-fold; atomic with settled |

**Total: 7 commands, 8 events. All emitted.**

---

## 8. Master Catalogue — Accounting (CN-5-001)

### Commands

| Command | Producer | Doc ref | State | Notes |
|---------|----------|---------|-------|-------|
| `accounting.chart_of_accounts.configure.request` | human:onboarding-accountant | CN-5-001 §3 | emitted | One-time per tenant onboarding |
| `accounting.account.add.request` | human:authorised-accountant | CN-5-001 §3 | emitted | Within pack-defined limits |
| `accounting.account.deactivate.request` | human:authorised-accountant | CN-5-001 §3 | emitted | Tenant sub-accounts only |
| `accounting.opening_balances.set.request` | human:onboarding-accountant + dual-actor | CN-5-001 §3, §10 | emitted | One-time per tenant (N3) |
| `accounting.adjustment.post.request` | human:authorised-accountant (CFO role) | CN-5-001 §3 | emitted | Current period only (UI-05) |
| `accounting.depreciation.run.request` | system:scheduler | CN-5-001 §3 | emitted | Periodic (G4 — engine-owned scheduler command) |
| `accounting.period.close.request` | human:authorised-accountant | CN-5-001 §3, §8 | emitted | CN-5-104 choreography initiator |

### Events

| Event | Producer | Consumers | Doc ref | State | UI inv | Notes |
|-------|----------|-----------|---------|-------|--------|-------|
| `accounting.chart_of_accounts.configured.v1` | Accounting | Reporting | CN-5-001 §3, §5 | emitted | UI-01 | Onboarding event |
| `accounting.account.added.v1` | Accounting | Reporting | CN-5-001 §3 | emitted | UI-01 | Chart customisation |
| `accounting.account.deactivated.v1` | Accounting | Reporting | CN-5-001 §3 | emitted | UI-01 | Chart customisation |
| `accounting.opening_balances.set.v1` | Accounting | Reporting | CN-5-001 §3, §10 | emitted | UI-01, UI-07 | Dual-actor audited |
| `accounting.journal.posted.v1` | Accounting | Reporting (financial projections) | CN-5-001 §3, §4 | emitted | UI-01, UI-05, UI-06, UI-07 | Auto-journal output |
| `accounting.journal.reversed.v1` | Accounting | Reporting (compensation) | CN-5-001 §3, §9 | emitted | UI-01, UI-02, UI-05 | Carries `compensates_event_id` + `posting_period_ref` (E5 / N1 — explicit forward-period anchor) |
| `accounting.depreciation.posted.v1` | Accounting | Reporting | CN-5-001 §3 | emitted | UI-01, UI-05 | Scheduler-driven |
| `accounting.period.opened.v1` | Accounting | Reporting, all engines (UI-05 policy) | CN-5-001 §3, §8 | emitted | UI-01, UI-05 | Period state |
| `accounting.period.closed.v1` | Accounting | Reporting, all engines (UI-05 enforcement) | CN-5-001 §3, §8 | emitted | UI-01, UI-05, UI-07 | UI-05 bus policy trigger; CN-5-104 choreography |

**Total: 7 commands, 9 events. All emitted.**

---

## 9. Master Catalogue — Cash Management (CN-5-002)

### Commands

| Command | Producer | Doc ref | State | Notes |
|---------|----------|---------|-------|-------|
| `cash.till.register.request` | human:authorised | CN-5-002 §4 | emitted | Till lifecycle |
| `cash.till.attributes_update.request` | human:authorised | CN-5-002 §4 | emitted | |
| `cash.till.deactivate.request` | human:authorised | CN-5-002 §4 | emitted | |
| `cash.session.open.request` | human:cashier | CN-5-002 §4, §5 | emitted | Workflow primitive |
| `cash.session.reconcile.request` | human:cashier | CN-5-002 §4, §5, §12 | emitted | Variance handled per C7 |
| `cash.session.close.request` | human:cashier | CN-5-002 §4, §5 | emitted | Close after reconcile |
| `cash.expense.record.request` | human:cashier | CN-5-002 §4, §7 | emitted | Generic operating expense |
| `cash.collection.record.request` | human:cashier | CN-5-002 §4, §G | emitted | AR settlement |
| `cash.payment.record.request` | human:cashier | CN-5-002 §4, §G | emitted | AP settlement |
| `cash.transfer.between_tills.request` | human:cashier | CN-5-002 §4, §10 | emitted | Within site (drop) |
| `cash.transfer.between_sites.request` | human:authorised | CN-5-002 §4, §10 | emitted | Cross-site (tenant scope) |
| `cash.deposit.initiate.request` | human:authorised | CN-5-002 §4, §11 | emitted | Bank deposit (CTR-031) |
| `cash.deposit.complete.request` | system:cash-orchestrator | CN-5-002 §4, §11 | emitted | On adapter outcome |
| `cash.equity.injection.record.request` | human:owner + dual-actor | CN-5-002 §4, §14 | emitted | Owner equity |
| `cash.equity.withdrawal.record.request` | human:owner + dual-actor | CN-5-002 §4, §14 | emitted | Owner equity |
| `cash.tip.record.request` | human:cashier | CN-5-002 §4, §16 | emitted | Cash tip passthrough |
| `cash.adjustment.record.request` | system:cash-orchestrator | CN-5-002 §4 | emitted | Variance auto-adjustment within threshold |
| `cash.advance_received.record.request` | human:cashier | CN-5-002 §4, §17 | emitted | Customer prepayment / layby instalment |

### Events

| Event | Producer | Consumers | Doc ref | State | UI inv | Notes |
|-------|----------|-----------|---------|-------|--------|-------|
| `cash.till.registered.v1` | Cash | Reporting | CN-5-002 §4 | emitted | UI-01 | |
| `cash.till.attributes_updated.v1` | Cash | Reporting | CN-5-002 §4 | emitted | UI-01 | |
| `cash.till.deactivated.v1` | Cash | Reporting | CN-5-002 §4 | emitted | UI-01 | |
| `cash.session.opened.v1` | Cash | Reporting | CN-5-002 §4, §5 | emitted | UI-01 | |
| `cash.session.closed.v1` | Cash | Reporting | CN-5-002 §4, §5 | emitted | UI-01 | |
| `cash.session.reconciled.v1` | Cash | Reporting | CN-5-002 §4, §5, §12 | emitted | UI-01 | |
| `cash.variance.detected.v1` | Cash | Reporting + Anomaly (CN-4-016) | CN-5-002 §4, §12 | emitted | UI-01 | Beyond auto-adjust threshold |
| `cash.variance.adjusted.v1` | Cash | Reporting | CN-5-002 §4, §12 | emitted | UI-01 | Within auto-adjust threshold |
| `cash.tender.received.v1` | Cash | Accounting, Reporting | CN-5-002 §4, §6 | emitted | UI-01, UI-04, UI-06 | Primary receipt; UI-04 origin chain |
| `cash.tender.disbursed.v1` | Cash | Accounting, Reporting, HR (filtered by origin) | CN-5-002 §4 | emitted | UI-01, UI-06 | Primary outflow |
| `cash.tender.requested.v1` | Cash | Term 7 adapters (async tenders) | CN-5-002 §4 | emitted | UI-01 | |
| `cash.expense.recorded.v1` | Cash | Accounting (#4), Reporting | CN-5-002 §4, §7 | emitted | UI-01, UI-06 | Umbrella for Accounting |
| `cash.collection.received.v1` | Cash | Accounting, Reporting, Promotion (cost-share), Procurement (filtered) | CN-5-002 §4, §G | emitted | UI-01, UI-04, UI-06, UI-08 | AR settlement; Obligation reduction |
| `cash.payment.disbursed.v1` | Cash | Accounting, Procurement (filtered), HR (filtered), Promotion | CN-5-002 §4, §G | emitted | UI-01, UI-04, UI-06, UI-08 | AP settlement; Obligation reduction |
| `cash.refund.issued.v1` | Cash | Accounting, Reporting | CN-5-002 §4, §13 | emitted | UI-01, UI-02 | Refund cash side (compensation) |
| `cash.transfer.initiated.v1` | Cash | Reporting | CN-5-002 §4, §10 | emitted | UI-01 | Within-site or cross-site |
| `cash.transfer.received.v1` | Cash | Reporting | CN-5-002 §4, §10 | emitted | UI-01 | |
| `cash.transfer.between_sites.v1` | Cash | Accounting (#5), Reporting | CN-5-002 §4, §10 | emitted | UI-01 | Umbrella event |
| `cash.deposit.requested.v1` | Cash | Term 7 banking adapter (CTR-031) | CN-5-002 §4, §11 | emitted | UI-01 | |
| `cash.deposit.completed.v1` | Cash | Accounting, Reporting | CN-5-002 §4, §11 | emitted | UI-01 | |
| `cash.deposit.failed.v1` | Cash | Reporting, Anomaly | CN-5-002 §4, §11 | emitted | UI-01 | |
| `cash.equity.injected.v1` | Cash | Accounting (#3), Reporting | CN-5-002 §4, §14 | emitted | UI-01 | Owner equity |
| `cash.equity.withdrawn.v1` | Cash | Accounting, Reporting | CN-5-002 §4, §14 | emitted | UI-01 | Owner drawings |
| `cash.tip.recorded.v1` | Cash | Reporting, HR (per pack tax routing) | CN-5-002 §4, §16 | emitted | UI-01 | Cash tip passthrough |
| `cash.tip.distributed.v1` | Cash | HR, Reporting | CN-5-002 §4, §16 | emitted | UI-01 | Pass-through to staff |
| `cash.advance_received.recorded.v1` | Cash | Accounting, Promotion, Reporting | CN-5-002 §4, §17 | emitted | UI-01, UI-08 | Creates Obligation (advance_received or layby) |
| `cash.adjustment.recorded.v1` | Cash | Accounting (#9 umbrella), Reporting | CN-5-002 §4 | emitted | UI-01 | Umbrella for Accounting |
| `cash.period.ready.v1` | Cash | Accounting (CN-5-104 fan-in) | CN-5-002 §4 | future | UI-01 | **CN-5-104 dependency** |

**Total: 18 commands, 28 events (1 future).**

---

## 10. Master Catalogue — Inventory (CN-5-003)

### Commands

| Command | Producer | Doc ref | State | Notes |
|---------|----------|---------|-------|-------|
| `inventory.item.register.request` | human:authorised | CN-5-003 §3 | emitted | Tenant-scoped |
| `inventory.item.update_attributes.request` | human:authorised | CN-5-003 §3 | emitted | |
| `inventory.item.deactivate.request` | human:authorised | CN-5-003 §3 | emitted | |
| `inventory.recipe.set.request` | human:authorised | CN-5-003 §3, §5 | emitted | Pattern A (versioned per Q9) |
| `inventory.recipe.revise.request` | human:authorised | CN-5-003 §3 | emitted | Recipe version bump |
| `inventory.recipe.deactivate.request` | human:authorised | CN-5-003 §3 | emitted | |
| `inventory.movement.record.request` | human:cashier OR system | CN-5-003 §3 | emitted | Generic entry (with type) |
| `inventory.stock.count.request` | human:authorised | CN-5-003 §3, §6 | emitted | Cycle counts per N5 (scope optional) |
| `inventory.reservation.create.request` | human:cashier | CN-5-003 §3, §9 | emitted | Hard hold |
| `inventory.reservation.confirm.request` | human:cashier OR system | CN-5-003 §3, §9 | emitted | Consume reserved |
| `inventory.reservation.release.request` | human:cashier | CN-5-003 §3, §9 | emitted | Return to available |
| `inventory.transfer.initiate.request` | human:authorised | CN-5-003 §3, §12 | emitted | Cross-site (tenant) |
| `inventory.transfer.receive.request` | human:cashier | CN-5-003 §3, §12 | emitted | Receiving side |
| `inventory.lot.expiry_run.request` | system:scheduler | CN-5-003 §3, §7 | emitted | G4 — engine-owned scheduler |

### Events

| Event | Producer | Consumers | Doc ref | State | UI inv | Notes |
|-------|----------|-----------|---------|-------|--------|-------|
| `inventory.item.registered.v1` | Inventory | Reporting | CN-5-003 §3 | emitted | UI-01 | |
| `inventory.item.attributes_updated.v1` | Inventory | Reporting | CN-5-003 §3 | emitted | UI-01 | |
| `inventory.item.deactivated.v1` | Inventory | Reporting | CN-5-003 §3 | emitted | UI-01 | |
| `inventory.recipe.set.v1` | Inventory | (internal projection) | CN-5-003 §3, §5 | emitted | UI-01 | Versioned per Q9 |
| `inventory.recipe.deactivated.v1` | Inventory | (internal projection) | CN-5-003 §3 | emitted | UI-01 | |
| `inventory.stock.received.v1` | Inventory | Accounting (#8), Reporting, Procurement (per workflow) | CN-5-003 §3, §6 | emitted | UI-01, UI-03, UI-09 | source_ref: procurement.grn or transfer.received |
| `inventory.stock.deducted.v1` | Inventory | Accounting (#1 COGS), Reporting | CN-5-003 §3, §6 | emitted | UI-01, UI-03, UI-09 | source_ref: checkout.settled or vertical.consumed |
| `inventory.stock.adjusted.v1` | Inventory | Accounting (via umbrella), Reporting | CN-5-003 §3, §6 | emitted | UI-01, UI-03, UI-09 | source_ref: count or adjustment |
| `inventory.stock.written_off.v1` | Inventory | Accounting (via umbrella), Reporting | CN-5-003 §3, §6, §7 | emitted | UI-01, UI-03, UI-09 | source_ref: expiry / damage |
| `inventory.adjustment.recorded.v1` | Inventory | Accounting (#9 umbrella), Reporting | CN-5-003 §3, §6 | emitted | UI-01 | Umbrella for Accounting |
| `inventory.lot.created.v1` | Inventory | Reporting | CN-5-003 §3, §7 | emitted | UI-01, UI-09 | On lot-tracked receipt |
| `inventory.lot.consumed.v1` | Inventory | Reporting | CN-5-003 §3 | emitted | UI-01 | Last unit deducted |
| `inventory.lot.expired.v1` | Inventory | Reporting, Accounting (via umbrella) | CN-5-003 §3, §7 | emitted | UI-01, UI-09 | Scheduler-triggered |
| `inventory.offcut.created.v1` | Inventory | Reporting | CN-5-003 §3, §10 | emitted | UI-01, UI-09 | Workshop cut produces sub-item |
| `inventory.reservation.created.v1` | Inventory | Reporting | CN-5-003 §3, §9 | emitted | UI-01, UI-09 | |
| `inventory.reservation.confirmed.v1` | Inventory | Reporting | CN-5-003 §3, §9 | emitted | UI-01, UI-09 | |
| `inventory.reservation.released.v1` | Inventory | Reporting | CN-5-003 §3, §9 | emitted | UI-01, UI-09 | |
| `inventory.transfer.initiated.v1` | Inventory | Reporting | CN-5-003 §3, §12 | emitted | UI-01, UI-09 | |
| `inventory.transfer.received.v1` | Inventory | Reporting | CN-5-003 §3, §12 | emitted | UI-01, UI-09 | |
| `inventory.promise.created.v1` | Inventory | Cash, Reporting | CN-5-003 §3, §13 | emitted | UI-01, UI-08 | Backorder Obligation (kind: delivery) |
| `inventory.promise.fulfilled.v1` | Inventory | Cash, Reporting | CN-5-003 §3, §13 | emitted | UI-01, UI-08 | |
| `inventory.promise.cancelled.v1` | Inventory | Reporting | CN-5-003 §3, §13 | emitted | UI-01, UI-08 | |
| `inventory.period.ready.v1` | Inventory | Accounting (CN-5-104 fan-in) | CN-5-003 §3 | future | UI-01 | **CN-5-104 dependency** |
| `inventory.reorder_point.reached.v1` | Inventory | Procurement (future v2) | CN-5-003 manifest | future | UI-01 | **v2 future** |

**Total: 14 commands, 24 events (2 future).**

---

## 11. Master Catalogue — Procurement (CN-5-004)

### Commands

| Command | Producer | Doc ref | State | Notes |
|---------|----------|---------|-------|-------|
| `procurement.supplier.register.request` | human:authorised | CN-5-004 §3 | emitted | Tenant-scoped |
| `procurement.supplier.attributes_update.request` | human:authorised | CN-5-004 §3 | emitted | |
| `procurement.supplier.deactivate.request` | human:authorised | CN-5-004 §3 | emitted | |
| `procurement.supplier.credit_terms_set.request` | human:authorised | CN-5-004 §3, §4 | emitted | |
| `procurement.rfq.issue.request` | human:authorised | CN-5-004 §3, §5 | emitted | Optional Phase 0 |
| `procurement.quote.record.request` | human:cashier | CN-5-004 §3, §5 | emitted | Optional Phase 0 |
| `procurement.requisition.create.request` | human:requester | CN-5-004 §3, §5 | emitted | Phase 1 |
| `procurement.requisition.approve.request` | human:approver (per pack threshold) | CN-5-004 §3, §9 | emitted | Phase 1 gate |
| `procurement.requisition.reject.request` | human:approver | CN-5-004 §3 | emitted | |
| `procurement.po.create.request` | system OR human:authorised | CN-5-004 §3, §5 | emitted | Phase 2 |
| `procurement.po.amend.request` | human:authorised | CN-5-004 §3 | emitted | Re-approval if value increases band |
| `procurement.po.cancel.request` | human:authorised | CN-5-004 §3 | emitted | |
| `procurement.grn.record.request` | human:cashier | CN-5-004 §3, §5 | emitted | Phase 3; may be partial |
| `procurement.invoice.record.request` | human:cashier | CN-5-004 §3, §5, §8 | emitted | Phase 4; three-way match |
| `procurement.invoice.dispute.request` | human:authorised | CN-5-004 §3, §7 | emitted | |
| `procurement.invoice.resolve_dispute.request` | human:authorised | CN-5-004 §3, §7 | emitted | |
| `procurement.payment.schedule.request` | human:authorised | CN-5-004 §3, §5 | emitted | Phase 5 |
| `procurement.payment.authorize.request` | human:authorised | CN-5-004 §3, §5 | emitted | Phase 5 |
| `procurement.return.initiate.request` | human:authorised | CN-5-004 §3, §12 | emitted | Compensation initiator |
| `procurement.credit_note.record.request` | human:cashier | CN-5-004 §3, §12 | emitted | Compensation document |

### Events

| Event | Producer | Consumers | Doc ref | State | UI inv | Notes |
|-------|----------|-----------|---------|-------|--------|-------|
| `procurement.supplier.registered.v1` | Procurement | Reporting | CN-5-004 §3 | emitted | UI-01 | |
| `procurement.supplier.attributes_updated.v1` | Procurement | Reporting | CN-5-004 §3 | emitted | UI-01 | |
| `procurement.supplier.deactivated.v1` | Procurement | Reporting | CN-5-004 §3 | emitted | UI-01 | |
| `procurement.supplier.credit_terms_set.v1` | Procurement | Reporting | CN-5-004 §3 | emitted | UI-01 | |
| `procurement.rfq.issued.v1` | Procurement (Document) | Reporting | CN-5-004 §3, §6 | emitted | UI-01 | BOS-issued Document |
| `procurement.quote.recorded.v1` | Procurement | Reporting | CN-5-004 §3, §6 | emitted | UI-01 | Recorded (supplier-owned) |
| `procurement.requisition.created.v1` | Procurement | Reporting | CN-5-004 §3 | emitted | UI-01 | |
| `procurement.requisition.approved.v1` | Procurement | Reporting | CN-5-004 §3 | emitted | UI-01 | |
| `procurement.requisition.rejected.v1` | Procurement | Reporting | CN-5-004 §3 | emitted | UI-01 | |
| `procurement.po.created.v1` | Procurement (Document) | Reporting, Supplier (Term 7 external?) | CN-5-004 §3, §6 | emitted | UI-01 | BOS-issued Document |
| `procurement.po.amended.v1` | Procurement | Reporting | CN-5-004 §3 | emitted | UI-01 | |
| `procurement.po.cancelled.v1` | Procurement | Reporting | CN-5-004 §3 | emitted | UI-01 | |
| `procurement.grn.received.v1` | Procurement (Document) | Inventory, Accounting (#8), Reporting | CN-5-004 §3, §6 | emitted | UI-01, UI-09 | BOS-issued Document |
| `procurement.grn.discrepancy.detected.v1` | Procurement | Reporting, Anomaly | CN-5-004 §3, §12 | emitted | UI-01 | |
| `procurement.grn.compensated.v1` | Procurement | Inventory, Reporting | CN-5-004 §3, §12 | emitted | UI-01, UI-02 | For returns |
| `procurement.invoice.recorded.v1` | Procurement | Accounting (#6), Reporting | CN-5-004 §3, §6, §8 | emitted | UI-01, UI-06, UI-08 | Creates AP Obligation |
| `procurement.invoice.matched.v1` | Procurement | Reporting | CN-5-004 §3, §7 | emitted | UI-01 | Three-way match passed |
| `procurement.invoice.dispute_raised.v1` | Procurement | Reporting, Anomaly | CN-5-004 §3, §7 | emitted | UI-01 | |
| `procurement.invoice.dispute_resolved.v1` | Procurement | Reporting | CN-5-004 §3, §7 | emitted | UI-01 | |
| `procurement.payment.scheduled.v1` | Procurement | Reporting | CN-5-004 §3 | emitted | UI-01 | |
| `procurement.payment.authorized.v1` | Procurement (Document Voucher) | Cash, Reporting | CN-5-004 §3, §6 | emitted | UI-01 | |
| `procurement.invoice.paid.v1` | Procurement | Reporting | CN-5-004 §3, §8 | emitted | UI-01, UI-08 | Workflow status after Cash settlement |
| `procurement.invoice.partially_paid.v1` | Procurement | Reporting | CN-5-004 §3, §8 | emitted | UI-01, UI-08 | Partial settlement (accepted per Q4 E7) |
| `procurement.return.initiated.v1` | Procurement | Reporting | CN-5-004 §3, §12 | emitted | UI-01, UI-02 | |
| `procurement.credit_note.recorded.v1` | Procurement (Document) | Accounting, Inventory, Cash, Reporting | CN-5-004 §3, §6, §12 | emitted | UI-01, UI-02 | Compensating Document |
| `procurement.return.completed.v1` | Procurement | Reporting | CN-5-004 §3, §12 | emitted | UI-01 | |
| `procurement.invoice.fx_applied.v1` | Procurement | Accounting, Reporting | CN-5-102 §3 (post-E4) | future | UI-01, UI-06 | **Per E4 — formalised in CN-5-103; producer emission pending CN-5-004 amendment** |

**Total: 20 commands, 26 events (1 future).**

---

## 12. Master Catalogue — HR & Payroll (CN-5-005)

### Commands

| Command | Producer | Doc ref | State | Notes |
|---------|----------|---------|-------|-------|
| `hr.employee.register.request` | human:authorised | CN-5-005 §3, §4 | emitted | Tenant scope |
| `hr.employee.attributes_update.request` | human:authorised | CN-5-005 §3 | emitted | |
| `hr.employee.deactivate.request` | human:authorised | CN-5-005 §3 | emitted | Termination prelude |
| `hr.role.define.request` | human:authorised | CN-5-005 §3, §5 | emitted | |
| `hr.role.update.request` | human:authorised | CN-5-005 §3 | emitted | |
| `hr.role.deactivate.request` | human:authorised | CN-5-005 §3 | emitted | |
| `hr.assignment.create.request` | human:authorised | CN-5-005 §3, §5 | emitted | Site-anchored |
| `hr.assignment.end.request` | human:authorised | CN-5-005 §3 | emitted | |
| `hr.assignment.transfer.request` | human:authorised | CN-5-005 §3 | emitted | Cross-site (tenant) |
| `hr.compensation.set.request` | human:authorised | CN-5-005 §3, §5 | emitted | |
| `hr.compensation.revise.request` | human:authorised | CN-5-005 §3 | emitted | |
| `hr.leave.balance.set.request` | human:authorised | CN-5-005 §3, §6 | emitted | Opening balance |
| `hr.leave.request.request` | human:employee | CN-5-005 §3, §6 | emitted | Site |
| `hr.leave.approve.request` | human:approver | CN-5-005 §3, §6 | emitted | |
| `hr.leave.cancel.request` | human:employee | CN-5-005 §3 | emitted | |
| `hr.attendance.record.request` | human:cashier OR system | CN-5-005 §3, §7 | emitted | Site |
| `hr.overtime.record.request` | human:cashier | CN-5-005 §3, §7 | emitted | Site |
| `hr.accrual.run.request` | system:scheduler | CN-5-005 §3, §6 | emitted | G4 — engine-owned scheduler (N2) |
| `hr.payroll.run.request` | human:authorised | CN-5-005 §3, §8 | emitted | Per period |
| `hr.payroll.approve.request` | human:authorised (per threshold) | CN-5-005 §3, §8 | emitted | Pre-disbursement gate (N3) |
| `hr.payroll.mark_paid.request` | system:hr-orchestrator | CN-5-005 §3, §8 | emitted | After all employees disbursed (N4) |
| `hr.loan.request.request` | human:employee | CN-5-005 §3, §10 | emitted | Site |
| `hr.loan.approve.request` | human:authorised | CN-5-005 §3, §10 | emitted | Creates Obligation |
| `hr.loan.reject.request` | human:authorised | CN-5-005 §3 | emitted | |
| `hr.loan.write_off.request` | human:authorised + dual-actor | CN-5-005 §3, §10 | emitted | Termination/default |
| `hr.termination.initiate.request` | human:authorised | CN-5-005 §3, §12 | emitted | |
| `hr.termination.finalize.request` | human:authorised + dual-actor | CN-5-005 §3, §12 | emitted | Loan resolution required |

### Events

| Event | Producer | Consumers | Doc ref | State | UI inv | Notes |
|-------|----------|-----------|---------|-------|--------|-------|
| `hr.employee.registered.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.employee.attributes_updated.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.employee.deactivated.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.role.defined.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.role.updated.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.role.deactivated.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.assignment.created.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.assignment.ended.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.assignment.transferred.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.compensation.set.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.compensation.revised.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.leave.balance.set.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.leave.requested.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.leave.approved.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.leave.rejected.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.leave.cancelled.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.leave.accrued.v1` | HR | Reporting | CN-5-005 §3, §6 | emitted | UI-01 | Scheduler-driven |
| `hr.attendance.recorded.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.overtime.recorded.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.payroll.computed.v1` | HR | Accounting (#10), Reporting | CN-5-005 §3, §8 | emitted | UI-01, UI-05, UI-06 | Accrual journals |
| `hr.payroll.approved.v1` | HR | Cash (N3 — schedules disbursement), Reporting | CN-5-005 §3, §8 | emitted | UI-01 | Post-gate |
| `hr.payroll.paid.v1` | HR | Accounting (#11), Reporting | CN-5-005 §3, §8 | emitted | UI-01, UI-05 | Atomic with deduction events (N4) |
| `hr.payroll.deduction.applied.v1` | HR | Cash (loan repayment), Reporting | CN-5-005 §3, §8, §10 | emitted | UI-01, UI-08 | Atomic with paid (N4) |
| `hr.loan.requested.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.loan.approved.v1` | HR | Cash (disburses), Reporting | CN-5-005 §3, §10 | emitted | UI-01, UI-08 | Creates Obligation |
| `hr.loan.rejected.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.loan.disbursed.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | Workflow status after Cash confirms |
| `hr.loan.write_off.requested.v1` | HR | Accounting, Cash (compensation), Reporting | CN-5-005 §3, §10 | emitted | UI-01, UI-02, UI-08 | |
| `hr.termination.initiated.v1` | HR | Reporting | CN-5-005 §3 | emitted | UI-01 | |
| `hr.termination.finalized.v1` | HR | Accounting, Cash, Reporting | CN-5-005 §3, §12 | emitted | UI-01, UI-08 | Final settlement |
| `hr.payroll.period.ready.v1` | HR | Accounting (CN-5-104 fan-in) | CN-5-001 §8 (referenced) | future | UI-01 | **CN-5-104 dependency (post-E1 alignment)** |

**Total: 27 commands, 31 events (1 future).**

---

## 13. Master Catalogue — Reporting & BI (CN-5-006)

### Commands

| Command | Producer | Doc ref | State | Notes |
|---------|----------|---------|-------|-------|
| `reporting.snapshot.create.request` | human:authorised OR system:scheduler | CN-5-006 §3, §10 | emitted | |
| `reporting.snapshot.invalidate.request` | human:authorised | CN-5-006 §3, §10 | emitted | |
| `reporting.statement.issue.request` | human:authorised (per pack) | CN-5-006 §3, §5, §8 | emitted | **Document via CN-4-012 per N3** |
| `reporting.statement.amend.request` | human:authorised | CN-5-006 §3 | emitted | New Document referencing original |
| `reporting.report.run.request` | human (any role with read access) | CN-5-006 §3, §5 | emitted | Live report (transient) |
| `reporting.kpi.define.request` | human:authorised | CN-5-006 §3, §9 | emitted | |
| `reporting.kpi.update.request` | human:authorised | CN-5-006 §3 | emitted | |
| `reporting.kpi.deactivate.request` | human:authorised | CN-5-006 §3 | emitted | |
| `reporting.alert.set.request` | human:authorised | CN-5-006 §3, §9 | emitted | |
| `reporting.alert.deactivate.request` | human:authorised | CN-5-006 §3 | emitted | |
| `reporting.export.generate.request` | human:authorised (per pack template) | CN-5-006 §3, §12 | emitted | |
| `reporting.projection.rebuild.request` | human:authorised OR system | CN-5-006 §3 | emitted | Heavy operation |

### Events (meta only — R1)

| Event | Producer | Consumers | Doc ref | State | UI inv | Notes |
|-------|----------|-----------|---------|-------|--------|-------|
| `reporting.snapshot.created.v1` | Reporting | (audit) | CN-5-006 §3, §10 | emitted | UI-01 | Non-truth marker per CN-4-018 |
| `reporting.snapshot.invalidated.v1` | Reporting | (audit) | CN-5-006 §3 | emitted | UI-01 | |
| `reporting.statement.issued.v1` | Reporting (Document) | (audit, downstream readers) | CN-5-006 §3, §5, §8 | emitted | UI-01, UI-05 | **Document terminal-fold (N3)** |
| `reporting.statement.amended.v1` | Reporting (Document) | (audit) | CN-5-006 §3 | emitted | UI-01, UI-05 | New Document references original |
| `reporting.report.run.v1` | Reporting | (audit) | CN-5-006 §3 | emitted | UI-01 | Live report meta-event |
| `reporting.kpi.defined.v1` | Reporting | (audit) | CN-5-006 §3 | emitted | UI-01 | |
| `reporting.kpi.updated.v1` | Reporting | (audit) | CN-5-006 §3 | emitted | UI-01 | |
| `reporting.kpi.deactivated.v1` | Reporting | (audit) | CN-5-006 §3 | emitted | UI-01 | |
| `reporting.kpi.recomputed.v1` | Reporting | (audit) | CN-5-006 §3 | emitted | UI-01 | |
| `reporting.alert.set.v1` | Reporting | (audit) | CN-5-006 §3 | emitted | UI-01 | |
| `reporting.alert.triggered.v1` | Reporting | Term 3 (dashboard) OR Term 7 (channels) per pack routing | CN-5-006 §3, §9 | emitted | UI-01 | N5 delivery boundary |
| `reporting.alert.cleared.v1` | Reporting | Term 3 / Term 7 | CN-5-006 §3 | emitted | UI-01 | |
| `reporting.export.generated.v1` | Reporting | (file recipient) | CN-5-006 §3, §12 | emitted | UI-01 | |
| `reporting.projection.rebuilt.v1` | Reporting (system) | (audit) | CN-5-006 §3 | emitted | UI-01 | |
| `reporting.metrics.published.v1` | Reporting | Term 1 platform aggregator | CN-5-101 §7 (post-E8) | emitted | UI-01 | Per-tenant metrics (E8 rename) |

**Total: 12 commands, 15 events. All emitted. All meta — no business truth (R1).**

---

## 14. Master Catalogue — Promotion (CN-5-007)

### Commands

| Command | Producer | Doc ref | State | Notes |
|---------|----------|---------|-------|-------|
| `promotion.campaign.create.request` | human:authorised | CN-5-007 §4 | emitted | |
| `promotion.campaign.activate.request` | human:authorised | CN-5-007 §4 | emitted | |
| `promotion.campaign.deactivate.request` | human:authorised | CN-5-007 §4 | emitted | |
| `promotion.campaign.amend.request` | human:authorised | CN-5-007 §4 | emitted | |
| `promotion.rule.define.request` | human:authorised | CN-5-007 §4 | emitted | |
| `promotion.rule.update.request` | human:authorised | CN-5-007 §4 | emitted | |
| `promotion.rule.deactivate.request` | human:authorised | CN-5-007 §4 | emitted | |
| `promotion.voucher.issue.request` | human:authorised OR system | CN-5-007 §4, §10 | emitted | Site OR tenant scope |
| `promotion.voucher.redeem.request` | human:cashier | CN-5-007 §4, §10 | emitted | At checkout |
| `promotion.voucher.cancel.request` | human:authorised | CN-5-007 §4 | emitted | |
| `promotion.voucher.expire_run.request` | system:scheduler | CN-5-007 §4 | emitted | G4 |
| `promotion.voucher.restoration.request` | system | CN-5-007 §4, §10 | emitted | Refund-driven |
| `promotion.loyalty.program.create.request` | human:authorised | CN-5-007 §4, §11 | emitted | |
| `promotion.loyalty.points.accrue.request` | system | CN-5-007 §4, §11 | emitted | From checkout.settled |
| `promotion.loyalty.points.redeem.request` | system | CN-5-007 §4, §11 | emitted | At checkout |
| `promotion.loyalty.points.expire_run.request` | system:scheduler | CN-5-007 §4 | emitted | G4 |
| `promotion.cost_share.declare.request` | human:authorised | CN-5-007 §4, §12 | emitted | Campaign setup |
| `promotion.cost_share.record.request` | system | CN-5-007 §4, §12 | emitted | At application |
| `promotion.manual_override.apply.request` | human:elevated (manager/owner) | CN-5-007 §4, §5 | emitted | Reason_ref required; Decision Journal |
| `promotion.outreach.send.request` | human:authorised | CN-5-007 §4, §13 | emitted | Consent-gated per PR5 |

### Events

| Event | Producer | Consumers | Doc ref | State | UI inv | Notes |
|-------|----------|-----------|---------|-------|--------|-------|
| `promotion.campaign.created.v1` | Promotion | Reporting | CN-5-007 §4 | emitted | UI-01 | |
| `promotion.campaign.activated.v1` | Promotion | Reporting | CN-5-007 §4 | emitted | UI-01 | |
| `promotion.campaign.deactivated.v1` | Promotion | Reporting | CN-5-007 §4 | emitted | UI-01 | |
| `promotion.campaign.amended.v1` | Promotion | Reporting | CN-5-007 §4 | emitted | UI-01 | |
| `promotion.rule.defined.v1` | Promotion | Reporting | CN-5-007 §4 | emitted | UI-01 | |
| `promotion.rule.updated.v1` | Promotion | Reporting | CN-5-007 §4 | emitted | UI-01 | |
| `promotion.rule.deactivated.v1` | Promotion | Reporting | CN-5-007 §4 | emitted | UI-01 | |
| `promotion.voucher.issued.v1` | Promotion (Document) | Reporting | CN-5-007 §4, §10 | emitted | UI-01 | Document terminal-fold |
| `promotion.voucher.redeemed.v1` | Promotion | Reporting, Cash (if voucher acts as tender) | CN-5-007 §4, §10 | emitted | UI-01 | |
| `promotion.voucher.cancelled.v1` | Promotion | Reporting | CN-5-007 §4 | emitted | UI-01 | Terminal |
| `promotion.voucher.expired.v1` | Promotion | Reporting, Accounting (if breakage active) | CN-5-007 §4 | emitted | UI-01 | Scheduler |
| `promotion.voucher.restoration.requested.v1` | Promotion | (engine resolves) | CN-5-007 §4, §10 | emitted | UI-01, UI-02 | N3 pack-driven |
| `promotion.voucher.restored.v1` | Promotion | Reporting | CN-5-007 §4 | emitted | UI-01, UI-02 | Pack policy |
| `promotion.voucher.burned.v1` | Promotion | Reporting | CN-5-007 §4 | emitted | UI-01, UI-02 | Pack policy |
| `promotion.voucher.partial_credit.v1` | Promotion | Reporting | CN-5-007 §4 | emitted | UI-01, UI-02 | Pack policy |
| `promotion.loyalty.program.created.v1` | Promotion | Reporting | CN-5-007 §4 | emitted | UI-01 | |
| `promotion.loyalty.points.accrued.v1` | Promotion | Reporting | CN-5-007 §4, §11 | emitted | UI-01, UI-08 | Obligation outstanding +/- |
| `promotion.loyalty.points.redeemed.v1` | Promotion | Cash (if tender), Reporting | CN-5-007 §4, §11 | emitted | UI-01, UI-08 | |
| `promotion.loyalty.points.expired.v1` | Promotion | Reporting | CN-5-007 §4 | emitted | UI-01, UI-08 | Scheduler |
| `promotion.cost_share.declared.v1` | Promotion | Reporting | CN-5-007 §4 | emitted | UI-01 | Campaign setup |
| `promotion.cost_share.recorded.v1` | Promotion | Accounting (#13), Reporting | CN-5-007 §4, §12 | emitted | UI-01, UI-08, UI-10 | UI-10 reconciliation; creates 3 receivable Obligations |
| `promotion.rule.applied.v1` | Promotion | Reporting (ROI analytics) | CN-5-007 §4 (post-E7/N2) | emitted | UI-01 | E7/N2 rename — was `promotion.applied.v1` |
| `promotion.manual_override.applied.v1` | Promotion | Decision Journal (CN-4-013), Reporting | CN-5-007 §4, §5 | emitted | UI-01 | Elevated audit |
| `promotion.outreach.requested.v1` | Promotion | Term 7 channel adapters | CN-5-007 §4, §13 | emitted | UI-01 | Consent-verified |
| `promotion.outreach.sent.v1` | Promotion | Reporting | CN-5-007 §4 | emitted | UI-01 | Adapter confirmation |
| `promotion.outreach.failed.v1` | Promotion | Reporting, Anomaly | CN-5-007 §4 | emitted | UI-01 | |
| `promotion.outreach.blocked.v1` | Promotion | Audit (consent absent) | CN-5-007 §4, §13 | emitted | UI-01 | Hard-block recorded |

**Total: 20 commands, 27 events. All emitted.**

---

## 15. Cross-Engine Subscription Matrix

Rows = producer event. Columns = consumer engine. Cell = handler kind (`P3` = command_emitting, `P2` = projection, `P4` = compensation). Marked **UI-04** if event participates in tender chain (UI-04 backward walk from `checkout.settled.v1`).

| Producer Event | Acct | Cash | Inv | Proc | HR | Rpt | Prom | UI-04 |
|----------------|:----:|:----:|:---:|:----:|:--:|:---:|:----:|:-----:|
| `checkout.settled.v1` | P3 | P3 | P3 | — | — | P2 | P3 | ✓ |
| `checkout.refunded.v1` | P4 | P4 | P4 | — | — | P2 | P4 | ✓ |
| `cash.tender.received.v1` | P3 | — | — | — | — | P2 | — | ✓ |
| `cash.tender.disbursed.v1` | P3 | — | — | — | P3 | P2 | — | — |
| `cash.expense.recorded.v1` | P3 | — | — | — | — | P2 | — | — |
| `cash.collection.received.v1` | P3 | — | — | — | — | P2 | P3 | ✓ |
| `cash.payment.disbursed.v1` | P3 | — | — | P3 | — | P2 | — | — |
| `cash.refund.issued.v1` | P3 | — | — | — | — | P2 | — | ✓ |
| `cash.transfer.between_sites.v1` | P3 | — | — | — | — | P2 | — | — |
| `cash.deposit.completed.v1` | P3 | — | — | — | — | P2 | — | — |
| `cash.equity.injected.v1` | P3 | — | — | — | — | P2 | — | ✓ |
| `cash.equity.withdrawn.v1` | P3 | — | — | — | — | P2 | — | — |
| `cash.adjustment.recorded.v1` | P3 | — | — | — | — | P2 | — | — |
| `inventory.stock.received.v1` | P3 | — | — | (P2) | — | P2 | — | — |
| `inventory.stock.deducted.v1` | P3 | — | — | — | — | P2 | — | — |
| `inventory.adjustment.recorded.v1` | P3 | — | — | — | — | P2 | — | — |
| `inventory.lot.expired.v1` | (P3 if breakage) | — | — | — | — | P2 | — | — |
| `inventory.promise.created.v1` | — | P3 | — | — | — | P2 | — | — |
| `procurement.grn.received.v1` | P3 (#8) | — | P3 | — | — | P2 | — | — |
| `procurement.invoice.recorded.v1` | P3 (#6) | — | — | — | — | P2 | — | — |
| `procurement.invoice.paid.v1` | — | — | — | (self) | — | P2 | — | — |
| `procurement.credit_note.recorded.v1` | P4 | — | P4 | — | — | P2 | — | — |
| `hr.payroll.computed.v1` | P3 (#10) | — | — | — | — | P2 | — | — |
| `hr.payroll.approved.v1` | — | P3 | — | — | — | P2 | — | — |
| `hr.payroll.paid.v1` | P3 (#11) | — | — | — | — | P2 | — | — |
| `hr.payroll.deduction.applied.v1` | — | P3 | — | — | — | P2 | — | — |
| `hr.loan.approved.v1` | — | P3 | — | — | — | P2 | — | — |
| `hr.loan.write_off.requested.v1` | P3 | P3 | — | — | — | P2 | — | — |
| `hr.termination.finalized.v1` | — | P3 | — | — | — | P2 | — | — |
| `promotion.cost_share.recorded.v1` | P3 (#13) | — | — | — | — | P2 | — | — |
| `promotion.voucher.redeemed.v1` | — | (P3 if tender) | — | — | — | P2 | — | (✓) |
| `promotion.loyalty.points.redeemed.v1` | — | (P3 if tender) | — | — | — | P2 | — | (✓) |

**UI-04 backward walk**: From any `cash.tender.received.v1`, the causation chain must terminate in one of: `checkout.settled.v1`, `cash.refund.issued.v1`, `cash.transfer.received.v1`, `cash.collection.received.v1`, `cash.equity.injected.v1`. Marked ✓ above. (✓) marks events that participate when pack permits tender mode.

---

## 16. Vertical Namespace Pre-Allocation (Q6 (c) Hybrid)

Per CTR-038, the following vertical namespaces are **reserved** for Term 6 ownership. **Namespace ownership is pre-allocated** here; **specific event names within each namespace are deferred** to Term 6 ratification per vertical doc.

| Namespace | Owner Term | Reserved for | First-doc target |
|-----------|-----------|--------------|------------------|
| `retail.*` | Term 6 | Retail vertical engine | CN-6-001 (forthcoming) |
| `restaurant.*` | Term 6 | Restaurant vertical engine | CN-6-002 (forthcoming) |
| `hotel.*` | Term 6 | Hotel vertical engine | CN-6-003 (forthcoming) |
| `workshop.*` | Term 6 | Workshop vertical engine | CN-6-004 (forthcoming) |
| `pharmacy.*` | Term 6 | Pharmacy vertical engine | CN-6-005 (forthcoming) |
| `clinic.*` | Term 6 | Clinic vertical engine | CN-6-006 (forthcoming) |

### Current Vertical-Placeholder References (in Term 5 docs)

Term 5 docs reference future vertical events using angle-bracket placeholder `<vertical>` per E6/E8 fix:

| Reference pattern | Used in | Concrete examples (when Term 6 ratifies) |
|-------------------|---------|--------------------------------------------|
| `<vertical>.bill.ready.v1` | CN-5-009 §3, §12 | `retail.bill.ready.v1`, `restaurant.bill.ready.v1`, `hotel.bill.ready.v1`, `workshop.bill.ready.v1` |
| `<vertical>.ingredient.consumed.v1` | CN-5-003 §3, §5 | `restaurant.ingredient.consumed.v1` |
| `<vertical>.cut.executed.v1` | CN-5-003 §3, §5, §10 | `workshop.cut.executed.v1` |
| `<vertical>.parametric.consumed.v1` | CN-5-003 §3 | `workshop.parametric.consumed.v1` |
| `<vertical>.amenity.consumed.v1` | CN-5-003 §3 | `hotel.amenity.consumed.v1` |
| `<vertical>.advance_consumed.v1` | CN-5-002 §4, §17 | Per-vertical fulfillment |
| `<vertical>.layby_release.v1` | CN-5-002 §4, §17 | Per-vertical layby release |
| `<vertical>.quote.accepted.v1` | CN-5-002 §17 (post-E6) | `workshop.quote.accepted.v1` |
| `<vertical>.purchase_need.recorded.v1` | CN-5-004 §3 | Per-vertical purchase trigger |
| `<vertical>.commission_earned.v1` | CN-5-005 §3, §11 | `retail.commission_earned.v1` |
| `<vertical>.piece_completed.v1` | CN-5-005 §3 | `workshop.piece_completed.v1` |
| `<vertical>.promotion_trigger.v1` | CN-5-007 §4 | Per-vertical promotion trigger (e.g., `restaurant.happy_hour.v1`) |

When Term 6 ratifies a vertical doc, the angle-bracket references in Term 5 docs become **emitted** (or amended via expansion notes — Overseer discretion).

---

## 17. Adapter Namespace Convention (G3) + Pack Namespace Pending CTR-037

### Adapter Namespace (G3 Ratified)

| Placeholder | Used by | Concrete examples (per deployment registry) |
|-------------|---------|----------------------------------------------|
| `<mm-adapter>` | CN-5-002, CN-5-009 | `mpesa-adapter`, `tigopesa-adapter`, `airtelmoney-adapter` |
| `<card-adapter>` | CN-5-002, CN-5-009 | `visa-adapter`, `mastercard-adapter`, `stripe-adapter` |
| `<bank-adapter>` | CN-5-002 (CTR-031) | `crdb-adapter`, `nmb-adapter` |
| `<fx-adapter>` | CN-5-001, CN-5-102 (post-E4) | `boi-fx-adapter`, `reuters-fx-adapter` |
| `<channel-adapter>` | CN-5-007 (CTR-021) | `sms-adapter`, `whatsapp-adapter`, `telegram-adapter`, `email-adapter` |

### Adapter Event Catalogue (Placeholder References)

| Pattern | Used by | Direction |
|---------|---------|-----------|
| `<mm-adapter>.tender.outcome.v1` | Cash, Checkout subscribe | Adapter → engine |
| `<mm-adapter>.statement.received.v1` | Cash subscribes | Adapter → engine (daily reconciliation) |
| `<card-adapter>.tender.outcome.v1` | Cash, Checkout subscribe | Adapter → engine |
| `<bank-adapter>.deposit.outcome.v1` | Cash subscribes | Adapter → engine |
| `<fx-adapter>.fx.rate.recorded.v1` | Accounting subscribes (per CN-5-001 N1) | Adapter → engine |
| `<channel-adapter>.outreach.outcome.v1` | Promotion subscribes | Adapter → engine |

### Pack Namespace — Pending CTR-037

Pack-emitted events (e.g., `pack.upgraded.v1`, `pack.rotation.scheduled.v1`) — namespace + ownership clarification pending Term 4 ruling per CTR-037. CN-5-103 §17 updates upon resolution. Until then:
- **Reserved**: namespace `pack.*` and `kernel.pack.*` both reserved pending decision; neither is allocated to a Term 5 engine.
- **Producer**: likely Compliance DSL evaluator (CN-4-015) or Term 1 (governance).
- **Consumers**: all engines on pack rotation (recognition rules re-evaluation, document template upgrades, etc.).

---

## 18. Naming Audit Findings (E1–E10) — Status

| # | Finding | Status |
|---|---------|--------|
| **E1** | `payroll.*` legacy references in CN-5-001 (manifest #10/#11, §4 catalogue, §8 fan-in, §6 mapping, §example) and CN-5-002 (manifest subscription, §H §J §9 §16 example) — 9 occurrences total | ✅ **Cosmetic applied** during CN-5-103 ratification (single pass — see commit) |
| **E2** | `inventory.movement.transferred.v1` (fictitious) referenced in CN-5-001 §4 catalogue #5 footnote | ✅ **Cosmetic applied** — replaced with `inventory.transfer.initiated.v1` (and `.received.v1`) paired |
| **E3** | `cash.position.snapshot.v1` (not in CN-5-002 manifest) referenced in CN-5-101 §6 multi-scope example | ✅ **Cosmetic applied** — replaced with `cash.tender.received.v1` (concrete emitted event) |
| **E4** | `procurement.fx.recorded.v1` ambiguity in CN-5-102 §3 UI-06 example vs G3 adapter convention | ✅ **Cosmetic applied** — split into `<fx-adapter>.fx.rate.recorded.v1` (adapter-emitted rate) + `procurement.invoice.fx_applied.v1` (procurement-emitted application). Latter added to §11 catalogue as `future` until CN-5-004 amendment formalises |
| **E5** | `accounting.correction.recorded.v1` vs actual `accounting.journal.reversed.v1` | ✅ **Resolved per N1** — use `accounting.journal.reversed.v1` with new `posting_period_ref` payload field (current open period; explicit forward-period anchor). Cosmetic applied to CN-5-001 §9 and manifest. No new event type |
| **E6** | Concrete vertical names (`workshop.layby_release.v1`, `workshop.quote.accepted.v1`) in CN-5-002 §17 layby example | ✅ **Cosmetic applied** — replaced with `<vertical>.*` angle-bracket per CTR-030 / Q6 |
| **E7** | Minor naming awkwardness: `reporting.metrics.monthly.v1` (adjective), `promotion.applied.v1` (missing noun); also flagged `procurement.invoice.partially_paid.v1` (adverb-form), `reporting.report.run.v1`, `cash.advance_received.recorded.v1` | ✅ **Per N2** — 2 renames applied: `reporting.metrics.monthly.v1` → `reporting.metrics.published.v1` (CN-5-101 §7); `promotion.applied.v1` → `promotion.rule.applied.v1` (CN-5-007 — all 6 occurrences). Other 3 accepted as-is per Q4 ruling (well-formed compound nouns or past-participle verbs). |
| **E8** | Vertical-placeholder events (~12 patterns across docs) | ✅ **Per Q6 (c) hybrid + CTR-038** — namespace ownership pre-allocated (§16); specific event names deferred to Term 6 |
| **E9** | Adapter placeholders consistent across docs | ✅ Clean — G3 ratifies; §17 catalogues |
| **E10** | `kernel.*` references — verify clean | ✅ Clean — only legitimate Foundation references; G2 honoured throughout. Per CTR-036, DC-042 will CI-enforce |

---

## 19. Versioning & Deprecation Policy

### Versioning (G6)

| Rule | Detail |
|------|--------|
| Baseline | `.v1` for every new event type at first ratification |
| Additive change | Within `.v1`, fields may be added (CN-4-002 §3); existing fields cannot be renamed or semantically redefined |
| Breaking change | Requires `.v2` (or new event type if semantics changed). `.v1` and `.v2` coexist during compatibility window |
| Compatibility window | Pack-driven (typically months). Producer emits both versions during window; subscribers handle both per CN-4-010 §7 (upcast or fail; never silently skip) |
| Sunset | After window, producer ceases `.v1` emission. Existing `.v1` events remain in store per CN-4-001 immutability (replay still supports them indefinitely) |
| Deprecated marking | State transitions `emitted` → `deprecated` per N4; CHANGELOG records |

### Deprecation Lifecycle

```
[emitted (v1)] ─── breaking change needed ───►
                                                 [v2 introduced; both emitted]
                                                                                ─── window expires ───►
                                                                                                          [v1 deprecated; v2 emitted only]
```

Subscribers must handle `.v2` before producer transitions `.v1` to deprecated. Doctrine check per CN-4-019 verifies handlers exist for all `emitted` versions.

### Replay Implications (D-009 Freeze)

A `.v1` event from 2026 remains interpretable in 2046 under its original `pack_version_ref` per D-009. Even if `.v1` is deprecated in 2030, existing 2026 events replay correctly via version-aware handlers (CN-4-010 §7).

---

## 20. Governance — Adding New Events Post-Ratification (G7)

### Process

| Step | Who | What |
|------|-----|------|
| 1 | Engine owner Term | File CTR or amend engine doc declaring new event/command |
| 2 | Overseer | Triage CTR; route to glossary review |
| 3 | Concept Lead | Ratify or request changes |
| 4 | Overseer | Update CN-5-103 catalogue; CHANGELOG records; producer-doc amendment |
| 5 | Producer engine | Emit new event in production |

### Constraints

- New event MUST conform to G1–G9 naming doctrine
- Namespace ownership respected (no engine A emits under engine B's namespace)
- If new vertical event: emerges through Term 6 vertical-doc ratification per CTR-038 (not silently added in Term 5)
- If new pack event: pending CTR-037 resolution

### Open Items

| Item | Owner | Notes |
|------|-------|-------|
| **CTR-036**: `kernel.*` namespace reservation → DC-042 in CN-4-019 | Term 4 | Resolution lands CI enforcement of G2 |
| **CTR-037**: Pack-emitted events namespace clarification | Term 4 | §17 updates upon resolution; producer + event catalogue published then |
| **CTR-038**: Vertical namespace pre-allocation per Q6 (c) hybrid | Term 6 | §16 lists reserved namespaces; specific event names per vertical doc |
| `cash.period.ready.v1`, `inventory.period.ready.v1`, `hr.payroll.period.ready.v1` (CN-5-104 fan-in) | CN-5-104 (forthcoming) | Currently `future` state; transitions to `emitted` upon CN-5-104 ratification |
| `inventory.reorder_point.reached.v1` | CN-5-003 v2 | Currently `future`; v2 enhancement |
| `procurement.invoice.fx_applied.v1` (per E4) | CN-5-004 amendment | Currently `future`; transitions to `emitted` upon CN-5-004 amendment |
| Vertical event names per CTR-038 | Term 6 | All `<vertical>.*` references transition from placeholder to concrete |
| Pack events (`pack.*` or `kernel.pack.*`) per CTR-037 | Term 4 | Reserved; pending |
| DC-042 (G2 enforcement) | CN-4-019 | Per CTR-036 |
| Doctrine check for G9 verb tense (CI-detectable on schema validation) | CN-4-019 + future CTR | Architect mechanism |
| Glossary state-transition CHANGELOG discipline (when CN-5-103 updates) | Overseer + CHANGELOG | Process: every catalogue update records event-name + old-state → new-state |
| Worked example (§21) coverage of all UI invariants in single trace | (this doc) | See §21 |

**Note:** D-DISC-001 and D-DISC-002 (deferred discussions) are **not relevant** to event naming/glossary concerns. They address tenant-customer UX (Term 3) and POS self-service (future cross-term) respectively — orthogonal to CN-5-103 scope.

---

## 21. Worked Example — Trace ya Sale Moja Karakana Mwanza

*Karakana ya Mzee Hassan serves as illustrative context per D-004 #4 (peer-technical audience). The trace shows how ~13 events flow through the glossary, naming each producer, consumers, UI-01 causation, and UI invariants touched.*

**Scenario:** Customer buys aluminium window TZS 182,220 (3 components: frame + glass × 2 + installation labour). Pays cash + M-Pesa. October promotion: 5% loyalty bonus for loyalty members. Tenant: `mzee-hassan-karakana`. Site: `karakana-mwanza-001`. Pack: `tz-compliance-2026.07`.

### Event Trace (Chronological)

| # | Event | Producer | Consumers | causation_id | UI invariants touched |
|---|-------|----------|-----------|--------------|-----------------------|
| 1 | `<vertical>.bill.ready.v1` (Term 6 workshop, future) | Workshop (future) | Checkout | (vertical command) | UI-01 |
| 2 | `checkout.started.v1` | Checkout | (none — workflow begin) | → event #1 | UI-01 |
| 3 | `checkout.tender.confirmed.v1` (cash TZS 100,000 — sync) | Checkout | (Cash via #6) | → cashier command | UI-01, UI-06 |
| 4 | `checkout.tender.requested.v1` (M-Pesa TZS 82,220 — async) | Checkout | `<mm-adapter>` Term 7 | → cashier command | UI-01 |
| 5 | `<mm-adapter>.tender.outcome.v1` (confirmed, external_ref) | mpesa-adapter | Checkout | → event #4 | UI-01 |
| 6 | `checkout.tender.confirmed.v1` (M-Pesa confirmation) | Checkout | (Cash via #7) | → event #5 | UI-01, UI-06 |
| 7 | **`checkout.settled.v1`** | Checkout | **Accounting, Cash, Inventory, Promotion, Reporting** | → cashier settle.request | UI-01, UI-04, UI-06 |
| 8a | `accounting.journal.posted.v1` (revenue + tax + COGS) | Accounting | Reporting | → event #7 | UI-01, UI-05, UI-06, UI-07 |
| 8b | `cash.tender.received.v1` (drawer, cash TZS 100k) | Cash | Accounting, Reporting | → event #7 | UI-01, UI-04, UI-06 |
| 8c | `cash.tender.received.v1` (M-Pesa wallet, TZS 82,220) | Cash | Accounting, Reporting | → event #7 | UI-01, UI-04, UI-06 |
| 8d | `inventory.stock.deducted.v1` (aluminium frame × 1, per Pattern B) | Inventory | Accounting, Reporting | → event #7 (or vertical cut event) | UI-01, UI-03, UI-09 |
| 8e | `inventory.stock.deducted.v1` (glass × 2) | Inventory | Accounting, Reporting | → event #7 | UI-01, UI-03, UI-09 |
| 8f | `inventory.stock.deducted.v1` (installation labour — depends on Pattern A recipe if labour is item) | Inventory | Accounting, Reporting | → event #7 | UI-01, UI-03, UI-09 |
| 8g | `promotion.loyalty.points.accrued.v1` (loyalty bonus) | Promotion | Reporting | → event #7 | UI-01, UI-08 |
| 8h | `promotion.rule.applied.v1` (post-E7/N2 rename) | Promotion | Reporting | → event #7 | UI-01 |
| 8i | `promotion.cost_share.recorded.v1` (if campaign-funded; UI-10 split) | Promotion | Accounting (#13), Reporting | → event #7 | UI-01, UI-08, UI-10 |
| 8j | `checkout.receipt.issued.v1` (Document via CN-4-012) | Checkout | Reporting | → event #7 | UI-01 |
| 8k | Reporting projections update (no event — kind: projection per R1) | Reporting | — | — | — |

### What the Trace Demonstrates

- **UI-01 causation chain**: every derived event names its trigger; audit walks from receipt back to bill.ready
- **UI-04 tender chain**: backward walk from `cash.tender.received.v1` (8b, 8c) terminates in `checkout.settled.v1` (7) — authorised origin
- **UI-06 currency**: all events in TZS (tenant functional); UI-06 enforced at tender add + settle + journal
- **UI-07 trial balance**: journal (8a) balanced; tenant trial balance holds at settled position
- **UI-03 source-ref**: every stock deduction (8d/e/f) carries `source_ref: checkout.settled` or vertical equivalent
- **UI-09 site_id**: every site-scope event (8b/c/d/e/f) carries `site_id: karakana-mwanza-001` validated against registry
- **UI-08 obligation bounds**: loyalty Obligation outstanding +/- maintained ≥ 0
- **UI-10 cost-share**: if campaign-funded (8i), `bos_share + agent_share + supplier_share + tenant_share = total_discount`
- **G2 (kernel reserved)**: no `kernel.*` emission anywhere in trace
- **G3 (adapter convention)**: `<mm-adapter>` placeholder (5) deployed concretely as `mpesa-adapter`
- **G4 (no system.*)**: scheduler-triggered events absent in this real-time trace; all triggered by human:cashier
- **G5 (compensation pair)**: not exercised in this happy path; refund variant would show `checkout.refunded.v1` paired with `compensates_event_id = #7`
- **G7 (governance)**: every event in trace exists in §§7–14 catalogue with state `emitted`; no surprises
- **G8 (producer-authoritative)**: subscribers reference exact names (e.g., Accounting references `checkout.settled.v1`, not `sale.settled.v1`)
- **G9 (verb tense)**: all event verbs past-tense (`started`, `confirmed`, `settled`, `posted`, `received`, `deducted`, `accrued`, `applied`, `recorded`, `issued`)

### Refund Variant (Partial)

If customer returns 1 of the 3 components a week later:

```
checkout.refund.request → human:cashier-refund-authorised
checkout.refunded.v1 → compensates_event_id: #7; refund_kind: partial
  → Accounting: accounting.journal.reversed.v1 (UI-02; with posting_period_ref to current open period per N1/E5)
  → Cash: cash.refund.issued.v1 (compensation; UI-04 chain extends)
  → Inventory: inventory.stock.adjusted.v1 (restoration; source_ref: refund)
  → Promotion: promotion.loyalty.points.accrued.v1 (compensating, -proportional)
              promotion.cost_share.recorded.v1 (compensating; UI-10 holds proportionally)
              promotion.voucher.restoration.requested.v1 (if voucher involved)
  → Reporting: projections update
```

Same naming discipline, same UI invariants, same governance.

---

*— End of CN-5-103 —*
