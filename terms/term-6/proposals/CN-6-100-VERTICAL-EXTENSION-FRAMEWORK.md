# CN-6-100 — Vertical Extension Framework

> **Term:** 6 — Vertical Engines
> **Status:** Concept Phase v1.0 — Draft for Overseer review
> **Branch:** `claude/brave-johnson-S7Lky`
> **Reading order:** BOS-CONCEPT-CHARTER → DECISION-LOG → TERM-6-VERTICAL-ENGINES → CN-4-005 → CN-4-020 → CN-4-011 → CN-4-021 → CN-5-100..105 → **this doc**
> **Ordering authority:** TERM-6 Brief §13 — CN-6-100 is the first Term 6 deliverable; CN-6-101..104 elaborate; CN-6-001..004 build on it.

---

## 1. Purpose & Boundary

### 1.1 Mission

CN-6-100 answers the Term 6 Mission Question (Brief §2):

> *"How do we structure these answers so that adding 'Insurance Brokerage' in 2027 or 'Hospital' in 2028 follows the same clean pattern, without anyone touching what already exists?"*

This document is **the recipe** — the mechanical and human-readable answer to "to add a new vertical, you do these N things." It is the framework against which every existing vertical (Retail, Restaurant, Hotel, Workshop) and every future vertical (Insurance, Healthcare, Education, Marketing, Logistics, anything) must conform.

CN-6-100 is a **meta-framework**. It states *that* there is a recipe, *that* there is a manifest, *that* there is a namespace, *that* there are conformance contracts. The detailed operational rules live in:

- **CN-6-101** Vertical Boundary Doctrine — what belongs in a vertical vs universal vs foundation
- **CN-6-102** Vertical Event Naming Conventions — operational naming rules
- **CN-6-103** Vertical Scope Policy — site/tenant/platform discipline for verticals
- **CN-6-104** Vertical-to-Universal Hand-Off Pattern — the standard emission-and-subscribe handshake
- **CN-6-001..004** existing verticals (Retail, Restaurant, Hotel, Workshop) — concrete instances of the framework
- **CN-6-005** Vertical Bridges — cross-vertical interaction patterns
- **CN-6-105** Mixed-Vertical Tenants — multi-vertical tenant operation
- **CN-6-901..904** future-vertical stress-test sketches — the Flexibility Test (Brief §14)

### 1.2 DOES vs DOES NOT (N4 — per CN-5-105 N8 pattern)

| CN-6-100 DOES | CN-6-100 DOES NOT |
|---------------|-------------------|
| Define the seven Vertical Extension doctrine principles (VE1–VE7) | Define detailed event naming syntax (see CN-6-102) |
| Specify the manifest delta a vertical declares on top of CN-4-005 | Replace or override CN-4-005 — it extends, never substitutes |
| Specify mandatory payload-conformance contracts at the vertical-emission boundary | Define internal vertical workflow logic (each vertical does so in its own CN-6-00X) |
| Specify the subscription model (who may subscribe to whom) | Specify universal-engine subscription wiring (that is CN-5-100) |
| Introduce the vertical-side invariant catalog (VI-NN starter) | Enumerate every VI-NN — Term 6 augments as CN-6-001..004 surface real cases |
| Publish the vertical onboarding governance schema | Author the governance content — Term 1 fills per CTR-045 |
| Commit Term 6 to the Flexibility Test and pre-demonstrate it via Logistics walkthrough | Build a Logistics vertical (Logistics is a *stress test*, not a Term 6 deliverable) |
| Cite gaps in upstream contracts and document how the framework handles them while pending | Pre-resolve gaps owned by other Terms |
| Document existing-vertical irregularities the framework prevents (Brief §12) | Audit or refactor existing-vertical code (concept phase — no code) |

### 1.3 Audience

CN-6-100 is read by:

- **Term 6 itself**, when authoring CN-6-001..004 and CN-6-901..904 — this doc is the template they conform to.
- **Architects** designing future verticals at implementation time.
- **Term 1 (Platform Stewards)** when extending the engine catalog with a new vertical (governance content).
- **Term 7 (Integration & Coherence)** when validating that new verticals do not bypass any Law.
- **Any new contributor** who must add a vertical they did not invent — the Flexibility Test reader (§11).

### 1.4 Charter Compliance

| Law | How CN-6-100 honours it |
|-----|--------------------------|
| Law 1 — State derived from events only | All vertical state changes flow through the command bus and emit events (VE1 + VE7); no vertical maintains a side ledger. |
| Law 2 — Engines isolated | VE2 mandates verticals never call other verticals; subscription model (§7) enforces. |
| Law 3 — AI advisory only | Vertical-specific advisors plug into CN-4-022 Advisor Framework + CN-5-010 wiring; never autonomous. §14 confirms CTR-010. |
| Law 4 — Flexibility first-class | This entire document exists to honour Law 4. The Flexibility Test (§11) is its acceptance criterion. |
| Law 5 — Compliance configured | Vertical pack hooks declared via `pack.<vertical>.<sub_domain>.<key>` (§4); no `if vertical == "x"` in code. |
| Law 6 — Distribution regional | Verticals do not embed regional logic; jurisdiction binding flows through CTR-027 per tenant. |

### 1.5 Gap Awareness Summary

CN-6-100 is published while the following are pending. The framework handles each by **citing and continuing** — no upstream gap blocks Term 6 from publishing the recipe; downstream uses of CN-6-100 adapt as gaps close.

| Gap | Status | CN-6-100 handling |
|-----|--------|--------------------|
| CTR-036 `kernel.*` namespace doctrine (DC-042) | Pending Term 4 | §5 cites; verticals respect doctrine without waiting for the runtime check |
| CTR-037 `pack.*` namespace ownership | Pending Term 4 | §4 cites; pack-hook pattern presumed; verticals adjust uniformly if Term 4 ratifies differently |
| CTR-041 `advisor_id` registry mechanism | Pending Term 4 | §12 mentions vertical advisor IDs; activation waits |
| CTR-044 `engine_kind: vertical` manifest flag | Open per `c757c7a` | §4 cites as in-flight Term 4 amendment |
| CTR-045 vertical onboarding governance | Open per `c757c7a` | §10 schema published; Term 1 fills when activated |
| CTR-004 checkout UI contract | Pending Term 3 | §6 emission contract complete; UI side waits |
| CTR-005 checkout always-on vs catalog | Pending Term 1 | Term 5 already committed always-on; CN-6-100 assumes same |
| CTR-010 advisory-only invariant | Pending Term 7 cross-check | §14 reaffirms Term 6 side; Term 7 confirms at integration phase |

---

## 2. Doctrine — VE1–VE7

These seven principles are non-negotiable for every existing and future vertical. A proposed vertical that violates any one is rejected (Brief §8 success criteria — "no vertical engine knows about any other vertical engine; even when bridging, communication is event-based").

**VE1 — A vertical is an engine first.** It registers per CN-4-020, conforms to CN-4-005 manifest, runs under the same isolation guarantees as any universal engine. "Vertical-ness" is a *role* declared via `engine_kind: vertical` (CTR-044), not a special primitive. The kernel does not branch on vertical identity.

**VE2 — A vertical never calls another vertical.** Cross-vertical interaction occurs exclusively via (a) Foundation primitives (especially the Obligation primitive per CN-4-011, for charge-to-room and similar patterns), or (b) subscription to a universal engine's emission (e.g., Restaurant subscribes to `inventory.stock.depleted.v1`, not to `retail.sale.completed.v1`). Direct cross-vertical import or call is rejected at the doctrine gate.

**VE3 — A vertical owns its namespace exclusively.** Per CTR-038, namespaces are reserved at the platform level (`retail.*`, `restaurant.*`, `hotel.*`, `workshop.*`, `pharmacy.*`, `clinic.*`). A vertical never emits inside `kernel.*` (CTR-036), `pack.*` (CTR-037), or another vertical's namespace. Sub-domains within the vertical's own namespace follow CN-5-103 G1–G9.

**VE4 — A vertical emits saleable lines, not tender.** Per D-001 and CTR-002, the universal vertical→checkout handoff is `<vertical>.bill.ready.v1` carrying `saleable_lines[]` (CN-4-021 value shape). Tender, splits, change, receipts, and payment-adapter routing are owned exclusively by CN-5-009 (Universal Checkout/Tender Engine). A vertical that knows the words "M-Pesa," "card," "cash," or "split" inside its own code is misdesigned.

**VE5 — A vertical's payloads conform to universal subscription contracts.** Per CTR-024 (`site_id` in payload), CTR-026 (UI-03 source-ref and UI-09 site_id-in-registry), and CTR-030 (Accounting-sufficient payloads for journal mapping), conformance is the *vertical's responsibility*, not universal engines' adaptation. Universal engines hard-fail at subscription dispatch on non-conformant payloads per CN-5-101 §5.

**VE6 — Adding a vertical never modifies Foundation, Universal Engines, or another vertical.** "Open for extension, closed for modification." A new vertical adds a manifest, a namespace, events, subscriptions, and pack hooks. It does **not** edit existing universal engines, existing verticals, or kernel primitives. Brief §8.6 success criterion is mechanizable as a doctrine check at the CN-4-019 gate.

**VE7 — Stateful vertical processes use the Workflow primitive (CN-4-011), not bespoke state machines.** Hotel reservation lifecycle, Workshop project lifecycle, Restaurant table session, Insurance policy issuance, Healthcare appointment scheduling — every multi-step stateful vertical process is a **Workflow instance**. The vertical contributes lifecycle states (vocabulary), valid transitions, and per-transition guards as payload-level data; the *mechanism* (state persistence, transition audit, replay, compensation) is the Foundation Workflow primitive. Bespoke state machines lose traceability and introduce bespoke compensation logic — the recurring root cause of the gaps Brief §12 cites in existing verticals.

---

## 3. The Recipe — "To Add a New Vertical, You..."

A new vertical joins BOS by completing the following twelve steps. Each step has a mechanical artifact (a manifest field, an event declaration, a pack section) **and** a human-readable check (an architect can answer "yes, we did that"). Steps follow CN-4-020 registration; nothing in the recipe edits existing engines.

### Step 1 — Reserve and declare the vertical namespace
Apply (via CTR-038-style amendment) for a top-level namespace if not pre-reserved. The current pre-reserved set is `retail.*`, `restaurant.*`, `hotel.*`, `workshop.*`, `pharmacy.*`, `clinic.*`. New verticals (e.g., `logistics.*`, `insurance.*`, `school.*`, `marketing.*`) extend §16 of CN-5-103 via Cross-Term Request. The namespace is the vertical's only sandbox; all event types, command types, and pack hooks live under it.

### Step 2 — Declare the engine manifest with `engine_kind: vertical`
Per CN-4-005 and the CTR-044 amendment (in-flight), the manifest declares `engine_kind: vertical`. This signals to the registration mechanism (CN-4-020) and to doctrine checks (CN-4-019) that the engine is bound by VE1–VE7. Required fields are catalogued in §4.

### Step 3 — Emit the universal handoff event
Every vertical that sells, charges, or invoices emits `<vertical>.bill.ready.v1` whose payload carries `saleable_lines[]` (CN-4-021), `site_id` (CTR-024), `originating_workflow_ref` (VE7), `business_date` (CN-5-105), and optional `payer_party_ref`. CN-5-009 consumes the event and owns everything thereafter. The vertical never re-engages with the bill once emitted, except via compensation (e.g., `<vertical>.bill.recalled.v1`) when the underlying workflow rejects before tender lands.

### Step 4 — Conform to mandatory payload contracts
Every event the vertical emits which is subscribed by a universal engine satisfies CTR-024 (site_id), CTR-026 (UI-03 source_ref, UI-09 site_id-in-registry), and CTR-030 (Accounting-sufficient payload). §6 below enumerates the full conformance set including Term 5 hooks.

### Step 5 — Use Workflow primitive for every stateful process (VE7)
For every multi-step process — reservation, project, appointment, layby, policy issuance, trip — declare a Workflow instance per CN-4-011. The vertical contributes:

- Lifecycle state vocabulary (e.g., for Hotel: `held → confirmed → checked_in → in_house → checked_out → archived`; abandoned variants: `cancelled`, `no_show`)
- Valid transitions and per-transition commands
- Per-transition guards (e.g., "confirmed → checked_in requires payment_received OR credit_approved")
- Compensation declaration (UI-02 compensation symmetry per CN-5-102)

The vertical does **not** implement state persistence, transition audit, or replay logic — these are Workflow primitive responsibilities.

### Step 6 — Declare scope policy
Per CN-6-103 (forthcoming) and CN-5-101, default is `site`. Exceptions require explicit justification — see §8.

### Step 7 — Declare pack-hook namespace (N6 four-segment pattern)
Per N6, vertical pack hooks use `pack.<vertical>.<sub_domain>.<key>`. Examples:

- `pack.workshop.cut_optimization.strategy`
- `pack.hotel.room_classes.baseline`
- `pack.restaurant.tip_pool.distribution_rule`
- `pack.logistics.fuel_cost.recognition`
- `pack.pharmacy.controlled_substance.retention_days`

The Term 1 governance lifecycle (per CTR-029 pattern; CTR-045 for vertical specifics) approves pack-hook content per jurisdiction.

### Step 8 — Declare vertical-specific tax treatment paths
Per CN-5-105 N7 and Tax-Aware Engines doctrine, the vertical never computes tax rates itself. It declares which `tax_treatment_ref` lookups apply to which event types, and passes the looked-up reference to checkout. Example for Hotel:

- Room revenue → `pack.tax.lookup(item_category='accommodation', tenant_tax_profile, business_date)`
- Mini-bar consumption → `pack.tax.lookup(item_category='goods', ...)`
- Tourism levy (jurisdiction-specific) → declared in pack; vertical emits the line

### Step 9 — Declare advisor wiring (pending CTR-041)
Per CN-5-010, vertical-specific advisors register their `advisor_id` (pending CTR-041 registry mechanism) and declare audience, data scope (per CN-4-022), and suggestion taxonomy. Verticals never write advisor logic; they declare presence and consume the framework.

### Step 10 — Provide stress-test sketch for Term 7 coherence review
Before activation, the vertical provides a 1-page sketch (per §11 Flexibility Test answers) demonstrating Charter Law audit, at least one mixed-vertical interaction, the cross-vertical bridge pattern if applicable, and a compliance-pack content placeholder for at least one jurisdiction.

### Step 11 — Register via CN-4-020 mechanism
The Foundation registration mechanism (CTR-018 resolved Foundation-side):

- Build-time doctrine gate validates manifest against VE1–VE7
- Runtime registration submits manifest
- Compatibility checks (no namespace collision; no `kernel.*` emission; no direct cross-vertical subscription)
- Atomic wiring/activation only if all checks pass
- Event-sourced registry entry

### Step 12 — Term 1 onboarding governance acceptance (pending CTR-045)
Per CTR-045 (open), the vertical is added to the engine catalog with combo/subscription assignments, always-on vs catalog-item determination, and per-tenant activation gates if regulated. §10 elaborates the governance schema; Term 1 authors the content.

### 3.13 Pattern-Level Guidance for Common Vertical Edge Cases (Gap F)

The recipe above is "happy path." Brief §7 catalogs ~40 edge cases across the four current verticals. CN-6-100 does not solve them individually (each vertical's CN-6-00X owns its own), but it provides **pattern guidance** so each vertical's author solves edge cases consistently.

| Edge Case Category | Examples (Brief §7) | Pattern Guidance |
|--------------------|---------------------|-------------------|
| State transitions mid-process | Hotel guest extends stay; Workshop style updated mid-quote; Restaurant table re-assigned | Workflow primitive transitions (VE7); guard with pack rules; pricing-at-extension explicit in pack hook |
| Conflicts | Two waiters tap same table; reservation overbooking; concurrent quote acceptance | Command-bus single-acceptance (CN-4-004); emit `<vertical>.<noun>.conflict.detected.v1`; losers receive compensating events |
| Walk-out / no-show / abandonment | Customer leaves without paying; reservation no-show; quote not accepted in window | Workflow terminal state `abandoned`/`no_show`; outstanding amounts become Obligation primitive instances |
| Cross-vertical handoff | Hotel guest charges restaurant to room | Obligation primitive — restaurant emits obligation; hotel resolves at folio close; never direct call (VE2) |
| Mid-process inventory shortage | Workshop bar runs out mid-cut; bar 86'd ingredient | Workflow `blocked` state; emit `<vertical>.<noun>.blocked.v1`; Procurement subscribes via universal trigger event |
| Customer disputes / refunds | Refund after price change; broken glass mid-transport; voided sale | UI-02 compensation symmetry; Document amend pattern (CN-5-006 N3); never delete events |
| Multi-actor concurrent operations | Two cashiers same drawer; two waiters same table | Bus serialization (CN-4-004); actor-identity (CN-4-007) on every event |
| Customer identity ambiguity | Regular customer of restaurant + hotel + retail | Party primitive (CN-4-011) — tenant-scope; verticals reference, never invent customer rows |
| Pricing precedence | Layered prices (business / branch / promo / loyalty / customer) | Resolution chain in pack (per CN-5-105 §4 pattern); vertical never invents resolution |

The author of each CN-6-00X consults this table for each edge case Brief §7 identifies for their vertical, picks the pattern, and documents how their specific instance applies it.

---

## 4. Vertical Manifest Delta

A vertical's manifest is a superset of the CN-4-005 Engine Contract Model. CN-4-005 (with the CTR-044 amendment, in-flight) is authoritative; this section summarizes the Term 6 delta.

### 4.1 Required new fields

```
engine_kind: vertical                          # CTR-044 (in-flight Term 4 amendment)
namespace_root: <vertical>                     # one of CTR-038-reserved or CTR-extended
emits[*].event_type: <vertical>.*              # VE3 — namespace exclusivity
mandatory_emission:
  - event_type: <vertical>.bill.ready.v1       # VE4 — Universal Checkout handoff
    payload_contract_ref: CN-4-021             # Saleable Line value shape
pack_hooks[*].namespace: pack.<vertical>.<sub_domain>.<key>   # N6 four-segment
workflow_instances[*]:
  - workflow_id: <vertical>.<process_name>
    lifecycle_states: [...]                    # vocabulary
    valid_transitions: [...]                   # per VE7
advisor_ids[]: <vertical>-<role>-advisor       # pending CTR-041
multi_site_capable: false                      # default; true for trip/freight-style verticals (§8.3)
data_sensitivity_tier: normal                  # candidate VI-05; pending CN-6-101 ratification
```

### 4.2 Subscription constraints (enforced at registration)

A vertical's `subscribes_to[]` may include only:

- Foundation primitive events (`primitive.<name>.<verb>.v<n>` — e.g., `obligation.created.v1`, `workflow.transitioned.v1`)
- Universal engine events (`<universal_engine>.<noun>.<verb>.v<n>` — e.g., `inventory.stock.depleted.v1`, `checkout.settled.v1`)
- Pack lifecycle events (`pack.*` — subject to CTR-037 ratification)
- Kernel meta-events read-only (e.g., `kernel.snapshot.taken.v1` for observability)

A vertical's `subscribes_to[]` **must not** include `<other_vertical>.*`. VE2 doctrine check at CN-4-020 registration rejects.

### 4.3 Pack-hook namespace pattern (N6)

The four-segment pattern is `pack.<vertical>.<sub_domain>.<key>`. The `<sub_domain>` segment groups related hooks within a vertical for governance review — a chartered accountant reviewing tax pack hooks need not read every workshop optimization knob.

Examples spanning current and future verticals:

- `pack.hotel.room_classes.baseline`
- `pack.hotel.tax_treatment.accommodation_levy_ref`
- `pack.workshop.cut_optimization.strategy`
- `pack.workshop.offcut.minimum_usable_length_mm`
- `pack.restaurant.tip_pool.distribution_rule`
- `pack.restaurant.kitchen_routing.default_station`
- `pack.logistics.fuel_cost.recognition`
- `pack.logistics.trip_definition.must_close_within_hours`
- `pack.pharmacy.controlled_substance.retention_days`
- `pack.clinic.appointment.no_show_charge_rule`

### 4.4 CTR-044 pending note

CTR-044 (open per `c757c7a`) requests Term 4 add `engine_kind: vertical` to the CN-4-005 manifest schema as an additive amendment (parallel to CTR-023). Until ratified, Term 6 verticals declare the field in their drafts; the doctrine gate runs in advisory mode.

### 4.5 "Pharmacy = vertical or retail-with-attributes?" — pointer

The criteria for declaring a new vertical vs an attribute-extension of an existing vertical (Brief §11.7 open question) belong to CN-6-101 (boundary doctrine). CN-6-100 only sets the principle: if it has its own Workflow, its own namespace need (e.g., regulatory event vocabulary), or its own regulated activation path, it is a vertical. Otherwise it may be retail-with-attributes. CN-6-101 ratifies the criteria.

---

## 5. Namespace + Naming Discipline

CN-6-102 (forthcoming) is authoritative for naming syntax. This section summarizes the discipline at the framework level.

### 5.1 Namespace reservation (CTR-038)

Per CN-5-103 §16, the following roots are reserved for Term 6 verticals:

- `retail.*` — general retail (duka, supermarket, pharmacy stocked products, boutique)
- `restaurant.*` — food service (café, restaurant, fast-food, bar, kiosk)
- `hotel.*` — hospitality (lodge, hotel, guesthouse)
- `workshop.*` — artisan / repair / manufacturing (karakana, garage, tailoring, fabrication)
- `pharmacy.*` — dispensing pharmacy (regulated)
- `clinic.*` — health-care service delivery (regulated)

Additional roots (e.g., `logistics.*`, `insurance.*`, `school.*`, `marketing.*`) are added via amendment to CN-5-103 §16, filed as CTR from Term 6 to Term 5.

### 5.2 Event type naming (G1–G9)

Per CN-5-103, vertical events follow `<vertical>.<noun>.<verb>.v<n>`:

- `retail.sale.completed.v1`
- `restaurant.table_session.opened.v1`
- `hotel.reservation.confirmed.v1`
- `workshop.cut.executed.v1`
- `pharmacy.prescription.dispensed.v1`
- `clinic.appointment.scheduled.v1`

Sub-domain grouping uses four segments: `<vertical>.<sub_domain>.<noun>.<verb>.v<n>` where required for clarity:

- `workshop.glass.cut.executed.v1` (distinct from `workshop.linear.cut.executed.v1`)
- `hotel.housekeeping.room.cleaned.v1`
- `restaurant.kitchen.ticket.fired.v1`

All verbs are past tense (G9). New event types follow G7 governance (CN-5-103 §16).

### 5.3 Command naming

Per Charter §8.1, commands use `.request` suffix:

- `retail.sale.complete.request`
- `hotel.reservation.confirm.request`
- `workshop.project.accept.request`

### 5.4 Pending upstream

- **CTR-036** — enforcement of `kernel.*` namespace exclusion as DC-042 is pending Term 4. Verticals respect the doctrine without waiting; the runtime check follows when DC-042 ratifies.
- **CTR-037** — `pack.*` namespace ownership confirmation is pending Term 4. The N6 pack-hook pattern assumes `pack.<vertical>.*` is permissible; if Term 4 ratifies a different root, Term 6 will adjust uniformly across CN-6-001..004.

---

## 6. Mandatory Payload Conformance

A vertical's emissions are read by universal engines (Accounting, Cash, Inventory, Procurement, HR, Reporting, Promotion, Checkout, Advisor wiring). The conformance contracts below are non-negotiable. Non-conformant payloads hard-fail at universal subscription dispatch per CN-5-101 §5.

### 6.1 Site identity — CTR-024

Every event whose subscription is **site-scoped** carries `site_id` in payload. The `site_id` must validate against the tenant's site registry (per CTR-027 — Term 2 populates at onboarding; Term 1 governs). Tenant-wide events (rare for verticals) omit `site_id`. The neutral term is `site_id` — never "branch_id," "outlet_id," "venue_id," or other vertical-flavoured names.

### 6.2 Inventory source reference — UI-03 (CTR-026)

Every event that causes an inventory movement (sale, consumption, transfer, return, write-off) carries a typed `source_ref` identifying the originating workflow / document / command. The `source_ref` is the audit anchor — Inventory's UI-03 invariant rejects movements without one.

### 6.3 Site registry resolution — UI-09 (CTR-026)

Every `site_id` in payload must resolve in the tenant's site registry at dispatch time. Verticals never invent `site_id` values — they receive them as command inputs or workflow context.

### 6.4 Accounting journal sufficiency — CTR-030

Every revenue or expense event carries payload sufficient for the active Accounting pack to derive a complete journal entry. Required minimum fields:

- `business_date` — for tax period attribution
- `tax_treatment_ref` — looked up via `pack.tax.lookup(...)`; never computed in vertical
- `recognition_timing` — at_event vs at_settlement vs at_milestone
- `chart_of_accounts_hint` — vertical-domain hint (e.g., `revenue_category: 'room_revenue'`); Accounting maps to GL via pack
- `cogs_basis` — for sales that consume inventory; verticals declare consumed items + valuation method per Pattern A/B (CN-5-003)

### 6.5 Term 5 hook conformance (Gap D)

The following Term 5 doctrines bind vertical emissions. CN-6-100 documents them once; CN-6-001..004 author per-vertical specifics.

| Hook | Source | Vertical-side commitment |
|------|--------|---------------------------|
| Tenant tax profile gate | CN-5-105 N7 | Vertical never computes tax rates; always `pack.tax.lookup(...)` with tenant_tax_profile passed; non-VAT-registered tenants get zero-rate path automatically |
| Period independence | CN-5-104 PC11 | Vertical Workflow instances do not cascade-close on period boundary; forward-correction (PC8) applies — events arriving after period close land in the next open period via compensating-correction pattern, not retroactive amendment |
| Statements ≠ Snapshots | CN-5-006 N3 | Vertical never emits Statement Documents directly; Reporting issues statements post-close; vertical emits only operational events from which Reporting derives statements |
| Checkout idempotency | CN-5-009 K1/K2 | `<vertical>.bill.ready.v1` payload is authoritative input — vertical does not pre-compute tender or cache derived totals; Checkout K2 recomputes deterministically from `saleable_lines[]` |
| Event naming G1–G9 | CN-5-103 | All vertical events follow `<vertical>.<noun>.<verb>.v<n>`; 4-segment sub-namespace allowed for grouped families; G9 past-tense; G7 governance for additions |
| Engine invariants UI-NN | CN-5-102 | Verticals comply UI-03 (source-ref), UI-09 (site_id registry), UI-06 (functional currency), UI-11 (closed tax-period inviolability) at emission boundary; universal subscriptions hard-fail on violation |

### 6.6 Saleable Line shape (CN-4-021)

The mandatory checkout emission carries `saleable_lines[]` where each element is a CN-4-021 Saleable Line value shape:

```
{
  line_id,
  item_ref,
  description,
  quantity,
  unit_price,            # pre-tax
  tax_treatment_ref,     # pack lookup result
  discount_refs[],       # optional
  source_tag,            # opaque vertical tag (audit only; no checkout branching)
  line_total             # non-authoritative cache; Checkout K2 recomputes
}
```

A vertical that omits `tax_treatment_ref` (expecting Checkout to "figure it out") violates CN-5-105. A vertical that includes `tender_method` (expecting Checkout to honour it) violates VE4.

---

## 7. Subscription Model

VE2 forbids vertical-to-vertical subscription. This section defines what is permitted.

### 7.1 What a vertical may subscribe to

- **Foundation primitive events** — `obligation.created.v1`, `workflow.transitioned.v1`, `consent.recorded.v1`, `document.issued.v1`, etc.
- **Universal engine events** — `inventory.stock.depleted.v1`, `accounting.period.closed.v1` (read-only signal), `promotion.applied.v1` (when a promotion modifies a line), `checkout.settled.v1` (terminal signal so the vertical can close its workflow).
- **Pack lifecycle events** — `pack.version.frozen.v1`, `pack.effective.v1` (subject to CTR-037 ratification).
- **Kernel meta-events read-only** — e.g., `kernel.snapshot.taken.v1` for observability. Never as a trigger for vertical state change.

### 7.2 What a vertical may not subscribe to

- **Other vertical's events** — categorically forbidden. If Hotel needs Restaurant data (in-stay dining), the path is: Restaurant emits an Obligation; Hotel subscribes to `obligation.created.v1` filtered by counterparty=self.
- **Internal kernel events as triggers** — `kernel.*` events are kernel meta-events; verticals are not their audience for state change.
- **Other tenant's events** — tenant isolation (Law 2 + CN-4-006) is absolute.

### 7.3 Cross-vertical interaction patterns (pointer to CN-6-005)

CN-6-005 (Vertical Bridges) elaborates. CN-6-100 only states the pattern:

- **Charge-to-room** (Hotel ↔ Restaurant) — Restaurant emits `obligation.created.v1 {counterparty_engine: hotel, counterparty_workflow_ref: <folio_id>, amount}`; Hotel's folio Workflow subscribes to its own obligations and resolves at checkout.
- **Sell-via-POS** (Workshop ↔ Retail) — Workshop completes a project; emits `workshop.deliverable.completed.v1 {item_ref}`; Retail Inventory picks it up via Inventory's universal `inventory.stock.added.v1` signal; Retail then sells via standard `retail.sale.completed.v1` whose `source_ref` points to the workshop project.

Three-way bridges (Hotel → Restaurant → Retail gift-shop sale charged to room) are addressed in CN-6-005 and CN-6-105 (Mixed-Vertical Tenants).

### 7.4 Universal engines subscribing to vertical emissions

This is the **primary** wiring direction. Per CN-5-100, every universal engine that consumes a vertical emission declares the subscription in its manifest. CN-6-001..004 publish the catalog of vertical emissions; universal engines pick what they need. A new vertical's emissions are picked up by universals that already subscribe to the *pattern* (e.g., Accounting subscribes to any `<vertical>.bill.ready.v1` via pattern subscription, not per-vertical hardcoding).

---

## 8. Scope Discipline

Per CN-6-103 (forthcoming) and CN-5-101, verticals declare scope per operation.

### 8.1 Default — site-scope

Every vertical operation is `site`-scoped unless explicitly justified otherwise. A restaurant table session, a hotel reservation, a workshop project, a retail sale — all bind to a single site within the tenant.

### 8.2 Tenant-scope exceptions

Justified tenant-scope operations:

- A hotel chain's central reservation pool (one tenant, many properties, one pool) — but each *reservation* still binds to a `site_id` when confirmed
- A workshop's tenant-wide quote engine (a quote is generated tenant-wide; production happens at a specific site)
- A retailer's tenant-wide loyalty programme (loyalty points accrue tenant-wide; redemption is site-scope)

### 8.3 Multi-site-by-nature (new pattern)

Logistics and freight-style verticals introduce a new pattern: a primary entity (a vehicle, a trip, a shipment) that *traverses* sites. The framework supports this by:

- The vertical declares `multi_site_capable: true` in manifest
- Events carry `origin_site_id` and `destination_site_id` (in addition to or instead of `site_id`)
- Universal engines subscribing to such events apply site-scope semantics per the appropriate field (Inventory uses origin for departure, destination for arrival; Accounting uses `business_date` and the primary cost-recognition site)

Logistics worked walkthrough §12 demonstrates.

### 8.4 Platform-scope (rare and forbidden for verticals)

Verticals never operate at platform-scope. Platform-scope (per CTR-016) is for cross-tenant aggregation owned by Term 1. A vertical that proposes platform-scope is misdesigned — what it wants is either a Reporting projection (Term 5) or platform-aggregator subscription (Term 1).

---

## 9. Vertical-Side Invariants — VI-NN Catalog (Starter)

### 9.1 Boundary clarification — VI-NN ↔ UI-NN (N2)

CN-5-102 owns the UI-NN catalog — invariants about **cross-engine state shape** that hold across two or more universal engines. Examples: UI-03 (every inventory movement has a `source_ref`), UI-09 (every `site_id` resolves in the registry), UI-06 (functional currency consistency), UI-11 (closed tax-period inviolability).

CN-6-100 introduces the VI-NN catalog — invariants about **vertical emission discipline**. Examples: every vertical emits `<vertical>.bill.ready.v1` for billable workflows; every vertical uses Workflow primitive for stateful processes; every vertical's namespace is exclusive.

The boundary:

- **UI-NN** = "this state shape holds after multiple engines have written" → enforced at the **subscription dispatch** boundary in universal engines (hard-fail on non-conformant inbound payload).
- **VI-NN** = "this discipline holds at the vertical-emission boundary" → enforced at **command acceptance and event emission** within the vertical, plus at **registration** by the CN-4-020 mechanism.

The two catalogs are complementary, not overlapping. A vertical emission that violates VI-NN is rejected at the vertical; if it slips through, the universal subscription's UI-NN check fails on the same event. UI-NN is the safety net; VI-NN is the front-line discipline.

### 9.2 VI-01 — Every billable workflow emits `<vertical>.bill.ready.v1`

A vertical that conducts business but never emits a billable handoff is misdesigned. Every Term 6 vertical sells, charges, or invoices; the universal checkout emission is therefore mandatory. The runtime check: for any Workflow instance whose terminal-success state is reached (e.g., `closed`, `served`, `delivered`, `completed`), at least one `<vertical>.bill.ready.v1` must have been emitted referencing that workflow — unless the workflow is explicitly declared `billable: false` in the manifest (rare; reserved for purely internal flows).

### 9.3 VI-02 — Every stateful process is a Workflow instance (VE7 enforcement)

For any vertical operation lasting more than one command (reservation, project, appointment, layby, policy issuance), the vertical declares a Workflow instance per CN-4-011. The doctrine check at CN-4-020 registration verifies the manifest declares at least one `workflow_instances[]` entry; the runtime check rejects state-bearing events that do not reference a `workflow_ref`.

### 9.4 Constraint — starter catalog only (Gap C)

VI-NN v1 includes only VI-01 and VI-02. Term 6 augments the catalog as CN-6-001..004 surface real cases. Candidates already identified:

- **VI-03 (candidate)** — Every vertical with multi-actor concurrent operations (Restaurant tables, Hotel rooms) emits a `<vertical>.<noun>.conflict.detected.v1` event family when bus single-acceptance rejects a contender (per §3.13 conflict pattern).
- **VI-04 (candidate)** — Every regulated vertical (Pharmacy, Clinic, future Insurance) declares pre-activation governance gates per CTR-045 — not in v1 because CTR-045 itself is open.
- **VI-05 (candidate)** — Every vertical that holds Party data with elevated sensitivity (Clinic patient records, future Insurance claimants) declares `data_sensitivity_tier` in manifest — pending CN-6-101 boundary doctrine ruling.

These are placeholders. CN-6-100 v1 does **not** ratify them. Augmentation happens when CN-6-001..004 surface concrete need; CN-6-101 ratifies the final catalog after boundary doctrine work.

---

## 10. Vertical Onboarding Governance (Schema; Term 1 Fills)

Per CTR-045 (open per `c757c7a`), Term 1 owns the content of vertical onboarding governance. CN-6-100 publishes the schema slots; Term 1 fills when activated.

### 10.1 Activation lifecycle slots

| Slot | What it answers | Term 1 fills |
|------|-----------------|---------------|
| `pre_activation_review` | Does the vertical require platform-admin approval before activating per tenant? | Yes/No per vertical (Pharmacy/Clinic likely yes; Retail/Restaurant likely no) |
| `regulatory_review_required` | Does activation require chartered-accountant or regulator sign-off? | Per jurisdiction (Pharmacy in TZ requires TFDA registration; Clinic requires medical board) |
| `combo_assignment` | Which subscription tier(s) include this vertical? | Per pricing decision |
| `always_on_vs_catalog` | Is the vertical sold separately or bundled? | Per Term 1 pricing (CTR-005 pattern) |
| `regional_agent_authority` | Per D-003, does the regional agent activate, or only platform? | Likely agent for non-regulated; platform for regulated |
| `tenant_self_disable` | Can a tenant disable a vertical post-activation? | Likely yes with audit, except for regulated where regulatory data retention applies |
| `cross_vertical_combo_rules` | Are there forbidden combinations (e.g., Pharmacy + alcohol Retail)? | Per jurisdiction |

### 10.2 Per-tenant activation audit fields

Activation events carry:

- `tenant_id`, `vertical_id`, `vertical_version`
- `activating_principal` (regional agent or platform admin per CN-4-007)
- `regulatory_evidence_refs[]` (e.g., TFDA license number for Pharmacy)
- `business_date`
- `compliance_pack_version_at_activation`

### 10.3 Term 1 dependencies (Gap E)

Term 1 is not yet activated. CN-6-100 §10 publishes the schema; full content authoring waits for Term 1. Term 6 verticals (CN-6-001..004) reference §10 as a stub until Term 1 fills.

---

## 11. The Flexibility Test (Brief §14)

Brief §14 commits Term 6 to a non-negotiable test:

> *"Pick a vertical that has never been built. Have someone unfamiliar with BOS read CN-6-100 (the framework) and write a 1-page sketch of how that vertical would fit into BOS. If they can do it without confusion, the framework holds. If they cannot, the framework is incomplete and we revise it."*

### 11.1 Pass criteria — ten questions

The 1-page sketch by an unfamiliar reader must answer, without consulting Term 6:

1. What namespace does the vertical claim?
2. What is the vertical's primary Workflow instance? Its lifecycle states?
3. What does the vertical emit to Universal Checkout (the `<vertical>.bill.ready.v1` payload shape)?
4. What does the vertical subscribe to from Foundation and Universal engines?
5. What pack hooks does the vertical declare?
6. What is the scope discipline (site / tenant / multi-site-by-nature)?
7. What are the regulated-vertical hooks (if any)?
8. What is one Charter Law audit the vertical must pass?
9. What is one mixed-vertical interaction the vertical might participate in?
10. What is one stress-test edge case the vertical must handle (referencing §3.13 pattern guidance)?

If the unfamiliar reader can answer all ten with reference only to CN-6-100 (and the upstream docs CN-6-100 transitively cites), the framework passes. If any question requires asking Term 6, the framework has a gap and CN-6-100 is revised.

### 11.2 Pre-emptive demonstration

§12 below pre-emptively answers all ten for a Logistics vertical, demonstrating the framework holds before external stress-test. The same exercise applies to CN-6-901..904 (Insurance, Healthcare, Education, Marketing Agency) when those stress-test sketches land.

---

## 12. Worked Walkthrough — Logistics Vertical

Logistics has not been built and is not in the Brief §6.3 future-vertical sketch list (those are Insurance, Healthcare, Education, Marketing Agency). It is chosen here precisely because (a) it lies outside Term 6's planned scope and (b) its multi-site-by-nature character stress-tests §8.3 — the only new pattern CN-6-100 introduces.

### 12.1 The 1-page sketch (per §11.1)

| # | Question | Logistics answer |
|---|----------|------------------|
| 1 | Namespace | `logistics.*` — new root; filed as amendment to CN-5-103 §16 |
| 2 | Primary Workflow | `logistics.trip` — lifecycle states: `planned → dispatched → in_transit → arrived → closed → archived`; abandoned variants: `cancelled`, `aborted_mid_transit` |
| 3 | Bill emission | `logistics.bill.ready.v1` carries `{trip_id, origin_site_id, destination_site_id, payer_party_ref, saleable_lines: [freight_charge, fuel_surcharge, demurrage], originating_workflow_ref: trip_id, business_date}` |
| 4 | Subscriptions | Foundation: `obligation.*` (advance freight payments, delivery requests from other verticals), `workflow.*`, `consent.*` (driver privacy). Universal: `inventory.stock.depleted.v1` (parts for vehicle maintenance), `accounting.period.closed.v1` (read-only signal), `checkout.settled.v1` (close trip workflow), `pack.effective.v1` (rate-card changes). |
| 5 | Pack hooks | `pack.logistics.trip_definition.must_close_within_hours = 72`; `pack.logistics.fuel_cost.recognition = at_trip_close`; `pack.logistics.tax_treatment.freight_ref = "transport_services"`; `pack.logistics.demurrage.threshold_hours = 4`; `pack.logistics.driver_compensation.basis = per_km \| per_trip \| salaried` |
| 6 | Scope | Multi-site-by-nature — manifest declares `multi_site_capable: true`; events carry `origin_site_id` + `destination_site_id`; depot-only operations (loading, inspection) carry `site_id` singular |
| 7 | Regulated hooks | Vehicle licensing, driver certification, cargo manifest documents — manifest references regulatory document templates; pack hooks declare per-jurisdiction (TZ TLB licensing, KE NTSA, etc.); §10 activation requires `regulatory_evidence_refs` for fleet license |
| 8 | Charter Law audit | **Law 4 (Flexibility)** — Logistics introduces multi-site-by-nature pattern (§8.3); the framework absorbs without modification because manifest extension `multi_site_capable` is a declared boolean, not a code branch. Universal engines that need to handle multi-site events do so via standard `origin_site_id`/`destination_site_id` payload fields, which are valid `site_id` shapes per CTR-024. |
| 9 | Mixed-vertical interaction | A workshop fabricates a window; logistics transports it to the customer site. Pattern: Workshop completes project (`workshop.deliverable.completed.v1` with `delivery_required: true`); Logistics trip planning subscribes to Foundation `obligation.created.v1` filtered for `obligation_kind: delivery_request`; Logistics emits `logistics.trip.planned.v1` referencing the obligation; on `logistics.trip.closed.v1`, the obligation resolves. Workshop and Logistics never call each other (VE2). |
| 10 | Stress-test edge case | **Mid-trip cargo damage** (per §3.13 customer-disputes pattern + UI-02 compensation symmetry) — Logistics workflow transitions to `aborted_mid_transit`; emit `logistics.cargo.damaged.v1 {trip_id, cargo_refs, damage_evidence_doc_ref}`; if `logistics.bill.ready.v1` already emitted, compensate via `logistics.bill.recalled.v1`; future insurance vertical subscribes via obligation primitive; never delete events; Document amend pattern per CN-5-006 N3 for delivery note. |

### 12.2 Narrative — Mama Halima the freight broker in Dar es Salaam

Mama Halima runs a small freight brokerage with two trucks serving Dar es Salaam → Arusha and Dar es Salaam → Mwanza. Her tenant is registered as `logistics`-only (no retail, no restaurant). She onboards via her regional agent in Dar.

**Day 1 — onboarding.** The agent activates the Logistics vertical for her tenant. Per §10, this is a regulated activation: the agent uploads her TLB transport license as `regulatory_evidence_ref`. Her two sites are registered: `dar-depot` and `arusha-receiving`. (Mwanza route uses Arusha-receiving as `destination_site_id` because Mama Halima has no Mwanza depot — `destination_site_id` may resolve to a customer-side site registered separately.)

**Day 2 — first trip.** Mama Halima's dispatcher creates a `logistics.trip` workflow: origin `dar-depot`, destination `arusha-receiving`, cargo manifest attached as CN-4-012 Document. State `planned → dispatched` when the driver leaves. Fuel issued via the workshop-style consumption pattern (Pattern B per CN-5-003 — `logistics.fuel.consumed.v1`); Inventory deducts with `source_ref: <trip_id>` per UI-03.

**Day 3 — arrival and billing.** Truck arrives in Arusha. State → `arrived`. Once cargo is offloaded, state → `closed`. `logistics.bill.ready.v1` emits with three saleable lines: freight charge (per pack rate-card), fuel surcharge (computed from `logistics.fuel.consumed.v1` total at `pack.logistics.fuel_cost.recognition: at_trip_close`), and demurrage if loading delays exceeded `pack.logistics.demurrage.threshold_hours`. Universal Checkout consumes; her customer pays via M-Pesa (per CTR-006 adapter); Accounting auto-journals revenue split between freight income and surcharge income per pack chart of accounts.

**Day 5 — disputed damage.** Customer reports two crates were damaged. Mama Halima's dispatcher logs the dispute: `logistics.cargo.damaged.v1 {trip_id: <day-2-trip>, damage_evidence_doc_ref}`. Per §3.13 disputes pattern, the original bill is partially compensated via `logistics.bill.amended.v1` (Document amend per CN-5-006 N3 — never delete). Accounting auto-journals a sales-return entry per pack rule. Mama Halima's accountant reviews the compensation evidence; no AI auto-decision (Law 3 + CTR-010 affirmed).

**Day 10 — period-close.** Accounting closes the month. Logistics workflows do not cascade-close (CN-5-104 PC11) — Mama Halima has three trips still `in_transit` at month-end. Their bills land in the next period when they emit `logistics.bill.ready.v1`; per PC8 forward-correction, this is the correct pattern, not retroactive amendment.

### 12.3 What this demonstrates

The Logistics walkthrough exercises every VE doctrine:

- **VE1** — Logistics is an engine, registered via CN-4-020, no special primitive
- **VE2** — Workshop interaction via Obligation, not direct call
- **VE3** — `logistics.*` namespace exclusive
- **VE4** — `logistics.bill.ready.v1` → Universal Checkout; Logistics never knows M-Pesa
- **VE5** — Payloads carry `site_id` family, `source_ref`, `tax_treatment_ref`
- **VE6** — No modification to Accounting, Inventory, Checkout, or any other engine
- **VE7** — `logistics.trip` is a Workflow primitive instance; not a bespoke state machine

Plus framework features:

- Multi-site-by-nature (§8.3) absorbed without framework change
- Compensation symmetry — disputed cargo handled via amendment + compensating event
- Period independence — in-transit trips at month-end do not block close
- Regulated activation — TLB license evidence captured at onboarding
- Mixed-vertical interaction — Workshop delivery obligation resolved by Logistics trip

The framework holds. No Term 6 author was consulted to write this sketch; every answer derives from CN-6-100 plus the upstream docs it transitively cites.

---

## 13. Boundaries

CN-6-100's responsibilities versus its neighbours:

| Concern | Owned by | CN-6-100 role |
|---------|----------|----------------|
| Recipe to add a new vertical | **CN-6-100** (this doc) | Authoritative |
| Vertical Boundary Doctrine (vertical vs universal vs foundation) | CN-6-101 | Pointer; CN-6-100 sets only the principle (§4.5) |
| Vertical Event Naming Conventions (syntax detail) | CN-6-102 | Pointer + summary (§5) |
| Vertical Scope Policy (site/tenant/platform detail) | CN-6-103 | Pointer + summary (§8) |
| Vertical-to-Universal Hand-Off Pattern (detailed handshake) | CN-6-104 | Pointer; §3 Step 3 and §6 summarize |
| Concrete vertical (Retail, Restaurant, Hotel, Workshop) | CN-6-001..004 | Conform to CN-6-100 |
| Vertical Bridges (cross-vertical patterns) | CN-6-005 | Pointer (§7.3) |
| Mixed-Vertical Tenants | CN-6-105 | Pointer; future verticals consult |
| Stress-test sketches (Insurance, Healthcare, Education, Marketing) | CN-6-901..904 | Each runs Flexibility Test against CN-6-100 |
| Foundation Engine Contract Model | CN-4-005 | CN-6-100 extends, never replaces |
| Foundation Extension Points / registration mechanism | CN-4-020 | CN-6-100 consumes via CTR-018 |
| Foundation Workflow primitive | CN-4-011 | CN-6-100 mandates via VE7 |
| Foundation Saleable Line / Tender value shapes | CN-4-021 | CN-6-100 mandates via VE4 |
| Universal Engine subscription wiring | CN-5-100 | CN-6-100 conforms on the subscriber-side |
| Universal Engine invariants (UI-NN) | CN-5-102 | CN-6-100 introduces VI-NN orthogonal catalog (§9.1) |
| Universal Event Glossary + namespace reservation | CN-5-103 §16 | CN-6-100 honours; new roots via amendment |
| Universal Checkout / Tender Engine | CN-5-009 | CN-6-100 mandates handoff per VE4 |
| Accounting journal sufficiency contract | CN-5-001 + CTR-030 | CN-6-100 §6.4 enforces vertical-side |
| Period-close choreography | CN-5-104 | CN-6-100 §6.5 forwards PC11 binding |
| Tax-aware engines + tax_treatment_ref | CN-5-105 | CN-6-100 §6.4 + §3 Step 8 enforce vertical-side |
| Engine catalog / pricing / combos / activation governance | Term 1 (pending) | CN-6-100 §10 publishes schema; Term 1 fills |
| Tenant UI for vertical workflows | Term 3 (pending) | CN-6-100 leaves UI surfacing to Term 3 |
| Channel adapters / external integrations | Term 7 (partial) | CN-6-100 references Term 7 for channels and peripherals |
| Cross-Term coherence / advisory-only invariant verification | Term 7 (Overseer) | CN-6-100 §14 reaffirms; Term 7 confirms at integration phase |

---

## 14. Open Items + Cross-Term Hooks

### 14.1 CTRs ratified Term 6 side (with this doc)

- **CTR-018** — Term 4's CN-4-020 registration API mechanism is sufficient as the basis of CN-6-100's recipe. Term 6 accepts the contract.
- **CTR-002** — Verticals feed saleable lines; tender is owned by Universal Checkout. Term 6 accepts the pattern; documented in §3 Step 3, §6.6, VE4.
- **CTR-024** — Verticals carry `site_id` in payload for site-scope events. Term 6 commits; documented in §6.1, §3 Step 6, §6.5.
- **CTR-026** — Verticals respect UI-03 (source-ref) and UI-09 (site_id-in-registry). Term 6 commits; documented in §6.2, §6.3.
- **CTR-030** — Vertical event payloads sufficient for Accounting journal mapping. Term 6 commits; documented in §6.4 + §6.5.
- **CTR-038** — Pre-allocated vertical namespaces (`retail.*` etc.) honoured; new roots via CN-5-103 §16 amendment. Term 6 acknowledges.

### 14.2 CTRs opened in flight (per `c757c7a`)

- **CTR-044** — `engine_kind: vertical` flag in CN-4-005 manifest. Open Term 4 side. CN-6-100 §4 uses the flag in advisory mode until ratified.
- **CTR-045** — Vertical onboarding governance content. Open Term 1 side. CN-6-100 §10 publishes schema; Term 1 fills.

### 14.3 CTRs pending upstream

- **CTR-036** — `kernel.*` namespace DC-042 doctrine check. Pending Term 4. Verticals honour the namespace exclusion without waiting for the runtime check.
- **CTR-037** — `pack.*` namespace ownership confirmation. Pending Term 4. CN-6-100 §4 N6 pattern assumes `pack.<vertical>.*` is permissible; uniformly adjustable if Term 4 ratifies differently.
- **CTR-041** — `advisor_id` registry mechanism. Pending Term 4. Vertical-specific advisors (referenced in §12) declare `advisor_ids` but activation waits.

### 14.4 CTRs pending downstream

- **CTR-004** — One checkout UI contract (Term 3). Term 6's vertical emissions are complete; UI rendering waits for Term 3 activation.
- **CTR-005** — Checkout always-on vs catalog (Term 1). Term 5 has committed always-on; CN-6-100 assumes same and will adapt if Term 1 rules otherwise.
- **CTR-010** — Advisory-only invariant cross-Term confirmation (Term 7). CN-6-100 §1.4 reaffirms Term 6 side: every vertical-specific advisor plugs into CN-4-022 + CN-5-010 with no autonomous action; Term 7 verifies at coherence review.

### 14.5 Open items inside Term 6 scope

- **VI-NN augmentation** — CN-6-100 v1 ratifies only VI-01 and VI-02. Candidates VI-03..VI-05 surface as CN-6-001..004 author; CN-6-101 (boundary doctrine) ratifies the final v1 catalog.
- **"Pharmacy = vertical or retail-with-attributes?" criteria** — CN-6-101 territory; CN-6-100 §4.5 references only.
- **Multi-jurisdiction verticals** — CN-5-105 §11 deferred to v2; CN-6-100 v1 supports single-jurisdiction binding per CTR-027. Multi-jurisdiction expansion lands with CN-5-105 v2.
- **Three-way vertical bridges** (Hotel → Restaurant → Retail gift-shop charged to room) — CN-6-005 territory; CN-6-100 §7.3 sets only the pattern.
- **Vertical state migration when Workflow primitive evolves** — Architect-phase concern (Charter §10); not a concept-phase deliverable.
- **Self-service POS expansion (D-DISC-002)** — touches Checkout, identity (CN-4-007 customer-as-actor), per-vertical kiosk workflows; CN-6-100 §3 Step 5 Workflow primitive accommodates without doctrine change; deferred per `DEFERRED-DISCUSSIONS.md`.

### 14.6 Existing-vertical irregularities the framework prevents (Brief §12, Gap B)

Brief §12 cites the existing verticals' gaps. CN-6-100 prevents recurrence:

| Existing irregularity (Brief §12 implicit) | CN-6-100 prevention |
|---------------------------------------------|----------------------|
| Bespoke per-vertical state machines (Hotel reservation, Workshop project, Restaurant table re-assign) | VE7 mandates Workflow primitive; §3 Step 5; VI-02 |
| Cross-vertical direct calls (Hotel → Restaurant for in-stay dining) | VE2 mandates obligation/universal intermediary; §7.3 |
| Verticals modifying universal event schemas (e.g., workshop adding `project_ref` to inventory event directly) | VE6 mandates additive extension via own namespace; §7 + §4.3 pack hooks |
| Inconsistent payload (some verticals include site_id, some don't) | VE5 + §6.1 mandate CTR-024 conformance; UI-09 hard-fail at universal subscription |
| Verticals computing tax independently (workshop computing VAT on quote without consulting pack) | §6.4 + §3 Step 8 mandate `pack.tax.lookup(...)`; never invent rates |
| Verticals issuing Statement Documents | §6.5 Gap D row 3; Statements come from Reporting post-close |
| Verticals pre-computing tender | §6.5 Gap D row 4; CN-5-009 K2 recomputes from `saleable_lines[]` |

### 14.7 The bar

> *"If we get CN-6-100 right, the future verticals will be easier than the existing ones were. That is the test."* (Brief §12)

CN-6-100 v1 is published in that spirit. CN-6-001..004 (Retail, Restaurant, Hotel, Workshop) will write themselves against this framework — and if any of them require CN-6-100 amendment to be written cleanly, the amendment is the work, not a workaround.

---

*— End of CN-6-100 Vertical Extension Framework v1 —*
