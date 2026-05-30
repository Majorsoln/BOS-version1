# CN-5-101 — Scope Policy Application

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 5 — Universal Engines
> **Status:** For Overseer review.
> **Governing decisions:** D-004 (neutrality — no vertical-flavoured naming); D-007 #4 (platform scope as parallel first-class scope); D-007 #5 (engine isolation).
> **CTRs:** CTR-023 (Term 5 → Term 4 — additive amendment to CN-4-005 manifest, CLOSED); CTR-024 (Term 5 → Term 6 — verticals carry `site_id` in payload of site-scope events, OPEN); CTR-016 (Term 4 → Term 1 — platform scope governance, OPEN — billing aggregation folded in).
> **Glossary:** See `MASTER-GLOSSARY.md` — Site, Scope level, Tenant Scope, Platform Scope.
> **Depends on:** CN-4-002 (Event Store Contract — envelope has `tenant_id` only); CN-4-005 (Engine Contract — `scope_policy` default + per-operation `scope_ref`, per CTR-023 amendment §1.1); CN-4-006 (Multi-Tenant Isolation — tenant scope object, platform scope as parallel); CN-4-011 (Primitive Catalog — primitives are scope-agnostic); CN-5-100 (Subscription Wiring Patterns — per-subscription `scope_ref`).
> **Boundaries:** Tenant isolation guarantees → CN-4-006; envelope shape → CN-4-002; primitive shapes → CN-4-011; manifest fields and per-operation override mechanics → CN-4-005 §1.1; subscription patterns and replay semantics → CN-5-100; engine invariants → CN-5-102; platform-scope governance (who may use it) → Term 1 (CTR-016); vertical-side `site_id` discipline → Term 6 (CTR-024).

---

## 1. What This Doc Defines

CN-5-101 applies CN-4-005's three scope levels — `site`, `tenant`, `platform` — to **universal engines**. It is the Term 5 policy that says, for every command and every subscription a universal engine declares:
- Which level it runs at.
- How `site_id` is sourced from payloads.
- How handler dispatch resolves the scope at runtime.
- When a universal engine may use platform scope (rare).

CN-4-005 §1.1 (the CTR-023 amendment) provides the **mechanism**: `scope_policy` as engine default plus per-operation `scope_ref`. CN-5-101 provides the **policy** that universal engines follow when they use that mechanism.

This doc does NOT define:
- Tenant isolation guarantees (CN-4-006 §1 — non-negotiable, inherited).
- The envelope or its fields (CN-4-002 — unchanged by this doc).
- Primitive shapes (CN-4-011 — scope-agnostic).
- Subscription patterns, laws, or replay semantics (CN-5-100).
- Platform governance — who may invoke platform scope (Term 1, CTR-016).
- How vertical engines populate `site_id` (Term 6, CTR-024).

---

## 2. Scope Levels

CN-5-101 recognises the three Kernel-level scopes (CN-4-005 §1.1) and applies them to universal engines:

| Level | Universal-engine meaning | Examples |
|-------|--------------------------|----------|
| **site** | An operation that runs **per intra-tenant location partition**. Site is a payload concept; `site_id` lives in the command or event payload. A site is a shop, a branch, a hotel property, a workshop floor, a clinic room — whatever the tenant treats as a distinct operational location. The Kernel does not interpret "site"; engines do. | Cash drawer/session per terminal; per-site stock balance; per-site cashier shifts. |
| **tenant** | An operation that runs **once per tenant**. The tenant is the isolation boundary (CN-4-006). "Business-wide" reads, ledgers, and aggregations all live here. | Accounting journals; consolidated cash position; tenant-wide chart of accounts; HR roster. |
| **platform** | An operation that runs **across tenants under the platform scope** (CN-4-006 §2). For universal engines this is rare and dual-audited; governance lives with Term 1 (CTR-016). | None by default — see §7. |

### Tags vs Levels

A tenant with many sites may want to group them — "all Mwanza sites," "all urban outlets," "northern region." This is not a fourth scope level. It is a **payload tag** on each site (e.g., `region: mwanza`, `tier: urban`) that engines can filter on at query time. Adding a formal "region" scope would rigidify the framework for a need that tags meet. If a Term 6 vertical (e.g., a national hotel chain) later requires region-level invariants, CN-5-101 may be revisited; for now, tags are sufficient.

---

## 3. The Five Laws

These laws apply to every universal engine's scope declaration. Violations are doctrine violations (CN-4-019 enforces).

| # | Law | Source / why |
|---|-----|--------------|
| **L1** | **Scope never crosses tenant.** `site` and `tenant` are intra-tenant; `platform` is a parallel scope (CN-4-006 §2), not a back door. No universal-engine operation reads or writes outside the originating tenant's data, except through an explicitly declared `platform` scope subject to L5. | CN-4-006 §1 — non-negotiable. |
| **L2** | **`site_id` lives in the payload, not the envelope.** CN-4-002's envelope is fixed (15 fields); `tenant_id` is the only location-bearing field. Intra-tenant site partitioning is therefore a payload concept — universal engines that need it read `site_id` from `payload.site_id`. Engines without site dispatch ignore it. | CN-4-002 envelope is canonical; adding fields would be a breaking Foundation change. |
| **L3** | **Scope is declared per operation, not per engine.** Each command and each `subscribes_to` entry in the manifest may declare its own `scope_ref` (CN-4-005 §1.1). The engine-level `scope_policy` is a default, not a constraint. Multi-scope engines are first-class (see §6). | CTR-023 amendment to CN-4-005 §1.1 gives universal engines this freedom; CN-5-101 ratifies its use. |
| **L4** | **Site-scope dispatch is per `(tenant_id, site_id)` tuple.** The framework reads `tenant_id` from the envelope and `site_id` from the payload, then dispatches the handler once per tuple. The handler sees only its own site's data — site isolation within a tenant is structural at the dispatch layer, not a handler discipline. | Inherits CN-4-010 §9 tenant-scoping and extends it with payload-derived site partitioning. |
| **L5** | **Platform scope on a universal engine is `requires`-rare and dual-audited.** A universal engine MAY declare a `subscribes_to` or `commands` entry at `scope_ref: platform` only when no per-tenant alternative exists. The entry must be flagged in the manifest as `platform_authorised: true`, the action is audited in **both** the platform audit trail and every contributing tenant's trail (CN-4-006 §2), and governance is gated by Term 1 (CTR-016). Default posture: universal engines do **not** subscribe at platform scope (see §7). | CN-4-006 §2 — platform scope is parallel and dual-actor-audited; relaxing this for engine convenience would weaken isolation. |

---

## 4. Scope Declaration in the Manifest

CN-5-101 does not re-specify what CN-4-005 §1.1 already establishes. It applies that mechanism to universal engines.

A universal-engine manifest declares:

| Field | Universal-engine usage |
|-------|------------------------|
| **scope_policy** | The engine's most common operating level. For most universal engines this is `tenant` (Accounting, HR, Reporting). For engines whose operations are predominantly per-location, this is `site` (Cash, parts of Inventory). |
| **commands[].scope_ref** | Optional per-command override. If absent, the engine's `scope_policy` applies. |
| **subscribes_to[].scope_ref** | Optional per-subscription override. Combines with CN-5-100 §4's per-subscription record (`{event_type, version, handler, kind, scope_ref}`). If absent, the engine's `scope_policy` applies. |

A subscription declared `scope_ref: site` reads `site_id` from the **incoming event's** payload. A command declared `scope_ref: site` reads `site_id` from the **submitted command's** payload. The Kernel rejects a site-scope dispatch where `site_id` is missing (per L2 + L4 — the framework requires the partition key for dispatch).

### Cross-Reference Summary

| Concept | Defined in |
|---------|-----------|
| Three scope levels (`site` / `tenant` / `platform`) | CN-4-005 §1.1 |
| Per-operation `scope_ref` mechanism | CN-4-005 §1.1 |
| Per-subscription record shape (5 fields incl. `scope_ref`) | CN-5-100 §4 |
| Universal-engine policy and runtime semantics (this doc) | CN-5-101 §§2–8 |

---

## 5. Scope Resolution at Runtime

When an event arrives or a command is submitted, the framework resolves scope deterministically:

```
1. Locate the relevant manifest entry (subscribes_to or commands).
2. Determine effective scope:
     entry.scope_ref  if declared
     else  manifest.scope_policy  if declared
     else  tenant   (CN-4-005 §1.1 fallback)
3. Dispatch:
     site     →  read site_id from payload; invoke handler with (tenant_id, site_id)
     tenant   →  invoke handler with (tenant_id) — once per tenant
     platform →  verify manifest.platform_authorised = true (L5);
                 invoke handler with platform_scope object (CN-4-006 §2);
                 emit dual-trail audit entries.
4. If a site-scope dispatch finds no site_id in payload, the dispatch is
   rejected with a kernel.command.rejected.v1 or a framework-level
   delivery rejection event (the event remains in the store; the
   subscriber's projection records a structural rejection, NOT the
   business handler being invoked).
```

This is the algorithm every universal-engine handler dispatch follows. The handler itself receives a fully-scoped invocation context — it never re-derives scope.

### Why Reject on Missing `site_id`

A site-scope subscription that silently runs at tenant scope when `site_id` is absent would leak per-site state across sites. Explicit rejection makes the contract visible: if an emitter (a vertical engine, per CTR-024) fails to include `site_id`, the universal-engine subscription fails loudly. The vertical fixes its payload; isolation holds.

---

## 6. Multi-Scope Engines

A single universal engine may legitimately operate at more than one scope. CN-5-101 makes this a first-class pattern, not an exception. The canonical example is Cash Engine (CN-5-002, forthcoming).

### Cash Engine Manifest Sketch

```
engine_id: cash
scope_policy: site            # most operations are per-terminal/per-drawer
commands:
  - cash.session.open.request           # scope_ref: site (default)
  - cash.session.close.request          # scope_ref: site (default)
  - cash.tender.receipt.request         # scope_ref: site (default)
  - cash.position.query.request         # scope_ref: tenant (override)
  - cash.transfer.between_sites.request # scope_ref: tenant (override — both sites)
emits:
  - cash.tender.received.v1             # payload includes site_id
  - cash.session.closed.v1              # payload includes site_id
  - cash.position.snapshot.v1           # tenant-wide, no site_id needed
subscribes_to:
  - {event_type: checkout.settled.v1,    version: 1, handler: record_tender_receipt, kind: command_emitting, scope_ref: site}
  - {event_type: accounting.period.closed.v1, version: 1, handler: lock_sessions, kind: command_emitting, scope_ref: tenant}
```

### Reading the Sketch

- The engine's **default** is `site` — most commands and subscriptions inherit it.
- `cash.position.query.request` overrides to `tenant` because a consolidated position is not per-terminal.
- `cash.transfer.between_sites.request` overrides to `tenant` because the operation **necessarily touches two sites** within the tenant; running it under either site's scope would mis-represent the operation.
- The subscription to `accounting.period.closed.v1` is `tenant` because period close is a tenant-level event from Accounting; reacting per site would attempt to lock sessions for sites the event does not name.

### What This Pattern Replaces

Without L3 (per-operation `scope_ref`), Cash would either:
- Split into two engines (`cash-site` and `cash-tenant`) — fragmenting one cohesive domain, **or**
- Run everything at `site` and re-aggregate consolidations in a downstream projection — duplicating logic and obscuring the "consolidated position is a first-class Cash operation" intent.

Multi-scope manifests express the domain cleanly without splitting.

---

## 7. Platform Scope for Universal Engines (Reporting → Term 1)

Universal engines avoid platform-scope subscriptions by default. The canonical case — billing aggregation for Term 1 — is handled via the **Reporting → Term 1 chain**, not by a universal engine reaching platform scope directly.

### The Pattern (D-005 / CTR-016 alignment)

| Layer | Operates at | Emits | Consumes |
|-------|-------------|-------|----------|
| **Reporting Engine (CN-5-006)** | `tenant` | `reporting.metrics.monthly.v1` per tenant (transaction counts, revenue, usage indicators) | Subscribes to engine events at `tenant` scope |
| **Term 1 platform aggregator** | `platform` (CN-4-006 §2) | Platform-scope subscription to `reporting.metrics.monthly.v1` across tenants → aggregates for billing, governance | Operates under platform audit (CTR-016 governance) |

### Why Not a Universal-Engine Platform Subscription

A Reporting Engine subscribing at platform scope would mean a Term 5 universal engine reading every tenant's events. That:
- Crosses the universal-engine boundary (CN-5-100 L1).
- Bypasses the deliberate Term 1 governance for platform reads (CTR-016).
- Couples engine logic to platform aggregation logic — which is a Term 1 concern.

Keeping the boundary clean: **universal engines emit per-tenant; Term 1 aggregates at platform scope under its own governance**. This is L5 in practice — universal engines do not subscribe at platform unless no per-tenant alternative exists, and for billing aggregation a per-tenant alternative is straightforward.

### When L5 Would Allow Platform Subscription

If a future universal engine genuinely needs cross-tenant data with no per-tenant decomposition (a hypothetical "anti-fraud cross-tenant pattern engine" might qualify), CN-5-101 L5's conditions apply: explicit `platform_authorised: true` in the manifest, dual-trail audit, Term 1 governance. No such engine exists in the current Term 5 catalogue.

---

## 8. Naming Neutrality

The Kernel uses `site` for the intra-tenant partition (CN-4-005 §1.1, MASTER-GLOSSARY). CN-5-101 enforces this naming in universal-engine payloads:

- The payload field is **`site_id`**, never `branch_id`, `outlet_id`, `terminal_id`, or `location_id`.
- A site may semantically *be* a branch (Retail), an outlet (Restaurant), a property (Hotel), a workshop floor (Workshop), or a clinic room (a future vertical) — but the field name is invariant.
- Tags on the site record (region, tier, vertical-flavoured label) carry the semantic descriptors a tenant or vertical wants — without contaminating the partition key.

This is D-004 neutrality applied to scoping. The same word means the same thing across every vertical, present and future.

---

## 9. Example — Mama Amina's Two Branches

*Mama Amina's duka serves as illustrative context, per D-004 #4 (peer-technical audience). The example shows the scope-policy contract operating; it is not a UX description.*

Mama Amina operates one tenant (`mama-amina-duka`) with two sites: Mwanza (`site_id: mama-amina-mwanza-001`) and Mbeya (`site_id: mama-amina-mbeya-001`).

### A Sale in Mwanza

1. Cashier A at the Mwanza terminal completes a sale. Retail (Term 6) emits `retail.bill.ready.v1` with `payload.site_id = mama-amina-mwanza-001` (per CTR-024).
2. The Universal Checkout/Tender Engine (CN-5-009, forthcoming) settles and emits:
   ```
   event_type:       checkout.settled.v1
   tenant_id:        mama-amina-duka                   (envelope, CN-4-002)
   payload.site_id:  mama-amina-mwanza-001             (Term 6 supplied; CTR-024)
   payload.lines:    [...]
   payload.tenders:  [{method: cash, amount: 30000}, {method: mpesa, amount: 15000}]
   ```
3. Three universal engines subscribe (per CN-5-100 §8 example, now with explicit scope):

| Engine | Subscription `scope_ref` | Dispatch |
|--------|--------------------------|----------|
| Accounting (CN-5-001) | `tenant` | Handler invoked once for tenant `mama-amina-duka`. Journal posts revenue/tax/COGS at tenant scope; `site_id` is carried into the journal payload as an **analytical dimension**, not as a ledger key (per CN-4-011 Ledger is scope-agnostic). |
| Cash (CN-5-002) | `site` | Framework reads `site_id = mama-amina-mwanza-001` from payload. Handler invoked with `(mama-amina-duka, mama-amina-mwanza-001)`. Updates only the Mwanza drawer/session; Mbeya state is untouched. |
| Inventory (CN-5-003) | `site` | Same dispatch as Cash. Stock deductions affect only Mwanza's stock projection. |

### A Consolidated Cash Position Query

Mzee Hassan (Mama Amina's accountant) submits `cash.position.query.request` — declared at `scope_ref: tenant` in Cash's manifest (per §6).

The handler folds Cash's per-site projections for both Mwanza and Mbeya into a single tenant-wide cash position. The query is a single dispatch at tenant scope; site-level state is read as input, not as dispatch partition.

### A Cash Transfer Between Branches

`cash.transfer.between_sites.request` carries `{from_site: mwanza-001, to_site: mbeya-001, amount: 500000}`. Declared `scope_ref: tenant` (per §6) because the operation touches both sites coherently.

The handler runs once at tenant scope, emitting `cash.transfer.initiated.v1` and `cash.transfer.received.v1` (each carrying its respective site_id in payload). Subsequent site-scope subscriptions on those events dispatch correctly to Mwanza (debit) and Mbeya (credit) projections.

### Platform Aggregation (Term 1, Not Term 5)

At month end, Reporting emits `reporting.metrics.monthly.v1` for `mama-amina-duka` (tenant-scope). Term 1's billing aggregator subscribes at platform scope and rolls metrics for billing — but no Term 5 engine reads outside this tenant. The platform read is audited in both the platform trail and Mama Amina's tenant trail (CN-4-006 §2; CTR-016 governance).

### A Doctrine Failure Caught at Dispatch

Suppose Retail (Term 6) regressed and stopped including `site_id` in `retail.bill.ready.v1`. Cash's site-scope subscription receives the derived `checkout.settled.v1` without `site_id`. The framework's site-scope dispatch (§5) rejects: a structural rejection event is emitted; Cash's projection records the gap; the lag escalates; an operator investigates; the vertical regression is fixed. **No silent dispatch at tenant scope** — the contract held.

---

## 10. Three Whys

### Why does this matter?

Without an applied scope policy, every universal-engine doc would re-invent how its operations partition. One engine would treat "branch" as a foreign key; another would put it in metadata; a third would have no partitioning at all and conflate two terminals' cash. The cumulative effect is a system where the same word means different things in different engines, where consolidated reports are wrong because they aggregate at the wrong level, and where vertical engines cannot reliably hand off site context. CN-5-101 makes one rule: site is `site_id` in payload; scope is declarative per operation; tenant is the isolation boundary; platform is rare and audited.

### Why does it belong here (and not in Foundation)?

Foundation provides the levels and the per-operation mechanism (CN-4-005 §1.1, post-CTR-023). Foundation does not name what a "site" is for universal engines, when to use which level, or what happens at dispatch when `site_id` is missing. Those are Term 5 policy decisions. CN-4-005 is engine-agnostic; CN-5-101 is universal-engine-specific. Foundation gives the materials; CN-5-101 specifies how universal engines build with them.

### Why this design?

A three-level model with per-operation override fits the actual shape of universal-engine domains: most domains have a clear default level, with a small minority of operations that legitimately need to escape it (consolidations, transfers, cross-site queries). Forcing every engine to one level would either split coherent domains (Cash → two engines) or push aggregation work outside the engine that owns it. Putting `site_id` in payload — not envelope — preserves Foundation's frozen contract and lets engines that don't care about sites ignore them. Defaulting universal engines away from platform scope (Reporting → Term 1 chain) preserves Term 1's governance authority over cross-tenant reads. Each design choice is the smallest change that meets the domain need without bending Foundation or expanding scope-language proliferation.

---

## 11. Boundaries

| Topic | Lives in |
|-------|----------|
| Three-level scope mechanism and per-operation `scope_ref` | CN-4-005 §1.1 (post-CTR-023) |
| Tenant isolation guarantee (non-crossable) | CN-4-006 §1 |
| Platform scope parallel + dual audit | CN-4-006 §2 |
| Envelope fields (`tenant_id` only; no `site_id`) | CN-4-002 |
| Scope-agnostic primitives | CN-4-011 |
| Subscription declaration shape (5-field record) | CN-5-100 §4 |
| Subscription patterns, laws, replay semantics | CN-5-100 |
| Vertical-side `site_id` payload discipline | CTR-024 (→ Term 6) |
| Platform-scope governance (who may invoke, billing aggregation) | Term 1 (CTR-016) |
| Engine invariants that span scope levels (e.g., debit=credit must hold per tenant) | CN-5-102 |
| Per-engine concrete scope declarations | per-engine docs (CN-5-001 … CN-5-007, CN-5-009, CN-5-010) |

---

## 12. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Vertical engines populate `site_id` consistently in site-scope event payloads | Term 6 via CTR-024 | OPEN — needed before site-scope universal-engine subscriptions can rely on vertical emissions in production |
| Platform-scope billing-aggregation governance (who, when, audit retention) | Term 1 via CTR-016 | OPEN — Foundation delivered platform-scope mechanism (CN-4-006 §2); Term 1 owns the policy |
| Tag taxonomy for intra-tenant grouping (region, tier, etc.) | per-engine docs + Term 1 catalog | Tags are payload concept; no formal scope. Per-engine docs declare tags they read; Term 1 governs canonical tag vocabularies |
| Doctrine enforcement: site-scope subscription rejects on missing `site_id` | CN-4-019 + Architect | Concept (§5) requires the rejection; mechanism and rejection-event shape are Architect's |
| Multi-site transfer event-pair pattern (initiated/received with both site_ids) | CN-5-002 (Cash) + CN-5-104 (Period Close) | This doc names the pattern; per-engine docs specify the event pair |
| Cross-tenant universal-engine subscription (hypothetical anti-fraud) | Future decision | Default posture: not allowed. If a real need arises, L5 conditions apply and Term 1 governance must approve |

---

*— End of CN-5-101 —*
