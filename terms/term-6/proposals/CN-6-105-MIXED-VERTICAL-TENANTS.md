# CN-6-105 — Mixed-Vertical Tenants

> **Term:** 6 — Vertical Engines
> **Status:** Concept Phase v1.0 — Draft for Overseer review
> **Branch:** `claude/brave-johnson-S7Lky`
> **Reading order:** CN-6-100..104 (cross-cutting framework) → CN-6-001..004 (concrete verticals) → CN-6-005 (bridge patterns) → **this doc**
> **Ordering authority:** TERM-6 Brief §13 — eleventh Term 6 deliverable; **tenant-level doctrine** completing the three-layer architecture.

---

## 1. Purpose & Boundary

### 1.1 Mission

CN-6-105 closes the tenant-level layer of Term 6's three-part architecture. Per-vertical mechanics live in CN-6-001..004 (what each vertical does); cross-vertical interaction patterns live in CN-6-005 (how verticals interact via Foundation primitives); CN-6-105 answers the remaining question: **who activates multiple verticals + how they coexist within one tenant**.

This is a real-world question because real businesses grow into multiple verticals. Mama Amina opened her Kariakoo duka in retail; six months later she added a pharmacy counter with her cousin Faraja. Mzee Karim ran his bucha for years; he added a nyama-choma corner on weekends. Mzee Hassan's Arusha workshop fabricates windows on commission; his showroom sells stock pieces via retail POS. Lodge Serengeti operates a hotel + restaurant within one property; the chain extends to Kilimanjaro Lodge Moshi.

In each case, the tenant remains one tenant — Mama Amina is Mama Amina expanded, not a new pharmacy-tenant alongside her old retail-tenant. The verticals operate independently at engine level but share tenant-property configuration (tax profile, payment methods, regional agent). Cross-vertical interactions use the bridge patterns established in CN-6-005 (Obligation for charges, Inventory for items, Party for identity, Document for compliance). The tenant grows; the framework absorbs the growth without architectural change.

### 1.2 DOES vs DOES NOT

| CN-6-105 DOES | CN-6-105 DOES NOT |
|----------------|--------------------|
| Define MV1-MV9 Mixed-Vertical doctrine (tenant identity preservation + vertical coexistence + cross-vertical pack hook semantics) | Define cross-vertical bridge mechanisms (CN-6-005 territory) |
| Distinguish tenant-property pack hooks (uniform across verticals) from per-vertical hooks (vertical-specific) per MV5 | Author per-vertical engine specifics (CN-6-001..004 territory) |
| Establish tenant-property override doctrine (MV9) including narrowing-not-widening sub-rule per N1 | Define Term 1 governance content (CTR-045 + CTR-049 + CTR-027 pending Term 1) |
| Catalog 5 established Mixed-Vertical tenants from CN-6-001..005 corpus + cross-link to CN-6-005 bridge patterns per N6 | Introduce new vertical types (Brief §3.3 future verticals = CN-6-901..904) |
| Document activation + deactivation lifecycles preserving cross-vertical Obligations (N2 doctrine) | Specify the regional agent's L1 compliance accountability mechanism (Term 1 + Term 2 territory) |
| Document 6 worked patterns (4 established + 2 hypothetical with N5 flagging) | Specify tenant UX for managing multi-vertical operations (Term 3 territory) |
| Inherit RE11 payment-method abstraction throughout (zero provider names) | Open new CTRs — uses established framework |
| Honour Charter §1.1 — tenant identity preserved through expansion | Address cross-tenant scenarios (deferred per CN-6-005 §11 to Term 1 marketplace future) |

### 1.3 Audience

Term 6 itself (this completes the verticals layer); Term 1 (the tenant onboarding governance side; CTR-045 lifecycle); Term 2 (regional agent onboarding flow for multi-vertical tenants); Term 3 (UI surfaces that span verticals — guest profile across hotel+restaurant; expansion onboarding flows); Term 7 (coherence verification across vertical layer); Architects implementing multi-vertical tenant activation flows; future tenants who will grow into Mixed-Vertical operations.

### 1.4 Charter Compliance

| Law | How CN-6-105 honours it |
|-----|--------------------------|
| Law 1 — State from events only | Activation + deactivation as events; historical events immutable across vertical lifecycles (N2) |
| Law 2 — Engines isolated | MV2 + MV3 + MV8 — shared tenant ≠ shared engine state; verticals don't bypass bridges because they share a tenant |
| Law 3 — AI advisory only | Cross-vertical advisor suggestions per CN-5-010 still advisory; activation governance involves human decision (Term 1) |
| Law 4 — Flexibility first-class | Tenants grow into multi-vertical without framework change; same engine + new vertical configuration |
| Law 5 — Compliance configured | Tenant-property pack hooks shared (tax_profile, payment_methods); compliance lives in pack content per jurisdiction |
| Law 6 — Distribution regional | regional_agent_ref is tenant-property per Charter Law 6; same agent serves the tenant across all verticals; vertical activation respects regional pack curation |

### 1.5 Parsimony — Mama Amina remains Mama Amina

When Mama Amina opens the pharmacy counter, she doesn't become "a pharmacy tenant who also runs retail." She is Mama Amina, expanded. Her duka's loyalty card still works; the customer-base she built over years still recognizes her; the regional agent who onboarded her in Kariakoo still serves her; her tax profile, her enabled payment methods, her functional currency stay the same. What changes is the operational layer — Faraja now works at the back counter dispensing prescriptions; the catalog grows to include prescription pharmaceuticals (handled by pharmacy.* vertical) alongside the OTC paracetamol that stays on her duka shelves (handled by retail.* vertical).

Lodge Serengeti is one lodge running a hotel + restaurant within the same property. Mzee Karim runs one shop with a bucha counter at the front and a BBQ corner out back. Mzee Hassan fabricates windows and sells the spares from his showroom. In every case, **the business is one business**. The verticals are operational specializations within that business. The framework's role is to honour that operational reality without forcing the tenant to maintain artificial separations.

**Parsimony is the bar.** MV1-MV9 doctrine below preserves tenant identity through expansion, maintains engine isolation between verticals, and lets cross-vertical interactions flow through Foundation primitive carriers cleanly. Charter §1.1's promise — that BOS serves the tenant as the tenant actually operates — is honoured at the Mixed-Vertical layer here.

---

## 2. Inputs and Relationship

### 2.1 Cross-cutting framework (parent)

- **CN-6-100** VE2 (engine isolation preserved within tenant); VE7 Workflow primitive (per-vertical Workflows independent)
- **CN-6-101** BD3 (vertical uniqueness); BD7 (no bridge engines even within one tenant)
- **CN-6-102** NC9 compensation pairs across vertical lifecycles
- **CN-6-103** SP3 tenant-scope exceptions (where state spans sites within tenant; relevant to multi-vertical tenants per HOT4 + CN-6-104 §10)
- **CN-6-104** HO5 fan-out (one event consumed by multiple universal engines per vertical); §10 Obligation cross-vertical doctrine

### 2.2 Concrete verticals (source of Mixed-Vertical examples)

- **CN-6-001 Retail** — RE6 B2C/B2B Party metadata applies across tenant; tenant-scope catalog
- **CN-6-002 Restaurant** — REST7 charge-to-room emission side (cross-vertical); §17 Mixed-Vertical Activation references
- **CN-6-003 Hotel** — HOT5 charge-to-room subscription; HOT4 chain guest profile (tenant-scope per SP3)
- **CN-6-004 Workshop** — §17 Mixed-Vertical Activation references; WS11 sell-via-Retail
- **CN-6-005 Bridges** — pattern catalog (the HOW that CN-6-105 builds the WHO on)

### 2.3 Foundation + Universal layer

- **CN-4-011** Party (shared identity across verticals at tenant); Document (shared compliance documents); Obligation (cross-vertical settlement)
- **CN-4-020** Extension Points (vertical registration mechanism)
- **CN-5-105** N7 tenant_tax_profile (canonical tenant-property pack hook)
- **CN-5-007** Promotion (cross-vertical loyalty adjudicator)

### 2.4 Sibling Term 6

- **CN-6-005 Vertical Bridges** — cross-vertical mechanism layer that CN-6-105 references
- **CN-6-901..904 stress sketches** (next, final) — Brief §14 Flexibility Test on future verticals

### 2.5 Cross-Term

- **Term 1 (Platform Stewards)** — CTR-045 vertical onboarding governance; CTR-049 regional payment curation; CTR-027 tenant property registries; activation approval workflow
- **Term 2 (Regional Distribution)** — onboarding flow for Mixed-Vertical activation; agent's L1 compliance accountability across tenant's verticals
- **Term 3 (Tenant Experience)** — UI surfaces spanning verticals (guest profile across stays + dining; expansion onboarding flows; D-DISC-001 promotion UX)

### 2.6 CTRs

- **No new CTRs.** Leverages CTR-018, CTR-027, CTR-045, CTR-049, CTR-050, CTR-046
- **Cumulative CTR-046 expansion queue (unchanged):** DC-NN-e + DC-NN-f
- §14 notes: future Term 1 activation may add tenant-property hooks via amendment

---

## 3. Mixed-Vertical Doctrine — MV1 to MV9

### MV1 — Tenant identity is preserved through vertical expansion

A tenant has one identity, one `tenant_id`, one regional_agent_ref, one tax_profile, one customer-base relationship. When the tenant activates a second vertical, **none of these change**. Mama Amina remains Mama Amina; her duka identity continues; her loyalty members continue to be hers. Vertical expansion is an operational change to the tenant's offerings, not an identity change.

Charter §1.1 is the foundational support for this doctrine — BOS serves the business as the business actually operates; the business is one business even when it spans operational categories.

### MV2 — Each vertical instance per tenant is independent at engine level

Verticals don't merge state when sharing a tenant. Retail.* events stay in retail.* namespace; pharmacy.* events stay in pharmacy.* namespace. Workflows per vertical run independently. Per VE2 isolation: a tenant operating retail + pharmacy has two engines running side-by-side, not one merged engine.

### MV3 — Cross-vertical interactions ALWAYS use CN-6-005 bridge patterns

Sharing a tenant does NOT permit cross-vertical event subscription, RPC, or shared state. Cross-vertical interactions within one tenant route through Foundation primitives exactly as cross-tenant interactions would: Obligation primitive carries charges (REST7+HOT5 within Lodge Serengeti, same as if hotel and restaurant were separate tenants); Inventory primitive carries items (Mzee Karim's carcass serving both bucha and BBQ, same as if they were separate); Party primitive carries identity (Mama Halima as customer at multiple verticals at one tenant).

**The tenant boundary is irrelevant to bridge mechanism.** This preserves VE2 + BD7 even at Mixed-Vertical scale.

### MV4 — Vertical activation is an auditable event

Each vertical activation emits two events:
- **Engine-side:** `<vertical>.activated.v1` per CN-4-020 registration (the technical activation of the engine for the tenant)
- **Tenant-side:** `tenant.vertical.activated.v1` per Term 1 governance (the tenant-property record of which verticals the tenant runs)

The tenant's active vertical set is auditable via projection over tenant.vertical.activated.v1 minus tenant.vertical.deactivated.v1 events. Per CTR-045 governance (pending Term 1).

### MV5 — Pack hooks have two levels: tenant-property vs per-vertical (refined)

**Tenant-property hooks** apply uniformly across ALL activated verticals:
- `tenant_tax_profile` (CTR-027 + CN-5-105 N7)
- `payment_methods_subset` (CTR-049)
- `regional_agent_ref` (Charter Law 6)
- `functional_currency_ref` (CTR-027)
- `site_registry` (CTR-027)
- Future: `default_language`, `default_timezone`, etc. (extensible per CTR to Term 1)

**Per-vertical hooks** apply only to that vertical's operations:
- `pack.retail.*` — retail.* operations only
- `pack.restaurant.*` — restaurant.* operations only
- `pack.hotel.*`, `pack.workshop.*`, `pack.pharmacy.*` — per vertical

Verticals **inherit** tenant-property hooks at activation; per-vertical hooks are vertical-specific configuration. §7 elaborates the canonical list.

### MV6 — Vertical deactivation is per CTR-045 lifecycle

A tenant deactivating a vertical (Mama Amina decides to close the pharmacy section; Lodge Serengeti closes a spa expansion) follows a controlled lifecycle:
- In-flight Workflows complete naturally per pack `pack.<vertical>.deactivation.workflow_continuation_window_days`
- Historical events remain immutable per Law 1
- Cross-vertical Obligations remain in Foundation primitive (N2 doctrine §11)
- `tenant.vertical.deactivated.v1` emits at completion
- Engine-side cleanup follows

### MV7 — Activation ordering is audit trail, not dependency hierarchy

Vertical activation ordering matters operationally — Mama Amina activated retail first (her original duka), then added pharmacy six months later. But this order does NOT create a hierarchical dependency: retail is not parent-of-pharmacy; pharmacy could remain if retail closed; pharmacy is not "second-class" vertical. The order is recorded as part of the audit trail; verticals coexist as peers.

**N3 concrete example:** Mama Amina activated retail in January 2026; added pharmacy in August 2026 with Faraja. If she ever closes the retail counter (rare; hypothetical), the pharmacy could continue independently — Faraja's TFDA license is separate from Mama Amina's basic business license; the pharmacy's events, Workflows, and customer relationships continue. The Jan-2026 retail activation is **audit trail** — it tells us when retail started — but creates no operational dependency between the two verticals.

### MV8 — Multi-vertical tenant SHARES Foundation primitives but NOT engine state

Within one tenant:
- **Shared:** Party primitive instances (one customer is one Party seen by all verticals); Document primitive (tenant's alcohol license is one Document); Obligation primitive (a charge from one vertical to another lives in Obligation); Inventory primitive (items can move between verticals via lot model); Workflow primitive (state machinery is shared mechanism)
- **NOT shared:** retail.* engine projection vs pharmacy.* engine projection are separate (cannot query directly); event store namespaces are separate (`retail.*` vs `pharmacy.*` events live separately); per-vertical Workflows don't cross-reference each other directly

This is engine isolation preserved at Mixed-Vertical scale.

### MV9 — Tenant-property hook overrides per-vertical default + narrowing-not-widening (N1)

**Sub-rule 9.1 — Tenant-property overrides per-vertical default:**

When a vertical's default behavior conflicts with tenant-property configuration, tenant-property wins. Example: Mama Amina activates retail (which has default tax computation behavior assuming VAT-registered tenant per pack.retail rules) + pharmacy (which has its own default tax assumption). If `tenant_tax_profile` says "non-VAT-registered" (most small dukas in TZ), the tenant-property profile overrides both verticals' defaults. Both verticals operate at zero-rate computations because the tenant's actual tax status is non-registered, regardless of pack-level defaults per vertical.

**Sub-rule 9.2 — Per-vertical may NARROW tenant subset but CANNOT WIDEN (N1):**

A vertical may narrow a tenant-property subset for regulatory or operational reasons but cannot widen beyond what the tenant has configured. Example with payment methods:

| Tenant-level (CTR-049) | What it means |
|------------------------|----------------|
| `payment_methods_subset = [cash, mobile_money, card, bank_push]` | These are the methods Mama Amina has configured for her tenant |

| Per-vertical narrowing | Example |
|------------------------|---------|
| Pharmacy controlled-substance dispensing | May narrow to `[mobile_money, bank_push]` only (regulatory pack rule preventing cash for narcotic class dispensing) |
| Retail standard sales | Inherits full tenant set `[cash, mobile_money, card, bank_push]` |

| Forbidden widening | Why prohibited |
|--------------------|----------------|
| Restaurant tries to add `cheque` if tenant didn't enable | NO — vertical cannot accept what tenant hasn't enabled at platform level |
| Retail tries to add `crypto` if tenant didn't enable | NO — regional agent's curation + tenant configuration are the ceiling |

**Why this doctrine:** the regulatory floor (per-vertical narrowing) preserves compliance per vertical's specific rules; the tenant ceiling (no widening) preserves Charter Law 6 distribution accountability — the regional agent vouches for the tenant's enabled set; verticals cannot bypass.

§8 elaborates with concrete examples.

---

## 4. Tenant Identity Preservation (MV1 Elaborated)

### 4.1 Charter §1.1 anchor

Charter §1.1's promise: *"African businesses — from the mama selling tomatoes in Kariakoo to the workshop building windows in Arusha to the hotel in Zanzibar — share a common pain... They cannot trust their own numbers."* The promise is the **business** trusting its numbers, where the business is the singular entity Mama Amina identifies as. If BOS forced Mama Amina to maintain "Duka la Mama Amina" and "Pharmacy ya Mama Amina" as separate tenants, BOS would have created the operational fragmentation it promised to solve.

MV1 doctrine — tenant identity preserved — is the operational mechanism that honours the Charter promise at Mixed-Vertical scale.

### 4.2 What stays the same through expansion

When Mama Amina activates pharmacy:
- `tenant_id` unchanged
- Onboarding regional agent (the agent who registered her in Kariakoo) continues to serve her
- `tenant_tax_profile` continues (whatever VAT status she has; pharmacy expansion doesn't auto-register her for VAT)
- `payment_methods_subset` continues (same set of enabled tender methods)
- `functional_currency_ref` continues (TZS for her tenant)
- `site_registry` continues (her duka at Kariakoo is the site; pharmacy is added at same site or new site as separate site_id)
- Customer base continues (loyalty members are tenant-property via Party primitive)
- Reporting + Accounting books continue (one set of books per tenant)

### 4.3 What changes

- A new vertical is activated technically (per MV4 events)
- Operational layer extends to include pharmacy-specific Workflows + events
- Catalog grows (pharmacy SKUs alongside retail SKUs)
- Possibly new sites added (if pharmacy is at a new location — but often at same site as retail)
- Possibly additional regulatory evidence required at activation (pharmacy needs TFDA license; retail typically doesn't)

### 4.4 The expansion narrative for tenants

Per Charter §1.1: Mama Amina grew from a single-line duka to a duka + pharmacy hybrid. From BOS's perspective: one tenant grew operations into a second vertical. From Mama Amina's perspective: her business grew; she serves more customer needs from the same premises. These two perspectives converge in MV1 doctrine — the tenant identity is preserved; the operational scope expanded.

---

## 5. Vertical Independence + Coexistence (MV2 + MV8 + MV7)

### 5.1 What "independence" means

Per MV2: each vertical's engine runs its own Workflows, emits its own events, maintains its own projections. Retail.* table_sale Workflow doesn't know pharmacy.* prescription Workflow exists. Pharmacy.* dispensing Workflow doesn't reference retail.* basket state. Per VE2 + BD7, this is engine isolation; per MV2 + MV3, the isolation is preserved even when verticals share a tenant.

### 5.2 What "coexistence" means

Per MV8: Foundation primitives are shared. One Party instance (the customer) is referenced by both verticals via party_ref. One Document instance (the tenant's regulatory licenses) is referenced by both verticals via document_ref. One Obligation instance (a cross-vertical charge) lives in the Obligation primitive and both verticals reference it via obligation_ref.

This is the architectural elegance of the framework: verticals isolated; Foundation primitives shared; the primitives carry cross-vertical state without bridging the engines.

### 5.3 MV7 concrete example — Mama Amina retail + pharmacy ordering

Per N3:

**January 2026:** Mama Amina onboarded retail.* for her Kariakoo duka. Tenant created; retail.* activated. Salma at the till; loyalty programme starts; customers accumulate.

**August 2026:** Mama Amina expands; activates pharmacy.* with Faraja. `tenant.vertical.activated.v1 {vertical: pharmacy, activated_by_party: <mama_amina>}` emits. Engine-side: `pharmacy.activated.v1` per CN-4-020 registration.

**Operational reality after expansion:**
- Retail continues as before; Salma at front till; nothing changes
- Pharmacy.* operations begin at back counter; Faraja dispenses prescriptions; her TFDA license recorded as Document per CTR-045
- Cross-vertical loyalty: customers who shop OTC at front + fill prescriptions at back accumulate loyalty across both per Identity-Linking bridge (CN-6-005 §7); Universal Promotion adjudicates per Party primitive — no cross_vertical events
- Books: one set per tenant; revenue projected per-engine in Accounting subscriptions; combined Accounting reports show split (retail revenue + pharmacy revenue) per pack chart-of-accounts mapping

**Hypothetical deactivation scenario (illustrating MV7 no-hierarchy):** Suppose Mama Amina decided in 2027 to close the retail counter entirely (she finds the pharmacy more profitable; she leases the front to another tenant). She deactivates retail.* per CTR-045 lifecycle. Pharmacy.* continues independently — Faraja's pharmacy operations don't depend on retail's existence. The Jan-2026 retail activation event remains in audit history; the Aug-2026 pharmacy activation continues to be valid; the verticals were peers, not parent-child.

Activation ordering is audit trail. Verticals coexist as peers.

### 5.4 What this prevents

Without MV7 doctrine, an implementation might create "vertical_parent_ref" or "primary_vertical" markers, leading to a class of bugs where verticals appear hierarchical. MV7 prevents that explicitly — the activation order is recorded but creates no operational dependency.

---

## 6. Established Vertical Pair Catalog

Per N6: cross-link table demonstrating the three-layer architecture. The catalog shows established Mixed-Vertical tenants from CN-6-001..005 corpus, the verticals involved, and the bridge patterns from CN-6-005 that they use.

### 6.1 The catalog

| Tenant | Verticals | Bridge category | CN-6-005 WP reference |
|--------|-----------|-------------------|------------------------|
| Mama Amina Kariakoo | retail + pharmacy | Identity-Linking (Party) + Item-Transfer (OTC Inventory) | CN-6-005 WP4 |
| Mzee Karim Kariakoo | retail (bucha) + restaurant (BBQ corner) | Item-Transfer (Inventory native lot) | CN-6-005 WP5 |
| Mzee Hassan Arusha | workshop + retail (showroom) | Item-Transfer (Inventory; WS11) | CN-6-005 WP2 |
| Lodge Serengeti | hotel + restaurant | Charge-Transfer (Obligation; REST7+HOT5) + Identity-Linking (Party guest profile) | CN-6-005 WP1 |
| Lodge Serengeti chain | hotel multi-site + restaurant multi-site | Identity-Linking (HOT4 chain guest profile per SP3) | CN-6-005 WP3-style |
| **Hypothetical:** Mama Amina growth | retail + pharmacy + restaurant (chai-jamia) | Three-vertical Identity-Linking + Item-Transfer | §13 WP5 demonstration |

### 6.2 What the cross-link demonstrates

The catalog makes the three-layer architecture concrete:

- **CN-6-001..004** authored each vertical's mechanics (Mama Amina's retail; the pharmacy expansion via CN-6-101 §11.6; Lodge Serengeti's hotel + restaurant; Mzee Hassan's workshop; etc.)
- **CN-6-005** catalogued the cross-vertical patterns (Identity-Linking, Item-Transfer, Charge-Transfer)
- **CN-6-105 (this)** documents the tenant-level activation perspective (who has which verticals + how they coexist)

For an Architect implementing, say, "Mama Amina expansion to pharmacy," the path is:
1. Read CN-6-001 for retail mechanics + CN-6-101 §11.6 for the pharmacy decision criterion
2. Read CN-6-005 WP4 for the Identity-Linking + Item-Transfer bridge patterns
3. Read CN-6-105 for the tenant-level activation lifecycle + tenant-property hook inheritance

Three layers; three docs; complete picture.

### 6.3 Speculative + future

The hypothetical Mama Amina 3-vertical growth (retail + pharmacy + chai-jamia restaurant) is documented in §13 WP5 per N5 flagging. It demonstrates the framework's extensibility but is not a v1 commitment — Mama Amina as of CN-6-101 §11.6 has retail + pharmacy only.

Future verticals (Insurance, Healthcare, Education, Marketing per Brief §3.3) will surface new Mixed-Vertical combinations as they get sketched in CN-6-901..904 and eventually concretely activated. The catalog grows per BR5 living-catalog (parallel to CN-6-005 §12 amendment gate).

---

## 7. Tenant-Property vs Per-Vertical Pack Hooks (MV5 Elaborated)

### 7.1 The distinction matters

A multi-vertical tenant has configuration that applies tenant-wide (one tax profile applies to all verticals; one payment-methods set is curated by one regional agent across the tenant) AND configuration that applies per-vertical (recipe content for restaurant; rate cards for hotel; styles for workshop). MV5 doctrine names this distinction explicitly.

### 7.2 N4 — Canonical tenant-property hooks list

Per N4, the canonical list (extensible per Term 1 activation + CTR amendment):

| Hook | Owner / Mechanism | Reference |
|------|--------------------|-----------|
| `tenant_tax_profile` | Pack content per CN-5-105; tenant-level | CTR-027 + CN-5-105 N7 |
| `payment_methods_subset` | Regional pack curation + tenant configuration | CTR-049 |
| `regional_agent_ref` | Charter Law 6 + D-003; one agent per tenant accountable | Charter Law 6 |
| `functional_currency_ref` | CTR-027 tenant property registry | CTR-027 |
| `site_registry` | CTR-027; canonical list of tenant's site_id values | CTR-027 |
| `default_language` (future) | Term 3 inheritance; UI surface default | Term 3 future |

When Term 1 activates and additional tenant-property concerns surface (e.g., default_timezone, business_classification per regulatory pack), the list extends via amendment to CN-6-105 + corresponding Term 1 pack governance.

### 7.3 Per-vertical hooks (illustrative, not exhaustive)

Per-vertical hooks live in their respective vertical pack namespaces — referenced in CN-6-001..004 + CN-6-002 §16 alcohol-licensing for example:

- `pack.retail.*` — retail-specific configuration
- `pack.restaurant.*` — F&B-specific configuration (service_model, ticket_routing, etc.)
- `pack.hotel.*` — hospitality-specific (rate_cards, charge_to_room_enabled, etc.)
- `pack.workshop.*` — workshop-specific (style storage, cut_optimization, etc.)
- `pack.pharmacy.*` — pharmacy-specific (controlled_substance handling, audit_scope_default)

Each per-vertical hook affects ONLY that vertical's operations within the tenant.

### 7.4 Cross-cutting: how the tenant sees configuration

Mama Amina onboarding her tenant configures once:
- Her regional agent records her business details + regulatory evidence
- `tenant_tax_profile` set (likely "non-VAT-registered" given small duka)
- `payment_methods_subset` selected from regional agent's enabled set
- `functional_currency_ref` set to TZS
- `site_registry` registers her Kariakoo duka site

When she later activates pharmacy.*:
- All the above stays — no re-configuration
- New: per-vertical pack hooks for pharmacy (controlled_substance handling, etc.)
- New: TFDA license document captured via CTR-045 (pharmacy requires this; retail didn't)

When she later activates restaurant.* (hypothetical WP5):
- All tenant-property hooks stay
- New: restaurant pack hooks (service_model: counter_service for chai-jamia; ticket_routing: none)

**Onboarding once; expansion per vertical.** That's the operational benefit of MV5's distinction.

---

## 8. Tenant-Property Override Doctrine (MV9 Elaborated)

### 8.1 Sub-rule 9.1 — Tenant-property overrides per-vertical default

When per-vertical default behaviour conflicts with tenant-property configuration, tenant-property wins.

**Example: Mama Amina tax computation:**

Without MV9, retail.* might default to "compute VAT per pack.retail.tax rules" + pharmacy.* might default to "compute per pack.pharmacy.tax rules." Each vertical computes per its defaults. If Mama Amina's tenant_tax_profile is "non-VAT-registered" (most small TZ dukas), both computations should yield zero-rate output.

With MV9: `tenant_tax_profile` overrides both verticals' default rate computation. Both verticals' Universal Tax wiring per CN-5-105 N7 checks tenant_tax_profile first; if non-registered, zero-rate path applies regardless of per-vertical defaults.

This prevents the failure mode where Mama Amina expanding to pharmacy suddenly has VAT-tagged transactions on her pharmacy side while her retail stays zero-rated — that would be a Mixed-Vertical bug. MV9 prevents it doctrinally.

### 8.2 Sub-rule 9.2 — Per-vertical may NARROW tenant subset but CANNOT WIDEN (N1)

**Direction matters.** Verticals can narrow the tenant-level set (for regulatory or operational reasons) but cannot widen it.

**Concrete payment method example:**

```
Tenant-level (CTR-049 + tenant onboarding):
  Mama Amina's payment_methods_subset = [cash, mobile_money]

Per-vertical narrowing (regulatory reason):
  Pharmacy controlled-substance dispensing:
    pack.pharmacy.controlled_substance.allowed_payment_methods = [mobile_money]
    (Excludes cash because pack rule requires traceable payment for controlled substances)
    
  Mama Amina's retail standard sales:
    Inherits full tenant set [cash, mobile_money]
```

**Forbidden widening:**

```
HYPOTHETICAL VIOLATION (not allowed):
  Pharmacy tries to add: pack.pharmacy.allowed_payment_methods = [cash, mobile_money, card]
  
  Why forbidden: Tenant-level subset doesn't include 'card'. Regional agent vouched 
  for tenant accepting [cash, mobile_money] only. Vertical cannot widen beyond what 
  tenant has enabled at platform level.
  
Doctrine gate: At CN-4-020 registration time, vertical pack hook payment_methods is 
validated as subset of tenant payment_methods_subset.
```

### 8.3 The doctrinal floor + ceiling structure

| Layer | Role |
|-------|------|
| **Regional pack (CTR-049, Term 2 curates)** | What methods are available in the region — the regional ceiling |
| **Tenant-level subset (Mama Amina's selection from regional set)** | What methods this tenant has enabled — the tenant ceiling |
| **Per-vertical narrowing (regulatory floor)** | What methods this vertical's specific regulations require — the regulatory floor |

A given vertical operation accepts payment methods at the intersection: regional ∩ tenant ∩ vertical-narrowed. Wider sets above are valid context; narrower sets below are valid restrictions.

This structure preserves Charter Law 6 (regional agent accountability) + tenant autonomy + per-vertical compliance.

### 8.4 Why MV9 is critical

Without MV9, Mixed-Vertical scale would surface a class of bugs where verticals contradict each other or override tenant intent. With MV9 explicit:

- Tax computations are consistent (tenant_tax_profile applies uniformly per CN-5-105 N7)
- Payment method availability is predictable (tenant ceiling enforced)
- Regulatory restrictions are applied per vertical (narrowing for compliance)
- Audit chain is clean (verticals can't introduce surprises by widening)

MV9 doctrinally prevents the class. Implementation enforces via tenant-property check at command-time across verticals.

---

## 9. Activation Lifecycle (MV4 + Q1 + Q2 Closures)

### 9.1 Two events per activation (Q1 closure)

**Per-vertical engine-side activation event** (existing per CN-4-020):

```
<vertical>.activated.v1 {
  tenant_id,
  engine_id: <vertical>,
  manifest_ref,
  activated_at,
  activated_by: <agent or platform>
}
```

This is the technical registration: the engine is now wired to the tenant's event store; the engine's manifest is loaded; the engine is ready to receive commands.

**Tenant-side governance event** (per Term 1 + CTR-045 pending):

```
tenant.vertical.activated.v1 {
  tenant_id,
  vertical: <vertical_name>,
  activating_party_ref: <tenant_owner OR regional_agent>,
  regulatory_evidence_refs: [<documents>],
  business_date,
  governance_approval_chain
}
```

This records the governance side: who decided this activation; what regulatory evidence was captured; what approval chain validated it.

The two events together form the complete activation record — technical + governance.

### 9.2 Sequential per-vertical activations (Q2 closure)

Multi-vertical tenants emerge sequentially:
- T0: `tenant.created.v1` (Term 1)
- T1: `retail.activated.v1` + `tenant.vertical.activated.v1 {vertical: retail}`
- T+N: `pharmacy.activated.v1` + `tenant.vertical.activated.v1 {vertical: pharmacy}` (months later)
- T+M: (future) `restaurant.activated.v1` + `tenant.vertical.activated.v1 {vertical: restaurant}`

Even when a tenant onboards with multiple verticals from day 1 (rare but valid), activation is still sequential per vertical to preserve audit clarity:

- T0: tenant.created
- T0+1: retail.activated + tenant.vertical.activated
- T0+2: restaurant.activated + tenant.vertical.activated
- T0+3: hotel.activated + tenant.vertical.activated

Each activation is its own discrete event chain. Sequence preserved in event timestamps.

### 9.3 What activation triggers

For each vertical activation:
- Engine registration per CN-4-020 mechanism
- Manifest loaded; subscriptions wired; commands accepted
- Tenant-property pack hooks inherited (tax_profile, payment_methods, etc.)
- Per-vertical pack hooks initialized to pack defaults
- Per-vertical Workflow primitive instances become creatable
- Catalog management Workflows (retail.catalog, restaurant.menu_management, etc.) become active
- Engine begins emitting + subscribing per manifest

### 9.4 What activation does NOT trigger

- No automatic data migration from other activated verticals
- No cross-vertical Workflow links (per MV3 — bridges via Foundation primitives only)
- No tenant identity change (per MV1)
- No automatic Universal layer reconfiguration (subscriptions automatic per CN-5-100)

The activation is **additive** to the tenant's operational scope; it does not modify existing state.

---

## 10. Cross-Vertical Pack Hook Inheritance

### 10.1 The inheritance flow

When a vertical activates, tenant-property pack hooks are inherited automatically. The vertical's Universal engine wiring (Tax via CN-5-105, Promotion via CN-5-007, etc.) reads tenant-property hooks via standard pack lookups:

```
pack.tax.lookup(item_category, tenant_tax_profile, business_date)
```

This works at retail, restaurant, hotel, workshop, pharmacy — every vertical because the lookup signature is universal; only the `item_category` argument varies per vertical context.

### 10.2 Concrete inheritance examples

**Tax profile across Mama Amina's verticals:**

```
Tenant: Mama Amina Kariakoo
tenant_tax_profile: { vat_registered: false, business_classification: micro_retail }

Retail.bill.ready.v1 saleable_lines:
  Line 1: rice 5kg → tax_treatment_ref = pack.tax.lookup('grain', tenant_tax_profile, business_date)
                  → zero-rate path (Mama Amina non-VAT-registered)

Pharmacy.bill.ready.v1 saleable_lines (after expansion):
  Line 1: paracetamol_strip → tax_treatment_ref = pack.tax.lookup('OTC_pharma', tenant_tax_profile, business_date)
                            → zero-rate path (same tenant_tax_profile)

Both verticals inherit the same tax_profile; both compute zero-rate per N7 of CN-5-105.
If Mama Amina ever registers for VAT, ONE update to tenant_tax_profile changes both 
verticals' tax behavior simultaneously.
```

**Payment methods across Lodge Serengeti's verticals:**

```
Tenant: Lodge Serengeti
payment_methods_subset: [mobile_money, card, bank_push, charge_to_room]

Hotel folio settlement (HO1):
  Available methods: [mobile_money, card, bank_push] (charge_to_room is intra-tenant, 
    handled at obligation level)

Restaurant bill settlement (HO1):
  Available methods: [mobile_money, card, bank_push, charge_to_room]
  (charge_to_room available because hotel is also activated AND pack.restaurant.
   hotel_charge_to_room_enabled: true per WP3 of CN-6-105)

Both verticals draw from same tenant-property subset; restaurant gains charge_to_room 
option because of the hotel activation + bridge pattern (CN-6-005 WP1).
```

**Regional agent accountability:**

```
Tenant: Mzee Hassan Arusha
regional_agent_ref: <agent_for_arusha_region>

Workshop project bill settlement → agent vouches for this tenant
Retail showroom sale settlement → same agent (one regional agent per tenant per 
  Charter Law 6)

Per MV1: regional_agent_ref is tenant-property; doesn't change with vertical 
expansion.
```

### 10.3 Why automatic inheritance matters

Without automatic inheritance, every vertical activation would require re-configuring tax + payment methods + agent + currency + sites. Multi-vertical tenants would face friction at every expansion. MV5 + automatic inheritance makes Mixed-Vertical operationally seamless — Mama Amina expanding to pharmacy doesn't re-do her tax registration; Lodge Serengeti adding a future spa doesn't re-configure payment methods.

---

## 11. Deactivation Lifecycle (N2 Doctrine + Q5 Closure)

### 11.1 The doctrine — N2: Vertical Deactivation Does Not Orphan Cross-Vertical Obligations

When a vertical deactivates, cross-vertical Obligations involving that vertical do NOT vanish. The Foundation Obligation primitive holds the obligation independent of either vertical's continued operation; counterparty obligations continue to be honoured; settlement happens via Foundation primitive coordination.

**Six principles:**

**Principle 1 — Obligations remain in Foundation primitive.** An obligation created by a vertical that subsequently deactivates is still in the Obligation primitive's state. The obligation_id remains valid; the obligation lifecycle continues.

**Principle 2 — Counterparty continues to honour.** If Workshop deactivates while a customer's project has an outstanding warranty obligation (workshop owes the customer a repair commitment), the obligation continues; the tenant remains responsible.

**Principle 3 — Foundation emits obligation.settled directly.** When the counterparty side completes (customer pays the outstanding balance via Universal Checkout; Term 1 closes the deactivation gracefully), the Foundation Obligation primitive emits obligation.settled.v1 even if the originating vertical's engine has been deactivated. The obligation primitive doesn't require the originating vertical to be active.

**Principle 4 — Tenant retains responsibility.** A deactivated vertical doesn't absolve the tenant. Mama Amina deactivating pharmacy doesn't escape her responsibility for outstanding controlled-substance audit obligations to TFDA. The tenant continues to be the responsible party.

**Principle 5 — In-flight Workflows complete per pack.** `pack.<vertical>.deactivation.workflow_continuation_window_days` (default 30) governs how long in-flight Workflows have to complete before being force-closed. During the window, the engine remains operationally active for completing in-flight only — no new commands accepted.

**Principle 6 — Historical events immutable.** Per Law 1: deactivation does not delete or modify historical events. All retail.* events from Mama Amina's retail period remain in the event store; replay can reconstruct her retail history forever.

### 11.2 Q5 closure — concrete deactivation flow

```
Tenant: Lodge Serengeti
Activated verticals: hotel + restaurant + (hypothetical) spa

Deactivation of spa (hypothetical):

1. Term 1 governance approves deactivation request
2. tenant.vertical.deactivation_initiated.v1 emits {vertical: spa}
3. Spa engine enters "deactivating" state — no new commands accepted; 
   existing in-flight Workflows complete (in-flight spa appointments + 
   any obligations referencing spa)
4. Pack window: pack.spa.deactivation.workflow_continuation_window_days = 30
5. During window:
   - Outstanding spa.appointment Workflows complete (guests get their appointments)
   - Any obligations involving spa (e.g., spa-charges-to-folio) settle naturally
6. At end of window OR all in-flight completed:
   - spa.deactivated.v1 emits (engine technical)
   - tenant.vertical.deactivated.v1 emits (governance)
   - Engine registration removed from active set
7. Historical events remain immutable
8. Cross-vertical Obligations involving spa that may not have settled yet:
   - Still in Obligation primitive's state
   - Foundation primitive continues to handle their lifecycle
   - When settlement occurs (future hotel folio that absorbed a spa charge), 
     Obligation primitive emits obligation.settled.v1 normally
   - Spa engine doesn't need to be active for this to work
```

### 11.3 What this preserves

- Customer trust: in-flight appointments complete; outstanding warranties honour
- Audit integrity: historical events immutable; full reconstruction possible
- Cross-vertical safety: Obligations don't orphan; settlements complete naturally
- Tenant responsibility: deactivation isn't escape from regulatory obligations
- Operational flexibility: tenants can experiment with verticals + close ones that don't work

The deactivation lifecycle is as carefully designed as the activation lifecycle. Both honour the framework's commitments to verifiability + tenant trust + cross-vertical isolation.

---

## 12. Activation Governance (Term 1 Dependency)

### 12.1 Term 1 owns the governance side

Per CTR-045 (pending Term 1 activation), the governance side of vertical activation lives in Term 1's domain:
- Regulatory evidence capture (TFDA license for pharmacy; alcohol license for pub; TLB license for hotel)
- Approval chain workflows (single-agent activation for simple cases; multi-step approval for regulated cases)
- Tenant-property hook initialization (tax_profile, payment_methods, etc.)
- Audit trail of who approved what activation when
- Deactivation governance (per N2 doctrine §11)

### 12.2 What CN-6-105 specifies (technical side)

CN-6-105 specifies what happens **after** Term 1 approves the activation:
- MV4 two-event emission (engine-side + governance-side)
- Tenant-property hook inheritance (MV5 + §10)
- Cross-vertical bridge availability (verticals can interact via CN-6-005 patterns)
- Tenant-property override doctrine (MV9 + §8)
- Deactivation lifecycle (N2 + §11)

### 12.3 The interface between Term 1 + Term 6

Term 1 governs activation; Term 6 specifies vertical mechanics. The interface is the activation event chain:

```
Term 1 governance approves activation request
    ↓
Term 1 emits tenant.vertical.activated.v1 (governance side)
    ↓
Engine-side CN-4-020 registration triggered by governance event
    ↓
Engine emits <vertical>.activated.v1 (technical side)
    ↓
Tenant-property hooks inherited automatically (per §10)
    ↓
Vertical is operationally active
```

Both terms' work meets at the activation boundary; neither owns the other's territory.

### 12.4 Pending CTR-045 closure

CTR-045 (vertical onboarding governance, Term 6 → Term 1) is pending Term 1 activation. When Term 1 activates and closes CTR-045 with concrete governance content, CN-6-105 will reference the closed content; until then, §12 acknowledges the dependency without speculating on Term 1's specific implementation.

---

## 13. Worked Patterns — Six Mixed-Vertical Scenarios

### 13.1 WP1 — Mama Amina activates pharmacy alongside existing retail

**Anchor:** Mama Amina's Kariakoo duka has been operating in retail.* for six months. Her customer Faraja completes her TFDA pharmacist exam; together they decide to add a pharmacy counter at the back of the shop. (Cross-doc continuity from CN-6-101 §11.6 + CN-6-102 §11.4 + CN-6-001 WP4 + CN-6-005 WP4.)

**Activation flow:**

1. Mama Amina's regional agent (the agent who originally onboarded her in Kariakoo) prepares the pharmacy activation:
   - Captures Faraja's TFDA license as Document (CN-4-012) per CTR-045 expansion
   - Captures premises license amendment (her duka's license now covers retail + pharmacy)
   - Submits activation request to Term 1 governance

2. Term 1 approves activation. Emits:
   ```
   tenant.vertical.activated.v1 {
     tenant_id: <mama_amina>,
     vertical: pharmacy,
     activating_party_ref: <her_regional_agent>,
     regulatory_evidence_refs: [<faraja_tfda_license>, <amended_premises_license>],
     business_date: 2026-08-15
   }
   ```

3. Engine-side activation per CN-4-020:
   ```
   pharmacy.activated.v1 {
     tenant_id: <mama_amina>,
     engine_id: pharmacy,
     manifest_ref,
     activated_at: 2026-08-15
   }
   ```

4. Tenant-property hooks inherited automatically per §10:
   - `tenant_tax_profile`: non-VAT-registered (unchanged from retail activation)
   - `payment_methods_subset`: [cash, mobile_money] (unchanged)
   - `regional_agent_ref`: same Kariakoo agent
   - `functional_currency_ref`: TZS

5. Per-vertical pack hooks initialized to pack defaults:
   - `pack.pharmacy.controlled_substance.audit_scope_default`: site (TZ TFDA pattern per CN-6-001/103)
   - `pack.pharmacy.controlled_substance.regulatory_body_ref`: TFDA

6. Pharmacy.* engine begins emitting + subscribing. First operation:
   - Customer presents prescription at Faraja's counter
   - `pharmacy.prescription.validated.v1` emits
   - First cross-vertical loyalty event: customer is identified per Party primitive (already known from her retail history); pharmacy.bill.ready.v1 carries party_ref; Universal Promotion adjudicates per CN-6-005 §7 Identity-Linking pattern

**What this proves:** Activation lifecycle (MV4 two-event); tenant-property inheritance (MV5); cross-vertical Identity-Linking emerging immediately (CN-6-005 WP4); MV1 — Mama Amina remains Mama Amina; she expanded.

### 13.2 WP2 — Mzee Karim activates BBQ restaurant.* corner alongside bucha retail.*

**Anchor:** Mzee Karim's Kariakoo bucha (retail.*) has been operating; weekends are busy. He activates a nyama-choma BBQ corner (restaurant.*) using the same carcass inventory. (Cross-doc continuity from CN-6-001 WP5 + CN-6-002 WP5 + CN-6-005 WP5.)

**Activation flow (abbreviated):**

1. Mzee Karim's regional agent captures restaurant activation request
2. Term 1 approves: `tenant.vertical.activated.v1 {vertical: restaurant}`
3. Engine side: `restaurant.activated.v1`
4. Tenant-property hooks inherited (same Kariakoo regional agent; same payment methods; same tax_profile)
5. Per-vertical hooks: `pack.restaurant.service_model: grill_to_table`; `pack.restaurant.ticket_routing: [grill]`; `pack.restaurant.menu_complexity: simple`
6. First operation: customer orders 1kg nyama-choma fillet:
   - `restaurant.order.placed.v1`
   - `restaurant.grill.ticket.fired.v1`
   - Recipe Pattern B fires: `restaurant.ingredient.consumed.v1 {ingredient_ref: ng'ombe_carcass, quantity: 0.25kg}` — referencing the SAME carcass lot the bucha is selling kg-portions from at the front
   - Inventory primitive's lot model handles both: bucha sales deduct kg quantities; restaurant grill consumption deducts kg quantities; one lot, two verticals, per CN-6-005 WP5

**What this proves:** MV2 + MV3 + MV8 — restaurant.* and retail.* run independently at engine level; Foundation Inventory primitive shared; cross-vertical Item-Transfer via Inventory primitive automatic per established bridge pattern.

### 13.3 WP3 — Lodge Serengeti hotel+restaurant canonical with tenant-property inheritance

**Anchor:** Lodge Serengeti tenant runs hotel + restaurant within one property. Mwemas (cross-doc canonical guests) charge dinner to room. The tenant-property hooks apply uniformly across both verticals (MV5 + MV9 illustrated).

**Configuration:**

```
Tenant: Lodge Serengeti (multi-site chain: Serengeti site + Kilimanjaro Lodge Moshi site)
tenant_tax_profile: VAT-registered (Lodge Serengeti is large enough to be registered)
payment_methods_subset: [mobile_money, card, bank_push, charge_to_room]
regional_agent_ref: <agent_for_serengeti_region>
functional_currency_ref: TZS

Activated verticals: hotel + restaurant
```

**Operational illustration:**

1. Mwemas check in at Lodge Serengeti hotel
2. They dine at the restaurant on night 2
3. Restaurant bill emits with `payment_method_hint: charge_to_room` — CN-6-005 WP1 chain fires
4. **Tax computation via tenant_tax_profile (MV5 + MV9 in action):**
   - Restaurant.* default tax behavior: 18% VAT per pack.restaurant rules
   - Tenant_tax_profile: VAT-registered
   - MV9 sub-rule 9.1 applies: tenant_tax_profile aligns with vertical default (both expect VAT)
   - Tax: 18% applied
   - Hotel folio: 18% VAT applied to room nights
   - Both verticals: same VAT treatment because same tenant_tax_profile
5. **Payment methods:**
   - Hotel folio settlement: available methods = [mobile_money, card, bank_push]
   - (charge_to_room is intra-tenant routing, handled at Obligation level)
   - Restaurant bill: would have same set + charge_to_room available (because hotel is also activated)
   - Per MV9 sub-rule 9.2: verticals cannot widen tenant subset
6. Mwemas check out; hotel folio settles via their chosen method (per RE11 abstract); all obligations resolve.

**What this proves:** Tenant-property uniform inheritance (MV5); tax + payment consistent across verticals; bridge pattern fires (CN-6-005 WP1); MV1 — Lodge Serengeti operates as one tenant with two engines.

### 13.4 WP4 — Mzee Hassan workshop + retail showroom Mixed-Vertical perspective

**Anchor:** Mzee Hassan Arusha runs workshop.* (fabrication) + retail.* (showroom). Cross-doc from CN-6-004 §17 + WP8 + CN-6-005 WP2.

**MV1 identity preservation:**
Mzee Hassan is Mzee Hassan, fundi and shopkeeper simultaneously. His workshop operations (commission projects per Brief §7) and his showroom retail (spec pieces sold off-the-shelf) are operations of the same business. His regional agent in Arusha vouches for him across both. His customers know him as Mzee Hassan, not "Mzee Hassan's workshop subsidiary" or "Mzee Hassan's retail subsidiary."

**Cross-vertical bridge: Item-Transfer (CN-6-005 WP2 / WS11):**
Workshop completes a window; designates for showroom; Inventory primitive holds; Retail catalog references; walk-in contractor buys; Inventory deducts. Workshop and Retail never communicate directly; Foundation Inventory primitive carries.

**Tenant-property:**
One Arusha regional agent; one tenant_tax_profile (VAT-registered for B2B commercial workshop work); one payment_methods_subset; functional currency TZS. All apply uniformly per MV5.

**What this proves:** Multi-vertical tenant with operational identity preserved (MV1); cross-vertical Item-Transfer per established bridge pattern (CN-6-005 WP2); tenant-property inheritance uniform; MV8 — Foundation primitives shared, engine states isolated.

### 13.5 WP5 — Mama Amina future growth hypothetical (3-vertical) *(hypothetical per N5)*

> ***N5 Flagging:** This worked pattern is hypothetical for 3-vertical demonstration; not a v1 commitment. Real Mama Amina has retail + pharmacy as of CN-6-101 §11.6.*

**Anchor (hypothetical):** Mama Amina in 2027 — having operated retail + pharmacy successfully for over a year — decides to add a chai-jamia (chai vendor) corner. This activates a third vertical: restaurant.* with counter_service + takeaway_only configuration.

**Three-vertical activation flow:**

1. Regional agent prepares restaurant.* activation request
2. Term 1 approves: `tenant.vertical.activated.v1 {vertical: restaurant}`
3. Engine side: `restaurant.activated.v1`
4. Tenant-property hooks inherited (unchanged from her retail + pharmacy state):
   - `tenant_tax_profile`: non-VAT-registered (still — she hasn't hit threshold yet)
   - `payment_methods_subset`: [cash, mobile_money] (her established set)
   - Other hooks unchanged
5. Per-vertical hooks for restaurant chai-jamia configuration:
   - `pack.restaurant.service_model: counter_service`
   - `pack.restaurant.ticket_routing: [none]` (Mama Amina or family member pours chai directly)
   - `pack.restaurant.session_model: takeaway_only`
   - `pack.restaurant.menu_complexity: simple`

**Three-vertical operational illustration:**

A regular customer arrives. She buys:
- Rice + soap at retail counter (Salma serves) → `retail.sale.completed.v1`
- Prescription refill at Faraja's pharmacy → `pharmacy.bill.ready.v1`
- Chai + mandazi at the chai corner → `restaurant.counter_order.placed.v1`

Three transactions; three verticals; one customer (same Party); one tenant. Loyalty across all three via Universal Promotion + Party primitive (cross-vertical Identity-Linking per CN-6-005 §7).

**MV9 narrowing-not-widening illustrated:**
All three verticals inherit tenant payment_methods_subset = [cash, mobile_money]. None widens (e.g., pharmacy can't add card if tenant didn't enable). If pharmacy controlled-substance pack rule narrows to [mobile_money] for narcotic class items, retail and chai-jamia stay at full set [cash, mobile_money].

**What this proves:** Three verticals at one tenant; no architectural limit (MV1 + MV2 + Q6); tenant identity preserved; bridge patterns scale to three verticals; framework absorbs without modification.

### 13.6 WP6 — Lodge Serengeti deactivates spa.* hypothetical *(hypothetical per N5)*

> ***N5 Flagging:** This worked pattern is hypothetical for deactivation demonstration; not a v1 commitment. Real Lodge Serengeti has hotel + restaurant as documented in CN-6-002/003; no spa activation exists.*

**Hypothetical scenario:** Lodge Serengeti activated a spa.* vertical in 2027 (hypothetical activation). After a year of operation, the spa proves unprofitable and Lodge Serengeti decides to close it. Deactivation flow per MV6 + N2:

1. Term 1 governance approves deactivation request from Lodge Serengeti management
2. `tenant.vertical.deactivation_initiated.v1 {vertical: spa}` emits
3. Spa engine enters "deactivating" state — no new spa appointments accepted; existing in-flight appointments + outstanding spa-folio charges allowed to complete
4. Pack window: `pack.spa.deactivation.workflow_continuation_window_days = 30`
5. During the window:
   - Guests with already-booked appointments receive their treatments
   - Spa charges that were absorbed into in-progress folios settle normally at guest checkout via the existing Obligation primitive chain
   - One particular guest had spa charges on a still-open folio; her hotel checkout 5 days into the deactivation window settles the spa charges normally; spa-side Obligation resolves; revenue recognised
6. At end of window: all in-flight completed; `spa.deactivated.v1` emits; `tenant.vertical.deactivated.v1` emits
7. Historical events remain immutable (months of spa operation preserved for audit)
8. Obligations that may not have settled (e.g., a small refund owed to a guest from a service complaint): remain in Foundation Obligation primitive; tenant remains responsible; settlement when refund issued via remaining tender path

**What N2 doctrine prevents:**
- Spa charges from completed appointments don't get orphaned mid-folio
- Historical spa data not lost (audit replay still works)
- Outstanding obligations don't vanish (tenant accountability preserved)
- Hotel + restaurant continue operating unaffected by spa closure

**What this proves:** Deactivation lifecycle preserves cross-vertical safety (N2 doctrine); Foundation primitives carry obligations across vertical lifecycle changes; tenant continues to honour commitments; framework is robust to vertical churn.

---

## 14. Boundaries + Open Items + Cross-Term Hooks

### 14.1 CN-6-105's place in the corpus

| Concern | Owned by | CN-6-105 role |
|---------|----------|----------------|
| Tenant-level Mixed-Vertical activation doctrine | **CN-6-105** (this doc) | Authoritative |
| Per-vertical engine mechanics | CN-6-001..004 | CN-6-105 references for vertical specifics |
| Cross-vertical bridge patterns | CN-6-005 | CN-6-105 uses as mechanism layer |
| Cross-cutting vertical framework | CN-6-100..104 | Parents — CN-6-105 honours VE2 + BD7 across Mixed-Vertical |
| Foundation primitives (shared layer) | CN-4-011 + CN-4-012 | CN-6-105 documents MV8 sharing semantics |
| Tenant onboarding governance | Term 1 (pending; CTR-045) | CN-6-105 §12 documents the dependency boundary |
| Regional agent multi-vertical onboarding flow | Term 2 (pending; CTR-049 + governance) | CN-6-105 §10.3 references |
| Cross-vertical tenant UX | Term 3 (pending; D-DISC-001) | CN-6-105 references; UI surface design future |
| Cross-tenant scenarios | Term 1 future marketplace | Deferred per CN-6-005 §11 + N5; not in CN-6-105 v1 |

### 14.2 CTRs (no new)

- CTR-018, CTR-027, CTR-045, CTR-049, CTR-050, CTR-046 — cited as-is
- **Cumulative CTR-046 expansion queue (unchanged):** DC-NN-e + DC-NN-f
- Future Term 1 activation may add tenant-property hooks via amendment (N4 list extensibility)

### 14.3 Open items inside Term 6 scope

- **CN-6-901..904 Flexibility Test stress sketches** — final Term 6 doc per Brief §13.12; Brief §14 commitment; will surface new Mixed-Vertical combinations (Insurance + Hotel for traveler insurance; Healthcare + Pharmacy for prescription routing; Education + Marketing for student-recruitment funnel)
- **Term 1 CTR-045 closure** — when Term 1 activates and authors vertical onboarding governance content, CN-6-105 §9 + §12 may need amendment to align with Term 1's specific mechanism
- **Term 3 D-DISC-001 closure** — cross-vertical UX (e.g., guest profile showing stays + dining + loyalty across verticals) is Term 3 territory; CN-6-105 §10 references but doesn't specify

### 14.4 D-DISC cross-references

- **D-DISC-001 — tenant-customer promotion UX**: Cross-vertical loyalty visible across multiple touchpoints (Mama Halima sees her balance whether she's at retail, restaurant, or hotel); Term 3 designs surfaces per CN-6-005 §7 Identity-Linking patterns
- **D-DISC-002 — POS self-service expansion**: Self-service across Mixed-Vertical tenant (e.g., guest at Lodge Serengeti can use mobile to order in-room dining, check out, manage stay — spanning hotel + restaurant); CN-6-105 confirms tenant-property hooks (single Party identity for customer-as-actor) work across verticals

### 14.5 The bar — Tenant identity through expansion

The three-layer architecture is now complete:

| Layer | Doc Range | Question Answered |
|-------|-----------|---------------------|
| **Per-vertical mechanics** | CN-6-001..004 | What each vertical does |
| **Cross-vertical interaction** | CN-6-005 | How verticals interact via Foundation primitives |
| **Tenant-level activation** | **CN-6-105 (this)** | **Who activates multiple + how they coexist** |

The doctrine MV1-MV9 ensures the framework honours Charter §1.1 at Mixed-Vertical scale: Mama Amina remains Mama Amina through her expansion to pharmacy and (hypothetically) to a third vertical; Lodge Serengeti operates as one lodge with one identity even with hotel + restaurant; Mzee Karim is a butcher and a BBQ guy under one Kariakoo shop awning. The tenant identity is the business identity; the framework adapts to serve the business as it actually operates.

**The framework breathes at tenant scale.** New verticals activate without architectural change; tenant-property hooks inherit uniformly; cross-vertical bridges fire automatically; deactivations preserve cross-vertical safety. The promise of Brief §6.2 — that the framework serves real Tanzanian SMEs through their growth — is honoured here.

Next: CN-6-901..904 Flexibility Test stress sketches — Brief §14 final commitment. After that, **Term 6 SCOPE COMPLETE.**

---

*— End of CN-6-105 Mixed-Vertical Tenants v1 —*
*— Three-Layer Architecture Doctrinally Complete —*
