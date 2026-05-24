# BOS Decision Log

> **Owner:** Term 7 — Integration & Coherence (concept CN-7-007)
> **Status:** Binding — every Term must honour these decisions in their proposals
> **Reading Order:** Read the Charter first, then this, then your Term Brief
> **Rule:** If a Term Brief predates a decision here, this log wins until the Brief is updated.

---

## 0. What This Document Is

This is the single, append-only record of **cross-Term decisions** made by the Concept Lead together with the Terms. Each decision states *what was decided*, *why*, *which Terms it touches*, and *what each Term must now produce or agree*.

Decisions are **not** the same as Charter laws. The Charter holds the non-negotiable doctrine; this log records concrete choices made within that doctrine. A decision may never contradict a Charter law — if it appears to, Term 7 escalates to the Concept Lead.

Each decision seeds one or more **Cross-Term Requests (CTRs)** — see `CROSS-TERM-COLLABORATION.md` and the live `CROSS-TERM-REQUEST-REGISTER.md`.

---

## D-001 — Universal Checkout (POS for any business type)

> **Date:** 2026-05-23 · **Status:** Accepted · **Charter check:** Honours Law 2 (engine isolation) and Law 4 (flexibility)

### Context
"POS" in the existing system is entangled with Retail. But every vertical needs to take payment and issue a receipt — a restaurant settles a table bill, a hotel settles a folio, a workshop collects on an accepted quote, a future clinic collects a consultation fee. Building a Retail-only POS cannot serve them; making one POS "know" every vertical would break engine isolation.

### Decision
Split checkout into **two layers**:

1. **Universal Checkout / Tender layer** — a shared surface that accepts payment, splits tender across methods (cash, mobile money, card, credit/obligation), computes change, and issues a receipt/document. It operates only on an abstract **Saleable Line / Charge** list. It does **not** know which vertical produced the lines.

2. **Verticals feed the lines** — Retail (basket), Restaurant (table bill), Hotel (folio), Workshop (accepted quote), and any future vertical produce `Saleable Lines` and hand them to checkout. The vertical never owns tender, change, or receipt issuance.

The result: any business — present or future — uses the **same** checkout by emitting its lines into it.

### What each Term must produce / agree
| Term | Must decide / produce |
|------|------------------------|
| Term 4 — Foundation | The `Saleable Line / Charge` and `Tender` shape. Decide: is Checkout a **primitive**, or a **universal engine** that uses existing primitives (obligation, document, ledger)? Define the boundary so verticals cannot leak into tender. |
| Term 5 — Universal Engines | A universal **Checkout / Tender Engine** (or a Cash-Engine extension): tender, splits, change, receipt issuance; Accounting + Cash subscribe to its events. |
| Term 6 — Vertical Engines | The **vertical → checkout hand-off**: each vertical emits its lines; defines nothing about payment. |
| Term 3 — Tenant Experience | **One** checkout UX shared across all verticals (cashier flow, split payment, receipt). |
| Term 1 — Platform Stewards | Is checkout an **always-on universal capability** for every tenant, or a catalog item priced in combos? |
| Term 7 — Integration & Coherence | Payment adapters (mobile money, card) attach at the **tender** boundary, not inside verticals. Verify all verticals truly share one checkout. |

### Seeds CTRs
CTR-001, CTR-002, CTR-003, CTR-004, CTR-005, CTR-006.

---

## D-002 — AI: Runtime Advisor Framework + Build-time Developer-AI

> **Date:** 2026-05-23 · **Status:** Accepted · **Charter check:** Must preserve Law 3 (AI is advisory only) at all times

This decision has **two parts**. They must never be confused, because confusing them is how the kernel loses its legal defensibility.

### Part A — Runtime Advisor Framework (inside BOS)

**Context.** BOS wants AI support at every level: tenants (business advice, BI/KPI explanations, reorder hints), agents (onboarding/support help), and platform staff. These are all *runtime* advisors that read data and make suggestions.

**Decision.** Create **one Advisor Framework**. An advisor is a plug-in defined by:
- **Audience** — which role/group it serves (platform steward, agent, a specific tenant role).
- **Scope** — the exact data it may read. Default: the current tenant only. Cross-tenant benchmarking is **opt-in only** and never exposes another tenant's identifiable data.
- **Model** — the model behind it, swappable without changing the framework.

Every advisor:
- is **advisory only** — it never commits state, never writes events, never approves a transaction (Law 3);
- logs every suggestion to the **Decision Journal**, recording the model **identity and version**, the data it saw (or a reference), the prompt reference, and whether a human accepted or rejected it;
- is constrained by Foundation's **AI Guardrails** (forbidden operations, scope guards).

**What each Term must produce / agree**
| Term | Must decide / produce |
|------|------------------------|
| Term 4 — Foundation | Extend AI Guardrails + Decision Journal to record **model identity/version**; define the **Advisor Framework contract** (audience, scope, model plug-in). |
| Term 5 — Universal Engines | Wire engine advisors (Inventory, Cash, Procurement) **and** the **BI / KPI advisor** for owners. |
| Term 2 — Regional Distribution | The **Agent Advisor** (helps agents with onboarding/support tasks). |
| Term 1 — Platform Stewards | Platform advisor + **AI governance**: model approval, the model registry, and how advisory-only is surfaced to staff. |
| Term 3 — Tenant Experience | How advice is **shown** per role, and how much of the reasoning is exposed (explainability). |
| Term 7 — Integration & Coherence | AI inside the integration pipeline (e.g., fraud hints) + **prompt-injection defence** for external data; verify advisory-only holds everywhere. |

### Part B — Developer-AI (build-time, OUTSIDE the kernel)

**Context.** We want an environment that helps **build and evolve BOS itself** — not a runtime feature.

**Decision.** Define a **Developer-AI environment** that lives **entirely outside the production kernel** and outside any tenant/agent/platform runtime. Its job:
1. **Know the codebases** — maintain an up-to-date understanding of how the system is structured and how its engines work.
2. **Receive an instruction** — the Concept Lead (human) gives it an idea / change request.
3. **Brief the implementer** — it shares system context with **Claude Code**, which is the **code implementer**.
4. **Implement via GitHub** — Claude Code makes the change as a pull request.
5. **Gate on tests** — the change merges and the **system version changes only after all tests pass**, including Foundation's doctrine-enforcement invariants (the 11+ architectural tests).

**Hard boundaries (non-negotiable):**
- Developer-AI **never runs inside the production kernel** and **never writes events or business state**. It produces *code proposals*, not runtime actions.
- Every change passes **human review + CI** (tests + doctrine invariants) before release.
- This is how Law 3 and "legally defensible kernel" survive: the runtime is never modified by an autonomous agent without a human-gated, test-gated pipeline.

**What each Term must produce / agree**
| Term | Must decide / produce |
|------|------------------------|
| Term 4 — Foundation | Declare the **kernel boundary**: Developer-AI is out-of-kernel; specify the **doctrine-enforcement gate** every proposed change must pass before release. |
| Term 1 — Platform Stewards | **Govern the Developer-AI environment**: who may trigger it, who approves changes/releases, the **model registry**, audit of build actions. |

### Seeds CTRs
CTR-007, CTR-008, CTR-009, CTR-010.

---

## D-003 — Multiple Regional Agents + Referral Program

> **Date:** 2026-05-23 · **Status:** Accepted · **Charter check:** Honours Law 6 (distribution is regional)

### Context
Should a region have **one** agent (with only a referral program), or **many** agents? A single agent per region is simple to govern but fragile: if that agent dies, is suspended, or underperforms, every tenant in the region is harmed (an explicit Term 2 edge case), and a monopoly can mistreat tenants on price and service.

### Decision
- **Multiple regional agents are allowed per region.** Compliance stays regional (Law 6 intact) — every agent is still accountable for their own region's documents. Competition improves coverage, price, and resilience.
- A **referral program** is added as an **acquisition layer on top**, not a replacement for agency. Referral brings new tenants/agents in; it does **not** transfer compliance accountability (a referrer who cannot file tax documents cannot be the responsible agent).

### What each Term must produce / agree
| Term | Must decide / produce |
|------|------------------------|
| Term 1 — Platform Stewards | Governance for **multiple agents per region**, **referral program rules**, **commission & attribution policy**, and **conflict/transfer arbitration**. |
| Term 2 — Regional Distribution | The **multi-agent experience**, the **referral flow** (who may refer: agents, tenants, public?), the **attribution model** (who "owns" a referred tenant), and fair-competition guardrails. Update the transfer workflow (CN-2-011) for attribution changes. |
| Term 3 — Tenant Experience | Tenant-side **agent discovery** and **referral touchpoints** (relates to Term 2 open question on public discovery). |
| Term 7 — Integration & Coherence | Verify referral does **not** create a de-facto "Global Agent" or bypass regional compliance accountability. |

### Seeds CTRs
CTR-011, CTR-012, CTR-013.

---

## D-004 — Concept-Writing Principles (Mission & Concept Docs)

> **Date:** 2026-05-23 · **Status:** Accepted · **Origin:** Term 4 Section 1 (Mission) review · **Charter check:** Reinforces §1.2 (legal defensibility), §1.4 (neutral by design), §2 (the six Laws), §8.2 (English body)

### Context
While reviewing Term 4's Mission section by section, the Term surfaced gaps and tensions. Resolving them produced principles that apply to **every** Term's concept work, plus specific guidance for the Foundation Mission.

### Decisions (cross-cutting — apply to all Terms unless noted)
1. **A Mission carries the *unchanging*.** Doctrine, truth, durability, restraint, isolation belong in the Mission. Anything that can change — performance budgets, decision-specific detail (e.g. Developer-AI under D-002B), illustrative figures — lives in the relevant CN doc, **not** the Mission. This keeps Missions from becoming living documents that need constant edits.
2. **No unverifiable hard numbers as commitments.** Replace "10/20 years" with **"decades, without periodic full rewrites"**; replace "50 verticals, 30 countries" with **"dozens of verticals, many countries."** Claim only what can be proven (this is what "legally defensible" demands). *This supersedes the conflicting figures in Term 4 Brief §1 ("20 years") and §2 ("10 years").*
3. **"Invisible" is scoped.** A kernel/engine is invisible to **runtime users** (tenants, agents); to **developers**, the kernel is the *wall* that rejects doctrine violations before production.
4. **Audience & tone follow the Term's Brief §5.** Foundation's audience is **other Terms + Architects** — peer-technical tone, and **no "For Mama Asha" section** in Foundation docs (that rule is Term 3's).
5. **Glossary terminology (see `MASTER-GLOSSARY.md`):** *Foundation* = the Term; *Kernel* = the artifact it designs. Use **"the Foundation Term designs the BOS Kernel."**

### Foundation-Mission specifics (Term 4 only)
The Foundation Mission must explicitly elevate three things that were previously only implied:
- **Legal defensibility** — hash chain + audit log + replay are what make BOS provable in court.
- **Neutrality by restraint** — Foundation knows no vertical, no country, no integration; this is its strength (promote Brief §14 into the Mission).
- **The six Laws as technical invariants** — Foundation is where the Laws become tests a developer cannot bypass (ties to CN-4-019).

### D1–D4 rulings (recorded)
- **D1 Performance:** out of the Mission; specified as a non-functional concern in CN-4-009 (Replay) and CN-4-018 (Snapshots).
- **D2 Developer-AI:** light touch in the Mission only; full boundary + doctrine-gate detail in CN-4-023.
- **D3 Figures:** removed per Decision 2 above.
- **D4 Audience:** Terms + Architects per Decision 4 above.

### Affects
Term 4 immediately; Decisions 1, 2, 4, 5 apply to all Terms' future concept docs.

### Deferred follow-up
- **Brief reconciliation pass** (Concept Lead approved, not yet scheduled): once Term 4's proposal is complete, scrub the legacy hard numbers ("10 years", "20 years", "50 verticals", "30 countries") from all seven Term Briefs so the Briefs match D-004. Until then, D-004 supersedes those figures.

---

## D-005 — AI Mode (Universal Per-Role Dashboard Advisor)

> **Date:** 2026-05-24 · **Status:** Accepted · **Charter check:** Honours Law 3 (advisory only) and §1.4 (neutral by design); it is the UI face of D-002A, not a new mechanism

### Context
Inspired by the "AI Mode" pattern now common on consumer products. BOS should offer the same on **every** dashboard — but bounded by BOS doctrine.

### Decision
**Every dashboard, for every role** — across tenants, platform stewards, and regional agents — includes an **"AI Mode"**: a per-role advisor surfaced on the dashboard (a prompt box for questions **plus** proactive, role-relevant advice, KPI/BI explanations, and alerts), built on the **single Advisor Framework (D-002A)**. Advice is tailored to the role's audience and strictly bounded by the role's data scope. **Enabled for all roles from the start** (not phased).

### Guardrails (restated from D-002A)
- **Advisory only** — never executes; any resulting action goes through the command bus with human approval (Law 3).
- **Scope/permission-bounded** — a role's AI Mode sees only what that role may see; no cross-role or cross-tenant leakage; cross-tenant benchmarking is opt-in only.
- **Reads projections only** — never writes events (replay-safe).
- **Journaled** — model id/version, prompt, data reference, suggestion, human decision (Decision Journal).
- **Explainable** — "show your work"; Kiswahili-first for tenants.
- **Cost-governed, offline-degraded, prompt-injection-defended.**

### Ownership
| Term | Owns |
|------|------|
| Term 4 | The Advisor Framework + role-scope binding (CN-4-022) |
| Term 3 | Tenant AI Mode UX + explainability (CN-3-017) |
| Term 1 | Platform-staff AI Mode + AI cost governance & model approval |
| Term 2 | Agent AI Mode (CN-2-013) |
| Term 5 | BI/KPI + engine advisors feeding it (CN-5-010) |
| Term 7 | Record as a Pattern (CN-7-004) so every dashboard implements it consistently; advisory-only coherence (CTR-010); prompt-injection (CN-7-115) |

### Seeds CTRs
CTR-014, CTR-015.

---

## D-006 — Collaboration Operating Model (`main` as shared truth)

> **Date:** 2026-05-24 · **Status:** Accepted · **Origin:** the Terms run as separate Claude Code sessions on separate branches; without a shared base they could not see each other's work (this caused duplicate Section 2 drafts and Term 4 not seeing D-004)

### Decision
1. **`main` is the single shared source of truth.** The Overseer publishes the shared docs (Charter, Decision Log, Glossary, Collaboration, Request Register, Changelog) and ratified proposals to `main`.
2. **Each Term authors ONLY its own files**, on its own branch created from `main`. The Overseer reviews and **merges ratified work into `main`**. Terms pull `main` to stay current.
3. **The Overseer (Term 7) does NOT author Term content.** Overseer duties: detect and fix coherence problems; **answer the Terms' questions**; **ensure every Term learns agreements and changes promptly** (via `main` + the Changelog); own and maintain the shared docs; review and merge.
4. The subagent-authored Term 4 Section 1/2 files were **discarded**; Term 4 authors its own Mission and Mission Question on its branch.

### Supersedes
The per-Term-branch note in `CROSS-TERM-COLLABORATION.md` §7 is now anchored to `main`.

---

## D-007 — Foundation Scope Resolutions (Section 3)

> **Date:** 2026-05-24 · **Status:** Accepted · **Origin:** Term 4 Section 3 (Scope · In) discussion · **Charter check:** Honours D-004 neutrality, Law 2 (isolation), Law 3 (advisory)

Ratified resolutions for what the Kernel provides:

| # | Resolution |
|---|-----------|
| 1 | **Saleable Line** & **Tender** are **value shapes** in Foundation (CN-4-021); **Checkout is a universal engine** in Term 5 built on Foundation primitives. *(CTR-001/003 — pending Term 5/6 acceptance.)* |
| 2 | Primitive set = the **nine**; **Consent is distinct from Approval**; **Workflow is distinct from Approval**. |
| 3 | **Compensating events:** no deletion ever; current truth = a **deterministic fold of all events including compensations**; the fold is defined per primitive. |
| 4 | Identity: a distinct **"advisor" principal** (model id/version + scope); **fine-grained system principals** (`system:<component>`); **platform scope is a first-class parallel scope** with its own audit trail — not an exception to isolation. |
| 5 | **Engine isolation** enforced by **both** runtime guards and CI doctrine checks. |
| 6 | **Advisor Framework scope is enforced at runtime by the Kernel**; the **Decision Journal** exposes recommendation + rationale + data reference (**not** full chain-of-thought). |
| 7 | **Document Verification:** Foundation owns the verification logic + hash check; **Term 7** owns the external surface + offline/cached verification. |
| 8 | **Compliance DSL:** grammar **and** sandboxed evaluator both in Foundation; the escape hatch is a **reviewed, sandboxed extension mechanism** (guarded against overuse). |
| 9 | **Resilience:** auto NORMAL→DEGRADED (audited, deterministic, replay-safe); human-gated for deeper transitions and all recovery. |
| 10 | **Snapshots:** non-truth **marker type** + CI doctrine check (runtime guard where feasible). |
| 11 | **Extension Points** include a **registration API concept** (manifest, capability declaration, compatibility check), not just a checklist. |
| 12 | **Document numbering:** deterministic, branch/terminal-scoped, **block-reserved** for DEGRADED mode. |
| 13 | **"Close period" is removed from the Ledger primitive** — period close is an Accounting (Term 5) concept; the Ledger knows only entries/balances. *(Neutrality, D-004.)* |

### Cross-term
Seeds **CTR-016** (→ Term 1, platform scope), **CTR-017** (→ Term 7, verification split), **CTR-018** (→ Term 6, registration API). Resolves **CTR-009**; partially resolves **CTR-007**; proposes resolution for **CTR-001/003** (pending Term 5/6).

### Affects
Term 4 (Section 3); Terms 1, 5, 6, 7 must accept the cross-term items before those parts are final.

---

## Decision Index

| ID | Title | Status | Primary Terms |
|----|-------|--------|---------------|
| D-001 | Universal Checkout | Accepted | 4, 5, 6, 3, 1, 7 |
| D-002 | AI Advisor Framework + Developer-AI | Accepted | 4, 5, 2, 1, 3, 7 |
| D-003 | Multiple Agents + Referral | Accepted | 1, 2, 3, 7 |
| D-004 | Concept-Writing Principles | Accepted | 4 (now); all (ongoing) |
| D-005 | AI Mode (per-role dashboard advisor) | Accepted | 1, 2, 3, 4, 5, 7 |
| D-006 | Collaboration Operating Model (`main`) | Accepted | all |
| D-007 | Foundation Scope Resolutions (Section 3) | Accepted | 4; 1, 5, 6, 7 |

*— End of Decision Log —*
