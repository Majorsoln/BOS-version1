# Term 4 — Foundation
## Section 1 — Mission (Proposal v1)

> **Status:** For discussion — not final until the Concept Lead and the Overseer (Term 7) both agree.
> **Following:** D-004 guiding principle — *the Mission carries only what is immutable; concrete choices that may evolve live in CN-4-xxx docs.*
> **Addresses:** B1 (legal defensibility), B2 (neutrality through restraint), B3 (the Six Laws made mechanical), C1 (invisible at runtime, immovable at build time).
> **Read first:** `BOS-CONCEPT-CHARTER.md`, `DECISION-LOG.md`, `CROSS-TERM-COLLABORATION.md`, `TERM-4-FOUNDATION.md`.

---

## Mission

Foundation is the kernel beneath BOS. Every engine, portal, dashboard, and document sits on what this Term designs. If Foundation cracks, BOS cracks. If Foundation is sound, BOS can grow for decades — absorbing verticals not yet imagined and countries not yet onboarded — without a rewrite.

Foundation is what makes BOS **legally defensible**. The event store, hash chain, audit log, and replay engine together let an auditor or a court reconstruct exactly what happened, prove nothing was silently changed, and verify any issued document years after it was signed. This is not a feature of BOS; it is the floor BOS stands on.

Foundation is **where the Six Laws of the Charter become mechanical**. State derived only from events, isolated engines, advisory-only AI, vertical flexibility, configured compliance, regional distribution — these are not policies developers must remember. They are invariants Foundation enforces: through primitives that admit only legal operations, through a command bus that rejects illegal ones, and through automated doctrine checks that block any change which violates them before it reaches production.

Foundation is **neutral by ignorance**. It does not know Retail or Hotel. It does not know Tanzania or Kenya. It does not know M-Pesa or any tax authority. This is not a gap; it is the design. Verticals, countries, and integrations are built on top of Foundation by other Terms using the primitives Foundation provides. Whenever a Foundation concept is tempted to name a vertical, a country, or an external system, the abstraction is wrong and must be rebuilt.

Foundation is **invisible at runtime, immovable at build time**. Tenants, agents, and platform staff never see it; they see only the engines and screens built above it. Developers, however, meet Foundation as a wall: the doctrine gate rejects any change that breaks an invariant — no matter how small, how urgent, or how well-intentioned — before it can be released.

Foundation's promise to every other Term is short:

> *If you build on us according to our doctrine, you cannot break the system by accident, and what you build today will still be defensible a decade from now.*

---

## Notes on what is deliberately NOT in this Mission (per D-004)

The following belong in CN-4-xxx docs, not in the Mission, because they are choices that may evolve:

- The specific hash algorithm, storage engine, or replay mechanism — CN-4-002, CN-4-003, CN-4-009.
- The list of primitives and their shapes — CN-4-011.
- The grammar of the Compliance DSL — CN-4-015.
- The shape of the Advisor Framework contract — CN-4-022.
- The exact doctrine-enforcement gate and its checks — CN-4-019, CN-4-023.
- The kernel boundary against Developer-AI — CN-4-023.

The Mission states *that* these exist and *why* they matter. The CN docs state *what* they are.

---

## Open points still to confirm with the Concept Lead

1. **"Decades" vs a number.** The Mission says "for decades." The original Brief said "20 years." Either is defensible. The Mission's job is to express a long-horizon commitment without inviting a literal audit of a year count.
2. **"Six Laws" reference.** The Mission names the Charter's Six Laws by their themes, not by number, so it remains readable if the Laws are ever renumbered or extended. Charter §2.1–§2.2 remains the canonical source.
3. **Audience.** This Mission is written for Architects and for other Term agents. It is not written for tenants or agents (who never read Foundation docs).

---

*— End of Section 1 Proposal —*
