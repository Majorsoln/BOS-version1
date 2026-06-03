# CN-6-101 — Vertical Boundary Doctrine

> **Term:** 6 — Vertical Engines
> **Status:** Concept Phase v1.0 — Draft for Overseer review
> **Branch:** `claude/brave-johnson-S7Lky`
> **Reading order:** BOS-CONCEPT-CHARTER → DECISION-LOG → TERM-6-VERTICAL-ENGINES → CN-6-100 → CN-4-011 → CN-5-100..105 → **this doc**
> **Ordering authority:** TERM-6 Brief §13 — CN-6-101 is the second Term 6 deliverable; CN-6-100 (the recipe) precedes; CN-6-102..104 elaborate downstream.

---

## 1. Purpose & Boundary

### 1.1 Mission

CN-6-101 answers Brief §6.2's question:

> *"What belongs in a vertical engine vs in universal engines vs in foundation?"*

CN-6-100 answered *how* to add a vertical. CN-6-101 answers the *prior* question every architect asks when looking at a feature, a concept, a piece of data:

> *"Should this even BE a vertical concern at all? Or is it universal? Or is it a foundation primitive?"*

This is the **decision tree**. It applies whenever someone proposes a new feature, a new event, a new pack hook, a new primitive. The wrong placement costs years of refactoring and creates the precise gaps Brief §12 cites in the existing four verticals — verticals that grew without this doctrine and now carry irregularities.

### 1.2 DOES vs DOES NOT

| CN-6-101 DOES | CN-6-101 DOES NOT |
|---------------|-------------------|
| Define the eight Boundary Doctrine principles (BD1–BD8) | Rewrite Charter §4 five-layer architecture — it honours, does not amend |
| Publish a decision tree for layer placement | Audit existing engines or verticals against the doctrine (Architect phase) |
| Close Brief §11.7 (Pharmacy = vertical or retail-with-attributes?) | Define Pharmacy's full vertical doc (that is CN-6-005 or a future CN-6-006) |
| Close Brief §11.2 (Vertical Bridges as first-class) | Define cross-vertical bridge patterns (that is CN-6-005) |
| Address Brief §11.9 ("almost a vertical": salon, car wash, repair) | Decide salon's final placement — that needs CN-6-905 stress-test sketch |
| Catalog anti-patterns (with concrete BOS evidence from Term 5 work) | Mechanize every anti-pattern as a doctrine check — those that are mechanizable seed CTR-046 to Term 4 |
| Ratify VI-NN candidates from CN-6-100 §9.4 (VI-03, VI-04) with boundary basis | Author primitive-level decisions on data sensitivity (VI-05 deferred — needs Term 4 mjadala) |
| Ground all doctrine in real Tanzanian business reality (Mama Amina kiosk-to-duka, Nakumatt supermarket scale, Lodge Serengeti hospitality, kinyozi + boda boda mechanic services cluster) | Specify UI surfaces for vertical workflows (Term 3 territory) |

### 1.3 Audience

CN-6-101 is read by **all seven Terms plus the Architect**, because boundary placement is a cross-Term concern:

| Audience | Why they read it |
|----------|------------------|
| Term 6 (self-reference) | When authoring CN-6-001..005 and CN-6-901..904; when reviewing whether a proposed Term-6 concept actually belongs in Term 6 |
| Term 4 (Foundation) | When deciding whether a proposed primitive truly is foundational or whether it is a misplaced universal engine concern |
| Term 5 (Universal Engines) | When deciding whether a proposed feature belongs in a universal engine or whether it is vertical-specific bleeding into universal scope |
| Term 1 (Platform Stewards) | When deciding which engines enter the catalog and at what tier |
| Term 2 (Regional Distribution) | When agents propose tenant-onboarding flows that imply new engine boundaries |
| Term 3 (Tenant Experience) | When designing UI that crosses vertical/universal layers |
| Term 7 (Integration & Coherence / Overseer) | When mediating CTRs whose root cause is a placement disagreement |
| Architect | When designing implementations after Concept Phase closes |

### 1.4 Charter Compliance

CN-6-101 elaborates Charter §4 (five-layer architecture) without amending it. It honours the Six Laws:

| Law | How CN-6-101 honours it |
|-----|--------------------------|
| Law 1 — State from events only | BD1 places event-store mechanism in Foundation; verticals contribute event vocabulary, not storage |
| Law 2 — Engines isolated | BD7 closes the "bridge engine" temptation; cross-vertical communication via primitives + events only |
| Law 3 — AI advisory only | BD5 split-it pattern places advisor *framework* (universal) separate from advisor *content* (vertical); never autonomous |
| Law 4 — Flexibility first-class | BD4 (push down) + BD8 (no hypothetical-driven up-promotion) preserve the flexibility CN-6-100 set up |
| Law 5 — Compliance configured | BD5 split-it places tax/promotion/pricing *mechanisms* universal and *content* in packs + vertical context |
| Law 6 — Distribution regional | BD8 prevents premature platform-scope generalization on speculative future-vertical reasoning |

### 1.5 Gap Awareness Summary

CN-6-101 is published while the following are outside its scope or pending:

| Gap | Status | CN-6-101 handling |
|-----|--------|--------------------|
| Existing-vertical retrofit to BD doctrine | Out of Concept Phase scope | §10 acknowledges; Architect phase reconciles AS-IS vs SHOULD-BE per CN-6-001..004 |
| CTR-044 `engine_kind: vertical` | Pending Term 4 | §6 cites; doctrine applies regardless of flag mechanism |
| CTR-045 vertical onboarding governance | Pending Term 1 | §11.1 Pharmacy worked example references; Term 1 fills |
| CTR-046 (new — proposed) mechanizable anti-patterns → DC checks | Will open with this doc to Term 4 | §10 marks mechanizable rows; CTR carries forward |
| CTR-047 (new — proposed) data_sensitivity_tier primitive mjadala | Will open with this doc to Term 4 | §12 defers VI-05 pending |
| Salon / light services cluster final placement | Pending CN-6-905 stress-test + Concept Lead market-priority ruling | §9 + §11.2 set criteria; placement decision deferred but elevated to high-priority |
| D-DISC-001 tenant-customer promotion UX boundary | Deferred to Term 3 activation | §14 cross-reference |
| D-DISC-002 POS self-service expansion | Deferred per `DEFERRED-DISCUSSIONS.md` | §14 cross-reference; BD3 confirms vertical owns self-service specifics |

---

## 2. The Three-Layer Question — Decision Tree

When an architect, a Term author, or a CTR participant proposes a new feature, a new event type, a new pack hook, or a new primitive, the layer-placement question runs through this tree:

```
                  Proposed concept X
                          │
                          ▼
      ┌───────────────────────────────────────────────┐
      │ Does X depend on a specific business domain?  │
      │ (retail vs hotel vs restaurant vs workshop vs │
      │  pharmacy vs logistics vs ...)                │
      └───────────────────────────────────────────────┘
                  │NO                       │YES
                  ▼                         ▼
      ┌─────────────────────────┐  ┌──────────────────────────┐
      │ Is X a reusable building │  │ Do TWO OR MORE existing  │
      │ block that any engine    │  │ or realistic verticals    │
      │ might need? (ledger,     │  │ need X with the SAME      │
      │ workflow, party,         │  │ semantics?                │
      │ obligation, document...) │  │                           │
      └─────────────────────────┘  └──────────────────────────┘
                  │YES        │NO         │YES         │NO
                  ▼           ▼           ▼           ▼
            ┌─────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
            │ FOUND-  │ │ UNIVERSAL│ │ UNIVERSAL│ │ VERTICAL │
            │ ATION   │ │ ENGINE   │ │ ENGINE   │ │ ENGINE   │
            │ (BD1)   │ │ (BD2)    │ │ (BD2)    │ │ (BD3)    │
            └─────────┘ └──────────┘ └──────────┘ └──────────┘
                  │           │            │           │
                  └───────────┴────────────┴───────────┘
                                    │
                                    ▼
              ┌─────────────────────────────────────────┐
              │ Sanity checks before finalising:        │
              │  • Push down preferred (BD4)            │
              │  • Split-it where shared+context (BD5)  │
              │  • Regulated ≠ automatically vertical   │
              │     (BD6 3-of-3 test)                   │
              │  • No "bridge engines" (BD7)            │
              │  • Hypothetical futures don't justify   │
              │     premature up-promotion (BD8)        │
              └─────────────────────────────────────────┘
```

### 2.1 How to use the tree

1. **Run X through the first question** ("Does X depend on a specific business domain?"). If the answer is genuinely "no" (the concept is domain-neutral — like "a workflow," "a document," "an obligation"), proceed left. If "yes," proceed right.
2. **For domain-neutral concepts**, ask the second-left question. If X is a reusable building block any engine might need (the ledger, the workflow primitive, the party primitive), it is **Foundation** (BD1). If X is a domain-neutral *service* (accounting, cash management, inventory) — not a building block but a complete capability used by every business — it is **Universal Engine** (BD2).
3. **For domain-specific concepts**, ask the second-right question. If two or more existing or realistic verticals would need X with the *same semantics* (not just superficially similar names), X is **Universal Engine** (BD2). If only one vertical genuinely needs X, X is **Vertical Engine** (BD3).
4. **Run all sanity checks** before finalising. The single most common mistake is putting in a vertical what should be a primitive (BD4 violation) — and the second most common is creating a "bridge engine" (BD7 violation).

---

## 3. Doctrine — BD1–BD8

These eight principles govern layer placement for every concept. They do not override Charter Laws or VE1–VE7 from CN-6-100; they elaborate them at the placement boundary.

**BD1 — Foundation owns what every engine needs and never changes per business domain.** Primitives (ledger, item, party, workflow, obligation, document, consent, identity, inventory movement), mechanism (event store, command bus, scope policy, registration, security), doctrine enforcement (CN-4-019 checks). The anti-test: if the concept varies by vertical OR by universal engine, it is NOT foundation.

**BD2 — Universal Engines own what every business needs regardless of vertical.** The acceptance test: a duka + a restaurant + a hotel + a workshop all use the concept with the same semantics. The anti-test: if a realistic vertical (Logistics, Insurance, Healthcare) does NOT need it, it is not universal — it is either vertical-specific or premature.

**BD3 — Verticals own what is unique to one business type.** The acceptance test: the concept does not make sense for businesses outside this vertical. The anti-test: if multiple verticals would want it, it belongs at universal layer (with vertical context per BD5) or as a primitive (per BD1).

**BD4 — When in doubt, push DOWN, not UP.** Foundation primitives are reused across all engines — pushing logic down maximises reuse. Vertical-specific code is hardest to evolve as verticals proliferate. The hierarchy: Foundation > Universal Engine > Vertical Engine. When uncertain, the lower placement is the safer bet.

**BD5 — When verticals share a concept, split it: Universal mechanism + Vertical context.** Examples that already exist: pricing = universal pricing-resolution + per-vertical pricing context; promotion = universal Promotion Engine + per-vertical trigger events (CN-5-007 §16); tax = universal pack-driven Tax + per-vertical `tax_treatment_ref` category (CN-5-105 N7). The mechanism stays one; the context stays many.

**BD6 — Regulated or compliance specificity does NOT automatically make a vertical.** A regulated retail (Pharmacy OTC) may still be retail-with-pack-rules if it shares workflow vocabulary with retail. The 3-of-3 test: does the proposed vertical have (1) its OWN namespace, (2) its OWN Workflow primitive instance, (3) its OWN regulatory event types that the parent vertical wouldn't have? Only if 3-of-3 pass is it a new vertical. Pharmacy passes 3-of-3 for prescription operations (§11.1). Salon currently fails 3-of-3 (§11.2).

**BD7 — Cross-vertical relationships do NOT justify "bridge engines."** Hotel↔Restaurant in-stay charge-to-room is solved via Obligation primitive (CN-4-011) + Hotel folio Workflow — not by inventing a "Hospitality Bridge Engine." Workshop↔Retail sell-via-POS is solved via Universal Inventory + Universal Checkout — not by inventing a "Manufacturing-Retail Bridge." A bridge engine would duplicate Accounting + Cash and couple verticals that should stay isolated (VE2). **Brief §11.2 closure: bridges are patterns, not engines.**

**BD8 — Future-vertical hypothetical does NOT justify premature Foundation/Universal placement.** Only if at least two existing or realistic verticals demonstrate need today should a concept move up. YAGNI applied to layers. Hypotheticals do not earn promotion. The exception is when the *pattern* is concrete (Logistics multi-site-by-nature per CN-6-100 §8.3) — patterns absorb without architectural change; engines do not.

---

## 4. Foundation Layer — What Belongs and the Test

### 4.1 Acceptance test for Foundation placement (BD1)

A concept belongs in Foundation if and only if **all** of these hold:

1. **Domain-neutral** — it has no business-vertical specificity; the same primitive serves retail and clinic alike.
2. **Universally reusable** — every engine (universal or vertical) might need it.
3. **Mechanism, not policy** — it implements the *how* of state change, not the *what* of business rules.
4. **Stable over decades** — adding a new vertical does not require modifying the primitive.

### 4.2 Foundation owns (per D-007 ratification)

- **Primitives (nine)**: ledger, item, party, workflow, obligation, document, consent, identity, inventory movement
- **Mechanism**: event store + hash chain (CN-4-002, CN-4-003), command bus (CN-4-004), engine contract model (CN-4-005), multi-tenant isolation (CN-4-006), scope policy (CN-4-005 + CN-5-101), replay engine (CN-4-009), projection framework (CN-4-010), document engine (CN-4-012), audit log (CN-4-008), security primitives (CN-4-016), time authority (CN-4-014)
- **Doctrine**: event sourcing doctrine (CN-4-001), compliance DSL (CN-4-015), doctrine enforcement (CN-4-019), extension points / registration API (CN-4-020), advisor framework + AI guardrails (CN-4-013, CN-4-022), kernel boundary + Developer-AI (CN-4-023), saleable line + tender value shapes (CN-4-021)

### 4.3 Anti-tests — what is NOT Foundation

- "Period close" was correctly removed from the Ledger primitive (D-007 #13) — period close is an Accounting (Universal) concern. The Ledger knows entries and balances; closing a period is policy, not mechanism.
- A hypothetical "Tax Engine" at Foundation level would violate BD1: tax content varies by jurisdiction (pack-driven); the *mechanism* (recognition, journaling) is policy embedded in Accounting (Universal). CN-5-105 correctly placed tax-awareness as cross-engine wiring, not a foundation primitive.
- A hypothetical "Notification primitive" at Foundation level would violate BD1: notifications are channel adapters (Term 7) plus consent-gated outreach (Universal Promotion). The mechanism does not belong in the kernel.

---

## 5. Universal Engine Layer — What Belongs and the Test

### 5.1 Acceptance test for Universal Engine placement (BD2)

A concept belongs in a Universal Engine if and only if **all** of these hold:

1. **Cross-vertical universality** — every realistic business needs it. The minimum bar is "all four current verticals (Retail, Restaurant, Hotel, Workshop) use it with the same semantics."
2. **Stateful capability, not just a building block** — it's a complete service, not a primitive that other engines compose.
3. **Builds on Foundation primitives, doesn't reinvent them** — Universal Accounting uses the Ledger primitive; it doesn't have its own ledger.
4. **Bounded scope** — it does one thing well (Accounting, Cash, Inventory, Procurement, HR, Reporting, Promotion, Checkout). It does not creep into adjacent capabilities.

### 5.2 Universal Engines own (per CN-5-* scope-complete)

- **CN-5-001 Accounting** — management-truth double entry, journal mapping, period close
- **CN-5-002 Cash Management** — drawers, sessions, reconciliation, deposits
- **CN-5-003 Inventory** — movements, FIFO/LIFO, lots, offcuts, costing
- **CN-5-004 Procurement** — requisition → PO → GRN → invoice → payment
- **CN-5-005 HR & Payroll** — employees, payroll, statutory deductions
- **CN-5-006 Reporting & BI** — KPIs, projections, snapshots, statements
- **CN-5-007 Promotion** — discounts, vouchers, loyalty, cost-share
- **CN-5-009 Universal Checkout / Tender** — tender, splits, change, receipt issuance
- **CN-5-010 AI Advisors Wiring** — advisor framework binding per universal engine

### 5.3 Anti-tests — what is NOT Universal

- **"Kitchen ticket management" is NOT universal.** Only restaurants need kitchen ticket vocabulary. BD2's "all four verticals use it" test fails — retail, hotel (lobby-only), and workshop have nothing analogous. It is vertical (Restaurant) per BD3.
- **"Cut list optimisation" is NOT universal.** Only workshops fabricate from parametric geometry. BD2 fails. It is vertical (Workshop).
- **"Room state board" is NOT universal.** Only hotels manage room availability. BD2 fails. It is vertical (Hotel).
- **A speculative "Universal Service Engine" covering salon + barber + car wash today** would violate BD8 — those are concrete light-service verticals that need their own concept work (CN-6-905 stress-test sketch).

---

## 6. Vertical Engine Layer — What Belongs

### 6.1 Acceptance test for Vertical placement (BD3)

A concept belongs in a Vertical Engine if and only if:

1. **Vertical-unique semantics** — the concept does not make sense for businesses outside this vertical.
2. **Vocabulary specificity** — the lifecycle states, event types, or commands have meaning only inside this vertical.
3. **No universal-engine substitute** — the concept cannot be expressed as a Universal Engine plus vertical context (if it can, BD5 split-it applies; the mechanism goes universal).
4. **No primitive substitute** — the concept is not just an instance of Workflow + Party + Obligation (if it is, the *mechanism* is the primitive; the *vocabulary* is the vertical contribution per VE7).

### 6.2 What lives in a vertical

Per VE1 (vertical is engine first) and the recipe (CN-6-100 §3), a vertical contains:

- **Workflow vocabulary** — lifecycle states, valid transitions, per-transition guards (vertical contributes vocabulary; Workflow primitive provides mechanism)
- **Domain event types** — events whose payload semantics are vertical-specific
- **Commands** — vertical-specific command names with vertical-specific validation
- **Pack hooks** — `pack.<vertical>.<sub_domain>.<key>` for jurisdiction-tunable parameters
- **Saleable line emission** — `<vertical>.bill.ready.v1` per VE4
- **Subscriptions to Foundation + Universal events** — per VE2

### 6.3 Worked prose example — Restaurant Kitchen Ticket

A common BD3 case the framework must explain: why is "kitchen ticket" a vertical concept and not a universal one?

The kitchen ticket workflow vocabulary — `created → confirmed → fired → in_prep → ready → picked_up → served` — has no meaning for retail (sales are point-of-transaction), hotel (room-state is the analogue, not ticket-state), or workshop (cuts and projects are not "ordered" by a kitchen). BD2's universality test fails: retail/hotel/workshop don't share the vocabulary or the operation. BD3 passes: only restaurants have kitchens that fire tickets.

The *mechanism* underneath is still Foundation Workflow primitive (VE7). The ticket itself is a payload-level data structure carried inside `restaurant.kitchen.ticket.fired.v1` and related events. The Workflow primitive provides persistence, audit, and replay; the Restaurant vertical provides the vocabulary that means "fired" implies "the kitchen is now cooking."

A hypothetical "Universal Kitchen Engine" would serve zero non-restaurant tenants — a BD2 anti-test failure. A "Universal Service Order Engine" generalising kitchen tickets to salon appointments and workshop cut-orders would mash incompatible semantics together — BD5 split-it would resolve into multiple verticals using Workflow primitive with their own vocabularies, which is what we have already.

---

## 7. The "Push Down" Discipline (BD4 Elaborated)

### 7.1 Why push down beats push up

When uncertain whether a concept is vertical or universal, place it lower. Reasons:

1. **Reusability multiplies value.** A primitive serves every engine forever; a vertical concept serves one business type. Lower placement maximises future reuse.
2. **Refactor pain is asymmetric.** Pushing a vertical concept down to universal (later) requires migrating every other vertical to consume the universal version. Pushing a universal concept up to vertical (later) requires extracting it from every engine that already integrated it. Both are painful — but the existing-verticals retrospective gaps (Brief §12) show "we shouldn't have hardcoded this in retail" is the more common regret.
3. **Doctrine evolution is slower than engine evolution.** Foundation primitives change rarely (decades); universal engines change at business-cycle speed (years); vertical engines change at market-cycle speed (months). The lower the layer, the more stable. Stability is a feature.

### 7.2 When push-down is wrong

The push-down discipline has a limit. Pushing a *genuinely* vertical concept down to universal creates BD2 anti-test failure: the universal engine carries dead weight for engines that don't need it. Examples of overshoot:

- Pushing "kitchen station routing" into Universal Workflow primitive would force every vertical to encode kitchen-station semantics they don't use.
- Pushing "controlled-substance ledger" into Universal Accounting would couple every duka's accounting to pharmacy regulatory vocabulary.

The test: if the concept *requires* vertical-specific vocabulary to be useful, it belongs in the vertical. If the concept can be expressed in vertical-neutral terms and the vertical contributes only a *label* or *category*, it can be pushed down.

---

## 8. The Split-It Pattern (BD5 Elaborated)

### 8.1 What "split-it" means

When multiple verticals share a concept but with vertical-specific specifics, split it into:

- **Universal mechanism** — the shared infrastructure (one engine, one event family, one resolution chain)
- **Vertical context** — the per-vertical inputs that drive the mechanism (event payload tags, pack hooks, category references)

The mechanism stays one; the context stays many. This is how a single Universal Promotion Engine serves Retail happy-hour, Hotel room-rate windows, Restaurant lunch specials, and Workshop repeat-customer loyalty — all via the same engine, with each vertical contributing its own trigger events.

### 8.2 Existing split-it instances (already in production)

| Concept | Universal mechanism | Vertical context |
|---------|----------------------|-------------------|
| Tax computation | CN-5-105 cross-engine wiring + pack `tax:` subsection | Per-vertical `tax_treatment_ref` category in payload (`accommodation`, `transport_services`, `prescription_drug`, etc.) |
| Promotion | CN-5-007 Promotion Engine | Per-vertical trigger events (`restaurant.happy_hour.started.v1`, `hotel.room_rate.window.opened.v1`, `workshop.repeat_customer.recognised.v1`) |
| Checkout / tender | CN-5-009 Universal Checkout | Per-vertical `<vertical>.bill.ready.v1` emission |
| Accounting journals | CN-5-001 Accounting | Per-vertical revenue/expense event payloads with `chart_of_accounts_hint` |
| Inventory consumption | CN-5-003 Inventory | Per-vertical consumption events (Pattern B: `restaurant.ingredient.consumed.v1`, `workshop.cut.executed.v1`) |
| AI advisors | CN-4-022 framework + CN-5-010 wiring | Per-vertical advisor IDs and audience contracts |
| Period close | CN-5-104 choreography | Per-vertical events do not cascade-close (PC11); forward-correction (PC8) |

### 8.3 When split-it is wrong

Split-it is wrong when the universal mechanism would carry vertical-specific assumptions in its core logic. Test: if removing one vertical means rewriting the universal mechanism, the split was misdrawn. The universal layer must be vertical-blind in its mechanism; only the *context inputs* know about verticals.

Example of split-it done right: CN-5-001 Accounting doesn't know what `accommodation` means; it knows how to map a `chart_of_accounts_hint` through a pack-driven rule table. The pack tells Accounting "accommodation maps to GL 4001 Room Revenue under TFRS-TZ"; Accounting maps. Verticals come and go; the mechanism is untouched.

---

## 9. The Pharmacy Decision (Brief §11.7 Closure) + "Almost-a-Vertical" (Brief §11.9)

### 9.1 Brief §11.7 — Pharmacy = vertical or retail-with-attributes?

**Closure: Pharmacy IS its own vertical for prescription operations. OTC operations remain retail via Mixed-Vertical Tenant pattern (CN-6-105).**

The BD6 3-of-3 test applied to pharmacy:

| Test dimension | Prescription operations | OTC operations |
|----------------|-------------------------|-----------------|
| (1) Own namespace? | ✓ `pharmacy.*` (CTR-038 pre-allocated) | ✗ shares `retail.*` semantically |
| (2) Own Workflow instance? | ✓ `pharmacy.prescription` (validate → consult → dispense → archive); has consultation, prescription verification, controlled-substance ledger entry — none exist in retail | ✗ identical to `retail.sale` (basket → settle); no prescription, no consultation |
| (3) Own regulatory event types? | ✓ `pharmacy.controlled_substance.dispensed.v1`, `pharmacy.prescription.validated.v1`, TFDA reporting events — none meaningful in retail | ✗ no regulatory vocabulary; OTC paracetamol is a retail SKU |

Prescription = 3-of-3 pass → Pharmacy IS a vertical for this side.
OTC = 1-of-3 → not a separate vertical; OTC sales reuse retail mechanism.

**The pattern: Mama Amina's tenant activates BOTH `retail.*` (her existing duka, now also selling OTC) AND `pharmacy.*` (her new prescription counter). CN-6-105 Mixed-Vertical Tenants is the framework for this; CN-6-101 BD6 + §11.1 + §11.6 justify it.**

### 9.2 Brief §11.9 — "Almost a vertical": salon, car wash, repair

**Status: Decision deferred. Concrete framework criteria published; placement decision deferred to CN-6-905 stress-test sketch + Concept Lead market-priority ruling.**

The cluster is concrete and high-volume in the Tanzanian SME landscape:

- **Salon** — kinyozi (barber), mama saluni (women's salon), nail bar, beauty parlor
- **Light vehicle service** — car wash, boda boda mechanic, fundi pikipiki
- **Small repair** — cobbler (fundi viatu), fundi simu (mobile phone repair), knife sharpener, mat weaver

All share: appointment-or-walk-in service Workflow + service delivery + optional parts/product retail + cash or mobile-money settlement.

**BD6 3-of-3 test currently fails for each**:

| Test dimension | Salon | Car wash | Boda mechanic |
|----------------|-------|-----------|---------------|
| (1) Own namespace need? | Possibly `services.*` or per-trade | Possibly shared | Possibly shared |
| (2) Own Workflow distinct from Workshop project? | Marginal (no parametric geometry; service-time-based not material-based) | Marginal | Closer to Workshop (parts + labour) |
| (3) Own regulatory event types? | Minimal (hygiene local-licensing only) | Minimal | Minimal (vehicle service records) |

Three candidate paths:

1. **Subsume to Workshop** with pack hooks per service-type (workshop-as-services with `pack.workshop.service_type` ∈ {salon, car_wash, repair, ...}). Risk: dilutes Workshop's manufacturing/fabrication identity.
2. **Generic `services.*` vertical** with `services.appointment` Workflow covering salon + barber + nail + car-wash. Risk: BD3 "concept does not make sense outside the vertical" is weak when the vertical includes too many distinct trades.
3. **High-volume few become own verticals** (e.g., `salon.*` if data shows it's a dominant SME segment), with remainder as Workshop sub-types or `services.*`.

**Per BD8 + the Concept Lead's "real lives" bar**: this is concrete (not hypothetical), high-volume in Tanzania, and meets BD8's ≥2 threshold easily. The decision is NOT "indefinite defer." It is **elevated to high-priority post-CN-6-104** — CN-6-905 stress-test sketch becomes the next batch of work after the cross-cutting framework docs close, with Concept Lead market-priority ruling determining the path among (1), (2), (3).

### 9.3 What CN-6-101 leaves to CN-6-905

- The specific Workflow vocabulary for service-business
- The pack-hook design for trade-specific variations
- The decision between path (1), (2), (3) based on Tanzanian market data
- The CN-6-105 Mixed-Vertical implications (some tenants will run salon + retail of products like shampoo)

---

## 10. Anti-Patterns Catalog

These are the placement mistakes Brief §12 hints at and CN-6-101 prevents. Each is mechanizable or judgmental; mechanizable rows seed CTR-046 to Term 4 for doctrine-check additions.

### 10.1 AP1 — Premature Universalization

Putting a feature in a Universal Engine because "someone might want it eventually." Symptom: a Universal Engine grows scope creep adding capability for hypothetical verticals.

**Mechanizable?** No — judgmental. The CN-4-019 doctrine gate cannot detect "premature." Review-checklist enforcement at the registration gate.

**Three concrete BOS examples from Term 5 work where premature universalization was correctly avoided:**

- **Procurement was NOT promoted to "Universal Supply Chain"** (CN-5-004). It was tempting to add freight tracking + warehouse routing + last-mile delivery management ("logistics features") to Procurement. BOS resisted — those belong to a future Logistics vertical (per CN-6-100 §12 worked example). Procurement stayed tight: requisition → PO → GRN → invoice → payment. Logistics features will land in a `logistics.*` vertical when 2+ tenants demonstrate need. **BD8 in action.**
- **Tax-Awareness was scoped to engines, NOT to a "Tax Engine"** (CN-5-105). Tax computation is content (pack-driven) plus a cross-cutting concern wired across Accounting, Procurement, Checkout, HR. A premature "Universal Tax Engine" would have duplicated Accounting and added unnecessary mechanism with no stateful workflow of its own. **BD5 split-it applied: pack content + per-engine wiring.**
- **Cash didn't become "Universal Treasury"** (CN-5-002). It was tempting to expand to FX hedging + bank loan management + investment portfolio tracking. BOS resisted — those are future verticals (FX bureau, savings cooperative, asset manager). Cash stayed focused on tills, sessions, reconciliation, deposits. **BD3 + BD8 combined.**

Lesson: every Universal Engine in CN-5-* exists because all four current verticals need it with the same semantics. Each universal feature passes BD2's "all four current verticals use it" test. Future universalisation waits until ≥2 verticals concretely demonstrate need.

### 10.2 AP2 — Vertical-to-Vertical Direct Subscription

A vertical subscribes to another vertical's events, creating coupling and violating VE2.

**Mechanizable?** Yes — CTR-046 row 1. Doctrine check: at CN-4-020 registration, `subscribes_to[*].event_type` MUST NOT begin with any reserved vertical root other than the registering engine's own namespace.

**Why it happens:** A naive solution to Hotel-needs-Restaurant-charge: Hotel subscribes to `restaurant.bill.ready.v1`. This couples them — Restaurant version bump breaks Hotel; Restaurant decommission breaks Hotel; the "isolation" of Law 2 is gone. **Right pattern**: Obligation primitive (§11.4 worked example).

### 10.3 AP3 — Bridge Engine

Creating a new engine specifically to mediate two verticals. Violates BD7.

**Mechanizable?** No — judgmental at architecture-review. The proposed "engine" would have its own manifest; the CN-4-020 doctrine gate could not distinguish "Hospitality Bridge Engine" from any other engine without a heuristic check.

**Symptom:** the proposed engine's emits and subscribes are dominated by two specific vertical namespaces; the engine does no meaningful work otherwise. Review-checklist enforcement.

### 10.4 AP4 — Vertical Computing What Pack-Content Owns

A vertical computes tax rates, recognition rules, or chart-of-accounts mapping itself rather than consulting the pack.

**Mechanizable?** Partially — CTR-046 row 2. Doctrine check: for events declared as Accounting-sufficient per CTR-030, `tax_treatment_ref` MUST be a pack-lookup reference, not an inline numeric rate. The check can detect numeric literals where a ref is required.

**Why it happens:** developer convenience. "Why look up VAT rate every time when I know it's 18%?" Because the rate is jurisdiction-specific, change-prone, and pack-frozen per D-009. The lookup is non-negotiable.

### 10.5 AP5 — Vertical Modifying Universal Event Schema

A vertical adds fields directly to a universal engine's event payload, breaking the universal subscription contract.

**Mechanizable?** Yes — CTR-046 row 3. Doctrine check: event schemas are namespaced + versioned; a vertical cannot register an event under a universal namespace.

**Right pattern (BD5 split-it)**: vertical emits its own event in its own namespace; universal subscribes; vertical's extra context lives in vertical-namespaced payload fields the universal engine reads via documented contract.

### 10.6 AP6 — Inventing a New Primitive Instead of Composing Existing Ones

A vertical (or even a universal engine) invents a state-machine, persistence pattern, or audit log specific to its needs rather than using Foundation primitives.

**Mechanizable?** Partially — CTR-046 row 4. Doctrine check: VI-02 enforcement (every stateful process declares a Workflow instance). Bespoke state machines without Workflow reference fail at registration.

**Why it happens:** "the Workflow primitive doesn't quite fit my use case." Almost always wrong; the primitive accommodates surprisingly broad cases. If it genuinely does not fit, the gap is a Foundation amendment (CTR to Term 4), not a vertical-side workaround.

### 10.7 AP7 — Platform-Scope Creep from a Vertical

A vertical declares operations at platform scope (cross-tenant) to enable a feature like "regional benchmarking" or "industry comparison."

**Mechanizable?** Yes — CTR-046 row 5. Doctrine check: for engines with `engine_kind: vertical`, `scope_policy` and all per-operation `scope_ref` overrides MUST be one of {`site`, `tenant`} — never `platform`.

**Right pattern:** platform-scope aggregation is owned by Term 1 (CTR-016) or Reporting (Term 5) projections; verticals never operate cross-tenant.

---

## 11. Worked Examples — Five Hard Cases

### 11.1 Pharmacy — Brief §11.7 closure + R1 hybrid + R4 grounding

**Question**: A pharmacy dispenses prescription drugs (regulated) and sells OTC drugs (retail-like). Should pharmacy be a vertical?

**Doctrine applied:**

- BD6 3-of-3 test for prescription side: ✓ namespace `pharmacy.*`, ✓ Workflow `pharmacy.prescription`, ✓ regulatory events (TFDA reporting, controlled-substance ledger). **Pass.**
- BD6 3-of-3 test for OTC side: 1-of-3 (namespace shareable; Workflow identical to retail.sale; no regulatory event vocabulary). **Fail.**
- BD4 (push down): can OTC be pushed to a primitive? No — OTC sale is universal (Checkout consumes retail/restaurant/hotel/workshop bill.ready). Already at the right layer (Universal Checkout consuming retail emission).
- BD5 (split-it): the universal mechanism is retail-sale + Universal Checkout; the vertical context for prescription is `pharmacy.*` namespace + `pharmacy.prescription` Workflow + regulatory events.

**Closure:** Pharmacy IS a vertical for prescription operations. OTC operations remain retail. Implementation = Mixed-Vertical Tenant per CN-6-105 (activate both `retail.*` and `pharmacy.*`). Brief §11.7 is closed.

### 11.1.1 Retail spectrum — duka, supermarket, mixed-items

Worth making explicit: `retail.*` vertical covers the **full basket-POS spectrum** — kiosk, duka, mini-mart, supermarket, hypermarket, department store. The Workflow (`retail.sale`) is identical across all; differences are pack hooks (`pack.retail.multi_till`, `pack.retail.barcode_required`, `pack.retail.aisle_layout`, `pack.retail.self_service_enabled`) and operational scale. Scale alone does not create a new vertical (BD3 + BD8). Mama Amina's Kariakoo duka and a Nakumatt supermarket in Mlimani City share the same vertical engine — different density, same business doctrine.

The corollary: retail handles **any item type** sold via basket POS — rice, sugar, soap, OTC drugs, cooking oil, sundries, clothing, electronics, hardware, stationery. The item primitive (CN-4-011) carries category metadata; pack rules per item category drive tax treatment (CN-5-105), inventory tracking (CN-5-003), and pricing (per CN-5-007 promotion mechanics). The retail vertical does not specialise by item type — it specialises by **selling mechanism** (basket POS). This is why Mama Amina's expansion (§11.6) adds `pharmacy.*` alongside, not within: prescription handling is a different mechanism, not a different item type.

### 11.2 Salon + light services cluster — Brief §11.9 framing

**Question**: Salon, car wash, boda boda mechanic, cobbler, fundi simu — each is a concrete Tanzanian SME pattern. Are they verticals?

**Doctrine applied:**

- BD3 test: do they share unique semantics? Marginal — they share appointment-or-walk-in + service delivery + optional parts. The semantics are *similar to* Workshop project but lack parametric geometry and material cut lists.
- BD6 3-of-3 test: each currently fails (namespace shareable; Workflow distinct-from-Workshop but not uniquely-per-trade; minimal regulatory event vocabulary).
- BD8 ≥2 threshold: easily met. Salon + barber + nail + car wash + boda mechanic + cobbler + fundi simu = 7+ concrete patterns. Not hypothetical.

**Closure:** Cluster is concrete and concept-worthy. Three candidate paths (subsume to Workshop / generic `services.*` vertical / split into specific verticals) require market data to choose. **Decision elevated to high-priority post-CN-6-104; CN-6-905 stress-test sketch chooses path with Concept Lead market-priority ruling.** Brief §11.9 is framed; placement deferred.

### 11.3 Insurance Claim Ledger — does it split from Accounting?

**Question** (hypothetical future vertical): An insurance brokerage needs to track claims paid, premiums received, reserves held. Does Insurance need its own ledger, or does universal Accounting cover it?

**Doctrine applied:**

- BD1 test: is "ledger" foundation? Yes — the ledger primitive is foundation. No vertical reinvents the ledger.
- BD2 test: is "accounting / journal mapping" universal? Yes — CN-5-001 Accounting is the universal mechanism for all financial recording.
- BD5 split-it: the *mechanism* (ledger primitive + Accounting journal mapping) stays universal. The *context* (insurance-specific account categories like "claim reserves," "unearned premium liability," "reinsurance ceded") lives in pack hooks per `pack.insurance.chart_of_accounts.*`.

**Closure:** Insurance does NOT split from Accounting. It uses universal Accounting + insurance pack hooks. The vertical's events carry `chart_of_accounts_hint: 'claim_reserve_increase'`; Accounting maps via pack. **No new universal "Insurance Accounting Engine" — that would be premature universalisation (AP1) and BD8 violation. Brief §3.3 Insurance feasibility confirmed under existing universal layer.**

### 11.4 Hotel + Restaurant in-stay charge — BD7 closure (R5)

**Scenario**: Lodge Serengeti — a hotel with a restaurant inside. A guest eats dinner at the restaurant and asks "charge to my room." This is the canonical cross-vertical case (Brief §7.5, §7.6).

**Wrong approach (BD7 violation):** Create a "Hospitality Bridge Engine" that knows both hotel folios and restaurant bills. It would duplicate Accounting (journaling the charge) + Cash (deferring settlement) + Document (folio statement), couple Hotel and Restaurant, and violate VE2 isolation. Decommissioning Hotel would break Restaurant.

**Right approach (BD5 + BD7 via Obligation primitive):**

1. Guest finishes dinner. Restaurant completes order Workflow → emits `restaurant.bill.ready.v1` carrying `saleable_lines[]`, `payer_party_ref: <guest>`, `payment_method_hint: charge_to_room`.
2. Universal Checkout consumes. The `charge_to_room` hint routes to Obligation primitive instead of tender — Checkout emits `obligation.created.v1 {kind: hospitality_charge, debtor_party_ref: <guest>, creditor_engine: restaurant, amount, hotel_folio_workflow_ref: <folio>}`.
3. Hotel folio Workflow subscribes to `obligation.created.v1` filtered by `kind: hospitality_charge` and matching `hotel_folio_workflow_ref`. Folio accrues the charge as a folio line.
4. On guest checkout (hotel checkout, not restaurant), Hotel folio Workflow emits `hotel.folio.ready.v1` aggregating ALL folio lines (room nights, restaurant charges, mini-bar, laundry). Universal Checkout settles via tender.
5. On settlement, Hotel folio Workflow resolves each underlying `hospitality_charge` Obligation. Restaurant's obligation resolves → restaurant revenue recognised.
6. Accounting auto-journals correctly throughout: restaurant revenue (when restaurant obligation resolves), hotel cash receipt (on tender), and the obligation lifecycle is audit-traceable end-to-end.

**What this proves:**

- Restaurant and Hotel communicate solely via Foundation Obligation primitive + standard `checkout.settled.v1` + standard Accounting subscriptions.
- **No bridge engine needed.** BD7 closed concretely.
- Restaurant and Hotel stay isolated (VE2). Decommissioning Hotel tomorrow does not break Restaurant — Restaurant can fall back to direct-tender for that guest.
- Mama Halima-style scaling: this same pattern handles in-stay laundry, mini-bar, spa — each as obligation-bearing micro-vertical or universal-housekeeping emission, all settled at hotel checkout.

### 11.5 Workshop Cut List — BD3 vertical uniqueness

**Question**: Workshop has parametric geometry, cut lists, offcuts, glass guillotine optimisation. Are these vertical concerns?

**Doctrine applied:**

- BD3 test: do "cut lists" make sense for retail (no), restaurant (no), hotel (no)? Confirmed: only workshops fabricate from parametric geometry. The concept is workshop-unique.
- BD2 universal test: does retail/restaurant/hotel/workshop all use cut lists with same semantics? No — only workshop. **Fail.**
- BD4 push-down test: can cut lists become a foundation primitive? No — they encode workshop-specific algorithms (linear bar optimisation, 2D guillotine). Foundation primitives are domain-neutral.
- BD5 split-it: could "cut lists" be universal "inventory consumption planning"? The vocabulary is too workshop-specific. The *mechanism* — pre-emission of consumption with offcut tracking — is in CN-5-003 Inventory (Pattern B vertical-managed consumption). The *vocabulary* — bar lengths, glass sheets, guillotine cuts — is workshop. Split-it is already correctly drawn.

**Closure:** Cut list logic belongs in Workshop vertical (`workshop.cut.executed.v1`, `workshop.offcut.recorded.v1`). Inventory subscribes to consumption events via Pattern B. The parametric formula engine, geometry computation, and guillotine optimiser are workshop-internal. **BD3 cleanly satisfied; no universal extension needed.**

### 11.6 Mama Amina expansion narrative (R4 grounding)

Mama Amina runs Duka la Mama Amina in Kariakoo. Her shelves carry rice, sugar, cooking oil, soap, basic medicines (paracetamol, panadol), and sundries. Through BOS via her regional agent, her tenant is `retail.*` activated.

Six months in, her cashflow is healthy. She decides to expand: hire a TFDA-licensed pharmacist (her cousin Faraja just graduated) and install a small prescription counter at the back. She wants to dispense prescription drugs alongside the regular duka.

**The expansion question reaches her agent. Her agent applies CN-6-101.**

**Step 1 — BD3 (vertical = unique business type)**: Are pharmacy operations distinct from retail? *Prescription side: yes — pharmacist consultation, prescription validation, TFDA-regulated dispensing, controlled-substance ledger. OTC side (paracetamol on her existing shelves): no — these are retail SKUs sold like any other.*

**Step 2 — BD6 3-of-3**: Does prescription side meet the 3-of-3 test? *✓ namespace `pharmacy.*` (CTR-038 pre-allocated), ✓ Workflow `pharmacy.prescription` distinct from `retail.sale`, ✓ regulatory event types unique. **Pass.*** Does OTC side meet 3-of-3? *Fail — only namespace, no workflow distinction, no regulatory vocabulary.*

**Step 3 — BD4 (push down) + BD5 (split-it)**: Don't invent a "Mama Amina hybrid duka+pharmacy" engine — that would be a bridge engine (BD7 violation). Instead:

- Mama Amina's tenant activates BOTH `retail.*` (existing, no change) AND `pharmacy.*` (new) per CN-6-105 Mixed-Vertical Tenants pattern
- OTC paracetamol stays on her duka shelves, sold via `retail.sale.completed.v1` → Universal Checkout → Accounting (revenue: general retail)
- Prescription drugs go through `pharmacy.prescription` Workflow at Faraja's counter → `pharmacy.prescription.dispensed.v1` → `pharmacy.bill.ready.v1` → Universal Checkout → Accounting (revenue: prescription dispensing, mapped per `pack.pharmacy.chart_of_accounts.*`)
- One tenant, one set of books, one Universal Checkout flow used by both, regulatory audit trail per pharmacy regulation

**What Mama Amina experiences day-to-day:**

- Her cashier (her daughter Salma) operates one POS screen. When a customer brings paracetamol from the shelf, it scans as a retail SKU; Salma settles like any retail sale. When a customer brings a prescription, Salma routes them to Faraja's counter; Faraja's screen shows the pharmacy prescription workflow.
- Accounting reports show "Retail Revenue" and "Prescription Revenue" separately (per pack chart of accounts) — Mama Amina sees which side is growing.
- Tax treatment is correct per item category: standard VAT on retail SKUs; per-pack pharmacy rules on prescription (some prescription items may be VAT-exempt in TFDA classifications).
- Regulatory: Faraja's TFDA license is captured at pharmacy activation per CTR-045 (`regulatory_evidence_refs`); controlled-substance ledger flows through `pharmacy.controlled_substance.dispensed.v1` events with audit-trail-by-design.
- Period close (per CN-5-104): closes for retail and pharmacy operations in the same period, against the same tenant tax profile (Mama Amina is VAT-registered post-expansion per `tenant_tax_profile` per CN-5-105 N7).

**Charter §1.1 honoured**: Mama Amina expanded without losing her duka identity. She trusts her numbers. She can answer "how much did pharmacy add this month?" with proof from her phone.

This is what the boundary doctrine delivers in real life. Without BD1-BD8 + Mixed-Vertical pattern, the expansion would have meant either (a) re-platforming her tenant ("now you are a pharmacy, not a duka"), (b) ad-hoc retail+pharmacy hybrid code (the gap Brief §12 warns of), or (c) two separate tenants ("Mama Amina Duka" and "Mama Amina Pharmacy") with two sets of books. None are right. Mixed-Vertical Tenant is the right answer; CN-6-101 is why we know that.

---

## 12. VI-NN Ratifications from CN-6-100 §9.4

CN-6-100 §9.4 listed three candidates for the vertical-side invariants catalog: VI-03, VI-04, VI-05. With CN-6-101's boundary doctrine in place, two ratify now; one defers pending upstream mjadala.

### 12.1 VI-03 — Conflict event family (ratified)

**Statement:** Every vertical with multi-actor concurrent operations on shared resources (Restaurant tables, Hotel rooms, Workshop equipment, Pharmacy controlled-substance cabinet) emits a `<vertical>.<noun>.conflict.detected.v1` event family when the command bus single-acceptance mechanism (CN-4-004) rejects a contender.

**Boundary-doctrine basis:** BD3 confirms conflict semantics are vertical-specific (the *what* — which resources, what kinds of conflict). BD4 confirms the *mechanism* (bus single-acceptance) stays Foundation. BD5 split-it: universal mechanism + vertical context.

**Ratification rationale:** the pattern appears across at least Restaurant (waiter contention), Hotel (overbooking), Workshop (equipment contention). Three current verticals demonstrate need; BD8 ≥2 threshold met.

### 12.2 VI-04 — Regulated activation gate (ratified)

**Statement:** Every regulated vertical (Pharmacy, Clinic, future Insurance, future Education) declares pre-activation governance gates per CTR-045, including `regulatory_review_required: true` and a `regulatory_evidence_refs[]` requirement at activation time.

**Boundary-doctrine basis:** BD6 (regulated specificity doesn't automatically make a vertical, BUT a regulated vertical has activation specifics that non-regulated does not). The gate is a vertical-side commitment; the *mechanism* (registration per CN-4-020 + Term 1 governance) is upstream.

**Ratification rationale:** Pharmacy (§11.1) demonstrates concretely. Clinic and future regulated verticals share the pattern. CTR-045 is in flight; VI-04 commits vertical-side discipline regardless of when CTR-045 closes.

### 12.3 VI-05 — Data sensitivity tier (deferred)

**Status:** Deferred from CN-6-101 v1. Reason: VI-05 candidate (`data_sensitivity_tier` in vertical manifest) implicates the Identity primitive (CN-4-007) and Document primitive (CN-4-012) at the Foundation layer — specifically, whether per-vertical encryption-at-rest requirements, retention rules, and access-audit gates can plug into existing primitives or require new primitive-level extension.

**CTR-047 (new, opened with this doc)** to Term 4: `data_sensitivity_tier` primitive mjadala. Pending Term 4 ruling on whether the Identity + Document primitives support tiered sensitivity declarations or require extension. VI-05 returns in a future CN-6-101 amendment (or moves to CN-6-006 Clinic if Clinic concept doc lands first) once Term 4 responds.

---

## 13. Boundaries

CN-6-101's responsibilities versus its neighbours:

| Concern | Owned by | CN-6-101 role |
|---------|----------|----------------|
| Decision tree for layer placement | **CN-6-101** (this doc) | Authoritative |
| The recipe to add a new vertical | CN-6-100 | Parent — CN-6-101 elaborates the placement question CN-6-100's recipe presupposes |
| Vertical Event Naming Conventions (syntax) | CN-6-102 | Pointer — CN-6-101's BD3/BD5 inform what events exist; CN-6-102 names them |
| Vertical Scope Policy (site/tenant detail) | CN-6-103 | Pointer — CN-6-101's BD3 + AP7 set scope rules; CN-6-103 elaborates |
| Vertical-to-Universal Hand-Off Pattern | CN-6-104 | Pointer — CN-6-101's BD5 split-it underpins; CN-6-104 details the handshake |
| Concrete verticals (Retail, Restaurant, Hotel, Workshop) | CN-6-001..004 | Conform to BD1-BD8 |
| Vertical Bridges | CN-6-005 | BD7 closes "no bridge engines"; CN-6-005 documents the patterns (Obligation-mediated, event-mediated) |
| Mixed-Vertical Tenants | CN-6-105 | §9.1 + §11.1 + §11.6 establish the Pharmacy + Retail Mixed-Vertical case; CN-6-105 generalises |
| Future-vertical stress-test sketches | CN-6-901..904 (+ CN-6-905 salon cluster) | Each runs BD1-BD8 against the candidate vertical |
| Charter §4 five-layer architecture | Charter (Term 7 maintains) | CN-6-101 honours, does not amend |
| Foundation primitive catalog | CN-4-011 + Term 4 docs | CN-6-101 BD1 derives placement criteria from existing catalog |
| Universal Engine scope | CN-5-100..105 + Term 5 docs | CN-6-101 BD2 derives placement criteria from existing universal scope |
| Engine catalog / activation governance | Term 1 (pending) | CN-6-101 references CTR-045 for regulated activation (Pharmacy §11.1) |
| Tenant UX for vertical workflows | Term 3 (pending) | CN-6-101 BD3 confirms vertical owns workflow specifics; Term 3 owns UX |
| Coherence verification across Terms | Term 7 (Overseer) | CN-6-101 surfaces placement disagreements for Term 7 mediation |

---

## 14. Open Items + Cross-Term Hooks

### 14.1 CTRs ratified Term 6 side (with this doc)

- All CTRs ratified by CN-6-100 §14.1 continue to apply; CN-6-101 introduces no new contracts to those.
- Brief §11.2 (vertical bridges as first-class) — closed via BD7. No bridge engines; bridges are patterns documented in CN-6-005.
- Brief §11.7 (Pharmacy = vertical or retail-with-attributes) — closed via BD6 + §11.1 + §11.6. Pharmacy IS a vertical for prescription operations; Mixed-Vertical for OTC.

### 14.2 CTRs opened with this doc

- **CTR-046** → Term 4 (Foundation): mechanizable anti-patterns as doctrine-check (DC) additions to CN-4-019 living catalog. Five mechanizable rows from §10:
  - AP2: vertical-to-vertical direct subscription
  - AP4 (partial): tax_treatment_ref must be a pack-lookup ref, not inline numeric
  - AP5: vertical cannot register events under a universal namespace
  - AP6 (partial): VI-02 enforcement — bespoke state machines without Workflow reference fail at registration
  - AP7: vertical engines cannot operate at platform scope
- **CTR-047** → Term 4 (Foundation): `data_sensitivity_tier` primitive mjadala. Whether Identity (CN-4-007) + Document (CN-4-012) primitives support tiered sensitivity declarations or require extension. Blocks VI-05 ratification.

### 14.3 CTRs pending upstream

- **CTR-036, CTR-037, CTR-041, CTR-044, CTR-045** — same status as CN-6-100 §14.3. CN-6-101 references but does not advance.

### 14.4 CTRs pending downstream

- **CTR-004, CTR-005, CTR-010** — same status as CN-6-100 §14.4.

### 14.5 Term 6 internal open items

- **Salon + light services cluster placement** (Brief §11.9 + §9.2). Decision elevated to high-priority. **Next-ordering ruling pending Concept Lead**: does CN-6-905 stress-test sketch run immediately after CN-6-104 (the cross-cutting framework batch), or after CN-6-001..004 (the existing-vertical concept docs)? Two reasonable orderings; market-priority determines.
- **Existing-vertical retrofit guidance**. CN-6-101 confirms retrofit is out of Concept Phase scope (§1.5). CN-6-001..004 will document AS-IS for reference + SHOULD-BE per BD1-BD8; Architect phase reconciles. **Confirmed.**
- **VI-05 data sensitivity tier**. Returns in a future amendment once CTR-047 closes Term 4 side.

### 14.6 D-DISC cross-references (R7)

Per `DEFERRED-DISCUSSIONS.md`:

- **D-DISC-001 — Tenant-customer promotion UX**. The customer-facing surface of promotions touches a vertical-vs-Term-3 boundary not yet ratified. CN-6-101 BD3 implies vertical owns vertical-specific customer flows (e.g., the prescription-pickup confirmation UX in a pharmacy differs from the kinyozi-appointment-reminder UX); Term 3 owns generic customer patterns. Resolution requires CTR to Term 3 when Term 3 activates.
- **D-DISC-002 — POS self-service expansion**. Customer-operated POS (kiosk in café, pharmacy self-pickup, retail self-checkout). CN-6-101 BD3 confirms vertical owns the self-service workflow specifics; the underlying mechanism (Universal Checkout, identity per CN-4-007, Workflow primitive) stays universal/foundation. Concrete activation deferred per `DEFERRED-DISCUSSIONS.md`.

### 14.7 The bar — restated

> *"If we get CN-6-100 right, the future verticals will be easier than the existing ones were."* (Brief §12)

CN-6-100 set up the recipe. CN-6-101 sets up the placement decision that determines *whether* the recipe even applies — by clarifying when something is a vertical at all versus when it belongs lower. Together they form the framework. CN-6-001..004 will conform; CN-6-901..904 + CN-6-905 will stress-test.

Mama Amina's pharmacy expansion (§11.6) is the proof. If the doctrine works for her — and CN-6-101 says it does — it will work for the next ten thousand SMEs that try to grow without losing themselves.

---

*— End of CN-6-101 Vertical Boundary Doctrine v1 —*
