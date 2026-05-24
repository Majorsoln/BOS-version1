# Term 4 — Foundation
## Section 3 — Scope (In)

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Status:** For Overseer review — authored per D-007 ratified resolutions.
> **Governing decisions:** D-004 (neutral, no hard numbers), D-007 (scope resolutions 1–13).
> **Glossary:** *Foundation* = this Term; *Kernel* = the artifact it designs (see `MASTER-GLOSSARY.md`).

---

## Scope (In)

The Foundation Term owns the concept work for the BOS Kernel — everything below the engines and above the infrastructure. The Kernel provides **materials and invariants**; other Terms build **structures** with those materials. Foundation does not know what structures will be built.

The scope is organised into five clusters.

---

### Cluster A — Storage & Truth

The Kernel's role as the single source of truth for the entire system.

| Component | What Foundation provides | CN doc |
|-----------|-------------------------|--------|
| **Event Store** | Append-only, hash-chained, immutable store. The sole source of state in BOS. No hidden mutation anywhere. | CN-4-001, CN-4-002 |
| **Command Bus & Policy Engine** | Every state-changing request enters through the bus. The bus applies policy checks, accepts or rejects, and emits events. No command bypasses it. | CN-4-004 |
| **Hash Chain & Tamper Detection** | Proves the event store is intact at any point. Defines the response protocol when integrity is breached. | CN-4-003 |
| **Replay Engine** | Rebuilds state from events deterministically — full, scoped, or partial. Any projection can be reconstructed from events alone. | CN-4-009 |
| **Projection Framework** | How read-models are defined, registered, and rebuilt from events. Projections are derived views, never sources of truth. | CN-4-010 |
| **Snapshot Storage** | Performance caches explicitly marked with a **non-truth marker type**. A CI doctrine check prevents any code from treating a snapshot as a source of state. Runtime guards enforce where feasible. | CN-4-018 |

**Compensating events (D-007 #3):** The Kernel never deletes an event. When a wrong event was emitted, a compensating event is appended. Current truth is the **deterministic fold of all events including compensations**. The fold logic is defined per primitive — each primitive knows how to incorporate a compensation into its state.

**Projection rebuilds:** A rebuild runs without taking the system offline. Stale-but-available projections continue serving reads while the rebuild runs in background. If a rebuild is too disruptive, resilience mode transitions apply (see Cluster E).

**Not in Foundation:**
- Engine-specific event types and payloads (owned by Terms 5/6 — engines define events within Foundation's contract).
- How projections are visualised as dashboards or reports (Term 3 / Term 1).
- Infrastructure-level decisions (which database engine, partitioning strategy) — deferred to the Architect phase.

---

### Cluster B — Primitives

Reusable building blocks shared across every engine. A primitive defines a **shape** (data structure) and a set of **legal operations** — nothing more. No primitive names a vertical, a country, or an integration.

#### The Nine Primitives

| Primitive | What it knows | What it does NOT know | CN doc |
|-----------|---------------|----------------------|--------|
| **Ledger** | Entries and balances (double-entry building block) | Accounting rules, period close, chart of accounts | CN-4-011 |
| **Item / Service** | The abstract thing being tracked, sold, or moved | Which vertical uses it, how it is priced | CN-4-011 |
| **Actor** | Who is performing an action — principal type, authority | Role names per vertical, UI permissions | CN-4-007, CN-4-011 |
| **Document** | Hash-verified, immutable issued structure; template versioning | What the document is for (invoice, receipt, PO) | CN-4-011, CN-4-012 |
| **Inventory Movement** | A record of something moving (in, out, transfer, adjustment) | Warehouse layout, FIFO/LIFO costing rules | CN-4-011 |
| **Obligation** | Something owed — payable, receivable, credit, deposit | Payment methods, settlement logic | CN-4-011 |
| **Approval** | A simple, auditable gate requiring human authorisation | Multi-step workflows, escalation rules | CN-4-011 |
| **Workflow** | A multi-step state machine moving through defined stages | Which business process it models | CN-4-011 |
| **Consent** | Explicit permission recorded immutably (data processing, terms acceptance) | Privacy law specifics per jurisdiction | CN-4-011 |

**Consent is distinct from Approval (D-007 #2).** Both gate a decision; they differ in legal weight. Approval gates an operation; Consent gates data-processing rights and carries data-law (GDPR-equivalent) implications. Workflow is distinct from Approval — Approval is a one-step gate; Workflow is a multi-step state machine.

**"Close period" is NOT in the Ledger primitive (D-007 #13).** Period close is an Accounting concept owned by Term 5. The Ledger primitive knows only entries and balances. This preserves neutrality — the Ledger serves any engine that needs double-entry, not just accounting.

#### Value Shapes (D-001, D-007 #1)

| Shape | What it carries | CN doc |
|-------|----------------|--------|
| **Saleable Line / Charge** | The abstract payable line any vertical produces: item reference, description, quantity, unit price, tax treatment reference, source tag (opaque to checkout), optional discount references. | CN-4-021 |
| **Tender** | A payment method applied at settlement: method, amount, external reference. | CN-4-021 |

Saleable Line and Tender are **value shapes** — immutable data structures with no lifecycle or operations of their own. They are not full primitives. The **Universal Checkout engine** (owned by Term 5) composes Foundation's primitives (Obligation, Document, Ledger) and these value shapes into a settlement flow. Foundation provides the shapes; Term 5 builds the engine.

*CTR-001 (Term 6 → us) and CTR-003 (Term 5 → us): pending acceptance by Terms 5/6.*

**Not in Foundation:**
- How engines compose primitives into business logic (journal entries, purchase orders, kitchen tickets) — Term 5/6.
- Vertical-specific uses of Saleable Lines — Term 6.
- The Checkout engine itself (settlement, change, receipt issuance) — Term 5.

---

### Cluster C — Security & Isolation

The Kernel's guarantees that tenants are isolated, actors are identified, engines are bounded, and every action is auditable.

| Component | What Foundation provides | CN doc |
|-----------|-------------------------|--------|
| **Multi-Tenant Isolation** | Tenant A cannot see, query, or affect tenant B's data — enforced at every layer. | CN-4-006 |
| **Identity Primitive** | The actor model: who is acting, with what principal type, with what authority. | CN-4-007 |
| **Security Kernel** | Rate limits, anomaly detection, scope guards. | CN-4-016 |
| **Audit Log Primitive** | Append-only journal beneath every other audit log. Every action, by every actor, is recorded. | CN-4-008 |
| **Engine Contract Model** | How an engine declares itself — manifest, capabilities, boundaries. Engines communicate only via events (Law 2). | CN-4-005 |
| **Extension Points & Registration API** | The contract by which a new engine is added: manifest schema, capability declaration, compatibility check. A new engine plugs in without touching the Kernel. | CN-4-020 |
| **Doctrine Enforcement Checks** | Invariant tests proving the Kernel is sound. Enforced in CI; block any change that violates doctrine before release. | CN-4-019 |

#### Identity: Principal Types (D-007 #4)

The Identity Primitive defines distinct principal types:

| Principal type | Identity carries |
|---------------|-----------------|
| **Human** | User reference, authentication chain, role bindings |
| **Advisor** | Framework reference, model identity/version, granted scope |
| **System** | Fine-grained component identifier (e.g., `system:replay-engine`, `system:scheduler`) |
| **Machine** | API client identifier, credential reference |

Every event, audit entry, and Decision Journal record names its actor with principal type. The "advisor" principal (D-007 #4) ensures AI actions are always distinguishable from human actions in the audit trail.

#### Platform Scope (D-007 #4)

Platform scope is a **first-class parallel scope** — not an exception to tenant isolation, not a superset that weakens it. Platform-scope access (e.g., aggregate queries across tenants) has its own audit trail and is governed by platform roles defined by Term 1. Foundation provides the scope mechanism; Term 1 defines who may use it.

*CTR-016 (us → Term 1): pending acceptance.*

#### Engine Isolation Enforcement (D-007 #5)

Law 2 (engines are isolated) is enforced by **both**:
- **Runtime guards** — reject any cross-engine direct call at execution time.
- **CI doctrine checks** — detect imports or calls across engine boundaries before code reaches production.

Both layers exist because CI catches design mistakes early and runtime catches unexpected runtime paths.

*CTR-018 (us → Term 6): registration API contract pending acceptance.*

**Not in Foundation:**
- Authentication mechanisms (OAuth, SSO, MFA) — Term 7 (integration) / Term 1 (policy).
- User management UI — Term 1 (platform), Term 3 (tenant).
- Role definitions per vertical (waiter, chef, fundi) — Terms 5/6 define roles within the engine contract.
- Platform access policies (which platform roles may access what) — Term 1.

---

### Cluster D — AI & Documents

The Kernel's constraints on AI and its guarantees for document integrity.

| Component | What Foundation provides | CN doc |
|-----------|-------------------------|--------|
| **AI Guardrails** | Forbidden operations list; scope guards; the boundary AI cannot cross. AI never commits state, never writes events, never approves a transaction. | CN-4-013 |
| **Decision Journal** | Immutable log of every AI suggestion: model id/version, prompt reference, data reference, recommendation, rationale, human decision. | CN-4-013 |
| **Advisor Framework contract** | The plug-in definition: audience, data scope, model identity. One framework for all advisors across all roles (D-002A, D-005). | CN-4-022 |
| **Advisor scope enforcement** | Runtime Kernel enforcement of scope boundaries — an advisor cannot read data outside its granted scope, regardless of what the engine implements. | CN-4-022 |
| **Developer-AI kernel boundary** | Declaration that Developer-AI is outside the Kernel. It produces code proposals, never runtime state. Every change passes human review + CI doctrine gate. | CN-4-023 |
| **Doctrine-enforcement gate** | The automated gate every proposed change must pass before release — the invariant tests that prove the Kernel is sound after any modification. | CN-4-019, CN-4-023 |
| **Document Engine** | Structured document builder with hash-verified, immutable outputs. Template versioning (old documents remain immutable under old templates). | CN-4-012 |
| **Numbering Engine** | Fiscal-compliant, branch-aware, deterministic document numbering. Block-reserved for DEGRADED mode so numbering never halts. | CN-4-012 |
| **Document Verification (logic)** | The hash-check capability that proves a document is authentic and unaltered. | CN-4-012 |

#### Decision Journal Fields (D-007 #6)

The journal schema exposes enough for tenant-facing explainability without leaking model internals:

- **Recommendation** — what the advisor suggested (human-readable).
- **Rationale** — short reasoning summary ("show your work").
- **Data reference** — what data the advisor saw (or a pointer to it).
- **Model identity/version** — which model produced the suggestion.
- **Human decision** — accepted, rejected, or ignored.

Full chain-of-thought is **not** stored — it is model-internal, potentially enormous, and not required for explainability or legal defensibility.

*CTR-007 (Terms 5,2,3 → us): partially resolved; pending full acceptance.*
*CTR-009 (Term 3 → us): resolved — schema includes recommendation + rationale + data ref.*
*CTR-014 (Term 1 → us): pending — cost governance and model approval remain with Term 1.*

#### Role-Scope Binding (D-005)

Foundation provides the **binding mechanism**: role → scope → advisor. It does not define which roles exist or what scope each role receives. Terms 1/2/3 and engines define specific roles and scopes within the framework. Foundation is neutral — it does not know "cashier" or "agent," only "a role with a scope."

#### Document Verification Split (D-007 #7)

Foundation owns the **verification logic and hash check** — the capability to prove a document's authenticity. Term 7 owns the **external-facing surface** (the public portal, QR-scan endpoint, offline/cached verification). The logic is Kernel; the exposure is integration.

*CTR-017 (us → Term 7): pending acceptance.*

#### Document Numbering in DEGRADED Mode (D-007 #12)

Numbering is deterministic and branch/terminal-scoped. In DEGRADED mode, **pre-reserved number blocks** per branch or terminal ensure numbering never halts, even without connectivity to the central store. Blocks are reconciled when NORMAL mode resumes.

**Not in Foundation:**
- Specific advisor implementations (inventory, cash, BI/KPI advisor) — Term 5.
- Agent advisor — Term 2.
- AI Mode UX / explainability presentation — Term 3.
- AI cost governance / model registry / model approval — Term 1.
- Developer-AI governance (who triggers, who approves releases) — Term 1 (CTR-008).
- AI Mode dashboard pattern (consistency across dashboards) — Term 7.
- Prompt-injection defence — Term 7.
- Specific document templates (invoice, receipt, PO, credit note) — Terms 5/6.
- Regional document legal requirements — Term 1 (approval) / Term 2 (input).

---

### Cluster E — Time & Resilience

The Kernel's handling of time, compliance grammar, and system operational states.

| Component | What Foundation provides | CN doc |
|-----------|-------------------------|--------|
| **Time Authority** | Deterministic clock protocol. No `datetime.now()` inside engine logic — all time goes through the clock protocol. Replay-safe: identical events produce identical state regardless of wall-clock time. | CN-4-014 |
| **Compliance DSL Framework** | The grammar compliance packs are written in, **plus** a sandboxed runtime evaluator. One language, one evaluator, consistent results across all engines and regions. | CN-4-015 |
| **Compliance DSL Extension Mechanism** | A reviewed, sandboxed extension point for rules the core grammar cannot express. Extensions are tested and reviewed like grammar changes; guarded against overuse so the escape hatch does not become the default. | CN-4-015 |
| **Resilience Modes** | The state machine: NORMAL → DEGRADED → READ_ONLY. Defines health checks, transition triggers, and recovery protocol. | CN-4-017 |

#### Resilience Transitions (D-007 #9)

| Transition | Trigger |
|-----------|---------|
| NORMAL → DEGRADED | **Automatic** — deterministic health-check failure, audited and replay-safe. Cannot wait for human in a safety-critical moment. |
| DEGRADED → READ_ONLY | **Human-gated** — severe state requiring human judgement. |
| Any recovery (back toward NORMAL) | **Human-gated** — always requires explicit human confirmation. |

Auto-transitions are audited as events (replay-safe). The Kernel never silently changes its own operational state without a record.

#### Time and Fiscal Day Boundaries

A business's fiscal time zone is anchored at registration and stored as an event. It does not change on replay. The Time Authority provides the clock; the fiscal zone is a tenant property, not a clock property. "Day" boundaries for replay and period logic are always computed from the registered fiscal zone — deterministic, never ambient.

**Not in Foundation:**
- Specific compliance packs (tax rules, document rules per country) — Term 1 (approval), Term 2 (regional input), Term 7 (validation).
- How resilience mode is surfaced to users — Term 3.
- How degraded mode affects integrations (paused webhooks, queued payments) — Term 7.
- Time zone display preferences in UI — Term 3.

---

## Cross-Term Items (Pending)

These items are part of Foundation's scope but depend on acceptance by other Terms:

| CTR | To Term | What is pending | Resolution when accepted |
|-----|---------|-----------------|--------------------------|
| CTR-001 | 5, 6 | Saleable Line & Tender as value shapes (not primitives) | Shapes finalised in CN-4-021 |
| CTR-003 | 5, 6 | Checkout = universal engine (Term 5), not a Foundation primitive | Foundation provides shapes; Term 5 builds engine |
| CTR-007 | 5, 2, 3 | Advisor Framework contract + Decision Journal fields | Framework contract finalised in CN-4-022 |
| CTR-016 | 1 | Platform scope as a first-class parallel scope | Platform roles and policies defined by Term 1 |
| CTR-017 | 7 | Document verification logic (Foundation) / surface (Term 7) | Split confirmed; Term 7 builds portal |
| CTR-018 | 6 | Extension Points include a registration API concept | Term 6 follows manifest + capability contract |

These parts of Section 3 are not final until the receiving Terms accept.

---

*— End of Section 3 —*
