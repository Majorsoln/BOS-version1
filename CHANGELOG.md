# Changelog

> Records every change to the Charter and to cross-Term decisions.
> Managed by Term 7 — Integration & Coherence (Charter §11).

## 2026-05-24

**Merged to `main`**
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
