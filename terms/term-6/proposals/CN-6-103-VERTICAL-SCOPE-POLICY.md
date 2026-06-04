# CN-6-103 — Vertical Scope Policy

> **Term:** 6 — Vertical Engines
> **Status:** Concept Phase v1.0 — Draft for Overseer review
> **Branch:** `claude/brave-johnson-S7Lky`
> **Reading order:** CN-5-101 (scope doctrine) → CN-5-102 UI-09 (site_id registry) → CN-6-100 §8 (scope discipline summary + multi-site-by-nature) → CN-6-101 BD3/BD8/AP1 → CN-6-102 NC-conformance → **this doc**
> **Ordering authority:** TERM-6 Brief §13 — fourth Term 6 deliverable; operational scope reference for CN-6-001..004 vertical authors.

---

## 1. Purpose & Boundary

### 1.1 Mission

CN-6-103 applies CN-5-101's universal scope doctrine (site / tenant / platform) to vertical engines specifically. It answers, for every vertical operation:

> *"At what scope does this event live? Site (one duka, one kitchen, one room)? Tenant (across all Mama Amina's locations)? Or — never for verticals — platform (across all tenants)?"*

CN-5-101 is the parent. CN-6-103 is the **operational application** with vertical-side discipline: what's the default, when do you opt out, what the multi-site-by-nature pattern means concretely, what platform-scope is prohibited, and how scope changes are migrated forward-only per Law 1.

### 1.2 DOES vs DOES NOT

| CN-6-103 DOES | CN-6-103 DOES NOT |
|---------------|-------------------|
| Apply CN-5-101 site/tenant/platform doctrine to verticals | Amend or replace CN-5-101 — gap is a CTR to Term 5 |
| Set vertical default = site (SP1) and platform = never (SP2/SP8) | Define the platform scope mechanism (CN-4-006 — Foundation) |
| Catalog tenant-scope exceptions (guest profile, style library, catalog, regulatory ledger) | Author the vertical's specific event vocabulary (CN-6-001..004) |
| Sharpen multi-site-by-nature criteria (SP4 3-of-3 test) | Define the site registry mechanism (CTR-027 — Term 1) |
| Specify per-operation `scope_ref` override application via CTR-023 | Define the registration manifest schema (CN-4-005 + CN-6-100 §4) |
| Document site deactivation lifecycle doctrine + scope migration discipline | Author site deactivation governance content (Term 1 fills) |
| Reaffirm SP8 mechanizability via CTR-046 next amendment cycle | Open new CTRs — zero-new-CTR goal honoured |
| Ground patterns in real Tanzanian businesses (Mzee Hassan, Faraja, Lodge Serengeti, Mama Halima) | Specify scope-aware UI surfaces (Term 3) |

### 1.3 Audience

Term 6 authors writing CN-6-001..004; Architects implementing vertical engines; Term 7 reviewing scope-relevant PRs; Term 1 implementing tenant onboarding + site registry + activation governance; future vertical contributors who must decide scope for new event families.

### 1.4 Charter Compliance

CN-6-103 honours Charter §4 layer separation and the Six Laws:

| Law | How CN-6-103 honours it |
|-----|--------------------------|
| Law 1 — State from events only | SP7/SP8: scope migration is forward-only manifest amendment; historical events immutable; read projections span boundaries |
| Law 2 — Engines isolated | SP2/SP8 prohibit platform-scope verticals; cross-tenant data goes through Reporting/Term 1 |
| Law 3 — AI advisory only | (Out of scope for CN-6-103 — handled by CN-5-010 + CN-4-022) |
| Law 4 — Flexibility first-class | Multi-site-by-nature pattern (SP4) absorbs Logistics, future Insurance, future cross-site Workshop without architectural change |
| Law 5 — Compliance configured | Pharmacy controlled-substance scope = pack-driven (`audit_scope_default`); not coded per jurisdiction |
| Law 6 — Distribution regional | Single-jurisdiction tenant via CTR-027 in v1; multi-jurisdiction deferred to v2 |

### 1.5 Parsimony — scope is where the number shows up

Scope policy is not abstract. Scope determines which dashboard each number appears on, for whom.

When Salma at Mama Amina's till in Kariakoo asks her phone "today's revenue," the answer comes from **site-scoped** retail events at that one duka. When Mama Amina herself asks "today's revenue across both my counters" (retail till + Faraja's pharmacy counter), she crosses two verticals at the same site — still site-scope reads, aggregated. When Mzee Hassan in Arusha asks "active projects across both my workshops" (Arusha + a future Moshi branch), he reads **tenant-scope** workflows. When the Lodge Serengeti GM looks at "tonight's occupancy" on the front-desk screen, that's site-scope hotel reservations for that property. When Faraja prints the **controlled-substance log** for TFDA monthly reporting, that's site-scope per TFDA premises registration.

Scope is what the question "for whom does this number exist?" answers. A site-scoped number exists for the people at that site. A tenant-scoped number exists for the owner across sites. A platform-scoped number — which no vertical ever produces — would exist for BOS itself.

**Parsimony is the bar.** Each SP1–SP8 below is justified against the question "does this match how the real person at the till, the front desk, the controlled-substance cabinet, the fundi station thinks?"

---

## 2. Inputs and Relationship

### 2.1 Parent doctrine

- **CN-5-101** authoritative for site / tenant / platform scope semantics, site_id as payload concept, scope-level dispatch
- **CN-5-102 UI-09** site_id-in-registry enforcement at universal subscription
- **CN-4-005 + CTR-023** per-operation `scope_ref` override mechanism (engine default + per-operation override)
- **CN-4-006** platform scope (rare, dual-audited, never for verticals)

### 2.2 Term 6 upstream

- **CN-6-100 §8** scope discipline summary + multi-site-by-nature manifest flag (`multi_site_capable: true`)
- **CN-6-100 VE5** vertical payload conformance per CTR-024
- **CN-6-101 BD3** vertical = unique business type; BD8 = no premature up-promotion; AP1 = no premature universalization
- **CN-6-101 AP7** — vertical engines cannot operate at platform scope (mechanizable per CTR-046)
- **CN-6-102 NC1-NC9** event naming inherits scope semantics; multi-site-by-nature events carry `origin_site_id` + `destination_site_id`

### 2.3 Cross-Term contracts cited

- **CTR-024** (site_id payload — Term 6 ratified Term 6 side)
- **CTR-026** (UI-03 source-ref + UI-09 registry — Term 6 ratified Term 6 side)
- **CTR-027** (site registry + currency registry — Term 1 governs; Term 2 populates at onboarding)
- **CTR-046** (mechanizable anti-patterns; expansion `2361014` includes AP7 / SP2 emission-side check; SP8 subscription-side check queued for next amendment)

### 2.4 Sibling boundary

CN-6-104 (Vertical-to-Universal Hand-Off Pattern) builds on CN-6-103's scope discipline — the hand-off mechanics consume scope-correctly-emitted events. CN-6-001..004 author per-vertical events at the scopes CN-6-103 prescribes.

---

## 3. Doctrine — SP1–SP8

**SP1 — Vertical default scope is site.** Real business operations happen at physical locations (duka, kitchen, room, fundi station, cabinet, depot). Site is the natural granularity. A vertical's `scope_policy` in manifest is `site` by default. Per-operation overrides require explicit justification per SP3.

**SP2 — Verticals never declare platform scope.** Platform scope (CN-4-006 §2) is reserved for Foundation-level cross-tenant aggregation owned by Term 1 (billing roll-up, regulatory pack governance). A vertical engine declaring `scope_policy: platform` or per-operation `scope_ref: platform` is rejected at the CN-4-020 doctrine gate. Mechanized via CTR-046 (per `2361014`).

**SP3 — Tenant-scope is explicit opt-in per operation.** Not default. Per-operation `scope_ref: tenant` override (CTR-023) requires the operation to genuinely span sites in the vertical's business semantics: chain-wide guest profile, tenant-wide style library, tenant-wide pricing catalog, tenant-wide regulatory ledger (pack-driven). §6 catalogs the justified cases.

**SP4 — Multi-site-by-nature requires the 3-of-3 criteria test.** A vertical declares `multi_site_capable: true` in manifest if and only if **all three** of the following hold for the operation in question:

1. **Primary entity has its own identity that traverses sites** — a trip, a long-running multi-site project, a multi-leg booking-bundle (not "two single-site events related by a foreign key")
2. **Operation needs BOTH origin and destination as authoritative payload fields** at event-emission time — not just at read-time aggregation
3. **Downstream universal engines need site disambiguation in payload** (e.g., Inventory needs origin for deduct, destination for add) — not in subscription pattern matching

The criteria are a **gate, not a heuristic**: any one failing → fall back to SP1 site-scope + read-side projection for cross-site aggregation. §7 walks fail cases.

**SP5 — site_id payload is mandatory on site-scope events.** Per CTR-024. Non-conformant payloads hard-fail at universal subscription dispatch per CN-5-101 §5. Tenant-wide events (rare for verticals — §6 catalog) omit `site_id`. Multi-site-by-nature events carry typed site fields (`origin_site_id`, `destination_site_id`).

**SP6 — site_id must resolve in the tenant's site registry.** Per CTR-027 + UI-09 (CN-5-102). Verticals never invent site_id values; they receive them from command inputs or workflow context. The Foundation event store rejects events with unknown `site_id` at acceptance time.

**SP7 — Scope migration is forward-only manifest amendment.** A vertical changing scope policy (e.g., site → tenant for a specific event family) does so via versioned manifest amendment (per CN-4-020 manifest-update flow). Historical events stay at their original scope per Law 1 (immutability). Read-side projections (CN-4-010) span the boundary. Runtime scope mutation of a single event's `scope_ref` is prohibited.

**SP8 — Verticals never subscribe at platform scope.** Parallel to SP2. A vertical's `subscribes_to[*].scope_ref` may never be `platform`. Cross-tenant data needs are routed through Universal Reporting (CN-5-006) tenant-scope projections or Term 1 platform-aggregator subscriptions per CN-5-101 §7. Mechanizable check queued for CTR-046 next amendment cycle (paired with SP2 emission-side check) — see §5.4.

---

## 4. Site-Default Discipline (SP1)

### 4.1 Why site is the right default

The Salma test (§1.5): a cashier at one duka asks "what's our revenue today?" The natural answer is the events emitted at that duka, not aggregated across Mama Amina's empire. The natural unit of operation is the site. SP1 codifies this: every vertical operation defaults site-scope unless the business semantics genuinely require otherwise.

### 4.2 Manifest declaration

A vertical's manifest declares site default at the top level:

```
engine_kind: vertical
scope_policy: site
```

Per-operation overrides (SP3) sit inside `commands[]` and `emits[]` entries:

```
emits:
  - event_type: workshop.cut.executed.v1
    # inherits scope_policy: site (no override)
  - event_type: workshop.style_library.entry.added.v1
    scope_ref: tenant                                # SP3 explicit opt-in
```

### 4.3 Site-scope event payload contract

Per CTR-024 + SP5, every site-scope event payload carries `site_id` resolving in the tenant's site registry (CTR-027 + UI-09). Examples:

- `retail.sale.completed.v1 {sale_id, site_id, saleable_lines, ...}`
- `restaurant.kitchen_ticket.fired.v1 {ticket_id, site_id, table_ref, items, ...}`
- `hotel.reservation.checked_in.v1 {reservation_id, site_id, room_ref, party_ref, ...}`
- `workshop.cut.executed.v1 {cut_id, site_id, bar_ref, source_ref: <project_id>, ...}`
- `pharmacy.controlled_substance.dispensed.v1 {dispensing_id, site_id, party_ref, controlled_category, ...}`

### 4.4 What "site" means per vertical

The site_id semantically resolves to:

| Vertical | Site_id semantic |
|----------|------------------|
| retail | One physical POS location (duka, kiosk, supermarket store) |
| restaurant | One physical venue (restaurant, café, bar — within the venue, multiple tables share one site_id) |
| hotel | One physical property (a single lodge, a single guesthouse) |
| workshop | One physical fundi location (a karakana, a fabrication branch) |
| pharmacy | One physical pharmacy premises (per TFDA registration in TZ) |
| clinic | One physical clinic premises |
| logistics | One physical depot, warehouse, or driver-base (operations that traverse sites use multi-site-by-nature per SP4) |

These are all "one physical location" — the natural unit Salma, Mzee Hassan, Faraja, and the Lodge Serengeti GM each work within.

---

## 5. Platform-Scope Prohibition (SP2 + SP8 + BD8 + AP1)

### 5.1 Why platform is never for verticals

Platform scope (CN-4-006 §2) is BOS-itself's scope — visible across all tenants, dual-audited, governed by Term 1. A vertical operating at platform scope would: (a) couple business logic to cross-tenant state (BD3 violation — vertical = unique-to-one-tenant-type), (b) bypass tenant isolation (Law 2 violation), and (c) invite premature universalization (AP1) by hiding cross-tenant aggregation inside vertical code instead of Reporting projections.

### 5.2 What verticals do instead

Cross-tenant data needs (rare for verticals) route through:

- **Universal Reporting (CN-5-006) tenant-scope projections** — Reporting publishes per-tenant KPIs; platform-aggregator (Term 1) rolls up cross-tenant for billing/governance per CTR-016
- **Term 1 platform-aggregator subscriptions** — Term 1 reads `accounting.period.closed.v1` cross-tenant at platform scope for billing roll-up (CN-5-104 Q12 ruling); verticals never participate in this layer

A vertical that thinks it needs platform scope is almost certainly trying to build a benchmark/comparison feature — that's Reporting + Term 1, not vertical territory.

### 5.3 Mechanization status

**SP2 emission-side** (verticals cannot emit at platform scope) is in CTR-046 expansion (`2361014`) as AP7 mechanizable check. Doctrine gate at CN-4-020 registration rejects manifests with `scope_policy: platform` or any `emits[*].scope_ref: platform` on engines with `engine_kind: vertical`.

**SP8 subscription-side** (verticals cannot subscribe at platform scope) is **queued for CTR-046 next amendment cycle** as DC-NN-e (parallel check on `subscribes_to[*].scope_ref`):

```
DC-NN-e proposed spec:
  For any engine with engine_kind: vertical,
  no subscribes_to[*].scope_ref: platform allowed.
  Doctrine gate rejects at CN-4-020 registration.
```

Paired with the emission-side check, this closes both directions of SP2/SP8 enforcement. **No new CTR — queues as expansion item on existing CTR-046.**

### 5.4 BD8 + AP1 connection

The platform-scope temptation is the textbook BD8 anti-pattern: "in the future Insurance will need cross-tenant policy-issuance benchmarking" leading someone to declare `insurance.policy.issued.v1` at platform scope. SP2/SP8 prevent this — even if Insurance someday needs benchmarking, that work lives in Reporting/Term 1, not in the vertical engine.

---

## 6. Tenant-Scope Exception Catalog (SP3)

The justified tenant-scope exceptions for verticals. Each requires per-operation `scope_ref: tenant` override (CTR-023) in the manifest.

| Exception | Pattern | Justification |
|-----------|---------|---------------|
| **Hotel guest profile** | `hotel.guest_profile.updated.v1` at tenant scope | Chain-wide guest history (Lodge Serengeti + Kilimanjaro Lodge Moshi same tenant) — stay preferences, loyalty, complaint history aggregated across sites. **Folios remain site-scope** — each stay's folio is one site, one property. The *profile* is the tenant-scope aggregator. |
| **Workshop style library** | `workshop.style_library.entry.added.v1` at tenant scope | Mzee Hassan's parametric formulas, cut-optimization templates, and fundi training references are tenant-wide intellectual property. If he opens a Moshi branch, both sites share the library. Cuts and projects remain site-scope. |
| **Retail tenant-wide pricing catalog** | `retail.catalog.updated.v1` at tenant scope | Multi-site retailers apply pricing changes uniformly. Individual sales remain site-scope; the catalog itself is tenant-wide. |
| **Pharmacy controlled-substance audit** | Site-default + pack-driven tenant opt-in via `pack.pharmacy.controlled_substance.audit_scope_default` ∈ {site, tenant} | **TZ TFDA practice = per-premises ledger** (each pharmacy location has its own log under its TFDA premises registration). Default site. Pack rule allows other jurisdictions (some Kenya configurations where one pharmacist license covers multiple premises) to opt tenant-scope. Law 5 (compliance configured, not coded) honoured. |

**Customer loyalty balance — removed from catalog.** Loyalty is handled by Universal Promotion + Party primitive linking (per CN-6-102 §11.4 Mama Amina + Faraja walkthrough). Verticals don't emit loyalty events at tenant scope; that's a BD5 split-it instance already settled at the universal layer.

### 6.1 The justification test

Before any vertical author adds a tenant-scope exception, they must answer:

1. **Does the operation genuinely span sites in business semantics?** (Not just "could be aggregated for reporting" — that's read-side projection)
2. **Is the entity tenant-shared by nature?** (Style library = yes; sales = no; folios = no; guest profile = yes)
3. **Would a real user (Mzee Hassan, Mama Amina, Lodge Serengeti GM) say "this number lives at my business level, not at one site"?**

If all three are yes, tenant-scope is justified. Otherwise, default to site (SP1) with read-side projection for cross-site display.

---

## 7. Multi-Site-by-Nature Catalog + SP4 3-of-3 Discipline

### 7.1 SP4 the 3-of-3 criteria as sequential AND gate (N1)

The criteria from §3 are a **sequential AND gate**, not a heuristic. Any one failing → fall back to SP1 site-scope + read-side projection.

| Criterion | Pass condition | Fail example |
|-----------|----------------|--------------|
| (1) Primary entity has its own identity traversing sites | A `trip`, a `multi_site_project`, a `multi_leg_booking_bundle` — an entity that is one thing across two sites | A restaurant-in-hotel charge: restaurant bill + hotel folio are TWO entities at TWO sites linked by Obligation — fails (1); use SP1 + Obligation primitive |
| (2) Both origin and destination authoritative at emission time | Logistics trip emits `logistics.bill.ready.v1` carrying both `origin_site_id` and `destination_site_id` — both needed at journal time | Cross-site loyalty: site A emits sale; site B emits redemption — independent events at independent sites; fails (2); use SP1 + Universal Promotion |
| (3) Universal engines need site disambiguation in payload | Inventory deduct at origin + add at destination — Inventory subscription needs both fields | Sequential workshop work-order moving stages within a single site: payload has site_id once + workflow state; fails (3); use SP1 |

**The discipline:** when in doubt, fail-down to SP1. Multi-site-by-nature is a narrow pattern reserved for genuinely-site-traversing entities. Over-claiming it overloads universal subscription dispatch and breaks parsimony.

### 7.2 The catalog

| Pattern | Vertical | Payload convention | Status |
|---------|----------|---------------------|--------|
| **Logistics trip** | logistics (future) | `origin_site_id` + `destination_site_id` | ✅ Canonical — Mama Halima Dar → Mwanza per CN-6-100 §12 + CN-6-102 §12.5; passes 3-of-3 |
| **Inter-site Inventory transfer** | universal Inventory (CN-5-103) | `origin_site_id` + `destination_site_id` | ✅ Cited for completeness — universal pattern, not vertical; passes 3-of-3 |
| **Hotel chain reservation** | hotel | Multi-leg bookings = TWO site-scope reservations bundled via `booking_id` Workflow reference; single-site reservations = SP1 site-scope | ❌ NOT multi-site-by-nature — fails (1) the primary-entity test; bundled-by-booking_id is the right pattern |
| **Workshop multi-site project** | workshop (future) | TBD per CN-6-004 | ⏸ Defer — Mzee Hassan currently single-site Arusha; future Moshi fabrication branch would activate this case; CN-6-004 elaborates when it lands |

### 7.3 The Q2 closure — why hotel chain ≠ multi-site-by-nature

A guest books one room at Lodge Serengeti for two nights. That is one reservation, one site, one folio. SP1 site-scope.

A guest books a 5-night holiday with the same tenant: 3 nights at Kilimanjaro Lodge Moshi (Friday–Monday) then 2 nights at Lodge Serengeti (Monday–Wednesday). This is **two reservations** (one per site) **bundled by a shared `booking_id`** carried in both reservations' payloads. Each reservation Workflow runs independently; the booking_id provides read-side aggregation for "the guest's full trip."

The chain's *guest profile* (preferences, history across stays) is the tenant-scope SP3 exception per §6, separately from any individual reservation. Reservation Workflow stays site-scope; profile is tenant-scope; they reference each other via `party_ref`.

This is cleaner than declaring hotel reservations multi-site-by-nature, which would force Universal Checkout, Accounting, and Reporting to handle dual-site payloads on every hotel emission — over-claiming the pattern for a rare use case.

---

## 8. Per-Operation `scope_ref` Override (CTR-023 Application)

### 8.1 The mixed-scope manifest pattern

Per CTR-023 (closed; CN-4-005 amendment in `76dd67d`), a vertical engine manifest may mix scope per operation. The engine declares a default `scope_policy` and individual `commands[]` / `subscribes_to[]` / `emits[]` entries may carry `scope_ref` overrides.

### 8.2 Workshop mixed-scope manifest (WP1 preview)

Mzee Hassan's Workshop runs cuts at site-scope but maintains style library at tenant-scope. The manifest:

```
engine_id: workshop
engine_kind: vertical
namespace_root: workshop
scope_policy: site                                 # default per SP1

commands:
  - command_type: workshop.cut.dispatch.request
    # inherits scope_policy: site
  - command_type: workshop.style_library.entry.add.request
    scope_ref: tenant                              # SP3 explicit opt-in

emits:
  - event_type: workshop.cut.executed.v1
    # inherits scope_policy: site; payload carries site_id (SP5)
  - event_type: workshop.style_library.entry.added.v1
    scope_ref: tenant                              # SP3 explicit opt-in; no site_id in payload
  - event_type: workshop.bill.ready.v1
    # inherits scope_policy: site

subscribes_to:
  - event_type: inventory.stock.depleted.v1
    # inherits scope_policy: site — receive per-site stock signals
```

### 8.3 The justification line

Every per-operation `scope_ref: tenant` override carries an inline justification comment in the manifest (review-checklist enforcement at registration). Examples:

- `# style library = tenant IP shared across all fundi sites`
- `# guest profile = chain-wide history aggregation per SP3 exception`
- `# pricing catalog = retailer-wide price updates`

This prevents the "scope_ref: tenant by copy-paste" failure mode where developers add the override without understanding the BD3 + SP3 discipline.

---

## 9. Site Registry Resolution (CTR-027 + UI-09)

### 9.1 The two registries (CTR-027)

Per CTR-027, Term 1 governs and Term 2 populates at onboarding two tenant-property registries:

- **Site registry** — the canonical list of `site_id` values for a tenant. Set by the regional agent at onboarding (Law 6); governance for additions/changes by Term 1.
- **Functional currency registry** — the tenant's declared functional currency (one per tenant in v1; multi-currency tenants deferred).

CN-6-103 consumes the site registry; CN-6-102 NC3-NC6 emissions reference site_ids from it.

### 9.2 UI-09 enforcement

Per CN-5-102 UI-09, every `site_id` in a vertical event payload must resolve in the tenant's site registry at dispatch time. Universal subscriptions hard-fail on unresolvable site_ids. Verticals never invent site_id values — they receive them from:

- **Command inputs** (the cashier's POS knows its site; the front desk knows its property)
- **Workflow context** (a hotel reservation Workflow carries the site through its lifecycle)
- **Source events** (a derivative event inherits site from its `source_ref`)

### 9.3 What verticals MUST NOT do

- **Invent site_ids** ("temp-site-1") — registration rejects unknown values
- **Hardcode site_ids** in manifests or pack hooks — site_ids are tenant-property data, not engine config
- **Emit site_id-less events** on site-scope event types — SP5 + CTR-024 violation

### 9.4 Cross-tenant site_id collision

site_ids are tenant-scoped — two tenants may both have a `site_id: "main_branch"` without conflict. The Foundation event store namespaces by `(tenant_id, site_id)` per CN-4-006 isolation. Verticals never see other tenants' site_ids.

---

## 10. Scope Migration + Site Deactivation Lifecycle

### 10.1 SP7 forward-only scope migration

A vertical that needs to change scope policy for an event family (e.g., promoting an event from site-scope to tenant-scope because the business semantics changed) does so via versioned manifest amendment (CN-4-020 manifest-update flow):

1. Author proposes amended manifest with new scope_ref for the affected event family
2. Doctrine gate validates (SP1-SP8 compliance, no platform scope, justified per SP3)
3. Atomic activation at a specified `pack_version_ref` boundary
4. From the boundary forward, new events emit at the new scope
5. Historical events stay at their original scope per Law 1 immutability
6. Read-side projections (CN-4-010) span the boundary, presenting unified views to the dashboard

### 10.2 N4 concrete migration walkthrough — Mzee Hassan opens Moshi (hypothetical for Q4 demonstration; not a v1 commitment)

**Day 1 — single-site Arusha.** Mzee Hassan operates one workshop. His style definitions live in site-scope events: `workshop.style.registered.v1 {style_id, site_id: "arusha-karakana", ...}`. This works because all his fundis are in one place.

**Day 90 — Moshi fabrication branch opens.** Mzee Hassan opens a second site. Now style definitions need to be shared (a window design created in Arusha should be cut at Moshi the next day). He files a manifest amendment to migrate `workshop.style_library.entry.added.v1` from site-scope to tenant-scope, citing SP3 tenant-scope exception (style library = tenant IP).

**Day 91 — manifest amendment activates.** The amended manifest registers at `pack_version_ref: workshop-pack-2026.06`. From this moment forward, `workshop.style_library.entry.added.v1` emissions are tenant-scope (no site_id in payload). Existing Arusha-scoped style events (Day 1–90) remain site-scope per Law 1.

**Day 92 onward — read-side projection spans.** The projection serving "all my styles" reads:
- Pre-boundary: site-scoped events filtered by tenant (all sites, since Mzee Hassan only had Arusha pre-Day 91)
- Post-boundary: tenant-scoped events directly

The projection logic checks each event's `pack_version_ref` (or emission_ts compared to boundary) and applies the appropriate scope-read rule. The dashboard shows a unified style library; the user sees no boundary.

**Crucially:** old events are NOT re-emitted at the new scope. They stay at site-scope forever. Replay reconstructs each event under the scope it was emitted at. This is Law 1 in action.

### 10.3 Site deactivation doctrine

A site is deactivated (Lodge Serengeti closes, a duka shuts down, a pharmacy premises is decommissioned) via a controlled lifecycle, never by delete:

1. Term 1 governance approves deactivation request (per CTR-045 expansion when Term 1 activates)
2. A platform-level event marks the site as deactivated — see §10.4 N3 namespace note
3. Verticals subscribed to the deactivation signal transition in-flight workflows to a `<workflow>.site_deactivating.v1` blocked state (per CN-6-100 §3.13 pattern)
4. In-flight workflows complete naturally within `pack.platform.site_deactivation.workflow_continuation_window_days` (default 30)
5. New events at the deactivated site are rejected at acceptance
6. Outstanding obligations (folio charges, layby commitments, hospitality_charge obligations) settle or transfer per their own compensation declarations

### 10.4 N3 — site deactivation event namespace TBD

The platform-level event marking site deactivation has a namespace pending Term 1 activation. Candidate roots:

- `platform.site.deactivated.v1` (if a `platform.*` namespace is reserved for Term 1)
- `tenant.site.deactivated.v1` (if scoped to tenant-property events)
- A new top-level namespace per future CN-5-103 §16 amendment

CN-6-103 **documents only the vertical-side subscription contract** — verticals subscribe to whatever namespace Term 1 ratifies. The vertical-side handling (block in-flight workflows, complete natural lifecycle, reject new events) is independent of the upstream namespace choice. This surfaces as Open Item §13 for Term 1 + Term 5 (CN-5-103 §16 amendment if needed).

### 10.5 Per-vertical elaboration

CN-6-001..004 each elaborate site deactivation for their vertical:

- Retail: in-flight sales abort with refund-or-complete; obligations transfer to surviving sites if multi-site tenant
- Restaurant: open table sessions finalize; kitchen tickets either complete or cancel-with-refund
- Hotel: in-house guests check out normally; future reservations cancel-with-refund or transfer to chain sibling site
- Workshop: in-flight projects either complete (if within window) or abort-with-customer-notification

This per-vertical detail is out of CN-6-103 scope; the doctrine here sets the lifecycle, vertical docs apply it.

---

## 11. Worked Patterns — Four Vertical Applications

Anchors build on the Term 6 character map established through CN-6-100 (Mama Halima Logistics), CN-6-101 (Mama Amina + Faraja + Lodge Serengeti + Mzee Hassan), and CN-6-102 (Salma + Kilimanjaro Lodge Moshi + Mama na Bwana Mwema).

### 11.1 WP1 — Mzee Hassan workshop mixed-scope manifest

**Anchor:** Karakana ya Mzee Hassan, Arusha **+ a hypothetical Moshi fabrication branch** *(hypothetical for Q4 demonstration; not a v1 commitment).*

**Demonstrates:** Q4 mixed-scope manifest per CTR-023; SP3 tenant-scope exception (style library); SP1 site-scope default (cuts); coexistence within one engine.

**Manifest excerpt (per §8.2):**

```
engine_id: workshop
scope_policy: site                                 # SP1 default

emits:
  - event_type: workshop.cut.executed.v1
    # scope_policy: site (inherited); payload carries site_id
  - event_type: workshop.style_library.entry.added.v1
    scope_ref: tenant                              # SP3 — style library tenant-scope
    # justification: style IP shared across fundi sites
  - event_type: workshop.project.completed.v1
    # scope_policy: site
  - event_type: workshop.bill.ready.v1
    # scope_policy: site
```

**Operational view:** Mzee Hassan registers a new parametric window-frame style. The event `workshop.style_library.entry.added.v1` lands tenant-scope — both Arusha and Moshi fundis see it. A few weeks later, an Arusha fundi cuts the first frame to that style: `workshop.cut.executed.v1 {site_id: "arusha", style_ref, ...}` — site-scope; only Arusha's cut history records it. The next month, a Moshi fundi cuts another frame to the same style: `workshop.cut.executed.v1 {site_id: "moshi", style_ref, ...}` — site-scope; only Moshi's cut history records it. The shared style flows tenant-wide; the cuts live where they happened.

### 11.2 WP2 — Hotel chain split-by-event-type

**Anchor:** Lodge Serengeti (Serengeti) + Kilimanjaro Lodge Moshi — same tenant per CN-6-102 character map.

**Demonstrates:** Q2 closure; rejection of "multi-site-by-nature for chains"; SP1 site-scope reservations + SP3 tenant-scope guest profile; multi-leg bookings bundled via `booking_id`.

**Single-site reservation (SP1 default):**

```
hotel.reservation.confirmed.v1 {
  reservation_id,
  site_id: "kilimanjaro-lodge-moshi",
  room_ref,
  party_ref,
  arrival_date,
  departure_date
}
```

**Multi-leg booking (Q2 closure — bundled, not multi-site-by-nature):**

Mama na Bwana Mwema (from CN-6-102 WP1) book a 5-night holiday: 3 nights Moshi → 2 nights Serengeti. The system emits TWO reservations sharing a `booking_id`:

```
hotel.reservation.confirmed.v1 {
  reservation_id: "res-A",
  site_id: "kilimanjaro-lodge-moshi",
  booking_id: "booking-X",
  arrival_date: "2026-08-14",
  departure_date: "2026-08-17",
  ...
}

hotel.reservation.confirmed.v1 {
  reservation_id: "res-B",
  site_id: "lodge-serengeti",
  booking_id: "booking-X",
  arrival_date: "2026-08-17",
  departure_date: "2026-08-19",
  ...
}
```

Each reservation Workflow runs independently at its site. The `booking_id` provides read-side aggregation for "the guest's full trip."

**Guest profile (SP3 tenant-scope exception):**

```
hotel.guest_profile.updated.v1 {
  party_ref: <mwema_party>,
  preferences: {...},
  loyalty_points,
  stay_history_ref: [...]
  # NO site_id — tenant-scope per SP3
}
```

The profile aggregates across the chain. The reservations remain site-scope. Universal Promotion handles loyalty (per CN-6-102 §11.4 pattern). Three event types, three scopes, all coherent.

### 11.3 WP3 — Faraja pharmacy controlled-substance site-default

**Anchor:** Faraja at Mama Amina's pharmacy counter, Kariakoo **+ a hypothetical second pharmacy at Mama Amina's Mwenge expansion under cousin Pendo's TFDA license** *(hypothetical for Q1 demonstration; not a v1 commitment).*

**Demonstrates:** Q1 closure; site-default + pack-driven tenant opt-in; TZ TFDA per-premises practice; Law 5 (compliance configured, not coded).

**TZ pack configuration (today):**

```
pack.pharmacy.controlled_substance.audit_scope_default = "site"
pack.pharmacy.controlled_substance.regulatory_body_ref = "tfda"
pack.pharmacy.controlled_substance.reporting_template_ref = "tfda_csr_monthly_v3"
```

**Site-scope controlled-substance dispensing:**

```
pharmacy.controlled_substance.dispensed.v1 {
  dispensing_id,
  site_id: "mama-amina-kariakoo",
  party_ref,
  controlled_category: "diazepam",
  quantity,
  dispenser_ref: <faraja>,
  cabinet_slot_ref,
  business_date
}
```

The ledger lives per-premises. Faraja's monthly TFDA report draws only from the Kariakoo site's events.

**Hypothetical Mwenge expansion:** Mama Amina opens a duka in Mwenge with cousin Pendo (also TFDA-licensed) running the pharmacy section. This is a separate TFDA premises registration: separate `site_id: "mama-amina-mwenge"`, separate ledger, separate monthly report. The pack does NOT change — site-default still applies. Pendo's TFDA report draws from Mwenge events; Faraja's from Kariakoo.

**Hypothetical Kenya jurisdiction (multi-jurisdiction v2 — deferred):** If/when CN-5-105 §11 v2 lands, a Kenyan pack might configure `audit_scope_default = "tenant"` (one pharmacist license covers multiple premises in some Kenya configurations). The vertical engine code does not change — pack interpretation does. The event vocabulary `pharmacy.controlled_substance.dispensed.v1` stays identical across jurisdictions per CN-6-102 NC6.

### 11.4 WP4 — Mama Halima Logistics multi-site-by-nature

**Anchor:** Mama Halima's freight brokerage, Dar es Salaam, trip Dar → Mwanza. Cross-reference CN-6-100 §12 + CN-6-102 §12.5.

**Demonstrates:** SP4 3-of-3 criteria satisfied; multi-site-by-nature manifest pattern; payload with `origin_site_id` + `destination_site_id`.

**SP4 3-of-3 test applied:**

| Criterion | Logistics trip status |
|-----------|------------------------|
| (1) Primary entity has its own identity traversing sites | ✓ The `logistics.trip` Workflow is one entity from dispatch to arrival, traversing both sites |
| (2) Both origin and destination authoritative at emission time | ✓ Bill payload carries both; freight rate computed from origin-destination pair; fuel consumption attributed to the trip not either site singly |
| (3) Universal engines need site disambiguation in payload | ✓ Inventory deducts fuel from origin depot; Accounting recognizes revenue against origin-destination pair per pack rate-card; Reporting projections distinguish origin-flow vs destination-flow KPIs |

All three pass → multi-site-by-nature justified.

**Manifest excerpt:**

```
engine_id: logistics
engine_kind: vertical
scope_policy: site
multi_site_capable: true                           # SP4 — all 3-of-3 criteria met

emits:
  - event_type: logistics.bill.ready.v1
    # multi-site payload — origin_site_id + destination_site_id
  - event_type: logistics.trip.in_transit_started.v1
    # multi-site payload
  - event_type: logistics.fuel.consumed.v1
    # site-scope — attributed to origin depot where issued
```

**Bill emission:**

```
logistics.bill.ready.v1 {
  bill_id,
  trip_id,
  origin_site_id: "dar-depot",
  destination_site_id: "mwanza-receiving",
  payer_party_ref,
  saleable_lines: [freight_charge, fuel_surcharge, demurrage],
  originating_workflow_ref: <trip_id>,
  business_date
}
```

Universal Inventory subscribes and deducts fuel from `origin_site_id`. Accounting auto-journals revenue against origin-destination pair per pack rate-card mapping. Reporting projections aggregate per-origin and per-destination KPIs. No site_id disambiguation required at subscription pattern level — the payload fields carry it.

**Counter-example for §7.1 fail discipline:** if Mama Halima's brokerage instead operated a single dispatch depot with all trips originating from Dar (no destination registry — destinations are customer-side, not in her registry), the trip would NOT pass SP4 criterion (3) since Universal engines wouldn't need destination_site disambiguation in her own books. She'd fall back to SP1 site-scope with destination as plain payload string. This shows the 3-of-3 discipline filtering correctly.

---

## 12. Boundaries + Open Items + Cross-Term Hooks

### 12.1 CN-6-103's place in the Term 6 corpus

| Concern | Owned by | CN-6-103 role |
|---------|----------|----------------|
| Vertical scope policy application (site/tenant/platform) | **CN-6-103** (this doc) | Authoritative |
| Universal scope doctrine | CN-5-101 | Parent — CN-6-103 applies, never amends |
| Platform scope mechanism | CN-4-006 | CN-6-103 honours boundary; verticals never participate |
| Per-operation scope_ref mechanism | CN-4-005 + CTR-023 | CN-6-103 §8 applies |
| Site registry mechanism | CTR-027 (Term 1 governs) | CN-6-103 §9 consumes |
| Vertical event naming | CN-6-102 | Sibling — naming syntax inherits scope semantics (multi-site payload conventions) |
| Vertical hand-off mechanics | CN-6-104 | Sibling — hand-off consumes scope-correctly-emitted events |
| Concrete vertical scope manifests | CN-6-001..004 | Consumers — each vertical declares its mixed-scope manifest |
| Mixed-Vertical Tenant scope coexistence | CN-6-105 | Sibling — multi-vertical tenants observe per-vertical scope rules |
| Future-vertical scope readiness | CN-6-901..904 + CN-6-905 | Apply SP1-SP8 to candidate verticals |

### 12.2 CTRs cited (no new CTRs)

- **CTR-023** (per-operation scope_ref) — closed; §8 applies
- **CTR-024** (site_id payload) — Term 6 ratified; SP5 reaffirms
- **CTR-026** (UI-09 site_id registry) — Term 6 ratified; SP6 + §9 reaffirm
- **CTR-027** (site + currency registry) — Term 1 governance pending; §9 consumes
- **CTR-044** (engine_kind: vertical) — Term 4 pending; §3 cites
- **CTR-045** (vertical onboarding governance) — Term 1 pending; §10 site deactivation governance dependency
- **CTR-046** (mechanizable anti-patterns) — expansion `2361014` covers SP2 emission-side; **SP8 subscription-side queued for next CTR-046 amendment cycle (DC-NN-e per §5.3)**

**No new CTRs opened.** Concept Lead parsimony bar honoured.

### 12.3 Open items inside Term 6 scope

- **SP8 subscription-side mechanizable check (DC-NN-e)** — queued for CTR-046 next amendment cycle; pairs with existing SP2 emission-side check to close both directions
- **Site deactivation event namespace** — pending Term 1 activation + potential CN-5-103 §16 amendment (`platform.*` or `tenant.*` root); CN-6-103 §10.4 documents vertical-side subscription contract only
- **Workshop multi-site project pattern** — deferred to CN-6-004; CN-6-103 §7.2 catalog placeholder
- **Multi-jurisdiction multi-site** — deferred to CN-5-105 §11 v2 + CN-6-103 v2 amendment; v1 single-jurisdiction per CTR-027

### 12.4 D-DISC cross-references

- **D-DISC-001 — tenant-customer promotion UX:** scope-relevant when customer-facing surfaces show "your loyalty balance" (tenant-scope read) vs "today's offer at this store" (site-scope read). CN-6-103 §6 + WP2 sets the underlying scope rules; Term 3 designs the surface.
- **D-DISC-002 — POS self-service expansion:** customer-operated POS at a kiosk is still site-scope (the kiosk is at one site). Scope-wise unchanged from cashier-operated POS; identity (CN-4-007) handles the customer-as-actor distinction.

### 12.5 The bar restated

Salma reads "today's revenue" and sees her site's number — site-scope. Mama Amina reads "all my counters today" and sees an aggregation across her two operations at the same site, still site-scope reads aggregated. Mzee Hassan, with Arusha plus a hypothetical Moshi branch, reads "all my active projects" and crosses tenant-scope — that aggregation is real to him because his business spans both. The Lodge Serengeti GM reads "tonight's occupancy" and sees one property's reservations — site-scope. Faraja prints her monthly TFDA controlled-substance report and the ledger is per-premises — site-scope by TZ regulation.

Each number lives at the scope that matches who's asking. SP1-SP8 are the rules that put it there. If a future vertical author proposes a scope that doesn't match how the real person at the till, the front desk, the controlled-substance cabinet, or the fundi station thinks — the proposal is wrong and CN-6-103 is the standard by which it is rejected.

---

*— End of CN-6-103 Vertical Scope Policy v1 —*
