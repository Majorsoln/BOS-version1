# Changelog

> Records every change to the Charter and to cross-Term decisions.
> Managed by Term 7 — Integration & Coherence (Charter §11).

## 2026-05-26

**Merged to `main` — Term 4 Phase 2 (2/3)**
- **Term 4 CN-4-022 (Advisor Framework & Role-Scope Binding)** ratified and merged (commit `1198e3e`). The single runtime Advisor Framework (D-002A, D-005). Contract = **audience** (three Charter §3 categories — Platform/Agent/Tenant, structural not roles) + **scope** (tenant / aggregate-opt-in / platform) + **model** (id/version/config_ref, swappable). **Runtime scope enforcement by the Kernel (D-007 #6)** covering **all advisor input including prompt assembly** (no out-of-band scope leakage; distinct from Term 7 prompt-injection defence). Consent-dependent scopes **re-evaluated every invocation** (revoke-wins, CN-4-011). Advisory-only (Law 3): reads projections only, replay-safe, **may produce inert draft commands** (no effect until a human submits via the command bus). **Dual-actor audit** — advisor principal + invoking human recorded together (CN-4-007). Journal fields **contributed to CN-4-013** (schema owner): incl. invoking_principal, prompt_ref, model_config_ref; **no full chain-of-thought** (CTR-009). Role→scope→advisor binding (Foundation = mechanism). Resolved N1 (cold-start data-sufficiency), N3 (model-swap, no re-consent), N4 (DEGRADED freshness / READ_ONLY block), q10 (benchmarking opt-in, anonymised aggregate). **CTR-007** resolution delivered (pending Terms 5/2/3); **CTR-014** Foundation side delivered (model plug-in point; awaiting Term 1 governance). No new CTRs; no Charter conflict.

**Merged to `main` — Term 4 Phase 2 begins (1/3)**
- **Term 4 CN-4-021 (Saleable Line & Tender Value Shapes)** ratified and merged (commit `5aad12d`) — first Phase 2 (CTR-driven) doc. The two value shapes (D-001, D-007 #1) connecting verticals to Universal Checkout. **Saleable Line:** authoritative inputs (quantity, unit_price pre-tax, discount_refs, tax_treatment_ref) vs **non-authoritative** derived `line_total` cache (Checkout must recompute, inputs win); **tax = input ref only**, computed at settlement by Checkout under the pack version active at emission (D-009, CN-4-015, Law 5 — verticals never compute tax); `source_tag` opaque + **non-branching guarantee** (doctrine-check enforceable, Law 2). **Tender:** {tender_id, method, amount, external_ref} — **no status field** (lifecycle-free; outcomes event-sourced by Checkout/Term 7); `method` = **registered category tag** (controlled vocabulary, extensible without Kernel change); **credit tender → Obligation primitive** (CN-4-011). **Currency invariant** (one settlement, one currency; cross-border = exception). Value-shapes≠primitives table; vertical→checkout flow; Karakana example with VAT computed at settlement. **CTR-001 & CTR-003** resolutions delivered (pending Terms 6/5 acceptance). No new CTRs; no Charter conflict.

**Merged to `main` — Term 4 Phase 1 COMPLETE (7/7 doctrine-foundation docs)**
- **Term 4 CN-4-020 (Extension Points)** ratified and merged (commit `0069608`) — **the final Phase 1 doc**. Two-phase onboarding: a **build-time doctrine gate** (CI per CN-4-019/CN-4-023; human review for Developer-AI pipeline, D-002B) followed by **runtime registration** (manifest submission → compatibility checks → **atomic** subscription wiring → activation, all-or-nothing rollback). Six compatibility checks; **event-type ownership transferable on sunset** (not permanently locked). **Registry is event-sourced** (`kernel.engine.registered/updated/deactivated.v1`; a CN-4-010 projection — replayable, auditable, deterministic; replaces a mutable registry store). **Manifest-update / re-registration** flow (version bump → re-check → additive wiring). Deregistration (4 steps, no event deletion; tenant notification = Term 1/3 hand-off). Resilience-mode gating (NORMAL allowed / DEGRADED human-gated / READ_ONLY blocked, D-007 #9). **Capability = advertise-only** (dependencies stay event-type-based; D-007 #11 without re-coupling). Registration is **platform-scoped + audited** (CN-4-006/007/008). **CTR-022 filed** (→ Term 1, activation & tenant-availability governance); **CTR-018 → NEGOTIATING** (contract delivered, pending Term 6 acceptance). **Phase 1 doctrine foundation is complete: CN-4-001/002/006/007/005/011/020.** Next: Phase 2 (CTR-driven) — CN-4-021, CN-4-022, CN-4-023.
- **Term 4 CN-4-011 (Primitive Catalog)** ratified and merged (commit `1054cb7`). The nine primitives — Ledger (no close-period, D-007 #13), Item/Service, **Party** (renamed from "actor" to the Charter §4.1 name; business entity, distinct from CN-4-007's system actor), Document (**terminal fold** — `issued` once; corrections = new linked documents per D-009), Inventory Movement, Obligation, Approval, Workflow, Consent (purpose+scope keying, revoke-wins, D-008) — each with shape, legal operations, and compensating-fold logic. Distinctiveness (Consent≠Approval by legal weight; Workflow≠Approval by complexity, D-007 #2); neutrality (D-004); engine-namespaced events (not primitive-namespaced); the Purchase Order composition example (§8). Four open items to Architect/Terms 5/6. **MASTER-GLOSSARY** gains **Party** and **Actor** entries (distinction recorded). No new CTRs; no Charter conflict.
- **Term 4 CN-4-005 (Engine Contract Model)** ratified and merged (commit `2d616aa`). The 10-field engine manifest (declarative; `requires` ⊆ `subscribes_to`; references event types, never engine names; deprecation marker within `emits`); Law 2 isolation enforced at both layers — runtime guards + CI doctrine checks (D-007 #5); subscription framework (emitter-unaware, event-type-based, new subscriber = manifest update only; at-least-once + ordered-within-stream + idempotent-expected, exactly-once = Architect); request-reply possible-but-discouraged (temporal coupling); the status-event + local-projection pattern (resolves O7/O8); pre-write malformed-event validation with hash-chain backstop and structural-vs-business-error distinction (O9/O10). Five open items assigned; CN-4-005/CN-4-020 boundary made explicit. No new CTRs; no Charter conflict.

## 2026-05-24

**Decision recorded**
- **D-009 — Temporal Interpretation / "Freeze" Doctrine.** Events and issued documents are interpreted under the rules/rates/pack-version active at emission/issuance; never retroactively reinterpreted. Implemented in CN-4-001/012/015; affects Terms 1/5/7. Arose from Term 4 Section 7 (edge cases O7, O15).
- **D-008 — Conversational / Messaging Channels.** A cross-term capability (not a new Term) for acquiring and serving customers over SMS, WhatsApp, Telegram, and email. Term 7 owns the channel adapters/gateway; Term 3 the tenant↔customer UX; Term 2 the agent use; Term 5 outreach/promotions; Term 4's consent primitive gates marketing. Guardrails: consent-first + opt-out, AI advisory/human-gated, channels via the Term 7 gateway. Boundary object BO-7; seeds CTR-019/020/021.

**Merged to `main`**
- **Term 4 CN-4-007 (Identity Primitive)** ratified and merged (commit `afccbba`). Four principal types (human, advisor, system, machine); authority chain (Kernel carries structure, not a permission engine); dual-actor record (resolves CN-4-006 R4); system-as-actor via the command bus (resolves Section 11 q4); advisor identity distinct from CN-4-022. Delegation flagged as a future decision (not a fifth type).
- **Term 4 CN-4-006 (Multi-Tenant Isolation)** ratified and merged (commit `0421bb4`). Seven-layer isolation + tenant scope object (non-bypassable); platform scope as a read-only parallel scope (dual-actor audit; policy = Term 1 via CTR-016); GDPR export scoping (cost = Architect); 7 leak-detection tests incl. snapshot isolation; data isolation = concept, resource isolation = Architect. Flags **right-to-erasure vs append-only** as a future decision (crypto-shredding direction).
- **Term 4 CN-4-002 (Event Store Contract)** ratified and merged (commit `50c2ce6`). The 15-field event envelope (engine-agnostic; pack_version_ref nullable for system events; record-time timestamp with business dates in payload; dedicated compensates_event_id; registered metadata), ordering (per-stream sequence + global_position + optimistic concurrency, resolving O1), additive-only schema versioning, and the append-only guarantee.
- **Term 4 CN-4-001 (Event Sourcing Doctrine)** ratified and merged (commit `587ea6c`) — the first CN concept document. Six doctrine assertions, the Three Whys with alternatives rejected, implications (compensating fold D-007 #3, freeze D-009, determinism), the Mzee Hassan/TRA example, and boundaries. Glossary terms **Compensating Event, Fold, Replay** added to `MASTER-GLOSSARY.md`.

**Decision recorded**
- **Term 4 Sections 12–14 intentionally omitted** (Concept Lead approved): §12 (Notes on Existing Repository) is reference-only; §13 (First Tasks) is superseded by Section 6's writing order; §14 (Restraint) is folded into Section 1 (Mission) and Section 4 (Scope). Term 4's brief-level proposal is therefore Sections 1–11. Next: the 23 CN concept documents, in the Section 6 order (Phase 1 → 2 → 3).

**Merged to `main`**
- **Term 4 Section 11 (Open Questions)** ratified and merged (commit `9090ed5`) — the **final brief-level section**. 7 resolved by decisions, 8 open assigned to CN docs (3 from Brief §11 + 5 new), 0 needs-ruling, 0 new CTRs. **Term 4's brief-level proposal (Sections 1–11) is complete.**
- **Term 4 Section 10 (Relationships with Other Terms)** ratified and merged (commit `0728389`). Depends-on (Term 7 only), depended-on-by (consistent with Section 5), the live cross-term picture (11 CTRs: 7 inbound incl. CTR-021 + 4 outbound; BO-1/2/3/5/6/7), and hand-off statements.
- **Term 4 Section 9 (Architect's Role)** ratified and merged (commit `6936b61`). Concept→Architect mapping for all 23 CN docs, 12 directives (6 from Brief refined + 6 from decisions), a "Too Detailed for Concept" boundary, and the Phase 1→2→3 sequence note. Architects framed as a future audience.
- **Term 4 Section 8 (Success Criteria)** ratified and merged (commit `87e35fe`). 15 machine-provable criteria (9 refined from Brief §8 + 6 from decisions), each with a "How Verified" column leaning on CN-4-019 invariant tests; a "Belongs Elsewhere" table excludes UX/pack-content/performance/cross-term items.
- **Term 4 Section 7 (Edge Cases)** ratified and merged (commit `8cc1f50`). 34 edge cases sorted: 19 resolved by decisions (D-007/D-002A/D-008/D-009/Charter/Section 3), 15 assigned to CN docs; no needs-ruling, no new CTRs.
- **Term 4 Section 6 (Key Concepts)** ratified and merged (commit `42b9d06`). The 23-doc CN catalogue grouped by clusters A–E with Decisions + CTR columns, overlap note, writing order (Phase 1 doctrine → Phase 2 CTR-driven → Phase 3 parallel), and cross-term summary. Supersedes Brief §13's First Tasks.
- **Term 4 Section 5 (Audiences Served)** ratified and merged (commit `1d2bbf8`). Audience table (Terms 1,2,3,5,6,7 + Architects as a future audience), promise referenced from Section 1, and an extensibility note (new consumers via CN-4-020; new materials via CTR + restraint test). No new CTRs.
- **Term 4 Section 4 (Scope · Out)** ratified and merged (commit `970709b`). Consolidated OUT list (26 items, 7 categories) with owners, the restraint statement, and three borderline rulings (Auth, prompt-injection, partitioning) — all OUT. No new CTRs.
- **Term 4 Section 3 (Scope · In)** ratified by the Concept Lead and merged (commit `3b8129d`). Encodes all 13 D-007 resolutions across five clusters; cross-term items (CTR-001/003/007/016/017/018) marked pending acceptance by Terms 1/5/6/7.

**Decision recorded**
- **D-007 — Foundation Scope Resolutions (Section 3).** 13 ratified resolutions for what the Kernel provides, incl. Saleable Line/Tender as value shapes (Checkout = Term 5 engine), the nine primitives (Consent ≠ Approval), compensating-event fold semantics, a distinct "advisor" principal + first-class platform scope, runtime advisor-scope enforcement, Foundation verification logic vs Term 7 surface, DSL grammar + sandboxed evaluator, resilience auto/human transitions, and **removal of "close period" from the Ledger primitive** (neutrality). Seeds CTR-016/017/018; resolves CTR-009; partially resolves CTR-007; proposes CTR-001/003.

**Merged to `main`**
- **Term 4 Sections 1 (Mission) & 2 (Mission Question)** ratified by the Concept Lead and merged to `main` (commit `65cb2a0`). Overseer review confirmed D-004/D-006 compliance; Term 4 applied two fixes (Parent links; "cracks" → "holds" consistency). First Term proposal sections to land on `main`.

**Ratified**
- **Term 4 Section 2 (Mission Question)** wording ratified by the Concept Lead — version E with "holds": *"How do we build a kernel that holds under decades of pressure — new verticals, new countries, AI that must remain advisory — and still proves to a court that nothing was ever silently changed?"* Charter §5.2 summary updated to match (de-numbered). Term 4 authors the file on its branch (D-006).

**Decisions recorded**
- **D-005 — AI Mode (Universal Per-Role Dashboard Advisor).** Every dashboard, for every role (tenant, platform steward, regional agent), gets an "AI Mode" built on the one Advisor Framework (D-002A) — prompt + proactive role-scoped advice, advisory-only, journaled, explainable. Enabled for all roles from the start. Seeds CTR-014, CTR-015.
- **D-006 — Collaboration Operating Model.** `main` is the single shared source of truth; each Term authors only its own files on a branch from `main`; the Overseer publishes shared docs, reviews, merges, answers Terms' questions, and keeps every Term current — and does NOT author Term content.

**Removed**
- The subagent-authored `terms/term-4/proposals/SECTION-1-MISSION.md` and `SECTION-2-MISSION-QUESTION.md` were discarded; Term 4 authors its own work on its branch (D-006).

**Shared docs published to `main`** so every Term session can read them.

## 2026-05-23 (Term 4 §1 review)

**Ratified**
- **Term 4 Section 1 (Mission)** — ratified by the Concept Lead. First proposal section to pass. Recorded in `terms/term-4/proposals/SECTION-1-MISSION.md`.

**Decision recorded**
- **D-004 — Concept-Writing Principles.** From the Term 4 Mission review: a Mission carries only the unchanging; no unverifiable hard numbers (supersedes the "10 years" vs "20 years" conflict in Term 4 Brief §1/§2); "invisible" is scoped to runtime users; audience/tone follow each Brief §5; Foundation Mission must elevate legal defensibility, neutrality-by-restraint, and the six Laws as technical invariants. D1–D4 rulings recorded.

**Glossary created**
- `MASTER-GLOSSARY.md` (Term 7 / CN-7-003), seeded with the Foundation-vs-Kernel ruling and core cross-Term terms.

## 2026-05-23 (later)

**Collaboration protocol clarified**
- `CROSS-TERM-COLLABORATION.md` — §5 rewritten to a discuss-then-write workflow (Terms bring points section by section to the Concept Lead; the proposal is written only after agreement). Added §6 Authority & Approval (Concept Lead is final; Overseer/Term 7 coordinates and must also agree; no complete system proposal until all seven Terms pass) and §7 the AI-agent operating model (one Term = one agent, one owner per file, CTR as the only message bus, per-Term folders, git branches).
- Terms 1–7 addenda updated to state the discuss-and-agree-first rule and the Concept Lead's final authority.

## 2026-05-23

**Added — concept-environment scaffolding**
- `DECISION-LOG.md` — binding cross-Term decisions (Term 7 / CN-7-007).
- `CROSS-TERM-COLLABORATION.md` — interaction map, boundary objects, and the Cross-Term Request (CTR) process.
- `CROSS-TERM-REQUEST-REGISTER.md` — living register of CTRs, seeded from D-001..D-003.
- This `CHANGELOG.md` (promised in Charter §11).

**Decisions recorded**
- **D-001 — Universal Checkout.** POS becomes a two-layer design: a universal Checkout/Tender layer over a shared Saleable Line abstraction, with each vertical feeding the lines. Lets any business type use one checkout.
- **D-002 — AI.** (A) One runtime Advisor Framework (advisory-only, scoped, model-swappable, Decision-Journal-logged) for tenants, agents, and platform. (B) A build-time Developer-AI environment that lives outside the kernel, briefs Claude Code (the implementer) to make GitHub changes, and releases only after human review + tests/doctrine invariants pass.
- **D-003 — Distribution.** Multiple regional agents per region (Law 6 intact) plus a referral program as an acquisition layer, not a replacement for compliance-accountable agency.

**Charter amendments**
- §2.4 added — pointer to standing decisions (Decision Log).
- §6.3 rule 6 added — Cross-Term Request requirement before the Architect phase.
- §11 updated — Decision Log + CHANGELOG now exist and are referenced.

**Term Briefs**
- Terms 1–7 each gained a "Standing Decisions & Cross-Term Work" addendum mapping D-001..D-003 to new concept docs and CTRs.
