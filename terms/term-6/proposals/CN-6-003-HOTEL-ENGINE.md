# CN-6-003 — Hotel Engine (Hospitality Stays Cluster)

> **Term:** 6 — Vertical Engines
> **Status:** Concept Phase v1.0 — Draft for Overseer review
> **Branch:** `claude/brave-johnson-S7Lky`
> **Reading order:** CN-6-100..104 → CN-6-001 (Retail; RE11 payment abstraction) → CN-6-002 (Restaurant; REST7 charge-to-room) → CN-5-003/009/100/103/105 → CN-4-011/012/021 → **this doc**
> **Ordering authority:** TERM-6 Brief §13 — **third concrete vertical doc**; first application of Hospitality Stays cluster doctrine (parallel F&B); confirms Hotel side of CN-6-002 §13 REST7 cross-vertical pattern.

---

## 1. Purpose & Boundary

### 1.1 Mission

CN-6-003 declares the **Hotel Engine** — the third concrete vertical, covering the **Hospitality Stays cluster** under the namespace `hotel.*`. Per the cluster doctrine ratified between CN-6-002 (F&B) and this doc, `hotel.*` is the canonical namespace per CTR-038 + CN-5-103 §16; the doctrinal scope covers the full Hospitality Stays spectrum: hotel (full-service), lodge (safari/eco), guesthouse / B&B, tented camp, serviced apartment, hostel, short-term rental.

One Hospitality Stays vertical mechanism (BD5 split-it) + per-style pack configuration absorbs the cluster. Bibi Sauda's small Iringa guesthouse and Kilimanjaro Lodge Moshi (a multi-room mid-range lodge serving Mt. Kilimanjaro trekkers) and Lodge Serengeti (a high-end safari lodge serving game-drive guests) all share the same engine — different in scale, configuration, and service style; same in business doctrine.

Per Brief §6.1: *"Rooms, reservations, check-in/out, folios, housekeeping."* This doc operationalises that plus the cross-vertical realities: confirming REST7 charge-to-room from the Hotel side (mirror of CN-6-002 §13), multi-day folio aggregation across all stay-charge sources, room state as Workflow primitive instance (not Inventory item — N1 boundary), and chain guest profile as tenant-scope SP3 exception.

### 1.2 DOES vs DOES NOT

| CN-6-003 DOES | CN-6-003 DOES NOT |
|----------------|--------------------|
| Declare `hotel.*` covering Hospitality Stays cluster (hotel + lodge + guesthouse + tented camp + serviced apartment + hostel + short-term rental) | Author cross-cutting framework or replicate CN-6-001/002 mechanics |
| Specify Hospitality Stays Workflows: `hotel.reservation`, `hotel.folio`, `hotel.room`, `hotel.guest_profile`, `hotel.rate_card_management` | Specify Universal Checkout flow (CN-5-009) |
| Confirm REST7 Hotel-side Obligation subscription for charge-to-room | Re-state REST7 Restaurant-side emission (CN-6-002 §13) |
| Treat room as Workflow primitive instance per VE7 (NOT Inventory item) — N1 boundary | Conflate room with Inventory item (CN-5-003 has Inventory primitive; rooms are bookable resources, distinct concept) |
| Distinguish folio (per-stay site-scope) from guest profile (tenant-scope SP3 exception) | Treat them as one concept |
| Place rate management as pack content (HOT9; parallel REST4 recipes-as-pack-content) | Create rate Foundation primitive |
| Document the BOS-is-not-licensing-authority doctrine for tourism licensing (parallel CN-6-002 §16) | Issue, verify, or autonomously gate based on tourism licensing |
| Inherit RE11 payment-method abstraction throughout (zero provider names) | Mention specific tender providers anywhere |
| Honour zero-new-CTR goal | Open new CTRs |

### 1.3 Audience

Term 6 itself (CN-6-004 author follows pattern); Architects implementing the hotel engine across the Hospitality Stays spectrum; Term 1 onboarding governance authoring tourism licensing capture; Term 3 designing receptionist + guest mobile UX; Term 7 confirming self-service check-in channel adapters per CTR-050; future tenants across hospitality cluster.

### 1.4 Charter Compliance

| Law | How CN-6-003 honours it |
|-----|--------------------------|
| Law 1 — State from events only | All hotel state via event store; folio is event-aggregated; reservation immutable, modifications via compensation events |
| Law 2 — Engines isolated | REST7 Obligation primitive carries Restaurant cross-vertical; never direct subscription to restaurant.* |
| Law 3 — AI advisory only | Hotel-floor + housekeeping + manager advisors per CN-5-010; tourism license expiry alerts advisory; never autonomous |
| Law 4 — Flexibility first-class | Cluster spectrum from B&B to fine-lodge via pack config; framework holds without amendment |
| Law 5 — Compliance configured | Rates, no-show policies, cancellation schedules, tourism licensing all pack-driven |
| Law 6 — Distribution regional | Tourism licensing region-specific via packs; regional agent accountability per Charter Law 6 + D-003 |

### 1.5 Parsimony — hotels are where Tanzania's travelers find rest

Tanzania travels. Mama na Bwana Mwema take a weekend at Kilimanjaro to escape Mwanza work. Safari guests from across the world land at Lodge Serengeti for a week of game drives. Mama Halima the freight broker drives Dar→Mwanza routes, stopping overnight in Iringa at Bibi Sauda's small guesthouse on her way. Office workers travel to Dodoma for government meetings, stay in mid-range hotels. Trekkers visit Moshi to climb Mt. Kilimanjaro, fill the lodges and guesthouses at the base. Conference attendees fill Arusha hotels during the East Africa summit season.

All of them are guests of Hospitality Stays tenants. Different price points, different room types, different service styles — same engine, different pack configuration. The framework must serve Bibi Sauda's three-room guesthouse (a single owner managing reservations + breakfast + cleaning by herself) and Lodge Serengeti's chain operation (multiple staff per shift, restaurant + spa + activities + transfers) with equal correctness.

**Parsimony is the bar.** Each HOT0-HOT9 below is justified against the question: "does this match how the real receptionist, real housekeeper, real lodge manager in real Tanzanian hospitality operations thinks?"

---

## 2. Inputs and Relationship

### 2.1 Cross-cutting framework (parent)

- **CN-6-100** — Recipe; manifest delta; VE1–VE7 (especially VE7 Workflow primitive — room state, folio aggregation, reservation lifecycle all Workflow instances)
- **CN-6-101** — BD1–BD8; BD5 split-it (Hospitality Stays cluster); BD6 3-of-3 (Hotel passes; sub-types fail individually per cluster doctrine)
- **CN-6-102** — NC1–NC9 naming; VI-03 conflict family
- **CN-6-103** — SP1–SP8; site-scope default; SP3 tenant-scope exception for guest profile (HOT4); SP4 multi-leg-bundled-by-booking_id (Q2)
- **CN-6-104** — HO1–HO9; HO9 settlement-back on folio close; HO6 obligation.settled subscription

### 2.2 Universal layer

- **CN-5-009** Universal Checkout — folio bill emission consumption
- **CN-5-001** Accounting — CTR-030 sufficiency; revenue recognition at checkout settlement
- **CN-5-003** Inventory — for amenities consumption (mini-bar items deducted) and supplies (towels, linen if tracked); **NOT for rooms** (N1 boundary)
- **CN-5-005** HR/Payroll — staff commission if applicable (concierge tips per pack)
- **CN-5-007** Promotion — multi-night discounts; loyalty across stays
- **CN-5-105** Tax-treatment — accommodation tax; tourism levy; per jurisdiction pack

### 2.3 Foundation

- **CN-4-011** Workflow + Party + Document + Obligation + Inventory Movement primitives
- **CN-4-012** Document Engine — tourism license at activation; folio Document at settlement
- **CN-4-021** Saleable Line + Tender value shapes

### 2.4 Sibling Term 6

- **CN-6-001** Retail — RE11 payment abstraction inherited
- **CN-6-002** Restaurant — REST7 charge-to-room emission; CN-6-003 confirms Hotel-side subscription
- **CN-6-004** Workshop (future) — parallel rate-as-pack-content pattern
- **CN-6-005** Bridges (future) — catalogs cross-vertical patterns; CN-6-003 confirms hotel side of restaurant↔hotel bridge
- **CN-6-105** Mixed-Vertical Tenants (future) — generalises hotel + restaurant within one property (Lodge Serengeti)

### 2.5 Brief grounding

- **Brief §6.1** Hotel concept summary
- **Brief §7.3** Six hotel edge cases (room not ready at check-in, overbooking, mid-stay extension, no-show, walk-in, in-stay dining charge-to-room)
- **Brief §11.5** Hotel + Restaurant cross-vertical case (REST7 + HOT5)
- **Brief §11.7** Pharmacy 3-of-3 closure model — applied to Hospitality Stays cluster identification per HOT0

### 2.6 CTRs

- **No new CTRs from hotel mechanics** — cluster + folio + room state fit existing contracts
- CTR-049/050 inherited from CN-6-001 payment abstraction
- CTR-018/002/024/026/030/038/044/045/046/028/006/027 — cited as-is
- **Cumulative CTR-046 queue (unchanged):** DC-NN-e + DC-NN-f

---

## 3. Hotel Doctrine — HOT0 + HOT1–HOT9

### HOT0 — Hospitality Stays Cluster Coverage (Pre-Doctrine)

**`hotel.*` covers the Hospitality Stays cluster.** Hotel (full-service), lodge (safari/eco), guesthouse / B&B, tented camp, serviced apartment, hostel, short-term rental (Airbnb-style). Namespace canonical per CTR-038 + CN-5-103 §16. Per BD5 split-it: one Hospitality Stays vertical mechanism + per-style pack configuration. Per BD6 3-of-3: Hotel passes (multi-day reservation + folio aggregation + tourism licensing distinct from F&B and Retail); sub-types within cluster fail 3-of-3 individually but share the mechanism.

**Placement criterion (vs other verticals):** hotel.* covers operations centered on **multi-day overnight stay with bookable spaces and folio aggregation**. NOT hospitality stays: day-only operations (restaurant, café — those are restaurant.*); event-venue day passes (retail.* with ticket as item); long-term residential property leases (out of BOS scope — real estate domain).

**Regulated layer (tourism licensing, fire safety, environmental compliance):** cross-vertical pack concerns NOT vertical-defining; captured at activation per CTR-045 + Foundation Document; advisor alerts for expiry per Law 3; **BOS is not a licensing authority** — parallel CN-6-002 §16 doctrine. See §18.

### HOT1 — Reservation is a multi-day Workflow distinct from per-visit

Per CN-6-001 RE2 a retail.sale is point-of-transaction; per CN-6-002 REST2 a restaurant.table_session is per-visit (hours). A hotel.reservation is **per-stay** (days, sometimes weeks). The time dimension is the fundamental difference. Reservation lifecycle: `held → confirmed → checked_in → in_house → checked_out → archived`; with `voided`, `cancelled`, and `no_show` compensation branches.

### HOT2 — Folio is site-scope per-stay; guest profile is tenant-scope persistent

Two distinct concepts:

| Concept | Scope | What it aggregates | Lifecycle |
|---------|-------|----------------------|-----------|
| **Folio** (`hotel.folio` Workflow) | Site (SP1) | Charges for THIS stay (room nights, restaurant via Obligation, spa, mini-bar, tourism levy, tax) | Per-stay; opens at check-in; settles at checkout via HO9 |
| **Guest profile** (`hotel.guest_profile` Workflow) | **Tenant (SP3 exception per CN-6-103 §6)** | Stay history across all chain sites; preferences; loyalty; complaint history | Long-running per Party primitive; updated by every stay (folio.completed.v1 → guest_profile.updated.v1); never per-stay-scoped |

The two interact: folio.completed updates guest profile; guest profile feeds rate decisions (returning-customer tier per HOT9), advisor recommendations, and loyalty accrual via Promotion (CN-5-007).

### HOT3 — Room state machine is a Workflow primitive instance (VE7)

A room is a bookable resource with a state cycle: `vacant → reserved → occupied → cleaning → vacant`. Per VE7, this is a Workflow primitive instance (CN-4-011) — NOT an Inventory item (CN-5-003 inventory has different semantics for consumables/SKUs). Multiple reservations can attach to a single room across time (the room is reserved Wed-Fri, then occupied Wed evening, then cleaning Fri morning, then vacant Fri midday, then reserved again Fri evening for a new guest). The room Workflow instance is long-running; its state changes per reservation lifecycle transitions and housekeeping operations. See §7 N1 boundary clarification.

### HOT4 — Chain guest profile per CN-6-103 §6 (tenant-scope SP3 exception)

Already established in CN-6-103 §6 catalog row. HOT4 reaffirms vertical-side commitment: `hotel.guest_profile.updated.v1` emits tenant-scope; aggregates chain-wide stay history. Mama na Bwana Mwema's profile sees stays at both Kilimanjaro Lodge Moshi and Lodge Serengeti as a unified history (same tenant, two sites). On their third visit to either property, Bibi Mwajuma (KLM receptionist) sees their full history + preferences + last room they liked.

### HOT5 — Hotel-side Obligation subscription confirming REST7

Mirror of CN-6-002 §13. Restaurant emits `obligation.created.v1 {kind: hospitality_charge}` at bill-with-charge-to-room. Hotel's `hotel.folio` Workflow subscribes; folio absorbs the obligation; aggregates at folio.ready; settles at checkout; Obligation resolves; Restaurant finalizes. The Foundation Obligation primitive is the coordinator — VE2 + BD7 honoured. See §11 N3 6-step settlement loop-back.

### HOT6 — Tourism licensing pack-driven; BOS is not a licensing authority

Pack hook `pack.hotel.tourism_licensing_required` + `pack.hotel.tourism_license_authority` (e.g., TLB-TZ, TBoT, region-equivalent). At activation, regional agent captures license document per CTR-045 expansion (regulatory_evidence_refs). License recorded as Foundation Document (CN-4-012); advisor alerts approaching expiry per Law 3 (non-autonomous). Licensing authority issues licenses; tenant holds + complies; regional agent provides L1 oversight; BOS records, computes (tourism levy per pack), advises. Parallel CN-6-002 §16 doctrine. See §18.

### HOT7 — Self-service check-in (D-DISC-002 partial closure)

Pack hooks `pack.hotel.self_service_check_in_enabled: true` + appropriate adapter coverage per CTR-050 activate the self-service path. Guest scans QR/booking confirmation on mobile, identifies via Party primitive, completes check-in remotely; room key delivered (physical key collection at desk OR digital lock per future integration). Customer-as-actor per CN-4-007. Same `hotel.reservation` Workflow; lifecycle entry at `checked_in` via remote command instead of receptionist command. Mirrors RE10 retail.fulfillment + REST8 QR-table-ordering — same Foundation customer-as-actor mechanism.

### HOT8 — Walk-in vs reservation: same Workflow with path-convergence

Both create a `hotel.reservation` Workflow instance. Manifest flag `pre_reservation: bool`:

- **Reservation path** (`pre_reservation: true`): `held → confirmed → checked_in → in_house → checked_out`
- **Walk-in path** (`pre_reservation: false`): enters at `confirmed` directly (skipping held), → `checked_in → in_house → checked_out`

Same state machine post-confirmation; same folio; same events. Bibi Sauda at her Iringa guesthouse rarely sees pre-reservations (most guests walk in from the road) — pack flag default for guesthouse style is `pre_reservation: false expected`; lodges and hotels skew opposite.

### HOT9 — Rate management is pack content (BD4 push-down parallel REST4)

Room rate computation is **pack content**, not Foundation primitive. Pack hook `pack.hotel.rate_cards.<room_type>.<season>.<rate_basis>`:

```yaml
pack.hotel.rate_cards.deluxe_room.high_season.per_room_per_night:
  base_rate: 200000
  base_currency_ref: <tenant_functional_currency>  # CTR-027
  applicable_dates: ["2026-06-01", "2026-09-30"]
  conditions:
    minimum_nights: 1
    customer_tier_overrides: { vip: -0.10, returning_chain: -0.05, corporate: -0.15 }
    length_of_stay_discount: { 7+ nights: -0.05, 14+ nights: -0.10 }
```

Vertical INDICATES intent via `rate_card_ref`; Checkout K2 + Promotion PR1 RESOLVE per pack (same multi-price layer model as RE3). Pack content varies wildly across hospitality (a B&B's flat per-room rate vs a chain's revenue-management dynamic rate vs a tented camp's per-person all-inclusive); pack content lets each tenant per jurisdiction declare their rate shape. BD5 split-it: Universal Promotion mechanism + per-vertical pack content.

---

## 4. The Manifest — Engine Declaration

```yaml
engine_id: hotel
engine_kind: vertical
namespace_root: hotel
multi_site_capable: false                          # SP1; chain operations use bundling per CN-6-103
scope_policy: site                                 # SP1 default

workflow_instances:
  - workflow_id: hotel.reservation
    billable: false                                # N2 — folio emits the bill, NOT reservation
    lifecycle_states: [held, confirmed, checked_in, in_house, checked_out, archived, voided, cancelled, no_show]
    subscribes_to_internal:                        # N2 dual-stage Workflow handoff
      - event_type: hotel.folio.completed.v1
        filter: reservation_ref == <self>
        transitions_to: checked_out

  - workflow_id: hotel.folio
    billable: true                                 # HO9 canonical
    lifecycle_states: [opened, accumulating, ready, completed, voided, archived]
    settlement_subscription:
      event_type: checkout.settled.v1
      filter: originating_workflow_ref == <self>
      transitions_to: completed

  - workflow_id: hotel.room                        # N1 — Workflow primitive instance, NOT Inventory
    billable: false
    lifecycle_states: [vacant, reserved, occupied, cleaning]  # cyclic; long-running instance per room

  - workflow_id: hotel.guest_profile               # HOT4 — tenant-scope
    billable: false
    scope_ref: tenant                              # SP3 exception per CN-6-103 §6
    lifecycle_states: [active, archived]

  - workflow_id: hotel.rate_card_management        # HOT9 — tenant-scope catalog
    billable: false
    scope_ref: tenant
    lifecycle_states: [draft, active, deprecated, archived]

commands:
  # reservation lifecycle
  - hotel.reservation.hold.request                 # initial booking inquiry
  - hotel.reservation.confirm.request              # payment/credit guarantee passes; reservation confirmed
  - hotel.reservation.cancel.request               # pre-arrival cancellation (per Q7 pack window)
  - hotel.reservation.check_in.request             # receptionist OR self-service per HOT7
  - hotel.reservation.check_out.request
  - hotel.reservation.extend.request               # Q6 mid-stay extension
  - hotel.reservation.walk_in.request              # Q5/HOT8 — pre_reservation: false path
  - hotel.reservation.no_show.request              # Q4 — emitted by scheduled timeout

  # folio operations
  - hotel.folio.open.request                       # auto-triggered at check_in
  - hotel.folio.add_charge.request                 # mini-bar, spa, laundry, etc.
  - hotel.folio.bill.request                       # close folio; emit bill.ready

  # room state
  - hotel.room.mark_occupied.request               # at check_in
  - hotel.room.mark_cleaning.request               # at check_out
  - hotel.room.mark_vacant.request                 # cleaning complete
  - hotel.room.assign.request                      # reservation → room linkage

  # guest profile (tenant-scope)
  - hotel.guest_profile.create.request
    scope_ref: tenant
  - hotel.guest_profile.update.request
    scope_ref: tenant

  # rate cards (tenant-scope)
  - hotel.rate_card.add.request
    scope_ref: tenant
  - hotel.rate_card.update.request
    scope_ref: tenant
  - hotel.rate_card.deprecate.request
    scope_ref: tenant

emits:
  # reservation lifecycle
  - event_type: hotel.reservation.held.v1
    compensation_pair: hotel.reservation.cancelled.v1
  - event_type: hotel.reservation.confirmed.v1
    compensation_pair: hotel.reservation.cancelled.v1
  - event_type: hotel.reservation.checked_in.v1
    compensation_pair: hotel.reservation.checked_out.v1  # post-check_in compensation is normal completion
  - event_type: hotel.reservation.checked_out.v1
    compensation_basis_none: "terminal-success; folio.completed.v1 is the audit anchor"
  - event_type: hotel.reservation.extended.v1
    compensation_basis_none: "extension is forward-only; new departure_date carried"
  - event_type: hotel.reservation.cancelled.v1
    compensation_basis_none: "cancellation IS terminal compensation"
  - event_type: hotel.reservation.no_showed.v1
    compensation_basis_none: "no-show IS terminal compensation; no_show_charge Obligation emits separately if pack rule"

  # folio (HO1 canonical bill)
  - event_type: hotel.folio.opened.v1
    compensation_pair: hotel.folio.voided.v1
  - event_type: hotel.folio.charge_added.v1
    compensation_pair: hotel.folio.charge_reversed.v1
  - event_type: hotel.folio.ready.v1               # NC3 + VE4 + HO1
    compensation_pair: hotel.folio.recalled.v1
  - event_type: hotel.folio.completed.v1
    compensation_basis_none: "settlement closes folio; guest_profile.updated.v1 follows"

  # room state
  - event_type: hotel.room.reserved.v1
    compensation_pair: hotel.room.released.v1
  - event_type: hotel.room.occupied.v1
    compensation_basis_none: "occupancy is state transition; cleaning follows checkout naturally"
  - event_type: hotel.room.cleaning_started.v1
    compensation_basis_none: "physical housekeeping work; observation"
  - event_type: hotel.room.vacant.v1
    compensation_basis_none: "vacant is cyclic; next reservation re-enters reserved"
  - event_type: hotel.room.conflict.detected.v1   # VI-03 overbooking
    compensation_basis_none: "conflict resolution recorded; loser receives own compensation"

  # cross-vertical (REST7 mirror — Hotel subscribes; folio.charge_added.v1 is the emission)
  # No additional Hotel-originated cross-vertical emit; subscription handles incoming

  # guest profile (tenant-scope per SP3)
  - event_type: hotel.guest_profile.updated.v1
    compensation_basis_none: "additive history aggregation; never deleted"
    scope_ref: tenant

  # rate cards (tenant-scope)
  - event_type: hotel.rate_card.added.v1
    compensation_pair: hotel.rate_card.removed.v1
    scope_ref: tenant
  - event_type: hotel.rate_card.updated.v1
    compensation_pair: hotel.rate_card.reverted.v1
    scope_ref: tenant
  - event_type: hotel.rate_card.deprecated.v1
    compensation_basis_none: "forward-only deprecation"
    scope_ref: tenant

  # amenity consumption (mini-bar etc; Pattern A inherited)
  - event_type: hotel.amenity.consumed.v1
    compensation_pair: hotel.amenity.consumption_reversed.v1

subscribes_to:
  - event_type: checkout.settled.v1
    scope_ref: site
    # HO9: routes to hotel.folio per originating_workflow_ref
  - event_type: obligation.created.v1
    scope_ref: site
    filter: kind == "hospitality_charge" && counterparty_workflow_ref == <self>
    # HOT5: REST7 from CN-6-002 §13; folio incorporates Restaurant charges
  - event_type: obligation.settled.v1
    scope_ref: site
    filter: kind == "hospitality_charge" && counterparty_workflow_ref == <self>
    # N3 step 5: Foundation Obligation primitive resolution post-checkout
  - event_type: inventory.stock.depleted.v1
    scope_ref: site
    # awareness for mini-bar restock; advisor signal
  - event_type: pack.effective.v1
    scope_ref: tenant

pack_hooks:
  # cluster configuration (HOT0)
  - pack.hotel.style                               # full_service | lodge | guesthouse | tented_camp | serviced_apartment | hostel | short_term_rental
  - pack.hotel.room_types                          # taxonomy per tenant per jurisdiction
  - pack.hotel.seasonality_calendar                # high/shoulder/low season dates per tenant

  # rates (HOT9)
  - pack.hotel.rate_cards.<room_type>.<season>.<rate_basis>

  # reservation policies
  - pack.hotel.no_show.charge_policy               # Q4 — full_charge | partial_charge | no_charge
  - pack.hotel.no_show.release_after_hours         # Q4 — grace period default per pack
  - pack.hotel.stay_extension.rate_policy          # Q6 — original_rate | current_rate | weighted_average
  - pack.hotel.cancellation.refund_schedule        # Q7 — array of {hours_before_arrival, refund_percentage}

  # tourism licensing (HOT6)
  - pack.hotel.tourism_licensing_required          # bool
  - pack.hotel.tourism_license_authority           # TLB-TZ / TBoT / regional equivalent
  - pack.hotel.fire_safety_compliance              # pack rules per jurisdiction
  - pack.hotel.environmental_compliance            # for lodges in conservation areas etc.

  # cross-vertical (HOT5)
  - pack.hotel.charge_to_room_enabled              # bool; receives REST7 emissions

  # self-service (HOT7)
  - pack.hotel.self_service_check_in_enabled
  - pack.hotel.self_service_check_in_business_id_format

  # amenities
  - pack.hotel.amenities.<amenity_type>.pricing    # mini-bar, spa, laundry per amenity

  # future extension placeholder (N6)
  - pack.hotel.host_managed                        # bool; for short-term rental Airbnb-style; v1 staff-managed default; full mechanism deferred

  # tax (inherits CN-5-105)
  - pack.hotel.tourism_levy_rate                   # accommodation-specific levy per jurisdiction

advisor_ids:
  - hotel-reception-advisor                        # suggests upgrades, reseating recommendations, problem-guest flags
  - hotel-housekeeping-advisor                     # cleaning prioritisation, supply alerts
  - hotel-manager-advisor                          # KPI explanations, occupancy trends, RevPAR, license-expiry alerts
```

The manifest is canonical. All worked patterns below resolve against it.

---

## 5. Workflow Lifecycle — `hotel.reservation` State Machine

### 5.1 States + transitions

```
                       command: reservation.hold.request (initial booking)
                                ▼
                       ┌────────────────┐
                       │      held      │ ── emit hotel.reservation.held.v1
                       └────────────────┘
                                │  payment/credit guarantee received
                                ▼
                       ┌────────────────┐
                       │   confirmed    │ ── emit hotel.reservation.confirmed.v1; hotel.room.reserved.v1
                       └────────────────┘
                                │  command: reservation.check_in.request (arrival day)
                                ▼
                       ┌────────────────┐
                       │   checked_in   │ ── emit hotel.reservation.checked_in.v1; hotel.folio.opened.v1; hotel.room.occupied.v1
                       └────────────────┘
                                │
                                ▼
                       ┌────────────────┐
                       │    in_house    │ ◄── (recurring; folio accumulates charges over stay)
                       └────────────────┘
                                │  command: reservation.check_out.request OR folio.ready triggers
                                ▼
                       ┌────────────────────┐
                       │  (folio_billing)   │ ── intermediate; folio.ready.v1 emits → Universal Checkout
                       └────────────────────┘
                                │  N2 internal subscription: hotel.folio.completed.v1 (filter: reservation_ref == self)
                                ▼
                       ┌────────────────┐
                       │  checked_out   │ ── emit hotel.reservation.checked_out.v1; hotel.room.cleaning_started.v1
                       └────────────────┘
                                │
                                ▼
                       ┌────────────────┐
                       │    archived    │
                       └────────────────┘

Branches:
    held or confirmed → cancelled (pre-arrival cancellation; refund per Q7 pack)
    confirmed → no_showed (grace period expires; charge per Q4 pack)
    in_house → extended (Q6 pack — original_rate default; departure_date moves)
    any pre-billing → voided (manager override)
```

### 5.2 Per-transition guards

| Transition | Guard |
|------------|-------|
| (initial) → held | Room availability check (hotel.room.reserved.v1 not yet emitted for those dates) |
| held → confirmed | Payment received OR credit-tier authorization (RE6 B2C/B2B inherited) |
| confirmed → checked_in | Arrival date reached; identity verified (Party primitive resolution per CN-4-011) |
| checked_in → in_house | Automatic upon checked_in (steady state) |
| in_house → checked_out | N2 internal subscription: folio.completed.v1 arrival with matching reservation_ref |
| any → cancelled | Cancellation command + Q7 pack refund schedule applied |
| confirmed → no_showed | Pack grace period (Q4) expires without check_in |

### 5.3 N2 dual-stage Workflow handoff

Per N2 refinement: `hotel.reservation` itself is `billable: false`. The hotel.folio Workflow is the canonical billable instance per HO9. The reservation Workflow's transition from `in_house → checked_out` is triggered by the internal subscription to `hotel.folio.completed.v1` (HO9-style internal handoff). This is mechanizable via existing DC-NN-f doctrine gate (HO9 expects one Workflow per billable instance; in hotel's case, the folio is the billable instance; the reservation depends on it).

### 5.4 Walk-in path (HOT8)

When `pre_reservation: false`, the Workflow skips `held` and enters at `confirmed` directly:

```
                       command: reservation.walk_in.request (front desk action)
                                ▼
                       ┌────────────────┐
                       │   confirmed    │ ── emit hotel.reservation.confirmed.v1 with pre_reservation: false
                       └────────────────┘
                                │  immediately
                                ▼
                       (proceed to checked_in via standard path)
```

Bibi Sauda's Iringa guesthouse rarely sees `pre_reservation: true` — most guests walk in from the road. The pack default for guesthouse style accommodates this. Same Workflow, different lifecycle entry point.

---

## 6. Workflow Lifecycle — `hotel.folio` State Machine (Multi-Day Accumulation)

### 6.1 States + transitions

```
                       triggered by: hotel.reservation.checked_in.v1
                                ▼
                       ┌────────────────┐
                       │     opened     │ ── emit hotel.folio.opened.v1 {folio_id, reservation_ref, site_id}
                       └────────────────┘
                                │
                                ▼
                       ┌────────────────────────┐
                       │     accumulating       │ ◄── (recurring; emit hotel.folio.charge_added.v1 per charge source)
                       └────────────────────────┘
                                │  command: folio.bill.request (at checkout time)
                                ▼
                       ┌────────────────┐
                       │     ready      │ ── emit hotel.folio.ready.v1 (NC3 + VE4 + HO1) → Universal Checkout
                       └────────────────┘
                                │
                                │  HO9: checkout.settled.v1 with matching originating_workflow_ref
                                │  OR (if charge-to-aggregator): obligation.settled.v1 path
                                ▼
                       ┌────────────────┐
                       │   completed    │ ── emit hotel.folio.completed.v1 → triggers reservation.checked_out.v1
                       └────────────────┘
                                │
                                ▼
                       ┌────────────────┐
                       │    archived    │
                       └────────────────┘

Branches:
    accumulating → voided (manager void; rare)
    ready → recalled (Universal Checkout rejection; corrections needed)
```

### 6.2 Sources of charge accumulation (HOT2 elaborated)

Per `hotel.folio.charge_added.v1` emissions throughout the stay:

| Source | Trigger | Payload includes |
|--------|---------|------------------|
| Room nights | Per-night auto-emission (or batched at folio.ready per pack) | room_type, rate_card_ref, night_count, rate_per_night |
| Restaurant in-stay (REST7) | Subscription to obligation.created.v1 kind=hospitality_charge | obligation_ref, amount, source_engine: restaurant, source_workflow_ref |
| Mini-bar consumption | Vertical-emitted at housekeeping audit | items_consumed, prices_per_item |
| Spa/laundry/extras | Per-service vertical emission | service_type, base_amount |
| Tourism levy | Computed at folio.ready per CN-5-105 + pack | levy_rate, base_amount, levy_amount |

Each `hotel.folio.charge_added.v1` emission is additive; folio accumulates monotonically. Modifications to existing charges use `hotel.folio.charge_reversed.v1` (compensation pair) followed by a fresh charge_added.

### 6.3 The bill.ready emission

At checkout time, folio aggregates and emits the canonical bill:

```
hotel.folio.ready.v1 {
  folio_id,
  reservation_ref,
  site_id,                                          # SP5
  saleable_lines: [room_night × N, restaurant_charge × M, mini_bar, spa, tourism_levy],
  obligation_refs: [<resolved-hospitality-charges>], # if applicable
  originating_workflow_ref: <self>,
  business_date,
  payer_party_ref: <primary_guest>
}
```

Universal Checkout K2 resolves final totals + tax; HO9 fires → folio.completed → reservation → checked_out (via N2 internal subscription).

---

## 7. Workflow Lifecycle — `hotel.room` State Machine (N1 Boundary Clarification)

### 7.1 Room is a Workflow primitive instance, NOT an Inventory item

Per VE7 + N1 boundary: a room is a bookable resource with a state cycle. It is NOT an Inventory item (CN-5-003 inventory primitive serves consumables, SKUs with quantity, lot tracking, expiry). A room has none of those characteristics — it is a fixed resource with a cyclic state lifecycle.

The Workflow primitive (CN-4-011) provides the persistence, audit, and replay for room state. Inventory primitive remains in its own lane for consumables (mini-bar items, supplies, linen if tracked as SKU-quantity).

**Availability queries** for "which rooms are vacant for date range X?" are answered via Workflow projection over `hotel.room.*` events — NOT via Inventory level query. This preserves the CN-5-003 boundary and respects VE7.

### 7.2 States + transitions (cyclic)

```
                       ┌────────────────┐
                       │     vacant     │
                       └────────────────┘
                                │  hotel.reservation.confirmed.v1 attaches to this room
                                ▼
                       ┌────────────────┐
                       │    reserved    │ ── emit hotel.room.reserved.v1
                       └────────────────┘
                                │  hotel.reservation.checked_in.v1
                                ▼
                       ┌────────────────┐
                       │    occupied    │ ── emit hotel.room.occupied.v1
                       └────────────────┘
                                │  hotel.reservation.checked_out.v1
                                ▼
                       ┌────────────────┐
                       │    cleaning    │ ── emit hotel.room.cleaning_started.v1
                       └────────────────┘
                                │  command: room.mark_vacant.request (housekeeping)
                                ▼
                       (cycle back to vacant)
                       ┌────────────────┐
                       │     vacant     │ ── emit hotel.room.vacant.v1
                       └────────────────┘

Branches:
    reserved → vacant (reservation cancelled or no_show → room released)
    occupied → reserved (rare; intra-stay room change)
```

### 7.3 Multiple reservations per room across time

A room Workflow instance is long-running. Over the course of months/years, the same room transitions through `vacant → reserved → occupied → cleaning → vacant` many times. Each transition references the relevant reservation_ref. The room's audit trail spans years.

### 7.4 Room overbooking conflict (VI-03)

If the reservation system tries to confirm two bookings for the same room over the same date range, the bus single-acceptance (CN-4-004) rejects the second; `hotel.room.conflict.detected.v1` fires per VI-03. Resolution per pack: loser receives compensation suggestion (alternative room of same type, alternative dates, upgrade offer per pack rule). See §17.

---

## 8. Pack-Driven Hospitality Stays Variations (HOT0 Elaborated)

The cluster doctrine lives in pack configuration. Each sub-type is the same engine with different pack.

### 8.1 Configuration matrix

| Sub-type | style | room_types | rate_basis | tourism_license | charge_to_room | self_service | host_managed |
|----------|-------|-------------|-------------|------------------|----------------|---------------|---------------|
| Lodge Serengeti (high-end safari) | lodge | [tent_luxury, suite_premium, family_unit] | per_room_per_night + per_person_inclusive | required (TLB + EIA) | true (attached restaurant) | true (HOT7) | false |
| Kilimanjaro Lodge Moshi (mid-range trekker lodge) | full_service | [standard, deluxe, family] | per_room_per_night | required (TLB) | true (attached restaurant) | true | false |
| Bibi Sauda's Guesthouse Iringa (small B&B) | guesthouse | [room_basic, room_with_bath] | per_room_per_night | required (TLB simplified) | false | false | false |
| Hostel Backpacker Dar | hostel | [dorm_bed, private_room] | per_bed_per_night + per_room_per_night | required | true (small bar) | true | false |
| Tented camp Tarangire | tented_camp | [tent_basic, tent_luxury] | per_person_inclusive | required (TLB + conservation) | true | true | false |
| Serviced apartment Dodoma | serviced_apartment | [studio, one_bedroom, two_bedroom] | per_unit_per_night + per_unit_per_week | required (varies) | varies | true | false |
| Future short-term rental (Airbnb-style) | short_term_rental | [whole_property] | per_unit_per_night | varies per jurisdiction | N/A | true | true (deferred per N6) |

### 8.2 N5 — guest profile update mechanism

Per HOT4 + HOT2 distinction. When a folio completes:

```
hotel.folio.completed.v1
       │
       ▼ (vertical-internal subscription)
       Foundation Party primitive updates guest profile metadata (stays count, preferences)
       │
       ▼
hotel.guest_profile.updated.v1 {
  party_ref: <guest>,
  stay_count_incremented: 1,
  cumulative_revenue_added: <folio_total>,
  preferences_updates: { ... },
  last_stay_site_id,
  last_stay_dates,
  loyalty_points_earned: <per Promotion>,
  business_date
}
                                              tenant-scope per SP3 exception
       │
       ▼ fan-out
       Promotion (loyalty accrual)
       Reporting (guest analytics)
       Manager Advisor (returning-customer signal)
       Future Marketing Engine (when activated; outreach via channels per CTR-019)
```

The guest profile aggregates chain-wide; advisors at Bibi Mwajuma's reception screen surface "Mama Mwema's 4th visit; prefers tent_luxury room 7; allergic to peanuts" before the guests even arrive.

### 8.3 N6 — short-term rental future extension placeholder

A short-term rental (Airbnb-style) introduces `host-as-actor` instead of `staff-as-actor`. Pack hook `pack.hotel.host_managed: true` flags this configuration but full mechanism is **deferred per BD8 ≥2 cases threshold** — until at least two concrete African short-term-rental tenants demonstrate need, the full host-managed Workflow (host-side identity, payout splits, cleaning fee separation, host review) remains a placeholder. Current v1 supports only staff-managed hospitality. When the pattern needs to land, it extends manifest with host-actor flow (per CN-4-007 customer-as-actor model + host-side commission split per CN-5-005 pattern).

### 8.4 What this proves

Same engine, seven distinct sub-types via pack. Bibi Sauda's guesthouse with three rooms and Lodge Serengeti's luxury chain operation both emit `hotel.reservation.checked_in.v1` and `hotel.folio.ready.v1` with identical shapes — different pack-driven content.

---

## 9. Rate Management as Pack Content (HOT9; Parallel REST4)

### 9.1 The pack content schema

Pack hook `pack.hotel.rate_cards.<room_type>.<season>.<rate_basis>` declares per tenant per jurisdiction:

```yaml
pack.hotel.rate_cards.deluxe_room.high_season.per_room_per_night:
  base_rate: 200000                                # TZS per room per night
  base_currency_ref: <tenant_functional_currency_ref>  # CTR-027
  applicable_dates: ["2026-06-01", "2026-09-30"]   # high season dates
  conditions:
    minimum_nights: 2                              # minimum stay during peak
    customer_tier_overrides:
      vip: -0.10                                   # 10% discount
      returning_chain: -0.05                       # loyalty discount
      corporate: -0.15                             # negotiated B2B rate
    length_of_stay_discount:
      7_plus_nights: -0.05
      14_plus_nights: -0.10
    package_inclusions: [breakfast]                # included in rate
```

### 9.2 The pricing-resolution flow

Same as RE3 multi-price model from CN-6-001 — vertical INDICATES via `rate_card_ref`; Checkout K2 + Promotion PR1 RESOLVE per pack layer order. Hotel-specific layers:

1. base_rate (from rate_card_ref)
2. seasonality_override (if mid-season transition during stay)
3. length_of_stay_discount (auto-applied per pack rule)
4. customer_tier_override (per Party metadata)
5. promotional_override (Promotion-driven; happy-week deals etc.)
6. tourism_levy + tax (final layer)

### 9.3 Why pack content not primitive

Rate structures vary wildly: a B&B's flat-per-room vs a chain's dynamic-revenue-management vs a tented camp's per-person-all-inclusive vs a serviced apartment's per-week. Pack content lets each tenant per jurisdiction declare their rate shape; Universal Promotion provides mechanism. BD5 split-it; BD4 push-down. Parallel REST4 recipes-as-pack-content.

### 9.4 Concrete TZ example (Lodge Serengeti)

Mama na Bwana Mwema book 3 nights at Lodge Serengeti during high season. Their booking attaches to `pack.hotel.rate_cards.tent_luxury.high_season.per_room_per_night`:
- base_rate: 200,000 TZS / night
- customer_tier: returning_chain (their 3rd visit) → -5%
- length_of_stay: 3 nights — no length discount (kicks in at 7+)
- Per-night final pre-tax: 190,000

Folio shows 3 × 190,000 = 570,000 TZS base accommodation + tourism levy + VAT (computed at K2). See WP1 N4 for full folio breakdown.

---

## 10. Folio Aggregation Mechanics (Multi-Day Accumulation; N4 Concrete TZ Example)

### 10.1 The aggregation lifecycle

A folio opens at check-in, accumulates over the stay, and emits one bill.ready at checkout. Charges arrive from multiple sources asynchronously over days; the folio receives each via `hotel.folio.charge_added.v1`.

### 10.2 N4 — Mwemas concrete TZ folio at Lodge Serengeti

**Stay:** 3 nights, tent_luxury, high season, returning_chain tier.

| Day | Source | Charge | Running Total |
|-----|--------|--------|----------------|
| Day 0 (check-in) | hotel.folio.opened.v1 | — | 0 |
| Day 0 | hotel.folio.charge_added.v1 (room_night × 1; per HOT9 = 190,000) | 190,000 | 190,000 |
| Day 0 evening | hotel.folio.charge_added.v1 (restaurant in-stay via REST7; via obligation hospitality_charge for dinner) | 65,000 | 255,000 |
| Day 0 night | hotel.folio.charge_added.v1 (mini-bar — 2 sodas, 1 beer) | 12,000 | 267,000 |
| Day 1 | hotel.folio.charge_added.v1 (room_night × 1) | 190,000 | 457,000 |
| Day 1 evening | hotel.folio.charge_added.v1 (restaurant — large dinner) | 80,000 | 537,000 |
| Day 1 | hotel.folio.charge_added.v1 (mini-bar) | 8,000 | 545,000 |
| Day 1 | hotel.folio.charge_added.v1 (spa: couple's massage) | 80,000 | 625,000 |
| Day 2 | hotel.folio.charge_added.v1 (room_night × 1) | 190,000 | 815,000 |
| Day 2 | hotel.folio.charge_added.v1 (laundry) | 15,000 | 830,000 |
| Day 2 | hotel.folio.charge_added.v1 (restaurant — final dinner) | 50,000 | 880,000 |
| Day 3 (checkout) | hotel.folio.ready.v1 emits aggregating all charges + computed taxes | | |

At folio.ready, the bill emits with:
- Base accommodation (3 nights): 570,000 TZS
- Restaurant in-stay (3 dinners): 195,000 TZS
- Mini-bar: 20,000 TZS
- Spa: 80,000 TZS
- Laundry: 15,000 TZS
- **Subtotal pre-levy**: 880,000 TZS
- Tourism levy (per pack — TZ TFRS rule ~5% on accommodation portion only): 28,500 TZS
- VAT (per pack — TZ standard 18% on most lines): 162,810 TZS (computed per pack rules per line per CN-5-105 N7)
- **Final folio total**: approximately 1,071,310 TZS

Mwemas settle via their chosen tender method (per RE11 — abstract; tenant's enabled set per Lodge Serengeti tenant configuration per CTR-049). HO9 fires → folio.completed → reservation.checked_out → guest_profile.updated (per N5).

### 10.3 Why folio.ready emits once, not per-charge

The folio is the canonical billable Workflow instance per N2. Universal Checkout consumes once per stay. Charges accumulate via `hotel.folio.charge_added.v1` (additive, audit-only at vertical layer) but don't each emit bill.ready. Bill.ready emits at folio.bill.request command — the singleton settlement event per stay.

### 10.4 Mid-stay folio inspection

A guest may ask for a folio snapshot mid-stay ("kati ya stay yetu, deni langu ni nini?"). The vertical computes via projection over current accumulated charges; presents to guest via Term 3 UI. No new bill.ready emits — this is read-side only.

---

## 11. Charge-to-Room — Hotel-Side Subscription (N3 6-Step Settlement Loop-Back)

### 11.1 Confirming REST7 from the Hotel side

CN-6-002 §13 established the Restaurant-side emission. CN-6-003 confirms the Hotel-side subscription + folio incorporation. Together they form the full cross-vertical bridge.

### 11.2 The 6-step settlement loop-back (N3)

The full chain spans both verticals. From the Hotel perspective:

**Step 1 — Hotel folio subscribes to obligation.created.v1.**

Manifest:
```yaml
subscribes_to:
  - event_type: obligation.created.v1
    filter: kind == "hospitality_charge" && counterparty_workflow_ref == <self_folio_id>
```

When a guest at Lodge Serengeti restaurant elects to charge-to-room (per REST7), Restaurant emits the Obligation. The matching obligation arrives at the Hotel folio.

**Step 2 — Hotel folio incorporates charge.**

The folio emits:
```
hotel.folio.charge_added.v1 {
  folio_id,
  source_engine: restaurant,
  source_workflow_ref: <restaurant_table_session>,
  obligation_ref: <hospitality_charge_obl>,
  line_amount: <bill_total>,
  line_description: "Restaurant — Lodge Serengeti dining",
  included_obligation_refs[].appended_with: [<obl>]   # folio tracks all included obligations
  business_date
}
```

Folio state remains `accumulating`. The Restaurant's table_session Workflow waits in `billing` state (does not transition to completed until step 5+).

**Step 3 — Folio aggregates included_obligation_refs.**

The folio tracks an array `included_obligation_refs[]` — every Restaurant-emitted Obligation (or other vertical's Obligation) that has been folded into the folio. At any moment, the folio knows which Obligations it has incorporated.

**Step 4 — Folio.ready carries the included_obligation_refs.**

When the guest checks out, folio.ready emits:

```
hotel.folio.ready.v1 {
  folio_id,
  reservation_ref,
  saleable_lines: [
    {room_night_lines × N},
    {restaurant_charge_line, obligation_ref: <obl_1>},
    {restaurant_charge_line, obligation_ref: <obl_2>},
    {mini_bar_line},
    ...
  ],
  obligation_refs: [<obl_1>, <obl_2>, ...],            # all hospitality_charge obligations being resolved
  originating_workflow_ref: <folio>,
  business_date
}
```

Universal Checkout K2 resolves; settlement happens per RE11 abstract method.

**Step 5 — Foundation Obligation primitive resolves on checkout.settled.**

When `checkout.settled.v1` for the hotel folio arrives, the Foundation Obligation primitive (CN-4-011) sees the settlement carrying the obligation_refs in payload. It emits per resolved obligation:

```
obligation.settled.v1 {
  obligation_id: <obl_n>,
  resolved_via: <checkout.settled event_id>,
  resolved_amount,
  business_date
}
```

One `obligation.settled.v1` per obligation. The Foundation primitive is the **coordinator** here — it sees the settlement payload, walks the obligation_refs array, and emits resolution events for each. This is BD7-honoured: Foundation primitive coordinates; the two verticals don't reference each other.

**Step 6 — Restaurant table_session.completed transitions.**

Each Restaurant table_session Workflow had subscribed to `obligation.settled.v1` filtered by its own obligation_refs. When the relevant obligation.settled arrives, that table_session transitions: `billing → completed`; restaurant emits `restaurant.commission_earned.v1` for the waiter; Accounting recognizes revenue (correct timing per CN-5-001 + N3 of CN-5-105 + CN-5-104 PC11).

### 11.3 Why this preserves BD7 + VE2

- Restaurant and Hotel **never reference each other's events** at any point
- The Obligation primitive is the **coordinator** at step 5 — emits per-obligation resolution events that each vertical subscribes to independently
- Either vertical can be decommissioned without breaking the other
- The audit chain is complete: from restaurant.table_session.opened → restaurant.bill.ready → obligation.created → hotel.folio.charge_added → hotel.folio.ready → checkout.settled → obligation.settled → restaurant.table_session.completed; all causation linked

### 11.4 Multiple obligations per folio

Mwemas dine at the restaurant three times during their 3-night stay. Each dinner emits its own Obligation. The folio incorporates all three via three separate `hotel.folio.charge_added.v1` events. At folio.ready, all three obligation_refs are in the bill payload. At settlement, three `obligation.settled.v1` events emit; three Restaurant table_session Workflows independently transition to completed.

### 11.5 Pack flag activation

Pack hook `pack.hotel.charge_to_room_enabled: true` activates the HOT5 subscription. Without it, the Hotel does not absorb Restaurant Obligations — for hotels without attached restaurants OR for tenants explicitly disabling the feature. Bibi Sauda's guesthouse (no restaurant) has the flag `false`.

---

## 12. Multi-Leg Booking + Group Bookings

### 12.1 Multi-leg bundled-by-booking_id (Q2 closure)

Per CN-6-103 SP4 sharpening + CN-6-103 WP2: a multi-leg booking is TWO site-scope reservations bundled by `booking_id`, NOT multi-site-by-nature. Each leg is its own `hotel.reservation` Workflow at its own site; the booking_id provides read-side aggregation.

Mwemas book "3 nights Moshi → 2 nights Serengeti" as a single booking. The reservation system creates:

```
hotel.reservation.confirmed.v1 {
  reservation_id: "res-A",
  site_id: "kilimanjaro-lodge-moshi",
  booking_id: "booking-X",                          # shared reference
  arrival_date: "2026-08-14",
  departure_date: "2026-08-17",
  party_ref: <mwemas>,
  ...
}

hotel.reservation.confirmed.v1 {
  reservation_id: "res-B",
  site_id: "lodge-serengeti",
  booking_id: "booking-X",                          # same shared reference
  arrival_date: "2026-08-17",
  departure_date: "2026-08-19",
  party_ref: <mwemas>,
  ...
}
```

Each reservation runs its own Workflow at its own site. Each emits its own folio at checkout. Two separate folios; two separate settlements. The `booking_id` provides read-side "the guest's full trip" aggregation for advisor and reporting purposes.

### 12.2 Group bookings

A corporate event books 10 rooms for the same dates at one site. The reservation system creates 10 individual `hotel.reservation` Workflows sharing a `group_booking_id` field, each attached to its own room.

Each room's folio runs independently. The group sponsor may elect to consolidate at one folio (one corporate folio that absorbs all 10 individual folios at checkout via REST7-like Obligation pattern internally) OR each guest pays their own folio per pack rule.

```
pack.hotel.group_booking.folio_consolidation: 
  options: [per_room, consolidated_at_sponsor]
```

Mwemas as individual guests don't trigger this; corporate events do.

---

## 13. Walk-In vs Reservation Paths (Q5 Closure)

### 13.1 Path convergence

Both walk-in and reservation produce a `hotel.reservation` Workflow. Lifecycle entry differs per HOT8:

- **Reservation:** `held → confirmed → checked_in → in_house → checked_out`
- **Walk-in:** enters at `confirmed` directly with `pre_reservation: false` flag

Same state machine post-confirmation; same folio; same events. Pack default per style varies:
- Lodge / large hotel: pack defaults expect `pre_reservation: true`
- Guesthouse / small B&B: pack defaults expect `pre_reservation: false`

### 13.2 The walk-in command

```
hotel.reservation.walk_in.request {
  site_id,
  party_ref: <guest_being_identified_now>,    # may be a new Party if not yet registered
  expected_departure_date: <today + N nights>,
  room_type_requested,
  estimated_total,
  receptionist_actor: <staff>
}
```

The receptionist creates the Party (if new), assigns a vacant room, opens the reservation directly at `confirmed`, then transitions to `checked_in` immediately.

### 13.3 Bibi Sauda's reality

At Bibi Sauda's Iringa guesthouse: 90%+ of guests are walk-ins. Mama Halima rolls in around 8pm after driving from Dar. Bibi Sauda greets her, checks she has a room available, opens the reservation, hands over the key, asks about breakfast preferences for the morning. Total interaction: 3-4 minutes. Same engine as Lodge Serengeti — just walk-in path + simplified pack config.

---

## 14. Stay Extensions + Modifications (Q6 Closure)

### 14.1 Mid-stay extension

Brief §7.3 explicit case. Mid-stay, a guest decides to stay one more night. The receptionist issues:

```
hotel.reservation.extend.request {
  reservation_id,
  new_departure_date,
  rate_basis: <per pack rule>
}
```

Per `pack.hotel.stay_extension.rate_policy` (Q6):

- **original_rate** (default — customer guarantee): extension uses the same rate_card_ref as original booking; customer doesn't pay more per-night even if market rates rose
- **current_rate**: extension uses today's rate_card per current pack lookup; could be higher (or lower) than original
- **weighted_average**: blend of original and current per pack rule

The Workflow emits:

```
hotel.reservation.extended.v1 {
  reservation_id,
  original_departure: <date>,
  new_departure: <date>,
  additional_nights,
  rate_basis_ref: <which rate card applied>,
  business_date
}
```

The folio absorbs the additional nights via standard charge_added events as the new nights pass.

### 14.2 Other mid-stay modifications

Room upgrades, room downgrades, party-size changes — each emits its own modification event per pack. The folio absorbs price adjustments via compensation pattern (charge_reversed + new charge_added).

### 14.3 Mwemas extend (WP7 anchor)

Mwemas decide on day 2 to add a 4th night because the safari is excellent. Mzee Lyimo (manager) processes the extension; original_rate applies; folio absorbs one more `room_night` charge at the same 190,000 TZS / night rate. New departure date set.

---

## 15. Cancellation + No-Show Handling

### 15.1 Cancellation (Q7 closure)

Pre-arrival cancellation: pack-driven refund schedule:

```yaml
pack.hotel.cancellation.refund_schedule:
  - hours_before_arrival: 168    # 7 days
    refund_percentage: 1.00       # full refund
  - hours_before_arrival: 72     # 3 days
    refund_percentage: 0.50       # 50% refund
  - hours_before_arrival: 24     # 1 day
    refund_percentage: 0.00       # no refund
  - hours_before_arrival: 0      # post-arrival
    refund_percentage: 0.00       # plus manager-override path
```

At cancellation command:

```
hotel.reservation.cancelled.v1 {
  reservation_id,
  cancellation_reason,
  hours_before_arrival_at_cancellation,
  refund_percentage_applied,
  refund_obligation_ref: <created if refund_percentage > 0>,
  business_date
}
```

If refund is due, a Foundation Obligation primitive instance is created (`kind: cancellation_refund`, debtor: tenant, creditor: guest). Resolution per tenant operational practice (manual or auto via banking adapter).

### 15.2 No-show handling (Q4 closure)

Per `pack.hotel.no_show.release_after_hours` (grace period; default e.g., 6 hours past arrival time):

- At grace expiry, scheduled action emits `hotel.reservation.no_showed.v1`
- Room releases (room state → vacant)
- Charge per `pack.hotel.no_show.charge_policy` ∈ {full_charge, partial_charge, no_charge}
- If charge applies, Foundation Obligation primitive emits `kind: no_show_charge`
- Guest profile updated (no-show count incremented; pack-driven sensitivity for blocklist consideration)

### 15.3 Why this is pack-driven

Cancellation generosity varies by business style and competitive positioning. A luxury lodge may offer 7-day full-refund window; a budget guesthouse may have strict 24-hour cutoff; a tented camp during peak season may charge full for late cancellations due to perishable supplies. Pack content per tenant captures their policy; advisors surface this to guests at booking time (Term 3 UI per D-DISC-001).

---

## 16. Self-Service Check-In (HOT7; D-DISC-002 Partial Closure)

### 16.1 Pack activation

```
pack.hotel.self_service_check_in_enabled: true
pack.hotel.self_service_check_in_business_id_format: <CTR-049 + Term 1 governance>
```

### 16.2 Customer-as-actor flow

Per CN-4-007 + D-DISC-002 partial closure (parallel RE10 retail.fulfillment + REST8 QR-table-ordering):

1. Guest arrives at lodge; opens hotel mobile app (Term 3 surface) on their phone; scans QR code on lobby display OR enters booking reference
2. App displays current reservation; prompts for check-in
3. Customer taps "Check In"; identity verifies via Party primitive (registered at booking time)
4. `hotel.reservation.check_in.request` accepted; actor = customer per CN-4-007
5. `hotel.reservation.checked_in.v1` emits; folio.opened.v1 fires
6. Room key delivered (physical key collection at desk OR digital lock — pack-configurable):
   - **Pack: physical_key**: customer prompted to collect at desk; staff hands over
   - **Pack: digital_lock**: room access code sent to app; room unlocks via mobile

### 16.3 What's still staff-mediated

Even with self-service check-in, certain steps remain staff-side per pack:
- Identity verification at high-security properties (passport check for international guests in some jurisdictions)
- Special-request acknowledgement (allergies, accessibility)
- Issue resolution if room not ready (housekeeping coordination)

Pack flag granularity: `pack.hotel.self_service.scope` ∈ {check_in_only, check_in_and_check_out, full_self_service}.

### 16.4 Nina's experience (WP8 anchor)

Nina (cross-doc safari guest from CN-6-001 WP6 + CN-6-002 WP6) arrives at Kilimanjaro Lodge Moshi for her trekking departure. She uses the lodge mobile app to check in remotely while still in the taxi. By the time she arrives at reception, her digital room key is on her phone; she walks straight to her room. Bibi Mwajuma at reception waves at her in greeting; no transaction friction.

---

## 17. Conflict Event Family (VI-03 — Overbooking)

### 17.1 The canonical conflict

The most common hotel conflict: two reservations attempting to confirm the same room for overlapping dates. Causes: multiple booking channels (direct + OTA), human error in availability calendar, system race conditions. The bus single-acceptance mechanism (CN-4-004) handles:

```
hotel.room.conflict.detected.v1 {
  contested_resource_ref: <room_id_for_date_range>,
  winning_workflow_ref: <reservation_first_confirmed>,
  contender_workflow_refs: [<rejected_reservation>],
  detection_ts,
  resolution_basis: "bus_single_acceptance_first_arrival",
  resolution_suggestion: { 
    alternative_room_type: <upgrade_offer>,
    alternative_dates: <next_available_window>,
    manager_intervention_required: <bool per pack>
  }
}
```

### 17.2 Resolution per pack

Per `pack.hotel.overbooking.resolution_policy`:

- **`auto_upgrade`**: offer the loser an upgrade to next-higher room type at same rate (if available)
- **`alternative_dates`**: suggest closest available dates
- **`manager_override`**: flag for manual handling; no auto-resolution
- **`partner_lodge`**: route to a partner-chain property if available

Lodge Serengeti uses `auto_upgrade` first, fallback to `manager_override`. Bibi Sauda's guesthouse has only one room type — `alternative_dates` is her only path.

### 17.3 Other conflict cases

Rarer:
- Two staff members try to occupy a vacant room simultaneously (housekeeping mark conflict)
- Two folio.bill.request commands hit the same folio (paranoid edge — bus single-acceptance handles)

---

## 18. Tourism Licensing + Compliance

### 18.1 BOS is not a licensing authority — restated

Per HOT6 + parallel CN-6-002 §16 doctrine:

> **BOS does not issue, verify, or autonomously gate based on tourism licensing.** Per Charter §1.3 + Law 3 + Law 6 + D-003:
> - **Licensing authorities** (TZ Tourism Board / TLB, TBoT, regional equivalents in Kenya KTB, Uganda UTB, etc.) issue and govern tourism licenses + classifications
> - **Tenants** hold licenses, comply with terms, pursue renewals
> - **Regional agents** (per Charter Law 6 + D-003) are L1 compliance-accountable for their region; capture license evidence at tenant onboarding per CTR-045 expansion
> - **BOS** records the license as a Foundation Document (CN-4-012) at activation; computes tourism levy per pack rules; surfaces license-expiry alerts via advisory framework (Law 3 — non-autonomous); supports authority inspection by providing evidence on demand

### 18.2 Tourism license pack mechanics

`pack.hotel.tourism_licensing_required: true` triggers regulatory evidence capture at onboarding:
- Tourism license document upload
- Property classification (star rating in some jurisdictions; TLB rating in TZ)
- Fire safety certificate
- Environmental impact assessment (for lodges in conservation areas)

`pack.hotel.tourism_license_authority: "TLB-TZ"` (or per-region equivalent) declares which authority's framework applies.

### 18.3 Tourism levy computation

CN-5-105 tax-treatment pack hooks include accommodation-specific levy:

```
pack.hotel.tourism_levy_rate: 0.05   # 5% in TZ
pack.hotel.tourism_levy_basis: accommodation_only   # not on F&B, not on extras
```

At folio.ready, the levy line is added per pack rule per CN-5-105 N7 (tenant_tax_profile applies — VAT-registered tenants compute levy + VAT; non-registered may have simplified treatment per jurisdiction).

### 18.4 License expiry advisory

Per CN-5-010 + Law 3, the hotel-manager-advisor surfaces license-expiry alerts approaching the date:

- 90 days before expiry: "Mzee Lyimo, your TLB license expires in 90 days. Begin renewal process."
- 30 days before: escalates to high priority
- Expired: manager-advisor flags critically; BOS continues recording operations; **no autonomous block**

If the license actually lapses and operations continue, the regulatory authority (TLB inspection) is the enforcement mechanism — not BOS. Per Law 3.

---

## 19. Worked Patterns — Eight Real Tanzanian Hospitality Scenarios

### 19.1 WP1 — Mwemas canonical reservation at Kilimanjaro Lodge Moshi

**Anchor:** Mama na Bwana Mwema (established CN-6-102 WP1 onward) book a 3-night trek-base stay at Kilimanjaro Lodge Moshi. Manager: Mzee Lyimo (NEW). Receptionist: Bibi Mwajuma (NEW).

**Setup:** Lodge tenant pack: style=full_service, room_types=[standard, deluxe, family], tourism_licensing=true (TLB-TZ), charge_to_room=true (attached restaurant), self_service_check_in=true.

**Flow (abbreviated for length):**

1. Mwemas book online 6 weeks ahead: `hotel.reservation.hold.request`, then `confirmed` upon payment (tenant's enabled tender method per regional curation per CTR-049 + RE11 abstract)
2. `hotel.reservation.held.v1` → `hotel.reservation.confirmed.v1` (with deluxe_room rate_card_ref pointing to `pack.hotel.rate_cards.deluxe.high_season.per_room_per_night`)
3. `hotel.room.reserved.v1` for room 12, dates Aug 14-17
4. Arrival day: Mwemas decide to use self-service check-in (HOT7 — Nina-style mobile flow; Mwemas appreciate the speed)
5. `hotel.reservation.check_in.request` (actor: Mama Mwema customer-as-actor per CN-4-007) accepted; `hotel.reservation.checked_in.v1`; `hotel.folio.opened.v1`; `hotel.room.occupied.v1`
6. Mzee Lyimo's manager dashboard pings: "Mwemas arrived; their 4th stay; previously preferred deluxe room 12 (same room as last visit)"
7. Stay unfolds; charges accumulate per N4 example above (3 room_nights × 190,000 TZS returning_chain rate after -5%; 3 restaurant dinners via REST7; mini-bar; spa; laundry)
8. Day 3 (checkout): Bibi Mwajuma processes departure; `hotel.reservation.check_out.request` → folio.bill.request → `hotel.folio.ready.v1` emits with full breakdown
9. Universal Checkout K2 resolves final total (per N4 ≈1,071,310 TZS); Mwemas settle via tenant's enabled method (RE11 abstract)
10. `checkout.settled.v1` → HO9 → `hotel.folio.completed.v1` → (via N2 internal subscription) → `hotel.reservation.checked_out.v1` → `hotel.room.cleaning_started.v1`
11. `hotel.guest_profile.updated.v1` (per N5 — tenant-scope per SP3): Mwemas' stay count → 4; cumulative_revenue updated; preferences re-confirmed (deluxe room 12); loyalty points accrued via Promotion

**Doctrine demonstrated:** HOT1 reservation lifecycle; HOT9 rate card; N4 folio aggregation; HOT5 charge-to-room (via REST7); HOT7 self-service; N5 guest profile update; HO9; RE11 abstract; new characters introduced.

### 19.2 WP2 — Chain guest profile (Mwemas across Moshi + Serengeti)

**Anchor:** After WP1's stay at Kilimanjaro Lodge Moshi, Mwemas drive to Lodge Serengeti for the next leg (per WP5 multi-leg below). They arrive at Lodge Serengeti for the first time. The chain (one tenant: Lodge Serengeti operating both properties) has tenant-scope guest profile per HOT4 + SP3.

**Flow:**

1. Lodge Serengeti receptionist looks up Mwemas by booking_id (per WP5)
2. The guest_profile query (tenant-scope per SP3) returns Mwemas' full history including the 4 Kilimanjaro Lodge Moshi stays
3. Receptionist greets: "Karibu sana Lodge Serengeti, Mama Mwema. Tunashukuru kwa kuendelea kuturudia. Tuna chumba kizuri kwa ajili yenu" — addressing them as returning chain guests
4. Pack rate-card applies returning_chain tier override (-5%)
5. The new stay produces another folio at the Lodge Serengeti site; charges accumulate as normal; at folio.completed, `hotel.guest_profile.updated.v1` again emits tenant-scope, updating Mwemas' chain-wide history (now 5 total stays; 4 at Moshi, 1 at Serengeti)
6. Advisor's downstream subscription notifies: "Mwemas have now stayed at both chain properties; consider escalating to VIP tier per pack rule"

**Doctrine demonstrated:** HOT4 + SP3 + CN-6-103 §6 catalog row; multi-site profile aggregation; cross-site loyalty visibility; N5 guest_profile update mechanism repeated across sites.

### 19.3 WP3 — Hotel-Restaurant charge-to-room (Hotel-side view; confirming REST7 from CN-6-002 §13)

**Anchor:** During Mwemas' Lodge Serengeti stay (per WP2), they dine at the Lodge Serengeti restaurant on their first night. They charge dinner to their room (per REST7 + HOT5).

**Flow (Hotel side per N3 6-step settlement loop-back):**

1. Mwemas finish dinner at Lodge Serengeti restaurant. Restaurant emits per CN-6-002 §13 + WP3 there: `restaurant.bill.ready.v1` with `payment_method_hint: charge_to_room`
2. Universal Checkout routes per hint to Foundation Obligation:

```
obligation.created.v1 {
  obligation_id: "obl-1",
  kind: hospitality_charge,
  debtor_party_ref: <mwemas>,
  creditor_engine: restaurant,
  creditor_workflow_ref: <restaurant_table_session>,
  amount: 65000,
  counterparty_workflow_ref: <mwemas_lodge_serengeti_folio>,
  payload_ref: <restaurant.bill.ready event_id>,
  business_date
}
```

3. Hotel folio at Lodge Serengeti site has subscription per HOT5:

```yaml
subscribes_to:
  - event_type: obligation.created.v1
    filter: kind == "hospitality_charge" && counterparty_workflow_ref == <self_folio>
```

4. Subscription fires; Hotel folio emits:

```
hotel.folio.charge_added.v1 {
  folio_id: <mwemas_folio>,
  source_engine: restaurant,
  source_workflow_ref: <restaurant_table_session>,
  obligation_ref: "obl-1",
  line_amount: 65000,
  line_description: "Lodge Serengeti Restaurant — dinner Day 1",
  included_obligation_refs[].appended_with: ["obl-1"],
  business_date
}
```

5. Stay continues; days later at checkout, folio.ready emits with `obligation_refs: ["obl-1", "obl-2", "obl-3"]` (three restaurant dinners across the stay)
6. Universal Checkout settles; `checkout.settled.v1` for the hotel folio
7. Foundation Obligation primitive sees the settlement payload's obligation_refs array; emits per obligation:

```
obligation.settled.v1 { obligation_id: "obl-1", resolved_via: <checkout.settled>, ... }
obligation.settled.v1 { obligation_id: "obl-2", resolved_via: <same checkout.settled>, ... }
obligation.settled.v1 { obligation_id: "obl-3", resolved_via: <same checkout.settled>, ... }
```

8. Each Restaurant table_session Workflow had subscribed to its own obligation.settled per filter; receives the matching obligation.settled; transitions billing → completed; emits `restaurant.commission_earned.v1` for that night's waiter; Accounting recognizes revenue per CN-5-001

**Doctrine demonstrated:** N3 full 6-step settlement loop-back from Hotel perspective; REST7 confirmed both sides; Foundation Obligation primitive as coordinator (BD7 + VE2); audit chain complete end-to-end across both verticals.

### 19.4 WP4 — Overbooking conflict (VI-03)

**Anchor:** Lodge Serengeti's reservation system receives two confirmations for the same tent_luxury room over the same dates — one through the direct-booking channel, one through a travel-agent OTA channel (the two booking systems momentarily out of sync due to network delay).

**Flow:**

1. Direct booking: `hotel.reservation.confirm.request {room: tent-luxury-7, dates: Aug 14-17, ...}` at t=0
2. OTA booking: `hotel.reservation.confirm.request {room: tent-luxury-7, dates: Aug 14-17, ...}` at t=0.4 seconds
3. Bus single-acceptance: direct booking wins (first arrival); emits `hotel.reservation.confirmed.v1` + `hotel.room.reserved.v1`
4. OTA booking rejected: bus emits `kernel.command.rejected.v1`; VI-03 fires:

```
hotel.room.conflict.detected.v1 {
  contested_resource_ref: tent-luxury-7-aug14-17,
  winning_workflow_ref: <direct_booking>,
  contender_workflow_refs: [<ota_booking>],
  detection_ts,
  resolution_basis: "bus_single_acceptance_first_arrival",
  resolution_suggestion: {
    alternative_room_type: "tent_premium",
    alternative_dates: null,
    manager_intervention_required: false
  }
}
```

5. Per Lodge Serengeti pack `auto_upgrade`: OTA system receives a counter-offer for tent_premium (one tier up) at same rate; OTA confirms with guest; new reservation confirms for tent_premium
6. Manager dashboard logs the upgrade for revenue reconciliation

**Doctrine demonstrated:** VI-03 concrete; bus single-acceptance; pack-driven resolution; manager visibility into compensation.

### 19.5 WP5 — Multi-leg booking Mwemas Moshi + Serengeti (Q2 closure)

**Anchor:** Mwemas book a 5-night holiday split between Kilimanjaro Lodge Moshi (3 nights) and Lodge Serengeti (2 nights) as one booking transaction.

**Flow:**

1. Booking interface accepts the multi-leg structure with shared booking_id "booking-Mwemas-Aug-2026"
2. Two reservation Workflows created — one per site — sharing the booking_id (per Q2)
3. Each leg runs independently: separate confirmations, separate check-ins, separate folios, separate checkouts
4. The booking_id provides read-side aggregation for "Mwemas' full trip view" — useful for advisor recommendations and reporting
5. Each leg's folio is settled independently at its respective checkout

**Doctrine demonstrated:** Q2 multi-leg pattern (bundled-by-booking_id, NOT multi-site-by-nature per CN-6-103 SP4); cleaner than treating chain reservations as multi-site events.

### 19.6 WP6 — Mama Halima walk-in at Bibi Sauda's Iringa Guesthouse (Hospitality Stays cluster + walk-in)

**Anchor:** Mama Halima (cross-doc freight broker from CN-6-100/102/104) is driving her Dar→Mwanza route. Around 8pm, she decides to stop overnight in Iringa rather than push through the night. She rolls into Bibi Sauda's Guesthouse (NEW character — small B&B operator).

**Setup:** Bibi Sauda's pack configuration:
- style: guesthouse
- room_types: [room_basic, room_with_bath]
- rate_basis: per_room_per_night (TZS 35,000 / room_basic)
- tourism_licensing: true (TLB simplified for small accommodations)
- charge_to_room: false (no restaurant)
- self_service_check_in: false (Bibi Sauda handles all check-ins)
- pre_reservation_expected: false (walk-ins are 90% of business)

**Flow:**

1. Mama Halima walks in; Bibi Sauda greets her at the reception desk (which is also Bibi Sauda's kitchen counter)
2. Bibi Sauda checks the room availability board (Workflow projection over hotel.room.*); room_basic 3 is vacant
3. Bibi Sauda issues `hotel.reservation.walk_in.request {site_id, party_ref: <halima_party>, expected_departure_date: tomorrow, room_type_requested: room_basic, estimated_total: 35000}`
4. The Workflow enters at `confirmed` directly per HOT8 (pre_reservation: false); emits `hotel.reservation.confirmed.v1` with `pre_reservation: false` flag
5. Bibi Sauda assigns room 3; emits `hotel.room.reserved.v1` then immediately processes check-in: `hotel.reservation.check_in.request`; emits `hotel.reservation.checked_in.v1` + `hotel.folio.opened.v1` + `hotel.room.occupied.v1`
6. Bibi Sauda asks: "Breakfast saa moja na nusu, sawa?" Mama Halima nods.
7. Total interaction: ~4 minutes. Mama Halima walks to room 3 with the key.
8. Next morning: breakfast (Bibi Sauda emits charge_added if pack rule charges; or it's included per rate). Mama Halima checks out; folio.bill.request; `hotel.folio.ready.v1` emits with single room_night line at 35,000 TZS (plus breakfast if separate; plus tourism levy per simplified TLB rate)
9. Mama Halima pays via her chosen tender method (per Bibi Sauda's enabled methods — small guesthouses in Iringa typically accept cash + mobile money push); HO9 closes folio → reservation → checked_out → cleaning

**Doctrine demonstrated:** HOT0 Hospitality Stays cluster (guesthouse style at simplest config); HOT8 walk-in path (pre_reservation: false); same engine as Lodge Serengeti — different pack; payment-method abstract per RE11; cross-doc Mama Halima continuity; new character Bibi Sauda introduced.

**The bar:** if BOS can serve Bibi Sauda's three-room guesthouse on a roadside in Iringa AND Lodge Serengeti's luxury chain operation with the same engine, the framework holds across the full Hospitality Stays spectrum.

### 19.7 WP7 — Stay extension (Mwemas +1 night at Lodge Serengeti)

**Anchor:** Mwemas' Lodge Serengeti stay (per WP2-3). On day 2 (of their original 2-night Serengeti leg), they decide to extend by 1 night because the safari has been excellent.

**Flow:**

1. Mwemas tell Lodge Serengeti reception they'd like to stay an extra night
2. Receptionist issues `hotel.reservation.extend.request {reservation_id: <mwemas_serengeti>, new_departure_date: <orig+1>, rate_basis: per pack}`
3. Pack rule: `pack.hotel.stay_extension.rate_policy = original_rate` (Lodge Serengeti's customer-guarantee pack)
4. Workflow emits:

```
hotel.reservation.extended.v1 {
  reservation_id,
  original_departure: 2026-08-19,
  new_departure: 2026-08-20,
  additional_nights: 1,
  rate_basis_ref: <same as original — tent_luxury high_season per_room_per_night with returning_chain tier -5%>,
  business_date
}
```

5. The room must be available for the extra night — bus checks via room.* projection (no conflicting reservation exists for Aug 19-20); confirms
6. Folio absorbs the additional night when day 3 arrives — `hotel.folio.charge_added.v1` for the new room_night at 190,000 TZS
7. At final checkout (one day later than originally planned), folio includes 3 room_nights instead of 2; final settlement per RE11

**Doctrine demonstrated:** Q6 closure (original_rate default); folio absorbs extension; no new bill.ready until actual checkout (single canonical settlement per stay).

### 19.8 WP8 — Self-service mobile check-in (Nina at Kilimanjaro Lodge Moshi)

**Anchor:** Nina (safari guest from CN-6-001 WP6 + CN-6-002 WP6 — cross-doc continuity) arrives at Kilimanjaro Lodge Moshi for her trek departure base. She uses the lodge mobile app to check in remotely.

**Flow:**

1. Nina opens the Kilimanjaro Lodge Moshi mobile app (Term 3 surface) while still in the taxi from Kilimanjaro Airport
2. App displays her current reservation (booked weeks earlier)
3. Nina taps "Check In Now"; identity verifies via Party primitive (registered at original booking time per Term 1 onboarding flow)
4. `hotel.reservation.check_in.request` accepted; actor = Nina per CN-4-007 customer-as-actor
5. Per Kilimanjaro Lodge Moshi pack: `pack.hotel.self_service_check_in_enabled: true`; `digital_lock: true` for selected room types
6. `hotel.reservation.checked_in.v1` emits; `hotel.folio.opened.v1`; `hotel.room.occupied.v1`; digital room key sent to Nina's app
7. Nina arrives at the lodge; walks straight to her room; uses mobile-key to unlock
8. Bibi Mwajuma at reception sees Nina pass; waves greeting; no transaction friction
9. Stay unfolds as normal; checkout via either self-service OR reception per pack scope

**Doctrine demonstrated:** HOT7 self-service; D-DISC-002 partial closure; customer-as-actor per CN-4-007; same mechanism as RE10 (retail mobile order) + REST8 (restaurant QR table) — three verticals share the customer-as-actor Foundation; cross-doc Nina continuity.

---

## 20. Boundaries + Open Items + Cross-Term Hooks

### 20.1 CN-6-003's place in the corpus

| Concern | Owned by | CN-6-003 role |
|---------|----------|----------------|
| Hotel / Hospitality Stays cluster engine declaration | **CN-6-003** (this doc) | Authoritative |
| Cross-cutting framework | CN-6-100..104 | Parents — CN-6-003 applies |
| Universal Checkout consumption | CN-5-009 | Consumer (folio.ready) |
| Universal Accounting | CN-5-001 | Consumer via CTR-030; revenue at checkout |
| Universal Inventory (amenities only — NOT rooms per N1) | CN-5-003 | Consumer for mini-bar etc. |
| Universal Promotion (loyalty + rate discounts) | CN-5-007 | Consumer |
| Universal Tax (accommodation + tourism levy) | CN-5-105 | Consumer per pack |
| Retail (CN-6-001) | Sibling | RE11 payment abstraction inherited |
| Restaurant (CN-6-002) | Sibling | REST7 cross-vertical; CN-6-003 confirms Hotel side |
| Workshop (CN-6-004 future) | Sibling | Parallel HOT9 rate-as-pack-content; parallel CN-6-103 §11 mixed-scope |
| Vertical Bridges (CN-6-005 future) | Sibling | CN-6-003 §11 + CN-6-002 §13 together establish concrete REST7 instance; CN-6-005 catalogues |
| Mixed-Vertical Tenants (CN-6-105 future) | Sibling | Lodge Serengeti hotel + restaurant; generalised case |
| Future-vertical stress sketches | CN-6-901..904 + CN-6-905 | Apply HOT0..HOT9 to candidate verticals where applicable |

### 20.2 CTRs (no new)

- CTR-018, CTR-002, CTR-024, CTR-026, CTR-027, CTR-030, CTR-038, CTR-044, CTR-045, CTR-046, CTR-049, CTR-050 — cited as-is
- CTR-028, CTR-006 — universal payment registries
- **Cumulative CTR-046 expansion queue (unchanged):** DC-NN-e + DC-NN-f

**No new CTRs.** Hospitality Stays cluster + folio mechanics + room state Workflow primitive + HOT9 rate-as-pack-content all fit existing contracts.

### 20.3 Open items inside Term 6 scope

- **CN-6-004 Workshop** — next per Brief §13.9; final concrete vertical; Mzee Hassan + Arusha; parametric geometry + cut lists + offcut tracking + project Workflow long-lifecycle
- **CN-6-005 Bridges** — catalogue REST7 + future cross-vertical patterns; CN-6-002 §13 + CN-6-003 §11 establish the canonical reference
- **CN-6-105 Mixed-Vertical Tenants** — generalises Lodge Serengeti (hotel + restaurant within one property); WP3 + Mwemas charge-to-room demonstrates
- **Short-term rental (host-managed) full mechanism** — deferred per N6; activates when ≥2 concrete African short-term-rental tenants demonstrate need
- **Multi-jurisdiction multi-property** — chain spanning Tanzania + Kenya (one tenant, multiple jurisdictions) deferred to CN-5-105 §11 v2

### 20.4 D-DISC cross-references

- **D-DISC-001 — tenant-customer promotion UX:** WP1 returning_chain tier surfaces in advisor + Term 3 UI; WP6 walk-in guest profile creation flows; rate cards visible to customers per pack permission
- **D-DISC-002 — POS self-service expansion:** HOT7 + WP8 partially close for hospitality; mobile check-in operational; physical-key vs digital-lock pack-configurable; consistent with retail (RE10) + restaurant (REST8) self-service patterns

### 20.5 The bar — framework holds again (Test #3)

CN-6-003 written without amending CN-6-100..104. HOT0 + HOT1-HOT9 derive from cross-cutting framework + cluster doctrine (BD5 + BD6) + Hotel-side REST7 mirror. Eight worked patterns demonstrate spectrum from Bibi Sauda's three-room Iringa guesthouse to Lodge Serengeti's chain operation to Kilimanjaro Lodge Moshi's mid-range trekker base — **all the same engine, all different pack configuration**. Multi-day folio aggregation with cross-vertical Obligation chains works correctly across all configurations. The room-as-Workflow-primitive vs Inventory-item boundary (N1) preserves CN-5-003 scope.

If the framework serves Bibi Sauda on the Iringa roadside and Lodge Serengeti in the safari heartland with equal correctness, it serves the full Tanzanian hospitality landscape. **Framework v1 holds on its third concrete test.**

Next: CN-6-004 Workshop. Mzee Hassan's Arusha karakana. Parametric geometry. Cut lists. Offcut tracking. Project Workflow long-lifecycle. Most complex vertical per Brief — final concrete vertical.

---

*— End of CN-6-003 Hotel Engine (Hospitality Stays Cluster) v1 —*
