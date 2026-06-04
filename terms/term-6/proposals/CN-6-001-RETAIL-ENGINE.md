# CN-6-001 — Retail Engine

> **Term:** 6 — Vertical Engines
> **Status:** Concept Phase v1.0 — Draft for Overseer review
> **Branch:** `claude/brave-johnson-S7Lky`
> **Reading order:** CN-6-100/101/102/103/104 (Term 6 cross-cutting cluster) → CN-5-009/100/103/105 → CN-4-011/021 → **this doc**
> **Ordering authority:** TERM-6 Brief §13 — **first concrete vertical doc**; applies and tests the cross-cutting framework on its simplest baseline case.

---

## 1. Purpose & Boundary

### 1.1 Mission

CN-6-001 declares the **Retail Engine** — the first concrete vertical built on the Term 6 cross-cutting framework (CN-6-100..104). It covers the full basket-POS spectrum (per CN-6-101 §11.1.1): from Mama Amina's Kariakoo duka to a Nakumatt supermarket in Mlimani City, from a kiosk to a hypermarket, from a single till to multi-till operations, from in-store cashier-attended to remote mobile-ordering. All share one engine — different in scale and configuration, same in business doctrine.

Per Brief §6.1: *"Baskets, multi-price, refunds, item-level discounts."* This doc operationalises that, plus the realities Concept Lead surfaced: bulk-splittable items (rice by the kilo, nyama from Mzee Karim's bucha by the kilo), remote ordering via tenant mobile apps, and payment-method abstraction across the regional spectrum.

### 1.2 DOES vs DOES NOT

| CN-6-001 DOES | CN-6-001 DOES NOT |
|----------------|--------------------|
| Declare the `retail.*` vertical engine per CN-6-100 §3 Recipe | Author the cross-cutting framework (CN-6-100..104 already complete) |
| Specify retail-specific Workflows (`retail.sale`, `retail.fulfillment`, `retail.catalog`) | Specify Universal Checkout flow (CN-5-009) — retail emits, Checkout settles |
| Cover full basket-POS spectrum: kiosk → duka → mini-mart → supermarket → hypermarket | Specialize by item type — retail handles any item sold via basket POS (RE7) |
| Support bulk-splittable items (kg, litre, m) per RE9 — duka + bucha + mafuta reality | Specify physical scale/measurement device integration (Term 7 + Architect phase) |
| Support remote ordering via tenant mobile app per RE10 — `retail.fulfillment` Workflow | Specify the mobile UX surface (Term 3) or the channel adapters (Term 7) |
| Abstract payment method per RE11 — Universal Checkout + CTR-028 + regional curation | Hardcode any payment provider name (M-Pesa, Tigo, card brand, bank) — methods are regional-agent-curated + tenant-configured |
| Define multi-price layer resolution (pack-driven; vertical INDICATES) | Resolve final prices — Checkout K2 + Promotion PR1 are authoritative |
| Specify refund + void + UI-02 compensation patterns | Reverse events (Law 1 — compensating events, never deletes) |
| Honour B2C default + B2B via Party metadata (no Party primitive extension) | Open new Foundation CTRs — uses CN-4-011 Party flexibility as-is |
| Open CTR-049 (regional curation → Term 2) + CTR-050 (adapter coverage → Term 7) per Overseer | Wait for those CTRs to close — CN-6-001 v1 references; downstream Terms resolve |

### 1.3 Audience

Term 6 itself (CN-6-002..004 authors copy this template); Architects implementing the retail engine; Term 1 onboarding governance authoring retail-side catalog entries; Term 3 designing the cashier and mobile-app UX; Term 7 wiring payment adapters per CTR-050; future SME tenants of any size and any item mix.

### 1.4 Charter Compliance

| Law | How CN-6-001 honours it |
|-----|--------------------------|
| Law 1 — State from events only | All retail state via event store; refunds are compensating events; voids are explicit events (RE5) |
| Law 2 — Engines isolated | Retail subscribes only to Foundation primitives + Universal events (HO8); cross-vertical via Obligation primitive |
| Law 3 — AI advisory only | retail-floor and retail-manager advisors per CN-5-010 wiring; never autonomous |
| Law 4 — Flexibility first-class | Full POS spectrum + bulk-splittable + remote ordering accommodated without architectural change |
| Law 5 — Compliance configured | Tax via pack lookups (CN-5-105); item categories pack-driven (Q3); payment methods pack-curated (RE11) |
| Law 6 — Distribution regional | RE11 + CTR-049 — regional agent curates payment methods per region; **never global provider preference** |

### 1.5 Parsimony — retail is where most BOS tenants begin

Mama Amina opens her Kariakoo duka. Salma — her daughter, 19, finishing form-six — works the till. They scan rice by the kilo, soap by the bar, paracetamol by the strip. Customers pay in cash, in mobile money push, sometimes on credit. At day's close, Mama Amina sees the day's numbers on her phone — revenue, what sold, what's running low, what she owes Bibi Mariam at the cooperative for tomorrow's restock.

This is where most BOS tenants begin. The retail vertical must serve Mama Amina's duka exactly as well as it serves the Nakumatt supermarket in Mlimani City — same engine, same doctrine, different pack configuration. If Salma can't trust the till at 7am, the framework has failed.

Across the spectrum, the same parsimony bar holds: **doctrine should match how the real cashier at the real till in real Tanzania thinks**. The eleven RE doctrines below are justified against that.

---

## 2. Inputs and Relationship

### 2.1 Cross-cutting framework (parent)

- **CN-6-100** — 12-step Recipe; manifest delta; VE1–VE7
- **CN-6-101** — BD1–BD8; retail spectrum §11.1.1; BD5 split-it (pricing, promotion, tax)
- **CN-6-102** — NC1–NC9 naming; bill.ready uniformity; conflict events; compensation pair manifest declaration
- **CN-6-103** — SP1–SP8 scope policy; site-default; tenant-scope catalog (retail catalog row); no platform
- **CN-6-104** — HO1–HO9 handoff; bill.ready → Universal Checkout + HO9 settlement-back; failure modes; payment-method abstraction at HO2

### 2.2 Universal layer

- **CN-5-009** Universal Checkout — K1/K2 idempotency; tender flow; CTR-028 method registry (RE11)
- **CN-5-001** Accounting — CTR-030 journal sufficiency
- **CN-5-002** Cash — indirect (subscribes to checkout.settled per HO catalog row)
- **CN-5-003** Inventory — Pattern A (auto from Checkout) + Pattern B (vertical-managed consumption per item category)
- **CN-5-007** Promotion — Multi-price resolution (RE3); cross-vertical loyalty per Party primitive
- **CN-5-105** Tax-treatment per item via pack lookup; tenant_tax_profile gate per N7
- **CN-5-010** AI advisors — retail-floor + retail-manager advisors (Law 3)

### 2.3 Foundation

- **CN-4-011** Workflow + Party + Document + Obligation + Inventory Movement primitives
- **CN-4-021** Saleable Line + Tender value shapes; `discount_refs` + `tax_treatment_ref`
- **CN-4-022** Advisor framework — retail advisors plug in

### 2.4 Brief grounding

- **Brief §6.1** retail concept summary
- **Brief §7.1** five retail edge cases (returns, multi-price layers, voided sales, retail↔workshop, B2C vs B2B)
- **Brief §11.7 closure** — Pharmacy is sibling vertical; retail handles OTC (RE7)
- **Brief §11.9** — Salon / car wash / light services are NOT retail; deferred to CN-6-905

### 2.5 CTRs

- **No new CTRs from retail mechanics** in CN-6-001 itself; uses CTR-018/002/024/026/030/038/044/045/046 as-is
- **CTR-049** (Term 6 → Term 2) regional payment-method curation — opened with this work cycle per `99725fd`
- **CTR-050** (Term 6 → Term 7) payment adapter coverage including remote-push and QR-presented — opened with this work cycle per `99725fd`
- **CTR-028** (existing, Term 7 + Term 1) — abstract tender-method registry; retail consumes
- **CTR-006** (existing, Term 7) — payment adapter contract; CTR-050 extends

---

## 3. Retail Doctrine — RE1–RE11

**RE1 — `retail.*` covers the full basket-POS spectrum.** Kiosk, duka, mini-mart, supermarket, hypermarket, department store, pharmacy OTC counter, bucha (butchery), mafuta station (within retail scope), specialty shop. Pack hooks differentiate scale and configuration; doctrine is one. Per CN-6-101 §11.1.1.

**RE2 — `retail.sale` is the single canonical in-store Workflow.** Lifecycle: `basket_opened → items_added → checkout_initiated → bill_ready → completed → archived` (success path); `voided` (terminal-abandonment, pre-bill); `refunded` (terminal-compensation, post-completed). State name `completed` is used in the Workflow vocabulary to avoid semantic collision with the `checkout.settled.v1` universal event that triggers the HO9 transition.

**RE3 — Multi-price resolution is pack-driven layer order; vertical INDICATES, Checkout RESOLVES.** Pack hook `pack.retail.pricing.layer_order` declares sequence (default: `[base_price, branch_override, active_promo, loyalty, customer_specific]`). Vertical's `discount_refs[]` carries layer **input refs**; Checkout K2 + Promotion PR1 do the **authoritative resolution**. Vertical never computes final price.

**RE4 — Refunds use UI-02 compensation per G5.** `retail.sale.completed.v1` declares `compensation_pair: retail.sale.refunded.v1`. The refund event itself declares `compensation_basis_none: "UI-02 closure event itself"`. Pack-driven refund window via `pack.retail.refund.window_days` (default 30).

**RE5 — Voided baskets emit events.** Pre-bill abandonment is not silent. `retail.basket.voided.v1` emits with reason (`customer_changed_mind`, `idle_timeout`, `system_void`); compensation pair of `retail.basket.opened.v1`. Per Brief §7.1 observability mandate. Reporting projections distinguish voided from completed for conversion-rate KPIs.

**RE6 — B2C is default; B2B via Party metadata.** Party primitive (CN-4-011) carries `party_role: customer`, `customer_type: b2c | b2b | tradesman`, and optional `negotiated_pricing_tier_ref` pointing to a tenant-scope tier registry. No separate B2B Workflow; same `retail.sale` lifecycle with different pricing-tier input. Avoids Party primitive extension (no Term 4 CTR).

**RE7 — Retail handles any item type via item primitive category metadata.** Per CN-6-101 §11.1.1 — retail is *mechanism*-specialised (basket POS), not type-specialised. `retail.catalog.entry` references item primitive (CN-4-011) instances; item categorisation (pack-driven taxonomy per Q3) drives tax treatment, inventory pattern, and pricing eligibility per item. Rice, sugar, oil, soap, paracetamol (OTC), clothing, hardware, electronics, sundries, prepared meals (bucha-cut nyama) — all retail items. The vertical is item-type agnostic.

**RE8 — Basket idle timeout pre-checkout per pack rule.** Carts abandoned mid-build trigger `retail.basket.timed_out.v1` (compensation pair of `basket.opened.v1`) → Workflow → `voided` terminal. Pack hook `pack.retail.basket.idle_timeout_minutes` (default 30 — covers a Salma stepping away or a customer reconsidering). Distinct from CN-6-104 N3 post-bill.ready timeout (which is 0h for retail per the canonical POS rule).

**RE9 — Bulk-splittable items priced by measurement unit.** Rice (kg), cooking oil (litre), maize flour (kg), nyama (kg, from Mzee Karim's bucha), fabric (m), salt (kg) — items where the customer says "kilo mbili za fillet" or "lita moja ya mafuta," the cashier cuts/measures, and price = unit_price × quantity_measure. Pack hooks: `pack.retail.bulk_splittable_categories` (which item categories are bulk-splittable) and `pack.retail.measurement_units` (catalog of supported units: kg, g, litre, ml, m, cm — pack-extensible per region). Inventory tracks in measurement units; refunds compute in measurement units; tax computes per pack rule per measured quantity.

**RE10 — Remote ordering via tenant mobile app uses the `retail.fulfillment` Workflow.** Customer-initiated orders outside the till: customer scans tenant's Business ID / QR (Term 1 onboarding governance + CTR-049), browses catalog, places order, settles via remote tender (RE11). Tenant fulfils via pickup or delivery. Workflow lifecycle: `placed → confirmed → preparing → ready_for_handover → handed_over → completed → archived` (success path); plus `cancelled`, `refunded`, `failed` terminals. Distinct from `retail.sale` (cashier-at-till model) but emits the same `retail.bill.ready.v1` to Universal Checkout — only the fulfilment lifecycle differs. Delivery integration with future Logistics vertical is deferred (pack hook `pack.retail.fulfillment.delivery_enabled` flags readiness).

**RE11 — Retail NEVER specifies payment method.** Universal Checkout (CN-5-009) handles tender per CTR-028 abstract registry. Regional agent (per Charter Law 6 + D-003 + CTR-049) curates the enabled payment method set per region (Tanzania has mobile money providers + card networks + bank transfer + cash; Kenya, Uganda, Nigeria, Ghana each have their own region-specific provider set). Tenant (per CTR-045 + CTR-049 tenant configuration) selects subset within the regional set. Customer chooses from tenant's enabled set at point of tender. **Doctrine is method-agnostic; specific provider names belong to pack content (regional + tenant), NEVER to vertical doctrine or worked-pattern narratives.**

---

## 4. The Manifest — Engine Declaration

```yaml
engine_id: retail
engine_kind: vertical
namespace_root: retail
multi_site_capable: false                          # SP1 site-default; multi-tenant chains use bundling per CN-6-103 WP2 (not multi-site-by-nature)
scope_policy: site                                 # SP1 default

workflow_instances:
  - workflow_id: retail.sale
    billable: true
    lifecycle_states: [basket_opened, items_added, checkout_initiated, bill_ready, completed, voided, refunded, archived]
    settlement_subscription:                       # HO9 mandatory
      event_type: checkout.settled.v1
      filter: originating_workflow_ref == <self>
      transitions_to: completed

  - workflow_id: retail.fulfillment                # RE10
    billable: true
    lifecycle_states: [placed, confirmed, preparing, ready_for_handover, handed_over, completed, cancelled, refunded, failed, archived]
    settlement_subscription:                       # HO9 mandatory
      event_type: checkout.settled.v1
      filter: originating_workflow_ref == <self>
      transitions_to: confirmed                    # settlement triggers confirmation; preparation begins after

  - workflow_id: retail.catalog                    # SP3 tenant-scope catalog management
    billable: false
    lifecycle_states: [draft, active, deprecated, archived]
    scope_ref: tenant

commands:
  # retail.sale lifecycle
  - retail.basket.open.request
  - retail.basket.item.add.request                 # supports quantity_measure for RE9 bulk-splittable
  - retail.basket.item.remove.request
  - retail.basket.checkout.request
  - retail.basket.void.request
  - retail.sale.refund.request

  # retail.fulfillment lifecycle (RE10)
  - retail.fulfillment.place.request               # customer remote action; actor = customer per CN-4-007
  - retail.fulfillment.confirm.request             # tenant action
  - retail.fulfillment.prepare.request
  - retail.fulfillment.mark_ready.request
  - retail.fulfillment.handover.request
  - retail.fulfillment.cancel.request

  # retail.catalog lifecycle (SP3 tenant-scope)
  - retail.catalog.entry.add.request
    scope_ref: tenant
  - retail.catalog.entry.update.request
    scope_ref: tenant
  - retail.catalog.entry.deprecate.request
    scope_ref: tenant

emits:
  # retail.sale events
  - event_type: retail.basket.opened.v1
    compensation_pair: retail.basket.voided.v1
  - event_type: retail.basket.item.added.v1
    compensation_pair: retail.basket.item.removed.v1
  - event_type: retail.basket.item.removed.v1
    compensation_basis_none: "removal IS the compensation; no further compensation"
  - event_type: retail.basket.voided.v1
    compensation_basis_none: "voiding IS terminal compensation"
  - event_type: retail.basket.timed_out.v1         # RE8
    compensation_basis_none: "timeout IS terminal compensation"
  - event_type: retail.bill.ready.v1               # NC3 + VE4 + HO1 canonical
    compensation_pair: retail.bill.recalled.v1
  - event_type: retail.sale.completed.v1
    compensation_pair: retail.sale.refunded.v1
  - event_type: retail.sale.refunded.v1
    compensation_basis_none: "UI-02 closure event itself"

  # retail.fulfillment events (RE10)
  - event_type: retail.fulfillment.placed.v1
    compensation_pair: retail.fulfillment.cancelled.v1
  - event_type: retail.fulfillment.confirmed.v1
    compensation_pair: retail.fulfillment.cancelled.v1
  - event_type: retail.fulfillment.preparing.v1
    compensation_basis_none: "preparation is observable physical work; cancellation = separate compensation event"
  - event_type: retail.fulfillment.ready_for_handover.v1
    compensation_basis_none: "readiness is observation; cancellation = separate event"
  - event_type: retail.fulfillment.handed_over.v1
    compensation_pair: retail.fulfillment.return_initiated.v1
  - event_type: retail.fulfillment.completed.v1
    compensation_pair: retail.fulfillment.refunded.v1
  - event_type: retail.fulfillment.cancelled.v1
    compensation_basis_none: "cancellation IS terminal compensation"
  - event_type: retail.fulfillment.failed.v1
    compensation_basis_none: "failure IS terminal compensation"

  # retail.catalog events (tenant-scope per SP3)
  - event_type: retail.catalog.entry.added.v1
    compensation_pair: retail.catalog.entry.removed.v1
    scope_ref: tenant
  - event_type: retail.catalog.entry.updated.v1
    compensation_pair: retail.catalog.entry.reverted.v1
    scope_ref: tenant
  - event_type: retail.catalog.entry.deprecated.v1
    compensation_basis_none: "deprecation is forward-only state change"
    scope_ref: tenant

  # customer interaction
  - event_type: retail.customer.identified.v1
    compensation_basis_none: "identification is observation; party_ref carries identity"

subscribes_to:
  - event_type: checkout.settled.v1
    scope_ref: site
    # HO9 mandatory settlement-back; routes to retail.sale OR retail.fulfillment per originating_workflow_ref
  - event_type: inventory.stock.depleted.v1
    scope_ref: site
    # HO6 situational; handler per pack.retail.basket.depleted_item_handling
  - event_type: promotion.rule.applied.v1
    scope_ref: site
    # awareness for retail UI (Term 3 surface); read-only
  - event_type: pack.effective.v1
    scope_ref: tenant
    # pack version pin per D-009 freeze

pack_hooks:
  # configuration spectrum (RE1)
  - pack.retail.multi_till                         # bool — supermarket / hypermarket
  - pack.retail.barcode_required                   # bool — supermarket; false for duka
  - pack.retail.aisle_layout                       # taxonomy for category navigation
  - pack.retail.self_service_enabled               # D-DISC-002 placeholder
  - pack.retail.item_categories                    # pack-driven taxonomy tree (Q3)

  # pricing layer (RE3)
  - pack.retail.pricing.layer_order                # default [base_price, branch_override, active_promo, loyalty, customer_specific]
  - pack.retail.pricing.combination_rules          # multiplicative vs additive per layer

  # basket lifecycle (RE5 + RE8)
  - pack.retail.basket.idle_timeout_minutes        # RE8; default 30
  - pack.retail.basket.depleted_item_handling      # Q6; {block_add | warn_at_checkout | auto_remove_at_bill_ready}
  - pack.retail.refund.window_days                 # RE4; default 30

  # bulk-splittable (RE9)
  - pack.retail.bulk_splittable_categories         # which item categories are bulk-splittable
  - pack.retail.measurement_units                  # {kg, g, litre, ml, m, cm, ...}; pack-extensible per region
  - pack.retail.measurement.minimum_increment      # per measurement unit (e.g., kg → 0.05 = 50g minimum)

  # inventory pattern (Pattern A vs B per item category)
  - pack.retail.inventory_expansion_mode           # {item_category: auto | vertical_managed}

  # fulfillment (RE10)
  - pack.retail.fulfillment.delivery_enabled       # future Logistics integration
  - pack.retail.fulfillment.pickup_window_hours    # how long ready-for-handover holds before timeout
  - pack.retail.fulfillment.remote_ordering_business_id_format  # CTR-049 + Term 1 onboarding

  # payment method abstraction (RE11)
  # NB: actual payment method curation lives in REGION-pack and TENANT-pack per CTR-049, NOT in retail vertical pack:
  #   pack.region.payment_methods_enabled — Term 2 regional agent curation per CTR-049
  #   pack.tenant.payment_methods_subset — tenant configuration within regional set per CTR-049
  # retail.bill.ready.v1 emission carries NO method hint; Checkout adjudicates per CTR-028 registry.

advisor_ids:
  - retail-floor-advisor                           # per CN-5-010; suggests restock, slow-movers, basket-completion patterns
  - retail-manager-advisor                         # per CN-5-010; KPI explanations, daily/weekly performance
```

The manifest is the **canonical declaration**. Every CN-6-001 worked pattern below resolves against it.

---

## 5. Workflow Lifecycle — `retail.sale` State Machine

### 5.1 The states + transitions

```
                       command: basket.open.request
                                ▼
                       ┌────────────────┐
                       │ basket_opened  │
                       └────────────────┘
                                │  command: basket.item.add.request (+/-)
                                ▼
                       ┌────────────────┐
                       │  items_added   │ ◄── (recurring; multiple add/remove cycles)
                       └────────────────┘
                                │  command: basket.checkout.request
                                ▼
                       ┌────────────────────────┐
                       │  checkout_initiated    │
                       └────────────────────────┘
                                │  (vertical-internal validation; tax_treatment_ref lookup; discount_refs assembly)
                                ▼
                       ┌────────────────┐
                       │   bill_ready   │ ── emit retail.bill.ready.v1 → Universal Checkout (HO1)
                       └────────────────┘
                                │
                                │ HO9: checkout.settled.v1 with matching originating_workflow_ref
                                ▼
                       ┌────────────────┐
                       │   completed    │ ── emit retail.sale.completed.v1
                       └────────────────┘
                                │  retention window per pack
                                ▼
                       ┌────────────────┐
                       │    archived    │
                       └────────────────┘

Compensation / terminal branches:
    items_added or checkout_initiated → voided (command: basket.void.request OR RE8 timeout)
    completed → refunded (command: sale.refund.request within pack window)
```

### 5.2 Per-transition guards

| Transition | Guard |
|------------|-------|
| basket_opened → items_added | First successful `basket.item.add` accepted |
| items_added → checkout_initiated | Cashier (or customer, if self-service per pack) commits basket |
| checkout_initiated → bill_ready | All saleable_lines have valid tax_treatment_ref; site_id resolves UI-09 |
| bill_ready → completed | HO9 settlement-back subscription fires with matching `originating_workflow_ref` |
| any pre-bill_ready → voided | RE8 timeout OR explicit void command OR Inventory depletion per pack rule |
| completed → refunded | Refund command within `pack.retail.refund.window_days` |

### 5.3 Concurrency

Per CN-6-103 Q7 closure — each till runs its own `retail.sale` Workflow instance. Multi-till supermarkets have multiple concurrent instances; bus single-acceptance (CN-4-004) handles inventory conflicts at the line-add boundary (per Q6 `pack.retail.basket.depleted_item_handling`).

---

## 6. Workflow Lifecycle — `retail.fulfillment` State Machine (RE10)

### 6.1 The states + transitions

```
                       command: fulfillment.place.request (actor: customer per CN-4-007)
                                ▼
                       ┌────────────────┐
                       │     placed     │ ── emit retail.fulfillment.placed.v1
                       └────────────────┘
                                │  HO9: checkout.settled.v1 (customer settles remotely at placement time)
                                ▼
                       ┌────────────────┐
                       │   confirmed    │ ── emit retail.fulfillment.confirmed.v1; tenant notification
                       └────────────────┘
                                │  command: fulfillment.prepare.request (tenant action)
                                ▼
                       ┌────────────────┐
                       │   preparing    │ ── emit retail.fulfillment.preparing.v1
                       └────────────────┘
                                │  command: fulfillment.mark_ready.request
                                ▼
                       ┌──────────────────────────┐
                       │  ready_for_handover      │
                       └──────────────────────────┘
                                │  command: fulfillment.handover.request (customer arrives OR delivery)
                                ▼
                       ┌────────────────┐
                       │  handed_over   │
                       └────────────────┘
                                │  internal confirmation
                                ▼
                       ┌────────────────┐
                       │   completed    │
                       └────────────────┘
                                │
                                ▼
                       ┌────────────────┐
                       │    archived    │
                       └────────────────┘

Branches:
    placed or confirmed → cancelled (customer cancellation pre-prepare OR tenant rejection)
    ready_for_handover beyond pack.retail.fulfillment.pickup_window_hours → failed
    handed_over → refunded (UI-02 compensation; pack window)
```

### 6.2 Key differences from `retail.sale`

| Aspect | retail.sale (in-store) | retail.fulfillment (remote) |
|--------|------------------------|------------------------------|
| Trigger actor | Cashier-at-till (Salma) | Customer remotely via mobile app |
| Settlement timing | Post-checkout (CN-6-104 N3 = 0h) | Pre-confirmation (customer pays at order placement; tenant confirms after) |
| Identity model | Cashier-as-actor; party_ref optional | Customer-as-actor per CN-4-007; party_ref mandatory (mobile app identifies) |
| Workflow length | Seconds-to-minutes | Minutes-to-hours (preparation + handover window) |
| Payment method | RE11 — abstract per regional + tenant pack | RE11 — abstract per regional + tenant pack; **remote-eligible subset only** (CTR-050) |
| Fulfilment | Immediate (customer leaves with goods) | Pickup or delivery; delivery deferred to Logistics-vertical integration |

### 6.3 Identity at remote ordering (CN-4-007 customer-as-actor)

Per CN-4-007 + D-DISC-002 self-service expansion, customer is the actor on `retail.fulfillment.place.request`. The customer's Party primitive instance (registered via tenant's mobile app onboarding flow) is the `actor` field on the event. This is identical to self-service in-store except the channel is remote.

---

## 7. Multi-Price Layer Resolution (RE3)

### 7.1 The five layers + pack-driven order

Pack hook `pack.retail.pricing.layer_order` declares sequence (default below). Pack hook `pack.retail.pricing.combination_rules` declares per-layer stacking semantics (multiplicative vs additive).

| Layer | Source | What it adjusts | Default order |
|-------|--------|------------------|---------------|
| 1. base_price | `retail.catalog.entry` per-item baseline | Starting line price | 1st |
| 2. branch_override | Site-scope override on top of catalog (per tenant configuration) | Branch-specific deviation | 2nd |
| 3. active_promo | `promotion.rule.applied.v1` per CN-5-007 | Time-bounded promotions (happy-hour, end-of-day clearance, Friday-duka day) | 3rd |
| 4. loyalty | `promotion.loyalty.redeemed.v1` per CN-5-007 | Customer redemption against accumulated balance | 4th |
| 5. customer_specific | Per-Party `negotiated_pricing_tier_ref` per RE6 | B2B tradesman rate, NGO discount | 5th |

### 7.2 The vertical's role — INDICATES only

The vertical emits `retail.bill.ready.v1` with `saleable_lines[]` where each line carries:

- `unit_price` — the **base catalog price** (pre-tax, pre-discount); never the resolved price
- `discount_refs[]` — array of layer **input refs**:
  - `{layer: branch_override, override_ref: <override_id>}` (if site has override)
  - `{layer: active_promo, promotion_ref: <promo_event_id>}` (if Promotion already emitted a rule)
  - `{layer: loyalty, intent: redeem, max_points: <n>}` (if customer redeems)
  - `{layer: customer_specific, tier_ref: <tier_id>}` (if B2B/negotiated)
- `tax_treatment_ref` — pack lookup result per item category
- `line_total` — non-authoritative cache; Checkout K2 recomputes

### 7.3 The Checkout's role — RESOLVES authoritatively

Universal Checkout K2 + Promotion PR1 walk `pack.retail.pricing.layer_order`, fetch each layer's input, apply per `pack.retail.pricing.combination_rules`, compute final unit_price, then apply tax per `tax_treatment_ref` lookup. The vertical does not know the final number.

### 7.4 Worked example — soap at Mama Amina's

```
Mama Amina sells washing soap. Mama Halima (regular, with loyalty balance) buys 3 bars on Friday.

Pack: pack.retail.pricing.layer_order = [base_price, branch_override, active_promo, loyalty, customer_specific]
Pack: pack.retail.pricing.combination_rules = "multiplicative for percentages; additive for absolute"

Catalog base_price (washing soap): TZS 2,000 per bar
Branch override (Mama Amina's Kariakoo): none
Active promo (Friday-duka 10% off all soap): -10% per bar
Loyalty (Mama Halima redeems 150 points = TZS 75 per bar): -75 TZS per bar
Customer-specific (Mama Halima is B2C, no tier): none

Vertical emits retail.bill.ready.v1:
  saleable_lines[0] = {
    line_id, item_ref: soap-001, quantity: 3, unit_price: 2000,
    discount_refs: [
      {layer: active_promo, promotion_ref: <friday-duka>},
      {layer: loyalty, intent: redeem, max_points: 450}
    ],
    tax_treatment_ref: <pack-lookup: vat-standard-or-exempt-per-jurisdiction>,
    source_tag: "till-1"
  }

Checkout K2 + Promotion PR1 resolve:
  After active_promo (-10%): 1,800 per bar
  After loyalty (-75 per bar via 150 pts/bar): 1,725 per bar
  Final pre-tax: 1,725 × 3 = 5,175 TZS
  Plus pack-determined tax (if VAT-registered tenant) OR zero (if non-registered per CN-5-105 N7)

Final line total: Checkout-authoritative.
```

The vertical never claims to know "5,175 TZS." It indicates the inputs; Checkout resolves.

---

## 8. Basket Operations + Voided Baskets + Idle Timeout (RE5 + RE8)

### 8.1 The basket lifecycle

The basket is the early portion of `retail.sale` Workflow (states `basket_opened` and `items_added`). Salma opens a basket when the customer arrives; adds items as they're scanned/keyed; commits with `checkout_initiated`.

### 8.2 Voided baskets (RE5)

Pre-bill abandonment emits an explicit event:

```
retail.basket.voided.v1 {
  basket_id,
  site_id,
  reason: customer_changed_mind | idle_timeout | system_void | cashier_void,
  items_added_count,
  actor: <cashier_or_system>,
  voided_at
}
```

Reasons (open enumeration per pack):
- `customer_changed_mind` — explicit cashier-marked
- `idle_timeout` — RE8 timeout (default 30 min)
- `system_void` — manager override during cashier shift change
- `cashier_void` — cashier discretion

Reporting projections distinguish voided from completed for conversion-rate KPIs (per CN-5-006 + Q5 closure).

### 8.3 Idle timeout (RE8)

```
pack.retail.basket.idle_timeout_minutes = 30   (default)
```

If no `basket.item.add` or `basket.checkout.request` command arrives within the window, scheduled action emits `retail.basket.timed_out.v1` (compensation pair of `basket.opened.v1`) → Workflow → `voided` terminal. Per CN-6-100 §3.13 abandonment pattern.

### 8.4 Why explicit void events matter

Mama Amina's daily Reporting projection shows:

- "Sales today: 47 (TZS 312,500)"
- "Voided baskets: 8 (avg 4 items each, abandoned at ~TZS 11,200 each)"

The voided count is **business intelligence**. Eight voided baskets in a day might mean Salma needs help during the morning rush (per retail-floor-advisor). Silent voids would hide this signal.

---

## 9. Bulk-Splittable Items (RE9) + Measurement-Unit Pricing

### 9.1 The reality

A Kariakoo duka stocks rice in 50kg sacks; customers buy by the half-kilo, kilo, or two-kilos. Cooking oil arrives in 20-litre containers; customers buy by the litre or half-litre. Mzee Karim's bucha receives full carcasses; customers say "kilo mbili za fillet" or "robo kilo ya minofu." Fabric merchants sell by the metre. These items are **not discrete units**.

### 9.2 Pack hooks

```
pack.retail.bulk_splittable_categories = ["grain", "oil", "meat", "spice", "fabric", "liquid_bulk", ...]
pack.retail.measurement_units = ["kg", "g", "litre", "ml", "m", "cm", "piece", ...]
pack.retail.measurement.minimum_increment = {kg: 0.05, litre: 0.1, m: 0.1, ...}
```

### 9.3 Catalog entry for bulk-splittable item

```yaml
retail.catalog.entry:
  item_ref: rice-pishori-001
  category: grain
  is_bulk_splittable: true
  base_price: 3500
  pricing_unit: kg               # the unit base_price refers to (TZS 3500 per kg)
  minimum_increment: 0.05        # 50g minimum sale
  measurement_unit_for_inventory: kg
```

### 9.4 basket.item.add for bulk-splittable

The command carries `quantity_measure` (decimal) instead of `quantity` (integer):

```
retail.basket.item.add.request {
  basket_id,
  item_ref: rice-pishori-001,
  quantity_measure: 2.5,            # 2.5 kg of pishori rice
  measurement_unit: kg              # validates against catalog entry's pricing_unit
}
```

### 9.5 The emit + Checkout resolution

```
retail.basket.item.added.v1 {
  basket_id,
  line_id,
  item_ref: rice-pishori-001,
  quantity_measure: 2.5,
  measurement_unit: kg,
  unit_price: 3500,                 # base price per kg (RE3 indicates, Checkout resolves)
  ...
}

Eventually retail.bill.ready.v1 carries:
  saleable_lines[0] = {
    ...,
    quantity: 2.5,                  # Saleable Line shape uses quantity field; vertical maps quantity_measure → quantity for CN-4-021
    unit_price: 3500,
    measurement_unit: kg,           # additional payload field per RE9
    ...
  }

Checkout K2 line_total = 3500 × 2.5 = 8,750 (then layer resolution + tax)
```

### 9.6 Inventory tracking

Inventory deducts in measurement units. Per CN-5-003 + pack hook `pack.retail.inventory_expansion_mode.grain = "auto"` (Pattern A), Inventory derives deduction from sale (2.5 kg deducted from rice-pishori-001 stock). The Inventory primitive (CN-4-011) supports decimal quantities per measurement unit; this is not a retail extension.

### 9.7 Refunds for bulk-splittable

Customer returns 1 kg of the 2.5 kg of rice purchased (perhaps the bag tore). Refund computes `quantity_measure: 1.0` of total `quantity_measure: 2.5` original. Pro-rated refund amount per pack rule. UI-02 compensation per RE4 holds — the refund event carries the partial quantity_measure.

### 9.8 Why this matters

Without RE9, Mama Amina's till can't sell rice by the kilo. The framework would have served the supermarket-with-pre-packaged-rice case while failing the duka-by-the-kilo case — leaving the majority of Tanzanian retail SMEs unserved. RE9 closes that gap.

---

## 10. Sale Completion + HO9 Settlement-Back

### 10.1 The bill.ready emission

When `retail.sale` Workflow reaches `bill_ready` state, it emits the canonical CN-6-102 NC3 + CN-6-104 HO1 event:

```
retail.bill.ready.v1 {
  bill_id,
  site_id,                                 # SP5 + CTR-024
  saleable_lines: [ ... CN-4-021 shapes ... ],
  originating_workflow_ref,                # filter target for HO9
  business_date,                           # CN-5-105 tax period
  payer_party_ref?,                        # optional; B2C may be anonymous
  expansion_mode_per_line: [...]           # Pattern A vs B per line per Q3 closure
}
```

### 10.2 The HO9 settlement-back

Per CN-6-104 HO9, the Workflow declares `settlement_subscription`:

```yaml
settlement_subscription:
  event_type: checkout.settled.v1
  filter: originating_workflow_ref == <self>
  transitions_to: completed
```

When `checkout.settled.v1` arrives with matching `originating_workflow_ref`, the Workflow primitive transitions from `bill_ready` to `completed` and emits `retail.sale.completed.v1`. Vertical author does not write handler code — declarative.

### 10.3 The completed event

```
retail.sale.completed.v1 {
  sale_id: <originating_workflow_ref>,
  bill_ref,
  site_id,
  saleable_lines,                          # final per Checkout K2 resolution (post-discount + post-tax line totals)
  tender_summary,                          # per checkout.settled.v1
  business_date,
  party_ref?,                              # if identified
  causation_id: <checkout.settled event_id>
}
```

This event is what Reporting subscribes to for "today's revenue." Accounting subscribes for the auto-journal. Promotion subscribes for loyalty accrual + cost-share. Mama Amina sees "today's revenue" tick upward.

---

## 11. Refunds + Compensation (RE4)

### 11.1 The refund command + window

```
retail.sale.refund.request {
  sale_id,
  refund_lines: [{line_id, refund_quantity?, refund_amount?}],
  reason,
  requester_party_ref,
  requested_at
}
```

Pack hook `pack.retail.refund.window_days` (default 30) gates acceptance. Refunds beyond the window are rejected at command-time (bus emits `kernel.command.rejected.v1`); resolution requires manager override (separate command per pack governance).

### 11.2 The refund event (UI-02 compensation)

```
retail.sale.refunded.v1 {
  refund_id,
  sale_ref: <original_sale_id>,
  refund_lines,
  refund_amount,
  reason,
  business_date,                           # business date of REFUND, not original sale
  refund_method,                           # how the customer is reimbursed; tender-method abstract per RE11
  causation_id
}
```

The event carries `compensation_basis_none: "UI-02 closure event itself"` per RE4 — refund IS the compensation; no further compensation pair.

### 11.3 Cross-engine effects

- **Accounting**: reverses revenue projection per pack chart-of-accounts rule (sales-return account)
- **Inventory**: increments stock per Pattern A (Checkout K2 driven) OR awaits explicit return event per Pattern B (depending on item category)
- **Promotion**: reverses loyalty accrual; recalls cost-share if active promotion was applied to original sale (per CN-5-007 N3)
- **Reporting**: subtracts from "today's revenue"; adds to "refunds" KPI

### 11.4 Partial refunds

`refund_quantity` (RE9 bulk-splittable) supports decimal. Customer returns 1 kg of 2.5 kg purchased — refund_quantity = 1.0 with measurement_unit = kg. Refund_amount pro-rated per pack rule (typically `original_unit_price × refund_quantity` adjusted for any discounts that applied).

---

## 12. Catalog Management (SP3 Tenant-Scope)

### 12.1 retail.catalog Workflow

Per CN-6-103 §6 tenant-scope exception catalog row, `retail.catalog.*` events are tenant-scope. Catalog entries (item definitions, base prices, categories, branch overrides) live tenant-wide. Per-site sale events (`retail.sale.*`) remain site-scope per SP1.

### 12.2 The catalog entry shape

```
retail.catalog.entry.added.v1 {
  entry_id,
  item_ref,                                # Foundation Item primitive (CN-4-011)
  category,                                # per pack.retail.item_categories taxonomy
  base_price,
  pricing_unit?,                           # if bulk-splittable per RE9
  is_bulk_splittable: false,
  minimum_increment?,
  branch_overrides?: [                     # optional per-site deviations
    {site_id, override_price}
  ],
  tax_treatment_category,                  # input for pack.tax.lookup
  inventory_expansion_mode,                # auto | vertical_managed (Pattern A vs B)
  active_from,
  added_by,
  business_date
}
```

### 12.3 Multi-site retailer

A chain like Nakumatt (multiple supermarkets across Tanzania, one tenant) maintains tenant-wide catalog. A catalog entry update propagates to all sites uniformly (tenant-scope per SP3). Site-specific deviations live in `branch_overrides` array within the same tenant-scope catalog entry.

---

## 13. Inventory Integration (Pattern A vs B per Item Category)

### 13.1 The pack-driven choice

Per CN-6-104 catalog row Inventory + Q3 closure:

```
pack.retail.inventory_expansion_mode = {
  packaged_goods:    "auto",              # Pattern A — Checkout K2 derives 1 sale → 1 deduct
  grain:             "auto",              # Pattern A with measurement_unit (decimal supported)
  oil:               "auto",              # Pattern A with measurement_unit
  meat:              "auto",              # bucha — kg deducted from carcass; offcuts tracked via Inventory primitive natively
  prepared_food:     "vertical_managed",  # if retail tenant prepares meals (rare; restaurant vertical otherwise)
  ...
}
```

### 13.2 Pattern A in retail (canonical)

For most retail items, Pattern A is correct: the sale itself implies the consumption. Customer buys 5 bars of soap → Inventory deducts 5 from soap stock. Customer buys 2.5 kg of rice → Inventory deducts 2.5 kg from rice stock. No vertical-side consumption event needed.

### 13.3 Pattern B in retail (rare)

A retail tenant that also prepares ready-to-eat meals on-site (an unusual case — most prepared-food tenants are restaurants, not retailers) may need Pattern B. The retail vertical declares `vertical_managed` for that category; emits `retail.ingredient.consumed.v1` per meal sold; Inventory subscribes. Boundary with Restaurant vertical (CN-6-002) is the engine kind — same item-consumption pattern, different engine identity.

### 13.4 Offcuts (bucha specifics)

Mzee Karim's bucha receives a full ng'ombe carcass (200 kg). Customers buy fillet (premium cut, sold by kg), minofu (general cuts), nyama-ya-kuchemsha (boil-grade). The Inventory primitive supports lot-based and offcut tracking natively (CN-5-003). Retail vertical does NOT need special bucha logic — Inventory's offcut model handles it. Mzee Karim's "carcass arrives" event is `inventory.lot.received.v1` (Inventory mechanism); sales emit `retail.bill.ready.v1` carrying line items per cut category; Inventory derives deduction per Pattern A.

---

## 14. Tax Integration (CN-5-105 + tenant_tax_profile Gate)

### 14.1 The pack-driven tax_treatment_ref lookup

Per CN-5-105 N7 + CN-6-100 §3 Step 8, the vertical never computes tax rates. At basket-line addition, retail looks up `tax_treatment_ref` via pack:

```
tax_treatment_ref = pack.tax.lookup(
  item_category: <from catalog entry>,
  tenant_tax_profile: <tenant's registration status per CTR-027>,
  business_date: <today>
)
```

### 14.2 tenant_tax_profile cases

Per CN-5-105 N7 + WP4 Mama Amina's reality:

- **Mama Amina, VAT-non-registered (below TZ TFRS threshold, default for small dukas)**: tax_treatment_ref returns "exempt" or "zero-rated" path; no output VAT on sale; line_total = pre-tax price
- **Mama Amina post-expansion if VAT-registered**: tax_treatment_ref returns "VAT-standard" for most retail items; Checkout K2 adds VAT line; receipt shows breakdown
- **Mama Amina with pharmacy addition (Faraja's counter)**: OTC retail items still subject to retail tax rules; prescription items go via `pharmacy.*` vertical with possibly different tax treatment per pack

The retail vertical is agnostic. It looks up, it passes the ref, it lets Checkout compute.

### 14.3 Special tax rules — pack handles

- **Tourism-levy items at lodge gift shop**: pack adds the levy line per item category
- **Excise items (matches, certain beverages)**: pack handles excise treatment
- **NGO exempt customer (B2B-NGO tier)**: tenant_tax_profile combined with customer-specific tier may yield zero-rated path
- **Multi-jurisdiction tenant**: deferred to CN-5-105 v2

All via pack. Vertical never branches on these.

---

## 15. B2C vs B2B + Party Integration (RE6 + Q4 Closure)

### 15.1 The default — B2C anonymous

Most retail transactions are B2C anonymous. The vertical does NOT require `payer_party_ref` — `retail.bill.ready.v1` may omit it. Customer walks in, buys, leaves; the receipt is the only record. Loyalty accrual requires identification (party_ref present); without it, no loyalty.

### 15.2 B2C identified

A loyalty-program customer (Mama Halima) presents her phone/loyalty card at the till. Salma emits `retail.customer.identified.v1 {party_ref}` early in the basket. The downstream `retail.bill.ready.v1` carries the party_ref; Promotion subscribes and accrues loyalty.

### 15.3 B2B tradesman / NGO / negotiated tier

The Party primitive instance carries:

```
party_role: customer
customer_type: b2b                       # OR tradesman, ngo
negotiated_pricing_tier_ref: <tier-x>    # points to tenant-scope tier registry
```

The tier registry (`retail.pricing_tier.*` — sub-namespace within retail) is tenant-scope per SP3. Tier entries define discount percentages per item category for that tier.

When a tradesman buys at Mama Amina's:
1. Salma scans tradesman's loyalty card → `retail.customer.identified.v1 {party_ref}`
2. Party primitive reveals `customer_type: tradesman`, `negotiated_pricing_tier_ref: <tradesman-tier>`
3. `retail.bill.ready.v1` carries party_ref; discount_refs includes `{layer: customer_specific, tier_ref: <tradesman-tier>}`
4. Checkout K2 applies tradesman-tier discount per pack rules

Same retail.sale Workflow. Same emission shape. Different pricing input via party metadata. Zero schema change.

### 15.4 Why no Party primitive extension

CN-4-011 Party primitive supports flexible metadata. `customer_type` and `negotiated_pricing_tier_ref` are metadata fields tenant-scope-scoped. No Foundation extension needed. Term 4 unburdened. CTR-zero respected.

---

## 16. Payment Method Abstraction (RE11)

This section makes RE11 explicit — the doctrine the rest of the doc honours throughout.

### 16.1 The three-layer governance

| Layer | Owns | Mechanism |
|-------|------|-----------|
| Universal Checkout (CN-5-009) | The abstract tender-method registry | CTR-028: queryable registry of method-category vocabulary + per-category providers |
| Regional agent (per Charter Law 6 + D-003) | Curates enabled methods per region | CTR-049: regional pack `pack.region.payment_methods_enabled` declares the approved set per region (Tanzania may include mobile money providers + card networks + bank transfer + cash; Kenya, Uganda, Nigeria, Ghana each have their own region-specific set) |
| Tenant | Configures subset within regional set | CTR-049: tenant pack `pack.tenant.payment_methods_subset` |
| Customer | Selects from tenant's enabled set at tender | Customer-facing UI surface (Term 3) |

### 16.2 What retail.bill.ready.v1 carries

**Nothing about method.** The emission is method-agnostic. Universal Checkout, on receiving the bill, presents the tenant's enabled methods to the cashier (in-store) or to the customer's mobile UI (RE10 remote). The cashier or customer chooses. Checkout invokes the appropriate Term 7 adapter per CTR-050.

### 16.3 Why this is non-negotiable

- **Law 5** — compliance configured, not coded. Embedding provider names in retail doctrine would code regional preference. Worse, it would lock retail into specific commercial relationships (which change — providers come and go, mergers happen, regional regulations shift).
- **Law 6** — distribution regional. Tanzania's mobile money landscape (multiple competing providers) is not Kenya's, is not Uganda's, is not Ghana's. Regional agents own this curation under their compliance accountability.
- **D-003** — multiple regional agents per region; referral does not replace agency. Curation is per-agent territory under platform governance.
- **CTR-028** — already-existing abstract method registry that Universal Checkout consumes; retail must use it consistently with every other vertical that emits to Checkout.

### 16.4 What worked-pattern narratives use

Throughout §17 below, payment narratives use **method-category** language ("mobile money tender", "card-present", "card-not-present push", "bank transfer", "cash", "credit-via-Obligation") and **never** specific provider names. When concretely-grounded, narratives say things like "via the mobile money method the regional agent enabled for Tanzania and Mama Amina configured for her duka" — making the chain explicit without picking a provider.

### 16.5 In-store vs remote method availability (RE10 + CTR-050)

The set of methods available for in-store tender (cashier-attended) may differ from the set available for remote ordering (mobile app fulfillment). Cash is in-store only; mobile push payment, card-not-present, and bank push are remote-eligible. CTR-050 (Term 6 → Term 7) asks Term 7 to declare which adapter capabilities support which channels (in-store vs remote vs QR-presented), so tenant configuration UI surfaces only the appropriate methods per channel.

CN-6-001 v1 references this dependency without resolving it; Term 7 confirms at integration phase.

---

## 17. Mixed-Vertical Activation

### 17.1 Retail + Pharmacy at one tenant (Mama Amina's expansion)

Per CN-6-101 §11.6 Mama Amina narrative + CN-6-102 §11.4 Mixed-Vertical loyalty walkthrough + Brief §11.7 closure: Mama Amina activates BOTH `retail.*` (her existing duka, OTC paracetamol et al.) AND `pharmacy.*` (Faraja's prescription counter) per CN-6-105 Mixed-Vertical Tenant pattern.

From the retail engine's perspective:
- Retail catalog includes OTC paracetamol, vitamins, plasters as standard catalog entries (RE7 — any item type)
- Retail.sale flow handles OTC purchases identically to any other retail item
- Pharmacy.*  events emit from the pharmacy vertical engine — NOT from retail
- Cross-vertical loyalty for Mama Halima (cross-doc cameo) flows through Universal Promotion + Party primitive per CN-6-102 §11.4 — no `cross_vertical.*` events

### 17.2 Retail + Restaurant at one tenant (lodge gift shop)

A future scenario: Lodge Serengeti's gift shop sells branded merchandise. Tenant activates both `retail.*` (gift shop) AND `restaurant.*` (lodge restaurant) AND `hotel.*` (lodge property) per Mixed-Vertical pattern. Each vertical operates its own Workflow per its own site (or sub-site within one property). Cross-vertical relationships (charge-to-room from gift shop) use Obligation primitive per CN-6-104 §10 doctrine; CN-6-005 future will catalog the specific pattern.

### 17.3 Retail + Workshop at one tenant (Brief §7.1 retail-as-workshop)

Mzee Hassan's workshop produces a custom window-frame; sells via retail POS. Per Brief §7.1: tenant activates both `retail.*` (POS face) AND `workshop.*` (production face). Retail catalog includes the workshop-produced item; retail.sale flow handles the sale; Inventory subscribes to the consumption (Pattern A per pack); Workshop's project workflow completes when the deliverable is sold. Cross-vertical pattern via Inventory + Foundation Item primitive — no direct call (VE2). CN-6-005 future will catalog.

---

## 18. Worked Patterns — Six Real Tanzanian Retail Scenarios

### 18.1 WP1 — Salma's morning at Mama Amina's till

**Time:** 7:15 am, Kariakoo, dry season. Mama Amina opens the duka; Salma — Mama Amina's daughter, 19, finishing form-six — works the till. First customer is a watu-wengi neighbour buying breakfast supplies.

**Items:**
- 1 kg sukari (sugar) — bulk-splittable per RE9
- 0.5 litre cooking oil — bulk-splittable per RE9
- 3 vipande vya sabuni za kuogea (bath soap) — packaged
- 1 strip ya paracetamol — OTC, packaged (RE7 retail handles)

**Flow:**

1. Salma opens basket — `retail.basket.open.request` → `retail.basket.opened.v1`
2. Adds items:
   - `retail.basket.item.add.request {item_ref: sukari-001, quantity_measure: 1.0, measurement_unit: kg}` → `retail.basket.item.added.v1`
   - `retail.basket.item.add.request {item_ref: oil-001, quantity_measure: 0.5, measurement_unit: litre}`
   - `retail.basket.item.add.request {item_ref: soap-002, quantity: 3}`
   - `retail.basket.item.add.request {item_ref: paracetamol-strip-001, quantity: 1}`
3. Customer presents loyalty card (he's a regular) → `retail.customer.identified.v1 {party_ref}`
4. Salma initiates checkout — `retail.basket.checkout.request` → Workflow → `checkout_initiated`
5. Vertical assembles bill — tax_treatment_ref lookups per item via pack (Mama Amina is VAT-non-registered per N7; refs return zero-rate path)
6. `retail.bill.ready.v1` emits with saleable_lines × 4
7. Universal Checkout K2 resolves prices (no promo active this morning; no loyalty redemption this transaction); presents tender choices per tenant's enabled methods (the regional agent has enabled cash and mobile money push for Tanzania; Mama Amina has enabled both)
8. Customer chooses mobile money push; Checkout invokes the Term 7 adapter for the customer's selected provider; settlement returns
9. `checkout.settled.v1` arrives → HO9 transitions Workflow → `completed` → `retail.sale.completed.v1` emits
10. Fan-out: Accounting projects revenue (zero-rated → no VAT line); Inventory deducts (1 kg sugar, 0.5 litre oil, 3 soap bars, 1 paracetamol strip — all Pattern A); Promotion accrues loyalty for the identified party; Reporting feeds the day's projection; retail-floor-advisor refreshes
11. Receipt prints. Customer leaves. Total elapsed: ~45 seconds.

**Doctrine demonstrated:** RE1 (duka in spectrum), RE2 (lifecycle), RE3 (multi-price no-discount path), RE7 (paracetamol = retail item), RE9 (bulk-splittable kg + litre), RE11 (payment method abstract — narrative uses category, never provider name), HO1 + HO9 (canonical handoff + settlement-back), HO5 (fan-out × 5).

### 18.2 WP2 — Mzee Juma returns soap (refund within window)

**Time:** Day 5 after WP1. Mzee Juma — a regular Kariakoo neighbour, 60s, retired teacher — returns to Mama Amina's duka with one of three soap bars he bought last week. He developed a skin reaction; wants a refund on the unused bar.

**Flow:**

1. Mzee Juma presents the soap bar + a receipt scrap (the bar's batch matches a recent sale; loyalty-card lookup confirms his party_ref + the sale_ref)
2. Salma initiates refund — `retail.sale.refund.request {sale_id: <original>, refund_lines: [{line_id: <soap-line>, refund_quantity: 1, reason: "skin_reaction"}], requester_party_ref: <juma>}`
3. Pack check: `pack.retail.refund.window_days = 30`; the original sale was day-5-ago → within window → command accepted
4. `retail.sale.refunded.v1` emits with refund_amount (TZS 2,000 if no discount was applied originally; pro-rated if discount applied)
5. Fan-out:
   - Accounting reverses revenue per pack sales-return chart-of-accounts
   - Inventory adds the returned bar back to stock per Pattern A (or marks defective per pack rule — for skin-reaction, likely "defective" disposal path)
   - Promotion reverses loyalty accrual proportionally
   - Reporting subtracts from current-period revenue + adds to "refunds" KPI
6. Salma hands Mzee Juma TZS 2,000 cash from the till (RE11 — refund method is also tenant-configured; Mama Amina has enabled cash refunds for in-store-returns per her tenant configuration). Mzee Juma signs the refund receipt. Done.

**Doctrine demonstrated:** RE4 (refund UI-02 compensation), RE5 (explicit events), RE11 (refund method abstract), pack-driven window, NC9 compensation pair.

### 18.3 WP3 — Mama Halima multi-price resolution (5 layers)

**Time:** Friday afternoon. Mama Halima (the Dar logistics broker on her weekly Kariakoo run; cross-doc cameo from CN-6-100 §12 + CN-6-102 §11.4 + CN-6-104 WP3) buys soap at Mama Amina's. She's a loyalty member with accumulated points. Friday is Mama Amina's "Friday-duka day" with 10% off all soap.

**Items:**
- 5 bars of washing soap @ catalog TZS 2,000/bar

**Flow:**

1. Salma opens basket; Mama Halima identifies — `retail.customer.identified.v1 {party_ref: <halima>}`
2. Soap added — `retail.basket.item.added.v1 {soap, qty: 5, unit_price: 2000}`
3. Mama Halima asks to redeem 750 loyalty points (worth TZS 375)
4. Salma initiates checkout; vertical assembles bill with `discount_refs` including:
   - `{layer: active_promo, promotion_ref: <friday-duka-10pct>}`
   - `{layer: loyalty, intent: redeem, max_points: 750}`
5. `retail.bill.ready.v1` emits; Checkout K2 + Promotion PR1 resolve per pack layer_order:
   - Base: 5 × 2,000 = 10,000
   - Branch override: none (Mama Amina hasn't set one)
   - Active promo (Friday-duka -10%): 9,000
   - Loyalty (750 points = -375): 8,625
   - Customer-specific: none (B2C standard)
   - Tax: zero-rated (Mama Amina VAT-non-registered per N7)
   - **Final total: TZS 8,625**
6. Customer chooses tender per tenant's enabled methods; settlement; HO9 closure
7. Promotion emits `promotion.loyalty.redeemed.v1 {party_ref: <halima>, points_consumed: 750, ...}` (universal split-it; cross-vertical loyalty pattern per CN-6-102 §11.4)

**Doctrine demonstrated:** RE3 multi-price 5-layer (with 2 inactive layers); BD5 split-it (universal Promotion handles loyalty); cross-doc cameo continuity; RE11 (no provider named in narrative); pack-driven tax (zero-rate per tenant_tax_profile).

### 18.4 WP4 — Mixed-Vertical: OTC retail + prescription pharmacy at Mama Amina's

**Time:** A Tuesday after Mama Amina's expansion with cousin Pendo and Faraja per CN-6-101 §11.6 (hypothetical; pharmacy.* vertical also activated per Mixed-Vertical Tenant pattern).

**Scenario:** Mama Halima brings two needs to Mama Amina's:
- 2 boxes of paracetamol (OTC) — buys at Salma's front retail till
- 1 prescription for her daughter's antibiotics — fills at Faraja's back pharmacy counter

**Flow (retail side — CN-6-001 scope):**

1. Salma opens basket; Mama Halima identifies → `retail.customer.identified.v1 {party_ref: <halima>}`
2. 2 boxes of paracetamol added — `retail.basket.item.added.v1` (paracetamol is a retail catalog entry per RE7 item-type agnosticism; category = OTC_pharmaceuticals per pack taxonomy)
3. Checkout; `retail.bill.ready.v1`; settlement; `retail.sale.completed.v1`
4. Promotion emits `promotion.loyalty.earned.v1` per CN-6-102 §11.4 pattern (universal mechanism)

**Flow (pharmacy side — out of CN-6-001 scope; cross-reference only):**

1. Mama Halima moves to Faraja's counter; Faraja's pharmacy workflow (`pharmacy.prescription`) handles validation + dispensing per CN-6-101 §11.6
2. `pharmacy.bill.ready.v1` emits; separate Universal Checkout transaction; separate `checkout.settled.v1`
3. If Mama Halima redeems loyalty at the pharmacy counter (different transaction), Promotion handles per same universal split-it pattern; Party primitive links the two

**Doctrine demonstrated:** RE7 (paracetamol = retail item; pharmacy ≠ retail per BD6 3-of-3); Mixed-Vertical Tenant (CN-6-105 pattern); no `cross_vertical.*` events; cross-vertical loyalty via Promotion + Party.

### 18.5 WP5 — Mzee Karim bucha: bulk-splittable nyama

**Time:** Saturday morning. Mzee Karim's bucha shop in Kariakoo. The shop received a full ng'ombe carcass earlier in the week; Inventory primitive tracks the 178 kg of remaining cuts (fillet, minofu, nyama-ya-kuchemsha, bones for soup).

**Scenario:** Bibi Asha — Mama Amina's neighbour, preparing nyama-choma for her family — walks in:

- "Mzee, naomba kilo mbili za fillet, na nusu kilo ya minofu kwa supu."

**Flow:**

1. Mzee Karim's till is set up with `retail.*` vertical activated; his pack has:
   - `pack.retail.bulk_splittable_categories` includes "meat"
   - `pack.retail.measurement_units` includes "kg"
   - `pack.retail.measurement.minimum_increment = {kg: 0.05}`
   - Catalog entries: `fillet-ngombe` (TZS 18,000/kg), `minofu-ngombe` (TZS 12,000/kg)
2. Mzee Karim opens basket; weighs cuts on his digital scale:
   - 2.1 kg of fillet (scale reading; minimum increment 0.05 = 50g, so 2.1 is valid)
   - 0.55 kg of minofu
3. `retail.basket.item.add.request {item_ref: fillet-ngombe, quantity_measure: 2.1, measurement_unit: kg}` and same for minofu
4. `retail.basket.item.added.v1` emits each
5. Checkout; `retail.bill.ready.v1` carries saleable_lines:
   - fillet: 2.1 kg × 18,000 = 37,800
   - minofu: 0.55 kg × 12,000 = 6,600
   - Total: 44,400 TZS pre-tax
6. Mzee Karim is VAT-non-registered → zero-rate path → Checkout K2 final 44,400 TZS
7. Bibi Asha pays per Mzee Karim's enabled tender methods (regional curation per CTR-049 + tenant configuration; Mzee Karim enables both cash and mobile money push)
8. Settlement; HO9 closure; `retail.sale.completed.v1`
9. Fan-out: Inventory deducts 2.1 kg from fillet stock and 0.55 kg from minofu stock per Pattern A (Inventory primitive handles decimal quantities natively + lot/offcut model from the ng'ombe carcass per CN-5-003); Accounting projects revenue; Reporting updates today's bucha-revenue KPI

**Doctrine demonstrated:** RE1 (bucha in spectrum), RE7 (meat = retail item type), RE9 (full bulk-splittable lifecycle with measurement units + minimum increment), RE11 (payment method abstract), Inventory's native lot/offcut handling per CN-5-003.

### 18.6 WP6 — Mobile app order (RE10 remote ordering + abstract payment)

**Time:** Tuesday 3pm. A regular customer — let's call her Nina, a mid-level office worker in Kariakoo — wants to pick up groceries on her way home. She uses Mama Amina's tenant mobile app (Term 3 surface; Term 7 channel adapter).

**Scenario:**

1. Nina opens the mobile app; scans Mama Amina's Business ID / QR (`pack.retail.fulfillment.remote_ordering_business_id_format` — Term 1 onboarding governance) — app loads Mama Amina's catalog
2. Nina browses categories per `pack.retail.item_categories` taxonomy; adds:
   - 2 kg sukari (RE9 bulk-splittable — app shows price per kg + lets her enter measure)
   - 1 litre cooking oil (bulk-splittable)
   - 1 pack of bread
   - 3 cans of tomato paste
3. Nina chooses "pickup at 5pm" (delivery is `pack.retail.fulfillment.delivery_enabled = false` for Mama Amina's duka v1; future Logistics integration)
4. Nina settles in-app via remote tender method — per tenant's enabled remote-eligible methods (regional agent's CTR-050-confirmed adapter set excludes cash for remote; includes mobile money push, card-not-present, and bank push). Nina chooses one of these — narrative is method-agnostic
5. `retail.fulfillment.place.request` accepted; actor field = Nina (customer-as-actor per CN-4-007 + D-DISC-002)
6. `retail.fulfillment.placed.v1` emits
7. `retail.bill.ready.v1` also emits (the bill is the same shape as any retail bill — the *fulfilment* lifecycle is what differs, not the bill)
8. Universal Checkout settles (settlement is pre-confirmation per RE10 design — customer pays at order placement); `checkout.settled.v1` arrives
9. HO9: `retail.fulfillment.confirmed.v1` emits — Mama Amina's tenant dashboard (Term 3 surface) shows the new order with confirmation; Salma sees the alert
10. Mama Amina/Salma starts preparing — `retail.fulfillment.prepare.request` → `retail.fulfillment.preparing.v1`
11. Items measured, bagged, ready — `retail.fulfillment.mark_ready.request` → `retail.fulfillment.ready_for_handover.v1`
12. App pings Nina: "Your order is ready for pickup."
13. Nina arrives at 5:10 pm; Salma hands over the bag — `retail.fulfillment.handover.request` → `retail.fulfillment.handed_over.v1`
14. Internal confirmation; `retail.fulfillment.completed.v1`
15. Fan-out: Accounting (revenue projection — actually projected at settlement time, refined at completion); Inventory deducted at confirmation (Pattern A from the bill); Reporting; advisors

**Doctrine demonstrated:** RE10 (full remote-fulfillment Workflow lifecycle), RE11 (in-store-vs-remote method availability per CTR-050; provider abstract), RE9 (bulk-splittable in mobile UX), CN-4-007 (customer-as-actor), Business ID / QR via CTR-049 + Term 1 onboarding, Term 7 mobile-app channel adapter dependency.

---

## 19. Boundaries + Open Items + Cross-Term Hooks

### 19.1 CN-6-001's place in the corpus

| Concern | Owned by | CN-6-001 role |
|---------|----------|----------------|
| Retail engine declaration | **CN-6-001** (this doc) | Authoritative |
| Cross-cutting vertical framework | CN-6-100..104 | Parents — CN-6-001 applies, never amends |
| Universal Checkout consumption | CN-5-009 | Consumer of `retail.bill.ready.v1` |
| Universal Promotion (loyalty, ROI) | CN-5-007 | Consumer; BD5 split-it |
| Universal Inventory (Pattern A primarily) | CN-5-003 | Consumer; decimal quantity native per RE9 |
| Universal Accounting (revenue + sales return) | CN-5-001 | Consumer via CTR-030 |
| Universal Tax-Aware Engines | CN-5-105 | Consumer via pack lookups + N7 gate |
| Mixed-Vertical Tenant patterns | CN-6-105 (future) | CN-6-001 §17 demonstrates with Mama Amina + Faraja; CN-6-105 generalises |
| Cross-vertical bridges (retail ↔ workshop sell-via-POS; gift-shop charge-to-room) | CN-6-005 (future) | CN-6-001 §17 references; CN-6-005 catalogs |
| Mobile app UX surface | Term 3 (pending) | CN-6-001 §6 + WP6 expose `retail.fulfillment` events; Term 3 designs surface |
| Channel adapters (mobile push, card-not-present, QR) | Term 7 (pending; partial) | CN-6-001 RE11 + WP6 + CTR-050 reference; Term 7 confirms coverage |
| Regional payment method curation | Term 2 (pending) | CN-6-001 RE11 + CTR-049 reference; Term 2 fills regional packs |
| Onboarding governance (Business ID / QR; activation gates; tier registry) | Term 1 (pending) | CN-6-001 §12 + §15 reference; Term 1 fills |

### 19.2 CTRs

**Filed alongside this work cycle (per `99725fd`):**

- **CTR-049** (Term 6 → Term 2) — Regional payment-method curation. Term 2 owns the regional agent governance for which payment methods are enabled per region per Charter Law 6 + D-003. CN-6-001 RE11 + WP1/WP3/WP5/WP6 reference; tenant configuration sits within regional set.
- **CTR-050** (Term 6 → Term 7) — Payment adapter coverage. Term 7 owns adapter capabilities including in-store (cashier-attended), remote push (mobile-app-initiated), card-not-present, bank push, QR-presented-to-customer. CN-6-001 RE10 + RE11 + WP6 reference; coverage determines which methods are remote-eligible vs in-store-only.

**Cited (existing, no new for retail mechanics):**

- CTR-018 (registration), CTR-002 (lines feed pattern), CTR-024 (site_id payload), CTR-026 (UI-03 + UI-09), CTR-030 (Accounting payload sufficiency), CTR-038 (namespace reservation — `retail.*` claimed), CTR-044 (engine_kind: vertical pending), CTR-045 (onboarding governance pending), CTR-046 (mechanizable anti-patterns; DC-NN-e + DC-NN-f queued), CTR-028 (abstract method registry; retail consumes), CTR-006 (existing payment adapter contract; CTR-050 extends), CTR-027 (site + currency registry; tenant_tax_profile gate)

**Cumulative CTR-046 expansion queue (unchanged from CN-6-104):** DC-NN-e (SP8 subscription-side platform-scope) + DC-NN-f (HO9 settlement-back-subscription mandatory).

### 19.3 Open items inside Term 6 scope

- **CN-6-002 Restaurant** — next per Brief §13.7; Lodge Serengeti restaurant anchor; recipe Pattern B
- **CN-6-003 Hotel** — Brief §13.8; Kilimanjaro Lodge Moshi + Lodge Serengeti chain
- **CN-6-004 Workshop** — Brief §13.9; Karakana ya Mzee Hassan; most complex existing vertical
- **CN-6-005 Bridges** — catalogs retail↔workshop sell-via-POS; gift-shop charge-to-room
- **CN-6-105 Mixed-Vertical Tenants** — generalises Mama Amina + Faraja pattern; Lodge Serengeti hotel + restaurant + gift shop trio
- **CN-6-905 Salon + light services cluster** — elevated per CN-6-101 §11.2
- **Multi-jurisdiction tenant** — deferred to CN-5-105 §11 v2

### 19.4 D-DISC cross-references

- **D-DISC-001 — Tenant-customer promotion UX**: WP3 Mama Halima multi-price + WP6 mobile app order touch the customer-facing side; CN-6-001 RE3 + RE10 set the engine surface; Term 3 designs UX.
- **D-DISC-002 — POS self-service expansion**: RE10 mobile app is a form of customer-as-actor self-service; in-store self-service kiosk would extend `retail.sale` Workflow with `pack.retail.self_service_enabled = true` + customer-as-actor identity. CN-6-001 manifest supports both pack flag + actor flexibility per CN-4-007.

### 19.5 The bar — retail is the framework's first concrete test

If the cross-cutting framework (CN-6-100..104) is correct, CN-6-001 written cleanly without amendments to upstream. This doc applies all eleven VE doctrines, all SP doctrines, all NC doctrines, all HO doctrines, and emerges with RE1-RE11 + 6 worked patterns + 2 CTRs filed — all within existing contracts. **The framework holds.** Mama Amina, Salma, Mzee Karim, Mzee Juma, Mama Halima, Faraja, Nina — each appears in a scenario whose mechanics derive directly from the framework without requiring an exception.

Next: CN-6-002 Restaurant. Lodge Serengeti's kitchen. Recipe Pattern B. Table conflicts. Same framework, second concrete test.

---

*— End of CN-6-001 Retail Engine v1 —*
