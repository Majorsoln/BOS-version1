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
| **Compensating Event** | A new event that corrects the effect of a prior event without deleting or modifying it. | CN-4-001; D-007 #3. |
| **Fold** | The deterministic function that replays a sequence of events to produce current state; defined per primitive. | CN-4-001; CN-4-011. |
| **Replay** | Running events through fold logic to reconstruct state at any point in time. | CN-4-001; CN-4-009. |

*— End of Master Glossary —*
