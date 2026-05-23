# Cross-Term Request Register

> **Owner:** Term 7 — Integration & Coherence (concept CN-7-006)
> **Status:** Living register — append new CTRs; never delete, only change status
> **Process:** See `CROSS-TERM-COLLABORATION.md` §4

This register tracks every Cross-Term Request. Below the summary table, each CTR is recorded in full using the template. The register is **seeded** with the requests that arise from decisions D-001, D-002, and D-003 — these are the first agreements the Terms must reach.

---

## Summary Table

| CTR | From | To | Decision | Topic | Status |
|-----|------|----|----------|-------|--------|
| CTR-001 | 6 | 4 | D-001 | Saleable Line / Tender primitive | OPEN |
| CTR-002 | 6 | 5 | D-001 | Verticals feed lines; tender owned by universal | OPEN |
| CTR-003 | 5 | 4 | D-001 | Checkout: primitive or universal engine? | OPEN |
| CTR-004 | 3 | 5, 6 | D-001 | One checkout UI contract + line shape | OPEN |
| CTR-005 | 1 | 5, 6 | D-001 | Checkout: always-on vs catalog item | OPEN |
| CTR-006 | 5 | 7 | D-001 | Payment adapters attach at tender boundary | OPEN |
| CTR-007 | 5, 2, 3 | 4 | D-002A | Advisor Framework contract + model identity | OPEN |
| CTR-008 | 4 | 1 | D-002B | Developer-AI governance + model registry | OPEN |
| CTR-009 | 3 | 4, 5 | D-002A | AI explainability fields surfaced to UI | OPEN |
| CTR-010 | 7 | all | D-002 | Advisory-only invariant holds everywhere | OPEN |
| CTR-011 | 2 | 1 | D-003 | Referral rules + commission attribution | OPEN |
| CTR-012 | 3 | 2 | D-003 | Tenant agent-discovery + referral touchpoint | OPEN |
| CTR-013 | 7 | 1, 2 | D-003 | Referral must not create a de-facto Global Agent | OPEN |

---

## Full Requests

### CTR-001 — Saleable Line / Tender primitive
- **From Term:** 6
- **To Term(s):** 4
- **Decision / Topic:** D-001 / Universal Checkout
- **Boundary Object:** BO-1
- **What is needed:** A Foundation-level `Saleable Line / Charge` shape that any vertical can produce, plus a `Tender` shape checkout can settle against.
- **Why:** Verticals must hand payable lines to a shared checkout without checkout knowing the vertical.
- **Proposed contract:** A line carries {item ref, description, qty, unit price, tax treatment ref, source vertical tag (opaque to checkout), optional discount refs}. Tender carries {method, amount, external reference}.
- **Status:** OPEN
- **Resolution:** —

### CTR-002 — Verticals feed lines; tender owned by universal
- **From Term:** 6
- **To Term(s):** 5
- **Decision / Topic:** D-001 / Universal Checkout
- **Boundary Object:** BO-1
- **What is needed:** Agreement that verticals emit lines and the Universal Checkout/Tender engine owns payment, splits, change, and receipt.
- **Why:** Preserves engine isolation (Law 2) and lets future verticals reuse checkout.
- **Proposed contract:** Vertical emits `<vertical>.bill.ready.v1` carrying saleable lines; checkout consumes it; checkout emits `checkout.settled.v1`.
- **Status:** OPEN
- **Resolution:** —

### CTR-003 — Checkout: primitive or universal engine?
- **From Term:** 5
- **To Term(s):** 4
- **Decision / Topic:** D-001 / Universal Checkout
- **Boundary Object:** BO-1
- **What is needed:** Foundation ruling on whether Checkout is a primitive or a universal engine built from existing primitives (obligation, document, ledger).
- **Why:** Determines where checkout logic lives and how it stays isolated.
- **Proposed contract:** Checkout is a **universal engine** in Term 5, built on Foundation's obligation + document + ledger primitives; Foundation adds only the `Saleable Line` + `Tender` value shapes if existing primitives are insufficient.
- **Status:** OPEN
- **Resolution:** —

### CTR-004 — One checkout UI contract + line shape
- **From Term:** 3
- **To Term(s):** 5, 6
- **Decision / Topic:** D-001 / Universal Checkout
- **Boundary Object:** BO-1
- **What is needed:** A stable contract for what a saleable line and a settled receipt look like, so one checkout screen serves all verticals.
- **Why:** A single cashier flow across Retail/Restaurant/Hotel/Workshop reduces training and error.
- **Proposed contract:** Term 5 publishes the line + receipt read-model; Term 3 designs one screen against it; verticals add only a label/context strip.
- **Status:** OPEN
- **Resolution:** —

### CTR-005 — Checkout: always-on vs catalog item
- **From Term:** 1
- **To Term(s):** 5, 6
- **Decision / Topic:** D-001 / Universal Checkout
- **Boundary Object:** BO-1
- **What is needed:** Decision on whether checkout is bundled with every tenant or sold as a catalog item in combos.
- **Why:** Affects pricing and what a minimal subscription includes.
- **Proposed contract:** Checkout is an always-on universal capability (every business takes payment); pricing differentiates by vertical engines, not by checkout.
- **Status:** OPEN
- **Resolution:** —

### CTR-006 — Payment adapters attach at tender boundary
- **From Term:** 5
- **To Term(s):** 7
- **Decision / Topic:** D-001 / Universal Checkout
- **Boundary Object:** BO-1
- **What is needed:** Confirmation that mobile-money/card adapters attach at the tender boundary, not inside any vertical.
- **Why:** Keeps integrations vertical-agnostic; one M-Pesa adapter serves all verticals.
- **Proposed contract:** Checkout emits a tender request; Term 7 adapter fulfils it and returns an external reference; checkout records the tender.
- **Status:** OPEN
- **Resolution:** —

### CTR-007 — Advisor Framework contract + model identity
- **From Term:** 5, 2, 3
- **To Term(s):** 4
- **Decision / Topic:** D-002A / Runtime Advisor Framework
- **Boundary Object:** BO-2
- **What is needed:** A Foundation Advisor Framework defining audience, data scope, model plug-in, and Decision Journal fields including model identity/version.
- **Why:** Engine advisors, BI/KPI advisor, agent advisor, and tenant-facing advice all need one consistent, advisory-only contract.
- **Proposed contract:** Advisor = {audience, scope, model ref}; every suggestion journals {model id+version, data ref, prompt ref, human decision}.
- **Status:** OPEN
- **Resolution:** —

### CTR-008 — Developer-AI governance + model registry
- **From Term:** 4
- **To Term(s):** 1
- **Decision / Topic:** D-002B / Developer-AI (build-time)
- **Boundary Object:** BO-3
- **What is needed:** Platform governance for the out-of-kernel Developer-AI environment: who triggers it, who approves changes/releases, the model registry, and audit of build actions.
- **Why:** Build-time AI must never modify the runtime without human + test gates; governance lives with Platform Stewards.
- **Proposed contract:** Developer-AI runs outside the kernel; Claude Code implements via PR; release only after human review + CI (tests + doctrine invariants); Term 1 owns approvals and the model registry.
- **Status:** OPEN
- **Resolution:** —

### CTR-009 — AI explainability fields surfaced to UI
- **From Term:** 3
- **To Term(s):** 4, 5
- **Decision / Topic:** D-002A / Runtime Advisor Framework
- **Boundary Object:** BO-2
- **What is needed:** The Decision Journal must expose enough (recommendation + reasoning summary + data basis) for tenant-facing explainability, per role.
- **Why:** A tenant must be able to judge whether to trust an AI suggestion (Term 3 trust moments).
- **Proposed contract:** Journal entry exposes a human-readable recommendation, a short rationale, and a "show your work" data reference; full chain-of-thought is not required.
- **Status:** OPEN
- **Resolution:** —

### CTR-010 — Advisory-only invariant holds everywhere
- **From Term:** 7
- **To Term(s):** all
- **Decision / Topic:** D-002 / AI
- **Boundary Object:** BO-2, BO-3
- **What is needed:** Confirmation from every Term that no AI feature commits state or acts autonomously, including in the integration pipeline.
- **Why:** Law 3 is a legal requirement; it must hold in runtime advisors, integrations, and the build pipeline alike.
- **Proposed contract:** Each Term lists its AI touchpoints and confirms each is advisory/human-gated; Term 7 maintains the list.
- **Status:** OPEN
- **Resolution:** —

### CTR-011 — Referral rules + commission attribution
- **From Term:** 2
- **To Term(s):** 1
- **Decision / Topic:** D-003 / Multiple Agents + Referral
- **Boundary Object:** BO-4
- **What is needed:** Platform policy for multiple agents per region, who may refer, and how commission is attributed when a referral is involved.
- **Why:** Agents need certainty about earnings; platform must prevent disputes.
- **Proposed contract:** Referral attributes acquisition credit but the **responsible regional agent** (compliance-accountable) is always recorded separately; commission split policy set by Term 1.
- **Status:** OPEN
- **Resolution:** —

### CTR-012 — Tenant agent-discovery + referral touchpoint
- **From Term:** 3
- **To Term(s):** 2
- **Decision / Topic:** D-003 / Multiple Agents + Referral
- **Boundary Object:** BO-4
- **What is needed:** A tenant-facing way to discover local agents and/or enter a referral, without bypassing the agent-led onboarding rule.
- **Why:** Tenants can't self-register, but multi-agent + referral implies a discovery/contact surface.
- **Proposed contract:** Tenant sees local agents / enters a referral code → request routes to an agent → agent completes compliant onboarding.
- **Status:** OPEN
- **Resolution:** —

### CTR-013 — Referral must not create a de-facto Global Agent
- **From Term:** 7
- **To Term(s):** 1, 2
- **Decision / Topic:** D-003 / Multiple Agents + Referral
- **Boundary Object:** BO-4
- **What is needed:** Coherence confirmation that referral never lets one party act across regions or escape regional compliance accountability.
- **Why:** Charter Law 6 — distribution is regional because compliance is regional.
- **Proposed contract:** A referrer earns referral reward only; the compliance-accountable agent is always region-local; no cross-region agency is created by referral.
- **Status:** OPEN
- **Resolution:** —

*— End of Cross-Term Request Register —*
