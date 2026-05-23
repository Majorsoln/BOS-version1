# Cross-Term Collaboration

> **Owner:** Term 7 — Integration & Coherence (concepts CN-7-001, CN-7-002, CN-7-006)
> **Status:** Process document — every Term reads this before writing proposals
> **Reading Order:** Charter → DECISION-LOG → this → your Term Brief

---

## 1. Why This Exists

The seven Terms work in parallel. The danger (Charter, Term 7 mission) is that each produces something internally brilliant that **does not fit** the others. A tenant screen (Term 3) asks for a number that no engine (Term 5) emits. A vertical (Term 6) needs a primitive that Foundation (Term 4) hasn't defined. An agent flow (Term 2) assumes a policy that platform (Term 1) never set.

This document gives every Term two things:
1. A **map** of who they overlap with (so they read the right neighbours before proposing).
2. A **mechanism** — the Cross-Term Request (CTR) — to formally ask a neighbour to agree on a shared thing.

**Rule:** A Term's proposal is **not "ready for the Architect phase"** while it has any **open CTR** to another Term. Shared things must be agreed before design begins.

---

## 2. The Interaction Map

What each Term **provides** to others and **consumes** from others. Read your row, then read the Briefs of the Terms you touch.

| Term | Provides to others | Consumes from others |
|------|--------------------|----------------------|
| 1 — Platform Stewards | Pricing, engine catalog, agent/referral rules, compliance approval, AI governance | Foundation primitives (4); agent/tenant/engine realities (2,3,5,6); coherence (7) |
| 2 — Regional Distribution | Agent experience, onboarding, referral flow, L1 support | Agent rules + commission policy (1); tenant first-day hand-off (3); identity/audit (4); coherence (7) |
| 3 — Tenant Experience | UX requirements that drive engine evolution | Pricing/compliance (1); onboarding hand-off (2); primitives (4); engine data (5,6); integrations (7) |
| 4 — Foundation | Primitives, event store, command bus, isolation, AI guardrails, doctrine gate | Use-case validation from all; coherence (7) |
| 5 — Universal Engines | Accounting/Cash/Inventory/Procurement/HR/Reporting + Checkout/Tender + advisors | Primitives (4); catalog (1); vertical events (6) |
| 6 — Vertical Engines | Vertical events + saleable lines into checkout | Primitives (4); universal engines to subscribe (5); catalog (1) |
| 7 — Integration & Coherence | Coherence reviews, conflict resolution, integration gateway, the CTR register | All Terms' drafts (Half A); event contracts (4,5,6) |

---

## 3. Boundary Objects (shared things multiple Terms co-own)

A **boundary object** is one artifact that several Terms must agree on. Each has one **lead** Term (who drafts it) and **participating** Terms (who must accept it). These are the highest-risk areas for incoherence.

| ID | Boundary Object | Lead | Participating | What must be agreed |
|----|-----------------|------|---------------|---------------------|
| BO-1 | **Saleable Line / Tender / Checkout** (D-001) | 4 (primitive) → 5 (engine) | 6, 3, 1, 7 | Shape of a line; what checkout owns vs verticals; tender methods; receipt issuance |
| BO-2 | **Advisor Framework + Model Identity** (D-002A) | 4 | 5, 2, 1, 3, 7 | Advisor contract (audience/scope/model); Decision Journal fields; advisory-only enforcement |
| BO-3 | **Developer-AI boundary + governance** (D-002B) | 4 (boundary) + 1 (governance) | 7 | Out-of-kernel guarantee; doctrine gate; who triggers/approves builds |
| BO-4 | **Agent attribution + referral model** (D-003) | 1 (policy) → 2 (experience) | 3, 7 | Multi-agent rules; who owns a referred tenant; transfer/arbitration |
| BO-5 | **Naming, scope policy, event contracts** (Charter §8.1) | 4 | all | `<engine>.<noun>.<verb>.v<n>`; branch- vs business-scope; stable event shapes |
| BO-6 | **Decision Journal & audit fields** | 4 | 1, 2, 3, 5, 6, 7 | What every audited/AI action records |

---

## 4. The Cross-Term Request (CTR)

When your proposal needs something from another Term — a primitive, a field, an event, a policy, a UI contract — you **file a CTR** in `CROSS-TERM-REQUEST-REGISTER.md`. This makes the dependency explicit and trackable instead of an untested assumption.

### 4.1 CTR Lifecycle
```
DRAFT ─► OPEN ─► NEGOTIATING ─► ACCEPTED ─► CLOSED
                      └────────► REJECTED ─► (escalate to Term 7 → Concept Lead)
```

### 4.2 Rules
1. **One CTR = one shared agreement.** Don't bundle unrelated needs.
2. **The receiving Term must respond** (Accept / Reject-with-reason / Negotiate). Silence is not acceptance.
3. **Term 7 triages** every CTR, links it to the Open Questions Registry (CN-7-006), and mediates conflicts (CN-7-002).
4. A proposal cannot advance to the Architect phase with an **open CTR** against it.
5. Rejections always carry a **reason** (Term 7 discipline: "no, with reasons").

### 4.3 CTR Template

```markdown
### CTR-NNN — <short title>
- **From Term:** N
- **To Term(s):** N (, N)
- **Decision / Topic:** D-00X / <topic>
- **Boundary Object:** BO-N (if any)
- **What is needed:** <the concrete thing — a field, event, primitive, policy, contract>
- **Why:** <the user/system reason>
- **Proposed contract:** <your suggested shape, so the other Term has something to react to>
- **Status:** DRAFT | OPEN | NEGOTIATING | ACCEPTED | REJECTED | CLOSED
- **Resolution:** <filled when closed — the agreed contract, or the reason for rejection>
```

---

## 5. How a Term Works, Start to Finish

> **Output is concept, not code.** A Term's job is to **discuss and describe *how the system will be*** — in prose, tables, flows, examples, and contracts. **No Term writes code, no implementations, no schemas-as-code during this phase** (Charter §10). The deliverable is a `.md` concept document that an Architect can later design from, and only after that an Implementer codes. Discussion now; code much later, and only once all proposals pass.

A Term does **not** write its proposal first and seek approval later. It earns the proposal through discussion. The order is deliberate:

1. **Read** the full stack: **Charter → Decision Log → this doc → your Brief → your open CTRs**.
2. **Identify** your boundary objects (§3) and neighbours (§2).
3. **Bring points to the table — section by section.** For each part of your scope, the Term prepares its *arguments and options* (the "why", the trade-offs, the open questions) and presents them to the **Concept Lead** for discussion. This is a conversation, not a submission.
4. **Discuss and agree.** The Concept Lead (and the Overseer / Term 7) work through each section with the Term. A section may be **improved, extended, or have new features/workflows added** during this discussion.
5. **Only after a section is agreed is its proposal written** (the CN-x-xxx concept doc). Writing follows agreement; it does not precede it.
6. **File CTRs** for every dependency on a neighbour, and resolve them (Term 7 mediating).
7. A Term's proposal is **"ready"** only when: every section is agreed with the Concept Lead, every CTR is closed, and the Overseer confirms coherence.

> The system has **no complete proposal** until **all seven Terms' proposals have passed**. Until then, everything is in discussion and subject to change.

---

## 6. Authority & Approval

Collaboration has a clear chain. No work becomes final by a Term acting alone.

| Role | Authority |
|------|-----------|
| **Concept Lead (the human)** | **Final authority.** Every section, every proposal, every decision requires the Concept Lead's agreement. Nothing is "done" without it. |
| **Overseer / Term 7 (the coordinator)** | Coordinates, checks coherence, routes and mediates CTRs, integrates work. A Term must also align with the Overseer — but the Overseer never overrides the Concept Lead. |
| **Each Term** | Owns its scope and proposals, but cannot finalise anything without agreement from **both** the Concept Lead and the Overseer. |

**Rule:** Every Term must agree with the Concept Lead — and with the Overseer — before a section is written or a proposal is marked ready. Discussion is step-by-step; agreement is explicit, not assumed.

---

## 7. How AI-Agent Terms Collaborate (Operating Model)

When each Term is run by a separate Claude Code agent, the agents collaborate the way BOS engines do — through **artifacts in git**, never by direct real-time calls. The repo is the shared memory: *what is not committed does not exist.*

**Five rules:**
1. **One Term = one agent = its own files.** New proposals live under `terms/term-N/proposals/CN-N-xxx.md`. (Existing root Briefs stay where they are for now.)
2. **One owner per file.** An agent **never edits another Term's files** — this removes merge conflicts entirely.
3. **Communication is via the CTR register only.** Need something from another Term? *Append* a CTR — don't touch their files. The receiving Term reads CTRs addressed to it and responds in the register.
4. **The Overseer (Term 7) owns the shared files** (Decision Log, this doc, the register, the glossary) and is the only one who integrates/merges.
5. **Git:** each Term works on its own branch (e.g., `term-4/work`); the Overseer reviews and merges in sequence. On a single branch, an agent touches only its own folder.

**Each agent's loop (the briefing the Overseer gives it):**
```
Read: Charter → DECISION-LOG → this doc → your Brief → your open CTRs
  → bring your section's points to the Concept Lead and discuss
  → on agreement, write that section's proposal (CN-x-xxx) in your folder — concept prose / flows / contracts, NO code
  → file CTRs for dependencies; respond to CTRs addressed to you
  → mark "ready" only when sections are agreed AND CTRs are closed
```

**Trade-off:** async, file-mediated collaboration is slower than live chatter, but it is auditable, conflict-free, survives ephemeral sessions, and matches BOS doctrine (artifacts are truth). Real-time agent-to-agent talk is not reliable; the artifacts are.

> *"Six brilliant ideas that don't fit are worse than one mediocre idea that does. The CTR is how the ideas are made to fit — and the Concept Lead's agreement is how they become real — before code is ever written."*

*— End of Cross-Term Collaboration —*
