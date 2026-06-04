# CN-6-002 — Restaurant Engine (F&B Cluster)

> **Term:** 6 — Vertical Engines
> **Status:** Concept Phase v1.0 — Draft for Overseer review
> **Branch:** `claude/brave-johnson-S7Lky`
> **Reading order:** CN-6-100..104 (cross-cutting cluster) → CN-6-001 (Retail; REST0 placement criterion) → CN-5-003/009/100/103/105 → CN-4-011/021 → **this doc**
> **Ordering authority:** TERM-6 Brief §13 — **second concrete vertical doc**; first application of F&B cluster doctrine; framework tested on more complex case (two-Workflow architecture + recipe Pattern B + cross-vertical Obligation).

---

## 1. Purpose & Boundary

### 1.1 Mission

CN-6-002 declares the **Restaurant Engine** — the second concrete vertical, covering the **Food & Beverage (F&B) cluster** under the namespace `restaurant.*`. Per the F&B cluster doctrine ratified between CN-6-001 and this doc, the namespace word "restaurant" is canonical per CTR-038 + CN-5-103 §16; the doctrinal scope covers the full F&B spectrum: restaurant (sit-down), café, BBQ / nyama choma, pub, bar, fast-food, buffet, mama ntilie (street food), chai-jamia, hotel-attached restaurant, catering operation.

One vertical mechanism (BD5 split-it) + per-business-style pack configuration absorbs the cluster. A street-corner mama ntilie and a Lodge Serengeti fine-dining restaurant share the same engine — different in scale, configuration, and service style; same in business doctrine.

Per Brief §6.1: *"Tables, orders, kitchen workflow, split billing, self-service."* This doc operationalises that, plus the realities surfaced through Term 6's analysis: the F&B cluster (per BD6 3-of-3 fail for café/pub/BBQ individually), preparation-step boundary with retail (REST0), alcohol-licensing as pack concern (not vertical-defining), Hotel-Restaurant charge-to-room as concrete Obligation pattern (concretising CN-6-101 §11.4 + CN-6-104 §10 doctrine), and self-service via QR-table-ordering for the D-DISC-002 partial closure.

### 1.2 DOES vs DOES NOT

| CN-6-002 DOES | CN-6-002 DOES NOT |
|----------------|--------------------|
| Declare `restaurant.*` covering F&B cluster (restaurant + café + BBQ + pub + bar + mama ntilie + catering + hotel-attached) | Author cross-cutting framework (CN-6-100..104 complete) or replicate retail mechanics from CN-6-001 |
| Specify F&B Workflows: `restaurant.table_session`, `restaurant.order`, `restaurant.kitchen_ticket`, `restaurant.menu_management`, `restaurant.qr_order` | Specify Universal Checkout flow (CN-5-009) — restaurant emits, Checkout settles |
| Apply preparation-step criterion (REST0) for retail.* vs restaurant.* boundary | Audit which existing operations should re-classify (Architect phase) |
| Concretise Hotel-Restaurant charge-to-room via Obligation primitive (CN-6-101 §11.4 + CN-6-104 §10) | Document the Hotel folio side (CN-6-003 territory) |
| Define recipe Pattern B consumption as pack content (`pack.restaurant.recipes.<menu_item_id>`) per BD4 push-down | Create a recipe Foundation primitive — pack content is correct placement |
| Define split billing, tip handling, kitchen-routed ticket lifecycle | Specify kitchen hardware integration (Term 7 + Architect) |
| Document alcohol-licensing + food-hygiene as cross-vertical pack concerns; clarify BOS is NOT a licensing authority | Issue, verify, or autonomously gate based on regulatory licensing (BOS records + computes; authorities license; agents oversee) |
| Inherit RE11 payment-method abstraction throughout — no provider names anywhere | Mention specific tender providers (M-Pesa, Tigo, Airtel, card brands, banks) — payment is method-category abstract per CTR-028 + CTR-049/050 |
| Honour zero-new-CTR goal — F&B cluster fits existing contracts | Open new CTRs |

### 1.3 Audience

Term 6 itself (CN-6-003/004 authors follow this pattern); Architects implementing the restaurant engine across F&B spectrum; Term 1 onboarding governance authoring pack content per F&B sub-type; Term 3 designing waiter/cashier/customer UX (table-service POS, QR-table app, kitchen display screen); Term 7 wiring channel adapters for QR ordering; future tenants across the F&B cluster.

### 1.4 Charter Compliance

| Law | How CN-6-002 honours it |
|-----|--------------------------|
| Law 1 — State from events only | All restaurant state via event store; conflict events explicit (VI-03); compensation pairs declared (NC9) |
| Law 2 — Engines isolated | Hotel-Restaurant cross-vertical via Obligation primitive only (REST7); never direct subscription |
| Law 3 — AI advisory only | Restaurant-floor + kitchen + manager advisors per CN-5-010; license-expiry alerts advisory; never autonomous |
| Law 4 — Flexibility first-class | F&B cluster — one vertical absorbs spectrum from street food to fine dining via pack config |
| Law 5 — Compliance configured | Recipes, tip pools, split methods, service models all pack-driven; no hardcoded F&B logic per jurisdiction |
| Law 6 — Distribution regional | Payment methods + alcohol licensing both region-specific via packs; regional agent accountability per Charter Law 6 + D-003 |

### 1.5 Parsimony — F&B is where Tanzania's heart eats

Tanzania eats at street corners, in cafés, at family restaurants, in lodge dining rooms, at hotel bars, at wedding caterings. From Bibi Khadija's mama-ntilie corner in Kariakoo serving ugali na samaki to office workers on lunch break, to Lodge Serengeti's fine-dining room serving safari guests at sunset, F&B is the most-touched commercial layer in the country. Mzee Karim's bucha now adds a nyama-choma corner on weekends. Mzee Hamisi's Kariakoo pub serves cold beer + simple bar food. A wedding hall caterer prepares for 500 people at Sunday lunch.

All of them are restaurant.* tenants. Same engine, different pack configuration. The framework must hold from the simplest case (Bibi Khadija's single charcoal stove) to the most complex (a multi-station kitchen with bar, hot-line, dessert station, and wine cellar serving 80 covers across 20 tables on a Friday). If the doctrine fails Bibi Khadija, BOS has failed the majority of Tanzanian F&B SMEs.

**Parsimony is the bar.** Each REST1-REST8 below is justified against the question: "does this match how the real cook, real waiter, real cashier in real Tanzanian F&B operations thinks?"

---

## 2. Inputs and Relationship

### 2.1 Cross-cutting framework (parent)

- **CN-6-100** — Recipe; manifest delta; VE1–VE7
- **CN-6-101** — BD1–BD8; BD5 split-it (F&B cluster); BD6 3-of-3 (Hotel separate; café/pub/BBQ fail individually)
- **CN-6-102** — NC1–NC9 naming; VI-03 conflict family
- **CN-6-103** — SP1–SP8; site-scope default; menu management tenant-scope
- **CN-6-104** — HO1–HO9 handoff; Obligation cross-vertical doctrine §10

### 2.2 Universal layer

- **CN-5-009** Universal Checkout — bill.ready consumption; K1/K2 idempotency
- **CN-5-001** Accounting — CTR-030 journal sufficiency
- **CN-5-003** Inventory — Pattern B vertical-managed consumption (recipe-driven per REST4)
- **CN-5-005** HR/Payroll — commission_earned per waiter (REST6)
- **CN-5-007** Promotion — multi-price layer resolution (inherited from retail)
- **CN-5-105** Tax-treatment per item + alcohol excise; tenant_tax_profile gate

### 2.3 Foundation

- **CN-4-011** Workflow + Party + Document + Obligation + Inventory Movement
- **CN-4-012** Document Engine — license documents at tenant activation (regulatory_evidence_refs)
- **CN-4-021** Saleable Line + Tender value shapes

### 2.4 Sibling Term 6

- **CN-6-001** Retail — RE11 payment abstraction inherited; REST0 preparation-step boundary with retail
- **CN-6-003** Hotel (future) — confirms Hotel-side Obligation subscription for charge-to-room (REST7)
- **CN-6-005** Bridges (future) — catalogs hotel↔restaurant in-stay dining; restaurant↔retail-shop (lodge gift shop) bundling

### 2.5 Brief grounding

- **Brief §6.1** Restaurant concept summary
- **Brief §7.2** Six restaurant edge cases (party-of-N split walk-out, simultaneous waiters, kitchen-modify, self-service QR, 86'd ingredient, regulars' tab)
- **Brief §11.5** Hotel + Restaurant cross-vertical case
- **Brief §11.9** Mama ntilie clustering (closed via REST0 cluster doctrine — mama ntilie is restaurant.* simplest config)

### 2.6 CTRs

- **No new CTRs from F&B mechanics** — cluster fits existing contracts
- CTR-049 (Term 6 → Term 2) regional payment curation — inherited from CN-6-001
- CTR-050 (Term 6 → Term 7) adapter coverage — inherited
- CTR-018/002/024/026/030/038/044/045/046/028/006/027 — cited as-is

---

## 3. Restaurant Doctrine — REST0 + REST1–REST8

### REST0 — F&B Cluster Coverage (Pre-Doctrine)

**`restaurant.*` covers the Food & Beverage (F&B) cluster.** Restaurant (sit-down), café, BBQ / nyama choma, pub, bar, fast-food, buffet, mama ntilie, chai-jamia, hotel-attached restaurant, catering. Namespace word "restaurant" is canonical per CTR-038 + CN-5-103 §16; doctrinal scope is the F&B cluster. Per BD5 split-it: one F&B vertical mechanism + per-business-style pack configuration. Per BD6 3-of-3: café/BBQ/pub fail individually; cluster passes as one vertical. Hotel remains separate per BD6 3-of-3 pass (multi-day reservation + folio + tourism licensing).

**Placement criterion (vs retail.*):** restaurant.* covers operations where **preparation happens between order and serve** (cooking, frying, grilling, plating, pouring, brewing, mixing). Retail.* covers operations that **transfer existing items without preparation** (scanning packaged goods, weighing bulk grain, handing over sealed bottle). Edge cases (e.g., a kiosk selling both packaged biscuits and freshly-poured chai) default to restaurant.* when the tenant identifies as F&B; otherwise Mixed-Vertical (retail.* + restaurant.*) is supported.

**Regulated layer:** alcohol licensing, food hygiene, and similar compliance concerns are **cross-vertical pack concerns**, NOT vertical-defining. Captured at tenant activation per CTR-045 + Foundation Document primitive (CN-4-012). **BOS is not a licensing authority** — licensing authorities issue licenses; tenants comply; regional agents capture evidence at onboarding per Charter Law 6 + D-003; BOS records, computes, and surfaces advisory alerts (Law 3). See §16.

### REST1 — `restaurant.*` covers F&B cluster spectrum

From simplest (mama ntilie single-cook-cashier with no tables, no tickets, takeaway only) to most complex (multi-station kitchen + bar + wine cellar + sommelier service across 20 tables of 80 covers). Pack hooks differentiate; doctrine is one. Per REST0.

### REST2 — Two-Workflow architecture (per-tenant variant via pack)

The session-level Workflow varies per `pack.restaurant.session_model`:

- **`table_session`** (sit-down): long-running, multi-order, table-anchored. Used by full-service restaurant, café-with-seating, hotel-attached restaurant.
- **`counter_order`** (counter service): per-customer one-shot. Used by café-takeaway, fast-food, mama ntilie, chai-jamia.
- **`bar_tab`** (bar service): long-running on a per-customer or per-party basis, drinks-centric. Used by pub, bar, lounge.
- **`takeaway_only`** (degenerate counter_order): single-transaction with no seating. Mama ntilie at a corner, kiosk chai vendor.

In all cases, the session contains one or more `restaurant.order` instances. The order is the **per-customer-decision unit** that flows to preparation (REST3).

### REST3 — Ticket lifecycle is pack-driven routing

Pack hook `pack.restaurant.ticket_routing` declares an enumerated list of stations (`kitchen`, `bar`, `grill`, `dessert`, `none`). Each `restaurant.order` is split into one ticket per station per relevant items. Tickets follow the lifecycle:

```
fired → cooking → ready → picked_up → served
```

Per CN-6-102 NC5 state-as-verb. Steady states between transitions (e.g., the kitchen is in `cooking` until ready) emit no events between named transitions.

Pack value `ticket_routing: [none]` (mama ntilie, single cook-cashier) means no separate ticket — the order itself is the preparation tracking unit; transitions emit on the order Workflow directly.

### REST4 — Recipe Pattern B consumption (pack content, not primitive)

Recipes belong in **pack content** (`pack.restaurant.recipes.<menu_item_id>`), not as a Foundation primitive extension. Per BD4 push-down + BD5 split-it: Universal Inventory (CN-5-003) provides the consumption *mechanism* (Pattern B); the *recipe content* (what ingredients in what quantities make this menu_item) is per-tenant per-jurisdiction pack content. Vertical emits `restaurant.ingredient.consumed.v1` per recipe per sold item; Inventory subscribes and deducts.

This parallels Workshop's parametric formulas (CN-6-004 future) — same Pattern B mechanism; different per-vertical pack content.

### REST5 — Split billing is pack-enumerated

Pack hook `pack.restaurant.split_billing.methods` enumerates supported methods: `[equal_share, by_share, by_item, by_item_with_shared]`. The customer chooses at checkout. The vertical emits `restaurant.bill.split.applied.v1 {split_method, split_lines: [...]}` carrying the chosen method and per-payer line breakdown. One event, enum field. Universal Checkout settles each split_line as separate tender (or as separate Obligations if any split-payer pays separately later).

### REST6 — Tip handling via CN-5-005 commission pattern

Tips are commissions earned per waiter. Vertical emits `restaurant.commission_earned.v1 {employee_ref, tip_amount, table_session_ref, period}` per CTR-030 expansion in CN-5-005. Tip pool rules pack-driven via `pack.restaurant.tip_pool_rules` ∈ {universal_pool, per_waiter, hybrid}:

- `per_waiter` — each waiter keeps tips on their assigned tables
- `universal_pool` — all tips pool; distributed per shift hours or other pack rule
- `hybrid` — front-of-house pool + back-of-house allocation per pack

Universal HR/Payroll subscribes; tax treatment per jurisdiction pack (some treat tips as taxable wages, some as pass-through, some as zero-rate).

### REST7 — Hotel-Restaurant charge-to-room via Obligation primitive

The canonical cross-vertical pattern (Brief §7.5 + §11.5; concretises CN-6-101 §11.4 + CN-6-104 §10 doctrine). Restaurant emits Obligation `kind: hospitality_charge` at bill emission when `payment_method_hint: charge_to_room`. Hotel folio Workflow (CN-6-003 future) subscribes via Foundation Obligation primitive; folio aggregates; settles at hotel checkout; Obligation resolves; Restaurant's `table_session` Workflow finalizes. **No direct hotel↔restaurant event subscription** (VE2 honoured). Full 5-step chain in §13.

### REST8 — Self-service via QR-table-ordering Workflow

Pack hooks `pack.restaurant.self_service_enabled: true` + `pack.restaurant.qr_ordering_enabled: true` activate the `restaurant.qr_order` Workflow — parallel architecture to retail.fulfillment (RE10 in CN-6-001). Customer scans table QR (Term 1 onboarding governance + CTR-049-style Business ID/QR format); browses menu; places order; settles via remote-eligible tender method (CTR-050); kitchen prepares; waiter serves. Customer-as-actor per CN-4-007 + D-DISC-002 partial closure.

---

## 4. The Manifest — Engine Declaration

```yaml
engine_id: restaurant
engine_kind: vertical
namespace_root: restaurant
multi_site_capable: false                          # SP1 default; chain operations use bundling per CN-6-103
scope_policy: site

workflow_instances:
  # Session-level Workflow — variant via pack.restaurant.session_model
  - workflow_id: restaurant.table_session          # used when session_model == table_session
    billable: true
    lifecycle_states: [opened, active, billing, completed, voided, archived]
    settlement_subscription:
      event_type: checkout.settled.v1
      filter: originating_workflow_ref == <self>
      transitions_to: completed

  - workflow_id: restaurant.counter_order          # used when session_model == counter_order
    billable: true
    lifecycle_states: [placed, preparing, ready, handed_over, completed, cancelled, archived]
    settlement_subscription:
      event_type: checkout.settled.v1
      filter: originating_workflow_ref == <self>
      transitions_to: preparing                    # settlement on place; preparation begins after

  - workflow_id: restaurant.bar_tab                # used when session_model == bar_tab
    billable: true
    lifecycle_states: [opened, accumulating, billing, completed, voided, archived]
    settlement_subscription:
      event_type: checkout.settled.v1
      filter: originating_workflow_ref == <self>
      transitions_to: completed

  - workflow_id: restaurant.qr_order               # REST8 — self-service customer-initiated
    billable: true
    lifecycle_states: [placed, confirmed, preparing, ready, served, completed, cancelled, archived]
    settlement_subscription:
      event_type: checkout.settled.v1
      filter: originating_workflow_ref == <self>
      transitions_to: confirmed

  # Order-level Workflow (within session)
  - workflow_id: restaurant.order
    billable: false                                # rolls up into session bill
    lifecycle_states: [placed, routed, in_preparation, completed, cancelled]

  # Ticket-level Workflow (per kitchen/bar/grill station)
  - workflow_id: restaurant.kitchen_ticket         # nominal name; sub-namespace per station
    billable: false
    lifecycle_states: [fired, cooking, ready, picked_up, served, recalled]

  # Menu management (tenant-scope per SP3)
  - workflow_id: restaurant.menu_management
    billable: false
    scope_ref: tenant
    lifecycle_states: [draft, active, deprecated, archived]

commands:
  # session-level
  - restaurant.table_session.open.request
  - restaurant.counter_order.place.request
  - restaurant.bar_tab.open.request
  - restaurant.qr_order.place.request              # actor: customer per CN-4-007

  # order-level
  - restaurant.order.place.request
  - restaurant.order.modify.request
  - restaurant.order.cancel.request

  # ticket-level
  - restaurant.kitchen_ticket.fire.request
  - restaurant.kitchen_ticket.mark_cooking.request
  - restaurant.kitchen_ticket.mark_ready.request
  - restaurant.kitchen_ticket.pick_up.request
  - restaurant.kitchen_ticket.mark_served.request
  - restaurant.kitchen_ticket.recall.request

  # billing
  - restaurant.session.bill.request                # generic — applies to table_session, counter_order, bar_tab
  - restaurant.bill.split.apply.request
  - restaurant.session.void.request

  # cross-vertical
  - restaurant.charge_to_room.request              # emits Obligation per REST7

  # menu management (tenant-scope)
  - restaurant.menu.entry.add.request
    scope_ref: tenant
  - restaurant.menu.entry.update.request
    scope_ref: tenant

emits:
  # session-level
  - event_type: restaurant.table_session.opened.v1
    compensation_pair: restaurant.table_session.voided.v1
  - event_type: restaurant.counter_order.placed.v1
    compensation_pair: restaurant.counter_order.cancelled.v1
  - event_type: restaurant.bar_tab.opened.v1
    compensation_pair: restaurant.bar_tab.voided.v1
  - event_type: restaurant.qr_order.placed.v1
    compensation_pair: restaurant.qr_order.cancelled.v1

  # order-level
  - event_type: restaurant.order.placed.v1
    compensation_pair: restaurant.order.cancelled.v1
  - event_type: restaurant.order.modified.v1
    compensation_basis_none: "modification is forward-only; original order superseded by causation chain"

  # ticket-level (sub-namespace per station)
  - event_type: restaurant.kitchen.ticket.fired.v1
    compensation_pair: restaurant.kitchen.ticket.recalled.v1
  - event_type: restaurant.kitchen.ticket.cooking_started.v1
    compensation_basis_none: "observable physical work; ticket.recalled at session level"
  - event_type: restaurant.kitchen.ticket.ready.v1
    compensation_basis_none: "ready is observation"
  - event_type: restaurant.kitchen.ticket.picked_up.v1
    compensation_basis_none: "pickup is observation"
  - event_type: restaurant.kitchen.ticket.served.v1
    compensation_basis_none: "service is observation"
  # parallel for bar / grill stations
  - event_type: restaurant.bar.ticket.fired.v1
    compensation_pair: restaurant.bar.ticket.recalled.v1
  - event_type: restaurant.grill.ticket.fired.v1
    compensation_pair: restaurant.grill.ticket.recalled.v1

  # consumption (Pattern B)
  - event_type: restaurant.ingredient.consumed.v1  # REST4
    compensation_basis_none: "consumed material physically used; no event reversal — disposal/waste tracked separately"

  # billing
  - event_type: restaurant.bill.ready.v1           # NC3 + VE4 + HO1 canonical
    compensation_pair: restaurant.bill.recalled.v1
  - event_type: restaurant.bill.split.applied.v1   # REST5; carries split_method enum
    compensation_basis_none: "split methodology applied at billing; rebill = new bill.ready"

  # cross-vertical
  - event_type: restaurant.charge.obligation_emitted.v1  # REST7; mirrors obligation.created.v1 for vertical audit
    compensation_basis_none: "emission is observation; Obligation lifecycle handles compensation"

  # commission/tip (REST6)
  - event_type: restaurant.commission_earned.v1
    compensation_pair: restaurant.commission_reversed.v1

  # conflict (VI-03)
  - event_type: restaurant.table.conflict.detected.v1
    compensation_basis_none: "conflict resolution recorded; losing workflow received own compensation"
  - event_type: restaurant.kitchen.ticket.conflict.detected.v1
    compensation_basis_none: "conflict resolution recorded"

  # customer interaction
  - event_type: restaurant.customer.seated.v1
    compensation_basis_none: "seating is observation"
  - event_type: restaurant.customer.identified.v1
    compensation_basis_none: "identification is observation"

  # menu management (tenant-scope)
  - event_type: restaurant.menu.entry.added.v1
    compensation_pair: restaurant.menu.entry.removed.v1
    scope_ref: tenant
  - event_type: restaurant.menu.entry.updated.v1
    compensation_pair: restaurant.menu.entry.reverted.v1
    scope_ref: tenant
  - event_type: restaurant.menu.86_listed.v1       # ingredient depleted; menu item temporarily unavailable
    compensation_pair: restaurant.menu.86_lifted.v1
    scope_ref: site

subscribes_to:
  - event_type: checkout.settled.v1
    scope_ref: site
    # HO9 dual: routes to table_session / counter_order / bar_tab / qr_order per originating_workflow_ref
  - event_type: inventory.stock.depleted.v1
    scope_ref: site
    # triggers menu.86_listed for affected items per pack rule
  - event_type: obligation.settled.v1
    scope_ref: site
    filter: kind == "hospitality_charge" && counterparty_engine == "restaurant"
    # REST7: when Hotel resolves the obligation at checkout, restaurant finalizes its workflow
  - event_type: promotion.rule.applied.v1
    scope_ref: site
    # read-only awareness
  - event_type: pack.effective.v1
    scope_ref: tenant

pack_hooks:
  # F&B cluster configuration (REST0)
  - pack.restaurant.service_model                  # table_service | counter_service | grill_to_table | bar_only | hybrid | catering
  - pack.restaurant.ticket_routing                 # multi-value: [kitchen, bar, grill, dessert, none]
  - pack.restaurant.session_model                  # table_session | counter_order | bar_tab | takeaway_only
  - pack.restaurant.menu_complexity                # simple | standard | complex
  - pack.restaurant.fulfilment_model               # dine_in | takeaway | catering | delivery

  # recipes (REST4)
  - pack.restaurant.recipes.<menu_item_id>         # ingredient_ref + quantity + measurement_unit per recipe

  # billing (REST5)
  - pack.restaurant.split_billing.methods          # enum subset of [equal_share, by_share, by_item, by_item_with_shared]

  # tip pool (REST6)
  - pack.restaurant.tip_pool_rules                 # universal_pool | per_waiter | hybrid

  # kitchen routing
  - pack.restaurant.kitchen_routing.default_station

  # cross-vertical
  - pack.restaurant.hotel_charge_to_room_enabled   # bool; activates REST7

  # self-service (REST8)
  - pack.restaurant.self_service_enabled
  - pack.restaurant.qr_ordering_enabled
  - pack.restaurant.qr_ordering_business_id_format  # via CTR-049 + Term 1

  # regulated layer (cross-vertical pack concerns; §16)
  - pack.restaurant.alcohol_licensing_required     # bool
  - pack.restaurant.food_hygiene_compliance        # pack rules per jurisdiction

  # payment method abstraction (RE11 inherited from CN-6-001)
  # Region + tenant pack hooks live OUTSIDE restaurant pack:
  #   pack.region.payment_methods_enabled — Term 2 regional curation per CTR-049
  #   pack.tenant.payment_methods_subset — tenant configuration per CTR-049
  # restaurant.bill.ready.v1 carries NO method hint; Checkout adjudicates per CTR-028.

advisor_ids:
  - restaurant-floor-advisor                       # suggests reseating, rush prep, slow tables
  - restaurant-kitchen-advisor                     # suggests pacing, 86 alerts, prep prioritisation
  - restaurant-manager-advisor                     # KPI explanations, daily/weekly performance, license expiry alerts
```

The manifest is the canonical declaration. Every CN-6-002 worked pattern below resolves against it.

---

## 5. Workflow Lifecycle — `restaurant.table_session` State Machine

### 5.1 States + transitions

```
                       command: table_session.open.request (waiter action)
                                ▼
                       ┌────────────────┐
                       │     opened     │ ── emit restaurant.table_session.opened.v1
                       └────────────────┘
                                │  customer seated; first order placed
                                ▼
                       ┌────────────────┐
                       │     active     │ ◄── (recurring; multiple restaurant.order workflows attached)
                       └────────────────┘
                                │  command: session.bill.request (waiter)
                                ▼
                       ┌────────────────┐
                       │    billing     │ ── emit restaurant.bill.ready.v1 → Universal Checkout (HO1)
                       └────────────────┘
                                │
                                │ HO9: checkout.settled.v1 with matching originating_workflow_ref
                                │ OR obligation.settled.v1 (REST7 charge-to-room resolution path)
                                ▼
                       ┌────────────────┐
                       │   completed    │ ── emit causal event (varies per settlement path)
                       └────────────────┘
                                │  retention window
                                ▼
                       ┌────────────────┐
                       │    archived    │
                       └────────────────┘

Branches:
    any pre-billing → voided (walk-out, system void, manager cancellation)
```

### 5.2 Per-transition guards

| Transition | Guard |
|------------|-------|
| opened → active | First successful `restaurant.order.place.request` accepted |
| active → billing | Waiter commits to bill (no further orders accepted; outstanding orders must be in `completed` or `cancelled` state) |
| billing → completed | HO9 settlement OR REST7 Obligation resolution (depending on payment_method_hint) |
| any → voided | Walk-out timeout (per CN-6-104 N3 Restaurant default 0.5h post-bill) OR manager void |

### 5.3 Orders within a session

A table_session may contain N order workflows. Each `restaurant.order` is a per-customer-decision instance — a couple at a 2-top may produce 2 orders (each person ordering individually); a party of 8 may produce 8+ orders (multiple courses, drinks added later). Orders accumulate against the same session_id; bill.ready aggregates all order line totals.

### 5.4 Concurrency

Multi-waiter operations: multiple waiters may attach to one table_session via the same instance (collaborative service). Conflict only on initial open — `restaurant.table.conflict.detected.v1` fires if two waiters simultaneously call open.request for the same table at the same moment (VI-03; see §15).

---

## 6. Workflow Lifecycle — `restaurant.order` State Machine

### 6.1 The order is per-customer-decision

Within a session, an order is what one customer decides at one moment. "I'll have the wali na nyama and a chai." That's one order. Later "and a fresh juice please" is a second order on the same table_session. Each order is independently routed to its preparation station(s) via REST3.

### 6.2 States + transitions

```
                       command: order.place.request
                                ▼
                       ┌────────────────┐
                       │     placed     │ ── emit restaurant.order.placed.v1
                       └────────────────┘
                                │  vertical-internal: route to ticket(s) per pack.restaurant.ticket_routing
                                ▼
                       ┌────────────────┐
                       │     routed     │ ── emit restaurant.<station>.ticket.fired.v1 (one per station per applicable items)
                       └────────────────┘
                                │  tickets begin cooking
                                ▼
                       ┌──────────────────────┐
                       │  in_preparation       │ (steady; no event — emit on ticket transitions)
                       └──────────────────────┘
                                │  all tickets served
                                ▼
                       ┌────────────────┐
                       │   completed    │
                       └────────────────┘

Branches:
    placed or routed → cancelled (customer changes mind; pre-cooking)
    in_preparation → cancelled with compensation (kitchen partially prepared; pack rule for refund/waste)
```

### 6.3 Order modification

A customer says "actually, no nyama — make it kuku." If the kitchen hasn't started cooking, the ticket is recalled (`restaurant.kitchen.ticket.recalled.v1`) and a new order/ticket replaces it. If cooking has started, business decision per pack rule — typically the original cooked item is set aside (waste tracking via Inventory) and a new ticket fires. Emit `restaurant.order.modified.v1` with reference to original.

### 6.4 The `none` routing case (mama ntilie)

When `pack.restaurant.ticket_routing: [none]`, the order doesn't split into station tickets. The single cook-cashier prepares directly from the order. Order lifecycle: `placed → in_preparation → completed`. No ticket workflow instantiated. Pack-driven simplification.

---

## 7. Workflow Lifecycle — `restaurant.<station>.ticket` State Machine

### 7.1 Per-station tickets

Each entry in `pack.restaurant.ticket_routing` instantiates its own ticket sub-Workflow. For a hot-line + bar setup, an order containing wali (kitchen) and a fresh juice (bar) splits into two tickets — one fires at the kitchen, one at the bar. They progress independently.

### 7.2 States + transitions (per ticket)

```
                       routing event from order.routed
                                ▼
                       ┌────────────────┐
                       │     fired      │ ── emit restaurant.<station>.ticket.fired.v1
                       └────────────────┘
                                │  cook/bartender starts
                                ▼
                       ┌────────────────────┐
                       │  cooking_started   │ ── emit restaurant.<station>.ticket.cooking_started.v1 (optional; pack rule)
                       └────────────────────┘
                                │  preparation complete
                                ▼
                       ┌────────────────┐
                       │     ready      │ ── emit restaurant.<station>.ticket.ready.v1; waiter notification
                       └────────────────┘
                                │  waiter picks up
                                ▼
                       ┌────────────────┐
                       │   picked_up    │
                       └────────────────┘
                                │  delivered to table
                                ▼
                       ┌────────────────┐
                       │    served      │ ── emit restaurant.<station>.ticket.served.v1
                       └────────────────┘

Branches:
    fired or cooking_started → recalled (order modification or cancellation pre-ready)
```

### 7.3 Recipe consumption fires at `cooking_started` (Pattern B)

Per REST4, when a ticket transitions to `cooking_started` (or `fired` for simple-prep items per pack), the vertical emits `restaurant.ingredient.consumed.v1` events per the recipe in `pack.restaurant.recipes.<menu_item_id>`. Inventory subscribes and deducts ingredient stock. See §10.

### 7.4 86-listing (ingredient depleted mid-service)

When an inbound `inventory.stock.depleted.v1` arrives for an ingredient that's used in active recipes, the vertical may emit `restaurant.menu.86_listed.v1 {menu_item_refs, depleted_ingredient_ref}` — temporarily removing affected menu items from availability per pack rule. When stock replenishes, `restaurant.menu.86_lifted.v1` reverses. Brief §7.2 ingredient-depletion case.

---

## 8. Pack-Driven F&B Variations (REST0 Elaborated)

The cluster doctrine lives in pack configuration. Each F&B sub-type is the same vertical engine with different pack hooks.

### 8.1 Configuration matrix

| Sub-type | service_model | ticket_routing | session_model | menu_complexity | alcohol | session Workflow |
|----------|----------------|------------------|-----------------|------------------|---------|------------------|
| Lodge Serengeti restaurant (sit-down fine dining) | table_service | [kitchen, bar, dessert] | table_session | complex | true | restaurant.table_session |
| Café-with-seating (Salma's hypothetical café expansion) | hybrid | [kitchen, bar] | table_session | standard | false | restaurant.table_session |
| Café-takeaway (chai-jamia counter) | counter_service | [kitchen] | counter_order | simple | false | restaurant.counter_order |
| Mzee Karim BBQ corner | grill_to_table | [grill] | table_session OR counter_order | simple | false | per pack |
| Kariakoo pub (Mzee Hamisi) | bar_only | [bar] | bar_tab | simple_bar_food | true | restaurant.bar_tab |
| Mama ntilie (Bibi Khadija) | counter_service | [none] | takeaway_only | simple | false | restaurant.counter_order |
| Fast-food chain branch | counter_service | [kitchen] | counter_order | simple | false | restaurant.counter_order |
| Catering operation | catering | [kitchen] | (multi-leg per event) | standard | configurable | restaurant.table_session (per event) |
| Hotel-attached restaurant (Lodge Serengeti restaurant) | table_service | [kitchen, bar] | table_session | standard | true + `hotel_charge_to_room_enabled: true` | restaurant.table_session |

### 8.2 One engine, many configurations

Each row in the matrix is **the same engine** with different `pack.restaurant.*` values. Bibi Khadija and Lodge Serengeti are both restaurant.* tenants — their event vocabulary uses identical types (`restaurant.counter_order.placed.v1`, `restaurant.kitchen.ticket.fired.v1`, etc.); their pack configurations differ.

### 8.3 Adding a new F&B sub-type

A new style — say, a mobile food truck (Tanzania emerging in urban areas) — doesn't require new vertical doctrine. Pack configuration: `service_model: counter_service`, `ticket_routing: [kitchen]` (single galley), `session_model: counter_order`, `menu_complexity: simple`, `alcohol: false`, plus `fulfilment_model: takeaway` and optional `pack.restaurant.location_mobile: true` (new pack hook). Same engine. Mama Halima's logistics business could expand into a food truck without re-platforming.

---

## 9. Multi-Price + Menu Pricing

### 9.1 Inherited from RE3

The five-layer multi-price model from CN-6-001 RE3 applies: base_price + branch_override + active_promo + loyalty + customer_specific, resolved by Universal Checkout K2 + Promotion PR1 per `pack.restaurant.pricing.layer_order`.

### 9.2 Menu prices vs catalog prices

Restaurant menus differ from retail catalogs in two ways:

- **Items are recipe-defined** (REST4) — `pack.restaurant.recipes.<menu_item_id>` declares ingredients; menu price set per menu item, not per ingredient
- **Time-of-day pricing common** — lunch menu vs dinner menu; happy-hour bar prices; weekend brunch — handled via Promotion active_promo layer with time-window pack rules

### 9.3 Drinks pricing (pubs and bars)

Beer-by-the-bottle: simple unit pricing. Draught beer-by-the-pint: bulk-splittable per RE9 (inherited from retail) — `pack.retail.bulk_splittable_categories` extends conceptually; restaurant's `pack.restaurant.menu_complexity: simple_bar_food` may flag draught items for measurement_unit pricing if pack supports.

### 9.4 Pricing for bulk-splittable F&B items

Catering operations price per-head or per-kg of food prepared. Same RE9 bulk-splittable mechanism reused. Wedding catering for 200 guests at TZS 25,000 per head: 200 × 25,000 = 5,000,000 TZS. Mechanically identical to retail's by-kg pricing — different domain meaning.

---

## 10. Recipe Pattern B Consumption (REST4)

### 10.1 Pack-content placement (BD4 push-down)

Recipes live as **pack content**, not Foundation primitive. Format per `pack.restaurant.recipes.<menu_item_id>`:

```yaml
pack.restaurant.recipes.wali_na_kuku:
  ingredients:
    - ingredient_ref: rice-pishori, quantity: 0.18, measurement_unit: kg
    - ingredient_ref: chicken-thigh, quantity: 0.25, measurement_unit: kg
    - ingredient_ref: cooking-oil, quantity: 0.03, measurement_unit: litre
    - ingredient_ref: tomato, quantity: 0.10, measurement_unit: kg
    - ingredient_ref: onion, quantity: 0.05, measurement_unit: kg
    - ingredient_ref: salt-spice-pack-A, quantity: 1, measurement_unit: portion
```

### 10.2 Emission at `cooking_started` (or `fired` per pack)

When ticket transitions to `cooking_started` for an order containing `wali_na_kuku`, the vertical emits:

```
restaurant.ingredient.consumed.v1 {
  consumption_id,
  source_ref: <ticket_id>,                          # UI-03
  site_id,                                          # SP5
  recipe_ref: wali_na_kuku,
  consumed_items: [
    {ingredient_ref: rice-pishori, quantity: 0.18, measurement_unit: kg},
    {ingredient_ref: chicken-thigh, quantity: 0.25, measurement_unit: kg},
    ...
  ],
  business_date
}
```

Universal Inventory subscribes (Pattern B per CN-5-003 N1) and deducts each ingredient per site stock.

### 10.3 Variations / substitutions

If a substitution happens (customer requests no onion, or kitchen substitutes due to local 86-list), the recipe-driven event includes `substituted_for` references per CN-5-003 §5. The vertical emits the actual consumed list, not the canonical recipe list, when they differ.

### 10.4 Why pack content not primitive

A recipe primitive would impose a single recipe shape on every F&B operation across BOS. But recipes vary wildly: a fine-dining kitchen tracks gram-precision; a mama ntilie operation uses portion-counts ("scoop ya wali"). Pack content lets each tenant per jurisdiction declare their recipe shape; Inventory subscribes via shared Pattern B mechanism. BD4 push-down satisfied; BD5 split-it applied.

---

## 11. Split Billing (REST5)

### 11.1 The four pack-enumerated methods

`pack.restaurant.split_billing.methods` ∈ subset of `[equal_share, by_share, by_item, by_item_with_shared]`:

- **equal_share** — bill total divided by N payers. Used for friendly groups. "Tugawane sawa-sawa."
- **by_share** — N payers; each declares a fraction (1/N default; customisable shares for couples or unequal contributions).
- **by_item** — each payer pays for specifically chosen line items. Used when the party orders separately and wants individual receipts.
- **by_item_with_shared** — hybrid; each pays for own items + agreed shared lines (appetisers, wine) split per shared rule.

### 11.2 The emission

When customers choose to split at billing time:

```
restaurant.bill.split.applied.v1 {
  split_id,
  session_ref,
  split_method: equal_share | by_share | by_item | by_item_with_shared,
  split_lines: [
    {payer_party_ref: <p1>, lines: [...], amount: <subtotal>},
    {payer_party_ref: <p2>, lines: [...], amount: <subtotal>},
    ...
  ],
  business_date
}
```

Then `restaurant.bill.ready.v1` emits ONE bill carrying the split structure; Universal Checkout settles each split_line as a separate tender resolution. Different payers may use different tender methods (one cash, two via mobile money push, one charge-to-room) — all method-abstract per RE11.

### 11.3 Recovery from walk-out mid-split

Per Brief §7.2 walk-out scenario: party-of-8 splitting; one walks out without paying their share. Per CN-6-104 §8 N3, the unpaid portion converts to Obligation (`kind: unpaid_walkout`, debtor party_ref) per `pack.restaurant.bill_settlement_timeout_action`. The other seven payers settle their shares; the walked-out share becomes a recoverable obligation. Restaurant manager may pursue per business practice; BOS records.

### 11.4 Customer-chosen at table

Salma (in this scenario, working as floor server at Lodge Serengeti — character cross-doc reuse) asks the table: "Mama, mtagawanaje bili?" The waiter taps the chosen method on her POS; the per-payer breakdown displays; payers confirm; Universal Checkout opens N parallel tender flows.

---

## 12. Tip Handling (REST6)

### 12.1 Pack-driven pool rules

`pack.restaurant.tip_pool_rules` ∈ `{per_waiter, universal_pool, hybrid}`:

- **per_waiter** — Each waiter keeps tips from their assigned tables. Used by smaller operations where service is one-on-one.
- **universal_pool** — All tips collected centrally; distributed per shift hours (or other pack rule like seniority weights). Common in chain restaurants and high-volume sit-down.
- **hybrid** — Front-of-house pool + per-back-of-house allocation per pack. Some operations include kitchen in the pool partially.

### 12.2 Emission per table_session close

At table_session.completed (post-settlement), the vertical computes per-waiter commission per pack rule:

```
restaurant.commission_earned.v1 {
  commission_id,
  employee_ref: <waiter>,
  source_ref: <session_id>,
  tip_amount,
  basis: per_waiter | pool_share,
  period: <pay_period_ref>,
  business_date
}
```

### 12.3 HR/Payroll subscribes

Per CN-5-005 + CTR-030 expansion, HR aggregates commission_earned events per employee per pay period; payroll computation includes tips per pack tax rules (in TZ, tips on restaurant bills typically pass through as wages; in some jurisdictions, separately taxed).

### 12.4 Walk-out impact on tips

If a customer walks out (recovery via Obligation), the waiter's tip on that portion is reduced or eliminated per pack rule. The original `restaurant.commission_earned.v1` is compensated by `restaurant.commission_reversed.v1` when the walk-out recovery completes (or remains uncompensated if recovery succeeds via Obligation collection).

---

## 13. Hotel-Restaurant Charge-to-Room via Obligation Primitive (REST7)

### 13.1 The canonical cross-vertical pattern

Concretises CN-6-101 §11.4 + CN-6-104 §10 doctrine. The Restaurant emits its bill carrying a charge-to-room payment_method_hint; the Universal Checkout routes settlement to Obligation primitive (`kind: hospitality_charge`); the Hotel folio Workflow subscribes; settles at hotel checkout; Obligation resolves; Restaurant finalizes.

### 13.2 Pack flag activation

`pack.restaurant.hotel_charge_to_room_enabled: true` activates this path. Without the flag, the restaurant only accepts direct tender. Lodge Serengeti enables it (sister hotel.* tenant runs the property); a standalone restaurant (Mzee Hamisi pub; Bibi Khadija mama ntilie) leaves it false.

### 13.3 The 5-step chain

**Step 1 — Restaurant emits Obligation alongside bill.ready.**

```
restaurant.bill.ready.v1 {
  bill_id,
  site_id,
  saleable_lines: [...],
  originating_workflow_ref: <table_session>,
  payment_method_hint: charge_to_room,
  payer_party_ref: <hotel_guest>,
  obligation_emission: true                        # flag for Checkout routing
}
```

Universal Checkout sees the hint and routes:

```
obligation.created.v1 {
  obligation_id,
  kind: hospitality_charge,
  debtor_party_ref: <hotel_guest>,
  creditor_engine: restaurant,
  creditor_workflow_ref: <table_session>,
  amount: <bill_total>,
  counterparty_workflow_ref: <hotel_folio>,        # known via party_ref → folio lookup
  payload_ref: <restaurant.bill.ready event_id>,
  business_date
}
```

The vertical also emits `restaurant.charge.obligation_emitted.v1` for restaurant-internal audit.

**Step 2 — Hotel folio Workflow subscribes.**

CN-6-003 (future) declares:

```
subscribes_to:
  - event_type: obligation.created.v1
    filter: kind == "hospitality_charge" && counterparty_workflow_ref == <self>
```

The matching obligation arrives; Hotel folio Workflow incorporates.

**Step 3 — Hotel folio incorporates.**

Hotel emits:

```
hotel.folio.charge_added.v1 {
  folio_id,
  obligation_ref: <obl>,
  line_amount,
  line_description: "Restaurant — Lodge Serengeti dining",
  source_workflow_engine: restaurant,
  source_workflow_ref: <table_session>,
  business_date
}
```

**Step 4 — Hotel aggregates at folio bill.ready.**

When the guest checks out, hotel folio emits its bill.ready aggregating all folio lines (room nights, restaurant charges, mini-bar, laundry):

```
hotel.folio.ready.v1 {
  folio_id,
  saleable_lines: [room_nights, restaurant_charge_line, mini_bar_line, ...],
  obligation_refs: [<hospitality_charge_obl>, ...],
  ...
}
```

Universal Checkout settles via the guest's chosen tender method (RE11 abstract; could be card, mobile push, etc.).

**Step 5 — Obligation resolves; Restaurant finalizes.**

On `checkout.settled.v1` for the hotel folio, Foundation Obligation primitive emits:

```
obligation.settled.v1 {
  obligation_id: <hospitality_charge_obl>,
  resolved_via: <checkout.settled event_id>,
  ...
}
```

The Restaurant's subscription (manifest: `obligation.settled.v1` filtered by `kind == "hospitality_charge"` AND `creditor_engine == "restaurant"`) fires; the Workflow transitions table_session → `completed`; `restaurant.commission_earned.v1` emits for the waiter; revenue recognition completes for the restaurant.

### 13.4 What this proves (BD7 + VE2 honoured)

- Restaurant and Hotel never reference each other's events directly
- The Obligation primitive carries the cross-vertical relationship
- Either vertical can be decommissioned without breaking the other
- Accounting recognizes restaurant revenue only at obligation resolution (correct timing per CN-5-001 + N3 of CN-5-105 tax timing)
- CN-6-005 future will catalog this with other bridge patterns; CN-6-002 establishes the concrete REST7 instance

---

## 14. Self-Service + QR-Table-Ordering (REST8)

### 14.1 Pack activation

```
pack.restaurant.self_service_enabled: true
pack.restaurant.qr_ordering_enabled: true
pack.restaurant.qr_ordering_business_id_format: <CTR-049 + Term 1 governance>
```

### 14.2 The customer-as-actor flow

Per CN-4-007 + D-DISC-002 partial closure:

1. Customer at café/restaurant table scans QR code on the table tent. App opens (Term 3 surface) with the tenant's menu.
2. Customer browses, builds order, selects "place order"
3. `restaurant.qr_order.place.request` accepted; actor = customer per CN-4-007; payer_party_ref = customer's Party (registered via mobile app)
4. `restaurant.qr_order.placed.v1` emits
5. `restaurant.bill.ready.v1` also emits (payment-on-placement model for QR; tenant configurable per pack: payment-on-placement vs payment-on-serve)
6. Universal Checkout settles via remote-eligible tender method (CTR-050 confirms which methods are remote-capable)
7. `checkout.settled.v1` arrives → HO9 → `restaurant.qr_order.confirmed.v1` → kitchen receives ticket
8. Kitchen prepares; ticket lifecycle as normal
9. Waiter delivers to table — `restaurant.qr_order.served.v1` → terminal completion

### 14.3 Counter-service kiosk variant

For café/fast-food self-service kiosks (in-store, not at a table), same `restaurant.qr_order` Workflow with `service_model: counter_service` pack. Customer orders at kiosk; pays at kiosk; pickup at counter. Identical event sequence except the "serve" step is "pickup" — pack-driven semantic.

### 14.4 D-DISC-002 closure

CN-6-002 partially closes D-DISC-002 for the F&B vertical. Same mechanism is reused by the upcoming CN-6-001 RE10 retail.fulfillment (already done) and future verticals supporting self-service. Each vertical configures pack flags + Workflow as appropriate; the Foundation customer-as-actor mechanism is shared.

---

## 15. Conflict Event Family (VI-03)

### 15.1 The two conflict patterns

Per VI-03 ratification (CN-6-101 §12.1), the F&B vertical emits conflict events for two contention cases:

**restaurant.table.conflict.detected.v1** — two waiters simultaneously open the same table_session. Brief §7.2 case. The bus single-acceptance mechanism (CN-4-004) accepts one; the loser receives:

```
restaurant.table.conflict.detected.v1 {
  contested_resource_ref: <table>,
  winning_workflow_ref,
  contender_workflow_refs: [losing_workflow],
  detection_ts,
  resolution_basis: "bus_single_acceptance_first_arrival"
}
```

The losing waiter's UI displays "Table tayari imefunguliwa na Salma" so they know to ask Salma about it.

**restaurant.kitchen.ticket.conflict.detected.v1** — two prep stations try to claim the same ticket (rare; happens if kitchen routing is mis-configured and an item routes to both hot-line and cold-line). Bus single-acceptance resolves; the loser's UI displays the conflict.

### 15.2 Beyond table-claim conflicts

Other restaurant operations rarely produce conflict events — orders are append-only per session; kitchen tickets fire to their designated station; bills are per-session. Concurrent operations across different sessions don't conflict. Only resource-contention (table, ticket-station mis-routing) emits the conflict family.

### 15.3 Why explicit conflict events matter

Per CN-6-101 §12.1 VI-03 doctrine: conflicts that the bus rejects must surface as audit events. Silent rejection hides operational issues (e.g., chronic table-conflict events at a busy Friday night indicates UI design needs a "this table currently being attended" indicator). Brief §12 gap-prevention.

---

## 16. Alcohol Licensing + Food Hygiene — Cross-Vertical Pack Concerns

### 16.1 BOS is not a licensing authority

This deserves explicit doctrinal restatement before discussing alcohol or food hygiene mechanics:

> **BOS does not issue, verify, or autonomously gate based on regulatory licenses.** Per Charter §1.3 + Law 3 + Law 6 + D-003:
> - **Licensing authorities** (TZ Liquor Board, MoH, KEBS, TFDA in respective domains, regional equivalents) issue and govern licenses
> - **Tenants** hold licenses, comply with terms, pursue renewals
> - **Regional agents** (per Charter Law 6 + D-003) are L1 compliance-accountable for their region; capture license evidence at tenant onboarding per CTR-045 expansion
> - **BOS** records the license as a Foundation Document (CN-4-012) at activation; computes tax including excise per pack rules; surfaces license-expiry alerts via advisory framework (Law 3 — non-autonomous); supports authority inspection by providing evidence on demand

BOS is a recorder + computer + advisor — not a regulator.

### 16.2 Alcohol licensing pack mechanics

`pack.restaurant.alcohol_licensing_required` is a boolean pack flag set per tenant configuration at onboarding:

- `true` for pub (Mzee Hamisi), bar, restaurant with liquor license, hotel-attached restaurant serving alcohol
- `false` for café, mama ntilie (Bibi Khadija), chai-jamia, BBQ-only (Mzee Karim's bucha-pivot), most fast-food

When `true`:
- Tenant activation requires `regulatory_evidence_refs` including the license document via CTR-045 expansion (regional agent uploads at onboarding)
- Excise tax computation per pack `pack.tax.alcohol_excise_rules` (CN-5-105 inheritance)
- Manager advisor (CN-5-010) surfaces license-expiry alerts as the date approaches
- Receipt issuance may include license number per jurisdiction Document template (CN-4-012)

The flag does **not** autonomously block sales. If the license expires and the tenant continues operating, BOS records the operations; the regulatory authority and the regional agent are responsible for next steps.

### 16.3 Food hygiene compliance

Similar pattern. `pack.restaurant.food_hygiene_compliance` declares pack rules per jurisdiction:

- Pack content per region: which document templates apply (food handler certificates, premises inspection certificates)
- Onboarding evidence captured per pack
- Periodic re-certification reminders via manager advisor
- No autonomous enforcement

### 16.4 Multi-vertical scope of regulatory concerns

Alcohol licensing affects retail (`pack.retail.alcohol_licensing_required` for supermarkets selling beer) and hotel (mini-bar) just as it does restaurant. Food hygiene affects pharmacy (food-grade vs medical-grade), clinic (patient meals), restaurant. **Compliance is a cross-vertical pack concern** captured per vertical that touches the regulated domain. CN-6-002 covers the restaurant.* side; sister verticals declare their own pack flags.

---

## 17. Catering + Food Courts

### 17.1 Catering (per Q13 ruling)

`pack.restaurant.fulfilment_model: catering` activates the catering operational pattern:

- Preparation happens at one site (the catering kitchen)
- Service happens at another site (the event venue — wedding hall, conference centre, beach front)
- Per-event pricing (per-head × N + tier adjustments)
- Long preparation lead time (days, not minutes)
- Delivery to event venue typically requires Logistics integration (deferred to future Logistics vertical)

Mechanically: the catering operation uses `restaurant.table_session` per event (the event-anchored session) with a long opened-to-billing window. Recipe Pattern B consumption happens at the catering kitchen site (origin); the served-at-venue tickets are essentially terminal-`served` upon delivery. Single bill.ready emits per event.

### 17.2 Food courts (per Q12 ruling)

Multi-vendor food courts at malls operationally consist of multiple independent F&B operators sharing a venue. Per legal ownership:

- **Default — multi-tenant food court**: Each vendor is a separate tenant (separate ownership, separate books). Each tenant runs restaurant.* configured per their operation. The mall is a venue (a real-estate landlord), not a BOS tenant. Customers may visit multiple vendors during one mall trip — each transaction is a separate restaurant.* transaction at a separate tenant. Loyalty programmes (if any) operate per-tenant unless explicitly shared via Party primitive.

- **Alternative — one-tenant-multi-stall**: One operator owns all stalls. Operates as multiple site_ids within one tenant. Each site_id can have different `pack.restaurant.service_model` per its operation (a Korean stall, a Tanzanian stall, a juice bar). One set of books; per-site reporting.

Both supported by existing framework — Mixed-Vertical Tenant (CN-6-105 generalisation) or independent tenants per legal practice.

---

## 18. Worked Patterns — Eight Real Tanzanian F&B Scenarios

### 18.1 WP1 — Lodge Serengeti dinner service (canonical table_session)

**Time:** Saturday 7:30 pm. Lodge Serengeti restaurant. A party of 4 (Mzee Hassan and 3 colleagues from Arusha — cross-doc continuity) seats at table 12. Waiter Salma (cross-vertical role from CN-6-001 retail; in this scenario she works at Lodge Serengeti restaurant as floor server during a school break) takes orders.

**Flow:**

1. Salma opens session — `restaurant.table_session.open.request {table: 12, waiter: salma, party_size: 4}` → `restaurant.table_session.opened.v1`
2. Four orders placed (one per person): wali na samaki for Mzee Hassan, ugali na kuku for two colleagues, vegetarian sukuma mix for the fourth. Plus 4 drinks (2 sodas + 1 beer + 1 fresh juice).
3. Order routing per pack: food items → `restaurant.kitchen.ticket.fired.v1` (4 tickets); beer + juice → `restaurant.bar.ticket.fired.v1` (one combined bar ticket); sodas → directly to floor (no station ticket; bottles served immediately)
4. Kitchen prepares; per ticket: `cooking_started.v1` (recipe consumption fires via REST4 — `restaurant.ingredient.consumed.v1` per ticket per recipe per CN-5-003 Pattern B); when ready: `ready.v1`; Salma picks up: `picked_up.v1`; serves: `served.v1`
5. Bar ticket served separately (bartender prepares beer + fresh juice)
6. After dessert + coffee, Mzee Hassan calls for the bill. Salma issues `restaurant.session.bill.request` → Workflow → `billing`
7. `restaurant.bill.ready.v1` emits aggregating all 4 orders' line totals + 1 bar ticket
8. Mzee Hassan elects to pay for the whole table — single tender. Universal Checkout presents tenant's enabled methods (per CTR-049 regional curation + Lodge Serengeti tenant configuration; payment-method abstract throughout per RE11). Mzee Hassan chooses one of the enabled methods; settles.
9. `checkout.settled.v1` arrives → HO9 → `restaurant.table_session.completed.v1`
10. `restaurant.commission_earned.v1` emits for Salma per `pack.restaurant.tip_pool_rules` (Lodge Serengeti uses `hybrid` — Salma keeps direct table tip; back-of-house gets pool allocation)
11. Fan-out: Accounting projects revenue; Inventory had deducted ingredients during cooking (Pattern B); Promotion records loyalty for Mzee Hassan if he identified; Reporting feeds dashboard

**Doctrine demonstrated:** REST2 (table_session), REST3 (multi-station ticket routing), REST4 (recipe Pattern B), REST6 (tip pool), HO1 + HO9, payment-method abstract per RE11.

### 18.2 WP2 — VI-03 conflict (two waiters tap same table simultaneously)

**Time:** Friday 8:15 pm. Lodge Serengeti restaurant during peak service. Salma and another floor server, Asha, both move to attend a newly-vacated table 7. Both tap their POS tablets to open the session within 200 milliseconds of each other.

**Flow:**

1. Salma's command: `restaurant.table_session.open.request {table: 7, waiter: salma, ...}` at t=0
2. Asha's command: `restaurant.table_session.open.request {table: 7, waiter: asha, ...}` at t=0.2s
3. Bus single-acceptance (CN-4-004): Salma's command wins (first arrival); session opens with Salma as assigned waiter; emit `restaurant.table_session.opened.v1`
4. Asha's command rejected: bus emits `kernel.command.rejected.v1 {command_id: asha_cmd, reason: "conflict_lost"}`
5. The conflict event fires:

```
restaurant.table.conflict.detected.v1 {
  contested_resource_ref: <table-7>,
  winning_workflow_ref: <salma_session>,
  contender_workflow_refs: [<asha_cmd>],
  detection_ts,
  resolution_basis: "bus_single_acceptance_first_arrival"
}
```

6. Asha's POS displays: "Salma ameanza kuhudumia meza 7. Tafuta meza nyingine au msaidie Salma."

**Doctrine demonstrated:** VI-03 conflict event family per CN-6-101 §12.1; CN-6-002 §15.1 concretized.

### 18.3 WP3 — Hotel-Restaurant charge-to-room (Mama na Bwana Mwema at Lodge Serengeti)

**Time:** During Mama na Bwana Mwema's stay at Lodge Serengeti (cross-doc continuity from CN-6-102 WP1 + CN-6-104 WP3 prefiguration). They dine at the lodge restaurant on their second night; charge the bill to their room.

**Flow (the 5-step Obligation chain concretised per §13.3):**

1. Restaurant session as normal — table opened, orders placed, kitchen prepares, food served. Bibi Halima at the front desk has alerted Mama na Bwana Mwema's `hotel.guest_profile` so when they identify at the restaurant (`restaurant.customer.identified.v1 {party_ref: <mwema>}`), the system recognises them as in-house guests
2. Bill comes; Mwema waves it: "Charge to our room." Waiter taps "Charge to Room" on the POS
3. `restaurant.bill.ready.v1` emits with `payment_method_hint: charge_to_room`, `payer_party_ref: <mwema>`, `obligation_emission: true`
4. Universal Checkout routes per hint to Obligation primitive:

```
obligation.created.v1 {
  obligation_id,
  kind: hospitality_charge,
  debtor_party_ref: <mwema>,
  creditor_engine: restaurant,
  creditor_workflow_ref: <table_session>,
  amount: <bill_total>,
  counterparty_workflow_ref: <mwema_folio>,
  business_date
}
```

5. `restaurant.charge.obligation_emitted.v1` emits for restaurant audit
6. Restaurant `table_session` transitions to `billing` state but **does not** complete yet — awaits obligation.settled.v1
7. Hotel folio Workflow (CN-6-003 future scope; demonstrated here as future capability) subscribes per its manifest; receives the obligation; emits `hotel.folio.charge_added.v1` carrying the line + obligation_ref
8. Mwemas continue their stay. Two days later they check out. Hotel folio aggregates all stay charges including the restaurant line; emits `hotel.folio.ready.v1` to Universal Checkout; settlement happens (per Mwemas' chosen method — abstract per RE11)
9. `checkout.settled.v1` for the hotel folio arrives; Foundation Obligation primitive sees the settlement carrying the hospitality_charge obligation_ref; emits `obligation.settled.v1 {obligation_id: <restaurant_charge_obl>, resolved_via: <checkout.settled event_id>}`
10. Restaurant's subscription fires (manifest: `obligation.settled.v1` filtered by `kind == "hospitality_charge"` AND `creditor_engine == "restaurant"`); Workflow transitions table_session → `completed`; `restaurant.commission_earned.v1` emits for that night's waiter
11. Accounting recognizes restaurant revenue at this moment (correct tax timing per CN-5-001 N3 of CN-5-105 + CN-5-104 PC11)

**Doctrine demonstrated:** REST7 Hotel-Restaurant Obligation; CN-6-101 §11.4 + CN-6-104 §10 + WP3 concretized; VE2 + BD7 preserved (no direct hotel↔restaurant subscription); CN-6-005 future generalization.

### 18.4 WP4 — Split billing party-of-5 (by_share method)

**Time:** Wednesday lunch. Mzee Hassan and 4 colleagues at Lodge Serengeti restaurant for a business lunch. Bill comes; they want to split — but unevenly. Mzee Hassan, as host, will pay a 30% share; the four colleagues split the remaining 70% equally (17.5% each).

**Flow:**

1. Salma asks: "Mtagawanaje?" Mzee Hassan: "Mimi nilipe 30%; wenzangu wagawane 70% sawa-sawa."
2. Pack supports `by_share` method; Salma selects in her POS; enters share assignments
3. `restaurant.bill.split.applied.v1` emits:

```
{
  split_id,
  session_ref,
  split_method: by_share,
  split_lines: [
    {payer_party_ref: <hassan>, share_fraction: 0.30, amount: <30% of total>},
    {payer_party_ref: <colleague_1>, share_fraction: 0.175, amount: <17.5% of total>},
    {payer_party_ref: <colleague_2>, share_fraction: 0.175, amount: <17.5% of total>},
    {payer_party_ref: <colleague_3>, share_fraction: 0.175, amount: <17.5% of total>},
    {payer_party_ref: <colleague_4>, share_fraction: 0.175, amount: <17.5% of total>}
  ],
  business_date
}
```

4. `restaurant.bill.ready.v1` emits referencing the split structure
5. Universal Checkout opens 5 parallel tender flows. Each payer chooses their method (abstract per RE11). One uses mobile money push; two use card; one cash; one charges-to-room (Mzee Hassan happens to be staying at the lodge).
6. As each tender settles, the corresponding `checkout.settled.v1` fires; restaurant tracks per-payer settlement progress
7. When all 5 settlements complete, HO9 fires for the session → `restaurant.table_session.completed.v1`
8. `restaurant.commission_earned.v1` emits for Salma — tip calculation per pack rule (Lodge Serengeti uses `hybrid`; Salma's commission base is the session total regardless of split structure)

**Doctrine demonstrated:** REST5 split billing + Q2 single event with split_method enum; REST6 + Q4 tip pool; RE11 payment abstract (each payer uses different method, none named); HO9 dual-channel handoff.

### 18.5 WP5 — Mzee Karim BBQ pivot (Mixed-Vertical bucha + grill_to_table)

**Time:** Sunday afternoon. Mzee Karim's bucha in Kariakoo (established CN-6-001 WP5). He has expanded weekends — adding a nyama-choma corner where customers can order grilled meat (prepared on-site) to eat at outside benches or take away.

**Setup:**
- Mzee Karim's tenant now activates **both** `retail.*` (the bucha — raw meat sold by the kg per WP5 of CN-6-001) AND `restaurant.*` (the nyama-choma corner)
- Same tenant, two verticals — Mixed-Vertical pattern (CN-6-105 future generalization)
- Restaurant pack: `service_model: grill_to_table`, `ticket_routing: [grill]`, `session_model: table_session` (for sit-down on outside benches) OR `counter_order` (for takeaway), `menu_complexity: simple`, `alcohol: false`

**Flow (sit-down nyama choma):**

1. Customer arrives, orders 1 kg fillet grilled with ugali + kachumbari. Mzee Karim opens table_session for the outside bench; takes order
2. `restaurant.order.placed.v1` emits with menu items: `grilled_fillet_1kg` + `ugali_serving` + `kachumbari_side`
3. Routing: grilled_fillet → `restaurant.grill.ticket.fired.v1`; ugali + kachumbari → kitchen-prep ticket (Mzee Karim's wife handles)
4. Grill ticket: per recipe Pattern B (`pack.restaurant.recipes.grilled_fillet_1kg` declares: fillet 1 kg, charcoal portion, salt-pepper-rub portion, vegetable-oil 0.02 litre), Inventory deducts ingredients at `cooking_started`
5. Fillet IS sourced from Mzee Karim's bucha inventory — Inventory primitive handles the cross-vertical link transparently per CN-5-003 + RE7 of CN-6-001 (item-type agnosticism). The same kg of meat that arrived as part of the carcass for retail.* can now be consumed via Pattern B for restaurant.*. Inventory's lot tracking handles it natively.
6. Grilling + ugali ready; served at the outside bench
7. Bill emits via session.bill.request; settlement per customer's chosen tender method (RE11 abstract)
8. HO9 closes session

**Doctrine demonstrated:** REST0 cluster (BBQ = restaurant.* via pack); Mixed-Vertical (bucha retail.* + nyama-choma restaurant.* at same tenant); Inventory cross-vertical via Foundation primitive (REST4 + RE7 elegance); cross-doc character continuity.

### 18.6 WP6 — QR-table-ordering at Lodge Serengeti café-bar

**Time:** Mid-morning. Lodge Serengeti has a café-bar separate from the formal restaurant. Pack flags `pack.restaurant.self_service_enabled: true` + `pack.restaurant.qr_ordering_enabled: true` activate the QR flow.

**Customer:** A safari guest (Nina, cross-doc cameo from CN-6-001 WP6 — in this scenario she's visiting Serengeti for the season) sits at a café-bar table; scans the QR code on the table tent.

**Flow:**

1. Nina's phone opens the lodge's café-bar mobile interface (Term 3 surface)
2. Nina browses: chooses cappuccino + scone + fresh orange juice
3. Taps "Place Order"
4. `restaurant.qr_order.place.request` accepted; actor = Nina per CN-4-007 (customer-as-actor; D-DISC-002)
5. `restaurant.qr_order.placed.v1` emits
6. `restaurant.bill.ready.v1` also emits (payment-on-placement model; pack-configured)
7. Universal Checkout presents remote-eligible methods per `pack.tenant.payment_methods_subset` filtered by `pack.region.payment_methods_enabled` AND CTR-050 remote-eligibility. Nina chooses one of the methods; settles.
8. `checkout.settled.v1` → HO9 → `restaurant.qr_order.confirmed.v1`; café-bar staff sees the order on their kitchen display
9. Bar ticket (cappuccino + juice) fires to bar; kitchen ticket (scone) fires to pastry
10. When ready, a runner delivers to Nina's table; `restaurant.qr_order.served.v1` → `restaurant.qr_order.completed.v1`
11. Fan-out as normal

**Doctrine demonstrated:** REST8 self-service via QR-table; customer-as-actor; D-DISC-002 partial closure; payment-method abstract throughout (no provider names in narrative); cross-doc character continuity (Nina from retail mobile-app scenario reuses same customer-as-actor mechanism in restaurant context).

### 18.7 WP7 — Bibi Khadija mama ntilie (simplest F&B config)

**Time:** Lunch hour, Wednesday. Bibi Khadija (NEW Term 6 character) runs a mama-ntilie stall on a Kariakoo street corner. Charcoal stove; one large pot of pilau; another of meat stew; a basket of mandazi. Office workers come on lunch break, point at what they want, take away on banana-leaf-lined paper plates.

**Pack configuration:**
- `service_model: counter_service`
- `ticket_routing: [none]` (Bibi Khadija IS the cook + cashier; no separate station tickets)
- `session_model: takeaway_only`
- `menu_complexity: simple`
- `alcohol_licensing_required: false`
- `recipes`: minimal pack content — `pilau_serving` (one scoop), `meat_stew_serving` (one ladle), `mandazi_piece` (one piece). Pattern B fires at order completion for ingredient deduction

**Customer:** An office worker arrives, says: "Mama, naomba pilau + nyama na mandazi mawili."

**Flow:**

1. Bibi Khadija's POS (a simple smartphone running the BOS mobile cashier app — Term 3 surface) — she taps:
   - `pilau_serving` × 1
   - `meat_stew_serving` × 1
   - `mandazi_piece` × 2
2. `restaurant.counter_order.place.request` accepted; emits `restaurant.counter_order.placed.v1`
3. `restaurant.bill.ready.v1` emits immediately (counter_order pattern — bill ready at order placement)
4. Customer pays cash (per Bibi Khadija's enabled methods — regional curation per CTR-049; she has cash + mobile money push enabled); settlement
5. `checkout.settled.v1` → HO9 → `restaurant.counter_order.preparing.v1`
6. Bibi Khadija plates the food (no separate tickets — she's cooking the whole order herself); ingredient consumption events fire per recipe (Pattern B); Inventory deducts:
   - `restaurant.ingredient.consumed.v1` for pilau ingredients (rice, oil, spices)
   - `restaurant.ingredient.consumed.v1` for meat stew (meat, tomato, onion, spices)
   - `restaurant.ingredient.consumed.v1` for 2 mandazi (flour, sugar, yeast, oil portions)
7. She hands the plate to the customer; `restaurant.counter_order.handed_over.v1` → `restaurant.counter_order.completed.v1`

**Doctrine demonstrated:** REST0 cluster (mama ntilie = restaurant.* simplest config); preparation-step criterion (Bibi Khadija prepares ingredients on demand even if pre-cooked — plating + warming + portioning IS preparation); same engine as Lodge Serengeti fine dining; pack simplification absorbs the extreme; payment-method abstract (no provider named); REST4 recipe Pattern B works at the simplest scale.

**The bar:** if BOS can serve Bibi Khadija, it serves the majority of Tanzanian F&B SMEs. **The framework holds at the simplest case.**

### 18.8 WP8 — Mzee Hamisi pub (bar_tab + alcohol licensing pack flag)

**Time:** Friday 6 pm. Mzee Hamisi (NEW Term 6 character) opens his pub — Baa ya Mzee Hamisi — in Kariakoo. Regulars start drifting in. A group of 5 men sit at the bar; one orders the first round of beers.

**Pack configuration:**
- `service_model: bar_only`
- `ticket_routing: [bar]`
- `session_model: bar_tab`
- `menu_complexity: simple_bar_food` (beer + sodas + simple sambusa + grilled corn on weekends)
- `alcohol_licensing_required: true`

**Onboarding (already done at tenant activation):** Regional agent captured Mzee Hamisi's Liquor License document via `regulatory_evidence_refs` per CTR-045 expansion. License recorded as Foundation Document; expiry date tracked; manager advisor will alert as expiry approaches per Law 3.

**Flow:**

1. The first customer (a regular) opens a tab: Mzee Hamisi taps "Open Tab" with the customer's name (or `walk_in`) — `restaurant.bar_tab.open.request` → `restaurant.bar_tab.opened.v1`
2. First round: 5 beers. Mzee Hamisi: `restaurant.order.placed.v1` × 1 with 5-beer-line item → routes to `restaurant.bar.ticket.fired.v1`; he opens 5 bottles, serves
3. Recipe Pattern B fires: `restaurant.ingredient.consumed.v1` for 5 beers (each beer's "recipe" = 1 bottle from packaged-beer stock; Pattern A would also work for packaged beer — pack-driven choice; Mzee Hamisi's pack uses Pattern A here since there's no preparation, just dispensing)
4. Hours pass; the same tab accumulates more rounds; perhaps a plate of sambusa added; another beer for one specific drinker who's pacing differently
5. At 9 pm the group is ready to settle. One asks for the bill
6. Mzee Hamisi taps "Bill Tab"; tab total computed
7. The group elects to split by_share (equal_share for the 5); split applied via REST5
8. Each pays through their chosen method (RE11 abstract); 5 parallel settlements
9. When all settled, HO9 → `restaurant.bar_tab.completed.v1`
10. Excise tax on the beer portions computed per pack — Tanzania pack has excise rates for alcohol; Accounting includes the excise line per pack rules at Checkout K2 resolution
11. Manager advisor (CN-5-010) had run a check earlier in the day: Mzee Hamisi's license expires in 45 days. An alert is queued for him to renew with the licensing authority. He acts on it Monday morning — BOS doesn't autonomously block sales.

**Doctrine demonstrated:** REST2 bar_tab session model; REST5 split billing (equal_share); REST6 commission (per_waiter pool — Mzee Hamisi IS the waiter); alcohol licensing as pack flag (NOT vertical-defining); §16 BOS-is-not-licensing-authority doctrine (license stored, alert surfaced, decision human); excise tax via pack rules; payment-method abstract throughout.

---

## 19. Boundaries + Open Items + Cross-Term Hooks

### 19.1 CN-6-002's place in the corpus

| Concern | Owned by | CN-6-002 role |
|---------|----------|----------------|
| Restaurant / F&B cluster engine declaration | **CN-6-002** (this doc) | Authoritative |
| Cross-cutting framework | CN-6-100..104 | Parents — CN-6-002 applies |
| Universal Checkout consumption | CN-5-009 | Consumer |
| Universal Promotion (multi-price + loyalty) | CN-5-007 | Consumer (inherited from CN-6-001 RE3) |
| Universal Inventory Pattern B (recipes) | CN-5-003 | Consumer; pack content drives consumption |
| Universal Accounting | CN-5-001 | Consumer via CTR-030; includes excise per alcohol |
| Universal HR/Payroll (tips/commission) | CN-5-005 | Consumer via REST6 |
| Universal Tax-Aware Engines | CN-5-105 | Consumer; alcohol excise; tenant_tax_profile gate |
| Retail (CN-6-001) | Sibling | REST0 preparation-step boundary applied; payment abstraction inherited; Mixed-Vertical for bucha-BBQ (WP5) |
| Hotel (CN-6-003 future) | Sibling | REST7 cross-vertical Obligation; Hotel-side subscription confirmation |
| Vertical Bridges (CN-6-005 future) | Sibling | REST7 concrete pattern; catalogue future |
| Mixed-Vertical Tenants (CN-6-105 future) | Sibling | WP5 (Mzee Karim) + WP3 (Lodge Serengeti hotel + restaurant) demonstrate; CN-6-105 generalizes |
| Future-vertical stress sketches | CN-6-901..904 + CN-6-905 | Apply REST0 + REST1..REST8 patterns |

### 19.2 CTRs (no new)

- CTR-018, CTR-002, CTR-024, CTR-026, CTR-030, CTR-038, CTR-044, CTR-045, CTR-046 — cited as-is
- CTR-049, CTR-050 — inherited from CN-6-001; CN-6-002 uses
- CTR-028, CTR-006, CTR-027 — universal payment + site registries
- **Cumulative CTR-046 expansion queue (unchanged):** DC-NN-e + DC-NN-f

**No new CTRs.** F&B cluster fits existing contracts.

### 19.3 Open items inside Term 6 scope

- **CN-6-003 Hotel** — confirms Hotel-side Obligation subscription for REST7; reservation lifecycle; folio aggregation; chain guest profile
- **CN-6-004 Workshop** — parallels REST4 recipe-as-pack-content with parametric formulas-as-pack-content
- **CN-6-005 Bridges** — catalogs REST7 concrete pattern + other cross-vertical instances
- **CN-6-105 Mixed-Vertical Tenants** — generalises WP5 (Mzee Karim bucha + nyama choma) + WP3 (Lodge Serengeti hotel + restaurant)
- **Logistics-Restaurant catering integration** — when future Logistics vertical lands, catering delivery becomes Logistics + Restaurant via Obligation primitive (parallel REST7)

### 19.4 D-DISC cross-references

- **D-DISC-001 — tenant-customer promotion UX:** WP1 customer identification + WP3 charge-to-room + WP6 QR ordering all touch customer-facing surfaces; Term 3 designs UX on top of restaurant emissions
- **D-DISC-002 — POS self-service expansion:** REST8 + WP6 partially close for F&B; QR-table-ordering operational; in-store self-service kiosk variant supported via same Workflow with `pack.restaurant.service_model: counter_service`

### 19.5 The bar — framework holds again

CN-6-002 written without amending CN-6-100..104. REST0 + REST1-REST8 derive from cross-cutting framework + cluster doctrine (BD5 + BD6) + preparation-step criterion (REST0). Eight worked patterns demonstrate F&B spectrum from Bibi Khadija's mama-ntilie corner to Lodge Serengeti fine dining to Mzee Hamisi's pub to Mzee Karim's bucha-BBQ pivot — **all the same engine, all different pack configuration**.

If the framework can serve Bibi Khadija on a Kariakoo street corner and a fine-dining safari lodge with equal correctness, it can serve the majority of Tanzania's F&B SMEs without architectural exceptions. **Framework v1 holds on its second concrete test.**

Next: CN-6-003 Hotel. Kilimanjaro Lodge Moshi + Lodge Serengeti chain. Multi-day reservations. Folio aggregation. Hotel-side confirmation of REST7 charge-to-room pattern. Third concrete test.

---

*— End of CN-6-002 Restaurant Engine (F&B Cluster) v1 —*
