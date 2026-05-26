# CN-4-006 — Multi-Tenant Isolation

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-007 #4 (platform scope, principal types).
> **CTRs:** CTR-016 (us → Term 1, platform scope policy — pending).
> **Glossary:** See `MASTER-GLOSSARY.md`.
> **Depends on:** CN-4-001 (Doctrine), CN-4-002 (Event Store Contract).
> **Boundaries:** Identity/principals → CN-4-007; audit fields → CN-4-008; scope guards → CN-4-016; advisor scope → CN-4-022; resource isolation → Architect phase.

---

## 1. The Isolation Guarantee

Tenant A cannot see, query, or affect tenant B's data. This holds at **every layer** the Kernel provides. It is a structural guarantee enforced by the Kernel — not a policy, not a convention, not a per-engine responsibility.

### Per-Layer Enforcement

| Layer | How isolation holds |
|-------|--------------------|
| **Event Store** | Every event carries `tenant_id` (CN-4-002). Queries are always scoped to a tenant. No query mechanism returns events across tenants without an explicit platform-scope principal. |
| **Command Bus** | Every command carries the tenant context. The bus rejects any command where the actor's tenant scope does not match the target tenant. A command cannot target another tenant's aggregate. |
| **Projections** | Projections are built and queried per tenant. A projection for tenant A never includes events from tenant B. Rebuild is per-tenant-scoped. |
| **Audit Log** | Audit entries carry `tenant_id`. Tenant audit queries return only that tenant's entries. |
| **AI Advisor reads** | The advisor's scope is bound to one tenant (CN-4-022). The runtime scope guard (D-007 #6) rejects reads outside that scope. Cross-tenant benchmarking is opt-in only and anonymised (CN-4-022). |
| **Engine logic** | Engines receive only tenant-scoped events via subscription. An engine processing tenant A's events never sees tenant B's events in the same context. |
| **Snapshots** | Snapshots are per-tenant. A snapshot for tenant A contains zero data from tenant B. |

### The Mechanism: Tenant Scope Object

Every request, command, query, and event-subscription in the Kernel carries a **tenant scope object** — not just a `tenant_id` parameter but a scope structure that the Kernel middleware validates before any operation proceeds.

The scope object is:

- **Injected at the boundary.** When a request enters the system, the tenant scope is resolved from the authenticated actor and attached to the request context.
- **Propagated through all layers.** Command bus, event store, projections, audit, advisor framework — all receive and enforce it.
- **Non-bypassable.** No API, query, or internal call can omit the scope object. The Kernel rejects scope-less operations.

---

## 2. Platform Scope

Platform staff (admins, support) need cross-tenant visibility: "all tenants in this region," "aggregate revenue this month," "tenant X's data for support."

### Not an Exception — A Parallel Scope

Platform scope is **not** a hole in tenant isolation. It is a separate, first-class scope that operates alongside tenant scope.

| Property | Tenant Scope | Platform Scope |
|----------|-------------|---------------|
| Who may use it | Tenant roles (owner, cashier, manager) | Platform roles (defined by Term 1) |
| What data it accesses | One tenant's data only | Cross-tenant or aggregate data |
| Audit trail | Tenant audit log | **Platform audit log** — separate, dedicated trail |
| How it's expressed | Scope object carries `tenant_id` | Scope object carries `platform_scope` with explicit visibility parameters (region, tenant set, aggregate-only) |
| Can it modify tenant data? | Yes (through commands) | **No** — platform scope is read-only against tenant data. Modifications require entering the specific tenant's scope, with dual-actor audit (CN-4-007, D-007 #4). |

### Platform Scope Operations

| Operation type | How it works |
|---------------|-------------|
| **Aggregate query** ("revenue across all tenants in region X") | Platform-scope query; returns aggregates, never individual tenant events. Audited in the platform audit trail. |
| **Specific-tenant support** ("view tenant Y's recent sales") | Platform admin enters tenant Y's scope explicitly. Dual-actor record: platform principal + tenant scope. Audited in both platform and tenant audit trails. |
| **Cross-tenant data export** (for platform analytics) | Anonymised aggregate only; never identifiable per-tenant data unless the tenant has opted in. |

**Foundation provides the mechanism.** Term 1 defines which platform roles may use it and under what policies (CTR-016, pending).

---

## 3. GDPR-Style Export Scoping

A tenant requests: "give me all my data" (right of export/portability).

### How "This Tenant's Data" Is Identified

| Data type | Scoping mechanism |
|-----------|-------------------|
| **Events** | All events where `tenant_id` matches the requesting tenant. No cross-tenant events exist (by construction). |
| **Projections** | Projections are per-tenant. The full projection state for this tenant is included. |
| **Audit log entries** | All audit entries where `tenant_id` matches. |
| **Decision Journal entries** | Journal entries where the advisor's scope was this tenant. |
| **Documents** | All documents issued under this tenant. |

### Guarantees

- The export includes **only** this tenant's data — no events, projections, or audit entries from any other tenant leak into the export.
- The export is **complete** — every event, projection, audit entry, journal entry, and document belonging to this tenant is included.
- The export is **verifiable** — the tenant can replay their events to reproduce their state (CN-4-009), proving the export is complete and consistent.
- Cross-tenant shared data (e.g., anonymised benchmarks the tenant opted into) is **not** included — it belongs to the platform, not the tenant.

**Export cost note:** The guarantee (complete + verifiable by replay) is concept-level. The cost and performance of replay-to-verify (e.g., using snapshots or checkpoints to avoid full replay for large tenants) is an Architect-phase concern.

---

## 4. Provability

Isolation must be **provable by automated tests**, not just claimed (Section 8 #5). This ties to CN-4-019 (Doctrine Enforcement Checks).

### Leak-Detection Tests

| Test | What it proves |
|------|---------------|
| **Cross-tenant query** | A query running in tenant A's scope returns zero events/projections from tenant B. |
| **Cross-tenant command** | A command issued in tenant A's scope targeting tenant B's aggregate is rejected. |
| **Platform-scope audit** | Every platform-scope operation produces a platform audit entry. |
| **Advisor scope** | An advisor running in tenant A's scope cannot read tenant B's projections. |
| **Export completeness** | A tenant export, replayed, produces identical state to the live tenant. |
| **Projection isolation** | A projection rebuild for tenant A processes zero events from tenant B. |
| **Snapshot isolation** | A snapshot for tenant A contains zero data from tenant B. |

These tests run in CI (CN-4-019). If any fails, the build is blocked.

---

## 5. Resource vs Data Isolation

**Foundation guarantees data isolation** — at the concept level, no data crosses tenant boundaries. This is structural and provable.

**Resource isolation** (one tenant's memory leak, CPU spike, or storage growth affecting another tenant on the same infrastructure) is an **Architect-phase concern** (Section 7, O11/O12). The concept does not specify infrastructure topology (shared servers, separate containers, database-per-tenant). It guarantees that regardless of the infrastructure choice, **data isolation holds**.

---

## 6. Three Whys

### Why does this matter?

A business owner must trust that their competitor — who may also use BOS — cannot see their prices, margins, customers, or sales. If isolation fails once, every tenant loses trust in the platform. One data leak and BOS's promise of truth is worthless — not just for the affected tenant, but for every tenant who hears about it.

### Why does it belong in the Kernel?

Isolation cannot be an engine-level responsibility. If each engine implements its own scoping, one engine's bug leaks data. The Kernel enforces isolation at the middleware level — before any engine logic runs, the tenant scope is validated. Engines receive only their tenant's data; they cannot opt out.

### Why this design?

A scope-object-at-the-boundary design (middleware injection + propagation) ensures isolation is structural, not per-query. The alternative — adding `WHERE tenant_id = X` to every query in every engine — is fragile: one missed clause leaks data. The scope object makes the default safe; an engine has to actively bypass the Kernel (which doctrine checks prevent) to break isolation.

---

## 7. Example — Platform Admin and Mama Amina

BOS platform admin in Dar es Salaam generates a regional report: "Total revenue for all tenants in Dar es Salaam, May 2026."

The platform scope produces aggregate numbers — total revenue per vertical category, total transactions, average ticket size. The report never shows Mama Amina's individual figures next to Mzee Hassan's. Each tenant's data contributed to the aggregate but is not identifiable.

The platform audit log records: who ran the query (platform admin principal), what scope was used (platform, region=Dar es Salaam), when, and what data class was returned (aggregate, not per-tenant).

Later, the admin needs to help Mama Amina with a support issue. The admin explicitly enters Mama Amina's tenant scope. The system records a dual-actor audit entry: platform admin (who) + Mama Amina's tenant (where). Mama Amina can later query her own audit log: "did anyone outside my business access my data this month?" BOS answers truthfully: "Yes — platform admin X, on this date, for support." The answer is provable because the audit log is append-only (CN-4-008) and the dual-actor entry is structurally required by the Kernel.

---

## 8. Boundaries

| Topic | Lives in |
|-------|----------|
| Actor model, principal types, dual-actor record structure | CN-4-007 (Identity Primitive) |
| Audit log fields, query API, platform audit trail | CN-4-008 (Audit Log Primitive) |
| Rate limits, anomaly detection, scope guard middleware | CN-4-016 (Security Primitives) |
| Advisor scope enforcement at runtime | CN-4-022 (Advisor Framework) |
| Platform roles and policies (who may use platform scope) | Term 1 (CTR-016, pending) |
| Resource isolation (noisy neighbour) | Architect phase |

---

## 9. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| How the dual-actor record is structured (admin + tenant scope) | CN-4-007 | Identity Primitive must define the composite actor record |
| How platform-scope operations are audited (separate trail) | CN-4-008 | Audit Log must support platform audit trail alongside tenant trail |
| How advisor scope boundaries are enforced at runtime | CN-4-022 | Advisor Framework owns runtime scope enforcement |
| Which platform roles may use platform scope | Term 1 (CTR-016) | Foundation provides the mechanism; Term 1 defines the policy |
| Anonymised aggregate definition for cross-tenant benchmarking | CN-4-022 | What "anonymised" means technically |
| **Right to erasure vs append-only immutability** | Future decision + Architect | A tenant may request data deletion (GDPR Article 17). But events are append-only and immutable (CN-4-001, Assertion 3). Likely resolution: **crypto-shredding** — destroy the encryption key for that tenant's data; the events remain in the store (hash chain intact) but are unreadable. This is not resolved in this doc; it requires a future decision and Architect-level specification of the encryption-per-tenant mechanism. |

---

*— End of CN-4-006 —*
