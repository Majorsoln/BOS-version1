# Term 4 — Foundation
## Section 4 — Scope (Out)

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Status:** For Overseer review.
> **Governing decisions:** D-004 (neutral, no hard numbers), D-007 (scope resolutions).
> **Glossary:** *Foundation* = this Term; *Kernel* = the artifact it designs (see `MASTER-GLOSSARY.md`).

---

## What Foundation Does NOT Own

The following is explicitly outside the Kernel's scope. Each item names the Term that owns it. This list is consistent with Section 3's "Not in Foundation" rows and with the original Brief §4.

### Engine & Vertical Logic

| What is out | Owner |
|-------------|-------|
| Engine-specific event types and payloads | Terms 5/6 — engines define events within Foundation's contract |
| How engines compose primitives into business logic (journal entries, purchase orders, kitchen tickets) | Terms 5/6 |
| Vertical-specific uses of Saleable Lines | Term 6 |
| The Checkout engine (settlement, change computation, receipt issuance) | Term 5 — built on Foundation's primitives and value shapes (D-001, D-007 #1) |
| Role definitions per vertical (waiter, chef, fundi) | Terms 5/6 — defined within engine contracts |

### UI / Dashboards

| What is out | Owner |
|-------------|-------|
| How projections are visualised (dashboards, reports) | Term 3 (tenant) / Term 1 (platform) |
| AI Mode UX / explainability presentation | Term 3 |
| How resilience mode is surfaced to users | Term 3 |
| Time zone display preferences in UI | Term 3 |
| User management UI | Term 1 (platform) / Term 3 (tenant) |

### Auth / Integration / External

| What is out | Owner |
|-------------|-------|
| Authentication mechanisms (OAuth, SSO, MFA) | Term 7 (mechanism) / Term 1 (policy) |
| Document Verification public-facing surface (portal, QR endpoint, offline cache) | Term 7 (CTR-017) |
| How degraded mode affects integrations (paused webhooks, queued payments) | Term 7 |
| Prompt-injection defence | Term 7 (CN-7-115) |
| AI Mode dashboard pattern (consistency across dashboards) | Term 7 (CN-7-004, CTR-015) |
| Payment adapters (mobile money, card, bank) | Term 7 — attach at the tender boundary, not inside any vertical (D-001) |

### AI Governance & Operations

| What is out | Owner |
|-------------|-------|
| Specific advisor implementations (inventory, cash, BI/KPI) | Term 5 |
| Agent advisor | Term 2 |
| AI cost governance / model registry / model approval | Term 1 |
| Developer-AI governance (who triggers, who approves releases) | Term 1 (CTR-008) |

### Compliance Pack Content

| What is out | Owner |
|-------------|-------|
| Specific compliance packs (tax rules, document rules per country) | Term 1 (approval), Term 2 (input), Term 7 (validation) |
| Regional document legal requirements | Term 1 / Term 2 |
| Specific document templates (invoice, receipt, PO, credit note) | Terms 5/6 — templates use Foundation's Document primitive |

### Platform Policy

| What is out | Owner |
|-------------|-------|
| Platform access policies (which platform roles access what) | Term 1 |
| Subscription billing logic | Term 1 (rules) / Term 5 (accounting integration) |

### Infrastructure (Architect Phase)

| What is out | Owner |
|-------------|-------|
| Database engine choice, partitioning strategy, deployment topology | Deferred to Architect phase — not concept scope |

---

## Restraint

> *"The mark of a good kernel is what it refuses to do."*

This list is not a gap. It is the design. The Kernel is neutral by ignorance (Section 1). Everything listed above is excluded because including it would force the Kernel to know something that changes — a vertical, a country, a specific integration, a UI preference, an infrastructure vendor.

The rule: **if including something would require the Kernel to name a vertical, a country, an integration, or a specific role — it does not belong in Foundation.** It belongs to the Term that owns that specificity. Foundation provides materials and invariants; other Terms build structures.

---

## Borderline Cases (Ruled OUT)

Three items that could plausibly sit in Foundation but are deliberately excluded:

| Item | Why it could be Foundation | Why it is NOT | Ruling |
|------|---------------------------|---------------|--------|
| **Authentication (OAuth/SSO/MFA)** | Identity is Foundation's; authentication proves identity. | Auth mechanisms change with providers and standards. Foundation owns the Identity Primitive (who you are + what authority); authentication is how you prove it — an integration concern. Foundation trusts the identity once proven. | OUT — Term 7 (mechanism), Term 1 (policy). |
| **Prompt-injection defence** | AI Guardrails are Foundation's; prompt injection attacks AI. | Prompt injection is an external-data problem at the integration boundary. Foundation constrains what AI may do (scope, advisory-only); Term 7 handles what comes in from outside. Foundation's scope guard already blocks AI from reading outside its granted scope. | OUT — Term 7 (CN-7-115). |
| **Event store partitioning / archival** | Events are Foundation's sole truth; storage is a truth concern. | Partitioning is infrastructure, not concept. The concept states "append-only, hash-chained, immutable." How the store scales is an Architect-phase decision. CN-4-002 defines the contract; the Architect decides the mechanism. | OUT — Architect phase. |

---

*— End of Section 4 —*
