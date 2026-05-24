# Term 4 — Foundation
## Section 5 — Audiences Served

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Status:** For Overseer review.
> **Governing decisions:** D-004 (neutral, audience per Brief §5).
> **Glossary:** *Foundation* = this Term; *Kernel* = the artifact it designs (see `MASTER-GLOSSARY.md`).

---

## Audiences

Foundation has no end-user audience. Tenants, agents, and platform staff never interact with the Kernel directly — they see only the engines and screens built above it. There is no "For Mama Asha" section in Foundation docs; that belongs to Term 3.

Foundation's audience is **other Terms** and **Architects**.

| Audience | What it gets from Foundation |
|----------|------------------------------|
| **Term 5 — Universal Engines** | All nine primitives, value shapes (Saleable Line, Tender), event store, command bus, projection framework, engine isolation contracts, Advisor Framework contract |
| **Term 6 — Vertical Engines** | Everything Term 5 gets, plus: extension points and the registration API (CN-4-020) for adding new verticals without touching the Kernel |
| **Term 1 — Platform Stewards** | Audit log primitive, identity primitive (including platform scope), security kernel, doctrine-enforcement gate, Developer-AI boundary declaration |
| **Term 2 — Regional Distribution** | Identity primitive, audit log, Document primitive, Advisor Framework contract (for the agent advisor) |
| **Term 3 — Tenant Experience** | Projection framework (read-models for dashboards), Time Authority, Document primitive, Decision Journal fields (for explainability), Advisor Framework (role-scope binding) |
| **Term 7 — Integration & Coherence** | Event-driven contracts (BO-5), document verification logic (hash check, CTR-017), Compliance DSL (for pack validation), engine contracts (for coherence checks) |
| **Architects** | All CN-4-xxx concept documents, translated into specifications, schemas, and implementation directives in the Architecture Phase |

**Architects are a future audience.** During the Concept Phase, the working audience is the other six Terms. Architects consume Foundation's output after all proposals are complete and ratified (Charter §10: Concept Phase → Architecture Phase → Implementation Phase). The concept documents are written in a tone Architects can translate — peer-technical, precise, with trade-offs and constraints surfaced — but the immediate readers are the Terms.

## Foundation's Promise

Stated once in Section 1 (Mission) and not duplicated here:

> *If you build on the Kernel according to our doctrine, you cannot break the system by accident, and what you build today will still be defensible a decade from now.*

Every row in the table above is a specific fulfilment of this promise.

## Extensibility

This audience table is current, not closed. A new consumer plugs in via the extension points (CN-4-020) without changing the Kernel. An existing consumer needing a new material files a CTR, which Foundation evaluates against the restraint test: a neutral, widely-usable material may be added; anything specific to one vertical, country, or integration stays with the requesting Term.

---

*— End of Section 5 —*
