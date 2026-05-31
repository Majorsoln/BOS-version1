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
| CTR-003 | 5 | 4 | D-001 | Checkout: primitive or universal engine? | CLOSED |
| CTR-004 | 3 | 5, 6 | D-001 | One checkout UI contract + line shape | OPEN |
| CTR-005 | 1 | 5, 6 | D-001 | Checkout: always-on vs catalog item | NEGOTIATING |
| CTR-006 | 5 | 7 | D-001 | Payment adapters attach at tender boundary | NEGOTIATING |
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
| CTR-023 | 5 | 4 | — | Per-operation scope_ref in manifest (additive CN-4-005 amendment) | CLOSED |
| CTR-024 | 5 | 6 | — | Verticals carry `site_id` in payload of site-scope events | OPEN |
| CTR-025 | 5 | 4 | — | Add DC-038..041 (Phase 0 mechanizable UI invariants) to CN-4-019 living catalog | OPEN |
| CTR-026 | 5 | 6 | — | Verticals respect UI-03 (inventory source ref) and UI-09 (site_id in registry) | OPEN |
| CTR-027 | 5 | 1, 2 | — | Tenant site registry (T2 onboarding; T1 governance) + tenant functional currency registry (T1) | OPEN |
| CTR-028 | 5 | 7, 1 | D-001 | Tender method registry — category vocabulary + provider declarations + governance | OPEN |
| CTR-029 | 5 | 1 | — | Accounting-standard pack-section content + governance per jurisdiction (IFRS, TFRS-TZ, GAAP, …) | OPEN |
| CTR-030 | 5 | 6 | — | Vertical event payloads carry data sufficient for Accounting journal mapping | OPEN |
| CTR-031 | 5 | 7 | D-001 | Banking adapter contract — deposit handoff + statement reconciliation | OPEN |

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
- **Resolution (delivered, CN-4-021 + CN-5-009):** Saleable Line is a **value shape** in Foundation carrying {line_id, item_ref, description, quantity, unit_price (pre-tax), tax_treatment_ref (input ref — tax computed at settlement), discount_refs, source_tag (opaque, non-branching), line_total (non-authoritative cache)}. Tender carries {tender_id, method (registered tag), amount, external_ref} — no status (lifecycle-free). CN-5-009 (Universal Checkout/Tender Engine) consumes these shapes successfully, demonstrating downstream usability. Pending Term 6 acceptance.

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
- **Status:** CLOSED
- **Resolution (delivered + accepted, CN-4-021 + CN-5-009):** Checkout = universal engine in Term 5 (CN-5-009); Foundation provides the value shapes only (Saleable Line + Tender). CN-5-009 demonstrates the engine: settlement workflow Phase 1–4, tax computation at settlement (D-009 pack-frozen), atomic finalisation (CN-4-004 §2), credit-tender→Obligation, receipt issuance via CN-4-012. Both sides agreed (Foundation delivered; Term 5 accepted and built upon).

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
- **Status:** NEGOTIATING
- **Resolution (Term 5 side delivered, CN-5-009):** Term 5 commits to Checkout-as-always-on for every tenant in CN-5-009 (every business takes payment; bundling as a catalog item adds barrier). Pending Term 1 acceptance and Term 6 alignment.

### CTR-006 — Payment adapters attach at tender boundary
- **From Term:** 5
- **To Term(s):** 7
- **Decision / Topic:** D-001 / Universal Checkout
- **Boundary Object:** BO-1
- **What is needed:** Confirmation that mobile-money/card adapters attach at the tender boundary, not inside any vertical.
- **Why:** Keeps integrations vertical-agnostic; one M-Pesa adapter serves all verticals.
- **Proposed contract:** Checkout emits a tender request; Term 7 adapter fulfils it and returns an external reference; checkout records the tender.
- **Status:** NEGOTIATING
- **Resolution (Term 5 side delivered, CN-5-009 §6):** Concrete request/reply contract via events with `callback_correlation_id`: Checkout emits `checkout.tender.requested.v1 {tender_id, method, amount, external_ref?, callback_correlation_id}`; Term 7 adapter consumes, invokes external system, emits `<adapter>.tender.outcome.v1 {tender_id, outcome: confirmed|failed, external_ref, reason?}`; Checkout's subscription handler submits `checkout.tender.confirm.request` or `checkout.tender.fail.request`. Pending Term 7 acceptance.

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
- **Status:** CLOSED
- **Resolution (delivered, CN-4-005 amendment commit `76dd67d`):** CN-4-005 amended additively — header carries CTR-023 reference; `scope_policy` becomes the engine **default** with levels `site` / `tenant` / `platform`; `subscribes_to` and `commands` entries may each carry an optional `scope_ref` override. New **§1.1 "Scope Levels and Per-Operation Override"** added (levels table, default+override mechanism, "additive" guarantee, provenance from CN-5-100/CN-5-101). `site` is a payload concept; CN-4-002 envelope unchanged. Closed by both sides (Term 5 requested → Term 4 delivered).

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

### CTR-025 — Add DC-038..041 (Phase 0 mechanizable UI invariants) to CN-4-019 living catalog
- **From Term:** 5
- **To Term(s):** 4
- **Decision / Topic:** — / arose from CN-5-102 (Engine Invariants) Phase 0 ramp-up
- **Boundary Object:** —
- **What is needed:** Add four new doctrine checks to CN-4-019's living catalog (Charter §11 + CN-4-019 §5 living-catalog rules): **DC-038** UI-01 causation_id integrity on derivative events; **DC-039** UI-03 inventory movement source-ref present; **DC-040** UI-06 functional-currency invariant per tenant (extends DC-037 from per-settlement to tenant scope); **DC-041** UI-09 site_id registry lookup. These are mechanizable now; Phase 1 invariants depend on per-engine docs and will arrive via later CTRs.
- **Why:** CN-5-102 catalog includes invariants that map to concrete doctrine checks; adding them to CN-4-019 now operationalises the cross-engine guarantees in CI/runtime rather than as prose-only.
- **Proposed contract:** Term 4 (re-activated kifupi per Concept Lead authorisation, like CN-4-005 amendment) extends CN-4-019 §2 with DC-038..041 in the appropriate categories; updates Type table (§3); updates Law mapping (§4) where relevant; increments doc total from 37 to 41.
- **Status:** OPEN
- **Resolution:** —

### CTR-026 — Verticals respect UI-03 (inventory source ref) and UI-09 (site_id in registry)
- **From Term:** 5
- **To Term(s):** 6
- **Decision / Topic:** — / arose from CN-5-102 (Engine Invariants)
- **Boundary Object:** BO-1
- **What is needed:** Vertical engines that emit events feeding universal-engine inventory or site-scope subscriptions must (a) include a typed **source ref** on every inventory-affecting event payload (UI-03), and (b) use a `site_id` that exists in the tenant's site registry (UI-09).
- **Why:** Universal-engine invariants (UI-03 stock movements have source; UI-09 site_id in registry) cannot hold if the upstream vertical emissions are loose. The doctrine has to be respected at the emission boundary.
- **Proposed contract:** Term 6 confirms its vertical engines (current and future) emit conformant payloads; UI-03/UI-09 enforcement happens at the universal-engine subscription dispatch (hard-fail) per CN-5-101 §5. Pairs with CTR-024 (site_id in payload).
- **Status:** OPEN
- **Resolution:** —

### CTR-027 — Tenant site registry + tenant functional currency registry
- **From Term:** 5
- **To Term(s):** 1, 2
- **Decision / Topic:** — / arose from CN-5-102 (Engine Invariants — UI-06 currency; UI-09 site_id)
- **Boundary Object:** —
- **What is needed:** Two tenant-property registries, governed under platform policy and populated at tenant onboarding:
  - **Site registry** — the canonical list of `site_id` values for a tenant (set by the regional agent at onboarding per Law 6, like fiscal zone in CN-4-014; governance for additions/changes by Term 1).
  - **Functional currency registry** — the tenant's declared functional currency (a tenant property, like fiscal zone); governance by Term 1.
- **Why:** UI-09 needs an authoritative list of valid `site_id`s to validate against. UI-06 needs an authoritative functional currency per tenant. Both are tenant properties, not engine state; Term 1 governs, Term 2 populates at onboarding.
- **Proposed contract:** Term 1 defines the governance lifecycle (who may register/change a site; who may change functional currency — likely with strict approval given freeze implications similar to fiscal zone). Term 2 wires registry population into the regional-agent onboarding flow. Term 5 universal engines look up these registries at dispatch (UI-09) or at command evaluation (UI-06). Not folded into CTR-016 — those are platform-scope mechanism; these are tenant-property registries.
- **Status:** OPEN
- **Resolution:** —

### CTR-028 — Tender method registry: vocabulary + adapters + governance
- **From Term:** 5
- **To Term(s):** 7, 1
- **Decision / Topic:** D-001 / Universal Checkout · arose from CN-5-009 (Universal Checkout/Tender Engine)
- **Boundary Object:** BO-1 (Saleable Line / Tender / Checkout)
- **What is needed:** Per CN-4-021 §3 D, tender `method` is a **registered category tag** validated against a method registry. CN-5-009 needs the registry defined: who owns the category vocabulary, who declares providers per category, and who governs additions.
- **Why:** Without an authoritative registry, Checkout cannot validate `method` at command time (would degrade to free-form or hardcoded enum, both rejected by CN-4-021 D). The bus must reject unknown methods; that requires a queryable registry.
- **Proposed contract:**
  - **Category vocabulary** (`cash`, `mobile_money`, `card`, `bank_transfer`, `credit`, `voucher`, …) governed by **Term 7** — payment categories are integration concerns; Term 7 owns the integration gateway.
  - **Provider per category** declared by **Term 7** adapters (e.g., `mobile_money: M-Pesa`, `mobile_money: Tigo Pesa`).
  - **Approval of new categories or providers** → **Term 1** (governance + audit + compliance review).
  - **Checkout reads** registry at command evaluation; bus rejects unknown methods.
- **Status:** OPEN
- **Resolution:** —

### CTR-029 — Accounting-standard pack-section content + governance per jurisdiction
- **From Term:** 5
- **To Term(s):** 1
- **Decision / Topic:** — / arose from CN-5-001 (Accounting Engine); operationalises Law 5 + Charter §1.4 (local by law)
- **Boundary Object:** —
- **What is needed:** CN-5-001 defines the **schema** of a new compliance-pack section (`accounting:` — chart-of-accounts base, recognition rules, reporting templates, depreciation, FX, period_calendar) per jurisdiction's accounting standard (IFRS, TFRS-TZ, US GAAP, …). Term 1 governs **content authorship and approval** per jurisdiction (pack approval gate, CN-4-015 + CN-4-023 two-gates model) and the chartered-accountant review process for pack content.
- **Why:** Each country's accounting standard differs (recognition rules, reporting formats, depreciation methods). Standards live in packs (Law 5, D-009 freeze); CN-5-001 owns the schema, Term 1 owns the content lifecycle and country-by-country approval.
- **Proposed contract:** Term 1 stands up a chartered-accountant review pipeline per supported jurisdiction; each accepted standard becomes a versioned pack (`tz-compliance-YYYY.MM` carrying TFRS-TZ, etc.). Standard updates (e.g., IFRS-15 amendment) follow normal pack-version freeze (D-009). Tenant↔jurisdiction binding (CTR-027) determines which pack — hence which standard — a tenant runs under.
- **Status:** OPEN
- **Resolution:** —
- **Expansion note (CN-5-003):** The pack section is **extended** with an `inventory_costing` subsection (allowed_methods, default_method, category_defaults, tenant_override_allowed, per_item_override_allowed, offcut_cost_allocation). Standard-dependent: IFRS packs exclude LIFO (IAS 2, 2003 revision); US GAAP packs may include it. Term 1 governance covers per-jurisdiction approval of the inventory-costing content alongside accounting-standard content.
- **Expansion note (CN-5-002):** The pack section is **further extended** with: (a) `operating_expense_categories` (14 baseline: rent, utilities, fuel, transport, repairs, supplies, communications, staff_welfare, professional_fees, financial_charges, insurance, marketing_advertising, bank_charges, other — pack may extend per jurisdiction); (b) `cash_reconciliation.variance_thresholds` per till kind (auto-adjust + anomaly bands; tenant override ±50%); (c) `multi_currency_policy` (allowed currencies, FX rate sourcing rules); (d) `tip_tax_routing` (taxable wages vs pass-through per jurisdiction).
- **Expansion note (CN-5-004):** The pack section is **also extended** with: (a) `procurement_approvals.thresholds` (amount-band approvers; multi-approver logic; tenant override ±50%); (b) `three_way_match.variance_thresholds` per dimension (quantity/price/item — auto_pass/flag/hard_block bands); (c) `supplier_credit_term_defaults` per jurisdiction (Net 30, 2/10 Net 30, COD); (d) `procurement_forex_recognition` (at-record vs at-payment per pack rule).

### CTR-030 — Vertical event payloads carry data sufficient for Accounting journal mapping
- **From Term:** 5
- **To Term(s):** 6
- **Decision / Topic:** — / arose from CN-5-001 (Accounting Engine)
- **Boundary Object:** BO-1 (lines/tenders) / BO-5 (event contracts)
- **What is needed:** Vertical engines (Retail, Restaurant, Hotel, Workshop, future verticals) must include sufficient typed payload on revenue/expense events for Accounting to derive (a) recognition timing, (b) correct chart-of-accounts mapping per pack, (c) COGS basis, (d) tax treatment ref, (e) site_id (CTR-024 ties in). Insufficient payload = Accounting cannot generate a complete journal entry = UI-07 trial balance integrity at risk.
- **Why:** A1 (standards in packs) and A2 (auto-journal completeness) only hold if upstream emissions carry the data Accounting needs for the active pack's recognition rules. The contract must be defined at the vertical-emission boundary.
- **Proposed contract:** Each Term 6 vertical event subscribed by Accounting (per CN-5-001 catalog) declares a payload contract sufficient for journal mapping under any supported standard pack. CN-5-001 publishes the field expectations per subscribed event type; Term 6 verticals conform.
- **Status:** OPEN
- **Resolution:** —
- **Expansion note (CN-5-003):** Pattern B (vertical-emitted consumption) requires concrete event contracts from each opting-in vertical — `restaurant.ingredient.consumed.v1`, `hotel.amenity.consumed.v1`, `workshop.cut.executed.v1` (with offcut spec), `workshop.parametric.consumed.v1`. Payload shapes specified in CN-5-003 §5/§10/§11 (consumed_items list with item_ref + qty + optional substituted_for/reason; for cuts, parent + cuts[] + offcuts[]). Verticals must also include `expansion_mode: auto | vertical_managed` per-line on `<vertical>.bill.ready.v1` so Inventory's `deduct_from_sale` knows whether to skip Pattern A and wait for Pattern B emissions. Pairs with CTR-024 (site_id payload) and CTR-026 (UI-03 source-ref / UI-09 site_id).
- **Expansion note (CN-5-002):** Verticals that support layby / advance-deposit business models must emit fulfillment events Cash subscribes to (e.g., `workshop.advance_consumed.v1` per fulfillment milestone; `<vertical>.layby_release.v1` on full payment). These drive the reduction of the corresponding Obligation (`kind: advance_received` / `layby`). Payload includes obligation_ref + reduction amount + business_date.
- **Expansion note (CN-5-004):** Verticals may emit `<vertical>.purchase_need.recorded.v1` events that Procurement subscribes to, automatically initiating requisitions from vertical-driven purchase signals (e.g., a workshop accepting a job that needs materials not currently in stock). Payload includes item_ref(s), qty, target_delivery_date, justification, originating_workflow_ref. Procurement validates and may create a requisition from the need (pack and tenant rules govern auto-creation vs manual).

### CTR-031 — Banking adapter contract — deposit handoff + statement reconciliation
- **From Term:** 5
- **To Term(s):** 7
- **Decision / Topic:** D-001 / arose from CN-5-002 (Cash Management Engine)
- **Boundary Object:** —
- **What is needed:** A Term 7 banking adapter pattern analogous to but distinct from CTR-006 payment adapters: Cash emits `cash.deposit.requested.v1 {till_id: main_safe, amount, currency, callback_correlation_id, bank_account_ref, business_date}`; banking adapter consumes, invokes bank API / cash-in-transit service; emits `<bank-adapter>.deposit.outcome.v1 {callback_correlation_id, outcome: confirmed | failed, external_ref: <bank txn id>, reason?}`; Cash subscribes and finalises via `cash.deposit.complete.request`. Also covers periodic provider statement events for mobile money wallet digital reconciliation (`<mm-adapter>.statement.received.v1`) which Cash uses to reconcile mobile money tills against provider truth.
- **Why:** Banking has different semantics from payment-at-tender: deposit destination is an external bank account, settlement times differ, and statement reconciliation is required for both mobile money and (future) bank account tills. Overloading CTR-006 would conflate distinct contract surfaces.
- **Proposed contract:** Distinct from CTR-006 (payment adapters at tender boundary). Banking adapter handles outbound deposits and inbound statement events. Same request/reply pattern via events + callback_correlation_id; same adapter-emits-outcome-event mechanism. Term 7 builds the adapter; Term 1 governs which banking providers are approved per jurisdiction.
- **Status:** OPEN
- **Resolution:** —

*— End of Cross-Term Request Register —*
