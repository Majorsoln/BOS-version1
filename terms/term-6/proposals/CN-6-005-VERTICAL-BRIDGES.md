# CN-6-005 — Vertical Bridges (Pattern Catalog)

> **Term:** 6 — Vertical Engines
> **Status:** Concept Phase v1.0 — Draft for Overseer review
> **Branch:** `claude/brave-johnson-S7Lky`
> **Reading order:** CN-6-100..104 (cross-cutting framework) → CN-6-001/002/003/004 (concrete verticals) → CN-4-011 (Foundation primitives) → **this doc**
> **Ordering authority:** TERM-6 Brief §13 — tenth Term 6 deliverable; **pattern catalog** documenting cross-vertical bridges established across the four concrete verticals.

---

## 1. Purpose & Boundary

### 1.1 Mission

CN-6-005 is the **Architect reference manual** for cross-vertical interactions. While CN-6-100..104 established the framework for adding verticals and CN-6-001..004 implemented four concrete verticals (Retail, Restaurant, Hotel, Workshop), real businesses operate across vertical boundaries: a hotel guest dines at the attached restaurant; a workshop sells a showroom item via retail POS; a regular customer earns loyalty across retail, restaurant, and hotel touchpoints; a tenant's alcohol license covers their pub, restaurant, and retail liquor section.

These cross-vertical interactions are not separate engines (per BD7 closure — "no bridge engines"). They are **patterns** by which Foundation primitives carry interactions between vertical engines without coupling them. Restaurant and Hotel never reference each other's events; the Obligation primitive carries the charge between them. Workshop and Retail never reference each other's catalogs; the Inventory primitive carries the item between them.

CN-6-005 catalogs these patterns by primitive carrier, enumerates the established instances from CN-6-001..004, and documents the anti-patterns (what NOT to do). It is **not** new doctrine — it is doctrine reaffirmation through enumeration, and a teaching reference for Architects implementing cross-vertical scenarios.

### 1.2 DOES vs DOES NOT

| CN-6-005 DOES | CN-6-005 DOES NOT |
|----------------|--------------------|
| Catalog cross-vertical patterns established in CN-6-001..004 + cross-cutting docs | Invent new bridge mechanisms — patterns are emergent from concrete verticals, not prescribed |
| Define BR1-BR5 bridge doctrine (Foundation primitives as carriers; VE2 + BD7 reaffirmation) | Define new Foundation primitives — uses existing five (Obligation, Inventory, Party, Document, Workflow) |
| Categorize patterns by carrier primitive (4 primary categories + Workflow edge case) | Define new vertical engines — concrete verticals already declared in CN-6-001..004 |
| Document the AP-B1..AP-B6 anti-pattern catalog (cross-vertical antipatterns consolidated) | Author per-vertical engine specifics — those live in CN-6-001..004 |
| Establish the living-catalog mechanism (additions via amendment when new verticals surface new patterns) | Establish a cross-tenant bridge framework (cross-tenant = Term 1 marketplace future per Q4 + N5) |
| Provide 5 concrete worked walkthroughs grounded in established characters | Introduce new characters — catalog reuses established corpus (Mwemas, Lodge Serengeti, Mzee Hassan, Mama Halima, etc.) |
| Inherit RE11 payment-method abstraction throughout (zero provider names) | Open new CTRs — pattern catalog uses existing contracts |

### 1.3 Audience

Architects implementing cross-vertical scenarios (primary); Term 6 itself authoring CN-6-105 Mixed-Vertical Tenants (the tenant activation side of bridges); Term 1 reviewing whether platform governance touches cross-vertical patterns; Term 3 designing UI surfaces that span verticals (e.g., a guest profile showing stays + dining history); Term 7 confirming integration coherence across vertical interactions; future vertical contributors who need to know what bridge patterns precede them.

### 1.4 Charter Compliance

| Law | How CN-6-005 honours it |
|-----|--------------------------|
| Law 1 — State from events only | All bridge patterns use event-store communication; AP-B6 explicitly prohibits side-channel coordination |
| Law 2 — Engines isolated | BR1 + BR2 enforce: verticals never subscribe to other verticals; Foundation primitives carry interactions |
| Law 3 — AI advisory only | Cross-vertical advisor suggestions (e.g., upsell suggestions on identified party) follow CN-5-010 advisory-only doctrine |
| Law 4 — Flexibility first-class | Living-catalog (BR5) accommodates new patterns from future verticals without architectural change |
| Law 5 — Compliance configured | Document-Sharing category includes tenant license + tax compliance documents (jurisdiction policy) |
| Law 6 — Distribution regional | Cross-tenant bridges (cross-region) deferred to Term 1 marketplace; v1 stays within-tenant per Charter Law 6 distribution doctrine |

### 1.5 Parsimony — Bridges are how verticals talk without touching

When Mwemas charge dinner to their room at Lodge Serengeti, the Restaurant and Hotel never speak to each other. Restaurant emits an Obligation; the Obligation primitive carries the charge; Hotel folio incorporates. When Mzee Hassan completes a window in his showroom and a contractor walks in to buy it, the Workshop and Retail never speak to each other. Workshop deposits the item into Inventory; Inventory primitive holds it; Retail catalog references it; the contractor pays. When Mama Halima earns loyalty at Kariakoo duka and redeems at Lodge Serengeti restaurant, Retail and Restaurant never speak to each other. They both reference the same Party primitive instance; Universal Promotion adjudicates.

This is the parsimony bar for bridges: **two verticals are decoupled if either can be decommissioned without breaking the other.** Foundation primitives are the carriers that preserve the decoupling. CN-6-005 doctrine BR1-BR5 below + anti-patterns §10 reinforce this discipline through enumeration.

---

## 2. Inputs and Relationship

### 2.1 Cross-cutting framework (parent)

- **CN-6-100** VE2 (no vertical-vertical direct calls); VE7 Workflow primitive
- **CN-6-101** BD7 closure (no bridge engines); BD5 split-it; AP3 catalog (CN-6-101 §10 — bridge engine anti-pattern established here)
- **CN-6-102** NC9 compensation pairs; §11.3 N3 cross-vertical reference schema (correct vs incorrect)
- **CN-6-103** SP3 tenant-scope exceptions (where bridges apply across sites)
- **CN-6-104** §10 cross-vertical Obligation doctrine (5-step pattern); HO5 fan-out; HO8 vertical reads Foundation primitive projections only

### 2.2 Concrete verticals (where patterns were established)

- **CN-6-001 Retail** — RE6 B2C/B2B Party metadata; RE7 item-type agnosticism; WS11 sell-via-Retail consumer side
- **CN-6-002 Restaurant** — REST7 charge-to-room (Obligation emission side); §13 5-step chain
- **CN-6-003 Hotel** — HOT5 charge-to-room subscription side; HOT4 chain guest profile (Identity-Linking); §11 6-step settlement loop-back
- **CN-6-004 Workshop** — WS11 sell-via-Retail emission side; §20 8-step chain; WS10 style export/import (Document-Sharing cross-tenant)

### 2.3 Foundation primitives (the carriers)

- **CN-4-011** Workflow + Party + Document + Obligation + Inventory Movement primitives — the five carriers
- **CN-4-012** Document Engine — license documents shared across verticals

### 2.4 Universal layer (subscribers + adjudicators)

- **CN-5-001** Accounting — recognises revenue at obligation/checkout settlement
- **CN-5-003** Inventory — native lot model carries Item-Transfer patterns
- **CN-5-007** Promotion — adjudicates Identity-Linking loyalty across verticals
- **CN-5-105** Tax — tenant_tax_profile shared across verticals (Document-Sharing)

### 2.5 Sibling Term 6 (related docs)

- **CN-6-105 Mixed-Vertical Tenants** (next, after CN-6-005) — the tenant activation side of bridges (how a tenant runs ≥2 verticals; CN-6-005 covers how the verticals interact)
- **CN-6-901..904** stress sketches — may surface new patterns to add to living catalog

### 2.6 CTRs

- **No new CTRs** — pattern catalog uses established Foundation primitives + cross-cutting framework
- Cited as-is: CTR-018/002/024/026/027/030/038/044/045/046/049/050/028/006/027
- **Cumulative CTR-046 expansion queue (unchanged):** DC-NN-e + DC-NN-f
- Future cross-tenant marketplace (Q4 deferred) will file CTR to Term 1 when activated (§11)

---

## 3. Bridge Doctrine — BR1 to BR5

### BR1 — All cross-vertical interaction routes via Foundation primitives

**Verticals never subscribe to other verticals' events. Cross-vertical interactions are carried by Foundation primitive events.** This is the concrete reaffirmation of VE2 (engine isolation) + BD7 (no bridge engines). A vertical engine's subscription manifest may include `obligation.*`, `inventory.*`, `party.*`, `document.*`, `workflow.*` (Foundation primitive events); it may NOT include `<other_vertical>.*`. The doctrine gate at CN-4-020 registration rejects.

When Restaurant needs to charge a Hotel guest's folio, Restaurant does not subscribe to `hotel.folio.opened.v1`. Restaurant emits an Obligation; Hotel subscribes to `obligation.created.v1` filtered by `kind: hospitality_charge` and `counterparty_workflow_ref == <self>`. The Foundation Obligation primitive is the only thing the two verticals share.

### BR2 — Foundation primitives are "carriers"

A carrier is a Foundation primitive that holds state about a cross-vertical interaction without itself being either vertical. The Obligation primitive carries a charge from creditor to debtor; neither Restaurant nor Hotel owns the Obligation — both reference it. The Inventory primitive carries an item from producer to seller; neither Workshop nor Retail owns the item record — both reference it. The Party primitive carries a customer identity; multiple verticals reference one Party.

Carriers are the architectural mechanism that lets verticals coexist without coupling. BR2 makes the carrier metaphor explicit doctrine.

### BR3 — Four primary pattern categories + Workflow edge case

The five Foundation primitives serve as bridge carriers, but they fall into four primary pattern categories based on the kind of cross-vertical interaction:

| Category | Carrier Primitive | What it carries |
|----------|---------------------|------------------|
| **Charge-Transfer** | Obligation (CN-4-011) | A monetary or settlement obligation between two engines (typically debtor-party / creditor-engine pairing) |
| **Item-Transfer** | Inventory (CN-5-003 + CN-4-011 Inventory Movement) | A physical or fungible item moving between engine contexts (workshop produces → retail sells; bucha lots → restaurant grills) |
| **Identity-Linking** | Party (CN-4-011) | A customer or counterparty identity referenced by multiple verticals (loyalty, history, B2B relationship) |
| **Document-Sharing** | Document (CN-4-012) | A document instance (license, certificate, tax profile, exported style) referenced by multiple verticals at one tenant |

**Workflow-Coordination edge case:** Most workflow coordination is intra-vertical (a Restaurant table_session aggregates its own orders; a Hotel folio aggregates its own charges). True cross-vertical workflow coordination — where two verticals' workflows must coordinate — reduces to the Obligation pattern. Example: a Hotel reservation gates on a Restaurant special-meal pre-arrival commitment; the gate is an Obligation, not a direct workflow link. §9 elaborates.

### BR4 — Patterns are emergent, not prescriptive

CN-6-005 catalogs patterns that **emerged** from the four established verticals (CN-6-001..004). It does not prescribe new patterns. When a future vertical (Insurance, Healthcare, Education, Marketing per Brief §3.3 stress sketches) interacts with established verticals in a novel way, the resulting pattern is added to the catalog via the living-catalog mechanism (§12).

This framing matters because it prevents over-engineering. Architects implementing today's cross-vertical scenarios consult the catalog for established patterns; they don't speculate about future bridge mechanisms. Each pattern in the catalog has provenance — a source vertical that established it.

### BR5 — CN-6-005 is a living catalog

Per Charter §11 living-document principle. Future additions:
- New verticals introduce new patterns → catalog amendment + CHANGELOG entry
- Existing patterns may be deprecated as verticals evolve → marked deprecated, never removed (preserve audit trail)
- New anti-patterns identified through field practice → AP-B7+ added

§12 elaborates the amendment gate per N3.

---

## 4. The Four Carrier Primitives

### 4.1 Obligation primitive (Charge-Transfer carrier)

Per CN-4-011: Obligation is a Foundation primitive representing a recorded debt or commitment between two parties. Manifest:

```
obligation.created.v1 {
  obligation_id,
  kind: <semantic_tag>,            # hospitality_charge, layby, no_show_charge, etc.
  debtor_party_ref,
  creditor_engine,                 # the vertical that issued the obligation
  creditor_workflow_ref,
  amount,
  counterparty_workflow_ref?,      # if cross-vertical (the other vertical's workflow that will fulfil)
  payload_ref,
  business_date
}
```

Cross-vertical use: Restaurant emits (creditor); Hotel subscribes filtered by `kind == "hospitality_charge"` AND `counterparty_workflow_ref == <self>`. Obligation primitive itself coordinates resolution at settlement.

### 4.2 Inventory primitive (Item-Transfer carrier)

Per CN-5-003 + CN-4-011 Inventory Movement. The Inventory primitive holds lots, quantities, locations. Items move between engine contexts via:

```
inventory.lot.added.v1 {
  item_ref,
  source_engine,                   # which vertical produced/received
  source_workflow_ref,
  category,
  quantity,
  site_id,
  business_date
}
```

```
inventory.lot.removed.v1 {
  item_ref,
  consuming_engine,                # which vertical consumed/sold
  consuming_workflow_ref,
  ...
}
```

Cross-vertical use: Workshop deposits a completed window (Workshop adds lot); Retail catalog references the item; customer buys via retail.sale.completed.v1; Inventory removes the lot. The Workshop never knows the Retail sale happened; the Retail never knows who produced the window. Both reference the same `item_ref`.

### 4.3 Party primitive (Identity-Linking carrier)

Per CN-4-011: Party is a Foundation primitive representing a customer, supplier, employee, or other external counterparty. Multiple verticals reference the same Party. Per RE6 + N5 of CN-6-001: Party metadata (`customer_type: b2c | b2b | tradesman`, `negotiated_pricing_tier_ref`, loyalty history references) supports cross-vertical context without primitive extension.

Cross-vertical use: Mama Halima is one Party instance. She shops at Mama Amina's duka → `retail.customer.identified.v1 {party_ref: <halima>}` + `retail.sale.completed.v1 {payer_party_ref: <halima>}`. Promotion sees `party_ref` and accrues loyalty. She dines at Lodge Serengeti → `restaurant.customer.identified.v1 {party_ref: <halima>}`. Promotion sees the same `party_ref` and applies her loyalty balance. She checks in at Lodge Serengeti hotel → `hotel.reservation.confirmed.v1 {party_ref: <halima>}`; hotel guest profile updates. Three verticals; one Party; no cross-vertical event subscriptions.

### 4.4 Document primitive (Document-Sharing carrier)

Per CN-4-012: Document is a Foundation primitive representing an issued, hash-verifiable document instance (license, certificate, statement, exported artifact). Multiple verticals reference the same Document at one tenant.

Cross-vertical use at one tenant: a tenant's alcohol license is one Document instance. Their Pub vertical (restaurant.* with bar_only config), Restaurant vertical (sit-down dining serving wine), and Retail vertical (supermarket selling packaged beer) all reference the same license Document — no per-vertical license duplication. The license document carries the tenant's compliance status across verticals.

### 4.5 Workflow primitive — edge case for cross-vertical coordination

Workflow primitive (CN-4-011 + VE7) is mostly intra-vertical. A vertical's Workflow instances coordinate the vertical's own state. Cross-vertical Workflow coordination is rare; when it happens, it reduces to the Obligation pattern. See §9.

---

## 5. Pattern Category 1: Charge-Transfer via Obligation Primitive

### 5.1 The category

Charge-Transfer patterns move a payment claim from one vertical (the creditor) to another (the resolver). The Obligation primitive carries the claim until settlement.

### 5.2 Pattern 1.1 — Hotel↔Restaurant in-stay charge-to-room (REST7 + HOT5)

**Established in:** CN-6-002 §13 (Restaurant emission side) + CN-6-003 §11 (Hotel subscription side)

**Mechanism (6-step end-to-end):**
1. Restaurant emits `restaurant.bill.ready.v1` with `payment_method_hint: charge_to_room` + `payer_party_ref: <hotel_guest>`
2. Universal Checkout routes per hint to Foundation Obligation: emits `obligation.created.v1 {kind: hospitality_charge, debtor: <guest>, creditor_engine: restaurant, counterparty_workflow_ref: <hotel_folio>}`
3. Hotel folio Workflow subscribes (filter: kind + counterparty); emits `hotel.folio.charge_added.v1` aggregating into folio
4. At guest checkout, `hotel.folio.ready.v1` emits aggregating all charges including the obligation_refs
5. Universal Checkout settles; Foundation Obligation primitive resolves per obligation_ref → emits `obligation.settled.v1`
6. Restaurant's subscription fires; restaurant.table_session.completed.v1 emits; restaurant revenue recognised

**Worked walkthrough:** WP1 §13.1 demonstrates concretely.

### 5.3 Pattern 1.2 — Multi-vertical Lodge Serengeti folio aggregation

**Established in:** CN-6-003 §10 + §11 (folio aggregates from multiple sources)

**Mechanism:** Lodge Serengeti tenant runs hotel + restaurant + spa (the spa being a sub-vertical service). All three emit Obligations against the guest's folio; the folio aggregates per Pattern 1.1's mechanism but with multiple creditor engines. Each charge type may carry distinct Obligation `kind` (`hospitality_charge` for restaurant + spa; `room_night_charge` for room-night accumulations).

The single folio.ready emission at checkout settles all obligations in one transaction.

### 5.4 Pattern 1.3 — Pub↔Hotel bar-tab-to-folio (placeholder)

**Status:** Speculative — not concretely established in CN-6-001..004 (Mzee Hamisi's pub is standalone, not adjacent to a hotel). Listed for living-catalog completeness. Future activation when a tenant runs pub + hotel adjacency would surface this pattern; mechanism would follow Pattern 1.1's REST7/HOT5 doctrine with `kind: bar_charge`.

### 5.5 Why Charge-Transfer is the canonical bridge

Charge-Transfer is the most common cross-vertical pattern because money is the most common cross-vertical flow. Mama Amina extends to pharmacy → Faraja dispenses → bill clearable via shared tender method? No — same tenant, single Universal Checkout, no cross-vertical bridge needed for that specific case (Mixed-Vertical Tenant). But hotel + restaurant in two separate engines requires Charge-Transfer because the engines are isolated per VE2.

The Obligation primitive's power: it lets two engines coordinate without knowing each other exists.

---

## 6. Pattern Category 2: Item-Transfer via Inventory Primitive

### 6.1 The category

Item-Transfer patterns move a physical or fungible item from one vertical's context (producer/depositor) to another's (consumer/seller). The Inventory primitive's native lot model carries the item.

### 6.2 Pattern 2.1 — Workshop completed item → Retail showroom sale (WS11)

**Established in:** CN-6-004 §20 (full 8-step chain)

**Mechanism (summarised):**
1. Workshop project completes (workshop.project.completed.v1)
2. Operator designates item for retail sale (workshop.deliverable.completed.v1 {inventory_destination: retail_showroom})
3. Foundation Inventory adds lot (inventory.lot.added.v1 {item_ref, source_engine: workshop, category: fabricated_window, site_id: showroom})
4. Retail catalog references the item by item_ref per RE7 (retail handles any item type)
5. Walk-in customer at retail till (retail.basket.item.added.v1 {item_ref})
6. retail.bill.ready.v1 emits
7. Universal Checkout settles
8. Inventory Pattern A deducts (inventory.lot.removed.v1 {consuming_engine: retail})

**Why it works:** Workshop and Retail never reference each other. The Inventory primitive holds the item between them.

### 6.3 Pattern 2.2 — Workshop bucha lots → Restaurant grill consumption (Mzee Karim)

**Established in:** CN-6-001 WP5 + CN-6-002 WP5

**Mechanism:** Mzee Karim's bucha (retail.* operation) receives a ng'ombe carcass as a lot (inventory.lot.added.v1 {item_ref: ng'ombe_carcass, source_engine: retail, lot_metadata: full_carcass_178kg}). Retail sales of cuts to walk-in customers deduct from this lot natively per RE9 bulk-splittable. Mzee Karim's nyama-choma corner (restaurant.* operation) consumes via Pattern B (restaurant.ingredient.consumed.v1 referencing the same carcass lot).

Inventory primitive's native lot offcut model (per CN-5-003 native + WS8 in workshop) lets a single carcass simultaneously serve retail kg-sales AND restaurant grill-consumption. Two verticals reference the same Inventory lot; neither speaks to the other.

### 6.4 The Inventory primitive's lot model as the key enabler

CN-5-003's native lot tracking + RE9 bulk-splittable + RE7 item-type agnosticism together let Item-Transfer patterns work cleanly. Without the lot model, each vertical would need its own item registry — leading to duplication + sync problems + a class of bugs Brief §12 cites as "existing-vertical irregularities."

The Inventory primitive solves Item-Transfer once for all vertical pairs.

---

## 7. Pattern Category 3: Identity-Linking via Party Primitive

### 7.1 The category

Identity-Linking patterns let multiple verticals reference one customer/counterparty identity. The Party primitive holds the identity; verticals reference via `party_ref` in event payloads.

### 7.2 Pattern 3.1 — Customer loyalty across Retail + Restaurant + Hotel

**Established in:** CN-6-102 §11.4 + multiple worked patterns across CN-6-001/002/003

**Mechanism:**
1. Customer (e.g., Mama Halima) is a Party instance — registered once via tenant onboarding or first-visit identification
2. She visits Retail → `retail.customer.identified.v1 {party_ref: <halima>}` + `retail.sale.completed.v1 {payer_party_ref: <halima>}`
3. Universal Promotion subscribes to retail.sale.completed.v1, sees party_ref, accrues loyalty: `promotion.loyalty.earned.v1 {party_ref: <halima>, points}`
4. She dines at Restaurant → `restaurant.customer.identified.v1 {party_ref: <halima>}` + restaurant.bill.ready.v1 with discount_refs including loyalty redeem intent
5. Universal Promotion subscribes to restaurant.bill.ready.v1, sees party_ref + loyalty intent, applies redemption: `promotion.loyalty.redeemed.v1 {party_ref: <halima>, points_consumed}`
6. She checks in at Hotel → `hotel.reservation.confirmed.v1 {party_ref: <halima>}` + hotel.guest_profile.updated.v1 (HOT4 tenant-scope guest profile)

Three verticals; one Party; Universal Promotion as adjudicator; **NO `cross_vertical.*` events.** The loyalty mechanism lives in Universal Promotion; the identity link lives in the Party primitive. Each vertical only references party_ref.

**Worked walkthrough:** WP3 §13.3 demonstrates the full Mama Halima chain per N6.

### 7.3 Pattern 3.2 — Hotel chain guest profile (HOT4 + SP3 tenant-scope)

**Established in:** CN-6-003 §8 (HOT4) + CN-6-103 §6 (SP3 tenant-scope exception)

**Mechanism:** A hotel chain (tenant with multiple sites — Lodge Serengeti + Kilimanjaro Lodge Moshi) maintains a tenant-scope guest profile. Mwemas' fourth visit to either site sees their full chain history. The guest profile is a Workflow instance per HOT4 with `scope_ref: tenant` (SP3 exception per CN-6-103). Each site emits site-scope hotel.reservation.* events; folio close updates tenant-scope hotel.guest_profile.updated.v1.

**Why it's a bridge:** Same vertical (hotel.*) across sites isn't strictly cross-vertical, but the tenant-scope profile bridges site-scope reservations through the shared Party primitive + tenant-scope guest_profile Workflow.

### 7.4 Pattern 3.3 — Workshop B2B customer history (RE6 customer_type)

**Established in:** CN-6-001 RE6 + CN-6-004 §15

**Mechanism:** A B2B customer (e.g., a building contractor) buys from Mzee Hassan's workshop repeatedly. Each project references the same `party_ref` with `customer_type: b2b` + `negotiated_pricing_tier_ref`. The workshop's project history per party is queryable via Party-anchored projection. If the contractor also buys from Mzee Hassan's retail showroom (Mixed-Vertical), the same party_ref appears in retail.sale events — workshop and retail see the customer as one Party instance.

### 7.5 Party primitive as the universal identity layer

Identity-Linking is the cleanest cross-vertical pattern because the Party primitive is by design the universal identity layer. Every vertical references parties via party_ref; cross-vertical aggregation happens at read-time via Party-anchored projections (per HO8 — verticals read Foundation primitive projections, not other verticals' state).

---

## 8. Pattern Category 4: Document-Sharing via Document Primitive

### 8.1 The category

Document-Sharing patterns let multiple verticals at one tenant reference the same compliance or configuration document. The Document primitive (CN-4-012) holds the document instance; verticals reference via document_ref.

### 8.2 Pattern 4.1 — Tenant license cross-vertical (alcohol license)

**Established in:** CN-6-002 §16 + CN-6-001 (alcohol pack flags)

**Mechanism:** A tenant operating both Pub (restaurant.* bar_only config) + Restaurant (restaurant.* table_service config serving wine) + Retail (selling packaged beer) holds one alcohol license per regulatory authority. The license is captured at tenant activation per CTR-045 as a Document; all three verticals reference the same `document_ref` via their `regulatory_evidence_refs[]` payload field. No per-vertical license duplication.

When license expires, all three verticals' advisors surface alerts referencing the same document. When license renews, the new Document version supersedes; all three verticals see the renewal.

### 8.3 Pattern 4.2 — Tax compliance shared (CN-5-105 N7 tenant_tax_profile)

**Established in:** CN-5-105 N7 + cited across CN-6-001/002/003/004

**Mechanism:** Tenant tax registration status (VAT registered? threshold tier? excise applicability?) is one tenant-property record per CTR-027. All verticals reference the same `tenant_tax_profile` at command-time via `pack.tax.lookup(item_category, tenant_tax_profile, business_date)`. Mama Amina expanding from retail to retail+pharmacy doesn't re-register VAT — the same tenant_tax_profile applies to both verticals' tax computations.

### 8.4 Pattern 4.3 — Workshop style export/import (cross-tenant)

**Established in:** CN-6-004 §18 WS10

**Mechanism:** Workshop Style exported as Document with content hash; transferred between tenants (out-of-BOS file transfer); imported into receiver via workshop.style.imported.v1. This is **cross-tenant Document-Sharing**, distinct from within-tenant cross-vertical. Included for completeness; cross-tenant strict pattern.

The Document primitive's hash-verifiable nature makes safe transfer possible — the receiver knows the content integrity matches the sender's export.

### 8.5 Document primitive as the compliance layer

Documents are the compliance + configuration layer; sharing them across verticals is natural because compliance is cross-cutting (alcohol affects pub + restaurant + retail; tax affects everything; certifications affect any vertical that handles regulated items).

---

## 9. Cross-Vertical Workflow Coordination (Edge Case)

### 9.1 The edge case

Most workflow coordination is intra-vertical. Restaurant table_session orchestrates its own kitchen tickets; Hotel folio orchestrates its own charge accumulation. True cross-vertical workflow coordination — where Vertical A's workflow gates on Vertical B's workflow state — is rare.

When it does happen, it reduces to the Obligation pattern. There is no separate "Workflow-Coordination primitive" — Workflow primitive itself is intra-vertical; cross-vertical coordination uses Obligation.

### 9.2 N1 Concrete example — Hotel reservation gating on Restaurant special-meal pre-arrival request

A guest (Mwemas, returning to Lodge Serengeti for their 5th stay) requests a special-meal pre-arrival commitment: "Please prepare gluten-free dinner for our arrival night." The Hotel reservation needs to gate confirmation on Restaurant's commitment to honour the request (some lodges may decline if dietary requirements exceed kitchen capacity that week).

**The wrong approach** (direct workflow link): Hotel reservation subscribes to `restaurant.kitchen.capacity.v1` — VE2 violation.

**The right approach** (Obligation primitive):

1. Hotel reservation emits an Obligation: `obligation.created.v1 {kind: pre_arrival_meal_request, debtor_party_ref: <guest>, creditor_engine: hotel, counterparty_workflow_ref: <restaurant_capacity_workflow>, payload_ref: <meal_specs>}`
2. Restaurant's capacity Workflow (a tenant-scope intra-Restaurant workflow that tracks weekly capacity bookings) subscribes to `obligation.created.v1` filtered by `kind == "pre_arrival_meal_request"` + counterparty
3. Restaurant capacity Workflow assesses; either accepts (emits `obligation.committed.v1`) or declines (emits `obligation.rejected.v1`)
4. Hotel reservation Workflow subscribes to the relevant Obligation lifecycle events; transitions reservation per response

**The Obligation kind here is `pre_arrival_meal_request`** — semantically different from `hospitality_charge` (charge-transfer) but mechanism-identical (Obligation primitive carries cross-vertical interaction).

### 9.3 Why most coordination doesn't need this

Most cross-vertical interactions are settlement-driven, not coordination-driven. Restaurant fires kitchen tickets without asking Hotel anything. Hotel checks guests in without asking Restaurant. The interactions emerge at the **charge** boundary (Pattern Category 1), not at the workflow boundary.

True workflow coordination (one workflow gating on another's commitment) is the edge case; when it happens, Obligation primitive handles it via semantic `kind` field.

---

## 10. Anti-Patterns Catalog — AP-B1 to AP-B6

Doctrine teaches better through counter-example. These six anti-patterns consolidate scattered anti-patterns from CN-6-100..104 + CN-6-101 §10 and explicitly document what NOT to do in cross-vertical scenarios.

### 10.1 AP-B1 — Direct vertical-to-vertical event subscription

**Description:** A vertical engine's subscription manifest includes events from another vertical's namespace. Example: Hotel manifest includes `subscribes_to: restaurant.bill.ready.v1`.

**Why it's wrong:** Violates VE2 (engine isolation) + BD7 (cross-vertical must route through Foundation primitives). Couples the two verticals at event schema level; restaurant's bill schema change breaks hotel; decommissioning restaurant breaks hotel.

**Right pattern:** Use Obligation primitive (BR3 Charge-Transfer category) — restaurant emits Obligation; hotel subscribes to `obligation.created.v1` filtered by kind. **Mechanizable per CTR-046** (parallel CN-6-101 AP2 + DC-NN check): doctrine gate at CN-4-020 registration rejects manifests with `<other_vertical>.*` in subscribes_to.

**Source:** VE2 + CN-6-101 AP2 + BR1.

### 10.2 AP-B2 — Vertical reads other vertical's engine state via API/synchronous call

**Description:** Vertical A queries Vertical B's projection or state via a synchronous read (REST call, direct DB query, RPC).

**Why it's wrong:** Violates HO8 (vertical reads only Foundation primitive projections + own engine projections) + Charter §1.3 (legal-defensibility through event-store communication) + Law 2 (engine isolation). Creates runtime coupling; B's downtime affects A; B's schema change breaks A.

**Right pattern:** Read via Foundation primitive projections — Party for customer state, Obligation for cross-vertical settlement state, Document for shared documents, Inventory for items. If the data needed isn't carried by an existing primitive, the design is wrong — either the data should be in a primitive (push down per BD4) or the interaction should be asynchronous via event subscription.

**Mechanizability:** Lower than AP-B1 (cannot statically detect runtime API calls); architectural review at CN-4-020 registration + Term 7 coherence verification per CTR-010.

**Source:** Charter §1.3 + Law 2 + HO8 + N5 of CN-6-104.

### 10.3 AP-B3 — Embedded foreign vertical event_id in payload

**Description:** Vertical A's event payload directly references another vertical's event_id. Example: `pharmacy.bill.ready.v1 {customer_history_event_id: "retail.sale.completed.v1#evt-77321"}`.

**Why it's wrong:** Per CN-6-102 §11.3 N3 — couples to other vertical's event schema, version, and lifecycle; the referenced event might be compensated; cross-vertical event_id assumes other vertical's namespace stays stable forever.

**Right pattern:** Reference via Foundation primitive ID instead:
- ✓ `party_ref: <halima>` (query-time resolution via Party projection)
- ✓ `obligation_ref: <obl-91>` (event-time relationship via Obligation primitive)
- ✗ `retail_sale_event_id: <evt>` (direct cross-vertical event coupling)

**Mechanizability:** Doctrine gate can check payload schemas for `<other_vertical>_event_id` field patterns; emit warnings during registration. Higher mechanizability than AP-B2.

**Source:** CN-6-102 §11.3 N3.

### 10.4 AP-B4 — Bridge engine creation

**Description:** A new engine specifically to mediate between two verticals. Example: proposing a "HospitalityBridge" engine that knows both restaurant bills and hotel folios.

**Why it's wrong:** Violates BD7 (no bridge engines). The "bridge engine" would have its own state; duplicate Accounting/Cash subscriptions; couple verticals that should stay isolated; create maintenance burden across vertical evolution.

**Right pattern:** Foundation primitive carries the bridge. Restaurant emits Obligation; Hotel subscribes to Obligation. No bridge engine — just primitive + two verticals.

**Concrete BOS examples avoided:**
- BOS resisted "HospitalityBridge" temptation; Obligation primitive carries hotel↔restaurant charges per Pattern 1.1
- BOS resisted "ManufacturingRetailBridge" temptation; Inventory primitive carries workshop↔retail items per Pattern 2.1
- BOS resisted "LoyaltyBridge" temptation; Party primitive + Universal Promotion carry cross-vertical loyalty per Pattern 3.1

**Mechanizability:** Hard to mechanize statically (a proposed engine looks like any engine). Review-checklist enforcement at architecture review per CN-6-101 §10 AP3.

**Source:** BD7 + CN-6-101 §10 AP3 + BR1/BR2.

### 10.5 AP-B5 — Foreign primitive (inventing a primitive specifically for one cross-vertical pattern)

**Description:** Creating a new Foundation primitive specifically to handle one cross-vertical pattern, instead of reusing existing primitives. Example: proposing a "HospitalityCharge" primitive to specifically handle restaurant↔hotel charges.

**Why it's wrong:** Violates BD4 (push-down: reuse existing primitives before introducing new ones). The existing Obligation primitive already handles via `kind: hospitality_charge`. Adding a new primitive bloats Foundation; future cross-vertical patterns would each demand their own primitive.

**N4 Concrete example — "HospitalityCharge primitive" would violate AP-B5:**

Proposed (wrong): "We need a new primitive `HospitalityCharge` to represent charges that flow from restaurant to hotel folio. It has its own lifecycle (issued → folio-absorbed → settled), its own audit chain, its own resolution semantics."

Correct alternative: Use existing Obligation primitive with `kind: hospitality_charge`. All the proposed lifecycle states map to existing Obligation lifecycle (created → committed → settled). The audit chain comes from Obligation primitive's hash chain (CN-4-003). Settlement semantics come from Obligation's settlement subscription.

**Right pattern:** Before introducing a primitive, exhaust existing primitives. The five (Obligation, Inventory, Party, Document, Workflow) cover almost every cross-vertical pattern.

**Mechanizability:** Architectural review at primitive-proposal time (Term 4 territory). Any proposal for a new primitive must explicitly demonstrate why existing five cannot handle the case.

**Source:** BD4 + BR3 + Foundation primitive catalog discipline.

### 10.6 AP-B6 — Side-channel coordination (non-event-store communication)

**Description:** Verticals coordinate via mechanisms outside the event store: shared database tables, file system communication, in-memory message buses, direct RPC, shared cache.

**Why it's wrong:** Violates Law 1 (state derived from events only) + Charter §1.2 (legal-defensibility through event chain) + audit replay determinism. Side-channel communication is invisible to event-sourced reconstruction; if a workshop-retail coordination uses a shared file, replay can't reproduce the state.

**Right pattern:** All cross-vertical coordination via event store — Foundation primitive events as carriers. If a use case seems to require side-channel, the design needs revision (probably an Obligation primitive instance carries the coordination).

**Mechanizability:** Low — side-channel communication is by definition outside event-store visibility; detection requires architectural review + runtime monitoring of cross-vertical traffic patterns.

**N2 Mechanizability summary** for the catalog:
- **High mechanizability (CTR-046 candidate DC checks):** AP-B1 (subscribes_to check), AP-B3 (payload field pattern check)
- **Medium mechanizability:** AP-B4 (manifest analysis at architecture review), AP-B5 (primitive proposal review process)
- **Low mechanizability:** AP-B2 (runtime API call detection), AP-B6 (side-channel detection — architectural review primarily)

**Source:** Law 1 + Charter §1.2 + audit replay invariants.

---

## 11. Cross-Tenant Bridges Deferred (Q4 Closure)

### 11.1 The cross-tenant case

CN-6-005 v1 catalogs **within-tenant** cross-vertical bridges. A separate class — **cross-tenant bridges** — would handle interactions between tenants:
- Mama Amina pharmacy supplies Lodge Serengeti via wholesale relationship
- Mzee Hassan workshop fulfils orders for a separate retail chain tenant
- Mzee Karim bucha sells to a separate hotel kitchen tenant

These cross-tenant patterns require additional governance: who owns the inter-tenant Document? How are commissions calculated? Who is the "responsible regional agent" per Charter Law 6 when tenants span regions?

### 11.2 N5 — Term 1 marketplace future framing

Cross-tenant bridges are deferred per BD8 (≥2 concrete cases needed before introducing new framework). When activated, they will involve:

| Concern | Owner |
|---------|-------|
| Commission + revenue-share mechanics | Term 1 — platform marketplace governance + platform-scope financial roll-up |
| Inter-tenant Documents (purchase orders, invoices crossing tenant boundary) | Term 4 Document primitive extension + Term 1 governance |
| Cross-tenant Party (supplier-customer relationship spanning tenants) | Term 4 Party primitive metadata + Term 1 governance for cross-tenant identity |
| Cross-tenant Obligation (settlement spanning tenants) | Term 4 Obligation primitive extension if needed + Term 1 marketplace contracts |
| Cross-tenant Inventory (item transfer between tenant inventories) | Term 4 Inventory Movement primitive extension + Term 7 integration for transfer mechanics |

**Future CTR to Term 1** will be filed when marketplace activation begins. This is post-Concept-Phase work — possibly Phase 2 of BOS Concept Phase or implementation phase.

### 11.3 Why BD8 applies

BD8 — future-vertical hypothetical doesn't justify premature Foundation/Universal placement. Cross-tenant marketplace falls under the same discipline: until ≥2 concrete African SME use cases demonstrate cross-tenant business relationships needing platform mediation (vs. handled out-of-BOS like normal supplier relationships), the marketplace framework stays unbuilt.

Current within-tenant cross-vertical patterns (CN-6-005 §5-§9) cover the established needs.

---

## 12. The Living-Catalog Mechanism

### 12.1 The amendment gate (N3)

Per BR5 + Charter §11 living-document principle. Adding a new pattern or anti-pattern to CN-6-005 follows the gate:

1. **Term 6 proposes** — typically when authoring a new vertical doc that establishes a new bridge pattern; or during architecture review of a proposed pattern that emerged from field implementation
2. **Overseer (Term 7) reviews** — verifies BR1-BR5 doctrine holds; checks pattern doesn't violate AP-B1..AP-B6; confirms primitive carrier is correctly identified
3. **Concept Lead ratifies** — final approval
4. **CN-6-005 amended** — pattern added to appropriate §5-§9 category + CHANGELOG entry

### 12.2 Pattern removal — never

Patterns are never removed from the catalog. Patterns may be **deprecated** (marked as no longer recommended; preserved for audit) but never deleted. This preserves the audit trail of how the catalog evolved and prevents removal of a pattern that some implementation still references.

### 12.3 Anti-pattern additions

New anti-patterns (AP-B7+) follow the same gate. Anti-patterns typically emerge from field practice — an Architect observes a wrong implementation, traces the root cause, identifies the doctrinal violation. The lesson goes into CN-6-005 §10.

### 12.4 CTR requirement (when)

A catalog amendment **does not** require a CTR if it only adds patterns using existing Foundation primitives. A CTR is required only if the new pattern requires Foundation primitive extension (rare). Most catalog additions are doc amendments + CHANGELOG only.

### 12.5 Coherence with CN-4-019 living doctrine catalog

CN-4-019 (Foundation's living doctrine catalog) is the parallel mechanism for doctrine-check additions. CN-6-005 is to bridge patterns what CN-4-019 is to doctrine checks — both are living catalogs maintained as systems evolve.

---

## 13. Worked Patterns — Five Cross-Vertical Walkthroughs

### 13.1 WP1 — Hotel↔Restaurant charge-to-room full chain (REST7 + HOT5)

**Anchor:** Mwemas (Mama na Bwana Mwema, cross-doc canonical hotel guests) at Lodge Serengeti on the second night of their stay. They dine at the lodge restaurant; charge to room.

**Setup:** Lodge Serengeti tenant runs hotel + restaurant. `pack.hotel.charge_to_room_enabled: true`; `pack.restaurant.hotel_charge_to_room_enabled: true`.

**Full chain (from CN-6-002 §13 + CN-6-003 §11):**

1. Mwemas finish dinner. Restaurant emits:
   ```
   restaurant.bill.ready.v1 {
     bill_id, site_id, saleable_lines,
     payment_method_hint: charge_to_room,
     payer_party_ref: <mwemas>,
     obligation_emission: true
   }
   ```

2. Universal Checkout routes per hint; emits:
   ```
   obligation.created.v1 {
     obligation_id: "obl-restaurant-dinner-night2",
     kind: hospitality_charge,
     debtor_party_ref: <mwemas>,
     creditor_engine: restaurant,
     creditor_workflow_ref: <restaurant_table_session>,
     amount: 65000,
     counterparty_workflow_ref: <mwemas_folio>,
     payload_ref: <restaurant.bill.ready event_id>,
     business_date
   }
   ```

3. Hotel folio Workflow subscribes (filter: `kind == "hospitality_charge"` AND `counterparty_workflow_ref == <self>`); emits:
   ```
   hotel.folio.charge_added.v1 {
     folio_id, source_engine: restaurant,
     obligation_ref: "obl-restaurant-dinner-night2",
     line_amount: 65000,
     line_description: "Lodge Serengeti Restaurant — dinner Night 2",
     included_obligation_refs.appended_with: ["obl-restaurant-dinner-night2"]
   }
   ```

4. (Days later) Mwemas check out. Hotel folio aggregates all charges including restaurant obligations:
   ```
   hotel.folio.ready.v1 {
     folio_id, reservation_ref,
     saleable_lines: [room_nights, restaurant_dinner_night2, mini_bar, ...],
     obligation_refs: ["obl-restaurant-dinner-night2", ...],
     ...
   }
   ```

5. Universal Checkout settles via Mwemas' chosen tender method (per RE11 abstract). `checkout.settled.v1` arrives carrying obligation_refs.

6. Foundation Obligation primitive resolves per obligation_ref:
   ```
   obligation.settled.v1 {
     obligation_id: "obl-restaurant-dinner-night2",
     resolved_via: <checkout.settled event_id>,
     resolved_amount: 65000,
     business_date
   }
   ```

7. Restaurant table_session Workflow subscribed to obligation.settled.v1 filtered by kind + creditor_engine; transitions billing → completed; emits `restaurant.commission_earned.v1` for that night's waiter; Accounting recognises restaurant revenue.

**What this proves:** Restaurant and Hotel never reference each other's events. The Obligation primitive is the coordinator. Either vertical can decommission without breaking the other. Six steps; full audit chain end-to-end.

**Primitive carrier:** Obligation primitive. **Pattern category:** Charge-Transfer (§5 Pattern 1.1).

### 13.2 WP2 — Workshop sell-via-Retail showroom (WS11)

**Anchor:** Mzee Hassan's Arusha showroom has 6 sample windows; a walk-in contractor wants to buy one on the spot.

**Setup:** Mzee Hassan's tenant runs both workshop.* AND retail.* (Mixed-Vertical pattern). `pack.workshop.inventory_destination_retail: enabled`.

**Full chain (from CN-6-004 §20 — 8 steps):**

1. **Workshop project completes:** A previous casement project completed; one extra window manufactured for showroom variety. `workshop.project.completed.v1` emitted earlier.

2. **Item designated for retail:** Mzee Hassan marks the window for showroom sale:
   ```
   workshop.deliverable.completed.v1 {
     project_ref, item_ref: <window-arusha-001>,
     inventory_destination: retail_showroom,
     business_date
   }
   ```

3. **Foundation Inventory adds lot:**
   ```
   inventory.lot.added.v1 {
     item_ref: <window-arusha-001>,
     source_engine: workshop,
     source_workflow_ref: <project>,
     category: "fabricated_casement_window_140x201",
     quantity: 1,
     site_id: <showroom>,
     business_date
   }
   ```

4. **Retail catalog references:** Mzee Hassan adds a retail catalog entry referencing the item via item_ref + sets price 800,000 TZS.

5. **Walk-in customer at retail till:** Contractor walks in; Salma (cashier) scans the showroom tag:
   ```
   retail.basket.item.added.v1 {
     basket_id, item_ref: <window-arusha-001>,
     quantity: 1, unit_price: 800000,
     site_id: <showroom>
   }
   ```

6. **retail.bill.ready.v1 emits:** Standard retail bill with the window line.

7. **Universal Checkout settles** via contractor's chosen tender method (RE11 abstract).

8. **Inventory Pattern A deduction:**
   ```
   inventory.lot.removed.v1 {
     item_ref: <window-arusha-001>,
     consuming_engine: retail,
     consuming_workflow_ref: <retail_sale>,
     quantity: 1,
     ...
   }
   ```

The contractor takes the window home. Workshop and Retail never referenced each other.

**Primitive carrier:** Inventory primitive (lot model). **Pattern category:** Item-Transfer (§6 Pattern 2.1).

### 13.3 WP3 — Mama Halima cross-doc loyalty linking (THREE verticals, ONE Party) — N6 concrete chain

**Anchor:** Mama Halima the Dar es Salaam freight broker (cross-doc canonical from CN-6-100 Logistics + CN-6-102/103/004 customer scenarios) is a loyalty member at a multi-vertical tenant (Lodge Serengeti also operates retail + restaurant; OR she touches three separate tenants each with loyalty — for this WP, three separate tenants linked by shared Universal Promotion via Party primitive at platform identity layer).

For simplicity in WP3, assume Mama Halima visits THREE touchpoints, each a distinct tenant, but all participating in a shared loyalty program (this models cross-tenant identity through Party primitive even where verticals are at different tenants — the Party primitive's tenant-scope is by default, but cross-tenant identity via referral codes / shared loyalty exists per future Term 1 marketplace; for this WP, assume one mega-tenant with Mixed-Vertical retail + restaurant + hotel for clean illustration).

**Mega-tenant scenario:** "BOS Hospitality Group" runs Mama Amina's Kariakoo duka (retail.*) + Lodge Serengeti restaurant (restaurant.*) + Lodge Serengeti hotel (hotel.*). All three verticals at one tenant. Universal Promotion adjudicates loyalty per Party primitive linking.

**Full chain (N6 concrete):**

**Week 1 — Kariakoo duka:**
1. Mama Halima visits Mama Amina's; identifies via loyalty card: `retail.customer.identified.v1 {party_ref: <halima>}`
2. She buys 5kg rice + 2L cooking oil + 6 bars soap: `retail.sale.completed.v1 {payer_party_ref: <halima>, total: 18500 TZS}`
3. Universal Promotion subscribes; emits: `promotion.loyalty.earned.v1 {party_ref: <halima>, points: 185}` (10% of bill in points)

**Week 3 — Lodge Serengeti restaurant (Mama Halima happens to drive Dar→Mwanza route stopping at Serengeti for a lunch):**
4. She identifies at restaurant: `restaurant.customer.identified.v1 {party_ref: <halima>}`
5. She finishes lunch; emits: `restaurant.bill.ready.v1 {payer_party_ref: <halima>, saleable_lines: [...], discount_refs: [{layer: loyalty, intent: redeem, max_points: 100}]}`
6. Universal Promotion subscribes; checks party_ref's loyalty balance (185 points from Week 1); approves redemption of 100 points; emits: `promotion.loyalty.redeemed.v1 {party_ref: <halima>, points_consumed: 100, discount_amount: 1000 TZS}`
7. Checkout settles with discount applied; remaining loyalty balance: 85 points

**Week 5 — Lodge Serengeti hotel:**
8. Mama Halima decides to stay overnight at the lodge before continuing to Mwanza; she identifies at front desk: `hotel.reservation.confirmed.v1 {party_ref: <halima>, ...}`
9. Hotel guest profile Workflow (HOT4 tenant-scope) subscribes to the reservation event; emits: `hotel.guest_profile.updated.v1 {party_ref: <halima>, stay_count_incremented: 1, ...}`
10. Hotel applies returning-customer tier per HOT9 rate card based on loyalty balance + total spend across tenant (from Party-anchored projection) — discount applied to room rate
11. At checkout, hotel.folio.ready.v1 with rate card-discounted lines; Universal Promotion may earn additional loyalty points on the stay

**What this proves:**

- Three verticals (retail + restaurant + hotel)
- ONE Party (Mama Halima as `<halima>` ref)
- Universal Promotion as the cross-vertical adjudicator
- Hotel's guest profile aggregates across verticals via Party-anchored projection
- **NO `cross_vertical.*` events anywhere**
- Each vertical only emits its own namespace events + references party_ref

**Primitive carrier:** Party primitive (identity link) + Universal Promotion (adjudication). **Pattern category:** Identity-Linking (§7 Pattern 3.1).

**Cross-doc continuity:** Mama Halima appears in CN-6-100 (Logistics owner) + CN-6-001/002/003 (customer scenarios) + CN-6-004 (workshop customer in different facet). Her identity persists across all vertical interactions because the Party primitive holds it.

### 13.4 WP4 — Mama Amina retail+pharmacy Mixed-Vertical loyalty (re-anchored)

**Anchor:** Mama Amina's Kariakoo duka with Faraja's pharmacy counter expansion (CN-6-101 §11.6 + CN-6-102 §11.4). A customer (could be Mama Halima from WP3 or any regular) buys OTC paracetamol at the front retail till + fills a prescription at the back pharmacy counter — two separate transactions at the same tenant.

**Established mechanism (cross-reference CN-6-102 §11.4):**

1. Customer buys OTC paracetamol at retail till: `retail.sale.completed.v1 {payer_party_ref: <customer>}`
2. Universal Promotion accrues loyalty per retail sale
3. Customer moves to Faraja's pharmacy counter; presents prescription: `pharmacy.prescription.validated.v1 {party_ref: <customer>}`
4. Faraja dispenses; emits: `pharmacy.bill.ready.v1 {payer_party_ref: <customer>, ..., discount_refs: [{layer: loyalty, intent: redeem, max_points: 100}]}`
5. Universal Promotion adjudicates; same party_ref + same loyalty balance + applies redemption: `promotion.loyalty.redeemed.v1`
6. Pharmacy bill settled with discount

**What this proves:**

- Retail and Pharmacy never reference each other
- Same Party primitive instance links them
- Universal Promotion adjudicates cross-vertical loyalty per identity
- Two verticals; one Party; clean separation

**Primitive carrier:** Party primitive. **Pattern category:** Identity-Linking + Item-Transfer (OTC items via retail catalog reference Inventory lots; prescription items via pharmacy.* path). Multi-category demonstration.

### 13.5 WP5 — Mzee Karim bucha+BBQ inventory cross-feed

**Anchor:** Mzee Karim's bucha in Kariakoo (retail.*) + nyama-choma BBQ corner (restaurant.*) at the same tenant. A full ng'ombe carcass arrives; serves both retail kg-sales AND restaurant grill consumption.

**Established mechanism (cross-reference CN-6-001 WP5 + CN-6-002 WP5):**

1. Carcass arrives: `inventory.lot.added.v1 {item_ref: ng'ombe_carcass_001, source: supplier_delivery, lot_metadata: full_carcass_178kg, category: meat_aluminium}`
2. **Retail side** — walk-in customer buys 1.5kg fillet via retail till:
   ```
   retail.basket.item.added.v1 {item_ref: ng'ombe_carcass_001, quantity_measure: 1.5, measurement_unit: kg, ...}
   ```
   ```
   retail.sale.completed.v1 (per Pattern A) → inventory.lot.removed.v1 {item_ref: ng'ombe_carcass_001, consuming_engine: retail, quantity: 1.5}
   ```
   Inventory primitive's lot now: 176.5kg remaining
3. **Restaurant side** — order grilled fillet plate at BBQ corner:
   ```
   restaurant.order.placed.v1 → restaurant.grill.ticket.fired.v1 → restaurant.ingredient.consumed.v1 (Pattern B per CN-5-003 N1)
   ```
   The ingredient consumed event references the same `item_ref: ng'ombe_carcass_001` with `quantity: 0.25kg` (a quarter-kilo plate-sized fillet portion).
   ```
   inventory.lot.removed.v1 {item_ref: ng'ombe_carcass_001, consuming_engine: restaurant, quantity: 0.25}
   ```
   Inventory primitive's lot now: 176.25kg remaining
4. Over the day, the carcass serves dozens of retail walk-ins AND a steady stream of BBQ orders. Both verticals reference the same lot; neither speaks to the other.

**What this proves:**

- Inventory primitive's native lot model handles cross-vertical lot sharing
- Bucha (retail) and BBQ (restaurant) at one tenant access the same carcass
- Two verticals; one Inventory lot; no cross-vertical event subscriptions
- The lot's `quantity` decrements correctly regardless of which engine consumes

**Primitive carrier:** Inventory primitive (lot model). **Pattern category:** Item-Transfer (§6 Pattern 2.2).

---

## 14. Boundaries + Open Items + Cross-Term Hooks

### 14.1 CN-6-005's place in the corpus

| Concern | Owned by | CN-6-005 role |
|---------|----------|----------------|
| Cross-vertical pattern catalog (within-tenant) | **CN-6-005** (this doc) | Authoritative |
| Cross-cutting vertical framework | CN-6-100..104 | Parents — CN-6-005 reaffirms VE2 + BD7 |
| Concrete vertical engines | CN-6-001..004 | Sources of established patterns — CN-6-005 catalogs |
| Foundation primitives (the carriers) | CN-4-011 + CN-4-012 | Foundational — CN-6-005 categorizes by carrier |
| Universal layer (subscribers + adjudicators) | CN-5-001/003/007/105 | Consumers — Universal Promotion adjudicates Identity-Linking; Inventory carries Item-Transfer |
| Mixed-Vertical Tenants (the tenant activation side) | CN-6-105 future | Sibling — CN-6-005 covers patterns; CN-6-105 covers how tenants activate ≥2 verticals |
| Future-vertical stress sketches | CN-6-901..904 future | Will surface new patterns to add to living catalog per BR5 + §12 |
| Cross-tenant marketplace | Term 1 future | Deferred per §11 + N5; not in scope v1 |
| Doctrine check additions for AP-B1/B3 mechanizable rows | Term 4 + CTR-046 | Existing CTR-046 queue (DC-NN-e + DC-NN-f); future expansion possible per AP-B catalog |

### 14.2 CTRs (no new)

- CTR-018/002/024/026/027/030/038/044/045/046/049/050/028/006/027 — cited as-is
- **Cumulative CTR-046 expansion queue (unchanged):** DC-NN-e + DC-NN-f
- Future cross-tenant marketplace work (§11 + N5) will file CTR to Term 1 when activated

### 14.3 Open items inside Term 6 scope

- **CN-6-105 Mixed-Vertical Tenants** — next doc per Brief §13.11; covers tenant activation patterns for ≥2 verticals; uses CN-6-005's bridge patterns as the cross-vertical mechanism layer
- **CN-6-901..904 stress sketches** — Brief §13.12 Flexibility Test; each sketch will likely surface new bridge patterns to add via §12 living-catalog mechanism (e.g., Insurance↔Hotel for guest insurance; Healthcare↔Pharmacy for prescription routing)
- **Cross-tenant marketplace** — deferred per BD8 + Q4 + N5; future Term 1 work
- **Anti-pattern mechanizability** — AP-B1 + AP-B3 candidates for CTR-046 doctrine check expansion (parallel DC-NN-e + DC-NN-f)
- **Workflow-Coordination edge case** — only one concrete example documented (N1 pre-arrival meal request); future verticals may surface more cases that warrant elevating to its own pattern category

### 14.4 D-DISC cross-references

- **D-DISC-001 — tenant-customer promotion UX**: WP3 Mama Halima loyalty linking visible across retail + restaurant + hotel surfaces; Term 3 designs how cross-vertical loyalty balance appears in each context
- **D-DISC-002 — POS self-service expansion**: Self-service touchpoints across verticals (mobile app for retail RE10 + restaurant REST8 QR + hotel HOT7 check-in) all use same Party primitive for customer-as-actor; bridge patterns from CN-6-005 carry across self-service contexts

### 14.5 The bar — Bridges are how the framework breathes

The four concrete verticals (CN-6-001..004) operate isolated. They emit, they subscribe, they own their workflows. But real businesses don't operate in vertical isolation — Mama Halima visits multiple verticals as one customer; Mzee Hassan's workshop and showroom serve one customer base; Lodge Serengeti's hotel and restaurant aggregate one guest's full stay experience.

The bridges in CN-6-005 are how the framework breathes across vertical boundaries while preserving isolation. The Obligation primitive carries charges; the Inventory primitive carries items; the Party primitive carries identity; the Document primitive carries compliance. Five Foundation primitives serve as the universal interconnect layer; verticals reference primitives, not each other.

The catalog is alive — it grows as new verticals arrive. When CN-6-901..904 stress sketches surface Insurance↔Hotel guest-insurance patterns, Healthcare↔Pharmacy prescription routing, Education↔Marketing student-recruitment flows, CN-6-005 §12 amendment gate adds them. The framework absorbs new patterns without architectural change because the Foundation primitive layer is generic enough.

**The framework breathes. Patterns 12 + counter-patterns 6 = doctrine made operational.**

Next: CN-6-105 Mixed-Vertical Tenants — the tenant activation side, where bridges meet tenant lifecycle.

---

*— End of CN-6-005 Vertical Bridges (Pattern Catalog) v1 —*
