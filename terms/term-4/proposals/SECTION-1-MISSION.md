# Term 4 — Foundation
## Section 1 — Mission

> **Status:** Ratified in concept (Concept Lead + Overseer agreed).
> **Governing decisions:** D-004 (concept-writing principles), D-006 (collaboration model).
> **Glossary:** *Foundation* = this Term; *Kernel* = the artifact it designs (see `MASTER-GLOSSARY.md`).

---

## Mission

The Foundation Term designs the BOS Kernel — the lowest layer of the system. Every engine, portal, dashboard, and document sits on it. If the Kernel cracks, BOS cracks. If the Kernel is sound, BOS can grow for decades — absorbing verticals not yet imagined and countries not yet onboarded — without a full rewrite.

The Kernel is what makes BOS **legally defensible**. The event store, hash chain, audit log, and replay engine together let an auditor or a court reconstruct exactly what happened, prove nothing was silently changed, and verify any issued document years after it was signed. This is not a feature of BOS; it is the floor BOS stands on.

The Kernel is **where the six Laws of the Charter become mechanical**. State derived only from events, isolated engines, advisory-only AI, vertical flexibility, configured compliance, regional distribution — these are not policies developers must remember. They are invariants the Kernel enforces: through primitives that admit only legal operations, through a command bus that rejects illegal ones, and through automated doctrine checks that block any change which violates them before it reaches production.

The Kernel is **neutral by ignorance**. It does not know Retail or Hotel. It does not know Tanzania or Kenya. It does not know M-Pesa or any tax authority. This is not a gap; it is the design. Verticals, countries, and integrations are built on top of the Kernel by other Terms using the primitives Foundation provides. Whenever a Foundation concept is tempted to name a vertical, a country, or an external system, the abstraction is wrong and must be rebuilt.

The Kernel is **invisible at runtime, immovable at build time**. Tenants, agents, and platform staff never see it; they see only the engines and screens built above it. Developers, however, meet the Kernel as a wall: the doctrine gate rejects any change that breaks an invariant — no matter how small, how urgent, or how well-intentioned — before it can be released.

Foundation's promise to every other Term:

> *If you build on the Kernel according to our doctrine, you cannot break the system by accident, and what you build today will still be defensible a decade from now.*

---

## Exclusions (per D-004)

The Mission states *that* these exist and *why* they matter. The CN docs state *what* they are:

- Hash algorithm, storage engine, replay mechanism — CN-4-002, CN-4-003, CN-4-009.
- Primitive list and shapes — CN-4-011.
- Compliance DSL grammar — CN-4-015.
- Advisor Framework contract and role-scope binding — CN-4-022 (see also D-005).
- Doctrine-enforcement gate and its checks — CN-4-019, CN-4-023.
- Kernel boundary against Developer-AI — CN-4-023.
- Performance budgets — CN-4-009, CN-4-018.

---

*— End of Section 1 —*
