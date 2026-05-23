# Term 6 — Vertical Engines

> **Parent:** [BOS-CONCEPT-CHARTER.md](../BOS-CONCEPT-CHARTER.md) — read first
> **Type:** System Term (Part B)
> **Status:** Concept Phase v1.0

---

## 1. Mission

Vertical Engines capture what is **unique to each type of business**. A restaurant has tables, kitchen tickets, and split bills. A workshop has parametric geometry, cut lists, and offcuts. A hotel has rooms, reservations, and check-ins. A retail shop has baskets, multi-price layers, and refunds.

Universal Engines (Term 5) handle what every business shares. Vertical Engines handle what makes each business its own thing.

This Term is also where **flexibility lives**. The Charter promises that BOS can absorb new verticals — insurance, healthcare, marketing, logistics, education, anything — without rewriting itself. Term 6 owns that promise. It must design verticals such that the seventh and eighth and ninth vertical can join cleanly years from now.

---

## 2. Mission Question

> **"What makes a Restaurant a Restaurant and not a Retail shop? What is unique to a Hotel that no other business has? And — most importantly — how do we structure these answers so that adding 'Insurance Brokerage' in 2027 or 'Hospital' in 2028 follows the same clean pattern, without anyone touching what already exists?"**

---

## 3. Scope (In)

### 3.1 Current Vertical Engines

This Term owns concepts for:

- **Retail Engine** — basket-based POS, multi-price (business default + branch override), refunds, item-level promotions, retail↔workshop bridge (workshop items sold via POS)
- **Restaurant Engine** — table service lifecycle, orders, item state machine (Created → Confirmed → In Prep → Ready → Served → Closed), kitchen ticket routing, split billing, self-service QR ordering
- **Hotel Engine** — rooms, reservations, check-in/out, stays, folios, housekeeping status, room billing
- **Workshop Engine** — style registry, parametric geometry, cut list generation (linear bars + glass/panel guillotine), offcut tracking, material consumption events, multi-item project quotes, quote acceptance/rejection

### 3.2 The Vertical Extension Framework

This Term also owns the **meta-design**: how new verticals join. This is critical because the existing four verticals must not become a template that constrains future ones.

- **Vertical Extension Concept** — what is the recipe for adding a new vertical?
- **Vertical Boundary Doctrine** — what belongs in a vertical engine vs. in universal engines vs. in foundation?
- **Vertical Catalog Governance** — what is the approval process when a new vertical is proposed?

### 3.3 Future Verticals (Within Scope to *Plan For*, Not to *Build*)

Term 6 must design extension points such that these verticals are feasible without architectural change:

- **Insurance Brokerage** — policies, claims, premium tracking, commission to underwriters
- **Healthcare/Clinic** — patients (with strict privacy), appointments, prescriptions, billing
- **Education/School** — students, classes, fees, term schedules, parent communication
- **Marketing Agency** — campaigns, deliverables, retainers, project billing
- **Logistics/Transport** — vehicles, trips, fuel, drivers, freight charges
- **Professional Services** — billable hours, time tracking, engagements, retainers

Concept docs may sketch these as **illustrative future verticals** to test whether the extension framework holds. They are not deliverables of the concept phase — they are tests of the design.

---

## 4. Scope (Out)

This Term does NOT own:

| What | Owned By |
|------|----------|
| Accounting, Cash, Inventory, Procurement, HR, Reporting, Promotion engines | Term 5 — Universal Engines |
| Foundation kernel and primitives | Term 4 — Foundation |
| Engine catalog governance (which engines exist in the platform) | Term 1 — Platform Stewards (with input from Term 6) |
| Tenant-facing UI for vertical workflows | Term 3 — Tenant Experience (Term 6 supplies the events and contracts) |
| Agent's experience onboarding a vertical-specific tenant | Term 2 — Regional Distribution |
| Regional compliance (tax/document rules) | Term 1 (rules); Term 4 (DSL) |
| External integrations specific to a vertical (e.g., M-Pesa for restaurant bills) | Term 7 — Integration & Coherence |

---

## 5. Audiences Served

Vertical Engines indirectly serve all tenant types and the staff who run them. But Term 6's *direct* audiences are:

| Audience | What they get from Vertical Engines |
|----------|--------------------------------------|
| Term 3 (Tenant Experience) | Vertical-specific workflows to design UI around (kitchen ticket screen, room status board, cut list display) |
| Term 5 (Universal Engines) | Events to subscribe to (e.g., `retail.sale.completed.v1` → Accounting subscribes) |
| Term 1 (Platform Stewards) | Engine definitions to put in the catalog, with combos and pricing |
| Term 7 (Integration & Coherence) | Cross-vertical events (e.g., a restaurant order at a hotel-branded restaurant must work) |

Indirectly: tenants in each vertical, their staff, and their customers.

---

## 6. Key Concepts This Term Will Produce

### 6.1 Per-Vertical Concepts

| ID | Concept | What it answers |
|----|---------|-----------------|
| CN-6-001 | Retail Engine | Baskets, multi-price, refunds, item-level discounts |
| CN-6-002 | Restaurant Engine | Tables, orders, kitchen flow, split billing, self-service |
| CN-6-003 | Hotel Engine | Rooms, reservations, check-in/out, folios, housekeeping |
| CN-6-004 | Workshop Engine | Styles, geometry, cut lists, offcuts, project quotes |
| CN-6-005 | Vertical Bridges | Retail↔Workshop (sell window via POS), Hotel↔Restaurant (in-stay dining), Restaurant↔Retail (sell branded goods) |

### 6.2 Cross-Vertical Concepts

| ID | Concept | What it answers |
|----|---------|-----------------|
| CN-6-100 | Vertical Extension Framework | How to add a new vertical — the recipe |
| CN-6-101 | Vertical Boundary Doctrine | What belongs in a vertical vs. Universal vs. Foundation |
| CN-6-102 | Vertical Event Naming Conventions | The discipline of `<vertical>.<noun>.<verb>.v<n>` |
| CN-6-103 | Vertical Scope Policy | Vertical operations are almost always branch-scoped — why |
| CN-6-104 | Vertical-to-Universal Hand-Off | The standard pattern: vertical emits, universal consumes |
| CN-6-105 | Mixed-Vertical Tenants | A hotel with a restaurant and a shop — how do verticals coexist? |

### 6.3 Future Vertical Sketches (Tests of the Framework)

| ID | Concept | Purpose |
|----|---------|---------|
| CN-6-901 | Sketch: Insurance Brokerage Vertical | Does the framework support policy lifecycles, claims, premiums? |
| CN-6-902 | Sketch: Healthcare Clinic Vertical | Does it support patient privacy, appointments, prescriptions? |
| CN-6-903 | Sketch: Education Vertical | Does it support enrolment, term-based fees, parent records? |
| CN-6-904 | Sketch: Marketing Agency Vertical | Does it support campaigns, retainers, deliverable tracking? |

These sketches are not full concepts — they are stress tests for CN-6-100 (the framework).

---

## 7. Edge Cases to Consider

### 7.1 Retail Specifics

- A customer returns an item bought 6 months ago, after a price change. Refund at original or current price? (Original — but how is "original" preserved?)
- Multi-price layers: business default TZS 1,000, branch override TZS 950, active promo -10%, loyalty -5%, customer-specific deal -100. Final price computation order?
- A cashier voids a sale in progress. Is "voided cart" an event or just a discarded state? (Events — observability matters.)
- A retail item is also a workshop item (e.g., a fabricated window). Selling it via POS triggers what in Workshop?
- B2C vs B2B: a tradesman buys at wholesale. Same POS or different flow? (Likely tagged customer with role/segment.)

### 7.2 Restaurant Specifics

- A party of 8 is splitting the bill 8 ways. Then one person walks out without paying. Recovery flow?
- Two waiters tap the same table to take orders simultaneously. Conflict resolution?
- A kitchen ticket sent to the kitchen, then the customer changes their mind. How is "modify" or "cancel" propagated to the kitchen screen?
- A customer scans a self-service QR and places an order. There's no waiter involvement. Who is the "actor" on the event?
- The bar runs out of an ingredient. Does the restaurant engine auto-mark items "86'd" (unavailable), and how is this surfaced?
- A regulars' tab — they pay weekly, not per visit. Is this an obligation primitive use case?

### 7.3 Hotel Specifics

- A guest checks in but their room isn't ready (housekeeping running late). What event represents "physically not in the room yet"?
- A reservation overbooking — two guests, one room. How is the conflict represented and resolved?
- A guest extends their stay mid-stay. Folio gets longer. Pricing for the extension — at original rate or current rate?
- A no-show. The room was held; the guest never arrived. Charge? Release? Both have rules.
- A walk-in (no reservation). Folio is created on the fly. Different event chain?
- In-stay dining: the guest orders from the hotel restaurant and asks to "charge to room." How does Restaurant talk to Hotel? (Likely an `obligation` primitive — restaurant emits an obligation; hotel resolves it at checkout.)

### 7.4 Workshop Specifics

- A style is updated mid-quote. Existing quotes use old style; new quotes use new style. How is versioning visible to the fundi?
- A cut list is generated, the bar is cut, but the fundi cut it wrong. Now there's an unplanned offcut and a missing piece. Recovery?
- A customer accepts a quote, then changes their mind on one item. Modify the project quote? Re-quote? What about the price guarantee?
- A glass piece is broken during transport. Inventory loss event. But the project still needs that piece. Re-fabricate workflow?
- A workshop sells a fabricated window through Retail (POS). How does the workshop project link to the retail sale?
- Material shortage detected mid-job. The workshop must order more. Procurement triggers. The job waits.

### 7.5 Cross-Vertical (Mixed Tenants)

- A tenant has a hotel, a restaurant inside it, and a small gift shop. All three verticals enabled. Branch structure?
- A restaurant inside a hotel: revenue is restaurant's, but charges-to-room flow goes through hotel's folio. Accounting must net this correctly.
- A retail shop that does workshop fabrications occasionally. Is this two engines enabled or one with cross-feature flags?

### 7.6 Future-Proofing the Framework

- An insurance vertical wants to emit `insurance.policy.issued.v1` and have Accounting auto-journal commissions. Will Universal Accounting accept arbitrary new event types? How?
- A healthcare vertical needs patient data that must be encrypted-at-rest beyond normal levels. Can Foundation support per-vertical encryption requirements?
- A school vertical's "items" (enrolment fees) have time semantics (term-based). Is the Item primitive flexible enough?

---

## 8. Success Criteria

This Term is doing well when:

1. **Each existing vertical (Retail, Restaurant, Hotel, Workshop) has a complete concept document** an architect can build from.
2. **The Vertical Extension Framework (CN-6-100) is concrete enough** that a developer reading it can sketch a new vertical's structure in a day.
3. **At least three future verticals (Insurance, Healthcare, Education) have been sketched** and the framework was sufficient for all three.
4. **No vertical engine knows about any other vertical engine** — even when bridging (retail↔workshop, hotel↔restaurant), communication is event-based.
5. **A vertical engine can be disabled per tenant** (not in their subscription combo) without breaking other engines.
6. **Adding a new event type to a vertical does not require any change to Universal Engines** that don't care about it.
7. **A mixed-vertical tenant (hotel + restaurant + shop) operates cleanly**, with cross-vertical interactions modelled as events.

---

## 9. Architect's Role for This Term

After Concept Team writes documents, the Architect translates them into:

| Concept Output | Architect Output |
|----------------|------------------|
| Retail (CN-6-001) | Retail engine module, event/command schemas, basket state machine, multi-price resolution algorithm |
| Restaurant (CN-6-002) | Order item state machine, kitchen ticket routing design, split bill calculator, table session lifecycle |
| Hotel (CN-6-003) | Reservation state machine, folio model, housekeeping status events, in-stay charge integration |
| Workshop (CN-6-004) | Style registry schema, parametric geometry engine, cut list optimiser (linear + 2D guillotine), offcut catalogue, project quote engine |
| Bridges (CN-6-005) | Cross-vertical event subscriptions, obligation primitive usage |
| Extension framework (CN-6-100) | New-vertical onboarding checklist, registration API, scaffold/template |
| Boundary doctrine (CN-6-101) | Decision tree: should this logic live in vertical, universal, or foundation? |

**Architect's directive to Developer for Term 6:**
- Each vertical lives under `engines/<vertical>/` mirroring universal engine structure
- Vertical engines only know about Foundation primitives and their own events; never another vertical engine
- Cross-vertical interaction is *always* via the obligation primitive or event subscription, never direct calls
- Each vertical defines its scope policy upfront (almost always branch-scoped)
- New verticals follow the same module template (commands, events, service, subscriptions, projections)
- Universal engines must not be modified to support a new vertical (test for this)

---

## 10. Relationships with Other Terms

### Term 6 depends on:

| Term | Why |
|------|-----|
| Term 4 (Foundation) | Primitives (especially obligation, document, workflow), event store, extension points |
| Term 5 (Universal Engines) | Universal engines consume vertical events; vertical engines must emit them in expected shapes |
| Term 1 (Platform Stewards) | Engine catalog: which verticals exist, in what combos |

### Term 6 is depended on by:

| Term | Why |
|------|-----|
| Term 3 (Tenant Experience) | Vertical UI is built on vertical engine outputs |
| Term 5 (Universal Engines) | Universal engines subscribe to vertical events |
| Term 7 (Integration & Coherence) | Some integrations are vertical-specific (e.g., kitchen printers for Restaurant) |

### Cross-Term Hand-Offs

- ← Term 4: "Here is the obligation primitive." Term 6 (Restaurant) uses it for "charge to room."
- → Term 5: "Restaurant emits `restaurant.bill.paid.v1`. Cash, Accounting, Promotion engines subscribe."
- → Term 3: "Here's what the kitchen ticket screen needs to show." Term 3 designs the UI.
- ← Term 1: "Add 'Insurance' to the engine catalog when ready." Term 6 ships the new vertical when CN-6-100 framework is honoured.

---

## 11. Open Questions

1. **Vertical proliferation governance.** Who approves adding a new vertical? Just Term 1 (Platform Stewards)? Or also Term 7 (Coherence) to check the framework was followed?
2. **Vertical bridges as first-class.** Is "Retail↔Workshop" a separate concept (a bridge engine), or just subscriptions? Where does composite logic live?
3. **Item identity across verticals.** A "Window — 1.2m × 1.5m, Aluminium" is a workshop deliverable AND a retail SKU. Same item ID? Two different items linked? The Item primitive must support this answer.
4. **Branch-scope universality.** Are all vertical engines always branch-scoped, or are there exceptions? (Workshop quote generation might be business-scoped.)
5. **Pricing precedence in verticals.** Each vertical might have its own pricing nuances. Is this a vertical concern or a shared "pricing" primitive in Foundation?
6. **Multi-currency in verticals.** A workshop quote for an export customer in USD. The workshop engine must handle this somewhere — but how? (Likely a primitive-level concern, not workshop-specific.)
7. **Vertical-specific compliance.** A pharmacy is a retail shop but has additional documentation rules. Is this a "retail" thing or a "pharmacy" sub-vertical?
8. **AI advisors per vertical.** Should there be a "Restaurant Advisor" (predicting kitchen rush times) or only generic advisors? Where does vertical-specific intelligence live?
9. **The "almost a vertical" problem.** A salon. A car wash. A small repair shop. Are these their own vertical, or all "Services" sharing one engine?
10. **End-customer identity across verticals.** A regular customer of a tenant's restaurant + retail + hotel. Three identities or one? Is "Customer" a primitive or a vertical concept?

---

## 12. Notes on Existing Repository

The existing repository has mature implementations of all four current verticals:
- `engines/retail/` — basket POS, multi-price, refunds
- `engines/restaurant/` — tables, orders, kitchen workflow (Phase 6 + GAP-09)
- `engines/hotel/` — rooms, reservations, check-in/out
- `engines/workshop/` — parametric geometry, cut lists (Phase 6 + GAP-02), Style Registry & Quote Engine (Phase 16), Multi-Item Project Quotes (Phase 17), Glass Cutting (Phase 18)

**Notable from existing code:**
- Workshop is highly developed — parametric formula engine, fundi cutting sheets, glass guillotine optimisation
- Restaurant has kitchen workflow and split billing already wired
- Cross-engine subscriptions (Workshop → Inventory, Retail → Cash, etc.) are wired

**Concept Phase guidance:** The verticals in code are good *reference for vertical complexity*. But Term 6's first job is to write the **Vertical Extension Framework (CN-6-100)** — because the existing four verticals were built without that explicit framework, which is partly why the system has gaps and irregularities.

If we get CN-6-100 right, the future verticals will be easier than the existing ones were. That is the test.

---

## 13. First Tasks for This Term

When this Term starts concept work, begin in this order:

1. **CN-6-100 first** — the Vertical Extension Framework, before any specific vertical doc.
2. **CN-6-101 second** — the boundary doctrine, deciding what belongs in a vertical.
3. **CN-6-102 third** — event naming conventions.
4. **CN-6-103 fourth** — scope policy for verticals.
5. **CN-6-104 fifth** — vertical-to-universal hand-off pattern.
6. **CN-6-001 sixth** — Retail (simplest existing vertical).
7. **CN-6-002 seventh** — Restaurant.
8. **CN-6-003 eighth** — Hotel.
9. **CN-6-004 ninth** — Workshop (most complex; benefits from the rest being settled).
10. **CN-6-005 tenth** — Bridges between verticals.
11. **CN-6-105 eleventh** — Mixed-vertical tenants.
12. **CN-6-901 to CN-6-904** — future vertical sketches, as stress tests.

---

## 14. The Flexibility Test

The Charter (Law 4) promises flexibility — that BOS can absorb new verticals. This Term is the proving ground.

The flexibility test, to be run at the end of the concept phase:

> Pick a vertical that has never been built (e.g., a marketing agency).
> Have someone unfamiliar with BOS read CN-6-100 (the framework) and write a 1-page sketch of how that vertical would fit into BOS.
> If they can do it without confusion, the framework holds.
> If they cannot, the framework is incomplete and we revise it.

This test is non-negotiable. If we fail it, we are not yet done.

---

## Standing Decisions & Cross-Term Work

> Read `DECISION-LOG.md`, `CROSS-TERM-COLLABORATION.md`, and `CROSS-TERM-REQUEST-REGISTER.md` before discussing these. File a CTR for every dependency. Bring each section to the **Concept Lead** and **Overseer (Term 7)**, discuss and agree first — the proposal is written only **after** agreement, and nothing is final without it.

| New Concept | From | This Term must decide |
|-------------|------|------------------------|
| CN-6-106 | D-001 | The **vertical → checkout hand-off**: each vertical (Retail basket, Restaurant bill, Hotel folio, Workshop accepted quote) emits **Saleable Lines** into the Universal Checkout and owns **nothing** about tender, change, or receipt |

**CTRs to expect:** files CTR-001 (line/tender primitive → 4) and CTR-002 (lines hand-off → 5); receives CTR-004 (checkout UI ← 3) and CTR-005 (catalog ← 1); confirms CTR-010 (advisory-only). Future verticals reuse this same hand-off — a test of the CN-6-100 framework.

---

*— End of Term 6 Brief —*
