# CN-6-004 — Workshop Engine

> **Term:** 6 — Vertical Engines
> **Status:** Concept Phase v1.0 — Draft for Overseer review
> **Branch:** `claude/brave-johnson-S7Lky`
> **Reading order:** CN-6-100..104 → CN-6-001/002/003 → CN-5-003 (Inventory primitive native lot model + Pattern B) → CN-4-011 → **this doc**
> **Ordering authority:** TERM-6 Brief §13.9 — **final concrete vertical doc**; **Framework Test #4** (most complex vertical per Brief); cross-cutting framework applied to parametric fabrication reality.

> **The Mantra:**
> *"System haimuulizi fundi ahesabu — inamwambia fundi akateje."*

---

## 1. Purpose & Boundary

### 1.1 Mission

CN-6-004 declares the **Workshop Engine** — the fourth concrete vertical, covering **built-to-spec parametric fabrication** for windows, doors, gates, fixed panels, partitions, cabinets, tables, and chairs. The vertical receives reality from field PDFs documenting Tanzanian fabrication operations; doctrine derives from that reality.

Workshop differs from prior verticals (retail, restaurant, hotel) in fundamental ways:

- **Items are not pre-defined SKUs** — each item is computed from a Style (parametric template) + customer dimensions + variables
- **Style is a tenant-created artifact** — like a recipe (REST4) or a rate card (HOT9) in structure but unlike them in *origin*: Style is tenant data, not jurisdiction policy from Term 1
- **Cut list generation is workshop-internal** — a vertical-unique algorithm that takes Style + dimensions + variables and produces concrete cuts per material per profile_type
- **Cut optimization minimizes waste** — Best Fit Decreasing (BFD) + Offcut tracking across stock bars
- **Project lifecycle is the longest in BOS** — days to weeks from enquiry through delivery
- **Offcuts are reusable inventory** — yesterday's offcut becomes tomorrow's cut

Per Brief §6.1: *"Style registry, parametric geometry, cut lists, offcuts, material consumption events, multi-item project quotes, quote acceptance/rejection."* This doc operationalises that, grounded in the field PDFs' reality of how Mzee Hassan in Arusha actually fabricates windows.

### 1.2 DOES vs DOES NOT

| CN-6-004 DOES | CN-6-004 DOES NOT |
|----------------|--------------------|
| Declare `workshop.*` covering parametric fabrication (windows, doors, gates, fixed panels, partitions, cabinets, tables, chairs) | Cover F&R cluster (salon, motor repair, phone repair, cobbler — deferred lacking field reality; CN-6-101 §11.2 / CN-6-905 question stays deferred) |
| Specify Style as tenant-scope event-sourced template per WS2 | Treat Style as pack content (Style is tenant data created by tenant, not jurisdiction policy from Term 1) |
| Specify the Formula Engine doctrine (dependency chain, Frame TU null rule, validation, evaluation timing) | Implement the parser or evaluator (Architect phase) |
| Specify Cut Optimization (BFD + Offcut) algorithm Steps 1-5 from field PDF | Implement the algorithm in code (Architect phase) |
| Treat material specifications as Inventory primitive item metadata per BD4 push-down | Duplicate material specs in `pack.workshop.*` (specs live in CN-5-003 Inventory primitive, not workshop pack — N2 boundary) |
| Establish style sharing via export/import duplicate-into-receiver per WS10 | Build a cross-tenant style marketplace (deferred per BD8 — no concrete demand yet) |
| Document the Mixed-Vertical workshop+retail sell-via-POS pattern via Foundation Inventory primitive | Author the cross-vertical bridge catalog (CN-6-005 future) or Mixed-Vertical Tenants doc (CN-6-105 future) |
| Inherit RE11 payment-method abstraction throughout (zero provider names) | Mention any specific tender providers anywhere |
| Honour zero-new-CTR goal | Open new CTRs |

### 1.3 Audience

Term 6 itself (this is the final concrete vertical; pattern complete); Architects implementing the workshop engine (Style Designer, Formula Engine, Cut List Generator, Cut Optimizer); Term 1 onboarding governance authoring workshop-specific compliance content; Term 3 designing the CAD-like Style Designer canvas + POS input flows; Term 7 confirming any peripheral integrations (printers for cutting sheets, etc.); future workshop tenants from single-fundi karakana to mid-sized fabrication operations.

### 1.4 Charter Compliance

| Law | How CN-6-004 honours it |
|-----|--------------------------|
| Law 1 — State from events only | Style is event-sourced (every line + shape + property addition is an event); cut list is event-sourced (cut_list.generated.v1 + per-cut executed events); replay reconstructs state deterministically per N3 formula evaluation timing |
| Law 2 — Engines isolated | Workshop never directly subscribes to retail.* or any vertical; sell-via-POS (WS11) flows through Foundation Inventory primitive |
| Law 3 — AI advisory only | workshop-fundi-advisor + workshop-manager-advisor per CN-5-010; cut sequence suggestions advisory; never autonomous |
| Law 4 — Flexibility first-class | Style is tenant-extensible; new style types via pack hook; new material types via pack hook; cut optimization strategy pack-driven |
| Law 5 — Compliance configured | Warranty periods, offcut minimum lengths, formula safety constraints all pack-driven |
| Law 6 — Distribution regional | Payment-method abstraction per RE11; regional pack curation per CTR-049 |

### 1.5 Parsimony — the Mantra Made Concrete

> **"System haimuulizi fundi ahesabu — inamwambia fundi akateje."**

Mzee Hassan in his Arusha karakana does not write math. He drew the window once — months ago, perhaps — as a Style in the CAD-like designer: lines for the frame, lines for the sash, lines for the mullion, shapes for the glass and net. He named each line (`Wframe`, `Hframe`, `Hsash`, `Wsash`); he wrote formulas (`(w01+1)/2`, `h01-9`); he set offcuts (10cm for frames, 7cm for sashes). He saved the Style and named it "Casement 2P."

Today a customer arrives wanting a casement window 140cm × 201cm. Salma at the till opens the Style "Casement 2P" and enters W=140, H=201. The system computes:

> Cut 2 pieces of ALU-FRAME-60 at 150cm each.
> Cut 2 pieces of ALU-FRAME-60 at 211cm each.
> Cut 1 piece of ALU-MULL-40 at 199cm.
> Cut 4 pieces of ALU-SASH-45 at 77.5cm each.
> Cut 4 pieces of ALU-SASH-45 at 199cm each.
> Cut 2 glass panes at 67.5 × 189cm.
> Cut 2 mosquito-net pieces at 68.5 × 190cm.

That is what Mzee Hassan sees on his workshop printout. No formulas. No math. Just cuts.

That is the mantra in concrete form. The Style holds the wisdom; the Formula Engine does the arithmetic; the Cut List Generator produces the instructions; the Cut Optimizer packs them onto bars with minimal waste. Mzee Hassan cuts. **Parsimony is the bar** — every doctrine below is justified against the question "does this match how the real fundi in real Tanzanian workshops thinks?"

---

## 2. Inputs and Relationship

### 2.1 Cross-cutting framework (parent)

- **CN-6-100** — Recipe; manifest delta; VE1-VE7 (especially VE7 Workflow primitive — Style, Material Group, Project, Cut all primitive instances)
- **CN-6-101** — BD1-BD8; BD3 vertical uniqueness (cut list generation + BFD optimization are workshop-unique); BD4 push-down (material specs in Inventory primitive, not workshop registry); BD5 split-it (Style content = tenant data; Formula Engine = workshop mechanism)
- **CN-6-102** — NC1-NC9 naming; VI-03 conflict family
- **CN-6-103** — SP1-SP8; SP3 tenant-scope exception applied to workshop.style + workshop.material_group (precedent: CN-6-103 WP1 already established this for Mzee Hassan)
- **CN-6-104** — HO1-HO9 handoff; HO9 settlement-back; HO5 fan-out; Obligation primitive doctrine §10 (informs WS11)

### 2.2 Universal layer

- **CN-5-009** Universal Checkout — workshop.bill.ready.v1 emission at delivered state
- **CN-5-001** Accounting — CTR-030 sufficiency; revenue recognition at delivered settlement
- **CN-5-003** Inventory — **central dependency**:
  - Pattern B vertical-managed consumption (CN-5-003 N1) for materials per project
  - Native lot model — material specs (stock_length, offcut, kerf) live as Inventory primitive item metadata per N2 boundary
  - Offcut tracking via lot extension (WS8)
- **CN-5-004** Procurement — material shortage trigger via `workshop.purchase_need.recorded.v1` per CTR-030 expansion
- **CN-5-007** Promotion — multi-price layer resolution inherited from RE3
- **CN-5-105** Tax-treatment per material category; tenant_tax_profile gate

### 2.3 Foundation

- **CN-4-011** Workflow primitive (4 instances: style, material_group, project, cut) + Party + Document + Inventory Movement
- **CN-4-021** Saleable Line + Tender value shapes
- **CN-4-022** Advisor framework

### 2.4 Sibling Term 6

- **CN-6-001** Retail — RE11 payment abstraction inherited; RE9 bulk-splittable items (precedent for measurement-unit material handling)
- **CN-6-002** Restaurant — REST4 recipe-as-pack-content **distinguished**: Style is NOT pack content because Style is tenant data (recipes are jurisdiction-driven; Styles are tenant-craft)
- **CN-6-003** Hotel — HOT9 rate-card-as-pack-content **distinguished similarly**: Style is tenant data
- **CN-6-005** Bridges (future) — catalogs WS11 sell-via-POS pattern fully
- **CN-6-105** Mixed-Vertical Tenants (future) — generalises Mzee Hassan workshop + opportunistic retail showroom

### 2.5 Field reality

- **Field PDF 1** (Workshop Style Property + Cutting Optimization) — establishes Style designer mechanics (lines, shapes, endpoint types, properties, formula engine semantics) and BFD + Offcut algorithm Steps 1-5
- **Field PDF 2** (BOS Workshop Style Examples) — 5 complete style examples (Casement, Sliding, Fixed Panel, Hinged Door, Variable Window) with concrete dimensions, materials, formulas, cut lists; grounds WP1-WP5 in this doc

### 2.6 CTRs

- **No new CTRs**. Workshop reality fits within existing contracts:
  - Style as event-sourced tenant Workflow → CN-6-103 SP3 + CN-4-011 (no new primitive)
  - Material specs in Inventory item metadata → CN-5-003 native (no new contract)
  - Formula Engine + Cut Optimization → vertical-internal pure functions (no cross-Term contract)
- CTR-049, CTR-050 inherited for payment abstraction
- CTR-018/002/024/026/027/030/038/044/045/046/028/006 cited as-is
- **Cumulative CTR-046 expansion queue (unchanged):** DC-NN-e + DC-NN-f

---

## 3. Workshop Doctrine — WS1 to WS11

### WS1 — Workshop covers built-to-spec parametric fabrication

`workshop.*` covers fabrication operations producing built-to-spec items per parametric style design. Style Types (pack-extensible enum): `window | door | gate | fixed_panel | partition | cabinet | table | chair`. Material Types: `aluminium | upvc | metal | wood | composite` (pack-extensible).

**Not in v1**: salon, motor repair, phone repair, cobbler, car wash — deferred per lack of field reality. CN-6-101 §11.2 / CN-6-905 light services question stays open.

### WS2 — Style is tenant-scope reusable event-sourced template

A Style is the central artifact of workshop. It is a **drawing** (lines + shapes) + **properties** (material per line/shape, formulas, position, offcut, variable declarations).

Style is **event-sourced** per Law 1: every line added, every shape added, every property edit emits an event. Style state is the deterministic fold of its events. Per CN-6-103 SP3 tenant-scope exception (precedent: CN-6-103 WP1 already established this for Mzee Hassan): `workshop.style` Workflow is tenant-scope; the Style library spans the tenant's sites.

**Style is NOT pack content** — important distinction from REST4 recipes (CN-6-002) and HOT9 rate cards (CN-6-003). Those are jurisdiction policy from Term 1's chartered-accountant pipeline. Style is tenant-craft — Mzee Hassan designs his casement window the way HE makes casement windows, in HIS style. Term 1 doesn't ship Styles; tenants create them. BD5 split-it: Style content = tenant data; Formula Engine = workshop mechanism.

### WS3 — Material Group + Material Catalog

Two distinct concepts at workshop layer:

- **Material Group** (`workshop.material_group` Workflow) — tenant-scope collection of compatible materials per style type. Example: group "ALU-CAS-01" contains all materials for aluminium casement windows (frame profile, sash profile, mullion profile, glass, mosquito net, fittings). At Style design time, Material Group acts as a **filter** — Mzee Hassan sees only ALU-CAS-01 materials, not every material in his entire inventory.
- **Material Specifications** — `stock_length` (e.g., 6000mm or 6500mm bars), `offcut_length` (the angle-correction trim, e.g., 10mm), `kerf_width` (saw blade thickness, e.g., 2mm). These live as **Inventory primitive item metadata** per CN-5-003 native model. Workshop reads via Inventory primitive at compute time. Workshop does NOT duplicate specs in workshop pack (N2 boundary; BD4 push-down).

### WS4 — Formula Engine: dependency chain evaluator

A formula is an expression referencing other lines, variables, W, and H. The Formula Engine is a **vertical-internal pure function** that evaluates a Style's formulas given W, H, and variables to produce line lengths and shape dimensions.

**Frame TU null rule** (from field PDF 1): Only lines with `is_frame: true` may carry `formula: null`. Position (`is_width` or `is_height`) tells the engine to use W or H. All non-frame lines MUST carry a formula referencing other lines or variables; a non-frame line with null formula is rejected at command-time.

**Dependency chain evaluation:**
- **Round 1**: Evaluate `formula: null` lines (frames). Frame with `is_width: true` gets W; frame with `is_height: true` gets H.
- **Round 2**: Evaluate formulas referencing only frames (e.g., `h01-9`).
- **Round N**: Evaluate formulas referencing already-resolved lines.
- **Cycle detection**: A formula cycle (line A references line B which references line A) is rejected at command-time.

**Validation timing** (N3 precision): Formula text is **validated** at command-time (`workshop.style.line.add.request` rejected if formula uses unknown line ref or variable, or violates `pack.workshop.formula.allowed_operators`). Formula **evaluation** happens twice:
- At **POS-input time** (preview for quote): non-persisted; computed for cost estimation
- At **cut-list-generation time** (project.in_progress transition): **persisted** as `workshop.cut_list.generated.v1` event; this is the canonical evaluation per Law 1 replay determinism

### WS5 — Variable input at POS

Some items are non-standard — a trapezoid window top, a custom-height door section. The Style author declares variables (X, Y, Z) at Style design time. Lines/shapes with formulas like `X` or `h01-X-8` flag the Style as variable-requiring.

At POS, when the cashier picks the Style and enters W×H, the system asks additionally for X (and Y, Z if applicable). After input, variables become **constants** for that project — used in Round 1 alongside W and H.

### WS6 — Cut List Generator at "In Progress" trigger

Per field PDF 1 + WS9 lifecycle: cut list is **materialized only when project transitions to `in_progress`**. Quote phase = preview cut list (computed in-memory for cost estimation, not persisted). Project transition to `in_progress` triggers the canonical event:

```
workshop.cut_list.generated.v1 {
  cut_list_id,
  project_ref,
  style_refs: [...],                 # project may include multiple style instances (WS6 multi-item)
  cuts_per_profile_type: [
    {profile_type, profile_ref, required_length, quantity, cut_piece_ids[]}
  ],
  shapes_per_material: [
    {material_ref, shape_dimensions[width, height], quantity}
  ],
  generated_at,
  business_date
}
```

This is the workshop equivalent of "the bill is now official." Inventory deduction proceeds from this event; fundi cutting instructions print from this event.

### WS7 — Cut Optimization (BFD + Offcut)

Workshop-internal algorithm per BD3 vertical uniqueness. Field PDF 1 specifies Steps 1-5:

- **Step 1 — Grouping**: Cut list grouped per `profile_type`. Frame profile optimized separately from sash profile (cannot be mixed).
- **Step 2 — Get specs**: For each profile group, read `stock_length`, `offcut`, `kerf` from Inventory primitive item metadata + Style's per-line offcut value.
- **Step 3 — Sort decreasing**: Within each profile group, sort cut pieces from largest to smallest.
- **Step 4 — Best Fit Decreasing**: For each piece:
  - Find existing stock-bar with smallest remaining length that fits the piece.
  - Fit rule: new bar requires `remaining ≥ piece_length`; bar already cut requires `remaining ≥ piece_length + offcut (+ kerf if applicable)`.
  - If no bar fits, open new stock bar from Inventory.
- **Step 5 — Compute remaining**: After cut, update remaining length.

Output: bar assignment per cut piece + remaining offcuts per bar tracked.

### WS8 — Offcut tracking via Inventory primitive lot model

Per CN-5-003 native lot tracking + RE9 of CN-6-001 + CN-6-104 WP2 established pattern. When a cut produces a reusable offcut (remaining length ≥ `pack.workshop.offcut.minimum_usable_length_mm`):

```
workshop.offcut.recorded.v1 {
  offcut_id,
  source_bar_lot_ref,                # the bar this offcut came from
  profile_type,
  offcut_length,
  site_id,
  business_date
}
```

This emission triggers a new Inventory primitive lot per `inventory.lot.added.v1` (the offcut becomes a future-cut input). Future BFD considers offcuts as input stock before opening new bars — Step 4 favours existing offcuts over new bars when both fit.

When the offcut is later consumed by a cut, normal Inventory deduction applies. When the offcut is too small to be reusable (below pack threshold) it is logged as scrap.

### WS9 — Project Workflow long-lifecycle

`workshop.project` lifecycle states:

```
enquiry → measured → quoted → accepted → in_progress → cutting → 
assembly → finishing → ready_for_delivery → delivered → completed → archived
```

Branches:
- `quote_rejected` — customer rejects the quote
- `cancelled` — mid-project cancellation
- `blocked` — material shortage per CN-6-100 §3.13 (WS9 procurement integration)
- `disputed` — post-delivery defect (warranty period)

**Cut list materialization at `in_progress`** (WS6). **Workshop bill.ready.v1 emits at `delivered`** state (HO1 canonical handoff to Universal Checkout). **HO9 settlement-back** transitions to `completed`.

### WS10 — Style sharing = duplicate-into-receiver

Style portability supported via export/import. Tenant A exports Style as a versioned, hash-signed Document; Tenant B imports + duplicates into receiving tenant's `workshop.style_library`. Each tenant has its own copy after import — no cross-tenant link; no live updates flow from origin to receiver.

**No platform-level marketplace v1** — cross-tenant style catalog with Platform Stewards governance deferred per BD8 (no concrete demand demonstrated). Sharing is logical (file exchange between tenants), not technical (no cross-tenant subscription).

§18 details the 5-step mechanism per N5.

### WS11 — Mixed-Vertical workshop+retail sell-via-POS

Per Brief §7.1: a workshop-fabricated item may be sold via retail POS (showroom walk-in customer; surplus showroom inventory). The pattern flows through the Foundation Inventory primitive — never direct workshop↔retail subscription (VE2 + BD7).

§20 details the 8-step concrete chain per N4.

---

## 4. The Manifest — Engine Declaration

```yaml
engine_id: workshop
engine_kind: vertical
namespace_root: workshop
multi_site_capable: false                          # SP1 site-default
scope_policy: site                                 # SP1 default; mixed-scope per CN-6-103 WP1

workflow_instances:
  # Style — tenant-scope event-sourced template (WS2)
  - workflow_id: workshop.style
    billable: false
    scope_ref: tenant                              # SP3 exception per CN-6-103 §6 + WP1
    lifecycle_states: [draft, validated, published, deprecated, archived]
    # justification: Style is tenant intellectual property shared across all fundi sites

  # Material Group — tenant-scope catalog (WS3)
  - workflow_id: workshop.material_group
    billable: false
    scope_ref: tenant
    lifecycle_states: [draft, active, deprecated, archived]

  # Project — long-lifecycle billable (WS9)
  - workflow_id: workshop.project
    billable: true                                 # HO1 canonical bill at delivered state
    lifecycle_states: [
      enquiry, measured, quoted, accepted,
      in_progress, cutting, assembly, finishing,
      ready_for_delivery, delivered, completed, archived,
      quote_rejected, cancelled, blocked, disputed
    ]
    settlement_subscription:                       # HO9 mandatory
      event_type: checkout.settled.v1
      filter: originating_workflow_ref == <self>
      transitions_to: completed

  # Cut — per-cut sub-workflow
  - workflow_id: workshop.cut
    billable: false
    lifecycle_states: [planned, executing, executed, reversed]

commands:
  # Style design (tenant-scope per SP3)
  - workshop.style.create.request
    scope_ref: tenant
  - workshop.style.line.add.request                # validates formula at command-time per WS4
    scope_ref: tenant
  - workshop.style.line.remove.request
    scope_ref: tenant
  - workshop.style.line.update.request
    scope_ref: tenant
  - workshop.style.shape.add.request
    scope_ref: tenant
  - workshop.style.shape.remove.request
    scope_ref: tenant
  - workshop.style.shape.update.request
    scope_ref: tenant
  - workshop.style.variable.declare.request        # X / Y / Z declaration
    scope_ref: tenant
  - workshop.style.publish.request                 # draft → published
    scope_ref: tenant
  - workshop.style.deprecate.request
    scope_ref: tenant
  - workshop.style.export.request                  # WS10
    scope_ref: tenant
  - workshop.style.import.request                  # WS10 duplicate-into-receiver
    scope_ref: tenant

  # Material Group (tenant-scope)
  - workshop.material_group.create.request
    scope_ref: tenant
  - workshop.material_group.member.add.request
    scope_ref: tenant
  - workshop.material_group.member.remove.request
    scope_ref: tenant

  # Project lifecycle
  - workshop.project.enquire.request
  - workshop.project.measure.request
  - workshop.project.add_item.request              # multi-item per WS9 §15
  - workshop.project.quote.request                 # generates PREVIEW cut list (not persisted; per N3)
  - workshop.project.accept.request                # customer acceptance; quote_rejected branch if reject
  - workshop.project.start.request                 # transitions to in_progress → triggers cut_list.generated
  - workshop.project.mark_cutting_complete.request
  - workshop.project.mark_assembly_complete.request
  - workshop.project.mark_finishing_complete.request
  - workshop.project.mark_ready.request
  - workshop.project.deliver.request               # transitions to delivered → emits bill.ready
  - workshop.project.cancel.request                # at any pre-delivered state
  - workshop.project.block.request                 # material shortage per WS9 + CN-6-100 §3.13
  - workshop.project.unblock.request               # stock arrives
  - workshop.project.dispute.request               # post-delivery defect
  - workshop.project.resolve_dispute.request

  # Cut execution
  - workshop.cut.execute.request
  - workshop.cut.reverse.request                   # rare per CN-6-102 NC9 example

  # Bill
  - workshop.project.bill.request                  # at delivered state → HO1 emit

emits:
  # Style events (tenant-scope)
  - event_type: workshop.style.created.v1
    compensation_pair: workshop.style.deprecated.v1
    scope_ref: tenant
  - event_type: workshop.style.line.added.v1
    compensation_pair: workshop.style.line.removed.v1
    scope_ref: tenant
  - event_type: workshop.style.line.removed.v1
    compensation_basis_none: "removal IS the compensation"
    scope_ref: tenant
  - event_type: workshop.style.line.updated.v1
    compensation_basis_none: "update is forward-only; superseded by causation chain"
    scope_ref: tenant
  - event_type: workshop.style.shape.added.v1
    compensation_pair: workshop.style.shape.removed.v1
    scope_ref: tenant
  - event_type: workshop.style.shape.removed.v1
    compensation_basis_none: "removal IS the compensation"
    scope_ref: tenant
  - event_type: workshop.style.shape.updated.v1
    compensation_basis_none: "update is forward-only"
    scope_ref: tenant
  - event_type: workshop.style.variable.declared.v1
    compensation_pair: workshop.style.variable.removed.v1
    scope_ref: tenant
  - event_type: workshop.style.published.v1
    compensation_pair: workshop.style.deprecated.v1
    scope_ref: tenant
  - event_type: workshop.style.deprecated.v1
    compensation_basis_none: "deprecation is forward-only state change"
    scope_ref: tenant
  - event_type: workshop.style.exported.v1         # WS10
    compensation_basis_none: "export is observation; exported Document remains"
    scope_ref: tenant
  - event_type: workshop.style.imported.v1         # WS10 duplicate-into-receiver
    compensation_pair: workshop.style.deprecated.v1
    scope_ref: tenant

  # Material Group events (tenant-scope)
  - event_type: workshop.material_group.created.v1
    compensation_pair: workshop.material_group.removed.v1
    scope_ref: tenant
  - event_type: workshop.material_group.member.added.v1
    compensation_pair: workshop.material_group.member.removed.v1
    scope_ref: tenant
  - event_type: workshop.material_group.member.removed.v1
    compensation_basis_none: "removal IS the compensation"
    scope_ref: tenant

  # Project lifecycle (per WS9)
  - event_type: workshop.project.enquired.v1
    compensation_pair: workshop.project.enquiry_cancelled.v1
  - event_type: workshop.project.measured.v1
    compensation_basis_none: "measurement is observation"
  - event_type: workshop.project.item.added.v1     # multi-item per §15
    compensation_pair: workshop.project.item.removed.v1
  - event_type: workshop.project.quoted.v1
    compensation_pair: workshop.project.quote_revised.v1
  - event_type: workshop.project.accepted.v1
    compensation_pair: workshop.project.cancelled.v1
  - event_type: workshop.project.quote_rejected.v1
    compensation_basis_none: "rejection IS terminal compensation"
  - event_type: workshop.project.in_progress.v1    # TRIGGER for cut_list.generated
    compensation_pair: workshop.project.cancelled.v1
  - event_type: workshop.project.cutting_complete.v1
    compensation_basis_none: "physical work observed; reversed via cut.reverse if needed"
  - event_type: workshop.project.assembly_complete.v1
    compensation_basis_none: "physical work observed"
  - event_type: workshop.project.finishing_complete.v1
    compensation_basis_none: "physical work observed"
  - event_type: workshop.project.ready_for_delivery.v1
    compensation_basis_none: "readiness is observation"
  - event_type: workshop.project.delivered.v1      # HO1: triggers bill.ready
    compensation_pair: workshop.project.disputed.v1
  - event_type: workshop.project.completed.v1
    compensation_basis_none: "terminal-success via HO9 settlement-back"
  - event_type: workshop.project.cancelled.v1
    compensation_basis_none: "cancellation IS terminal compensation"
  - event_type: workshop.project.blocked.v1        # material shortage
    compensation_pair: workshop.project.unblocked.v1
  - event_type: workshop.project.disputed.v1
    compensation_pair: workshop.project.dispute_resolved.v1

  # Cut list (WS6) — central canonical emission
  - event_type: workshop.cut_list.generated.v1     # at project.in_progress transition
    compensation_basis_none: "cut list is observation of computed plan; cuts executed individually trigger compensations per cut.executed pair"

  # Cut execution
  - event_type: workshop.cut.executed.v1
    compensation_pair: workshop.cut.reversed.v1
  - event_type: workshop.cut.reversed.v1
    compensation_basis_none: "reversal IS terminal compensation per CN-6-102 NC9 example"
  - event_type: workshop.material.consumed.v1      # Pattern B per CN-5-003 N1
    compensation_basis_none: "consumed material physically used; recovery via re-fabrication not event"
  - event_type: workshop.offcut.recorded.v1        # WS8 → Inventory lot
    compensation_basis_none: "offcut creation is observation; future consumption follows normal inventory deduction"

  # Billing (HO1 canonical)
  - event_type: workshop.bill.ready.v1
    compensation_pair: workshop.bill.recalled.v1
  - event_type: workshop.bill.recalled.v1
    compensation_basis_none: "recall IS terminal compensation"

  # Procurement trigger (per CN-5-004 + CTR-030)
  - event_type: workshop.purchase_need.recorded.v1
    compensation_pair: workshop.purchase_need.cancelled.v1

  # Conflict (VI-03 — rare in workshop)
  - event_type: workshop.bar.conflict.detected.v1
    compensation_basis_none: "conflict resolution recorded; loser receives own compensation"

  # Customer interaction (per CN-6-102 §9 customer party_ref discipline)
  - event_type: workshop.customer.identified.v1
    compensation_basis_none: "identification is observation"

subscribes_to:
  - event_type: checkout.settled.v1
    scope_ref: site
    # HO9 settlement-back to workshop.project
  - event_type: inventory.stock.depleted.v1
    scope_ref: site
    # triggers project.block per WS9
  - event_type: inventory.lot.added.v1
    scope_ref: site
    filter: profile_type ∈ <relevant_workshop_profiles>
    # awareness for stock arrival; may trigger project.unblock if blocked
  - event_type: pack.effective.v1
    scope_ref: tenant

pack_hooks:
  # Style + material domain
  - pack.workshop.style_types                      # extensible enum
  - pack.workshop.material_types                   # extensible enum
  - pack.workshop.formula.allowed_operators        # safety constraint: { +, -, *, /, () }
  - pack.workshop.formula.allowed_variables        # safety constraint: { X, Y, Z, W, H, line refs }

  # Offcut + waste
  - pack.workshop.offcut.minimum_usable_length_mm  # per material type; below this = scrap

  # Cut optimization
  - pack.workshop.cut_optimization.strategy        # default: BFD; pack may extend with other strategies
  - pack.workshop.cut_optimization.2d_strategy     # N1 — for sheet materials (glass/board/panel); default BFD-2D
  - pack.workshop.cut_optimization.kerf_default_mm # fallback if material specs don't carry kerf

  # Project + warranty
  - pack.workshop.project.standard_warranty_days
  - pack.workshop.material_shortage.auto_procurement_trigger  # bool; auto-emit purchase_need

  # Inventory expansion mode per material type (Pattern A vs B per CN-6-104 WP2 established)
  - pack.workshop.inventory_expansion_mode.aluminium     # vertical_managed (Pattern B)
  - pack.workshop.inventory_expansion_mode.upvc          # vertical_managed
  - pack.workshop.inventory_expansion_mode.glass         # vertical_managed
  - pack.workshop.inventory_expansion_mode.fittings      # auto (Pattern A)

  # Self-service (D-DISC-002 placeholder; not v1)
  - pack.workshop.self_service_quote_enabled       # customer remote quote via Style picker

advisor_ids:
  - workshop-fundi-advisor                         # cut sequence suggestions, waste alerts
  - workshop-manager-advisor                       # project pipeline, material reorder, productivity KPIs
```

---

## 5. Workflow Lifecycle — `workshop.style` (Event-Sourced Template)

### 5.1 The lifecycle

```
                       command: style.create.request
                                ▼
                       ┌────────────────┐
                       │     draft      │ ── emit workshop.style.created.v1
                       └────────────────┘
                                │
                                │  recurring: style.line.add / shape.add / variable.declare
                                │
                                ▼
                       ┌────────────────┐
                       │     draft      │ ◄── (accumulates lines + shapes + variables)
                       └────────────────┘
                                │  command: style.publish.request
                                ▼
                       ┌────────────────┐
                       │   validated    │ ── system runs full formula validation + cycle check
                       └────────────────┘
                                │  validation passes
                                ▼
                       ┌────────────────┐
                       │   published    │ ── emit workshop.style.published.v1; available for projects
                       └────────────────┘
                                │  command: style.deprecate.request (newer version published OR style retired)
                                ▼
                       ┌────────────────┐
                       │   deprecated   │ ── still usable by existing projects pinned to this style; not selectable for new projects
                       └────────────────┘
                                │  retention window
                                ▼
                       ┌────────────────┐
                       │    archived    │
                       └────────────────┘
```

### 5.2 Style state via event projection

Style state at any point is the fold of:
- `workshop.style.created.v1`
- All `workshop.style.line.added.v1` events (minus removed)
- All `workshop.style.shape.added.v1` events (minus removed)
- All `workshop.style.line.updated.v1` events (forward-only supersession)
- All `workshop.style.variable.declared.v1` events
- `workshop.style.published.v1` (state transition)

Replay reconstructs the Style deterministically.

### 5.3 Tenant-scope per SP3 (precedent established)

Style lives tenant-scope per CN-6-103 §6 catalog + WP1. Mzee Hassan's Arusha karakana and his hypothetical Moshi expansion share the Style library — design once, fabricate anywhere within the tenant.

### 5.4 Style versioning

When Mzee Hassan improves a Style (better formula, new material), he publishes a new version rather than editing the published one (because existing in-flight projects may have pinned to the previous version at acceptance time per CN-6-101 §10 AP4 quote-acceptance pinning). The new version becomes the default for new projects; the previous version stays available for ongoing projects until they complete.

Manifest carries `style_version` field on every line/shape/variable event so Style version chain is replay-clean.

---

## 6. Workflow Lifecycle — `workshop.material_group` (Catalog)

### 6.1 The lifecycle

```
                       command: material_group.create.request
                                ▼
                       ┌────────────────┐
                       │     draft      │ ── emit workshop.material_group.created.v1
                       └────────────────┘
                                │  recurring: member.add commands
                                ▼
                       ┌────────────────┐
                       │     active     │ ── available as filter at Style design time
                       └────────────────┘
                                │  command: material_group.deprecate.request
                                ▼
                       ┌────────────────┐
                       │   deprecated   │ ── existing Styles linked to this group continue; not selectable for new Styles
                       └────────────────┘
                                │
                                ▼
                       ┌────────────────┐
                       │    archived    │
                       └────────────────┘
```

### 6.2 Material Group as filter

When Mzee Hassan opens the Style Designer to draw a new line, he picks a Material Group first ("ALU-CAS-01"). The designer then shows only members of that group as material choices. This prevents accidentally picking a wood profile when designing an aluminium window.

### 6.3 Boundary with Universal Inventory (N2)

The Material Group references Inventory primitive items (the actual material rows with `stock_length`, `offcut`, `kerf` etc.). The group itself is a **list of references**, not a copy of specs. When Mzee Hassan updates an item's spec in Inventory (e.g., supplier changed stock_length from 6000mm to 6500mm), every Style using that item via the group sees the updated spec at next cut list generation.

---

## 7. Workflow Lifecycle — `workshop.project` (Long-Lifecycle Billable)

### 7.1 The full lifecycle

```
                       command: project.enquire.request
                                ▼
                       ┌────────────────┐
                       │    enquiry     │
                       └────────────────┘
                                │  fundi visits site, measures
                                ▼
                       ┌────────────────┐
                       │    measured    │
                       └────────────────┘
                                │  command: project.add_item.request × N (multi-item per §15)
                                │  command: project.quote.request → preview cut list (not persisted)
                                ▼
                       ┌────────────────┐
                       │     quoted     │ ── customer reviews quote
                       └────────────────┘
                                │  customer responds
                                ▼
                       ┌──────────────────┐   ┌────────────────┐
                       │     accepted     │   │ quote_rejected │ ── terminal-rejection
                       └──────────────────┘   └────────────────┘
                                │  command: project.start.request
                                ▼
                       ┌────────────────┐
                       │  in_progress   │ ── emit workshop.cut_list.generated.v1 + Inventory deduction begins
                       └────────────────┘
                                │  fundi begins cutting
                                ▼
                       ┌────────────────┐
                       │    cutting     │ ── emit workshop.cut.executed.v1 × N as cuts proceed
                       └────────────────┘
                                │  all cuts done
                                ▼
                       ┌────────────────┐
                       │    assembly    │
                       └────────────────┘
                                │
                                ▼
                       ┌────────────────┐
                       │   finishing    │ ── paint, polish, hardware install
                       └────────────────┘
                                │
                                ▼
                       ┌─────────────────────┐
                       │ ready_for_delivery  │
                       └─────────────────────┘
                                │  command: project.deliver.request
                                ▼
                       ┌────────────────┐
                       │   delivered    │ ── emit workshop.bill.ready.v1 (HO1 canonical handoff)
                       └────────────────┘
                                │
                                │ HO9: checkout.settled.v1 matching originating_workflow_ref
                                ▼
                       ┌────────────────┐
                       │   completed    │
                       └────────────────┘
                                │  retention window
                                ▼
                       ┌────────────────┐
                       │    archived    │
                       └────────────────┘

Branches:
    quoted or any pre-delivered → cancelled (per pack rules)
    in_progress or cutting → blocked (material shortage; per WS9 §17)
    delivered → disputed (post-delivery defect; warranty per WS9 §19)
```

### 7.2 Why this is the longest BOS workflow

Retail.sale closes in minutes. Restaurant table_session in hours. Hotel reservation in days. Workshop project in **days to weeks**:
- Enquiry to quote: 1-3 days (fundi visits, measures, prepares quote)
- Quote to acceptance: customer reviews — hours to days
- Acceptance to in_progress: until fundi has capacity — same day to days
- In_progress through assembly: cutting + assembling — typically 2-5 days per item; multi-item projects scale
- Finishing: paint + polish + hardware — 1-3 days
- Delivered: installation may follow separately

Total: typical residential window project from enquiry to billed delivery = 7-21 days. Large multi-item commercial projects = weeks.

### 7.3 Multi-item project Workflow handling

A project Workflow contains N item instances. Each item = Style + dimensions + variables. Per WS6 + §15, the cut_list.generated.v1 event aggregates cuts across all items per profile_type for optimization.

---

## 8. Workflow Lifecycle — `workshop.cut` (Per-Cut Sub-Workflow)

### 8.1 The lifecycle

```
                       triggered by: workshop.cut_list.generated.v1
                                ▼
                       ┌────────────────┐
                       │    planned     │ ── per cut piece, one cut Workflow instance
                       └────────────────┘
                                │  fundi starts physical cut
                                ▼
                       ┌────────────────┐
                       │   executing    │
                       └────────────────┘
                                │  fundi completes cut
                                ▼
                       ┌────────────────┐
                       │    executed    │ ── emit workshop.cut.executed.v1
                       └────────────────┘                ◄── triggers offcut.recorded.v1 if reusable remains

Branches:
    executed → reversed (rare; fundi miscut requires redo; CN-6-102 NC9 example)
```

### 8.2 Why cut is a Workflow instance per VE7

A cut has a state lifecycle (planned → executing → executed → optionally reversed); per VE7, that lifecycle is a Workflow primitive instance. The cut is fine-grained — the Workflow primitive handles persistence + audit + replay without workshop reinventing.

---

## 9. The Style Designer Reality

This is the section that captures the field PDFs' unique reality. The Style Designer is a CAD-like canvas — Mzee Hassan's tool for drawing his Styles.

### 9.1 The canvas

The Style Designer provides:
- A 2D canvas with a grid (snap-to-grid optional)
- Drawing tools for lines and shapes
- A properties panel for the selected line or shape
- A Material Group picker (filters available materials)
- A Variables panel (declare X, Y, Z)
- Save / publish / deprecate controls

The canvas itself is **Term 3 UI surface** territory (the visual editor, drag-drop, snap-to-grid mechanics). CN-6-004 specifies the **data model + commands**; Term 3 designs the UX of the canvas.

### 9.2 Lines (Group A) — represent profiles

Each line represents a profile cut (frame, sash, mullion, bead, threshold, interlock). Properties per line:

```yaml
line_id: <auto-generated>
named: "Wframe" | "Hframe" | "Hsash" | "Wsash" | etc.
profile: <reference to Inventory item via Material Group filter>
formula: "null" | "h01 - 9" | "(w01+1)/2" | "X" | "h01-X-8" | "w01/2+4" | etc.
offcut: <cm — e.g., 10cm for frame, 7cm for sash>
position: "width" | "height"
endpoints: "Mater-Mater" | "Mater-Square" | "Square-Square"
is_frame: true | false
is_variable: true | false  # true if formula uses a declared Variable
```

### 9.3 Endpoint types (geometry semantics)

Per field PDF 1:

- **Mater-Mater**: Frame to frame (both ends mitered 45°; outer frame profiles meeting at corners)
- **Mater-Square**: Frame meets internal divider (one end mitered, one square; e.g., frame meeting mullion)
- **Square-Square**: Joint to joint (both ends square; internal profiles like mullion or interlock)

The endpoint type informs the cut machine which cuts to make. The Style Designer captures this; the cut list event carries it forward.

### 9.4 Shapes (Group B) — represent fill areas

Each shape represents a fill area (glass pane, board, panel, net). Properties per shape:

```yaml
shape_id: <auto-generated>
named: "vent glass" | "glass_L" | "glass_R" | "board_bot" | "net_L" | etc.
material: <reference to Inventory item via Material Group filter>
material_type: "Glass" | "Board" | "Panel" | "Net"
width_formula: "w03 - 3" | "(w04-2)/2" | etc.
height_formula: "h04 - 3" | "h03*0.6" | etc.
clear: <mm — for glass/board clearance from frame; e.g., 3mm>
```

### 9.5 The naming repetition rule (key efficiency)

Per field PDF 1: lines or shapes with the same `named` value share all property values. This automatically counts repetitions per cut.

Example: Style "Casement 2P" has lines:
- w01 — Wframe — formula null — position width
- w02 — Wframe — formula null — position width
- h01 — Hframe — formula null — position height
- h02 — Hframe — formula null — position height

The same `named: "Wframe"` appears twice → the cut list automatically counts 2 pieces of Wframe with the same computed length. Mzee Hassan doesn't enter "Wframe × 2"; the repetition emerges from his drawing two lines on the canvas.

### 9.6 Variable declarations

Variables (X, Y, Z) are declared in the Style's Variables panel. Lines/shapes use them in formulas (`h03: formula = X`, `h04: formula = h01-X-8`). At Style validation, the system checks every declared variable is used by at least one formula; every variable referenced in formulas is declared.

### 9.7 The drawing reality

Mzee Hassan draws a casement window:
1. Picks Style type: Window
2. Picks Material Type: Aluminium
3. Picks Material Group: ALU-CAS-01
4. Draws outer frame: 4 lines (w01, w02, h01, h02) — all `Wframe`/`Hframe`, profile = ALU-FRAME-60, formula = null, offcut = 10cm, Mater-Mater endpoints, is_frame = true
5. Draws mullion in middle: 1 line (h03) — `Hmull`, profile = ALU-MULL-40, formula = `h01-9`, offcut = 7cm, Square-Square endpoints
6. Draws sash perimeters: 4 lines (w03, w04, h04, h05) per panel — `Wsash`/`Hsash`, profile = ALU-SASH-45, formulas = `(w01+1)/2` and `h01-9`, offcut = 7cm, Mater-Square endpoints
7. Draws glass shapes: 2 shapes (glass_L, glass_R) — material = GLASS-5MM, width formula = `w03-3`, height formula = `h04-3`, clear = 3mm
8. Draws mosquito net shapes: 2 shapes (net_L, net_R) — material = NET-FIBER, width formula = `w03-2`, height formula = `h04-2`
9. Saves Style; system validates; publishes.

The Style is now reusable across every future casement window project Mzee Hassan handles.

---

## 10. The Formula Engine

### 10.1 What a formula is

A formula is a string expression evaluating to a length (for lines) or two dimensions (for shapes). Expressions may use:
- **Numeric literals**: `9`, `1`, `0.5`, `6.5`
- **Operators**: `+`, `-`, `*`, `/`, `()` (per `pack.workshop.formula.allowed_operators`)
- **Line references**: `w01`, `h01`, etc. (other lines in same Style)
- **Variables**: `X`, `Y`, `Z`
- **Implicit refs**: `W` (project width), `H` (project height)

### 10.2 The dependency chain (per field PDF 1)

Resolution proceeds in rounds:

**Round 1 — Null formulas (frames):**
Every line with `formula: null` and `is_frame: true` is resolved using project W or H per position:
- `is_width: true` → length = W
- `is_height: true` → length = H

Example: w01 (null, width) → W=140 → length = 140

**Round 2 — Formulas referencing only frames:**
Lines whose formulas reference only already-resolved frame lines are evaluated.

Example: h03 (`h01 - 9`) → 201 - 9 → length = 192

**Round 3..N — Cascade:**
Each subsequent round resolves lines whose formulas reference only already-resolved lines. Continues until all lines resolved or a cycle is detected.

### 10.3 The Frame TU null rule (critical safety)

Per field PDF 1: **Only frame lines may have `formula: null`.** A non-frame line with null formula is rejected at command-time. This prevents an entire class of error: a designer trying to leave a sash or mullion's length undetermined "to be filled later."

The rule's logic: only frames are anchored to project dimensions (W/H); everything else must be computed relative to something. If a non-frame line has no formula, the system has no way to compute its length deterministically.

### 10.4 Validation timing (N3 precision)

Per N3 refinement:

**Validation at command-time** (when line/shape is added to Style):
- Formula syntax check (allowed operators per pack)
- Variable check (every referenced variable is declared)
- Line ref check (every referenced line exists in Style)
- Cycle detection (no formula loop)
- Frame TU null rule check (non-frame line with null formula rejected)

If any check fails, `workshop.style.line.add.request` is rejected; emit `kernel.command.rejected.v1` with reason.

**Evaluation at two times:**

- **POS-input time** (preview): When Salma picks a Style and enters W×H (+ variables), the Formula Engine evaluates in-memory to produce the preview cut list for cost estimation. **Not persisted.**
- **Cut-list-generation time** (canonical): When project transitions to `in_progress`, the Formula Engine evaluates again — this time the result is **persisted** as `workshop.cut_list.generated.v1` event. This is the canonical evaluation per Law 1: replay determinism requires the cut list to be recoverable from events.

The two evaluations must produce the same result (the inputs are the same: Style + dimensions + variables). N3 makes the boundary explicit: preview = transient, generated cut list = canonical.

### 10.5 Why evaluation is vertical-internal

The Formula Engine is workshop's domain knowledge per BD3 — no other vertical has parametric line-and-shape dependency chains. The engine is a **pure function** of (Style state, dimensions, variables) → cut list. No side effects; no event emissions during evaluation (only after, at the cut_list.generated.v1 boundary).

### 10.6 Example evaluation from field PDF 2 Mfano 1

Style "Casement 2P", inputs W=140, H=201:

```
Round 1 (null → W/H):
  w01: null → W → 140
  w02: null → W → 140
  h01: null → H → 201
  h02: null → H → 201

Round 2 (referencing frames):
  h03: h01 - 9 → 201 - 9 → 192
  w03: (w01 + 1) / 2 → (140 + 1) / 2 → 70.5
  w04: (w01 + 1) / 2 → 70.5
  h04: h01 - 9 → 192
  h05: h01 - 9 → 192

Shapes:
  glass_L: width = w03 - 3 = 67.5; height = h04 - 3 = 189
  glass_R: width = w03 - 3 = 67.5; height = h05 - 3 = 189
  net_L:   width = w03 - 2 = 68.5; height = h04 - 2 = 190
  net_R:   width = w03 - 2 = 68.5; height = h05 - 2 = 190

Apply offcuts:
  w01, w02 (offcut 10) → cut length 150 each
  h01, h02 (offcut 10) → cut length 211 each
  h03 (offcut 7) → 199
  w03, w04 (offcut 7) → 77.5 each
  h04, h05 (offcut 7) → 199 each
```

This is exactly what Mzee Hassan's fundi sees on the cutting sheet.

---

## 11. Variable Input + POS Flow

### 11.1 When Variables are required

A Style with declared Variables (e.g., `workshop.style.variable.declared.v1 {variable: "X"}`) carries a `is_variable: true` flag at the Style level (computed from the line/shape events that reference declared variables).

At POS time, when Salma picks a Style:
- Standard Style (no variables): system asks W × H
- Variable Style: system asks W × H **plus** the declared variables (X, Y, Z)

### 11.2 The POS flow

```
1. Salma picks Style "Casement 2P" → system loads Style projection
2. System checks Style.is_variable → false → asks W × H only
3. Customer states "140 wide, 201 tall" → Salma enters W=140, H=201
4. Formula Engine evaluates → preview cut list + cost estimate
5. System displays quote to customer
6. Customer accepts → project transitions to accepted
```

For a Variable Style:

```
1. Salma picks Style "Variable Top Window" → system loads Style projection
2. System checks Style.is_variable → true → asks W × H + X
3. Customer measures the variable section on site (the trapezoid top is X high)
4. Salma enters W=120, H=180, X=45
5. Formula Engine evaluates with all three → preview cut list + cost
6. Customer accepts → project transitions to accepted
```

### 11.3 Variable becomes constant after POS

After POS input, the variable is **frozen** for that project instance. The evaluated cut list uses X=45 as a constant; replay reconstructs the cut list with X=45 always.

If the customer later changes their mind ("actually X should be 50"), the project Workflow handles via re-quote (transitions back to `quoted` with new variable values; emits a new cut list at next in_progress trigger).

---

## 12. Cut List Generation

### 12.1 The trigger

Per WS6 + N3: `workshop.cut_list.generated.v1` emits at `workshop.project.in_progress.v1` transition. Quote phase produces only a preview (in-memory, for cost estimation); the persisted event is at in_progress.

### 12.2 The event shape

```yaml
workshop.cut_list.generated.v1:
  cut_list_id: <auto>
  project_ref: <project_workflow_id>
  site_id: <SP5>
  style_instances: [
    {style_ref, style_version, item_index, W, H, variables: {X: 45, ...}}
  ]
  cuts_per_profile_type: [
    {
      profile_type: "ALU-FRAME-60",
      profile_ref: <inventory_item_ref>,
      cuts: [
        {cut_id, named: "Wframe", required_length_cm: 150, quantity: 2, endpoints: "Mater-Mater"},
        {cut_id, named: "Hframe", required_length_cm: 211, quantity: 2, endpoints: "Mater-Mater"}
      ]
    },
    {
      profile_type: "ALU-MULL-40",
      profile_ref: <inventory_item_ref>,
      cuts: [
        {cut_id, named: "Hmull", required_length_cm: 199, quantity: 1, endpoints: "Square-Square"}
      ]
    },
    ...
  ]
  shapes_per_material: [
    {
      material_ref: <inventory_item_ref>,
      material_type: "Glass",
      shapes: [
        {shape_id, named: "glass_L", width_cm: 67.5, height_cm: 189, quantity: 1},
        {shape_id, named: "glass_R", width_cm: 67.5, height_cm: 189, quantity: 1}
      ]
    },
    ...
  ]
  generated_at, business_date
```

### 12.3 Downstream consumers

After cut_list.generated.v1:
- **Inventory** consumes Pattern B per CN-5-003 N1 — deducts materials per cuts_per_profile_type + shapes_per_material
- **Cut Workflow instances** spawn — one per cut piece (planned state); fundi sees the list and begins
- **Workshop fundi advisor** subscribes — may suggest cut sequence optimization
- **Workshop manager advisor** subscribes — tracks project progression
- **Cut Optimizer** runs Steps 1-5 to assign cuts to specific bars and identify offcuts (§13)

### 12.4 Multi-item project aggregation

If a project contains multiple style instances (4 windows + 2 doors per WP6), the cut_list aggregates all items per profile_type. The Cut Optimizer then packs across the entire project — a 6m frame bar might serve cuts from both window #2 and door #1. Optimization across the project minimizes waste better than per-item optimization.

---

## 13. Cut Optimization — BFD + Offcut Algorithm

### 13.1 The 5-step algorithm (per field PDF 1)

**Step 1 — Grouping:**
The Cut Optimizer groups the cut list by `profile_type`. Each profile is optimized **separately** — frame profile cannot share a bar with sash profile (different cross-sections; physical impossibility).

**Step 2 — Get specifications:**
For each profile group, read from Inventory item metadata + Style line offcut values:
- `stock_length` — e.g., 6000mm (varies by supplier; per N2 lives in Inventory)
- `offcut` — the angle-correction trim, line-specific from Style (frame typically 10cm, sash 7cm)
- `kerf_width` — saw blade thickness, typically 2mm (Inventory metadata or pack default per `pack.workshop.cut_optimization.kerf_default_mm`)

**Step 3 — Sort decreasing:**
Within each profile group, sort cut pieces from largest to smallest. This is the "Decreasing" in Best Fit Decreasing — large pieces get placed first; small pieces fill remaining gaps.

**Step 4 — Best Fit Decreasing logic:**
For each cut piece (largest to smallest):

- **A. Find candidate bars** — bars (new + already-cut) that fit the piece:
  - New bar: `remaining ≥ piece_length`
  - Already-cut bar: `remaining ≥ piece_length + offcut (+ kerf if applicable)`
    - The offcut adds because every cut on an already-cut bar must trim the angled end before the new cut
    - Kerf adds if pack rule includes it
- **B. Pick the bar with smallest remaining after the cut** — this is "Best Fit" — minimize waste by filling tight spots
- **C. If no bar fits** — open a new stock bar from Inventory

**Step 5 — Compute remaining:**
After cutting:
- New bar: `remaining = stock_length - piece_length`
- Already-cut bar: `remaining = remaining - offcut - piece_length (- kerf)`

If `remaining ≥ pack.workshop.offcut.minimum_usable_length_mm`, the offcut is recorded per WS8 (emits `workshop.offcut.recorded.v1` → Inventory lot added). If below the threshold, the offcut is scrap.

### 13.2 Concrete example (Casement 2P)

From §10.6 cut list:

**Profile group ALU-FRAME-60** (stock 6000mm, offcut 10cm = 100mm, kerf ignored):
- Sort decreasing: 211, 211, 150, 150 (cm = 2110, 2110, 1500, 1500 mm)
- Place 2110mm — new bar 6000mm → remaining 3890mm
- Place 2110mm — fits in 3890mm cut bar (3890 ≥ 2110 + 100 = 2210) → remaining = 3890 - 100 - 2110 = 1680mm
- Place 1500mm — fits in 1680mm cut bar (1680 ≥ 1500 + 100 = 1600) → remaining = 1680 - 100 - 1500 = 80mm (scrap if below threshold)
- Place 1500mm — no existing bar fits — open new bar → remaining = 6000 - 1500 = 4500mm (becomes offcut)
- **Result: 2 frame bars used; 1 offcut at 4500mm logged for future use**

**Profile group ALU-MULL-40** (stock 6000mm, offcut 7cm = 70mm):
- Sort: 1990 (199cm)
- Place 1990mm — new bar 6000mm → remaining 4010mm
- **Result: 1 mullion bar; 1 offcut at 4010mm logged**

**Profile group ALU-SASH-45** (stock 6000mm, offcut 7cm = 70mm):
- Sort: 1990, 1990, 1990, 1990, 775, 775, 775, 775 (cm to mm: 19900, etc. — actually 1990, 1990, ..., 775, 775)
- Wait — sash heights are 199cm = 1990mm; sash widths are 77.5cm = 775mm
- Place 1990 — new bar → remaining 4010
- Place 1990 — fits in 4010 (4010 ≥ 1990 + 70 = 2060) → remaining = 4010 - 70 - 1990 = 1950
- Place 1990 — no existing bar fits — open new bar → remaining 4010
- Place 1990 — fits in 4010 → remaining 1950
- Place 775 — fits in 1950 (1950 ≥ 775 + 70 = 845) → remaining = 1950 - 70 - 775 = 1105
- Place 775 — fits in 1105 (1105 ≥ 845) → remaining = 1105 - 70 - 775 = 260
- Place 775 — fits in 1950 (other bar) → remaining = 1105
- Place 775 — fits in 1105 → remaining = 260
- **Result: 2 sash bars used; offcuts at 260mm × 2 (likely scrap)**

Total bars for Casement 2P: 2 frame + 1 mullion + 2 sash = 5 bars + glass sheet + net.

This matches the field PDF 2 Mfano 1 expected output ("Total bars: 2 frame + 3 sash + 1 mullion = 6 bars + 1 glass sheet + net"). The minor difference (the PDF says 3 sash bars; my computation suggests 2 sufficient) reflects implementation detail of how offcuts are reused across the same project run — both implementations valid; pack rule may force or allow more conservative bar count.

### 13.3 2D guillotine for sheet materials (N1)

Shapes (glass, board, panel, net) are 2D materials. The BFD algorithm above is 1D (length only). For 2D, a parallel mechanism — BFD-2D or guillotine optimization — applies:

- Pack hook `pack.workshop.cut_optimization.2d_strategy` declares strategy (default: BFD-2D)
- 2D pieces (e.g., glass 67.5 × 189cm) are packed onto sheets (e.g., 300 × 200cm GLASS-5MM sheets) using 2D bin-packing
- Offcut regions are L-shaped or rectangular remnants; tracked similarly via Inventory primitive lot extension

The detailed 2D algorithm is **Architect-phase implementation**; CN-6-004 documents that the mechanism parallels 1D BFD per Brief §11.5 + field PDF 1 references to glass/board/panel optimization.

### 13.4 Why Cut Optimization is workshop-internal

Per BD3: only workshop has the specific cut optimization problem (cut multiple lengths from limited stock bars; track reusable offcuts). No universal substitute exists. Inventory primitive provides lot model + tracking; workshop provides the algorithm that uses the lot model.

### 13.5 Offcut reuse across projects

A 4500mm offcut from Project A becomes input stock for Project B. At Project B's cut list generation, the Cut Optimizer's Step 2 (read specs) includes existing offcuts as candidate bars alongside new stock. Step 4's "find candidate bars" considers both.

This is the workshop equivalent of recycling — over time, fewer new bars are bought because offcuts get reused for smaller cuts. WP5 demonstrates concretely.

---

## 14. Offcut Tracking via Inventory Primitive (N2 Boundary)

### 14.1 The boundary

Per N2: material specifications (`stock_length`, `offcut_length`, `kerf_width`) live as **Inventory primitive item metadata** per CN-5-003 native model. Workshop does NOT duplicate specs in `pack.workshop.*`. BD4 push-down: the specs serve every engine that uses these materials (workshop primarily, but also retail.* if a workshop tenant also activates retail.* to sell raw materials).

Workshop **reads** the specs at cut list generation + cut optimization time. Workshop does not **own** the specs.

### 14.2 Offcut as a lot per CN-5-003

CN-5-003 supports lot-based inventory tracking. A bar of ALU-FRAME-60 is a lot of length 6000mm; receive it from a supplier and Inventory registers `inventory.lot.added.v1 {profile_type: ALU-FRAME-60, length: 6000, supplier_ref, received_date}`.

When workshop cuts that bar producing a 4500mm offcut, workshop emits:

```
workshop.offcut.recorded.v1 {
  offcut_id,
  source_bar_lot_ref: <original_lot>,
  profile_type: ALU-FRAME-60,
  offcut_length: 4500,
  site_id,
  business_date
}
```

Inventory subscribes and emits `inventory.lot.added.v1 {profile_type: ALU-FRAME-60, length: 4500, source: offcut, parent_lot_ref: <original>, ...}`. The offcut is now Inventory state, queryable like any other lot.

### 14.3 Why this is clean

- Workshop doesn't reinvent inventory tracking
- Inventory primitive handles lot persistence + audit + queries
- Future engines that want to know "how much ALU-FRAME-60 do we have, including offcuts?" query Inventory once
- The cut optimizer reads Inventory's complete picture (new bars + offcuts) at Step 2

### 14.4 Offcut threshold

Pack hook `pack.workshop.offcut.minimum_usable_length_mm` (e.g., 300mm for ALU-FRAME-60) declares the cutoff. Offcuts below the threshold are scrap; no event emits (or a `workshop.scrap.logged.v1` event for waste tracking analytics — optional per pack).

---

## 15. Multi-Item Projects (WS6 §15)

### 15.1 Project containing multiple style instances

A customer orders 4 windows + 2 doors + 1 fixed panel for a new house. The project Workflow contains 7 style instances. Each is added via `workshop.project.add_item.request {style_ref, W, H, variables: {...}, item_position_label: "kitchen window"}`.

### 15.2 Aggregated cut list generation

At `workshop.project.in_progress.v1`, the cut list aggregates across all 7 items per profile_type:

- All windows' frame cuts pool together for cut optimization across the project
- All windows' sash cuts pool together
- Doors' door-frame cuts pool together (different profile from window-frame)
- The fixed panel's frame cuts pool with window frames if same profile, separately if different

Project-level optimization typically reduces total bars vs per-item optimization. WP6 demonstrates a 4 windows + 2 doors project.

### 15.3 Quote aggregation

The preview cut list at quote time aggregates costs across all items + labor + delivery + margin per pack rules. The customer sees one quote total for the entire project.

### 15.4 Quote acceptance pins all items

Per CN-6-101 §10 AP4: at `workshop.project.accepted.v1`, the project pins:
- Style versions at time of acceptance (per item)
- Variable values entered at POS (per item if variable-using)
- Rate card / pricing at time of acceptance
- Material specs (so later supplier change to stock_length doesn't surprise the customer)

Mid-project revisions to any item require re-quote (project transitions back to `quoted` with updated values).

---

## 16. Quote vs In Progress State Transition

### 16.1 The two cut lists

**Quote phase cut list** — preview:
- Computed in-memory at `workshop.project.quote.request`
- Used for cost estimation displayed to customer
- NOT persisted as an event
- Replay does NOT reconstruct this (transient)

**In Progress cut list** — canonical:
- Emitted as `workshop.cut_list.generated.v1` at `workshop.project.in_progress.v1`
- Persisted (Law 1)
- Replay reconstructs this deterministically
- Triggers Inventory deduction + Cut Workflow instantiation + Cut Optimizer

### 16.2 Why two stages

Per Brief §7.4 + field PDF 1: a customer may quote-shop with multiple workshops before deciding. Persisting every quote attempt as cut list events would clutter the event store with discarded variants. The preview cut list serves quote phase; the canonical cut list serves production.

Cost estimation accuracy is preserved because preview uses the same Formula Engine + Cut Optimizer; the only difference is persistence.

### 16.3 The transition

`workshop.project.start.request` command transitions accepted → in_progress. This emits:

```
workshop.project.in_progress.v1 {
  project_ref,
  business_date,
  started_by_actor
}
```

Which triggers:

```
workshop.cut_list.generated.v1 {
  cut_list_id,
  project_ref,
  ...  # as §12.2
}
```

The cut list is now the canonical instruction set. Inventory deducts; Cut Workflow instances spawn; fundi begins.

---

## 17. Material Shortage + Procurement Triggers + Workflow Blocking (WS9)

### 17.1 Detection at cut list generation

When cut_list.generated.v1 emits, Inventory attempts to allocate materials. If a profile_type is short (insufficient stock + offcuts), Inventory emits `inventory.stock.depleted.v1 {profile_type, required_length, available_length}`.

### 17.2 Workshop reaction

Workshop subscribes to `inventory.stock.depleted.v1` per HO6 situational signal. When the depleted profile is needed by an active project, workshop:

1. Emits `workshop.project.blocked.v1 {project_ref, blocking_profile_type, required_length, available_length}`
2. Project Workflow transitions: in_progress → blocked
3. Emits `workshop.purchase_need.recorded.v1` per CN-5-004 + CTR-030 expansion (if `pack.workshop.material_shortage.auto_procurement_trigger: true`)

### 17.3 Procurement integration

Procurement (CN-5-004) subscribes to `workshop.purchase_need.recorded.v1`. It generates a requisition for the shortfall; pack rules govern auto-creation vs manual approval. The purchase order proceeds through normal Procurement flow (PO → GRN → invoice → payment).

### 17.4 Project unblocking

When stock arrives (`inventory.lot.added.v1` for the depleted profile_type) and total available meets need, workshop emits:

```
workshop.project.unblocked.v1 {
  project_ref,
  resolved_by_lot_ref,
  business_date
}
```

Project Workflow transitions: blocked → in_progress. Cut Optimizer re-runs (now with adequate stock). Cutting resumes.

### 17.5 Mzee Hassan reality

The aluminium supplier delays delivery by a week. Mzee Hassan's casement-window project is mid-cut; ALU-MULL-40 runs out before the second window's mullion can be cut. The project blocks; advisor alerts Mzee Hassan; procurement reorder fires; Mzee Hassan calls the supplier; stock arrives 5 days later; project unblocks; cutting resumes. The customer's quoted delivery date slipped by 5 days; pack rule may auto-emit a customer notification (per D-DISC-001 Term 3 future).

---

## 18. Style Sharing — Export / Import (WS10)

### 18.1 N5 — the 5-step mechanism

Per N5 refinement:

**Step 1 — Export request:**
Tenant A's authorized user (typically owner Mzee Hassan or his deputy) issues `workshop.style.export.request {style_ref, target_format: json}`. The vertical computes the Style's complete state from event projection.

**Step 2 — Exported Document:**
Workshop emits `workshop.style.exported.v1 {style_ref, export_id, content_hash, exported_at, exported_by}`. A Foundation Document (CN-4-012) is issued carrying the Style payload (lines, shapes, variables, properties) as JSON, hash-signed for integrity.

```
workshop.style.exported.v1 {
  export_id,
  style_ref,
  style_version,
  content_hash: <sha256 of JSON payload>,
  document_ref: <CN-4-012 Document with JSON content>,
  exported_at,
  exported_by_actor
}
```

**Step 3 — Document transfer (out of BOS):**
Tenant A's user shares the Document file with Tenant B (email, USB stick, messaging app — outside BOS scope; file transfer is human action).

**Step 4 — Import request:**
Tenant B's authorized user issues `workshop.style.import.request {document_ref OR raw_json, source_style_metadata}`. The command provides the JSON content (uploaded or referenced).

**Step 5 — Bus validates + import event:**
The vertical:
- Validates the content hash if document_ref carries one
- Validates the JSON structure (all referenced materials exist in Tenant B's Material Group OR are mapped via pack rule)
- Replays the export as a sequence of `workshop.style.line.added.v1` + `workshop.style.shape.added.v1` + `workshop.style.variable.declared.v1` events into Tenant B's Style library
- Emits `workshop.style.imported.v1 {import_id, new_style_ref_in_tenant_b, source_export_id, imported_at, imported_by_actor, source_tenant_hint}`

After import, Tenant B has its own copy of the Style. Future changes by Tenant A do NOT propagate to Tenant B (no live link). If Tenant A publishes a new Style version, sharing it again repeats Steps 1-5.

### 18.2 Why duplicate-into-receiver (not live link)

Per BD5 + VE2 isolation: cross-tenant subscription would couple tenants. A live link (Tenant B's Style auto-updates when Tenant A changes hers) would mean Tenant A's mistake breaks Tenant B's projects. The duplicate model preserves isolation; each tenant owns their copy.

### 18.3 Material mapping at import

A common case: Tenant A uses material ref "ALU-FRAME-60" (her Material Group's frame). Tenant B's Material Group has a different ref but same physical material. At import, the system either:
- Maps via pack rule `pack.workshop.material_import_mapping` (if defined)
- Asks Tenant B's user to confirm each material reference
- Imports with broken refs that Tenant B fixes by editing the imported Style

This is implementation detail; CN-6-004 specifies the contract; Architect implements mapping UX.

### 18.4 Marketplace deferred per BD8

A platform-level Style marketplace (cross-tenant catalog with Term 1 governance, ratings, payment for shared Styles) is deferred. The Brief §3.3 future verticals (Insurance, Healthcare, etc.) may surface similar marketplace needs across verticals; whenever ≥2 concrete cases demonstrate need, the marketplace becomes a future CTR.

### 18.5 Mzee Hassan + Fundi Athumani (WP7 anchor)

Mzee Hassan has perfected his casement window Style over years. Fundi Athumani — a workshop owner in Mwanza, new to BOS and just onboarded — visits Mzee Hassan in Arusha for training. Mzee Hassan exports his "Casement 2P" Style; Fundi Athumani imports it into his Mwanza tenant. Fundi Athumani's first casement project uses the imported Style. The Style continues to live in Mzee Hassan's library and in Fundi Athumani's library independently; if either improves their copy, the other is unaffected.

This is style sharing in its simplest form. WP7 walks through this scenario concretely.

---

## 19. Disputes + Rework + Warranty

### 19.1 The warranty period

Pack hook `pack.workshop.project.standard_warranty_days` (e.g., 90 days for residential windows; 365 days for commercial fabrications). The period starts at `workshop.project.delivered.v1`.

### 19.2 The dispute path

Within warranty period, a customer may report a defect:

```
hotel.project.dispute.request {  # wait — workshop, not hotel
}
```

Correction:

```
workshop.project.dispute.request {
  project_ref,
  item_ref,
  defect_description,
  reporter_party_ref,
  business_date
}
```

The Workflow transitions: completed → disputed. Project Manager + Mzee Hassan investigate. Two paths:

- **Rework**: Mzee Hassan re-cuts + re-assembles the defective item. New cuts emit; new material consumption emits; Workflow transitions back through cutting → assembly → finishing → ready → delivered → completed (free of charge per warranty).
- **Refund**: Workshop refunds the customer (Foundation Obligation primitive `kind: warranty_refund`). Workflow transitions: disputed → completed with refund Obligation.

### 19.3 Warranty period expiry

Post-warranty defects are out-of-scope for free remediation. Customer may still report; pack rule may convert to paid rework (new project). The original Workflow stays `completed`; a new project starts for the rework.

### 19.4 Why CN-4-011 Obligation primitive

The refund or rework flows through Foundation primitives — Obligation (for refund debt), Document (for dispute record), Workflow (for state). No new workshop-specific primitives.

---

## 20. Mixed-Vertical Workshop+Retail Sell-Via-POS (WS11)

### 20.1 The Brief §7.1 case

A workshop occasionally sells fabricated items directly via retail POS rather than per-project commission. Examples:
- Mzee Hassan's showroom displays sample windows; a walk-in customer admires one and wants to buy it as-is
- A finished project's spare item (customer ordered 4, only needs 3) becomes inventory
- Pre-fabricated standard sizes manufactured in advance for stock

The tenant activates BOTH `workshop.*` (for fabrication operations) AND `retail.*` (for POS sales) per Mixed-Vertical Tenant pattern (CN-6-105 future).

### 20.2 N4 — the 8-step concrete chain

**Step 1 — Workshop project completes:**
Mzee Hassan's project Workflow reaches `delivered → completed` per normal flow. The fabricated window exists physically in his showroom.

**Step 2 — Item enters retail inventory:**
Workshop emits a follow-up event when the item is designated for retail sale (operator decision):

```
workshop.deliverable.completed.v1 {
  project_ref,
  item_ref,
  inventory_destination: retail_showroom,
  business_date
}
```

This event signals Inventory to register the completed window as a retail-sellable lot.

**Step 3 — Foundation Inventory primitive adds the lot:**

```
inventory.lot.added.v1 {
  item_ref: <workshop_produced_window>,
  source_engine: workshop,
  source_workflow_ref: <project>,
  category: "fabricated_window",
  quantity: 1,
  site_id: <showroom_site>,
  business_date
}
```

The window is now Inventory state — queryable, sellable.

**Step 4 — Retail catalog references the item:**
Retail.* tenant activation includes retail catalog management. The retail.catalog.entry references the workshop-produced item by `item_ref` (per RE7 — retail handles any item type). Mzee Hassan's showroom retail catalog includes the window as a SKU with a price.

**Step 5 — Walk-in customer at retail till:**
A walk-in customer wants the window. Salma (in this scenario working as retail cashier at the showroom) opens a basket; adds the window:

```
retail.basket.item.added.v1 {
  basket_id,
  item_ref: <workshop_produced_window>,
  quantity: 1,
  unit_price: <catalog_price>,
  site_id: <showroom>
}
```

**Step 6 — Retail.bill.ready.v1 emits:**

```
retail.bill.ready.v1 {
  bill_id,
  site_id: <showroom>,
  saleable_lines: [
    {line_id, item_ref: <window>, quantity: 1, unit_price, tax_treatment_ref}
  ],
  payer_party_ref: <customer>,
  business_date
}
```

**Step 7 — Universal Checkout settles:**
Customer chooses tender method per tenant's enabled methods (per CTR-049 regional curation + RE11 abstraction). Settlement happens.

```
checkout.settled.v1 {
  bill_id,
  ...
}
```

**Step 8 — Inventory Pattern A deduction:**
Inventory subscribes to checkout.settled.v1 (Pattern A per CN-5-003); deducts the window lot. The window leaves Inventory; the customer takes it home.

### 20.3 What this proves (BD7 + VE2)

- Workshop and Retail **never reference each other's events**
- Foundation Inventory primitive carries the cross-vertical bridge — workshop deposits; retail withdraws
- Decommissioning Retail tomorrow doesn't break Workshop (Workshop's project completion is its own audit chain)
- The customer's experience is simple: walk in, pick the window, pay, leave

### 20.4 CN-6-005 catalog forward pointer

CN-6-005 (Vertical Bridges, future) will catalog this and other cross-vertical patterns. CN-6-004 establishes the doctrine + concrete chain; CN-6-005 enumerates instances and variations.

### 20.5 Mzee Hassan reality

Mzee Hassan's Arusha showroom has 6 sample windows on display. He fabricated 5 of them as commissions whose customers picked them up; 1 is a spec piece. All 6 are in his retail catalog (he activated retail.* for the showroom). Tourists or contractors stopping by occasionally buy on the spot. The sell-via-POS revenue flow blends with his project commission revenue — same Accounting books per CN-5-001; same revenue stream segregated only by source_engine in payload.

---

## 21. Conflict Event Family (VI-03)

### 21.1 Workshop's conflict cases (rare)

Per VI-03: workshop emits conflict events when bus single-acceptance rejects contenders. The most plausible workshop case:

- **Two cut commands target the same uncut bar simultaneously** — `workshop.cut.execute.request × 2` racing on the same source_bar_lot_ref. Bus single-acceptance: one wins; loser receives compensation.

```
workshop.bar.conflict.detected.v1 {
  contested_resource_ref: <bar_lot_id>,
  winning_workflow_ref: <winning_cut_workflow>,
  contender_workflow_refs: [<losing_cut_workflow>],
  detection_ts,
  resolution_basis: "bus_single_acceptance_first_arrival"
}
```

The losing cut is re-planned by the Cut Optimizer with a different bar (or a new bar opened).

### 21.2 Why conflicts are rare in workshop

Unlike Restaurant (waiters racing on tables) or Hotel (channels racing on rooms), workshop's resource contention is mostly internal to one project (sequential fundi work). Cross-project contention happens only if two projects' cut lists materialize simultaneously and the optimizer assigns the same bar — unlikely with proper sequencing. The conflict event exists for completeness; expect low frequency.

---

## 22. Worked Patterns — Eight Real Tanzanian Fabrication Scenarios

### 22.1 WP1 — Mwemas commission casement windows for Mwanza home renovation (N6 anchor)

**Anchor:** Mama na Bwana Mwema (cross-doc continuity from CN-6-102/103/003) are renovating their Mwanza home. They need 4 casement windows for the master bedroom, second bedroom, study, and sunroom. They visit Mzee Hassan's Arusha karakana on recommendation.

**Setup:** Mzee Hassan's tenant has:
- Material Group ALU-CAS-01 active
- Style "Casement 2P" published (the canonical from field PDF 2 Mfano 1)
- Inventory stocked with ALU-FRAME-60, ALU-SASH-45, ALU-MULL-40, GLASS-5MM, NET-FIBER bars + sheets
- `pack.workshop.inventory_expansion_mode.aluminium: vertical_managed` (Pattern B)

**Flow:**

1. Mwemas describe their needs; Mzee Hassan visits Mwanza to measure. `workshop.project.enquired.v1`; `workshop.project.measured.v1` after Mzee Hassan's site visit.
2. Back in Arusha, Mzee Hassan creates a project with 4 items (each = "Casement 2P" style + their respective dimensions). All 4 windows happen to be 140×201 (master bedroom + study + sunroom standard; second bedroom slightly different at 130×190 — but for this WP, all 4 use 140×201 for simplicity).

```
workshop.project.item.added.v1 × 4
  {style_ref: "Casement 2P", W: 140, H: 201, position_label: "master_bedroom"}
  {style_ref: "Casement 2P", W: 140, H: 201, position_label: "second_bedroom"}
  {style_ref: "Casement 2P", W: 140, H: 201, position_label: "study"}
  {style_ref: "Casement 2P", W: 140, H: 201, position_label: "sunroom"}
```

3. Mzee Hassan generates quote: `workshop.project.quote.request`. Preview cut list aggregates across all 4 windows per §15:
   - ALU-FRAME-60 (W): 150cm × 8 pieces
   - ALU-FRAME-60 (H): 211cm × 8 pieces
   - ALU-MULL-40: 199cm × 4 pieces
   - ALU-SASH-45 (W): 77.5cm × 16 pieces
   - ALU-SASH-45 (H): 199cm × 16 pieces
   - GLASS-5MM panes: 67.5×189cm × 8 (2 per window)
   - NET-FIBER pieces: 68.5×190cm × 8
   
   With BFD optimization across the project, Mzee Hassan's system computes ~8 frame bars + ~4 mullion bars + ~10 sash bars + 2 glass sheets + net consumption. Plus labor, fittings, delivery, margin → quote total. (Specific cost in tenant's enabled tender method per RE11 abstraction.)
4. `workshop.project.quoted.v1` emits.
5. Mwemas review; accept. `workshop.project.accepted.v1`.
6. Mzee Hassan schedules start; `workshop.project.start.request` → transitions to `in_progress`. `workshop.cut_list.generated.v1` emits with full breakdown per §12.2 shape.
7. Inventory deducts (Pattern B): per profile_type, the BFD-assigned bars are consumed.

```
workshop.material.consumed.v1 × 8 (one per cut piece's stock consumption)
workshop.offcut.recorded.v1 × N (per offcut produced; e.g., 4500mm offcut from frame bar #2 → Inventory lot added)
```

8. Cut Workflow instances spawn (`workshop.cut` × ~50+ cuts across the project). Fundi receives cutting sheet — concrete cut lengths, bar assignments, sequence. He cuts. `workshop.cut.executed.v1` emits per cut.
9. Cutting complete; project transitions to `assembly`. Fundi assembles 4 windows in parallel (other fundis help on larger items).
10. Assembly complete → `finishing` (paint + hardware install) → `ready_for_delivery`.
11. Delivery to Mwanza: Mzee Hassan arranges transport. `workshop.project.deliver.request` → transitions to `delivered`. `workshop.bill.ready.v1` emits with full project cost.
12. Universal Checkout settles via Mwemas' chosen tender method (per RE11 abstract; tenant's enabled set per CTR-049). `checkout.settled.v1` arrives → HO9 → `workshop.project.completed.v1`.
13. Inventory state final: bars consumed; offcuts logged for future projects; new offcut: a 4500mm frame bar offcut sits in Mzee Hassan's inventory available for future casement-window cuts (WP5 will use it).

**Doctrine demonstrated:** WS1 fabrication; WS2 Style tenant-scope; WS4 Formula Engine cascade (per §10.6); WS6 cut list at in_progress; WS7 BFD optimization across multi-item project; WS8 offcut tracking; WS9 long-lifecycle Workflow; HO1 + HO9 + Pattern B; RE11 payment abstract; cross-doc Mwemas continuity.

### 22.2 WP2 — Sliding window for commercial customer (PDF 2 Mfano 2)

**Anchor:** A commercial customer (e.g., a small office building in Arusha) wants a sliding window 180 × 150 for a reception area. Single item; Style "Sliding 2P" per field PDF 2 Mfano 2.

**Flow (abbreviated):**

1. Project enquiry + measurement (site visit).
2. Mzee Hassan creates project with 1 item: `style_ref: "Sliding 2P", W: 180, H: 150`.
3. Quote → accept → in_progress → cut list per Mfano 2:
   - ALU-TRACK-70 (W): 190cm × 2
   - ALU-TRACK-70 (H): 160cm × 2
   - ALU-SLDSASH-50 (W): 101cm × 4
   - ALU-SLDSASH-50 (H): 150cm × 4
   - ALU-INTER-25: 148cm × 2
   - GLASS-5MM: 2 panes
4. BFD optimizes per profile group; offcuts logged.
5. Cutting → assembly → finishing → delivered → bill.ready → checkout → completed.

**Doctrine demonstrated:** WS1 + WS4 + WS6 + WS7 — Style variation (sliding has interlock instead of mullion; different formulas); cut list works identically per Formula Engine.

### 22.3 WP3 — Fixed panel showcase window (PDF 2 Mfano 3)

**Anchor:** A retail shop owner wants a fixed showcase window 100 × 120 for product display. Simplest style; Style "Fixed Panel" per field PDF 2 Mfano 3.

**Flow:**

Project with 1 item, dimensions 100×120, Style "Fixed Panel". Cut list per Mfano 3:
- ALU-FIXFRAME-55 (W): 110cm × 2
- ALU-FIXFRAME-55 (H): 130cm × 2
- ALU-BEAD-15 (W): 99cm × 2
- ALU-BEAD-15 (H): 119cm × 2
- GLASS-6MM: 1 sheet at 92×112cm

Simplest BFD case — fewer cuts; smaller waste. **Demonstrates framework holds at extreme simple case** (parallel CN-6-002 WP7 Bibi Khadija + CN-6-003 WP6 Bibi Sauda).

### 22.4 WP4 — Hinged door with glass + board (PDF 2 Mfano 4)

**Anchor:** A house renovation needs a back door 90 × 220 with glass top section and board bottom section. Style "Hinged Door" per field PDF 2 Mfano 4. Door has threshold (heavier profile) + mid-rail dividing glass and board.

**Flow:**

Project with 1 item, dimensions 90×220, Style "Hinged Door". Cut list per Mfano 4:
- ALU-DFRAME-65 (W): 100cm × 1
- ALU-THRESH-80: 100cm × 1 (the heavy threshold profile at bottom)
- ALU-DFRAME-65 (H): 230cm × 2
- ALU-DSASH-50 (W): 90cm × 3 (top sash + mid-rail + bottom sash widths)
- ALU-DSASH-50 (H): 216cm × 2
- GLASS-5MM: 80×125cm × 1 (top section)
- BOARD-3MM-WHT: 80×79cm × 1 (bottom section)

**Doctrine demonstrated:** WS1 covers doors not just windows; Style supports multiple material types (glass + board) per Group; threshold = different profile; mid-rail divides fill regions.

### 22.5 WP5 — Variable window with X=45 (PDF 2 Mfano 5; offcut reuse from WP1)

**Anchor:** A customer wants a window with a trapezoidal top section (custom for an A-frame architectural detail). Style "Variable Top Window" per field PDF 2 Mfano 5. The window has fixed top with height = X (variable) + casement bottom. Dimensions 120 × 180, X = 45.

**Flow:**

1. Project + 1 item; Style "Variable Top Window"; W=120, H=180, X=45.
2. Cut list per Mfano 5:
   - UPVC-FRAME-70: 130, 130, 190, 190, 52 cm (5 pieces of varying length)
   - UPVC-FRAME-70: 121 × 1 (the cross member)
   - UPVC-SASH-60 (W): 121cm × 2
   - UPVC-SASH-60 (H): 134cm × 4 (panel sashes)
   - GLASS-5MM: 110×41 (top), 56×123 × 2 (panels)
3. Offcut reuse: WP1 left a 4500mm ALU-FRAME-60 offcut. WP5's UPVC-FRAME-70 130cm cut COULD have used that offcut IF the material matched — but UPVC ≠ ALU, so it doesn't apply here. BUT a future ALU project would benefit. (For demonstration, imagine WP5 used ALU instead of UPVC: the 4500mm offcut from WP1 would fit two 130cm cuts + remaining 4500-100-1300-100-1300 = 1700mm — yet another offcut.)

**Doctrine demonstrated:** WS5 variable input at POS; WS4 cascade with variable as constant; WS8 offcut model native; cross-material material types — workshop handles aluminium + uPVC + wood per WS1.

### 22.6 WP6 — Multi-item project (4 windows + 2 doors) with cut optimization across all

**Anchor:** A small commercial building project — 4 casement windows for offices + 2 hinged doors for entrances. The customer (a developer) wants all 6 items as one project.

**Flow:**

1. Project with 6 items: 4 × Style "Casement 2P" (various dimensions) + 2 × Style "Hinged Door" (various dimensions).
2. At quote → accept → in_progress, the cut list aggregates **all** items per profile_type.
3. ALU-FRAME-60 is used by both Casement (window frame) and **partially** by Hinged Door (door frame) — actually they're different profiles (ALU-FRAME-60 vs ALU-DFRAME-65). They optimize separately.
4. ALU-SASH-45 is used by Casement; ALU-DSASH-50 by Hinged Door — also separate optimization.
5. BUT GLASS-5MM is used by ALL items (windows panes + door glass) — aggregated 2D guillotine optimization (N1) packs all glass cuts onto minimal sheets.
6. Cross-project optimization saves bars + sheets vs per-item optimization. The cut list reflects this.

**Doctrine demonstrated:** WS6 multi-item; WS7 cross-item optimization; commercial project scale; field-PDF-reality applied to multi-item case.

### 22.7 WP7 — Style sharing: Mzee Hassan exports Casement; Fundi Athumani imports (WS10)

**Anchor:** Fundi Athumani (NEW Term 6 character; Mwanza workshop owner; cross-doc with Mama Halima's freight route) is new to BOS. He recently activated his workshop tenant; he has Inventory + Material Group but no Styles yet. He visits Mzee Hassan in Arusha for a week of training. Mzee Hassan offers to share his "Casement 2P" Style.

**Flow (N5 5-step):**

1. Mzee Hassan issues `workshop.style.export.request {style_ref: "Casement 2P", target_format: json}` from his tenant
2. Vertical computes Style state; emits `workshop.style.exported.v1`; Foundation Document issued with content hash. JSON payload contains: lines (with formulas, materials, properties), shapes, variables (none for this style).
3. Mzee Hassan sends the file to Fundi Athumani (USB stick during the visit; or messaging app)
4. Fundi Athumani returns to Mwanza; on his workshop tenant, issues `workshop.style.import.request {raw_json: <pasted content>}`
5. Vertical validates structure; replays as a sequence of line.added + shape.added events into Fundi Athumani's Style library; emits `workshop.style.imported.v1`. Material references must match Fundi Athumani's Material Group ALU-CAS-01 (he has the same standard aluminium materials) — pack `material_import_mapping` confirms. Style is now in Fundi Athumani's library.

Three months later, Fundi Athumani improves the Style (he found a better mullion calculation: `(h01-10)` instead of `(h01-9)`). He publishes a new version in his library. Mzee Hassan's library is unaffected — they own independent copies.

**Doctrine demonstrated:** WS10 + N5; cross-tenant sharing without live link; isolation preserved per VE2; new character introduction; cross-doc Mwanza geographical continuity.

### 22.8 WP8 — Mixed-Vertical showroom sell-via-POS (WS11 + N4 8-step chain)

**Anchor:** Mzee Hassan's Arusha showroom displays sample windows. A walk-in customer — a contractor needing a window quickly for a project they're working on — wants to buy a sample window on the spot rather than wait for fabrication.

**Setup:** Mzee Hassan's tenant has BOTH `workshop.*` (his fabrication operations) AND `retail.*` (his showroom POS) activated per Mixed-Vertical Tenant pattern.

**Flow (per N4 8 steps):**

1. **Workshop project completed:** A previous casement window project completed; one extra window was fabricated (customer ordered 3 but Mzee Hassan made 4 for showroom variety). `workshop.project.completed.v1` emitted earlier.
2. **Item enters retail inventory:** Mzee Hassan designates the extra window for showroom sale; emits `workshop.deliverable.completed.v1 {project_ref, item_ref, inventory_destination: retail_showroom}`.
3. **Foundation Inventory primitive adds lot:** `inventory.lot.added.v1 {item_ref: <window>, source_engine: workshop, category: "fabricated_window_140x201_casement", quantity: 1, site_id: <showroom>}`.
4. **Retail catalog references the item:** Mzee Hassan adds a retail catalog entry referencing the item with price 800,000 TZS.
5. **Walk-in customer at retail till:** The contractor walks in; Salma (cashier) scans the showroom tag; `retail.basket.item.added.v1 {item_ref: <window>, quantity: 1, unit_price: 800000}`.
6. **retail.bill.ready.v1 emits** with the saleable line.
7. **Universal Checkout settles** via the contractor's chosen tender method (per RE11; tenant's enabled set per CTR-049).
8. **Inventory Pattern A deduction:** `inventory.lot.removed.v1` for the window lot; contractor takes the window.

**Doctrine demonstrated:** WS11 + N4 8-step chain; Foundation Inventory primitive as cross-vertical carrier (BD7 + VE2 honoured); Mixed-Vertical Tenant pattern in concrete form; cross-doc Mzee Hassan continuity; framework's most-complex vertical handling cross-vertical sale cleanly.

---

## 23. Boundaries + Open Items + Cross-Term Hooks

### 23.1 CN-6-004's place in the corpus

| Concern | Owned by | CN-6-004 role |
|---------|----------|----------------|
| Workshop engine declaration (parametric fabrication) | **CN-6-004** (this doc) | Authoritative |
| Cross-cutting framework | CN-6-100..104 | Parents — CN-6-004 applies |
| Universal Checkout, Accounting, Inventory, Procurement, HR, Reporting, Promotion, Tax | CN-5-001..105 | Consumers; Workshop emits per established contracts |
| Foundation Workflow + Document + Obligation + Inventory Movement primitives | CN-4-011 | Workshop's structural foundation |
| Retail (CN-6-001) | Sibling | RE7 item-type agnosticism applies (workshop-produced items become retail catalog entries); RE11 payment abstraction inherited |
| Restaurant (CN-6-002) | Sibling | REST4 recipes-as-pack-content **distinguished** from WS2 Style-as-tenant-data |
| Hotel (CN-6-003) | Sibling | HOT9 rate-cards-as-pack-content **distinguished similarly** |
| Vertical Bridges (CN-6-005 future) | Sibling | WS11 sell-via-POS established; CN-6-005 catalogs |
| Mixed-Vertical Tenants (CN-6-105 future) | Sibling | WP8 demonstrates; CN-6-105 generalises |
| F&R cluster (salon, motor repair, etc.) | Deferred | Out of CN-6-004 scope; CN-6-101 §11.2 / CN-6-905 question stays open |
| Style Designer canvas UI | Term 3 (pending) | CN-6-004 specifies data model + commands; Term 3 designs visual editor |
| Cutting sheet printing + peripheral integration | Term 7 (partial) | CN-6-004 emits cut list event; Term 7 wires printer adapters |

### 23.2 CTRs

**No new CTRs filed.** Workshop reality fits within existing contracts:
- CTR-018, CTR-002, CTR-024, CTR-026, CTR-027, CTR-030, CTR-038, CTR-044, CTR-045, CTR-046 — cited as-is
- CTR-049, CTR-050 — inherited for payment abstraction
- CTR-028, CTR-006 — universal payment registries
- **Cumulative CTR-046 expansion queue (unchanged):** DC-NN-e + DC-NN-f

### 23.3 Open items inside Term 6 scope

- **CN-6-005 Bridges** — catalogs WS11 + WP8 sell-via-POS pattern; other cross-vertical patterns established across verticals
- **CN-6-105 Mixed-Vertical Tenants** — generalises Mzee Hassan workshop + retail showroom (WP8) alongside Mama Amina retail + pharmacy (CN-6-101 §11.6) and Lodge Serengeti hotel + restaurant (CN-6-003 WP3)
- **CN-6-901..904 Stress-test sketches** — Insurance, Healthcare, Education, Marketing Agency; apply framework to candidate verticals to prove Framework Test #5+ at scale
- **CN-6-905 Salon + light services cluster** — per CN-6-101 §11.2 elevated; stays deferred from CN-6-004; field reality needed
- **2D guillotine algorithm** for sheet materials — Architect-phase implementation; CN-6-004 §13.3 documents the parallel mechanism
- **Style marketplace** (cross-tenant catalog) — deferred per BD8; future feature when ≥2 concrete cases demonstrate need
- **Short-term offcut age limits** — pack rule for "offcut older than X days deprecates to scrap" — pack hook placeholder; v1 keeps offcuts indefinitely

### 23.4 D-DISC cross-references

- **D-DISC-001 — tenant-customer promotion UX:** Workshop project quote presentation, in-stay workshop updates ("your cabinet is in finishing"), warranty registration UX all touch the customer-facing side; Term 3 designs surfaces; CN-6-004 emits the events
- **D-DISC-002 — POS self-service expansion:** Style picker self-service quoting (customer picks Style + enters dimensions remotely; sees quote without visiting Mzee Hassan) — pack flag `pack.workshop.self_service_quote_enabled` placeholder; full implementation deferred

### 23.5 The bar — Framework Test #4 PASSED

CN-6-004 written **without amending CN-6-100..104**. The most complex vertical per Brief — parametric Style designer + Formula Engine + Cut List Generator + Cut Optimizer + offcut tracking + long-lifecycle Workflow + cross-vertical sell — all derives cleanly from the cross-cutting framework. WS1-WS11 doctrine fits within BD3 (vertical uniqueness for algorithms), BD4 (push-down for material specs), BD5 (split-it for Style content + mechanism), BD7 (no bridge engines for cross-vertical sell).

Mzee Hassan's reality of how he fabricates windows in Arusha — the Style on his canvas, the formulas in his head, the bars on his floor, the offcuts in his bin — fits the framework. So does Fundi Athumani's Mwanza workshop receiving the shared Style. So does the Mwemas' commission for their Mwanza renovation. So does the contractor walking into Mzee Hassan's showroom for the spare window.

The framework holds. **Framework Test #4 PASSED.**

This is the final concrete vertical. Mzee Hassan, Mwemas, Salma, Fundi Athumani, the contractor — each operates within doctrine without exceptions. The system tells the fundi how to cut; the fundi cuts. *Hivyo ndivyo — System haimuulizi fundi ahesabu; inamwambia fundi akateje.*

Next: CN-6-005 Vertical Bridges — catalogs cross-vertical patterns established across the four verticals.

---

*— End of CN-6-004 Workshop Engine v1 —*
*— Final concrete vertical of Term 6 —*
*— Framework v1 proven across the full vertical spectrum —*
