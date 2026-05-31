# CN-5-007 — Promotion Engine

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 5 — Universal Engines
> **Status:** For Overseer review.
> **Governing decisions:** D-001 (Universal Checkout — Promotion populates inputs; Checkout settles); D-008 (Conversational/Messaging Channels — outreach over channels with consent); D-009 (freeze doctrine — vouchers/promotion records under pack at issuance); Law 2 (engine isolation — read-side targeting; no inter-engine commands); Law 5 (compliance is configured — calculation precedence, tax treatment, cost-share rules in packs); UI-01 (causation chain); UI-02 (compensation symmetry on refunds); UI-08 (Obligation bounds — loyalty + cost-share receivables); UI-10 (cost-share reconciliation — bos + agent + supplier + tenant = total_discount).
> **CTRs:**
> - **OPEN (this doc extends with expansion notes at merge):** CTR-021 (Term 5 → Term 4/7 — outreach over channels + consent; Promotion commands hard-block on missing Consent); CTR-024 (Term 5 → Term 6 — verticals carry `site_id`; relevant for agent_share cost-share routing); CTR-029 (Term 1 — pack gains: stacking policy, calculation precedence, voucher restoration policy, voucher namespace, voucher breakage account, loyalty earn basis, cost-share cadence); CTR-030 (Term 6 — vertical-specific promotion triggers + cost-share GL account mapping per vertical).
> - **No new CTRs** for v1.
> **Glossary:** See `MASTER-GLOSSARY.md` — Universal-engine Invariant (UI), Site, Scope level, Obligation, Document, Consent, Party.
> **Depends on:** CN-4-002 (envelope); CN-4-004 (command bus + atomic emission + policy rejection); CN-4-005 (engine manifest contract); CN-4-007 (Identity — elevated principal for manual_override per Q3); CN-4-011 (Obligation primitive for loyalty + cost-share receivables; Document primitive for vouchers; Party primitive for customer profile; Item primitive for target-item rules; Consent primitive for outreach gating); CN-4-012 (Document Engine — voucher hash + numbering + template + freeze); CN-4-013 (Decision Journal — manual_override authorisation records); CN-4-014 (clock protocol — time-window evaluation; voucher expiry scheduling); CN-4-015 (Compliance DSL — pack rules); CN-4-021 (Saleable Line — discount_refs populated by Promotion); CN-5-009 (Checkout — applies promotions; Promotion never settles tender); CN-5-001 (Accounting auto-journal #13 cost-share); CN-5-002 (Cash settles cost_share_receivable Obligations via AR flow per N4); CN-5-003 (Inventory subscribes for stock-availability targeting); CN-5-006 (Reporting subscribes for promotion ROI analytics); CN-5-100 (subscription patterns — auto-apply P3; refund compensation P4); CN-5-101 (scope policy — Promotion is multi-scope per §6: tenant default + site overrides); CN-5-102 (UI-01, UI-02, UI-08, UI-10).
> **Boundaries:** Document Engine mechanics → CN-4-012; Obligation primitive fold → CN-4-011; Consent primitive (grant/revoke/revoke-wins) → CN-4-011; Checkout settlement mechanics → CN-5-009; Accounting cost-share auto-journal mappings → CN-5-001 §4 #13; Cash settlement of cost_share_receivable Obligations → CN-5-002 AR flow; channel adapter internals → Term 7 via CTR-019/020/021; pack content per jurisdiction → Term 1 via CTR-029; vertical-specific promotion triggers → Term 6 via CTR-030; tenant-customer-facing UX (voucher entry, loyalty card scan, customer history) → Term 3 (deferred per D-DISC-001); POS self-service flow → future cross-term (deferred per D-DISC-002).

---

## 1. Purpose & Boundary

CN-5-007 is the Promotion Engine — the **commerce-promotion truth layer** for any business reaching its end-customers. Discounts, vouchers, loyalty, bundles, time-based campaigns, customer-specific offers, cost-share funded campaigns — all unified under one engine that **populates inputs** (rules + voucher refs + loyalty redemptions) which **Checkout applies** at settlement (D-001 universal-checkout intent).

The engine spans full lifecycle: campaign definition → rule + voucher + loyalty setup → consent-gated outreach → application at customer purchase → cost-share recording → refund compensation. It uses Foundation primitives (Obligation for loyalty + cost-share receivables; Document for vouchers; Consent for outreach gating) and composes them — no new primitives invented.

### Scope (in)

- Commerce promotion (tenant → customer): discounts, vouchers, loyalty, bundles, BOGO, time-based, customer-specific, campaign-funded, manual overrides
- Cost-share split across 4 funding sources (bos / agent / supplier / tenant) with UI-10 enforcement
- Outreach over channels (consent-gated per D-008 / CTR-021)
- Tax treatment of discounts (pack-driven via `tax_treatment_ref`)
- Refund compensation (UI-02 symmetric reversal)
- Cross-engine event flows (Checkout / Accounting / Cash / Reporting / Channels)

### Scope (out)

- Agent acquisition strategy (trial days, subscription credits) — Term 1 + Term 2 territory
- Tenant-customer-facing UX (voucher entry, loyalty card display, customer history) — **Term 3, deferred per D-DISC-001**
- POS self-service flow (customer-operated POS) — **future cross-term, deferred per D-DISC-002**

---

## 2. Doctrine (PR1–PR7)

| # | Law | Source / why |
|---|-----|--------------|
| **PR1** | **Promotion populates rules; Checkout applies them.** Promotion never settles tender; Checkout never invents discounts. Promotion writes line-level `discount_refs` (CN-4-021) and emits bill-level `promotion.applied.v1` events; Checkout reads and applies at settlement per K2 of CN-5-009 (Checkout recomputes from authoritative inputs). | D-001 Universal Checkout intent; CN-4-021 discount_refs design; CN-5-009 K2 (Checkout never trusts cached values blindly). Single-source-of-truth at settlement. |
| **PR2** | **Vouchers ARE Documents (CN-4-012).** A voucher is hash-verified, immutable, template-versioned, pack-frozen. Not a new primitive — composes the Document primitive. Verifiable years later by hash; transferable per pack rules. | Per CN-5-006 N3 doctrine — formal artefacts deserve Document treatment. Same pattern as Statements, Invoices, POs. |
| **PR3** | **Loyalty balances ARE Obligations (CN-4-011, kind: `loyalty_credit`).** Not a new primitive — composes the Obligation primitive. Outstanding = current point balance; creditor = customer; debtor = tenant. Accrual increases outstanding; redemption reduces. UI-08 bounds. | Unifies with AR/AP/loans/delivery/advance/layby — all Obligations. CN-5-002 §G + CN-5-003 §13 + CN-5-005 §10 — same primitive everywhere. |
| **PR4** | **Cost-share splits into 4 funding sources summing to `total_discount` (UI-10).** `bos_share + agent_share + supplier_share + tenant_share = total_discount`. The first three create `cost_share_receivable` Obligations (CN-4-011 kind extension); `tenant_share` is absorbed (revenue reduction or expense per pack PR6). Settlement via Cash Engine (CN-5-002) AR flow — no new payment rail. | UI-10 (CN-5-102); D-001 cost-share intent; unified Obligation primitive. |
| **PR5** | **Outreach is consent-gated at command time.** Promotion broadcast commands without a valid Consent record (CN-4-011 Consent primitive) are **rejected**, not soft-warned. D-008 + CTR-021 govern channel-specific consent. | D-008; Law 5 (data law); Consent primitive non-bypassable; CN-4-011 Consent fold (revoke-wins). |
| **PR6** | **Tax treatment is pack-driven via `tax_treatment_ref` (CN-4-021 input).** Whether a discount reduces taxable base or not is jurisdiction-specific. Promotion writes the `tax_treatment_ref`; Checkout and Accounting resolve via pack. | Law 5; D-009 freeze; CN-4-021 input mechanism; IFRS-vs-local-rules divergence accommodated. |
| **PR7** | **Calculation precedence is pack-driven (region/tenant reorderable).** Default order: `line → bundle → bill → voucher → loyalty`. Pack declares; tenant may override within pack permission. Same inputs + same pack version → same result. | Replay determinism (CN-4-001); customer dispute resolution requires predictable computation; Law 5. |

---

## 3. Primitives Used

CN-5-007 composes Foundation primitives — no new primitives are invented. The table below enumerates each composition.

| Primitive | Used for | Pattern |
|-----------|----------|---------|
| **Obligation** (CN-4-011) | Loyalty point balance (kind: `loyalty_credit`); cost-share receivables for bos/agent/supplier (kind: `cost_share_receivable_<source>`); voucher breakage if pack defers expiry write-off as Obligation | Outstanding increases on accrual; reduces on redemption/settlement; UI-08 bounds enforced |
| **Document** (CN-4-012) | Vouchers (PR2) — issuance, hash, number, template version, pack-version freeze, terminal fold | Issued once; never modified; corrections via new linked Document (CN-4-012 §6) |
| **Party** (CN-4-011) | Customer profile for targeting (`target_customers`); customer reference on loyalty balance, voucher issuance, outreach delivery | Identity-only; engine projection adds customer-segment tags |
| **Item** (CN-4-011) | Target-item rules (`target_items`); bundle composition; loyalty earn basis per item attribute | Existing Item primitive; engine reads attributes |
| **Consent** (CN-4-011) | Outreach gating (PR5); per-customer × per-channel × per-purpose consent records | CN-4-011 Consent fold — revoke-wins; bus invokes at command time |
| **Decision Journal** (CN-4-013) | Manual override audit (manual_override kind, Q3) — records elevated principal, reason, original computed outcome, override outcome | CN-4-013 §3 — every override produces a journal entry |
| **Workflow** (CN-4-011) | Campaign lifecycle (drafted → active → deactivated); voucher state machine (issued → redeemed | expired | cancelled | restored | burned) | State machine event-sourced; truth = fold |

---

## 4. Engine Manifest

Promotion is a **multi-scope engine** per CN-5-101 §6: `tenant` default with explicit `site`-scope overrides for site-specific operations.

```yaml
engine_id:        promotion
display_name:     Promotion Engine
version:          1
scope_policy:     tenant                                # default — promotion definitions tenant-wide
primitives_used:
  - obligation                                          # loyalty + cost-share receivables
  - document                                            # vouchers
  - party                                               # customer profile + targeting
  - consent                                             # outreach gating
  - workflow                                            # campaign + voucher lifecycle

commands:
  # Campaign lifecycle (tenant)
  - id: promotion.campaign.create.request               scope_ref: tenant
  - id: promotion.campaign.activate.request             scope_ref: tenant
  - id: promotion.campaign.deactivate.request           scope_ref: tenant
  - id: promotion.campaign.amend.request                scope_ref: tenant

  # Rules (per campaign)
  - id: promotion.rule.define.request                   scope_ref: tenant
  - id: promotion.rule.update.request                   scope_ref: tenant
  - id: promotion.rule.deactivate.request               scope_ref: tenant

  # Vouchers (PR2)
  - id: promotion.voucher.issue.request                 # scope_ref: site (or tenant for campaign-batch)
  - id: promotion.voucher.redeem.request                # scope_ref: site (at checkout)
  - id: promotion.voucher.cancel.request                scope_ref: tenant
  - id: promotion.voucher.expire_run.request            scope_ref: tenant     # system:scheduler

  # Voucher restoration (N3 — pack-driven on refund)
  - id: promotion.voucher.restoration.request           # scope_ref: site (driven by refund choreography)

  # Loyalty (PR3)
  - id: promotion.loyalty.program.create.request        scope_ref: tenant
  - id: promotion.loyalty.points.accrue.request         # scope_ref: site (system, from checkout.settled)
  - id: promotion.loyalty.points.redeem.request         # scope_ref: site (at checkout)
  - id: promotion.loyalty.points.expire_run.request     scope_ref: tenant     # system:scheduler

  # Cost-share (PR4 / UI-10)
  - id: promotion.cost_share.declare.request            scope_ref: tenant     # campaign funding split
  - id: promotion.cost_share.record.request             # scope_ref: site (system, at application)

  # Manual override (Q3 — N1 catalogue item)
  - id: promotion.manual_override.apply.request         # scope_ref: site; elevated principal required (CN-4-007); reason_ref required; CN-4-013 journals

  # Outreach (PR5 / D-008)
  - id: promotion.outreach.send.request                 scope_ref: tenant     # consent-gated; bus rejects on absence

emits:
  # Campaigns
  - promotion.campaign.created.v1
  - promotion.campaign.activated.v1
  - promotion.campaign.deactivated.v1
  - promotion.campaign.amended.v1
  - promotion.rule.defined.v1
  - promotion.rule.updated.v1
  - promotion.rule.deactivated.v1

  # Vouchers (Documents per PR2)
  - promotion.voucher.issued.v1                         # Document terminal-fold (hash + number + template_version + pack_version_ref)
  - promotion.voucher.redeemed.v1
  - promotion.voucher.cancelled.v1
  - promotion.voucher.expired.v1                        # scheduler-triggered
  - promotion.voucher.restoration.requested.v1          # N3 — pack resolves to restored | burned | partial_credit
  - promotion.voucher.restored.v1                       # state returns to issued
  - promotion.voucher.burned.v1                         # state terminal; no restore
  - promotion.voucher.partial_credit.v1                 # partial value retained

  # Loyalty (Obligation events per PR3 — N2 namespace)
  - promotion.loyalty.program.created.v1
  - promotion.loyalty.points.accrued.v1                 # Obligation outstanding increases
  - promotion.loyalty.points.redeemed.v1                # Obligation outstanding decreases
  - promotion.loyalty.points.expired.v1                 # outstanding reduces (write-off path)

  # Cost-share (UI-10 / N4)
  - promotion.cost_share.declared.v1
  - promotion.cost_share.recorded.v1                    # CN-5-001 #13; UI-10 reconciliation; creates 3 cost_share_receivable Obligations

  # Application (read by Checkout)
  - promotion.applied.v1                                # batched per sale; consolidated via N1 handler

  # Manual override
  - promotion.manual_override.applied.v1                # CN-4-013 Decision Journal entry recorded

  # Outreach (PR5 — N2 namespace)
  - promotion.outreach.requested.v1                     # consent-verified; → Term 7 channel adapter
  - promotion.outreach.sent.v1                          # adapter confirmation
  - promotion.outreach.failed.v1
  - promotion.outreach.blocked.v1                       # consent absent — hard-block recorded

subscribes_to:
  # Checkout settlement — primary trigger (N1 consolidated handler)
  - {event_type: checkout.settled.v1,                   version: 1, handler: apply_promotions_at_settlement, kind: command_emitting, scope_ref: site}
    # Internal routing on promotion.kind:
    #   voucher_ref present     → emit promotion.voucher.redeemed.v1
    #   loyalty-eligible        → submit promotion.loyalty.points.accrue.request
    #   loyalty_redemption used → submit promotion.loyalty.points.redeem.request
    #   cost-share applicable   → submit promotion.cost_share.record.request per applied promotion
    # All emissions batched atomically per CN-4-004 §2; UI-01 causation chain preserved.

  - {event_type: checkout.refunded.v1,                  version: 1, handler: reverse_promotions_at_refund,   kind: compensation,     scope_ref: site}
    # UI-02 symmetric reversal: voucher restoration request, loyalty reversal, cost-share reversal — all batched atomically.

  # Term 7 channel adapter outcomes
  - {event_type: <channel-adapter>.outreach.outcome.v1, version: 1, handler: record_outreach_outcome,       kind: command_emitting, scope_ref: tenant}

  # Vertical promotion triggers (CTR-030 expansion)
  - {event_type: <vertical>.promotion_trigger.v1,       version: 1, handler: ingest_vertical_trigger,        kind: command_emitting, scope_ref: site}

  # Cash settlement of cost-share receivables (N4 — promotion tracks workflow status)
  - {event_type: cash.collection.received.v1,           version: 1, handler: mark_cost_share_settled,        kind: command_emitting, scope_ref: tenant}
    # Filters on payload.obligation_ref.kind starts with cost_share_receivable_*

# Pack hooks read at command time (no separate subscription; bus consults pack via CN-4-015):
pack_hooks_consumed:
  - pack.promotion.stacking_policy                      # best_only (default) | all_eligible | custom_precedence
  - pack.promotion.calculation_precedence               # ordered list of kinds; PR7 default [line, bundle, bill, voucher, loyalty]
  - pack.promotion.voucher_restoration_policy           # restored | burned | partial_credit (N3)
  - pack.promotion.voucher_namespace                    # tenant (default) | campaign
  - pack.promotion.voucher_breakage_account             # GL ref for expired-voucher writeback (Q9)
  - pack.loyalty.earn_basis                             # bill_total (default) | line_net | item_attribute (Q4)
  - pack.promotion.cost_share_cadence                   # monthly_net (default) | weekly_net | immediate (Q6)

requires:
  - checkout.settled.v1                                 # cannot apply promotions without settlement events
```

---

## 5. Promotion Catalogue (10 Kinds)

| # | Kind | Definition | Where applied |
|---|------|------------|---------------|
| 1 | **line_discount** | Discount on a specific line (item × qty) — % or fixed | CN-4-021 `discount_refs` on the line; Checkout K2 recomputes |
| 2 | **bill_discount** | Discount on bill total (e.g., "10% off > TZS 50k") | `promotion.applied.v1`; Checkout reads after line aggregation |
| 3 | **bogo** | Buy-One-Get-One (free or discounted) | Line-level via rule on items; Promotion injects extra discount_ref |
| 4 | **bundle** | Set of items priced together; **atomic — all components present or no discount** (Q10) | Line-level discount_refs across bundle's lines + bundle metadata; Checkout verifies all components present before applying |
| 5 | **voucher** | Discrete redeemable Document (PR2); presented at checkout | Voucher redemption consumed; discount per voucher's encoded value |
| 6 | **loyalty_redemption** | Customer redeems loyalty points (PR3) for discount or free items | Loyalty Obligation reduces; discount applied |
| 7 | **time_based** | Active only within time window (happy hour, weekend) | Rule with `time_window`; engine evaluates at application time |
| 8 | **customer_specific** | Targeted at specific customer or segment | Rule with `target_customers`; verified at application |
| 9 | **campaign_funded** | BOS / agent / supplier funds part or all of the discount (PR4) | Cost-share split declared; recorded per UI-10 |
| 10 | **manual_override** | Cashier ad-hoc adjustment (Q3) | Requires `reason_ref` + elevated principal per CN-4-007; fully audited via CN-4-013 Decision Journal |

Combinations are common (e.g., a campaign that runs as "BOGO during happy hour for loyalty members" = `bogo` + `time_based` + `customer_specific`).

### Manual Override — Audit Requirements

`promotion.manual_override.apply.request` carries:
- `original_computed_outcome` (what the rule engine would have produced)
- `override_outcome` (cashier's adjusted figure)
- `reason_ref` (pack-defined enum + free-form text)
- `elevated_principal` (CN-4-007 — manager / owner role required; bus rejects others)

The override emits `promotion.manual_override.applied.v1` AND a Decision Journal entry (CN-4-013 §3) capturing all five fields. Reporting may flag overrides for review.

---

## 6. Targeting (4 Orthogonal Dimensions — Read-Only Resolution per N5)

Rules combine any subset of four dimensions. All resolution is **read-side** — Promotion reads Party / Item / Site projections + wall-clock (CN-4-014); never sends inter-engine commands.

```yaml
promotion.rule.define.request:
  payload:
    rule_id:                 <unique>
    campaign_ref:            <campaign>
    target_customers:        ["all"] | [<party_refs>] | <segment_filter>
    target_items:            ["all"] | [<item_refs>] | <category_filter>
    target_sites:            ["all"] | [<site_ids>]
    time_window:
      from:                  <business_date>
      until:                 <business_date>
      days_of_week:          [Mon, Tue, ...]
      hours:                 {from: "17:00", until: "20:00"}      # happy hour
    discount:
      kind:                  percent | fixed | bogo | bundle | free_item
      value:                 <decimal | item_ref>
      max_per_sale:          <decimal>                             # optional cap
    stacking:                stackable | exclusive | exclusive_with: [<rule_ids>]
    cost_share_ref:          <cost-share declaration>
    priority:                <integer; for precedence per PR7>
```

### Resolution at Application

When `apply_promotions_at_settlement` runs:

1. **Read** Party primitive for customer (segment, history) — read-only projection lookup
2. **Read** Item primitive for line items (categories, attributes) — read-only
3. **Match** sale's `site_id` (CN-5-101 L4) against `target_sites`
4. **Evaluate** wall-clock against `time_window` via injected clock (CN-4-014)
5. **Filter** rules matching all four dimensions
6. **Compute** discounts per rule's `discount` block
7. **Resolve** stacking per pack (§9)

No commands sent to other engines for targeting evaluation. Read-only by design (PR1 + N5).

---

## 7. Pack Hooks

Per CN-4-015 (Compliance DSL), pack declares rules consumed at command time. Promotion-relevant hooks (CTR-029 expansion):

```yaml
# In pack content
promotion:
  stacking_policy:                best_only           # best_only (default) | all_eligible | custom_precedence
  calculation_precedence:                              # ordered list; PR7 default
    - line_discount
    - bundle
    - bill_discount
    - voucher
    - loyalty_redemption
  voucher_restoration_policy:     restored             # restored | burned | partial_credit (N3)
  voucher_namespace:              tenant               # tenant (default) | campaign
  voucher_breakage_account:       "6730"               # GL ref for expired-voucher writeback (Q9)
  cost_share_cadence:             monthly_net          # monthly_net (default) | weekly_net | immediate (Q6)

loyalty:
  earn_basis:                     bill_total           # bill_total (default) | line_net | item_attribute (Q4)
```

Pack rotation governs changes (D-009 freeze): historical promotion applications retain their pack-version interpretation forever.

---

## 8. Calculation Precedence (PR7)

Pack's `calculation_precedence` declares the order. Default sequence (applies left → right):

```
1. line_discount      (most specific — per-line item discounts)
2. bundle             (multi-line atomic bundling)
3. bill_discount      (bill-total promotions)
4. voucher            (voucher value reduces post-discount subtotal)
5. loyalty_redemption (loyalty points reduce final tender owed)
```

At each step, eligible promotions of that kind are computed; result feeds the next step. Tenant may reorder within pack permission via `pack.promotion.calculation_precedence` override.

### Determinism

Same inputs (sale, customer profile, item categories, time, eligible rules) + same pack version + same precedence list → same result, every replay. PR7 guarantees replay-determinism; CN-4-001 inherits.

---

## 9. Stacking Policy (Q2)

Pack declares `pack.promotion.stacking_policy`:

| Value | Meaning |
|-------|---------|
| **best_only** (default) | When multiple promotions match the same target (item × customer × time), only the one producing the highest discount applies. Others are skipped. |
| **all_eligible** | All matching promotions apply; total discount = sum of all eligible. |
| **custom_precedence** | Pack provides per-conflict resolution rules (e.g., "voucher beats loyalty"; "BOGO beats fixed discount") |

Rule-level `stacking: stackable | exclusive | exclusive_with: [<rule_ids>]` overrides per-rule (within pack permission).

### Conflict Resolution at Application

The handler evaluates pack stacking_policy + rule exclusivity together:
- Group eligible rules by target
- Apply `exclusive_with` constraints first (these are hard exclusivity)
- Within remaining group, apply pack policy (best_only / all_eligible / custom)

Result is deterministic given same inputs.

---

## 10. Voucher Lifecycle (PR2 — Documents)

Vouchers are formal Documents (CN-4-012). Each issuance is hashed, numbered, template-versioned, pack-frozen.

### State Machine

```
[issued (Document)] ──redeem──► [redeemed]
                   │
                   ├──expire──► [expired]
                   │
                   ├──cancel──► [cancelled]
                   │
                   └──refund_triggers_restoration──► [pending_restoration]
                                                  → (per pack N3) restored | burned | partial_credit
```

### Issuance

```yaml
promotion.voucher.issue.request:
  scope:           site | tenant
  payload:
    voucher_code:        <unique within pack.promotion.voucher_namespace (Q5 — tenant or campaign)>
    discount_kind:       percent | fixed | free_item | tier
    discount_value:      <decimal | item_ref>
    eligibility:         {target_items, target_sites, customer_ref?, time_window?}
    expiry_date:         <business_date>
    transferable:        boolean (per pack)
    issuance_template_ref: <pack template>
```

Emits `promotion.voucher.issued.v1` carrying full Document fields (hash, number per pack format + namespace, template_version, pack_version_ref).

### Voucher Namespace (Q5)

Per `pack.promotion.voucher_namespace`:
- **tenant** (default): voucher codes unique across tenant (e.g., `VCH-MA-2026-10-0001`)
- **campaign**: voucher codes unique within campaign only (e.g., `OCT-SODA-0001`); useful for campaign-batch issuance with parallel numbering

### Redemption (at Checkout)

Customer presents voucher (code, scan, or auto-detect via Party profile per D-DISC-001 future). Checkout settle flow detects `voucher_ref` in payload; Promotion's `apply_promotions_at_settlement` handler emits `promotion.voucher.redeemed.v1` (single-use → state: redeemed; multi-use → remaining_uses decrement).

### Expiration

Scheduler runs `promotion.voucher.expire_run.request` per pack frequency (typically daily); for vouchers with `expiry_date ≤ current_business_date AND state = issued`, emits `promotion.voucher.expired.v1`.

### Voucher Breakage Account (Q9)

When vouchers expire unused, the unredeemed value writebacks to a GL account per `pack.promotion.voucher_breakage_account`. Accounting subscribes; auto-journals: Dr Voucher Liability (the loyalty-style obligation pre-redemption — if pack defers redemption as deferred revenue); Cr Breakage Income (account per pack ref). Pack rules govern whether breakage is recognised gradually (per historical redemption rate) or at expiry (full).

### Restoration (N3 — Refund-Driven)

On `checkout.refunded.v1` with redeemed voucher reference, Promotion emits `promotion.voucher.restoration.requested.v1`. Pack resolves via `pack.promotion.voucher_restoration_policy`:

| Policy | Effect | Engine emission |
|--------|--------|------------------|
| **restored** | Voucher returns to `issued` state; customer may use again before expiry | `promotion.voucher.restored.v1` |
| **burned** | Voucher remains terminal; no restore | `promotion.voucher.burned.v1` |
| **partial_credit** | Voucher's residual value retained per refund proportion | `promotion.voucher.partial_credit.v1` (carries `remaining_value`) |

UI-02 compensation symmetry: every engine that posted on original (Accounting cost-share, Cash flow) reverses appropriately for the refunded portion regardless of voucher resolution.

---

## 11. Loyalty Model (PR3 — Obligation, kind: `loyalty_credit`)

```yaml
# Loyalty program definition
promotion.loyalty.program.create.request:
  scope:           tenant
  payload:
    program_id:                <unique>
    earn_basis:                <per pack.loyalty.earn_basis: bill_total | line_net | item_attribute>
    accrual_rules:             {per_currency_unit: 1 point per TZS 100; bonus_items: [{item, multiplier}]}
    redemption_rules:          {points_per_currency_unit: 100 points = TZS 1; min_redemption: 500}
    expiry_policy:             {points_expire_after_days: 365; expiry_method: oldest_first}
    customer_eligibility:      [<segment_filter>]
```

### Earn Basis (Q4)

Per `pack.loyalty.earn_basis`:
- **bill_total** (default): points = bill grand_total × accrual rate
- **line_net**: points = sum(line × accrual rate) per line — useful when some items exclude loyalty
- **item_attribute**: points = sum(line × item.loyalty_multiplier) — per-item bonus tiers

### Obligation Lifecycle

```
At sale (checkout.settled.v1):
  apply_promotions_at_settlement detects loyalty-eligible
  Submit promotion.loyalty.points.accrue.request
  → Obligation (kind: loyalty_credit) updated:
      outstanding += points_earned
      currency: <points unit, pack-defined>
  → Emit promotion.loyalty.points.accrued.v1

At redemption (next checkout.settled.v1 with loyalty_redemption tender):
  Submit promotion.loyalty.points.redeem.request
  → Obligation outstanding -= points_spent (UI-08: ≥ 0)
  → Discount applied to settlement

At expiry (scheduler):
  promotion.loyalty.points.expire_run.request
  → Per expiring tranche per program's expiry_method:
    Obligation compensation reduces outstanding
    Emit promotion.loyalty.points.expired.v1
```

### Refund Compensation

`checkout.refunded.v1` triggers Promotion's `reverse_promotions_at_refund`:
- Submit Obligation compensation reducing outstanding by accrued points × refund_proportion
- Emit `promotion.loyalty.points.accrued.v1` compensating event (`compensates_event_id` to original accrual)
- UI-08 bound check: outstanding cannot go negative; if customer already redeemed > refund-eligible accrual, hard-block (pack may opt for partial-recovery rules)

---

## 12. Cost-Share & Funding Sources (PR4 / UI-10 / N4)

### 4 Funding Sources

| Source | Meaning | Obligation created? | Settlement |
|--------|---------|---------------------|------------|
| **bos_share** | BOS platform funds (campaign sponsorship) | Yes — `cost_share_receivable_bos` Obligation | Term 1 billing offset; Cash via AR flow per cadence |
| **agent_share** | Regional agent contributes (agent acquisition incentive); per CTR-024 site_id determines responsible agent | Yes — `cost_share_receivable_agent` Obligation | Cash via AR flow per cadence |
| **supplier_share** | Supplier funds the discount (supplier rebate) | Yes — `cost_share_receivable_supplier` Obligation | Cash via AR flow OR AP credit against supplier Obligation per pack |
| **tenant_share** | Tenant absorbs cost | **No Obligation** — direct revenue reduction OR expense per pack PR6 | Recognised immediately in P&L |

### Declaration

```yaml
promotion.cost_share.declare.request:
  scope:           tenant
  payload:
    campaign_ref:        <campaign>
    funding_split:
      bos_share_percent:        50
      agent_share_percent:      20
      supplier_share_percent:   15
      tenant_share_percent:     15
    settlement_cadence:  <per pack.promotion.cost_share_cadence: monthly_net | weekly_net | immediate (Q6)>
```

### Recording at Application (UI-10)

Per promotion application (each `checkout.settled.v1` carrying this promotion):

```yaml
promotion.cost_share.recorded.v1:
  payload:
    campaign_ref, sale_ref:        <checkout.settled event_id>
    total_discount:                TZS 5000
    bos_share:                     TZS 2500
    agent_share:                   TZS 1000
    supplier_share:                TZS 750
    tenant_share:                  TZS 750
    pack_version_ref:              <D-009>
    obligations_created:
      - {kind: cost_share_receivable_bos,      amount: 2500, debtor_party: bos_platform}
      - {kind: cost_share_receivable_agent,    amount: 1000, debtor_party: <agent_party from site>}
      - {kind: cost_share_receivable_supplier, amount: 750,  debtor_party: <supplier_party from campaign>}
```

**UI-10 enforces:** `bos_share + agent_share + supplier_share + tenant_share = total_discount`. CN-5-102 integration DC verifies across sampled events.

### CN-5-001 #13 Auto-Journal

Accounting subscribes `promotion.cost_share.recorded.v1`:
- Dr Revenue or Promotion Expense (`tenant_share` per pack PR6)
- Dr Cost-Share Receivable BOS (`bos_share`) — creates Obligation
- Dr Cost-Share Receivable Agent (`agent_share`) — creates Obligation
- Dr Cost-Share Receivable Supplier (`supplier_share`) — creates Obligation
- Cr Revenue (`total_discount` reduction in revenue per pack treatment)

Per CTR-030, vertical-specific cost-share GL account mapping (e.g., workshop projects use different cost centers than retail) is pack content.

### Settlement (N4)

Per `pack.promotion.cost_share_cadence`:
- **monthly_net** (default): aggregate by source, settle once per month via Cash collection
- **weekly_net**: weekly
- **immediate**: settle per-sale (rare; only for high-trust supplier rebates)

When Cash receives settlement (`cash.collection.received.v1` with `obligation_ref.kind = cost_share_receivable_*`), Promotion's `mark_cost_share_settled` handler updates workflow status and emits status events. No new payment rail — Cash AR flow per CN-5-002 §G.

---

## 13. Outreach (PR5 / CTR-021 / D-008)

Promotion broadcast over channels requires recorded consent. Bus invokes Consent primitive (CN-4-011) per target customer × channel × purpose combination.

### Send Flow

```yaml
promotion.outreach.send.request:
  scope:           tenant
  payload:
    campaign_ref:        <campaign>
    target_customers:    [<party_refs>] | <segment_filter>
    channel:             sms | whatsapp | telegram | email          # per registered Term 7 adapter
    message_template_ref: <pack-defined template>
    scheduled_send_at:   <timestamp> | "immediate"
    purpose:             promotion_outreach
```

### Consent Verification — Hard-Block (PR5)

Bus consults Consent primitive per `(customer_ref, channel, purpose: promotion_outreach)`:
- **Consent granted, not revoked** → proceed; emit `promotion.outreach.requested.v1` per customer
- **Consent absent or revoked** → reject the command for that customer:
  ```
  rejected_by_policy:  PR5.outreach_requires_consent
  rejection_reason:    "Customer <ref> has no valid Consent for promotion_outreach via <channel>"
  ```
  Emit `promotion.outreach.blocked.v1` for audit (records the attempt and the block reason).

### Delivery Boundary (Per CN-5-006 N5)

`promotion.outreach.requested.v1` → Term 7 channel adapter (CTR-019 / CTR-020 / CTR-021 mechanism) consumes → invokes external channel (SMS gateway, WhatsApp Business API, etc.) → emits `<channel-adapter>.outreach.outcome.v1` back.

Promotion's `record_outreach_outcome` handler emits `promotion.outreach.sent.v1` or `promotion.outreach.failed.v1`.

**Promotion does NOT deliver itself**; Term 7 owns the medium per Law 2 isolation.

### Opt-Out Honoured Immediately

If customer revokes Consent (Consent primitive fold — revoke-wins), subsequent outreach commands targeting that customer are blocked immediately. No grace period. Bus enforces at command time.

---

## 14. Tax Treatment (PR6 — `tax_treatment_ref` Input)

Promotion populates `tax_treatment_ref` on `discount_refs` (CN-4-021); Checkout and Accounting resolve via pack:

```yaml
# In pack content (CTR-029 expansion)
promotion_tax_treatment:
  default:
    discount_reduces_taxable_base:  true        # TZ/most-IFRS — VAT computed on net
  per_kind_overrides:
    - {discount_kind: "bos_funded_campaign",   treatment: gross_with_separate_adjustment}
  voucher_tax_treatment:
    issuance:                       no_vat_at_issuance
    redemption:                     vat_on_redemption_value
  loyalty_tax_treatment:
    accrual:                        no_tax_at_accrual
    redemption:                     vat_on_redemption_value
```

At Checkout K3 (CN-5-009):
- If `discount_reduces_taxable_base: true` → tax computed on net amount after discount
- If false → tax on gross; discount as separate revenue/expense adjustment per pack

CN-5-001 #1 (sale auto-journal) and #13 (cost-share auto-journal) reflect the pack-driven treatment.

---

## 15. Refund Compensation (UI-02 Symmetry)

When `checkout.refunded.v1` arrives, Promotion's `reverse_promotions_at_refund` handler (kind: compensation) propagates symmetrically. All emissions batched atomically per CN-4-004 §2.

### Per Effect, Per Compensation

| Original effect | Compensation |
|-----------------|--------------|
| Loyalty points accrued | Obligation outstanding reduces by accrued × refund_proportion; emit `promotion.loyalty.points.accrued.v1` compensating event |
| Voucher redeemed | Emit `promotion.voucher.restoration.requested.v1`; pack resolves (N3) to restored / burned / partial_credit |
| Cost-share recorded | Emit `promotion.cost_share.recorded.v1` compensating event with all 4 shares reversed (`compensates_event_id` to original); CN-5-001 #13 subscribes (kind: compensation) and reverses journals; the 3 `cost_share_receivable_*` Obligations reduce |
| Promotion applied | Emit `promotion.applied.v1` compensating event referencing original |

### Partial Refund (Q7)

For a refund of subset of original lines:
- Loyalty: reverse points by `original_points × (refunded_value / original_value)`
- Cost-share: reverse 4 shares proportionally; UI-10 invariant holds at the proportional level
- Voucher: pack-driven (typically all-or-none — restoration policy applies to whole voucher)
- Bundle (Q10): **bundles are atomic** — refund of one component invalidates bundle pricing; the full bundle discount reverses; remaining items revert to their non-bundle line prices (recomputed at refund)

UI-02 symmetry: every engine that posted on original (Accounting cost-share journal, Cash receipt, Inventory restoration) reverses appropriately.

---

## 16. Cross-Engine Subscriptions

### Promotion Emits → Subscribed By

| Promotion event | Subscribed by | Effect |
|-----------------|---------------|--------|
| `promotion.cost_share.recorded.v1` | CN-5-001 #13 Accounting | Auto-journal cost-share split; creates 3 receivable Obligations |
| `promotion.voucher.issued.v1` (Document) | CN-5-006 Reporting | Voucher metrics; available-vouchers projection |
| `promotion.voucher.redeemed.v1` | CN-5-006 Reporting + CN-5-002 (if voucher acts as tender per pack) | Analytics + possible Cash recording |
| `promotion.voucher.expired.v1` | CN-5-001 (if breakage recognition active) + CN-5-006 | Pack-driven breakage journal + metrics |
| `promotion.loyalty.points.accrued.v1` | CN-5-006 Reporting | Loyalty program metrics |
| `promotion.loyalty.points.redeemed.v1` | CN-5-002 (if redemption acts as tender per pack) + CN-5-006 | Cash tender path + analytics |
| `promotion.outreach.sent.v1` | CN-5-006 Reporting | Outreach metrics, conversion analytics |
| `promotion.applied.v1` | CN-5-006 Reporting | Promotion ROI analytics |
| `promotion.manual_override.applied.v1` | CN-4-013 Decision Journal + CN-5-006 | Override audit + reporting flag |

### Promotion Subscribes To

| Source event | Handler | Effect |
|--------------|---------|--------|
| `checkout.settled.v1` | `apply_promotions_at_settlement` (N1 consolidated) | Internal routing: voucher / loyalty accrual / cost-share / manual override — batched atomic emission |
| `checkout.refunded.v1` (kind: compensation) | `reverse_promotions_at_refund` | UI-02 symmetric reversal — batched atomic |
| `<channel-adapter>.outreach.outcome.v1` | `record_outreach_outcome` | Delivery confirmation |
| `<vertical>.promotion_trigger.v1` | `ingest_vertical_trigger` | Vertical-specific (CTR-030 — happy hour, room rate, etc.) |
| `cash.collection.received.v1` (filtered on `cost_share_receivable_*`) | `mark_cost_share_settled` | Workflow status update on cost-share Obligation settlement |

### Channel Adapters (Term 7)

`promotion.outreach.requested.v1` is consumed by registered Term 7 channel adapters per the CTR-021 mechanism. Each adapter declares which channel it serves; Promotion's request carries `channel` field for routing. Term 7 owns the medium; Promotion owns the gating.

---

## 17. UI Invariants Touched

| Invariant | Application |
|-----------|-------------|
| **UI-01** (causation chain) | Every promotion meta-event chains back to triggering event. N1 atomic batches preserve causation; consolidated `apply_promotions_at_settlement` handler ensures all derived events carry `causation_id` to `checkout.settled.v1`. |
| **UI-02** (compensation symmetry) | Refund triggers atomic reversal batch (§15). Each engine that posted on original posts symmetric reverse. Partial refunds reverse proportionally; bundles atomic. |
| **UI-08** (Obligation bounds) | Loyalty `outstanding ≥ 0` (never negative); cost_share_receivable Obligations bounded `[0, original_amount]`. Bus rejects redemptions that would breach loyalty bound. Settlement reduces receivables per UI-08. |
| **UI-10** (cost-share reconciliation) | `bos_share + agent_share + supplier_share + tenant_share = total_discount` per event. CN-5-102 Phase 1 DC verifies. |

---

## 18. Failure Modes & Compensation

| Failure | Effect | Recovery |
|---------|--------|----------|
| **Consent revoked between scheduled outreach declaration and send time** | Bus blocks at send command; `promotion.outreach.blocked.v1` recorded | Customer excluded from this send; future sends require re-consent |
| **Voucher redeemed twice** (race condition) | Bus rejects second redemption via Document terminal-fold guarantee (CN-4-011 + CN-4-012 — voucher state already redeemed) | First redemption wins; second customer informed |
| **Loyalty redemption exceeds outstanding** (race condition or stale projection) | Bus rejects per UI-08 bound (`outstanding ≥ 0`) | Customer informed; partial redemption possible if amount > min_redemption |
| **Cost-share recipient settlement fails** (BOS billing offset, agent insolvent) | Obligation outstanding remains; Cash settlement not received | Cost-share write-off via compensating Obligation event; pack rules govern (typically Term 1 governance for BOS share; agent contract for agent share) |
| **Bundle component out of stock at checkout** | Bundle eligibility check fails at apply_promotions_at_settlement | Bundle not applied; constituent items revert to non-bundle line prices; emit `promotion.applied.v1` with `bundle_skipped: true` |
| **Manual override without elevated principal** | Bus rejects per CN-4-007 identity check | Cashier must escalate; manager submits with elevated identity |
| **Channel adapter outcome never arrives** (Term 7 outage) | `promotion.outreach.requested.v1` remains in flight; no follow-up event | After timeout (pack-defined), Promotion emits `promotion.outreach.failed.v1` with reason `adapter_timeout`; retry per pack policy |
| **Pack rotation mid-campaign** | Active campaign continues under pre-rotation pack version (D-009 freeze); new rules take effect for sales after rotation | No silent re-computation; campaign may be amended to align if tenant chooses |

---

## 19. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| **D-DISC-001 — Tenant-customer promotion application + dashboard visibility** | Term 3 (CN-3-*) | **Deferred per Concept Lead.** How customers apply promotions (voucher code entry, phone auto-detect, loyalty card scan) and see loyalty balance / available promotions / redemption history on customer-facing surfaces. CN-5-007 publishes the read surface (Reporting projections + voucher Documents + Obligation state); Term 3 designs the UX. To be picked up when Term 3 starts work on tenant-customer-facing surfaces. |
| **D-DISC-002 — POS self-service expansion** | Future cross-term (Term 5/6) | **Deferred per Concept Lead.** Customer-operated POS (kiosk ordering, self-checkout). Started in café/restaurant/hotel; framework should expand to any business needing it. Architectural framing: self-service is a **mode of operation** on existing engines, not a new engine — customer-actor identity (CN-4-007), bus enforcement, UI-04 chain still hold. To be picked up after Term 5 universal engines complete and Term 6 begins. |
| Pack content for stacking policy, calculation precedence, voucher restoration, voucher namespace, voucher breakage account, loyalty earn basis, cost-share cadence per jurisdiction | Term 1 via CTR-029 expansion | Schema in §7; per-jurisdiction content is Term 1's authorship |
| Vertical-specific promotion triggers (restaurant happy hour, hotel room-rate discount, etc.) + cost-share GL account mapping per vertical | Term 6 via CTR-030 expansion | Per-vertical contracts |
| Outreach over channels — consent flow + Term 7 adapter contracts | CTR-021 (already OPEN) | Promotion §13 operationalises; Term 7 finalises adapter shape |
| Bundle-component invalidation rules (when bundle is partially redeemable / partially substitutable) | Pack content + future decision | Default: atomic (Q10); some retail patterns may need partial-bundle flexibility |
| Loyalty point tiers / multipliers per customer segment | Pack content + future tenant policy | Beyond basic earn-basis; future enhancement |
| Manual override threshold limits (max override amount per role) | Term 1 governance + pack | Default: elevated principal required; per-tenant amount limits deferred |
| Performance budget for stacking evaluation at scale (many concurrent rules) | Architect phase | Concept guarantees determinism; performance is Architect's |
| Voucher transferability mechanism (when transferable per pack — how ownership changes are tracked) | CN-4-012 + future decision | Document supports terminal-fold; transfer = new Document referencing original |
| Phase 1 doctrine check for UI-10 (cost-share reconciliation) | CN-4-019 + future CTR | Aligns with CN-5-102 §7 Phase 1 ramp-up; CN-5-007 ratification triggers DC addition |
| Phase 1 doctrine check for UI-08 across loyalty + cost-share Obligations | CN-4-019 + future CTR | Arrives with CN-5-007 ratification |

---

## 20. Worked Example — Mama Amina October Soda Promotion

*Mama Amina's duka serves as illustrative context per D-004 #4 (peer-technical audience). The example walks the full lifecycle: campaign definition → vouchers → loyalty bonus → outreach → customer redemption → cost-share settlement → partial refund → voucher expiry.*

**Setting:** Mama Amina's duka (2 sites: Mwanza, Mbeya). Tenant functional currency TZS. Pack `tz-compliance-2026.07`. October "Soda Festival" — 20% off Coca-Cola 500ml + 100 bonus loyalty points per qualifying purchase. BOS funds 50% of discount; tenant absorbs 50%. Agent share 0%; supplier share 0% (this campaign).

### Phase 1 — Campaign Setup (September 28)

```
promotion.campaign.create.request {name: "Soda Festival Oct 2026", campaign_id: SFEST-2026-10}
  → promotion.campaign.created.v1

promotion.rule.define.request:
  rule_id:                 SFEST-rule-001
  campaign_ref:            SFEST-2026-10
  target_customers:        ["all"]                              # available to anyone
  target_items:            [coca-cola-500ml]
  target_sites:            ["all"]                              # Mwanza + Mbeya
  time_window:             {from: 2026-10-01, until: 2026-10-31}
  discount:                {kind: percent, value: 20}
  loyalty_bonus:           100 points per qualifying purchase   # combined with base earn_basis
  stacking:                stackable
  → promotion.rule.defined.v1

promotion.cost_share.declare.request:
  campaign_ref:            SFEST-2026-10
  funding_split:           {bos_share: 50%, agent_share: 0%, supplier_share: 0%, tenant_share: 50%}
  settlement_cadence:      monthly_net                          # per pack default
  → promotion.cost_share.declared.v1

promotion.campaign.activate.request → promotion.campaign.activated.v1
```

### Phase 2 — Voucher Issuance (October 1)

Mama Amina issues 50 vouchers for distribution to top loyalty customers:

```
For each voucher (50 in batch):
  promotion.voucher.issue.request:
    voucher_code:         per pack format VCH-MA-2026-10-{seq:04}
    discount_kind:        fixed
    discount_value:       TZS 1000
    eligibility:          {target_items: [<any soda>], expiry_date: 2026-10-31}
    transferable:         false                                 # per pack
    issuance_template_ref: tz-voucher-template-v2
  → promotion.voucher.issued.v1 with Document fields:
      voucher_code:        VCH-MA-2026-10-0001 (per pack.promotion.voucher_namespace = tenant)
      document_hash:       <CN-4-012 hash>
      template_version:    v2
      pack_version_ref:    tz-compliance-2026.07
```

Each voucher is a Document — verifiable in 2046 by recomputing hash from stored content + references.

### Phase 3 — Outreach (October 1)

```
promotion.outreach.send.request:
  campaign_ref:         SFEST-2026-10
  target_customers:     <loyalty_segment of 100 customers>
  channel:              sms
  message_template_ref: oct-soda-promo-sms-v1
  purpose:              promotion_outreach

Bus invokes Consent primitive per (customer, sms, promotion_outreach):
  87 customers consented ✓
  13 customers no consent → blocked
    → 13 × promotion.outreach.blocked.v1 (audit records)

87 × promotion.outreach.requested.v1 → Term 7 SMS adapter handles
  → 85 successful, 2 failed
85 × promotion.outreach.sent.v1
2 × promotion.outreach.failed.v1
```

### Phase 4 — Customer Sale (October 15, Mwanza site)

Customer (loyalty member, party-ref `cust-amos-001`) buys 3 × Coca-Cola TZS 1,500 each = TZS 4,500 gross.

```
Checkout's bill construction (via Retail vertical) populates discount_refs from Promotion rule SFEST-rule-001:
  line discount_refs include {rule_ref: SFEST-rule-001, kind: percent, value: 20}

checkout.settled.v1 emitted:
  payload:
    lines:                [Coca-Cola × 3 with discount_refs]
    pre_tax_total:        TZS 3,600 (4,500 × 0.80)
    tax_breakdown:        [...]                                # per pack PR6 — discount reduces taxable base
    tax_total:            TZS 648 (3,600 × 18%)
    grand_total:          TZS 4,248
    tenders:              [TZS 4,248 cash]

Promotion's apply_promotions_at_settlement (N1 consolidated) routes:
  - SFEST-rule-001 matched → discount applied (handled by Checkout)
  - Loyalty-eligible customer → submit promotion.loyalty.points.accrue.request
      base: 4,248 × (pack.loyalty.earn_basis = bill_total) × accrual_rate (1 point per TZS 100) = 42 points
      bonus: 100 points (from SFEST rule loyalty_bonus)
      total: 142 points
    → promotion.loyalty.points.accrued.v1
      Obligation (loyalty_credit for cust-amos-001) outstanding: previous + 142
  - Cost-share applicable → submit promotion.cost_share.record.request
      total_discount: TZS 900 (20% × 4,500)
      bos_share: 450, tenant_share: 450, agent_share: 0, supplier_share: 0
    → promotion.cost_share.recorded.v1
      UI-10 check: 450 + 0 + 0 + 450 = 900 ✓
      Creates cost_share_receivable_bos Obligation (outstanding 450)
  - All emissions batched atomically per CN-4-004 §2

CN-5-001 #13 auto-journal:
  Dr Cost-Share Receivable BOS    450
  Dr Revenue                       450     (tenant_share absorbed)
  Cr Revenue                       900     (total discount as revenue reduction per pack)
```

### Phase 5 — Partial Refund (October 20)

Customer returns 1 of 3 Coca-Colas. Refund 1/3 of original.

```
checkout.refunded.v1:
  refund_kind:    partial
  lines_refunded: [Coca-Cola × 1]
  refund_total:   TZS 1,416 (4,248 / 3)
  compensates_event_id: <original checkout.settled.v1>

Promotion's reverse_promotions_at_refund (kind: compensation) — atomic batch:
  - Loyalty reversal: 142 × (1/3) = 47 points
    → promotion.loyalty.points.accrued.v1 compensating event
      Obligation outstanding reduces by 47 (UI-08 ≥ 0 check)
  - Cost-share reversal: 900 × (1/3) = 300 total reversal
    bos_share reversal: 150; tenant_share reversal: 150
    → promotion.cost_share.recorded.v1 compensating event
      cost_share_receivable_bos Obligation reduces by 150
    UI-10 holds at proportional level: 150 + 0 + 0 + 150 = 300 ✓
  - No voucher involved this sale; no restoration request
  - promotion.applied.v1 compensating event referencing original

CN-5-001 #13 compensating journal: reverse the original entries proportionally
```

### Phase 6 — Cost-Share Settlement (End of October)

Per `pack.promotion.cost_share_cadence = monthly_net`:

```
End-of-month aggregation: all cost_share_receivable_bos Obligations summed for October
  Total accumulated: TZS 12,500 (sum across all SFEST applications minus refund reversals)

Term 1 billing run picks up the aggregated Obligation:
  Credits Mama Amina's November subscription invoice by TZS 12,500
  Issues cash.collection.received.v1 against cost_share_receivable_bos Obligations
    → Obligations transition to settled
    → Promotion's mark_cost_share_settled handler emits workflow status
```

### Phase 7 — Voucher Expiry (November 1)

```
system:scheduler submits promotion.voucher.expire_run.request

For 12 unredeemed vouchers (out of 50 issued):
  promotion.voucher.expired.v1 × 12

Per pack.promotion.voucher_breakage_account:
  Accounting subscribes; breakage recognised per pack rules:
    Dr Voucher Liability (deferred revenue, if pack pre-recognised at issuance)
    Cr Breakage Income (account 6730)
  Amount: 12 × TZS 1,000 = TZS 12,000 (if breakage recognised at expiry; some packs recognise gradually)
```

### Phase 8 — Reporting & ROI

Mzee Hassan (Mama Amina's bookkeeper) views Reporting projections:

```
Live dashboard:
  - SFEST-2026-10 ROI: +TZS 145K incremental sales vs September Coca-Cola baseline
  - Cost-share recovered from BOS: TZS 12,500
  - Net tenant cost: TZS 12,500 (tenant_share)
  - Vouchers issued: 50; redeemed: 38; expired: 12; redemption rate: 76%
  - Outreach: 87 attempted, 85 sent, 2 failed, 13 blocked (no consent)
  - Loyalty points accrued total: ~5,400 (across all qualifying sales)

Year-end statement: SFEST-2026-10 reflected in P&L's Promotion Expense line per pack template;
  cost-share recovery as separate income line per pack rules.
```

### What the Example Demonstrates

- **PR1 in action**: Promotion populated discount_refs; Checkout applied at settlement.
- **PR2 in action**: 50 vouchers issued as Documents — hash-verifiable.
- **PR3 in action**: Loyalty accrual + redemption via Obligation primitive (kind: loyalty_credit).
- **PR4 + UI-10**: Cost-share split recorded; 4 sources sum to total; bos_share creates Obligation; tenant_share absorbed.
- **PR5**: Outreach hard-blocked 13 customers without consent; auditable.
- **PR6**: Pack-driven discount-reduces-taxable-base treatment applied.
- **PR7**: Calculation precedence (line discount in this case; no stacking needed).
- **N1**: Single consolidated handler `apply_promotions_at_settlement` routed internally; all derived events batched atomically with UI-01 causation.
- **N2**: All namespaces `promotion.<noun>.<verb>.v<n>`.
- **N3**: No voucher in refunded sale, so restoration not triggered; would have followed pack policy if present.
- **N4**: cost_share_receivable_bos Obligation settled via Cash AR flow at month end; no new payment rail.
- **N5**: Targeting (customer / item / site / time) resolved via read-only projection lookups at command time.
- **D-DISC-001 + D-DISC-002**: Throughout the example, the customer's voucher entry / loyalty card scan / self-service interaction would happen on a Term 3 surface (D-DISC-001) or self-service kiosk (D-DISC-002) — both deferred to their respective future home. CN-5-007 produces the data; those future surfaces consume it.

---

*— End of CN-5-007 —*
