# Term 4 — Foundation
## Section 10 — Relationships with Other Terms

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Status:** For Overseer review.
> **Governing decisions:** D-004 (neutral), D-007 (scope resolutions), D-008 (consent/messaging).
> **Glossary:** *Foundation* = this Term; *Kernel* = the artifact it designs (see `MASTER-GLOSSARY.md`).

---

## Foundation Depends On

Foundation is the bottom. It depends on no other Term for materials.

| Term | Why Foundation needs it |
|------|----------------------|
| **Term 7 — Integration & Coherence** | Validates that Foundation's doctrine does not accidentally block legitimate use cases. Term 7 is the coherence check, not a provider of materials. |

---

## Depended On By

Every other Term builds on Foundation's materials. This table is consistent with Section 5 (Audiences Served).

| Term | What it depends on from Foundation |
|------|-----------------------------------|
| **Term 5 — Universal Engines** | All 9 primitives, value shapes (Saleable Line, Tender), event store, command bus, projection framework, engine isolation contracts, Advisor Framework contract |
| **Term 6 — Vertical Engines** | Everything Term 5 gets, plus extension points + registration API (CN-4-020) |
| **Term 1 — Platform Stewards** | Audit log primitive, identity primitive (incl. platform scope), security kernel, doctrine-enforcement gate, Developer-AI boundary declaration |
| **Term 2 — Regional Distribution** | Identity primitive, audit log, Document primitive, Advisor Framework contract (for agent advisor) |
| **Term 3 — Tenant Experience** | Projection framework, Time Authority, Document primitive, Decision Journal fields (for explainability), Advisor Framework (role-scope binding) |
| **Term 7 — Integration & Coherence** | Event-driven contracts (BO-5), document verification logic (CTR-017), Compliance DSL (for pack validation), engine contracts (for coherence checks) |

---

## Live Cross-Term Picture

### Inbound CTRs (Other Terms → Foundation)

| CTR | From | Topic | Status |
|-----|------|-------|--------|
| CTR-001 | Term 6 | Saleable Line / Tender as value shapes | NEGOTIATING |
| CTR-003 | Term 5 | Checkout: Foundation provides shapes, not engine | NEGOTIATING |
| CTR-007 | Terms 5, 2, 3 | Advisor Framework contract + model identity | NEGOTIATING |
| CTR-009 | Term 3 | AI explainability fields in Decision Journal | ACCEPTED |
| CTR-010 | Term 7 (all) | Advisory-only invariant holds everywhere | OPEN |
| CTR-014 | Term 1 | AI Mode cost governance + model approval | OPEN |
| CTR-021 | Term 5 | Outreach/promotions over channels + consent (D-008); consent primitive gates outreach (BO-7) | OPEN |

### Outbound CTRs (Foundation → Other Terms)

| CTR | To | Topic | Status |
|-----|-----|-------|--------|
| CTR-008 | Term 1 | Developer-AI governance + model registry | OPEN |
| CTR-016 | Term 1 | Platform scope as first-class parallel scope | OPEN |
| CTR-017 | Term 7 | Document verification: Foundation logic / Term 7 surface | OPEN |
| CTR-018 | Term 6 | Extension Points include registration API concept | OPEN |

### Boundary Objects

| BO | Topic | Foundation's Role |
|----|-------|------------------|
| **BO-1** | Saleable Line / Tender / Checkout | **Lead** — defines value shapes (CN-4-021); Term 5 builds Checkout engine |
| **BO-2** | Advisor Framework + Model Identity | **Lead** — defines the framework contract + Decision Journal schema (CN-4-022) |
| **BO-3** | Developer-AI Boundary + Governance | **Co-lead** (boundary) — declares Developer-AI out-of-kernel (CN-4-023); Term 1 owns governance |
| **BO-5** | Naming, Scope Policy, Event Contracts | **Lead** — defines `<engine>.<noun>.<verb>.v<n>`, event shape, scope policy |
| **BO-6** | Decision Journal & Audit Fields | **Lead** — defines what every audited/AI action records |
| **BO-7** | Messaging Channels | **Participant** — consent primitive governs outreach (D-008); Term 7 leads |

---

## Cross-Term Hand-Offs

| Direction | Hand-Off |
|-----------|----------|
| → **Term 5** | "Here are the 9 primitives, the Saleable Line and Tender value shapes (CN-4-021), the Advisor Framework contract (CN-4-022), the event store, and the command bus. Build your engines on these." |
| → **Term 6** | "Here are the extension points and registration API (CN-4-020). Follow the manifest + compatibility check to add new verticals without touching the Kernel." |
| → **Term 1** | "Here is the audit log primitive, the identity primitive with platform scope (CTR-016), the Developer-AI boundary declaration (CTR-008), and the doctrine-enforcement gate. Govern the platform with these." |
| → **Term 2** | "Here is the identity primitive, the audit log, the Document primitive, and the Advisor Framework contract. Build the agent experience on these." |
| → **Term 3** | "Here is the projection framework, the Time Authority, the Document primitive, the Decision Journal fields for explainability (CTR-009 resolved), and the Advisor Framework role-scope binding. Define your read-models and tenant UX against these." |
| → **Term 7** | "Here is the document verification logic and hash check (CTR-017). Build the public verification surface and offline cache around it. Here are the event contracts (BO-5) and the Compliance DSL for pack validation." |
| ← **Term 7** | "Your event store doctrine breaks SMS receipt sending under intermittent connectivity. Help?" — Term 7 tests Foundation's doctrine against real-world use cases and raises issues where the doctrine needs to accommodate edge conditions. |

---

## Consistency Check

| Check | Result |
|-------|--------|
| Section 10 "depended on by" matches Section 5 audience table | ✓ No mismatch |
| Hand-offs name only materials Foundation owns (Section 3) | ✓ No scope creep |
| CTR list matches current CTR Register (11 CTRs: 7 inbound + 4 outbound) | ✓ All accounted for |
| BO list matches CROSS-TERM-COLLABORATION.md | ✓ BO-1 through BO-7 (Foundation participates in 6) |
| No new CTRs created | ✓ Section 10 reports existing ones only |

---

*— End of Section 10 —*
