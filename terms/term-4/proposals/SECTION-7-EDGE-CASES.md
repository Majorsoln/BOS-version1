# Term 4 — Foundation
## Section 7 — Edge Cases to Consider

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Status:** For Overseer review.
> **Governing decisions:** D-004 (neutral), D-007 (scope resolutions), D-008 (consent/messaging), D-009 (freeze doctrine).
> **Glossary:** *Foundation* = this Term; *Kernel* = the artifact it designs (see `MASTER-GLOSSARY.md`).

---

## Edge Cases

The hard questions Foundation must answer, sorted into two groups: those already settled by a decision, and those that remain open for a CN doc to resolve.

---

### Resolved — Settled by Decision

These are closed. Section 7 records the resolution so they are not re-litigated.

| # | Edge Case | Resolution | Decision |
|---|-----------|------------|----------|
| R1 | A bug emits a wrong event. Charter says no deletion. So what? | Compensating event appended. Truth = deterministic fold of all events including compensations, per-primitive fold logic. | D-007 #3 |
| R2 | Platform admin queries "all tenants in a region this month." Allowed? | Platform scope is a first-class parallel scope with its own audit trail — not an exception to tenant isolation. Platform roles defined by Term 1. | D-007 #4 |
| R3 | Human, system, AI advisor, machine — how distinguished in audit log? | Four distinct principal types: human, advisor (model id/version + scope), system (fine-grained `system:<component>`), machine (API client + credential ref). | D-007 #4 |
| R4 | Platform admin acts "as" a tenant for support. Who is the actor? | Both recorded: the human principal (admin identity) and the scope they are operating within (tenant scope). CN-4-007 must make the dual-actor record explicit (human principal + operating scope), and CN-4-008 must ensure the audit log captures both. | D-007 #4 |
| R5 | An API client (machine) calls BOS. What identity? | Machine principal type with API client identifier and credential reference. | D-007 #4 |
| R6 | AI advisor's query returns cross-tenant data. What stops it? | Advisor scope enforcement is a runtime Kernel responsibility. Scope guard blocks reads outside granted scope regardless of engine implementation. | D-007 #6, D-002A |
| R7 | AI suggestion was wrong and harmful. Liability? | Decision Journal records: model id/version, prompt ref, data ref, recommendation, rationale, human decision. Advisory-only (Law 3) — the human decided to act. Journal provides the evidence trail. | D-002A, D-007 #6 |
| R8 | Document number needed and system is offline. | Deterministic, branch/terminal-scoped, block-reserved for DEGRADED mode. Blocks reconciled on recovery to NORMAL. | D-007 #12 |
| R9 | QR code verification without internet. | Foundation owns verification logic + hash check. Term 7 owns the offline/cached verification surface. | D-007 #7, CTR-017 |
| R10 | Projection rebuild blocks the dashboard. Contingency? | Stale-but-available projections serve reads during rebuild. DEGRADED mode if rebuild is too disruptive. Auto NORMAL→DEGRADED (audited); human-gated deeper/recovery. | D-007 #9 |
| R11 | Snapshot gets out of sync. Tenant sees stale data. | Non-truth marker type + CI doctrine check. Snapshots are never source of state; always rebuildable from events. | D-007 #10 |
| R12 | Compliance DSL: what if a rule can't be expressed? | Reviewed, sandboxed extension mechanism; guarded against overuse so the escape hatch does not become the default. | D-007 #8 |
| R13 | DSL grammar extended — how without breaking existing packs? | Extension mechanism is sandboxed. New extensions do not modify existing grammar; existing packs are unaffected. | D-007 #8 |
| R14 | `datetime.now()` in engine logic breaks replay. | Forbidden by Charter doctrine. Enforced by the Time Authority clock protocol (CN-4-014) and doctrine enforcement checks in CI (CN-4-019). | Charter §2, CN-4-014, CN-4-019 |
| R15 | Tenant closes day at 11pm local. "Day" boundary on replay? | Fiscal time zone anchored at registration, stored as event. "Day" always computed from the registered fiscal zone — deterministic, never ambient. | Section 3 (Cluster E, ratified) |
| R16 | "Close period" — is it a Ledger concept? | No. Period close is Accounting (Term 5). The Ledger primitive knows only entries and balances. | D-007 #13 |
| R17 | Retroactive compliance change — replay with new rules or freeze old events? | Freeze. Events are interpreted under the rules active at emission. A later rule change applies only to new events. Each event references the compliance-pack version active at creation. | D-009 |
| R18 | Law changes. Old documents under old law; new under new. How enforced? | Documents reference the compliance pack + template version at issuance. Old documents stand under old law; new documents follow new law. | D-009 |
| R19 | Outreach/marketing message sent without customer consent. | Consent primitive (CN-4-011) governs: outreach requires recorded consent; opt-out is honoured. No send without consent (D-008 guardrail). | D-008 |

---

### Open — To Be Resolved in CN Docs

These are assigned to specific concept documents. Each will be resolved when that CN doc is written.

| # | Edge Case | Assigned CN | Notes |
|---|-----------|-------------|-------|
| O1 | Two commands arrive at the exact same nanosecond. Ordering? | CN-4-002 | Event Store Contract must define ordering guarantees (sequence numbers, tie-breaking). |
| O2 | Event emitted but projection update failed. Dashboard stale. Recovery? | CN-4-010 | Projection Framework must define failure recovery (re-process from last checkpoint, idempotent handlers). |
| O3 | Hash chain broken (disk corruption, attack). Recovery protocol? | CN-4-003 | Must define: detection mechanism, alert protocol, partial chain verification, recovery/quarantine procedure. |
| O4 | Event store at massive scale. Partitions/archives? | CN-4-002 + Architect phase | CN-4-002 guarantees append-only, immutable, and replayable at any scale. The partitioning mechanism is deferred to the Architect phase. |
| O5 | Projection rebuild takes hours during business hours. Modes? | CN-4-009 + CN-4-017 | Replay Engine defines the rebuild strategy; Resilience Modes defines the DEGRADED transition. Intersection of two docs. |
| O6 | Tenant data export (GDPR-style right). How scoped from multi-tenant store? | CN-4-006 | Isolation doc must define how "this tenant's events" are identified and extracted without exposing cross-tenant data. |
| O7 | Engine A wants to know "is month closed in Accounting?" No direct calls. | CN-4-005 | Engine Contract Model must define the event-subscription pattern — Engine A subscribes to a status event, never calls Engine B directly. |
| O8 | New engine subscribes to an existing event. Does the emitting engine need to know? | CN-4-005 + CN-4-020 | No — subscription framework guarantees decoupling. Engine Contract + Extension Points must define how subscriptions are registered without modifying the emitter. |
| O9 | Engine emits malformed events. How does Foundation isolate the damage? | CN-4-005 + CN-4-004 | Engine Contract defines event schema validation; Command Bus rejects malformed emissions before they enter the store. |
| O10 | Tenant asks "did anyone outside our business access our data?" Can BOS answer? | CN-4-008 + CN-4-006 | Audit Log records every access with actor + scope; Isolation ensures scoping. BOS can answer — audit log + tenant scope makes this queryable. |
| O11 | Two tenants share a server. Memory leak in one — does the other notice? | CN-4-006 + Architect phase | CN-4-006 guarantees data isolation (concept). Resource isolation is deferred to the Architect phase. |
| O12 | AI suggestion ignored. Same situation tomorrow — suggest again or stay quiet? | CN-4-013 + CN-4-022 | Decision Journal records prior suggestion + rejection. Advisor Framework can define a "previously suggested" check. Policy is per-advisor, not Foundation-wide. |
| O13 | Tenant edits document template. Old documents immutable. Template versioning? | CN-4-012 | Document Engine must define: template is versioned; documents reference the template version at issuance; old documents are never re-rendered. |
| O14 | Compliance pack has a bug. Rollback speed? Tenant notification? | CN-4-015 | DSL Framework must define: pack versioning, rollback mechanism, notification interface (Term 3 surfaces the notification). |
| O15 | Audit log queried by regulators with massive date ranges. Pagination? Indexes? | CN-4-008 | Audit Log must define the query contract (pagination, scoping). Pre-computed indexes are Architect-phase. |

---

### Needs Ruling

None. All 34 edge cases are either resolved by a decision (19) or assigned to a CN doc (15). No new CTRs are required.

---

*— End of Section 7 —*
