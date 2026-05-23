# Term 4 — Foundation

> **Parent:** [BOS-CONCEPT-CHARTER.md](../BOS-CONCEPT-CHARTER.md) — read first
> **Type:** System Term (Part B)
> **Status:** Concept Phase v1.0

---

## 1. Mission

Foundation is the unbreakable bottom of BOS. Everything else — every engine, every portal, every dashboard — sits on top of what this Term designs. If Foundation cracks, BOS cracks. If Foundation is well-built, BOS can grow for 20 years and absorb verticals no one has imagined yet (insurance, healthcare, marketing, logistics, education) without rewriting itself.

This Term thinks about the deepest, longest-lasting decisions in BOS: how events are stored, how state is rebuilt, how engines stay isolated, how security is enforced, how identity flows, how documents stay legally defensible, how AI is constrained, how the kernel proves to a court that nothing was tampered with.

Foundation is invisible to tenants, agents, and most platform staff. But every line of code in every engine depends on it being right.

---

## 2. Mission Question

> **"How do we build a kernel that doesn't crack after 10 years of constant change — that can absorb 50 new verticals, 30 new countries, AI integration we haven't imagined, and a court audit demanding proof of every transaction — without ever requiring a rewrite?"**

---

## 3. Scope (In)

This Term owns concepts for:

- **Event Store** — append-only, hash-chained, immutable record of everything that happens
- **Command Bus** — request → policy check → accept/reject → event emission flow
- **Engine Contract Model** — how engines declare themselves and stay isolated
- **Primitives** — reusable building blocks shared across engines
  - Ledger primitive
  - Item/Service primitive
  - Actor primitive
  - Document primitive
  - Inventory Movement primitive
  - Obligation primitive
  - Approval primitive
  - Workflow primitive
  - Consent primitive
- **Identity & Multi-Tenant Isolation** — making sure tenant A cannot ever see tenant B's data
- **Security Kernel** — rate limits, anomaly detection, scope guards
- **Audit Log Primitive** — the append-only journal beneath every other audit log
- **Replay Engine** — rebuilding state from events deterministically
- **Projection Framework** — how read-models are defined, registered, and rebuilt
- **Snapshot Storage** — for performance, but never as a source of truth
- **Hash Chain & Tamper Detection** — proving the event store is intact
- **AI Guardrails** — the framework that constrains AI to advisory-only
- **Decision Journal** — immutable log of every AI suggestion (accepted or rejected)
- **Document Engine** — structured document builder with hash-verified outputs
- **Numbering Engine** — fiscal-compliant, branch-aware, deterministic document numbering
- **Document Verification Portal** — public hash verification of issued documents
- **Time Authority** — how BOS handles clocks deterministically (for replay)
- **Compliance DSL Framework** — the language compliance packs are written in (not the packs themselves; those are per region)
- **Resilience Modes** — NORMAL / DEGRADED / READ_ONLY states and transitions

---

## 4. Scope (Out)

This Term does NOT own:

| What | Owned By |
|------|----------|
| Regional compliance packs themselves (TZ, KE rules) | Term 1 (approval); Term 2 (input from agents); Term 7 (validation) |
| Engine-specific logic (how Accounting computes journals) | Term 5 — Universal Engines |
| Vertical-specific logic (how Restaurant tracks kitchen tickets) | Term 6 — Vertical Engines |
| Tenant or agent UI | Term 3 / Term 2 |
| Platform admin UI | Term 1 |
| External integration adapters (M-Pesa, banks) | Term 7 — Integration & Coherence |
| Subscription billing logic | Term 1 (platform rules); Term 5 (accounting integration) |

**Important distinction:** Foundation provides the *materials* (primitives, event store, security). Other Terms build *structures* with those materials. Foundation does not know what verticals exist.

---

## 5. Audiences Served

Foundation has no end-user audience. Its audience is **other Terms** and **Architects**.

| Audience | What they get from Foundation |
|----------|-------------------------------|
| Term 5 (Universal Engines) | Primitives, event store, command bus, isolation |
| Term 6 (Vertical Engines) | Same as above + ability to add new engines without modifying Foundation |
| Term 1 (Platform Stewards) | Audit log primitive, identity, security primitives |
| Term 2 (Regional Distribution) | Identity, audit, document primitive |
| Term 3 (Tenant Experience) | Projection framework, time authority, document primitive |
| Term 7 (Integration & Coherence) | Event-driven contracts that integrations attach to |

**Foundation's promise to other Terms:** *"If you follow our doctrine, you cannot accidentally break the system. The kernel will catch your mistakes before they escape."*

---

## 6. Key Concepts This Term Will Produce

Concept documents under this Term:

| ID | Concept | What it answers |
|----|---------|-----------------|
| CN-4-001 | Event Sourcing Doctrine | Why events are the only source of truth, and what this implies |
| CN-4-002 | Event Store Contract | What an event is, what it must contain, how it is stored |
| CN-4-003 | Hash Chain & Tamper Detection | How we prove the store is intact, and what happens if it isn't |
| CN-4-004 | Command Bus & Policy Engine | How a request becomes an event (or doesn't) |
| CN-4-005 | Engine Contract Model | How an engine declares itself, what it can do, what it cannot |
| CN-4-006 | Multi-Tenant Isolation | How tenant A cannot see tenant B — at every layer |
| CN-4-007 | Identity Primitive | Who is acting, with what authority |
| CN-4-008 | Audit Log Primitive | The unedited journal of every action |
| CN-4-009 | Replay Engine | Rebuilding state, scoped replay, partial replay |
| CN-4-010 | Projection Framework | Defining, registering, and rebuilding read-models |
| CN-4-011 | Primitive Catalog | What primitives exist and why (ledger, item, actor, document, etc.) |
| CN-4-012 | Document Engine & Numbering | Hash-verified documents and fiscal-compliant numbering |
| CN-4-013 | AI Guardrails & Decision Journal | The cage AI lives in, and the journal of its suggestions |
| CN-4-014 | Time Authority | Deterministic time, replay-safe clocks |
| CN-4-015 | Compliance DSL Framework | The grammar regional packs are written in |
| CN-4-016 | Security Primitives | Rate limits, anomaly detection, scope guards |
| CN-4-017 | Resilience Modes | NORMAL / DEGRADED / READ_ONLY — when and how |
| CN-4-018 | Snapshot Storage (non-truth) | Performance caches that must never be trusted as source |
| CN-4-019 | Doctrine Enforcement Checks | The invariant tests that prove Foundation is sound |
| CN-4-020 | Extension Points (How to Add a New Engine) | The clean process by which Term 6 adds a vertical without touching Foundation |

---

## 7. Edge Cases to Consider

The hard questions this Term must answer:

**Event Store Realities**
- What if two commands arrive at the exact same nanosecond? Ordering?
- An event was emitted but the projection update failed. Now state is rebuildable but the dashboard is stale. How do we recover?
- A bug emits a wrong event. Charter says no deletion. So what? (Answer: compensating events. How exactly?)
- The hash chain is broken (disk corruption, attack). What's the recovery protocol?
- The event store is 50TB after 5 years. Are partitions/archives part of the design?

**Replay Realities**
- A projection rebuild takes 6 hours and needs to happen during business hours. What modes does this require?
- Tenant A wants their data exported (GDPR-style right). How is "their" data scoped from a multi-tenant store?
- A regional compliance change retroactively affects how old events should be interpreted (e.g., a tax rate was wrong). Replay with new rules? Or freeze old events with old rules? (Strongly: freeze. But the doc must reason this out.)

**Engine Isolation Realities**
- Engine A wants to know "is the current month closed in Accounting?" Charter says no direct calls. So how?
- A new engine (Insurance, in 2027) needs to subscribe to retail.sale.completed.v1. Does Retail need to know Insurance exists? (Answer: no. How does the subscription framework guarantee this?)
- An engine has a bug that emits malformed events. How does Foundation isolate the damage?

**Multi-Tenant Realities**
- A platform admin runs a query "all tenants in TZ this month." Is this allowed? At what permission level?
- A tenant runs a query "did anyone from outside our business access our data this month?" Can BOS answer truthfully?
- Two tenants share a server. One has a memory leak. Does the other notice?

**Identity Realities**
- A human user, an automated system, an AI advisor, a regional agent: each is an "actor." How are they distinguished in the audit log, with what fields?
- A platform admin is reviewing as "themselves" but they're acting *as* a tenant for support. Who is the actor on the audit log? (Both. How?)
- An API client (machine) calls BOS. What identity does it carry?

**AI Realities**
- An AI advisor sees a tenant's data, makes a suggestion. The tenant ignores it. The same situation arises tomorrow. Does AI suggest again, or stay quiet? (Decision Journal can answer.)
- An AI advisor is asked to write a query. The query returns cross-tenant data. What stops it from including that in the suggestion?
- An AI advisor's suggestion is later shown to have been wrong and harmful. Who is liable? (BOS, the tenant who acted on it, the AI provider?) The Decision Journal must capture enough to settle this.

**Document Engine Realities**
- A tenant edits a document template. Old documents must remain immutable. How is template versioning handled?
- A document is issued, then the law changes. Old documents stand under old law; new documents follow new law. How is this enforced?
- A document number is needed and the system is offline. What happens? (Cannot pre-allocate non-deterministically.)
- A QR code is scanned to verify a document. Verification must work without internet (cached public keys). How?

**Compliance DSL Realities**
- A new tax in Rwanda needs to be expressed. Is the DSL expressive enough? What if it isn't?
- A compliance pack has a bug. How quickly can it be rolled back? Are tenants notified?
- A compliance pack uses a function not yet in the DSL grammar. How is the grammar extended without breaking existing packs?

**Time Realities**
- Replay must produce identical state given identical events. But events have timestamps. What if a developer used `datetime.now()` inside engine logic? (Charter forbids it. Foundation enforces it.)
- A tenant in Tanzania closes their day at 11pm local. An event arrives at 23:59:59. The replay later sees that as a different "day" because of timezone. How is "day" defined?

**Performance Realities**
- A projection rebuild blocks the dashboard for 10 minutes. Tenant cannot operate. What's the contingency?
- Snapshot storage gets out of sync with events. Tenant sees stale data. How are stale snapshots detected and refreshed?
- The audit log is queried by regulators with massive date ranges. Pagination? Pre-computed indexes?

---

## 8. Success Criteria

This Term is doing well when:

1. **A new engine can be added by Term 6 in 2 weeks without modifying Foundation.** Extension points work.
2. **A new country can be added without Foundation knowing about it.** Compliance DSL is expressive enough.
3. **An auditor can prove the event store has not been tampered with.** Hash chain holds.
4. **A projection can be rebuilt in production without taking the system offline.** Replay is non-blocking.
5. **Two tenants cannot accidentally share data.** Isolation is provable, not just claimed.
6. **An AI suggestion can be traced back to the exact data and prompt that produced it.** Decision Journal is complete.
7. **A document issued today can be verified in 10 years.** Hash + numbering + template version all preserved.
8. **A junior developer cannot accidentally violate the Three Laws.** Doctrine enforcement (linters, invariant tests) catches it.
9. **A bug in one engine cannot crash the kernel.** Engine isolation is real.

---

## 9. Architect's Role for This Term

After Concept Team writes documents, the Architect translates them into:

| Concept Output | Architect Output |
|----------------|------------------|
| Event store contract (CN-4-002) | Postgres schema, append-only triggers, hash computation spec |
| Hash chain (CN-4-003) | Hash algorithm choice, chain verification API, tamper response procedure |
| Command bus (CN-4-004) | Dispatcher architecture, policy evaluation flow, rejection event design |
| Engine contracts (CN-4-005) | Engine registry data model, manifest schema, compatibility check API |
| Tenant isolation (CN-4-006) | Tenant scope object, middleware enforcement, leak detection tests |
| Identity (CN-4-007) | Actor model, principal types, authentication chain |
| Audit log (CN-4-008) | Append-only audit table, query API, retention policy |
| Replay (CN-4-009) | Replay engine architecture, checkpoint storage, scope handling |
| Projections (CN-4-010) | Projection protocol, registry, rebuild orchestration |
| Primitives (CN-4-011) | Each primitive's data shape and operations |
| Document engine (CN-4-012) | Template schema, render pipeline, numbering allocator |
| AI guardrails (CN-4-013) | Forbidden operations list, journal schema, action types |
| Time authority (CN-4-014) | Clock protocol, FixedClock for tests, SystemClock for runtime |
| Compliance DSL (CN-4-015) | Grammar spec, validation engine, runtime evaluator |
| Security (CN-4-016) | Rate limiter design, anomaly rules, scope guard middleware |
| Resilience (CN-4-017) | Mode state machine, health checks, transition triggers |
| Snapshots (CN-4-018) | Snapshot storage table, expiry logic, source-vs-cache markers |
| Doctrine checks (CN-4-019) | Invariant test suite design, CI hooks |
| Extension points (CN-4-020) | New engine checklist, registration API, isolation tests |

**Architect's directive to Developer for Term 4:**
- All Foundation code is pure Python (no Django dependency in core)
- Every primitive is an immutable dataclass (`frozen=True`)
- No `datetime.now()` inside engine logic — must go through Clock protocol
- Every audit entry, event, decision-journal entry is append-only
- Every command goes through `policy_name="function_name"` on rejection
- 11+ architectural invariants enforced as automated tests in CI

---

## 10. Relationships with Other Terms

### Term 4 depends on:

Effectively, nothing. Foundation is the bottom.

But it does need:
| Term | Why |
|------|-----|
| Term 7 (Integration & Coherence) | Validates Foundation doctrine doesn't accidentally prevent legitimate use cases |

### Term 4 is depended on by:

| Term | Why |
|------|-----|
| **All other Terms** | Foundation supplies the building materials |

### Cross-Term Hand-Offs

- → Term 5: "Here is the ledger primitive, the obligation primitive, the inventory movement primitive. Build your engines on these."
- → Term 6: "Here is the extension point for new engines. Here is the checklist."
- → Term 1: "Here is the audit log primitive. Wrap your admin actions in it."
- → Term 3: "Here is the projection framework. Define your read-models against it."
- ← Term 7: "Your event store doctrine breaks SMS receipt sending under intermittent connectivity. Help?"

---

## 11. Open Questions

1. **Snapshot trust boundary.** Charter says "rebuildable from events." But for performance, projections are cached. How do we mark cached state as "non-truth" clearly so no developer accidentally trusts it as source?
2. **Compensating events.** A wrong event was emitted. Charter forbids deletion. So we emit a "correction" event. How is the corrected reading derived? Is "the truth" the latest, or the sum, or something else? Per primitive?
3. **Cross-tenant queries (platform).** Platform admin needs to see "all tenants in TZ this month." This crosses tenant scope. How is the exception modelled — by a special "platform" actor with documented privileges?
4. **Identity as system.** When BOS itself acts (e.g., scheduled month-end close), what actor is recorded? A "system" principal? With what audit visibility?
5. **AI provider identity.** When an AI advisor makes a suggestion, the suggestion is from "the AI." But which AI — Anthropic Claude? An open-source model? In the audit log, is the provider/version recorded?
6. **Time zones across replay.** A business in Tanzania at UTC+3. Replay must handle "day" boundaries. Is the business's local time anchored at registration, or recomputed each replay?
7. **Compliance DSL escape hatch.** What if a country has a rule the DSL can't express? Do we extend the DSL (slow, careful) or allow a "custom function" (dangerous, but expressive)?
8. **Document immutability vs corrections.** A document was issued with the wrong VAT amount. The buyer disputes. We issue a credit note. Is the credit note a new document, an event that modifies the old, or both?
9. **Replay during incidents.** A bug in an engine emitted bad events for 4 hours. We want to ignore those events on replay. How is "selective replay" expressed without violating event store immutability?
10. **AI knowledge boundary.** AI has read access to a tenant's events to make suggestions. Does AI also see other tenants' aggregated data (for benchmarking)? If yes, is this opt-in?

---

## 12. Notes on Existing Repository

The existing repository has:
- `core/event_store/` — event storage with hash-chaining (Phase 0)
- `core/commands/` — command bus
- `core/engines/` — engine registry
- `core/identity/` — identity primitives
- `core/audit/`, `core/time/`, `core/security/`, `core/resilience/`, `core/config/`, `core/business/` (Phase 11 / GAP-11)
- `core/replay/`, `core/projections/` — replay and projection framework
- `core/document_issuance/`, `core/documents/` — document engine
- `ai/guardrails.py`, `ai/journal/`, `ai/advisors/`, `ai/decision_simulation/` — AI advisory

**Concept Phase guidance:** The Foundation in code is the most mature part of the repo. This Term's concept docs will largely **describe and refine** what already exists, plus identify gaps. Term 4 is the most "reference-heavy" Term — much can be reused if the doctrine matches.

**Critical:** Even where the code is correct, the *concept doc* must be written for someone who has never seen the code. The doc is the authority; the code is the implementation.

---

## 13. First Tasks for This Term

When this Term starts concept work, begin in this order:

1. **CN-4-001 first** — event sourcing doctrine, because it justifies every other Foundation decision.
2. **CN-4-002 second** — event store contract, the concrete shape of an event.
3. **CN-4-006 third** — multi-tenant isolation, because every other doc must respect it.
4. **CN-4-007 fourth** — identity primitive, the actor on every event.
5. **CN-4-005 fifth** — engine contract model, since this enables all of Term 5 & 6 work.
6. **CN-4-011 sixth** — primitive catalog, the materials Term 5 & 6 will use.
7. **CN-4-020 seventh** — extension points, the contract for adding new engines.
8. Then CN-4-008 through CN-4-019 in parallel.

---

## 14. A Note on Foundation's Restraint

The biggest risk for this Term is **over-engineering**. Foundation must do just enough — and not more.

- Foundation does not know about Retail or Restaurant. It provides primitives they can use.
- Foundation does not know about Tanzania or Kenya. It provides a DSL they can be written in.
- Foundation does not know about M-Pesa. It provides event subscriptions that integrations can attach to.

Whenever a Foundation concept doc is tempted to mention a specific vertical, country, or integration — that is a sign the abstraction is wrong. Step back, re-abstract, and let the higher Terms build with the materials.

> *"The mark of a good kernel is what it refuses to do."*

---

## Standing Decisions & Cross-Term Work

> Read `DECISION-LOG.md`, `CROSS-TERM-COLLABORATION.md`, and `CROSS-TERM-REQUEST-REGISTER.md` before discussing these. File a CTR for every dependency.

| New Concept | From | This Term must decide |
|-------------|------|------------------------|
| CN-4-021 | D-001 | The **Saleable Line / Charge** and **Tender** value shapes; whether Checkout is a primitive or a universal engine built on obligation + document + ledger |
| CN-4-022 | D-002 | The **Advisor Framework contract** (audience/scope/model plug-in); extend AI Guardrails + **Decision Journal** to record **model identity/version** and explainability fields |
| CN-4-023 | D-002 | The **kernel boundary**: declare **Developer-AI is out-of-kernel**; define the **doctrine-enforcement gate** every proposed change must pass before release |

**CTRs to expect:** receives CTR-001 (line/tender ← 6), CTR-003 (checkout boundary ← 5), CTR-007 (advisor framework ← 5,2,3), CTR-009 (explainability ← 3); files CTR-008 (Developer-AI governance → Term 1); confirms CTR-010 (advisory-only).

---

*— End of Term 4 Brief —*
