# Master Glossary

> **Owner:** Term 7 — Integration & Coherence (concept CN-7-003)
> **Status:** Authoritative — when a term-of-art is used differently across Terms, this file resolves it
> **Relationship:** Extends Charter §12. If this file and a Term Brief disagree on a word, this file wins; if this file and the Charter disagree, the Charter wins.

Every Term uses these words the same way. New entries are added by Term 7 when a cross-Term ambiguity is found.

| Term | Authoritative meaning | Note |
|------|-----------------------|------|
| **Foundation** | The **Term** (number 4) that designs the lowest layer. It is people/concept work, not the software. | Use "the Foundation Term". (D-004) |
| **Kernel** | The **artifact** the Foundation Term designs: event store, command bus, primitives, security, replay. | Use "the Foundation Term designs the BOS Kernel". (D-004) |
| **Concept Lead** | The human with **final authority** over every section, proposal, and decision. | Nothing is final without the Concept Lead. |
| **Overseer** | Term 7 acting as coordinator: routes/mediates CTRs, checks coherence, integrates work. **Never overrides the Concept Lead.** | See `CROSS-TERM-COLLABORATION.md` §6. |
| **CTR (Cross-Term Request)** | A formal, tracked request from one Term to another to agree a shared contract. | Register: `CROSS-TERM-REQUEST-REGISTER.md`. |
| **Boundary Object** | One artifact several Terms co-own and must agree on. | BO-1..BO-6 in the collaboration doc. |
| **Saleable Line / Charge** | The abstract payable line any vertical produces and Checkout settles. | Defined by Term 4 (CN-4-021), D-001. |
| **Tender** | A payment applied at Checkout (cash, mobile money, card, credit). | D-001. |
| **Universal Checkout** | The shared layer that settles tender and issues receipts over Saleable Lines, knowing no vertical. | D-001. |
| **Advisor** | A runtime AI plug-in defined by audience + data scope + model; **advisory only**, journalled. | D-002A; framework in CN-4-022. |
| **Developer-AI** | A **build-time** capability **outside the kernel** that briefs the implementer (Claude Code) to make changes; never runs in production, never writes state. | D-002B; boundary in CN-4-023. |
| **Regional Agent** | The compliance-accountable distributor in one region. Referral never replaces this role. | Charter Law 6; D-003. |
| **Proposal** | A `.md` concept document describing *how the system will be* — **no code**. | Charter §10; collaboration doc §5. |
| **Party** | A **business-domain entity** the business interacts with — customer, supplier, employee, or other external entity. A Foundation primitive. | CN-4-011; Charter §4.1. **Distinct from Actor.** |
| **Actor** | The **system identity** recorded on every event — who acted (human, advisor, system, machine). Not a business entity. | CN-4-007. **Distinct from the Party primitive.** |
| **Compensating Event** | A new event that corrects the effect of a prior event without deleting or modifying it. | CN-4-001; D-007 #3. |
| **Fold** | The deterministic function that replays a sequence of events to produce current state; defined per primitive. | CN-4-001; CN-4-011. |
| **Replay** | Running events through fold logic to reconstruct state at any point in time. | CN-4-001; CN-4-009. |
| **Site** | An **intra-tenant location/site partition** (a shop, branch, hotel property, outlet, clinic) — a Term 5 **payload** concept (`site_id`), not a Foundation envelope field. | CN-5-101. The informal "branch-level" scope in CN-4-005/CN-4-011 means **site-level**; neutral term is "site" (D-004). |
| **Scope level** | Where an engine operation runs: **site** (intra-tenant partition), **tenant** (whole tenant / "business-wide"), or **platform** (parallel cross-tenant, rare, dual-audited). | CN-5-101; CN-4-006 (platform scope). |
| **Universal-engine Invariant (UI-NN)** | A **state-shape** rule that must hold across two or more universal engines (cross-engine), distinct from per-engine business rules and from primitive folds. Numbered UI-01, UI-02, … The catalog is living (mechanized invariants become DC checks in CN-4-019 via CTR). | CN-5-102. Boundary: cross-engine state-shape, not behavior (CN-5-100/101 own behavior). |

*— End of Master Glossary —*
