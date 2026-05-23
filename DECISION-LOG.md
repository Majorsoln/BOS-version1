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

## Decision Index

| ID | Title | Status | Primary Terms |
|----|-------|--------|---------------|
| D-001 | Universal Checkout | Accepted | 4, 5, 6, 3, 1, 7 |
| D-002 | AI Advisor Framework + Developer-AI | Accepted | 4, 5, 2, 1, 3, 7 |
| D-003 | Multiple Agents + Referral | Accepted | 1, 2, 3, 7 |

*— End of Decision Log —*
