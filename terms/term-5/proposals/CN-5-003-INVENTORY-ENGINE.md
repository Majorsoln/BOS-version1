# CN-5-003 — Inventory Engine

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 5 — Universal Engines
> **Status:** For Overseer review.
> **Governing decisions:** Law 4 (flexibility — one engine serves café, workshop, pharmacy, retail); Law 5 (compliance is configured — costing methods, lot policy, expiry rules in packs); D-004 (neutrality — no vertical named in engine code); D-009 (freeze-doctrine analog for recipe versioning); UI-03 (CN-5-102 — inventory movement source-ref required); UI-09 (CN-5-102 — site_id registry resolution).
> **CTRs:**
> - **OPEN (expansion notes added at merge):** CTR-029 (Term 1 — accounting/inventory pack-section adds `inventory_costing` subsection); CTR-030 (Term 6 — vertical event contracts for `<vertical>.ingredient.consumed.v1` / `workshop.cut.executed.v1` / etc., Pattern B emissions).
> - **OPEN (this doc depends on):** CTR-027 (Term 1/2 — tenant site registry; UI-09 source of truth).
> **Glossary:** See `MASTER-GLOSSARY.md` — Universal-engine Invariant (UI), Item, Inventory Movement, Site, Scope level.
> **Depends on:** CN-4-002 (envelope, ordering, causation_id); CN-4-004 (command bus + atomic emission + policy rejection); CN-4-011 (Item primitive, Inventory Movement primitive, Workflow primitive for reservations, Obligation primitive for backorder wrap); CN-4-014 (clock protocol — business_date, expiry-date computation); CN-5-009 (Checkout — `checkout.settled.v1` is the primary deduction trigger); CN-5-001 (Accounting — subscribes to `inventory.adjustment.recorded.v1` for write-up/down journals); CN-5-100 (subscription patterns — auto-deduct via P3; refund via P4 compensation); CN-5-101 (scope policy — Inventory runs default `site`, with `tenant`-scope overrides for items/recipes/transfers/expiry runs per CN-5-101 §6); CN-5-102 (engine invariants — UI-03 source-ref, UI-09 site registry).
> **Boundaries:** Item / Inventory Movement / Workflow / Obligation primitive folds → CN-4-011; procurement lifecycle producing receipts → CN-5-004 (CN-5-003 consumes its emissions); vertical bill / ingredient logic → Term 6 + CTR-030; Accounting journal mechanics (COGS, write-off accounting) → CN-5-001; pack costing-method content per jurisdiction → Term 1 (CTR-029); statutory inventory-valuation reporting templates → CN-5-006 (Reporting); FX revaluation of inventory cost (multi-currency) → CN-5-001 + future FX engine.

---

## 1. What This Doc Defines

CN-5-003 is the Inventory Engine — the second concrete engine of Term 5, the stock-truth layer for any business. A café tracks beans and milk; a karakana tracks aluminium sheets and offcuts; a pharmacy tracks lots of paracetamol with expiry dates; a retail shop tracks tomatoes one for one. **One engine serves all of them** by combining:
- Universal **movements** (receive, deduct, transfer, adjust, reserve, release, write-off, offcut).
- **Recipe expansion** in two patterns that coexist: Pattern A (auto-expand from a recipe Inventory holds) and Pattern B (vertical-emitted consumption events).
- **Lots / expiry** as an opt-in attribute (pharmacy needs it; hardware nails do not).
- **Costing methods** (FIFO / LIFO / weighted average) driven by pack and overridable per item.
- **Reservations** with hard-hold semantics for quotes and folios.
- **Multi-site** projections with explicit transfers between sites.

This doc defines:
- The **engine manifest** with multi-scope per-operation overrides (§3).
- **Items**: atomic, composite, sub-items (offcuts as a special case) (§4).
- **Recipe expansion**: Pattern A and Pattern B coexisting; expansion-mode signaling (§5).
- **Movements catalog** of 8 types, each with typed `source_ref` (§6).
- **Lots, batches, expiry** model (§7).
- **Costing methods** with pack-driven configuration and standard-aware allowed sets (§8).
- **Reservations** with state machine and hard-hold semantics (§9).
- **Offcuts** as lazily-registered sub-items (§10).
- **Substitutions** as vertical-driven metadata on consumption events (§11).
- **Multi-site projections and transfers** with explicit in-transit semantics (§12).
- **Negative-stock policy and backorder** with Obligation primitive wrap (§13).
- **UI-03 / UI-09 enforcement** at every emission boundary (§14).

This doc does NOT define:
- Item, Inventory Movement, Workflow, or Obligation primitive folds (CN-4-011).
- Procurement lifecycle emitting GRN events (CN-5-004 consumed by this engine).
- Vertical bill construction or ingredient logic (Term 6 + CTR-030).
- Accounting journals downstream of inventory events (CN-5-001).
- Pack content per jurisdiction (Term 1 authors via CTR-029).
- Statutory inventory-valuation reporting templates (CN-5-006).

---

## 2. The Six Laws

These laws govern every Inventory operation.

| # | Law | Source / why |
|---|-----|--------------|
| **I1** | **Site-scoped stock truth.** Stock-on-hand is per `(tenant_id, site_id, item_id, lot_id?)`. Tenant-wide views are aggregations, not separate ledgers. | CN-5-101 site scope; complementary with CN-5-001 A4 (Accounting is tenant-scope; Inventory is site-scope; they meet via analytical tags on journals). |
| **I2** | **Every movement is typed at source.** Every emitted movement event carries `source_ref` ∈ {`procurement.grn` / `checkout.settled` / `<vertical>.ingredient.consumed` / `workshop.cut` / `transfer` / `adjustment` / `count` / `write_off`}. No naked stock change. | UI-03 (CN-5-102); DC-039 (CN-4-019 Phase 0, CTR-025). |
| **I3** | **Negative stock blocked by default.** A deduction that would drive `available_qty` below zero is rejected at the bus. Pack/tenant may opt into the back-order pattern, which wraps an Obligation primitive (§13). | Conservative default prevents silent over-sells; backorder is an explicit business choice expressed through an Obligation, not through tolerance. |
| **I4** | **Costing method pack-driven, per-item overridable.** The pack declares allowed methods (FIFO / LIFO / weighted average) and a default. Categories may set their own default; tenants may override per item where the pack permits. The pack's `allowed_methods` reflects the accounting standard it encodes — IFRS packs exclude LIFO (IAS 2, 2003 revision); US GAAP packs may include it. | Law 5 (compliance configured); IFRS-vs-GAAP divergence requires pack-driven menu, not engine-level choice. |
| **I5** | **Recipes are Inventory engine state, not Item primitive.** Recipe (component) data is captured via `inventory.recipe.set.v1` events and held in an Inventory projection. Different recipes for the "same" abstract item across tenants are first-class; recipes are versioned and historical sales remain interpretable at their recipe-version-at-sale (D-009 analog). | CN-4-011 Item primitive is identity-only; recipes are operational logic. Avoids unbounded primitive extension (no new CTR to Term 4). |
| **I6** | **Site-scope events carry registry-resolved `site_id`.** Per CN-5-101 L4 + UI-09. Movements naming an unknown `site_id` are rejected at dispatch. | DC-041 (Phase 0); CTR-027 site registry as authority. |

---

## 3. Engine Manifest

Inventory is a **multi-scope engine** per CN-5-101 §6: the default scope is `site` (most operations are per-site), with explicit `tenant`-scope overrides for items, recipes, transfers, and scheduler-driven runs.

```yaml
engine_id:        inventory
display_name:     Inventory Engine
version:          1
scope_policy:     site                      # I1 — stock is per-site truth (default)
primitives_used:
  - item                                    # CN-4-011 (atomic, composite, sub-items)
  - inventory_movement                      # CN-4-011 (the core movement primitive)
  - workflow                                # CN-4-011 (reservation state machine)
  - obligation                              # CN-4-011 (backorder wrap — kind: delivery)

commands:
  # Item lifecycle (tenant — items are tenant-wide identity)
  - id: inventory.item.register.request                  scope_ref: tenant
  - id: inventory.item.update_attributes.request         scope_ref: tenant
  - id: inventory.item.deactivate.request                scope_ref: tenant

  # Recipes (tenant — recipes are tenant-wide logic)
  - id: inventory.recipe.set.request                     scope_ref: tenant
  - id: inventory.recipe.revise.request                  scope_ref: tenant
  - id: inventory.recipe.deactivate.request              scope_ref: tenant

  # Stock movements (site default)
  - id: inventory.movement.record.request                # scope_ref: site
  - id: inventory.stock.count.request                    # scope_ref: site (full / item-list / category-list / lot-list per N5)

  # Reservations (site)
  - id: inventory.reservation.create.request             # scope_ref: site
  - id: inventory.reservation.confirm.request            # scope_ref: site
  - id: inventory.reservation.release.request            # scope_ref: site

  # Transfers (tenant — touches two sites coherently per CN-5-101 §6)
  - id: inventory.transfer.initiate.request              scope_ref: tenant
  - id: inventory.transfer.receive.request               # scope_ref: site (receiving side acknowledges)

  # Scheduler-driven (tenant — runs across all sites of the tenant)
  - id: inventory.lot.expiry_run.request                 scope_ref: tenant

emits:
  # Item lifecycle
  - inventory.item.registered.v1
  - inventory.item.attributes_updated.v1
  - inventory.item.deactivated.v1
  # Recipes
  - inventory.recipe.set.v1                              # Pattern A recipe captured (versioned)
  - inventory.recipe.deactivated.v1
  # Stock movements (umbrella event for Accounting + specialised events)
  - inventory.stock.received.v1                          # source: procurement.grn | transfer.received
  - inventory.stock.deducted.v1                          # source: checkout.settled | <vertical>.consumed
  - inventory.stock.adjusted.v1                          # source: count | adjustment
  - inventory.stock.written_off.v1                       # source: expiry | damage | write_off
  - inventory.adjustment.recorded.v1                     # umbrella → CN-5-001 #9 auto-journal
  # Lots
  - inventory.lot.created.v1                             # on receipt of lot-tracked item
  - inventory.lot.consumed.v1                            # last unit deducted
  - inventory.lot.expired.v1                             # scheduler-triggered
  # Offcuts
  - inventory.offcut.created.v1                          # cut produces sub-item
  # Reservations
  - inventory.reservation.created.v1
  - inventory.reservation.confirmed.v1
  - inventory.reservation.released.v1
  # Transfers
  - inventory.transfer.initiated.v1
  - inventory.transfer.received.v1
  # Backorder (Obligation-wrapped per N3)
  - inventory.promise.created.v1
  - inventory.promise.fulfilled.v1
  - inventory.promise.cancelled.v1

subscribes_to:
  # Procurement (receipt)
  - {event_type: procurement.grn.received.v1,         version: 1, handler: record_receipt,             kind: command_emitting, scope_ref: site}
  - {event_type: procurement.grn.compensated.v1,      version: 1, handler: reverse_receipt,            kind: compensation,     scope_ref: site}
  # Checkout (sale)
  - {event_type: checkout.settled.v1,                 version: 1, handler: deduct_from_sale,           kind: command_emitting, scope_ref: site}
  - {event_type: checkout.refunded.v1,                version: 1, handler: restore_from_refund,        kind: compensation,     scope_ref: site}
  # Pattern B vertical-emitted consumption (per-vertical entries — manifest grows by addition)
  - {event_type: restaurant.ingredient.consumed.v1,   version: 1, handler: record_vertical_consumption, kind: command_emitting, scope_ref: site}
  - {event_type: hotel.amenity.consumed.v1,           version: 1, handler: record_vertical_consumption, kind: command_emitting, scope_ref: site}
  - {event_type: workshop.cut.executed.v1,            version: 1, handler: record_cut_with_offcuts,     kind: command_emitting, scope_ref: site}
  - {event_type: workshop.parametric.consumed.v1,     version: 1, handler: record_vertical_consumption, kind: command_emitting, scope_ref: site}

requires:
  - procurement.grn.received.v1                       # cannot serve a business with no inflow
  - checkout.settled.v1                               # cannot serve a transacting business with no outflow
```

---

## 4. Items: Atomic, Composite, Sub-Items

### Atomic Items

The base case — a single tracked thing. Tomatoes, hammers, paracetamol pills (each), aluminium sheets. Atomic items are deducted 1:1 on sale: 1 sale unit → 1 stock unit decrement.

### Composite Items

A "thing" that is sold as one unit but is internally composed of other items. A café latte (sold as one latte; internally beans + milk + cup + lid). A pharmacy kit (sold as one kit; internally pills + bandage + instruction sheet). A retail bundle ("breakfast pack" = bread + eggs + milk).

Composite identity lives in the Item primitive (CN-4-011) — the latte is an item. The **recipe** (which atomic items it consumes and in what quantity) is **Inventory engine state** (I5), captured by `inventory.recipe.set.v1` and held in a recipe projection.

### Sub-Items (Offcuts)

When a parent item is cut and yields usable remainder, the remainder becomes a tracked **sub-item** with its own item_id. Sub-items are atomic from Inventory's perspective — they have movements, lots (if applicable), and lifecycles like any item. The "parent-child" relationship is captured at registration (`parent_item_ref` on `inventory.item.registered.v1` for offcuts) but is not used for fold logic; it is analytical metadata.

### Item Attributes

```
item.attributes:
  unit_of_measure:    string                 # "g" / "ml" / "each" / "m²"
  is_composite:       boolean                # true if recipe-expanded
  parent_item_ref:    item_id | null         # non-null for offcuts (§10)
  lot_tracked:        boolean                # default false; pack may default true per category
  expiry_relevant:    boolean                # default false
  shelf_life_days:    integer | null         # default null
  costing_method:     fifo | lifo | weighted_average | null   # null = use category/pack default
  backorder_allowed:  boolean                # default false (I3); opt-in per item
```

---

## 5. Recipe Expansion — Pattern A + Pattern B

Two patterns coexist. They cover the operational reality across café, workshop, pharmacy, hotel, restaurant, retail.

### Pattern A — Auto-Expand (Inventory-Driven)

Applies when a sale's `item_ref` is composite **and** the item has an active recipe in Inventory's projection.

| Step | What happens |
|------|--------------|
| 1 | `checkout.settled.v1` arrives at handler `deduct_from_sale` |
| 2 | For each line: if line's `item_ref` is composite and has an active recipe at the sale's recipe-version-at-sale (I5 / D-009 analog), expand: per-component `qty_per_unit × line_qty` |
| 3 | Emit one `inventory.stock.deducted.v1` per component item; each carries `source_ref = checkout.settled` and the originating sale's `causation_id` |
| 4 | Atomic items deduct 1:1 directly without expansion |

### Pattern B — Vertical-Emitted (Term 6 Drives)

Applies when expansion is parametric, substitution-aware, or otherwise vertical-specific (workshop cuts that depend on sheet sizing; restaurant orders with mid-prep substitutions; hotel breakfast that varies by weekday).

| Step | What happens |
|------|--------------|
| 1 | Vertical engine subscribes to `checkout.settled.v1` first (P3); computes its own consumption from business logic |
| 2 | Vertical emits `<vertical>.ingredient.consumed.v1` (or `workshop.cut.executed.v1`, etc.) with full payload: items, qty per item, lot preferences if any, substitution metadata if any |
| 3 | Inventory subscribes (per Term 6 vertical entry); handler `record_vertical_consumption` (or `record_cut_with_offcuts`) emits deductions; **no Pattern A expansion** for the originating sale |

### N1 — Expansion-Mode Signaling

To prevent double-deduction (Pattern A and Pattern B both firing), the bill payload from the vertical declares per line:

```
bill.ready.v1 payload (vertical-emitted):
  lines:
    - line_id, item_ref, qty, …,
      expansion_mode: auto | vertical_managed
```

| `expansion_mode` | Inventory's `deduct_from_sale` behaviour |
|------------------|------------------------------------------|
| `auto` (default) | Pattern A: if composite with recipe → expand; else atomic 1:1 deduct |
| `vertical_managed` | Skip deduction entirely. Wait for the vertical's `*.consumed.v1` event to perform the actual deductions. |

This is **payload metadata**, not a change to CN-4-021 Saleable Line value shape — verticals carry it as a hint to downstream subscribers. Inventory reads it; other downstream engines (Accounting, Cash) ignore it (they journal/record based on the settlement total regardless).

### Why Both Patterns Coexist (Not One)

- Pattern A serves the simple-recipe majority (café, kit, bundle) with **no vertical code** to maintain — Inventory does the expansion.
- Pattern B handles the cases that no Inventory-side recipe can model (cut dimensions varying per order, substitutions during prep, conditional breakfast).
- Forcing Pattern A everywhere would push vertical-specific logic into recipes (an unbounded explosion of recipe variants); forcing Pattern B everywhere would require every café to write code to expand a latte.

Both patterns produce the same downstream event shape (`inventory.stock.deducted.v1`); only the triggering path differs. Accounting, Reporting, and other subscribers see one uniform stream.

### Recipe Versioning + Freeze (Q9)

`inventory.recipe.set.v1 { recipe_version, components, effective_from }` is the recipe-update event. Each sale's expansion records the recipe-version-at-sale-moment in its deduct payload. Historical replay reproduces the expansion under the historical recipe version, not the current one. This is the D-009 freeze doctrine applied analogously to recipes — a café changing its latte recipe in July does not change what was deducted for June sales.

---

## 6. Movements Catalog

Every movement event carries `source_ref` (I2), `site_id` (I6), `item_id`, `lot_id` (if lot-tracked), `quantity`, `unit_cost` (per costing method at this moment), and `business_date` (CN-4-014).

| Type | Triggered by | Direction | Notes |
|------|--------------|-----------|-------|
| **receive** | `procurement.grn.received.v1` or `inventory.transfer.received.v1` | +qty | Creates a new `inventory.lot.created.v1` if item is lot-tracked |
| **deduct** | `checkout.settled.v1` (Pattern A expansion or atomic) or `<vertical>.ingredient.consumed.v1` (Pattern B) | −qty | Lot picked per costing method (§8) |
| **transfer** | `inventory.transfer.initiate.request` / `inventory.transfer.receive.request` | −qty (from) → +qty (to) | Two-phase: initiated → in-transit → received (§12) |
| **adjust** | `inventory.stock.count.request` (physical count discrepancy) | ±qty | Carries `reason_ref`; emits `inventory.adjustment.recorded.v1` for Accounting |
| **reserve** | `inventory.reservation.create.request` | 0 (on_hand unchanged; reserved_qty +qty) | Hard hold (§9) |
| **release** | `inventory.reservation.release.request` | 0 (reserved_qty −qty) | Stock returns to available |
| **write-off** | `inventory.lot.expiry_run.request` (scheduler) or damage/theft adjustment | −qty | Carries reason; emits `inventory.adjustment.recorded.v1` |
| **offcut** | `workshop.cut.executed.v1` | −parent qty / +offcut qty (paired emission) | §10 |

### N5 — Cycle Counts

`inventory.stock.count.request` payload optionally narrows the count scope:

```yaml
inventory.stock.count.request:
  site_id:    <site>
  scope:      full | item_list | category_list | lot_list
  targets:    [<item_ids>] | [<category_tags>] | [<lot_ids>]    # required if scope ≠ full
  counts:     [{item_id, lot_id?, counted_qty}]
```

A pharmacy may cycle-count one therapy category daily without locking the whole store. A supermarket may cycle-count fresh-produce lots nightly. Full counts remain valid (e.g., year-end). Cycle counts emit the same `inventory.stock.adjusted.v1` events at the count granularity.

### The Umbrella Event `inventory.adjustment.recorded.v1`

Whenever a movement results in a value change that Accounting must journal (write-off, count discrepancy, damage adjustment, expiry write-off), Inventory emits the umbrella `inventory.adjustment.recorded.v1` in addition to the specific movement event. CN-5-001's subscription #9 reads this umbrella, keeping Accounting's manifest focused on adjustment-class events rather than every movement subtype.

---

## 7. Lots, Batches, Expiry

### Per-Item Attribute Set

```yaml
item.attributes:
  lot_tracked:       boolean       # opt-in; default false (Q4)
  expiry_relevant:   boolean       # default false (true for perishables, pharma)
  shelf_life_days:   integer | null
```

Pack-defined category defaults may flip these to true (e.g., the TFRS-TZ pack may set pharmaceutical category items `lot_tracked: true` by default).

### Lot Creation

On receive of a lot-tracked item:

```
inventory.lot.created.v1
  payload:
    lot_id:        <unique>
    item_id:       <item>
    site_id:       <receiving site>
    received_date: <business_date, CN-4-014>
    expiry_date:   <null if not expiry_relevant; else computed>
    initial_qty:   <received qty>
    unit_cost:     <per costing method at receipt>
```

Movements for lot-tracked items name a specific `lot_id`. Cycle counts at lot granularity (§6 N5) reconcile per lot.

### Expiry Run

`system:scheduler` submits `inventory.lot.expiry_run.request` per pack-defined frequency (typically daily). The engine scans the lot projection for lots with `expiry_date ≤ current_business_date AND remaining_qty > 0`; for each, emits paired:

- `inventory.lot.expired.v1` (the lot transitions to expired)
- `inventory.stock.written_off.v1` (movement type: write-off, source_ref: expiry, qty: remaining_qty)
- `inventory.adjustment.recorded.v1` (umbrella for Accounting)

### Why Opt-In

Lot tracking carries overhead. For hardware nails, generic flour, or office paper, it adds friction without value. For pharmacy doses, food perishables, or regulated chemicals, it is mandatory. The attribute is per-item, with pack-driven category defaults to make the right choice automatic in most cases.

---

## 8. Costing Methods

### Pack-Section Schema (Extension to CN-5-001 §6, per CTR-029 expansion)

```yaml
inventory_costing:
  allowed_methods:           [fifo, lifo, weighted_average]   # standard-dependent
  default_method:            weighted_average
  category_defaults:
    perishable:              fifo
    durable:                 weighted_average
    raw_material:            fifo
  tenant_override_allowed:   true
  per_item_override_allowed: true
```

**Standard-dependent allowed_methods (LIFO availability):**
- **IFRS pack** (TFRS-TZ and most non-US jurisdictions): `[fifo, weighted_average]` — IAS 2 (2003 revision) prohibits LIFO.
- **US GAAP pack**: may include `lifo` since US GAAP permits it.

CTR-029 governance ensures pack content reflects the standard's actual rules. CN-5-003 enforces only what the pack declares — the engine never hard-codes a standard's preferences.

### Method Application

| Method | Pick on deduct | Cost recording |
|--------|----------------|----------------|
| **FIFO** | Oldest non-empty lot (by `received_date`; for expiry-relevant items, FIFO-by-`expiry_date`) | Movement's `unit_cost` = picked lot's `unit_cost` |
| **LIFO** | Newest non-empty lot first | Movement's `unit_cost` = picked lot's `unit_cost` |
| **Weighted Average** | Any non-empty lot (N2 — see below) | Movement's `unit_cost` = pool weighted-average at deduct moment |

### N2 — Lot-Tracked + Weighted-Average

When an item is both `lot_tracked = true` and uses `weighted_average` costing (common for pharmaceuticals where expiry matters but cost is averaged across lots for reporting):
- **Lot pick** is **FIFO-by-expiry-date** (or FIFO-by-received-date if not expiry-relevant) — physical depletion order is deterministic.
- **Cost recorded** on the movement is the **pool weighted-average** at the moment of deduct, **not** the picked lot's cost.
- This decouples physical-depletion accounting (expiry hygiene) from cost-flow accounting (smoothed across receipts).

Pack governance may further constrain combinations; the engine implements the matrix the pack permits.

### Per-Item Override

Item attribute `costing_method` (nullable). Resolution at deduct time: `item.costing_method` → `category_defaults[item.category]` → `pack.default_method`. Overrides are subject to pack permission (`tenant_override_allowed`, `per_item_override_allowed`).

---

## 9. Reservations

### State Machine (via Workflow primitive, CN-4-011)

```
[created] ──confirm──► [consumed]   (deduct movement emitted)
    │
    └──release──► [released]        (returned to available)
```

### Stock Projection per `(tenant, site, item, lot?)`

- `on_hand_qty`: physical stock present (not yet deducted).
- `reserved_qty`: sum of active reservations against this tuple.
- `available_qty = on_hand_qty − reserved_qty`.

### Hard-Hold Semantics (Q6)

A deduction targeting reserved quantity is **rejected** unless the deduction is itself a `reservation.confirm.request` naming the matching reservation:

```
rejected_by_policy:  inventory.reservation_hard_hold
rejection_reason:    "Insufficient available stock; reserved for reservation <id>"
```

This prevents Cashier-B from accidentally selling the materials reserved for Cashier-A's pending workshop quote.

### Use Cases

- **Workshop quote**: customer approves quote; vertical creates reservation for the cut materials; if quote becomes order, reservation is confirmed (stock deducts); if quote expires, reservation is released.
- **Hotel folio**: guest checks in; reserve breakfast servings for stay duration; on consumption, confirm per serving; on check-out, release any unused.
- **Restaurant pre-order**: reserve ingredients for tomorrow's banquet; consume when prepared; release if cancelled.

### Reservation Lifecycle Versus Sale Lifecycle

A reservation is **independent of any sale**. Reservation → confirmation produces a deduction; that deduction may or may not be linked to a sale (workshop confirmation is typically downstream of `checkout.settled.v1` via vertical event chain; hotel breakfast consumption is downstream of folio-charge events). The reservation is a pre-claim on physical stock, not a financial commitment.

---

## 10. Offcuts (Sub-Items)

When `workshop.cut.executed.v1` is consumed, handler `record_cut_with_offcuts`:

| Step | What happens |
|------|--------------|
| 1 | Read parent item, parent qty, and offcut spec from the event payload |
| 2 | If offcut sub-item is not yet registered for this tenant, emit `inventory.item.registered.v1` for it lazily (with `parent_item_ref` set, dimensions/identity tags from the cut event) |
| 3 | Emit `inventory.stock.deducted.v1` (parent item, full consumed qty, source_ref: workshop.cut) |
| 4 | Emit `inventory.stock.received.v1` (offcut sub-item, offcut qty, source_ref: workshop.cut, movement type: offcut) |
| 5 | Emit `inventory.offcut.created.v1` for analytical / reporting purposes |

### Why Sub-Items, Not a Special Movement Type

An offcut becomes a tracked item: it can be sold, consumed in another cut, written off if it ages, transferred between sites. Treating it as a fully-fledged sub-item gives it the full lifecycle vocabulary without inventing offcut-specific machinery. The cost of one extra item registration per first-of-kind cut is small compared to the analytical value of full lifecycle tracking.

### Offcut Cost Allocation

The pack's `inventory_costing` section declares the offcut cost allocation method:
- **Full-cost-on-parent** (default): offcut is recorded at zero cost; salvage value emerges if/when it sells.
- **Dimension-split**: parent cost is allocated across cut + offcut by dimension/weight ratio per pack rule.

Tenants override where the pack permits. Accounting consumes the resulting `inventory.adjustment.recorded.v1` events without needing to know offcut semantics — the cost is in the event.

---

## 11. Substitutions

Substitution is a Pattern B reality — restaurant orders with mid-prep substitutions, hotel breakfasts with missing items replaced. The substitution semantics live on the **vertical's consumption event**:

```
restaurant.ingredient.consumed.v1
  payload:
    sale_ref:               <checkout.settled event_id>
    consumed_items:
      - {item_ref: parsley_5g, qty: 5g, substituted_for: cilantro_5g, reason: out_of_stock}
      - {item_ref: tomato_50g, qty: 50g}
```

Inventory's handler emits `inventory.stock.deducted.v1` per consumed item, carrying the substitution metadata in payload:

```
inventory.stock.deducted.v1
  payload:
    site_id, item_id, lot_id, qty, unit_cost,
    source_ref: restaurant.ingredient.consumed,
    substituted_for: cilantro_5g,
    substitution_reason: out_of_stock
```

Pattern A has no substitution — recipes are fixed; if a café cannot make a latte with the recipe components, the sale should not have completed (vertical handles the pre-check). Substitution metadata is therefore Pattern-B-only by design.

### Reporting Value

CN-5-006 Reporting can derive substitution rates per ingredient (how often parsley substitutes for cilantro, by site, by season). This supports purchasing decisions and menu engineering — but those analytics live in Reporting; Inventory simply preserves the metadata.

---

## 12. Multi-Site Projections and Transfers

### Per-Site Projection

The stock projection is keyed `(tenant, site, item, lot?)`. Tenant-wide consolidated views are queries over all sites — analytical aggregation, not a separate ledger (I1 + CN-5-101 A4).

### Transfer Workflow

```
Phase 1 — INITIATE (from-site)
  inventory.transfer.initiate.request
    payload: {from_site, to_site, items: [{item_ref, qty, lot_ref?}], reason_ref}
  Bus validates: from-site has available_qty (after reservations) for each item.
  Engine creates an internal reservation at from-site holding the stock.
  Emits inventory.transfer.initiated.v1
  Emits inventory.reservation.created.v1 at from-site (linked to transfer_id)

Phase 2 — IN-TRANSIT (N4)
  Physical movement happens (truck, courier). No event during transit.
  The reservation at from-site is the audit record that stock is committed to transit.
  Analytical "in-transit" view = query over open transfers (initiated without matching received).

Phase 3 — RECEIVE (to-site)
  inventory.transfer.receive.request {transfer_id, items_received: [...]}
  Bus validates: receipt matches initiation. Discrepancies are recorded.
  At from-site: reservation.confirmed → inventory.stock.deducted.v1
  At to-site:   inventory.stock.received.v1 (new lot or merged into existing pool per costing)
  Emits inventory.transfer.received.v1
```

### N4 — In-Transit Semantics

There is **no separate "in-transit" inventory pool**. The reservation at from-site is the canonical mechanism — the stock is logically held there, accounted as part of from-site's inventory until physically received elsewhere. This:
- Honours I1 (every stock has a site).
- Avoids inventing a third location category beyond site_id.
- Makes audit straightforward: open transfers = open reservations linked to transfer_ids.

### Transit Loss

When receipt qty < initiation qty (loss in transit, damage), the receive command emits both:
- The normal transfer events for the actually-received qty.
- A paired `inventory.adjustment.recorded.v1` with `reason_ref = transit_loss` for the discrepancy.

Accounting consumes the adjustment per CN-5-001 #9.

### No Accounting Journal for Pure Transfer

Per CN-5-001 §4 footnote: inventory transfer between sites within the same legal entity produces **no Accounting journal in v1**. Site dimension is analytical only; the same Inventory account aggregated tenant-wide. Transit losses **do** produce journals (via the adjustment).

---

## 13. Negative-Stock Policy and Backorder

### Default: Blocked (I3)

A deduction movement that would drive `available_qty` below zero is rejected at the bus:

```
rejected_by_policy:  I3.negative_stock_blocked
rejection_reason:    "Insufficient available stock at <site> for <item>: requested <qty>, available <available_qty>"
```

### Backorder Opt-In (N3 — Obligation-Wrapped)

Pack/tenant may declare specific items as `backorder_allowed: true`. For these items, a deduction with insufficient stock does not reject; instead the engine creates an Obligation primitive (CN-4-011, kind: `delivery`) and emits:

```
inventory.promise.created.v1
  payload:
    site_id, item_id, promised_qty,
    expected_fulfillment_date,
    sale_ref:           <originating checkout.settled event_id>
    obligation_ref:     <new obligation_id, kind: delivery>
```

The Obligation primitive holds the commitment: `outstanding = promised_qty`, `original_amount = promised_qty`, bounded per UI-08. The sale completed (revenue recognised per pack rules); the physical delivery is owed.

### Fulfillment

When stock arrives that can satisfy an open promise (typically via procurement receipt), the engine emits:

- `inventory.promise.fulfilled.v1` (matching obligation reduces; UI-08 bounds respected)
- `inventory.stock.deducted.v1` (the deferred deduction)

Pack rules govern fulfillment ordering (FIFO of promises by `created_at`, priority by customer tier, etc.). Default: FIFO.

### Cancellation

A customer may cancel a back-ordered sale; the engine emits `inventory.promise.cancelled.v1` (Obligation voided per CN-4-011) and any associated refund choreography (via the original `checkout.refunded.v1` path).

### Why Obligation-Wrap (N3)

A backorder is a debt — the business owes the customer the item. Modelling it as an Obligation (CN-4-011 primitive) gives it the full primitive vocabulary (create / partial-settle / fully-settle / void; bounds; fold) coherent with how credit tenders create Obligations in CN-5-009. Reporting can show "outstanding delivery obligations" alongside outstanding receivables.

---

## 14. UI-03 / UI-09 Application

- **UI-03** (movement source-ref): every movement event listed in §6 carries `source_ref` from the closed set in I2. Phase 0 DC-039 (CN-4-019) enforces presence at schema validation; runtime bus rejects missing or unknown source_ref values.
- **UI-09** (site_id registry): every site-scope emission (most of CN-5-003's events) carries `site_id` validated against the tenant's site registry (CTR-027). Phase 0 DC-041 enforces; dispatch (CN-5-101 §5) rejects unknown sites.

Inventory is the engine most affected by these invariants — most movements are site-scope; nearly all have a source from elsewhere. Discipline at the emission boundary makes UI-03 and UI-09 structural, not aspirational.

---

## 15. Examples — Four Verticals, One Engine

*Examples use illustrative tenants and sites per D-004 #4 (peer-technical audience). They demonstrate that the same engine, the same manifest, and the same laws serve four very different operational realities.*

### 15.1 Café Latte (Mama Asha — Pattern A)

Mama Asha registers an item `latte` with `is_composite: true`. She submits `inventory.recipe.set.request`:

```
inventory.recipe.set.v1
  recipe_version: 3
  parent_item: latte
  components:
    - {item: beans-arabica,  qty_per_unit: 30g}
    - {item: milk-whole,     qty_per_unit: 200ml}
    - {item: cup-12oz,       qty_per_unit: 1}
    - {item: lid-12oz,       qty_per_unit: 1}
```

A customer orders one latte → `retail.bill.ready.v1` (Pattern A default; `expansion_mode: auto`) → Checkout emits `checkout.settled.v1` → Inventory's `deduct_from_sale` reads recipe v3 → emits 4 `inventory.stock.deducted.v1` events at Mwanza site (beans 30g, milk 200ml, cup 1, lid 1). FIFO pick (per pack for perishables); causation chain to the sale.

### 15.2 Workshop Window (Karakana — Pattern B + Offcuts + Reservation)

Karakana sells a custom aluminium window. The vertical engine produces the bill with `expansion_mode: vertical_managed` for the parent item (Inventory will not auto-expand).

```
workshop.cut.executed.v1
  causation_id: <checkout.settled event_id>
  payload:
    site_id: karakana-mwanza-001
    parent_item: aluminium-sheet-2400x1500
    parent_qty: 1
    cuts:
      - {out_item: window-frame-1200x900, qty: 1}
    offcuts:
      - {out_item: aluminium-300x900-offcut, qty: 1}
```

Inventory's `record_cut_with_offcuts` handler:
- Emits `inventory.item.registered.v1` for `aluminium-300x900-offcut` (lazy first-time registration; parent_item_ref set).
- Emits `inventory.stock.deducted.v1` for the parent sheet (qty 1, source_ref: workshop.cut).
- Emits `inventory.stock.received.v1` for the offcut sub-item (qty 1, source_ref: workshop.cut, type: offcut).
- Emits `inventory.offcut.created.v1` for analytical record.

Earlier, when the quote was approved, Karakana had created a reservation for the aluminium sheet. The cut event arrives after `reservation.confirm.request` releases the hold and allows the deduction. If a different cashier had tried to sell the reserved sheet in between, the bus would have rejected (hard hold, §9).

### 15.3 Pharmacy Dose (Lot-Tracked, FIFO, Expiry)

A pharmacy stocks paracetamol with `lot_tracked: true`, `expiry_relevant: true`, costing method `weighted_average` (pack default for pharmaceuticals).

Procurement receives a batch of 1000 tablets, expiry 2027-03-15 → `inventory.lot.created.v1 {lot_id: par-lot-0142, expiry_date: 2027-03-15, initial_qty: 1000, unit_cost: 25}`.

A customer is dispensed 30 tablets → `checkout.settled.v1` → Inventory's `deduct_from_sale` (atomic 1:1, lot-tracked) → pick FIFO-by-expiry → lot par-lot-0142 has earliest expiry → emit `inventory.stock.deducted.v1 {lot_id: par-lot-0142, qty: 30, unit_cost: <pool weighted-average>}` (N2 — physical pick is FIFO-by-expiry; cost recorded is pool weighted-average).

On 2027-03-16, scheduler runs `inventory.lot.expiry_run.request`. Any par-lot-0142 remaining → `inventory.lot.expired.v1` + `inventory.stock.written_off.v1` + `inventory.adjustment.recorded.v1` for Accounting.

### 15.4 Retail Tomato (Atomic 1:1, Weighted-Average)

Mama Amina sells tomatoes. Item `tomato` is `lot_tracked: false`, atomic, weighted-average costing. Customer buys 5 tomatoes → `checkout.settled.v1` → `inventory.stock.deducted.v1 {item: tomato, qty: 5, unit_cost: <pool weighted-average>}`. No lot tracking, no expiry handling, no recipe expansion. Engine moja — pattern rahisi.

### What Stays the Same Across All Four

- The manifest is identical.
- The laws (I1–I6) apply equally.
- Every movement carries `source_ref` (UI-03).
- Every site-scope event carries `site_id` from registry (UI-09).
- Accounting consumes the same umbrella `inventory.adjustment.recorded.v1` events for write-offs and adjustments.
- Reporting sees one consistent event stream regardless of vertical.

This is what "universal engine" means in practice.

---

## 16. Three Whys

### Why does this matter?

Stock is one of the two physical truths every product business has (the other is cash). A café that runs out of beans cannot sell coffee. A pharmacy that ignores expiry kills customers. A workshop that forgets its offcuts wastes raw materials. A retailer that oversells loses trust. CN-5-003 makes the system's stock truth match physical reality at every event — every sale deducts; every receipt adds; every lot expires; every offcut becomes a tracked sub-item. The auto-deduction subscription (P3) makes this completeness emergent: cashiers do not "remember" to update inventory; events do it automatically with full causation lineage.

### Why does it belong here (and not in Foundation)?

Foundation provides the Item primitive (identity, attributes), the Inventory Movement primitive (the movement event shape), the Workflow primitive (state machines for reservations), and the Obligation primitive (used for backorder wrapping). Foundation does not say "every sale deducts via Pattern A or Pattern B," "offcuts become sub-items," "lots are opt-in per item," "weighted-average with lots picks FIFO-by-expiry." Those are universal-engine design choices appropriate for any business that holds physical stock. CN-5-003 makes those choices as the universal inventory pattern. A specialised engine (e.g., a futures-trading inventory) could compose the primitives differently — but that is not universal inventory.

### Why this design?

Pattern A + Pattern B coexistence handles the operational reality without forcing every vertical to write custom logic (café gets auto-expansion; workshop gets the freedom to compute its own cuts) and without forcing every café to expose its recipe through a vertical engine (Inventory's projection holds it). Site-scope default with tenant-scope overrides (multi-scope per CN-5-101 §6) cleanly expresses the "stock-is-per-site, items-are-per-tenant" reality. Pack-driven costing with per-item override accommodates IFRS-vs-GAAP and per-tenant business pattern (pharmacy expiry vs retail durability) without engine code. Lots opt-in keeps overhead off items that do not need them. Hard-hold reservations prevent silent over-sells. Offcuts as sub-items get the full lifecycle vocabulary. Backorder via Obligation gives the wrap coherent with credit tenders. The whole engine composes Foundation primitives into one cohesive answer to "how does any business, in any vertical, track its physical stock?"

---

## 17. Boundaries

| Topic | Lives in |
|-------|----------|
| Item, Inventory Movement, Workflow, Obligation primitive folds | CN-4-011 |
| Event envelope (causation_id, compensates_event_id, pack_version_ref) | CN-4-002 |
| Command bus, atomic emission, rejection policy | CN-4-004 |
| Clock protocol (business_date, expiry-date computation) | CN-4-014 |
| Procurement lifecycle (PO/GRN/invoice/payment cycle producing the receipts Inventory consumes) | CN-5-004 |
| Checkout settlement and refund event shapes (Inventory's primary triggers) | CN-5-009 |
| Accounting auto-journal subscription on `inventory.adjustment.recorded.v1` and `inventory.stock.deducted.v1` | CN-5-001 |
| Subscription patterns + replay semantics + compensation kind | CN-5-100 |
| Scope policy (site default + per-operation overrides; CN-5-101 §6 multi-scope pattern) | CN-5-101 |
| Cross-engine invariants (UI-03 source-ref, UI-09 site registry, UI-08 obligation bounds) | CN-5-102 |
| Pack content for inventory costing (IFRS allowed methods, GAAP allowed methods, category defaults) | Compliance packs authored under CTR-029 (Term 1) |
| Vertical event contracts for Pattern B (`<vertical>.ingredient.consumed.v1`, `workshop.cut.executed.v1`, etc.) | Term 6 (per-vertical docs) + CTR-030 expansion |
| Tenant site registry (UI-09 source of truth) | Term 1 + Term 2 via CTR-027 |
| Reporting templates (P&L inventory line, balance-sheet inventory valuation) | CN-5-006 |
| FX revaluation of multi-currency inventory cost | CN-5-001 + future FX engine |
| Statutory inventory disclosures, lower-of-cost-or-market write-downs | Pack content authored under CTR-029 |

---

## 18. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Pack content for inventory costing per jurisdiction (TFRS-TZ FIFO/avg only, US GAAP including LIFO, etc.) | Term 1 via CTR-029 expansion | Schema in §8; content per jurisdiction is Term 1's authorship |
| Pattern B vertical event contracts (full payload schema per vertical) | Term 6 via CTR-030 expansion | CN-5-003 §5 + §11 specify required payload fields; Term 6 finalises per-vertical |
| Recipe versioning storage / projection mechanism | Architect phase | Concept (§5 + I5) requires versioned, freeze-respecting recipes; mechanism is Architect's |
| Backorder fulfillment ordering policy (FIFO of promises, priority tiers) | Pack content + Term 1 | Default FIFO; tenant elections via pack rules |
| Cycle-count scheduling (who initiates, how often) | Term 3 (UX) + per-tenant policy | CN-5-003 §6 N5 supports the scope; orchestration is per-tenant |
| Lot merging on receipt into existing pool (weighted-average) — whether new lot is always separate or merged | Architect phase + pack | Concept supports both; per-item or per-pack choice |
| Offcut cost allocation rules (full-on-parent vs dimension-split) | Pack content + Term 1 | §10 declares the menu; pack content authorises per jurisdiction |
| Multi-currency inventory cost (a tenant importing in USD but functional-currency TZS) | CN-5-001 + future FX engine | v1: single-currency tenants; multi-currency deferred |
| Lower-of-cost-or-market write-downs (IFRS net-realisable-value) | Pack content via CTR-029 | Periodic re-valuation rules live in pack; CN-5-003 emits the write-down adjustments per pack instructions |
| Phase 1 doctrine check for UI-04 tender chain integrity tied to inventory write-offs from refund choreography | CN-4-019 + future CTR | Aligns with CN-5-102 §7 Phase 1 ramp-up |
| Doctrine check for UI-03 enforcement (DC-039 Phase 0) — CN-5-003 ratification confirms applicability | CN-4-019 | Already in CN-4-019 Phase 0 via CTR-025 |

---

*— End of CN-5-003 —*
