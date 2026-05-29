# Cross-Term Request Register

> **Owner:** Term 7 — Integration & Coherence (concept CN-7-006)
> **Status:** Living register — append new CTRs; never delete, only change status
> **Process:** See `CROSS-TERM-COLLABORATION.md` §4

This register tracks every Cross-Term Request. Below the summary table, each CTR is recorded in full using the template. The register is **seeded** with the requests that arise from decisions D-001, D-002, and D-003 — these are the first agreements the Terms must reach.

---

## Summary Table

| CTR | From | To | Decision | Topic | Status |
|-----|------|----|----------|-------|--------|
| CTR-001 | 6 | 4 | D-001 | Saleable Line / Tender primitive | NEGOTIATING |
| CTR-002 | 6 | 5 | D-001 | Verticals feed lines; tender owned by universal | OPEN |
| CTR-003 | 5 | 4 | D-001 | Checkout: primitive or universal engine? | NEGOTIATING |
| CTR-004 | 3 | 5, 6 | D-001 | One checkout UI contract + line shape | OPEN |
| CTR-005 | 1 | 5, 6 | D-001 | Checkout: always-on vs catalog item | OPEN |
| CTR-006 | 5 | 7 | D-001 | Payment adapters attach at tender boundary | OPEN |
| CTR-007 | 5, 2, 3 | 4 | D-002A | Advisor Framework contract + model identity | NEGOTIATING |
| CTR-008 | 4 | 1 | D-002B | Developer-AI governance + model registry | OPEN |
| CTR-009 | 3 | 4, 5 | D-002A | AI explainability fields surfaced to UI | CLOSED |
| CTR-010 | 7 | all | D-002 | Advisory-only invariant holds everywhere | OPEN |
| CTR-011 | 2 | 1 | D-003 | Referral rules + commission attribution | OPEN |
| CTR-012 | 3 | 2 | D-003 | Tenant agent-discovery + referral touchpoint | OPEN |
| CTR-013 | 7 | 1, 2 | D-003 | Referral must not create a de-facto Global Agent | OPEN |
| CTR-014 | 1 | 4, 7 | D-005 | AI Mode cost governance + model approval | OPEN |
| CTR-015 | 7 | 1, 2, 3 | D-005 | Adopt one AI Mode dashboard pattern (CN-7-004) | OPEN |
| CTR-016 | 4 | 1 | D-007 | Platform scope as a first-class parallel scope | OPEN |
| CTR-017 | 4 | 7 | D-007 | Document verification: Foundation logic / Term 7 surface | NEGOTIATING |
| CTR-018 | 4 | 6 | D-007 | Extension Points include a registration API concept | NEGOTIATING |
| CTR-019 | 3 | 7 | D-008 | Messaging channel contract (tenant↔customer) | OPEN |
| CTR-020 | 2 | 7 | D-008 | Agent channel contract (acquisition/support) | OPEN |
| CTR-021 | 5 | 4, 7 | D-008 | Outreach/promotions over channels + consent | OPEN |
| CTR-022 | 4 | 1 | D-007 | Registered-engine activation + tenant-availability governance | OPEN |
| CTR-023 | 5 | 4 | — | Per-operation scope_ref in manifest (additive CN-4-005 amendment) | OPEN |
| CTR-024 | 5 | 6 | — | Verticals carry `site_id` in payload of site-scope events | OPEN |

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
- **Status:** NEGOTIATING
- **Resolution (delivered, CN-4-021):** Saleable Line is a **value shape** in Foundation carrying {line_id, item_ref, description, quantity, unit_price (pre-tax), tax_treatment_ref (input ref — tax computed at settlement), discount_refs, source_tag (opaque, non-branching), line_total (non-authoritative cache)}. Tender carries {tender_id, method (registered tag), amount, external_ref} — no status (lifecycle-free). Pending Term 6 acceptance.

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
- **Status:** NEGOTIATING
- **Resolution (delivered, CN-4-021):** Checkout = universal engine in Term 5; Foundation provides the value shapes only (Saleable Line + Tender). Settlement logic, tax computation at settlement, change, receipt issuance, and credit→Obligation creation all live in the Term 5 Checkout engine. Pending Term 5 acceptance.

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
- **Status:** NEGOTIATING
- **Resolution (delivered, CN-4-022):** Advisor = {audience (3 Charter §3 categories), scope (tenant / aggregate-opt-in / platform), model (id/version/config_ref)}; **runtime scope enforcement by the Kernel** covering all advisor input incl. prompt assembly; consent-scopes re-evaluated every invocation; advisory-only (reads projections, may produce inert draft commands); **dual-actor audit** (advisor + invoking human); journal fields contributed to CN-4-013 (incl. prompt_ref, model id/version; no full chain-of-thought). Pending Terms 5/2/3 confirmation.

### CTR-008 — Developer-AI governance + model registry
- **From Term:** 4
- **To Term(s):** 1
- **Decision / Topic:** D-002B / Developer-AI (build-time)
- **Boundary Object:** BO-3
- **What is needed:** Platform governance for the out-of-kernel Developer-AI environment: who triggers it, who approves changes/releases, the model registry, and audit of build actions.
- **Why:** Build-time AI must never modify the runtime without human + test gates; governance lives with Platform Stewards.
- **Proposed contract:** Developer-AI runs outside the kernel; Claude Code implements via PR; release only after human review + CI (tests + doctrine invariants); Term 1 owns approvals and the model registry.
- **Status:** OPEN
- **Resolution:** Foundation side delivered (CN-4-023): the out-of-kernel boundary (Developer-AI never runs in/writes to the kernel; no special treatment for AI PRs; synthetic/anonymised test data) and the doctrine gate (no-override, fail-closed, heightened doctrine-check-update process, build-provenance attestation). Awaiting Term 1 to govern trigger authority, approval authority, the model registry, build-action audit, and attestation retention.

### CTR-009 — AI explainability fields surfaced to UI
- **From Term:** 3
- **To Term(s):** 4, 5
- **Decision / Topic:** D-002A / Runtime Advisor Framework
- **Boundary Object:** BO-2
- **What is needed:** The Decision Journal must expose enough (recommendation + reasoning summary + data basis) for tenant-facing explainability, per role.
- **Why:** A tenant must be able to judge whether to trust an AI suggestion (Term 3 trust moments).
- **Proposed contract:** Journal entry exposes a human-readable recommendation, a short rationale, and a "show your work" data reference; full chain-of-thought is not required.
- **Status:** CLOSED
- **Resolution (delivered, CN-4-013):** Decision Journal full 14-field schema delivered — recommendation, short rationale, point-in-time `data_ref` ("show your work"), **no full chain-of-thought** (D-007 #6). The journal is an event-sourced specialised audit log (extends CN-4-008); Term 3 reads these fields for explainability UX. Contract delivered and accepted — closed.

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

### CTR-014 — AI Mode cost governance + model approval
- **From Term:** 1
- **To Term(s):** 4, 7
- **Decision / Topic:** D-005 / AI Mode
- **Boundary Object:** BO-2
- **What is needed:** Platform governance for AI Mode running on every dashboard: per-tenant/role cost controls, model approval, and the model registry.
- **Why:** AI Mode on all dashboards means many model calls; cost and model choice must be governed without changing engine code.
- **Proposed contract:** Term 1 owns AI cost policy + model registry; Term 4 exposes the model plug-in point; Term 7 confirms advisory-only across all surfaces.
- **Status:** OPEN
- **Resolution:** Foundation side delivered (CN-4-022): the model plug-in point ({model_id, model_version, model_config_ref}, swappable without framework change). Awaiting Term 1 to define cost governance, model approval, and the model registry.

### CTR-015 — Adopt one AI Mode dashboard pattern
- **From Term:** 7
- **To Term(s):** 1, 2, 3
- **Decision / Topic:** D-005 / AI Mode
- **Boundary Object:** BO-2
- **What is needed:** All dashboards (platform, agent, tenant) implement the **same** AI Mode pattern, recorded in the Pattern Library (CN-7-004).
- **Why:** Consistency — owner, cashier, agent, and platform staff get one coherent AI Mode, differing only in scoped advice.
- **Proposed contract:** Term 7 publishes the AI Mode pattern (prompt + proactive advice, role-scoped, journaled, explainable); Terms 1/2/3 surface it per their dashboards.
- **Status:** OPEN
- **Resolution:** —

### CTR-016 — Platform scope as a first-class parallel scope
- **From Term:** 4
- **To Term(s):** 1
- **Decision / Topic:** D-007 / Identity & isolation
- **Boundary Object:** BO-5, BO-6
- **What is needed:** Confirmation that platform cross-tenant access (e.g., "all tenants in region X this month") is modelled as a **first-class platform scope** with its own audit trail — not an exception that weakens tenant isolation.
- **Why:** Platform governance (Term 1) needs cross-tenant visibility without breaching Law of isolation.
- **Proposed contract:** Foundation provides a platform scope parallel to tenant scope; every platform-scope access is audited; Term 1 defines which platform roles may use it.
- **Status:** OPEN
- **Resolution:** —
- **Note (CN-5-101):** Term 5 keeps universal engines OFF platform scope — Reporting emits per-tenant metrics (tenant-scope); **billing/usage aggregation across tenants is Term 1's platform-scope concern and folds into this CTR** (no separate CTR). Dual-trail audit (CN-4-006 §2) applies.

### CTR-017 — Document verification: Foundation logic / Term 7 surface
- **From Term:** 4
- **To Term(s):** 7
- **Decision / Topic:** D-007 / Document verification
- **Boundary Object:** —
- **What is needed:** Agreement that Foundation owns the verification **logic + hash check**, while Term 7 owns the **external-facing surface** (the public portal) and **offline/cached** verification.
- **Why:** Verification = hash-chain integrity (Foundation's core promise), but the public exposure is an integration concern.
- **Proposed contract:** Foundation exposes a verify(document, hash) capability; Term 7 builds the public/QR surface and offline cache around it.
- **Status:** NEGOTIATING
- **Resolution (Foundation side delivered, CN-4-012):** Foundation owns the verification **logic + hash check** — the document hash (over content + template/pack version refs + number) is stored inside the `document.issued.v1` event and is itself chain-protected (CN-4-003). **Open question raised for Term 7:** the hash proves **integrity** (content unchanged), not **authenticity** (that BOS issued it). Term 7 must decide whether the external/offline surface (portal, QR, offline cache) requires **integrity-only (hash check)** or **integrity + authenticity (hash + cryptographic signature)**. Pending Term 7 acceptance + signature decision.

### CTR-018 — Extension Points include a registration API concept
- **From Term:** 4
- **To Term(s):** 6
- **Decision / Topic:** D-007 / Extension Points
- **Boundary Object:** BO-5
- **What is needed:** Confirmation that the new-engine contract includes a **registration API concept** (engine manifest, capability declaration, compatibility check), not just a checklist — since Term 6 depends on it to add verticals.
- **Why:** Term 6 adds new verticals; a concrete registration contract makes that clean and isolated.
- **Proposed contract:** Foundation (CN-4-020) defines a manifest + capability declaration + compatibility check; Term 6 follows it to register each vertical without touching the Kernel.
- **Status:** NEGOTIATING
- **Resolution (proposed, CN-4-020):** Foundation delivered the concrete contract — build-time doctrine gate → runtime registration (manifest submission, compatibility checks, atomic wiring/activation), event-sourced registry, manifest-update flow, capability-as-advertise-only (no coupling), and platform-scoped audited registration. Pending Term 6 acceptance that this contract suffices to add verticals.

### CTR-019 — Messaging channel contract (tenant ↔ customer)
- **From Term:** 3
- **To Term(s):** 7
- **Decision / Topic:** D-008 / Messaging Channels
- **Boundary Object:** BO-7
- **What is needed:** An outbound/inbound channel contract so tenants can serve customers over SMS, WhatsApp, Telegram, email — receipts, order updates, notifications, and support replies.
- **Why:** Tenant ↔ customer service over messaging is core to acquisition and retention.
- **Proposed contract:** Term 7 exposes a channel-agnostic send/receive capability with delivery status and an `external_reference`; Term 3 designs the tenant UX on top. AI replies are advisory/human-gated.
- **Status:** OPEN
- **Resolution:** —

### CTR-020 — Agent channel contract (acquisition / support)
- **From Term:** 2
- **To Term(s):** 7
- **Decision / Topic:** D-008 / Messaging Channels
- **Boundary Object:** BO-7
- **What is needed:** A channel contract for agents to reach and support tenants, and to help tenants enable their own channels.
- **Why:** Agents acquire and provide L1 support, increasingly over messaging apps.
- **Proposed contract:** Same Term 7 gateway as CTR-019; agent actions over channels are tagged with the agent's ID and audited.
- **Status:** OPEN
- **Resolution:** —

### CTR-021 — Outreach / promotions over channels + consent
- **From Term:** 5
- **To Term(s):** 4, 7
- **Decision / Topic:** D-008 / Messaging Channels
- **Boundary Object:** BO-7
- **What is needed:** A way to run customer acquisition/outreach and promotions over channels, gated by recorded consent and opt-out.
- **Why:** Marketing over messaging must respect consent (Foundation primitive) and anti-spam/regional law.
- **Proposed contract:** Promotion Engine (Term 5) composes campaigns; Foundation's consent primitive (Term 4) gates sends; Term 7 delivers. No send without consent; every send honours opt-out.
- **Status:** OPEN
- **Resolution:** —

### CTR-022 — Registered-engine activation & tenant-availability governance
- **From Term:** 4
- **To Term(s):** 1
- **Decision / Topic:** D-007 / Extension Points (CN-4-020)
- **Boundary Object:** —
- **What is needed:** Platform governance for: (a) whether a newly registered engine activates automatically after checks pass or requires platform-admin approval; (b) how a registered engine is made available to tenants (always-on vs catalog item per subscription plan).
- **Why:** Foundation automates registration (the mechanism); Term 1 owns the engine catalog and what tenants see and pay for. The two must not be conflated.
- **Proposed contract:** Foundation registers and activates engines technically; Term 1 controls tenant-facing availability and may add a human-approval gate before activation.
- **Status:** OPEN
- **Resolution:** —

### CTR-023 — Per-operation scope_ref in manifest (additive CN-4-005 amendment)
- **From Term:** 5
- **To Term(s):** 4
- **Decision / Topic:** — / arose from CN-5-101 (Scope Policy Application)
- **Boundary Object:** BO-5 (manifest)
- **What is needed:** An additive amendment to CN-4-005's manifest: `scope_policy` becomes the engine **default**; each command and each subscription entry may declare its own `scope_ref` (`site` | `tenant` | `platform`). This lets one engine host operations at different scopes (e.g., Cash: drawer = site, consolidated position = tenant). Also aligns scope-level names to `site/tenant/platform` (supersedes the informal "branch/business" wording in CN-4-005/CN-4-011).
- **Why:** Multi-scope universal engines (Cash, Reporting) cannot work under a single engine-wide scope. CN-5-100 already used per-subscription `scope_ref`; this formalises it and adds per-command scope.
- **Proposed contract:** Additive only — existing single `scope_policy` remains valid as the default; per-operation `scope_ref` is optional and overrides the default where present. No breaking change.
- **Status:** OPEN
- **Resolution:** —

### CTR-024 — Verticals carry `site_id` in payload of site-scope events
- **From Term:** 5
- **To Term(s):** 6
- **Decision / Topic:** — / arose from CN-5-101 (Scope Policy Application)
- **Boundary Object:** BO-1
- **What is needed:** Vertical engines that emit events consumed by site-scoped universal-engine subscriptions must include a neutral `site_id` (intra-tenant location/site partition) in the event payload, so site-scope handler dispatch (CN-5-101) resolves correctly.
- **Why:** `site_id` is a Term 5 payload concept, not a Foundation envelope field (CN-4-002 has tenant_id only). Universal engines need it from the vertical's emission to scope per site.
- **Proposed contract:** Site-scope events carry `site_id` in payload; tenant-wide events need not. Naming is neutral (`site_id`, not retail-flavoured "branch").
- **Status:** OPEN
- **Resolution:** —

*— End of Cross-Term Request Register —*
