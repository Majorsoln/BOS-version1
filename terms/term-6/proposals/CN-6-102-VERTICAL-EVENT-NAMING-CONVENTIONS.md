# CN-6-102 — Vertical Event Naming Conventions

> **Term:** 6 — Vertical Engines
> **Status:** Concept Phase v1.0 — Draft for Overseer review
> **Branch:** `claude/brave-johnson-S7Lky`
> **Reading order:** CN-5-103 (G1–G9 doctrine) → CN-6-100 (manifest delta + VE3/VE4/VE7) → CN-6-101 (BD3/BD6 + VI-03/VI-04) → **this doc**
> **Ordering authority:** TERM-6 Brief §13 — third Term 6 deliverable; operational reference for CN-6-001..004 and all future vertical authors.

---

## 1. Purpose & Boundary

### 1.1 Mission

CN-6-102 is the **operational naming reference** for vertical events. It applies CN-5-103's universal doctrine (G1–G9) to vertical-specific patterns, catalogues the concrete shapes (bill.ready, conflict, regulatory, workflow lifecycle, compensation), and gives every vertical author a template they can copy without re-deriving the rules.

CN-5-103 is the parent doctrine; CN-6-102 does not amend it. Where CN-5-103 says "events follow `<engine>.<noun>.<verb>.v<n>`," CN-6-102 says: for vertical engines specifically, here is how that applies to bills, workflows, conflicts, regulatory events, document amendments, and customer interactions — with the concrete examples and the patterns CN-6-001..004 will reuse.

### 1.2 DOES vs DOES NOT

| CN-6-102 DOES | CN-6-102 DOES NOT |
|---------------|-------------------|
| Apply CN-5-103 G1–G9 to vertical-specific event families | Amend or override CN-5-103 — any gap is a CTR to Term 5 |
| Define NC1–NC9 operational naming conventions | Author the vertical event vocabulary (that is CN-6-001..004) |
| Specify sub-namespace depth limits and the bill.ready uniformity rule | Define manifest mechanism (CN-6-100 §4) or pack hooks (CN-6-100 §3 Step 7) |
| Catalogue patterns: bill.ready, workflow lifecycle, conflict, regulatory, document amendment, customer interaction, compensation | Define vertical scope policy (CN-6-103) or handoff mechanics (CN-6-104) |
| Concretize VI-03 (conflict event family) and VI-04 (regulatory activation) as naming patterns | Ratify new VI-NN invariants (CN-6-101 territory) |
| Reaffirm NC9 compensation-pair declaration as mechanizable via CTR-046 expansion (per `2361014`) | Open new CTRs — operationalisation only |
| Ground patterns in real Tanzanian business (Faraja, Mzee Hassan, Kilimanjaro Lodge Moshi) | Specify UI surfaces for vertical events (Term 3) |

### 1.3 Audience

Term 6 authors writing CN-6-001..004 events; Architects implementing vertical engines; Term 7 reviewing event-schema PRs for naming conformance; future vertical contributors (Insurance, Healthcare, Education, Marketing, Logistics, Pharmacy operations expansion) who need a template.

### 1.4 Charter Compliance

CN-6-102 inherits CN-5-103 G1–G9 (Charter §8.1 naming convention) and CN-6-100 VE3/VE4/VE7. It introduces no new doctrine outside that inheritance — every NC is either a CN-5-103 application or a CN-6-100/101 derivative. Law 5 (compliance configured, not coded) drives NC6's jurisdiction-neutral regulated_subject rule (no `tfda`, `kebs`, `pharmboard` in event names; pack-bound).

### 1.5 Parsimony — the bar

Naming patterns are not abstract. They are spoken by real people. Salma — Mama Amina's daughter, working the duka POS in Kariakoo — does not parse a 5-segment event type when something is wrong at the till. Mzee Hassan in Arusha, reading the audit trail of yesterday's cut list before the morning team arrives, needs event names whose meaning is obvious from the words. Faraja, the pharmacist at the back of Mama Amina's expanded duka, must trust that `pharmacy.controlled_substance.dispensed.v1` means exactly what it sounds like — no jurisdiction-specific acronyms, no internal codes, no synonyms across her workflow. The front desk at Kilimanjaro Lodge Moshi onboards seasonal staff every dry season — a regional agent training them cannot afford patterns that take a week to internalise.

Patterns that use ordinary business words, version cleanly, and reject ambiguity are the patterns that survive production. Patterns that require a glossary die in the first month. **Parsimony is the bar.** Every NC1–NC9 below is justified against it.

---

## 2. Inputs and Relationship

### 2.1 Parent doctrine

- **CN-5-103 G1–G9** is authoritative for naming. G1 `<engine>.<noun>.<verb>.v<n>`; G2 `kernel.*` reserved; G3 adapter sub-namespace pattern; G4 `system.*` prohibited at universal level; G5 compensation pair symmetry; G6 versioning + deprecation; G7 governance for new event types; G8 producer-authoritative semantics; G9 past-tense verb.
- **CN-5-103 §16** pre-allocates vertical namespaces (`retail.*`, `restaurant.*`, `hotel.*`, `workshop.*`, `pharmacy.*`, `clinic.*`).

### 2.2 Term 6 upstream

- **CN-6-100 VE3** — vertical owns namespace exclusively; CN-6-102 NC8 enforces what cannot mean inside that namespace.
- **CN-6-100 VE4** — mandatory `<vertical>.bill.ready.v1` emission; CN-6-102 NC3 fixes its exact shape.
- **CN-6-100 VE7** — Workflow primitive for stateful processes; CN-6-102 NC5 defines the lifecycle event pattern.
- **CN-6-101 §12.1 VI-03** — conflict event family invariant; CN-6-102 NC4 concretizes the naming.
- **CN-6-101 §12.2 VI-04** — regulatory activation gate; CN-6-102 NC6 concretizes the regulated_subject naming.
- **CN-6-101 §10 AP5** — vertical cannot register events under a universal namespace; CN-6-102 NC8 reaffirms.

### 2.3 Sibling boundary

CN-6-103 (Scope Policy) and CN-6-104 (Hand-Off Pattern) elaborate scope and handshake mechanics; CN-6-102 names what they hand off.

---

## 3. Doctrine — NC1–NC9

**NC1 — Verticals follow CN-5-103 G1–G9 verbatim.** No naming exceptions. CN-6-102 is operationalisation; any gap is a CTR to Term 5, not a vertical-side workaround.

**NC2 — Sub-namespace depth: 3-segment minimum, 4-segment when grouped, 5+ prohibited.** Permitted shapes are `<vertical>.<noun>.<verb>.v<n>` (3-segment standard) and `<vertical>.<sub_domain>.<noun>.<verb>.v<n>` (4-segment when grouping logically related events). 2-segment violates G1 (no noun). 5+ creates parsing and subscription ambiguity.

**NC3 — Bill emission is exactly `<vertical>.bill.ready.v1`.** 3-segment, no sub-domain, no version branching by vertical. Universal Checkout subscribes by canonical pattern (`*.bill.ready.v1`); deviation breaks subscription.

**NC4 — Conflict event family is `<vertical>.<noun>.conflict.detected.v1`.** 4-segment with `conflict` as `<sub_domain>` and `detected` as past-tense verb. Per VI-03 (CN-6-101 §12.1). The conflict event names the contested resource (`table`, `room`, `equipment`, `cabinet_slot`) as the noun.

**NC5 — Workflow lifecycle events use `<vertical>.<workflow_name>.<state>.v<n>`.** 4-segment. `<state>` is the past-participle of the transition verb (G9). The event marks the transition INTO the state, not the steady occupation of the state. Steady states without transition verbs do not emit events; the prior transition is sufficient.

**NC6 — Regulatory events use `<vertical>.<regulated_subject>.<noun>.<verb>.v<n>` with jurisdiction-neutral `regulated_subject`.** `regulated_subject` names the regulated domain concept (`controlled_substance`, `patient_record`, `prescription`, `policy`, `student_record`). Jurisdiction binding — which regulatory body, which template, which retention rule — lives in pack hooks per §8.1, never in event names. Per VI-04 (CN-6-101 §12.2).

**NC7 — Versioning inherits CN-5-103 G6 verbatim.** Additive payload additions stay `.v1`; breaking schema changes become `.v2` with pack-driven compatibility window. No vertical-specific override.

**NC8 — Reserved keys within a vertical namespace.** A vertical may NOT emit events under any of:

- `<vertical>.kernel.*` — kernel meta-events live at top-level `kernel.*` only
- `<vertical>.pack.*` — pack lifecycle events are pack-owned (per CTR-037)
- `<vertical>.test.*` — production event store carries no test events (see §11)
- `<vertical>.system.*` — parallel to G4 universal prohibition; misleads as kernel-level
- `<vertical>.checkout.*` — Universal Checkout owns checkout vocabulary (per VE4)
- `<vertical>.<other_vertical>.*` — cross-vertical namespace claim violates VE3
- `<vertical>.advisor.*` — **discouraged** (not strictly prohibited): advisor events flow through `kernel.advisor.*` per CN-5-010; vertical-namespaced advisor events mislead

**NC9 — Compensation pair declaration is mandatory in the manifest.** Per G5 (CN-5-103), every state-effecting vertical event must declare its compensation pair (or explicitly declare no compensation basis) at registration. The CN-4-020 doctrine gate verifies. Mechanizable via CTR-046 expansion (DC check per `2361014` ratification). See §10 for manifest schema.

---

## 4. Sub-Namespace Depth Rules (NC2 Elaboration)

### 4.1 The three permitted shapes

| Shape | Form | When to use |
|-------|------|-------------|
| 3-segment | `<vertical>.<noun>.<verb>.v<n>` | Default. Single domain concept. Examples: `retail.sale.completed.v1`, `pharmacy.bill.ready.v1` |
| 4-segment | `<vertical>.<sub_domain>.<noun>.<verb>.v<n>` | Grouping related events. Examples: `restaurant.kitchen.ticket.fired.v1`, `workshop.glass.cut.executed.v1`, `hotel.housekeeping.room.cleaned.v1` |
| 4-segment workflow | `<vertical>.<workflow>.<state>.v<n>` | Workflow lifecycle (NC5). Examples: `hotel.reservation.confirmed.v1`, `workshop.project.completed.v1` |

### 4.2 Why 5+ is prohibited

Subscription pattern matching becomes ambiguous (`a.b.c.d.e.v1` — is `b.c` the sub-domain or `c.d`?), event indices balloon, and parsimony fails the Salma-at-the-till test. If 5 segments seem needed, the design is wrong: split into two events (different concerns) or use a payload `kind` discriminator (NC1 + Q10 pattern).

### 4.3 Sub-domain naming rules

Sub-domains are single lowercase tokens (snake_case allowed for compound concepts): `kitchen`, `housekeeping`, `glass`, `linear`, `controlled_substance`, `patient_record`. They are **not** themselves verticals (no `pharmacy.retail.*`); they are *internal grouping* within the vertical's own namespace.

---

## 5. Bill.ready Emission Discipline (NC3)

### 5.1 The canonical shape

Every vertical that sells, charges, or invoices emits exactly:

```
<vertical>.bill.ready.v1
```

3-segment. No sub-domain prefix. Universal Checkout (CN-5-009) subscribes by pattern `*.bill.ready.v1` and dispatches to any vertical emission.

### 5.2 Why uniformity is non-negotiable

A subscription `restaurant.bill.ready.v1` + `hotel.bill.ready.v1` + `pharmacy.bill.ready.v1` works because the shape is identical. The moment one vertical emits `pharmacy.dispensing.bill.ready.v1` or `hotel.folio.bill.ready.v1`, Universal Checkout must hardcode per-vertical patterns — VE6 violation, BD2 anti-test failure, and the gap Brief §12 warns of.

### 5.3 Payload shape per CN-6-100 §3 Step 3

```
{
  bill_id,
  site_id,                        # CTR-024
  saleable_lines: [...],          # CN-4-021
  originating_workflow_ref,       # VE7 Workflow primitive
  business_date,                  # CN-5-105 tax period
  payer_party_ref?                # Party primitive
}
```

---

## 6. Workflow Lifecycle Event Pattern (NC5 + State-as-Verb Mechanics)

### 6.1 The transition-into-state rule

The event marks the moment the workflow transitions INTO the named state. The state name is the past-participle of the transition verb (G9). The event is emitted exactly once per transition; replaying transitions reconstructs the workflow history.

### 6.2 Correct vs incorrect state-as-verb naming (N4)

| Workflow context | ✓ Correct | ✗ Incorrect | Why |
|-------------------|-----------|--------------|-----|
| Hotel reservation enters "confirmed" | `hotel.reservation.confirmed.v1` | `hotel.reservation.confirming.v1` | Present-participle (G9 violation) |
| Hotel reservation enters "checked in" | `hotel.reservation.checked_in.v1` | `hotel.reservation.in_house.v1` | State noun, not verb |
| Restaurant table opened for service | `restaurant.table.opened.v1` | `restaurant.table.occupied.v1` | State adjective, not transition verb |
| Workshop project enters cutting phase | `workshop.project.cutting_started.v1` | `workshop.project.cutting.v1` | Present-participle (steady state, not entry) |
| Pharmacy prescription validated by pharmacist | `pharmacy.prescription.validated.v1` | `pharmacy.prescription.valid.v1` | State adjective, not verb |
| Logistics trip enters in-transit phase | `logistics.trip.in_transit_started.v1` | `logistics.trip.in_transit.v1` | Steady-state noun, not entry verb |

### 6.3 The steady-state rule

States like `in_house`, `cooking`, `in_transit`, `occupied` are *steady states between transitions*. They do not emit events on their own — the prior transition event is sufficient. If a downstream subscriber needs the steady-state start moment as a distinct event, use a transition verb: `<state>_started.v1` (`in_transit_started.v1`, `occupancy_started.v1`). Past-tense form, not present-participle.

### 6.4 Terminal states

Workflow terminal states use past-tense terminal verbs: `closed`, `archived`, `cancelled`, `abandoned`, `aborted`. Examples: `hotel.reservation.cancelled.v1`, `workshop.project.archived.v1`, `restaurant.table_session.closed.v1`.

---

## 7. Conflict Event Family (NC4 — VI-03 Ratification)

### 7.1 The canonical shape

```
<vertical>.<contested_resource>.conflict.detected.v1
```

4-segment. `conflict` is the sub-domain; `detected` is the past-tense verb. The contested resource (the noun in `<contested_resource>` position) names what the bus single-acceptance mechanism (CN-4-004) rejected the contender on.

### 7.2 Examples per existing vertical

| Vertical | Conflict | Event |
|----------|----------|-------|
| Restaurant | Two waiters claim same table | `restaurant.table.conflict.detected.v1` |
| Hotel | Overbooking on same room-night | `hotel.room.conflict.detected.v1` |
| Workshop | Two cuts dispatched on same bar | `workshop.bar.conflict.detected.v1` |
| Pharmacy | Two pharmacists access controlled cabinet | `pharmacy.cabinet_slot.conflict.detected.v1` |

### 7.3 Payload shape

```
{
  contested_resource_ref,
  contender_workflow_refs: [losing_workflow, ...],
  winning_workflow_ref,
  detection_ts,
  resolution_basis             # e.g., "bus_single_acceptance_first_arrival"
}
```

The losers (in `contender_workflow_refs`) receive compensating events per their own workflow's compensation declaration (NC9). The winner proceeds.

---

## 8. Regulatory Event Family (NC6 — VI-04 Application)

### 8.1 The canonical shape + jurisdiction-neutrality rule

```
<vertical>.<regulated_subject>.<noun>.<verb>.v<n>
```

4-segment. `regulated_subject` is the **jurisdiction-neutral** domain concept (`controlled_substance`, `patient_record`, `prescription`, `policy`, `student_record`, `currency_exchange`). Jurisdiction-specific bodies (`tfda`, `kebs`, `nda`, `kra`, `tra`, `moh`, `medical_board`) MUST NOT appear in event names.

### 8.2 Examples

| Vertical | Regulated subject | Event |
|----------|--------------------|-------|
| Pharmacy | controlled_substance | `pharmacy.controlled_substance.dispensed.v1` |
| Pharmacy | prescription | `pharmacy.prescription.validated.v1` |
| Clinic | patient_record | `clinic.patient_record.created.v1` |
| Clinic | patient_record | `clinic.patient_record.amended.v1` |
| Insurance (future) | policy | `insurance.policy.issued.v1` |
| Insurance (future) | claim | `insurance.claim.adjudicated.v1` |

### 8.3 Pack hook schema for regulated subjects (N2)

The jurisdiction-specific interpretation lives in pack hooks per `pack.<vertical>.<regulated_subject>.*`:

```
pack.<vertical>.<regulated_subject>.regulatory_body_ref       # e.g., "tfda" in TZ pack; "kebs" in KE pack
pack.<vertical>.<regulated_subject>.reporting_template_ref    # CN-4-012 Document template
pack.<vertical>.<regulated_subject>.audit_retention_days      # e.g., 730 (2 yrs) for TFDA controlled substance
pack.<vertical>.<regulated_subject>.notification_required     # bool — does emission trigger external notification?
pack.<vertical>.<regulated_subject>.notification_channel_ref  # if true: which channel adapter per CTR-019/020
pack.<vertical>.<regulated_subject>.controlled_categories     # list of item categories under regulation
```

Concrete example for Pharmacy controlled-substance in Tanzania:

```
pack.pharmacy.controlled_substance.regulatory_body_ref = "tfda"
pack.pharmacy.controlled_substance.reporting_template_ref = "tfda_csr_monthly_v3"
pack.pharmacy.controlled_substance.audit_retention_days = 730
pack.pharmacy.controlled_substance.notification_required = false
pack.pharmacy.controlled_substance.controlled_categories = ["morphine", "pethidine", "diazepam", ...]
```

Kenya pack would fill the same slot names with `kebs` / different template / different retention. The event `pharmacy.controlled_substance.dispensed.v1` is identical across both jurisdictions; only the pack interpretation differs. Multi-jurisdiction expansion (CN-5-105 §11 v2) requires no event-name change.

### 8.4 Why this matters

Embedding `tfda` in event names couples namespace to jurisdiction; a Kenyan pharmacy onboarding would either replay TZ-named events (semantically wrong) or fork the namespace (CTR-038 violation + framework breakage). Jurisdiction-neutral subjects + pack-bound bodies preserve Law 5 (compliance configured, not coded) and Law 6 (distribution regional).

---

## 9. Pattern Catalog

The reference table for vertical event naming. CN-6-001..004 authors copy patterns from here.

| Operation family | Pattern | Examples |
|-------------------|---------|----------|
| Bill emission (VE4) | `<vertical>.bill.ready.v1` | `retail.bill.ready.v1`, `pharmacy.bill.ready.v1` |
| Workflow state transition (NC5) | `<vertical>.<workflow>.<state>.v<n>` | `hotel.reservation.checked_in.v1`, `workshop.project.completed.v1` |
| Conflict detection (NC4 / VI-03) | `<vertical>.<contested_resource>.conflict.detected.v1` | `restaurant.table.conflict.detected.v1` |
| Regulatory event (NC6 / VI-04) | `<vertical>.<regulated_subject>.<noun>.<verb>.v<n>` | `pharmacy.controlled_substance.dispensed.v1` |
| Inventory consumption (Pattern B per CN-5-003) | `<vertical>.<consumed_kind>.consumed.v1` | `restaurant.ingredient.consumed.v1`, `workshop.material.consumed.v1` |
| Compensation pair (G5 / NC9) | `<vertical>.<noun>.<verb>.v1` + paired `<vertical>.<noun>.<reverse_verb>.v1` | `restaurant.table.opened.v1` ↔ `restaurant.table.closed.v1`; `hotel.folio.charged.v1` ↔ `hotel.folio.refunded.v1` |
| Document issuance | `<vertical>.<doc_kind>.issued.v1` | `hotel.invoice.issued.v1`, `pharmacy.dispensing_label.issued.v1` |
| **Document amendment (new — CN-5-006 N3 vertical-level)** | `<vertical>.<doc_kind>.amended.v1` | `hotel.invoice.amended.v1` (never delete; amendment = compensating-additive) |
| **Workflow blocked state (new — CN-6-100 §3.13)** | `<vertical>.<workflow>.blocked.v1` | `workshop.project.blocked.v1 {reason: material_shortage}` |
| Customer interaction (D-DISC-001 placeholder) | `<vertical>.customer.<verb>.v<n>` carrying `party_ref` | `restaurant.customer.seated.v1 {party_ref}`, `pharmacy.customer.consultation_started.v1 {party_ref}` |
| Vertical command | `<vertical>.<noun>.<verb>.request` | `hotel.reservation.confirm.request`, `pharmacy.prescription.validate.request` |
| Tax-treatment reference (CN-5-105) | Payload field `tax_treatment_ref`; no event family | (payload convention, not event name) |

**No `cross_vertical.*` row.** Q5 closure: cross-vertical loyalty, charge-to-room, and other inter-vertical flows route through Universal Promotion / Foundation Obligation / Party primitives — never via a `cross_vertical.*` namespace. See §11 for the Mama Amina + Faraja worked walkthrough.

### 9.1 Customer-interaction party_ref discipline

Every `<vertical>.customer.<verb>.v<n>` event carries `party_ref` resolving to an existing Party primitive (CN-4-011) instance. Verticals MUST NOT invent customer rows; the Party primitive owns customer identity tenant-wide. Per CN-6-100 §3.13 customer-identity pattern.

---

## 10. Versioning + Deprecation + Compensation Symmetry

### 10.1 Versioning (NC7)

Inherit CN-5-103 G6 verbatim. Summary for vertical authors:

- Additive payload field additions remain `.v1` — subscribers ignore unknown fields per producer-authoritative semantics (G8).
- Breaking schema changes (field rename, type change, required-field removal) increment to `.v2`. Both versions coexist during the pack-driven compatibility window; `pack.event_compatibility.deprecation_window_days` (default per jurisdiction pack) bounds the window.

### 10.2 Deprecation cycle

Per CN-5-103 G6, deprecation runs pack-driven. Vertical authors do not invent vertical-specific compatibility windows. Jurisdiction packs MAY set tighter or looser windows; tenant onboarding inherits per CTR-027 jurisdiction binding.

### 10.3 Compensation pair declaration (NC9 — N1 manifest schema)

Every state-effecting vertical event declares its compensation pair (or explicit no-compensation rationale) at registration in the manifest. The CN-4-020 doctrine gate verifies. CTR-046 expansion (`2361014`) adds a DC check that mechanizes this — manifests missing both a `compensation_pair` reference and a `compensation_basis_none` rationale are rejected at registration.

**Manifest schema (per emit entry):**

```
emits:
  - event_type: <vertical>.<noun>.<verb>.v<n>
    payload_contract_ref: <ref>
    compensation_pair: <vertical>.<noun>.<reverse_verb>.v<n>   # OR
    compensation_basis_none: "<rationale text>"                # mutually exclusive
```

**Examples — pair declared:**

```
- event_type: restaurant.table.opened.v1
  compensation_pair: restaurant.table.closed.v1

- event_type: hotel.folio.charged.v1
  compensation_pair: hotel.folio.refunded.v1

- event_type: pharmacy.prescription.dispensed.v1
  compensation_pair: pharmacy.prescription.recalled.v1
```

**Examples — basis-none declared:**

```
- event_type: restaurant.kitchen.ticket.fired.v1
  compensation_basis_none: "Fired tickets cannot be unfired — kitchen has begun cooking; modifications use ticket.amended.v1 additive event."

- event_type: hotel.housekeeping.room.cleaned.v1
  compensation_basis_none: "Cleaning is a physical act; the event records observation, not state change."
```

Either compensation_pair OR compensation_basis_none MUST be present. The doctrine gate rejects manifests with neither. This is the NC9 enforcement that prevents the silent-compensation-gap pattern Brief §12 warns of.

---

## 11. Reserved Keys + Prohibited Patterns

### 11.1 Reserved key prohibition list (NC8)

Within a vertical's own namespace, the following sub-namespaces are reserved and emission is rejected at registration:

| Reserved | Reason |
|----------|--------|
| `<vertical>.kernel.*` | Kernel meta-events live at top-level `kernel.*` only (per CN-5-103 G2) |
| `<vertical>.pack.*` | Pack lifecycle events are pack-owned (per CTR-037 pending Term 4) |
| `<vertical>.test.*` | Production event store carries no test events; tests use Developer-AI sandbox per D-002B + CN-4-023 |
| `<vertical>.system.*` | Parallel to G4 universal-level prohibition; "system" misleads as kernel-level |
| `<vertical>.checkout.*` | Universal Checkout (CN-5-009) owns checkout vocabulary; verticals never emit checkout events |
| `<vertical>.<other_vertical>.*` | Cross-vertical namespace claim violates VE3 exclusivity |
| `<vertical>.advisor.*` | **Discouraged** — advisor events flow through `kernel.advisor.*` per CN-5-010; vertical-namespaced advisor sub-namespace misleads |

### 11.2 Test events — production prohibition

Architect-phase integration tests use the Developer-AI sandbox per D-002B and CN-4-023 kernel-boundary doctrine. Tests run against synthetic events that never enter the production event store. `<vertical>.test.*` emission at runtime is rejected at the doctrine gate. This is consistent with Charter §10's "no code during Concept Phase" + CN-4-023's out-of-kernel build-time-AI boundary.

### 11.3 Cross-vertical reference — concrete schema (N3)

Per Q8, cross-vertical event references via embedded event_id are an anti-pattern. The correct mechanism is via Foundation primitives (Party, Obligation, Workflow).

**✗ INCORRECT — embedded event_id reference:**

```
{
  prescription_id: "rx-9281",
  ...
  retail_sale_ref: "retail.sale.completed.v1#evt-77321"   # ❌ direct cross-vertical event_id
}
```

This couples Pharmacy to Retail's event schema, version, and lifecycle. Decommissioning Retail breaks Pharmacy. VE2 violation.

**✓ CORRECT — Party-primitive link (query-time resolution):**

```
{
  prescription_id: "rx-9281",
  ...
  party_ref: "party-44192"                # Foundation Party primitive ID
}
```

If Pharmacy needs to know whether the same Party also bought retail items, it queries the Party-anchored event history at *read time* via projection (CN-4-010). No coupling at emission time. Party primitive is tenant-scope and shared across all verticals.

**✓ CORRECT — Obligation-primitive link (event-time relationship):**

```
{
  bill_id: "bill-7733",
  obligation_refs: ["obligation-91"]      # Foundation Obligation primitive ID
}
```

When a relationship must be explicit at event time (e.g., the charge-to-room case from CN-6-101 §11.4), the Obligation primitive carries it. The verticals on either side reference the same `obligation_ref` without referencing each other's events.

### 11.4 Mama Amina + Faraja Mixed-Vertical loyalty walkthrough (N6)

Mama Amina's tenant runs both `retail.*` (Salma at the till) and `pharmacy.*` (Faraja at the counter). A regular customer — Mama Halima (the Dar logistics broker, in town for the week) — earned loyalty points buying rice and soap last Tuesday. Today she brings a prescription for her daughter's antibiotics. She wants to redeem the points against the prescription bill.

**No cross-vertical event family is invented.** The flow uses universal Promotion + Party primitive linking:

```
Tuesday — retail earning:
  retail.sale.completed.v1 {sale_id, party_ref: <halima>, total: 18500 TZS, site_id, business_date}
       │
       ▼  (Promotion subscribes via universal subscription per CN-5-007)
  promotion.loyalty.earned.v1 {party_ref: <halima>, points: 185, basis: bill_total, ...}

Today — pharmacy redemption:
  pharmacy.prescription.validated.v1 {prescription_id, party_ref: <halima>, dispenser_ref: <faraja>, ...}
       │
       ▼  (Pharmacy emits bill with loyalty redeem intent)
  pharmacy.bill.ready.v1 {
    bill_id,
    site_id,
    payer_party_ref: <halima>,
    saleable_lines: [...],
    discount_refs: [{kind: "loyalty_redeem_intent", points_offered: 185}],
    originating_workflow_ref: <prescription_workflow>,
    business_date
  }
       │
       ▼  (Universal Checkout consumes; Promotion subscribes to compute discount)
  promotion.loyalty.redeemed.v1 {party_ref: <halima>, points_consumed: 185, discount_amount: 1850 TZS, ...}
       │
       ▼
  checkout.settled.v1 {bill_id, tendered: ..., change: ..., receipt_doc_ref}
```

Observations:

- The link between Tuesday's retail sale and today's pharmacy redemption is `party_ref` — a Foundation Party primitive ID. Both verticals reference it; neither references the other.
- The mechanism (loyalty earning, balance, redemption) lives in Universal Promotion (`promotion.loyalty.*`). Per BD5 split-it.
- **No `cross_vertical.*`, `retail.pharmacy.*`, or `pharmacy.retail.*` event is emitted.** Q5 closure concretely demonstrated.
- Retail and Pharmacy can be decommissioned independently; the loyalty mechanism survives because it lives in Promotion + Party, not at the vertical boundary.

This is the pattern every Mixed-Vertical Tenant (CN-6-105) reuses.

---

## 12. Worked Patterns — Four Vertical Applications

Building on the Term 6 character map established in CN-6-100 (Mama Halima Logistics, §12) and CN-6-101 (Mama Amina + Faraja + Lodge Serengeti + Mzee Hassan), the patterns below demonstrate NC1–NC9 in concrete vertical context.

### 12.1 WP1 — Hotel reservation lifecycle (Kilimanjaro Lodge Moshi)

Mama na Bwana Mwema from Mwanza book a weekend at Kilimanjaro Lodge Moshi via a regional travel agent. The reservation Workflow `hotel.reservation` follows lifecycle `held → confirmed → checked_in → checked_out → archived` (per CN-6-100 §3 Step 5). Each transition emits a 4-segment lifecycle event (NC5):

```
hotel.reservation.held.v1            (held by agent at booking time)
hotel.reservation.confirmed.v1       (confirmed when payment_received OR credit_approved guard passes)
hotel.reservation.checked_in.v1      (Friday evening front desk action)
hotel.reservation.checked_out.v1     (Sunday morning bill settlement)
hotel.reservation.archived.v1        (post-checkout cleanup; folio + audit retained)
```

Compensation pairs (NC9 manifest declarations):

```
hotel.reservation.held.v1            ↔ hotel.reservation.cancelled.v1
hotel.reservation.confirmed.v1       ↔ hotel.reservation.cancelled.v1
hotel.reservation.checked_in.v1      ↔ hotel.reservation.early_departed.v1
```

Conflict event (NC4 / VI-03) — if the system tries to confirm the same room-night to two reservations:

```
hotel.room.conflict.detected.v1 {contested_resource_ref: <room>, winning_workflow_ref, contender_workflow_refs}
```

The losing reservation receives its own `hotel.reservation.cancelled.v1` compensation; the winning one proceeds to `hotel.reservation.confirmed.v1`.

### 12.2 WP2 — Pharmacy prescription dispensing (Faraja at Mama Amina's pharmacy counter)

Faraja receives a prescription from a customer. Workflow `pharmacy.prescription` runs:

```
pharmacy.prescription.received.v1
pharmacy.prescription.validated.v1     (Faraja confirms doctor signature + dosage)
pharmacy.prescription.dispensed.v1     (drugs handed over; controlled-substance subset → separate regulatory event below)
pharmacy.prescription.archived.v1
```

Where the prescription includes a controlled substance (e.g., diazepam), the regulatory event (NC6 — jurisdiction-neutral subject) ALSO emits:

```
pharmacy.controlled_substance.dispensed.v1 {
  prescription_ref,
  party_ref,
  controlled_category: "diazepam",
  quantity,
  dispenser_ref: <faraja>,
  cabinet_slot_ref,
  business_date
}
```

Pack interpretation (per §8.3):

```
pack.pharmacy.controlled_substance.regulatory_body_ref = "tfda"
pack.pharmacy.controlled_substance.reporting_template_ref = "tfda_csr_monthly_v3"
pack.pharmacy.controlled_substance.audit_retention_days = 730
```

If a Kenyan pack is later swapped in (multi-jurisdiction v2), the event names do not change — only the pack interpretation. Compensation pair (NC9):

```
pharmacy.controlled_substance.dispensed.v1 ↔ pharmacy.controlled_substance.recalled.v1
```

Bill emission (NC3):

```
pharmacy.bill.ready.v1 {bill_id, site_id, saleable_lines, originating_workflow_ref: <prescription>, business_date, payer_party_ref}
```

### 12.3 WP3 — Restaurant table session (Lodge Serengeti restaurant — continuity with CN-6-101 §11.4)

The Lodge Serengeti restaurant runs `restaurant.table_session` Workflow per table. The sub-domain `kitchen` groups kitchen-ticket events; the noun is the compound `kitchen_ticket` (Q9 compound-noun heuristic — the ticket has its own lifecycle).

```
restaurant.table_session.opened.v1
restaurant.kitchen_ticket.fired.v1
restaurant.kitchen_ticket.cooked.v1
restaurant.kitchen_ticket.served.v1
restaurant.table_session.closed.v1
```

Conflict event (NC4 — two waiters claim same table):

```
restaurant.table.conflict.detected.v1 {contested_resource_ref: <table>, ...}
```

Cross-vertical (charge-to-room from CN-6-101 §11.4):

```
restaurant.bill.ready.v1 {payment_method_hint: "charge_to_room", obligation_refs: [<hospitality_charge_obligation>]}
```

The Obligation primitive (per §11.3 N3 + CN-6-101 §11.4) carries the cross-vertical relationship to the hotel folio. No `restaurant.hotel.*` event.

### 12.4 WP4 — Workshop project quote-to-cut (Karakana ya Mzee Hassan, Arusha)

Mzee Hassan runs `workshop.project` Workflow from quote acceptance through cut completion:

```
workshop.project.quoted.v1
workshop.project.accepted.v1            (customer accepts quote; production cleared to start)
workshop.project.cutting_started.v1     (NC5 steady-state entry — past-tense; not "cutting" present-participle)
workshop.glass.cut.executed.v1          (each cut emits; 4-segment with sub-domain "glass" or "linear" per material)
workshop.glass.cut.executed.v1
workshop.material.consumed.v1           (Pattern B per CN-5-003)
workshop.project.completed.v1
workshop.project.archived.v1
```

Compensation pairs (NC9):

```
workshop.project.accepted.v1            ↔ workshop.project.cancelled.v1
workshop.glass.cut.executed.v1          ↔ workshop.glass.cut.reversed.v1    (rare: customer dispute reverts a cut)
workshop.material.consumed.v1           compensation_basis_none: "Consumed material physically used; recovery via re-fabrication, not event compensation."
```

Versioning (NC7) — if a customer's style update mid-project requires a payload field expansion (e.g., adding `style_version_pinned_at` to track which style snapshot the project froze to), additive only → `.v1` survives. A breaking change (e.g., restructuring `cuts[]` into typed `linear_cuts[]` + `glass_cuts[]`) would require `.v2` with the pack-driven compatibility window.

### 12.5 Cross-reference — Logistics (Mama Halima, Dar)

CN-6-100 §12 established Mama Halima's `logistics.trip` Workflow with multi-site-by-nature scope (§8.3). The same NC1–NC9 patterns apply: `logistics.trip.planned.v1 → logistics.trip.dispatched.v1 → logistics.trip.in_transit_started.v1 → logistics.trip.arrived.v1 → logistics.trip.closed.v1`; conflict event `logistics.vehicle.conflict.detected.v1`; bill emission `logistics.bill.ready.v1`; cross-vertical Workshop-delivery obligation via Obligation primitive. The Logistics walkthrough in CN-6-100 §12 is the original Term 6 worked example; CN-6-102 patterns conform to and extend it.

---

## 13. Boundaries + Open Items + Cross-Term Hooks

### 13.1 CN-6-102's place in the Term 6 corpus

| Concern | Owned by | CN-6-102 role |
|---------|----------|----------------|
| Vertical event naming syntax + family patterns | **CN-6-102** (this doc) | Authoritative |
| Universal naming doctrine G1–G9 | CN-5-103 | Parent — CN-6-102 applies, never amends |
| Manifest mechanism + namespace ownership | CN-6-100 | CN-6-102 NC3, NC8, NC9 cite |
| Boundary doctrine (layer placement) | CN-6-101 | CN-6-102 inherits VI-03, VI-04 ratification |
| Vertical scope policy | CN-6-103 | Sibling — CN-6-102 names what CN-6-103 scopes |
| Vertical-to-Universal hand-off mechanics | CN-6-104 | Sibling — CN-6-102 names the events CN-6-104 hands off |
| Concrete vertical events (Retail, Restaurant, Hotel, Workshop) | CN-6-001..004 | Consumers — they copy CN-6-102 patterns |
| Mixed-Vertical Tenant patterns | CN-6-105 | CN-6-102 §11.4 (Mama Amina + Faraja) demonstrates loyalty case; CN-6-105 generalises |
| Future-vertical naming | CN-6-901..904 + CN-6-905 | Apply CN-6-102 patterns to new namespaces |

### 13.2 CTRs cited (no new CTRs)

- **CTR-046 expansion** (`2361014` on main) — DC check for NC9 compensation-pair declaration is mechanized. CN-6-102 §10 cites; vertical authors rely on doctrine-gate enforcement.
- **CTR-044** — `engine_kind: vertical` manifest flag still pending Term 4; CN-6-102 NC9 manifest schema assumes flag presence.
- **CTR-045** — Vertical onboarding governance still pending Term 1; affects activation-time naming pattern review.
- **CTR-047** — `data_sensitivity_tier` primitive mjadala still pending Term 4; may affect regulated-subject NC6 expansion in a future amendment.
- **CTR-037** — `pack.*` namespace ownership still pending Term 4; NC8 `<vertical>.pack.*` prohibition assumes ratification.

### 13.3 Open items inside Term 6 scope

- **D-DISC-001 — Tenant-customer promotion UX**: CN-6-102 §9 customer-interaction row sets the naming pattern (`<vertical>.customer.<verb>.v<n>` with party_ref). Term 3 designs the surface; naming is ready.
- **D-DISC-002 — POS self-service expansion**: CN-6-102 patterns accommodate without naming change. Vertical-specific self-service events use existing customer-interaction + workflow-lifecycle patterns.
- **Compound noun heuristic** — Q9 closure documented at §9 catalogue level and §12.3 (kitchen_ticket example). CN-6-001..004 authors apply heuristic per case.
- **Multi-jurisdiction expansion** — When CN-5-105 v2 lands, NC6 jurisdiction-neutral subject pattern absorbs without naming change. Pack hooks per §8.3 are the only adjustment point.

### 13.4 The bar restated

Salma reads the events at her till when a sale fails. Mzee Hassan reads them in his audit log when a fundi flagged a wrong cut. Faraja reads them when reconciling controlled-substance monthly reports for TFDA. The receptionist at Kilimanjaro Lodge Moshi reads them when a guest disputes a folio charge. The patterns in CN-6-102 are the words they speak.

If a future vertical author proposes a pattern Salma or Faraja cannot read aloud and understand within five seconds, the pattern is wrong and CN-6-102 is the standard by which it is rejected.

---

*— End of CN-6-102 Vertical Event Naming Conventions v1 —*
