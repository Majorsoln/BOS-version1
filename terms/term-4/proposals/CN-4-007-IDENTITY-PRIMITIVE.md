# CN-4-007 — Identity Primitive

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-007 #4 (principal types, platform scope, dual-actor).
> **Glossary:** See `MASTER-GLOSSARY.md`.
> **Depends on:** CN-4-001 (Doctrine), CN-4-002 (Event Store Contract), CN-4-006 (Multi-Tenant Isolation).
> **Boundaries:** Authentication → Term 7/Term 1; role definitions → Terms 1/5/6; audit fields → CN-4-008; advisor scope → CN-4-022; isolation scope → CN-4-006.

---

## 1. The Four Principal Types

Every actor in BOS is one of four principal types. The principal type determines what identity fields the actor carries and how it appears in the `actor` field on events (CN-4-002), audit entries (CN-4-008), and Decision Journal records (CN-4-013).

### Human

| Field | Description |
|-------|-------------|
| **principal_type** | `human` |
| **user_id** | Unique identifier for the person |
| **authentication_ref** | Reference to how identity was proven (session, token) — opaque to the Kernel; details owned by Term 7/Term 1 |
| **role_bindings** | Which roles this human holds in the current scope. Role definitions are owned by higher Terms; the Kernel carries the bindings. |
| **tenant_scope** | Which tenant this human is currently operating within |

Covers: tenant owner, cashier, manager, platform admin, agent, support staff. The Kernel does not distinguish these — it sees "a human with role bindings in a scope."

### Advisor

| Field | Description |
|-------|-------------|
| **principal_type** | `advisor` |
| **advisor_id** | Identifier for this advisor instance |
| **model_id** | Which AI model (provider + model name) |
| **model_version** | Specific version of the model |
| **granted_scope** | The data scope this advisor may read (tenant, aggregate-opt-in, etc.) |
| **framework_ref** | Reference to the Advisor Framework registration (CN-4-022) |

Covers: every runtime AI advisor (tenant BI advisor, agent advisor, platform advisor, engine-specific advisors). The Kernel sees "an advisor with a model identity and a scope boundary."

### System

| Field | Description |
|-------|-------------|
| **principal_type** | `system` |
| **component_id** | Fine-grained identifier: `system:replay-engine`, `system:scheduler`, `system:projection-rebuild`, `system:numbering-allocator`, `system:resilience-monitor`, etc. |
| **trigger_ref** | What caused this system action (schedule definition, health-check result, event cascade) |

Covers: scheduled jobs (month-end close triggers), projection rebuilds, number block allocation, resilience mode transitions, any Kernel action that happens without a human initiating it directly.

### Machine

| Field | Description |
|-------|-------------|
| **principal_type** | `machine` |
| **client_id** | Identifier for the API client |
| **credential_ref** | Reference to the credential used (API key, service token) — opaque to the Kernel; details owned by Term 7/Term 1 |
| **tenant_scope** | Which tenant this machine client is operating for |

Covers: external API integrations, automated systems calling BOS on behalf of a tenant, webhook consumers. The Kernel sees "a machine client authenticated with a credential, scoped to a tenant."

---

## 2. Authority Chain

### What "Authority" Means at the Kernel Level

The Kernel does not implement RBAC, ACL, or permission tables. Those are higher-Term concerns. The Kernel's concept of authority is structural:

| Kernel-level authority | What it means |
|----------------------|---------------|
| **Principal type** | What kind of actor this is — determines what actions are structurally possible |
| **Scope** | Which tenant (or platform scope) this actor operates within — enforced by CN-4-006 |
| **Role bindings** | A set of role tags this principal holds in the current scope — the Kernel propagates them but does not define what each role means |
| **Constraints** | Hard limits the Kernel enforces regardless of role: advisor cannot write events (Law 3), system actions are audited, platform scope is read-only against tenant data (CN-4-006) |

### The Kernel's Responsibility vs Higher Terms

| Kernel (CN-4-007) | Higher Terms |
|-------------------|-------------|
| Defines the four principal types and their fields | Define specific roles (cashier, owner, agent, admin) |
| Carries role bindings on the actor | Define what each role may do (permissions) |
| Enforces structural constraints (advisor can't write, platform scope is read-only) | Define policy-level permissions (who may approve a refund) |
| Validates that an actor has a scope before any operation | Define which roles exist per vertical/platform |

The Kernel is a **carrier and enforcer of structure** — not a permission engine.

---

## 3. Dual-Actor Record

When a platform admin acts "as" a tenant — viewing their data for support, investigating an issue — both identities must be recorded. This resolves CN-4-006 open item R4.

### Composite Actor Shape

| Field | Description |
|-------|-------------|
| **acting_principal** | The human who is performing the action (the platform admin) — full human principal record |
| **operating_scope** | The scope they are acting within (e.g., tenant Mama Amina's scope) |
| **own_scope** | The principal's home scope (platform scope) — recorded to prove they had authority to cross scopes |

### Rules

- The dual-actor record is **required** whenever an actor operates in a scope different from their home scope.
- Both scopes are audited: the action appears in the **tenant's** audit trail (someone accessed my data) and the **platform's** audit trail (admin X accessed tenant Y).
- The tenant can query: "who from outside my business accessed my data?" The answer is complete and provable because the dual-actor record is structural — the Kernel requires it, not policy.

---

## 4. System-as-Actor

When BOS itself acts — a scheduled job, a projection rebuild, a resilience mode transition — a system principal is recorded. This resolves Section 11, q4.

### Which Components Get Identities

Every Kernel component that can cause an event or audit entry has a registered `component_id`. The following list is illustrative; the Architect defines the complete registry:

| Component | component_id | Typical triggers |
|-----------|-------------|-----------------|
| Scheduler | `system:scheduler` | Scheduled events (e.g., daily aggregation) |
| Replay Engine | `system:replay-engine` | Projection rebuilds, state reconstruction |
| Numbering Allocator | `system:numbering-allocator` | Number block reservation in DEGRADED mode |
| Resilience Monitor | `system:resilience-monitor` | NORMAL→DEGRADED transitions (auto, audited) |
| Projection Framework | `system:projection-framework` | Projection registration, rebuild orchestration |

### Audit Visibility

System actions are **fully auditable** — they appear in the audit trail with the same completeness as human actions. The `trigger_ref` field traces **why** the system acted: which schedule definition, which health check failure, which event cascade. A human can always answer: "what did the system do, when, and why?"

### System Actions and the Command Bus

System principals are **not privileged over policy**. A scheduled action (e.g., a month-end close trigger) goes through the command bus — the scheduler submits a command; the bus evaluates policy; the event is emitted or the command is rejected. The system principal is not a backdoor.

---

## 5. Advisor Principal Detail

The advisor principal carries enough to distinguish any AI action in the audit trail:

| What is traceable | How |
|-------------------|-----|
| Which AI made the suggestion | `model_id` + `model_version` |
| What data it could see | `granted_scope` |
| Which framework registration | `framework_ref` |
| What it suggested and what the human decided | Decision Journal (CN-4-013) — separate record, linked via the advisor principal |

The advisor principal makes AI actions **structurally distinguishable** from human actions. No confusion is possible in the audit trail between "a human did this" and "an AI suggested this."

CN-4-007 defines **who the advisor is** (identity). CN-4-022 defines **what the advisor may do** (scope, enforcement, framework contract). The two docs reference each other but do not overlap.

---

## 6. Three Whys

### Why does this matter?

When a tax officer, auditor, or tenant owner asks "who did this?" — BOS must answer precisely. Not "someone" or "the system" — but exactly which human, which AI model at which version, which system component triggered by what, which machine client. Without distinct principal types, the audit trail is ambiguous, and ambiguity in an audit is worse than no audit at all.

### Why does it belong in the Kernel?

Identity is the Kernel's answer to "who" on every event. If identity were per-engine, each engine would invent its own actor model — some would track AI, some wouldn't; some would record dual-actor, some wouldn't. The audit trail would be inconsistent across engines. The Kernel defines one actor model so that every event, in every engine, carries the same identity structure.

### Why this design?

Four principal types cover every actor in BOS without over-engineering. Human and machine are standard. Advisor is required by Law 3 (AI must be distinguishable). System is required because BOS acts on its own (scheduled jobs, resilience transitions). Fewer types would conflate actors (a system job recorded as "human" is a lie in the audit trail); more types would add complexity without clarity.

---

## 7. Example — One Hour in the Audit Trail

**Duka la Mama Amina, 14:30–15:10:**

| Time | Actor | Action | Principal type |
|------|-------|--------|---------------|
| 14:30 | `human:cashier-b` (tenant_scope=mama-amina-duka) | Completes sale TZS 12,000 | **Human** — cashier in tenant scope |
| 14:35 | `advisor:inventory-advisor` (model=claude-sonnet/v4.6, scope=mama-amina-duka) | Suggests reorder for low-stock item | **Advisor** — model identity and scope visible |
| 14:35 | *(Decision Journal)* | Mama Amina ignores the suggestion | Recorded: advisor principal + recommendation + human decision = ignored |
| 15:00 | `system:scheduler` (trigger=daily-aggregation) | Runs daily sales aggregation | **System** — component and trigger visible |
| 15:10 | `human:admin-042` (own_scope=platform, operating_scope=mama-amina-duka) | Views recent sales for support | **Human (dual-actor)** — platform admin + tenant scope both recorded |

Five entries, four principal types, each fully traceable. Mama Amina can query: "who accessed my data today?" and see all five — including the platform admin's support visit and the AI advisor's suggestion.

---

## 8. Boundaries

| Topic | Lives in |
|-------|----------|
| Authentication mechanisms (OAuth, SSO, MFA) | Term 7 (mechanism) / Term 1 (policy) — Scope Out |
| Role definitions per vertical (cashier, waiter, fundi) | Terms 5/6 — within engine contracts |
| Platform role definitions and policies | Term 1 |
| Which platform roles may use platform scope | Term 1 (CTR-016) |
| Audit log fields, query API, platform audit trail | CN-4-008 |
| Advisor scope enforcement at runtime | CN-4-022 |
| Tenant isolation scope mechanism | CN-4-006 |

---

## 9. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| How authentication proof is verified before identity is established | Term 7 / Term 1 | The Kernel trusts the identity once proven; it does not own the proof mechanism |
| How role bindings are assigned and managed (RBAC/ACL) | Term 1 (platform roles), Terms 5/6 (engine roles) | The Kernel carries bindings; it does not define role semantics |
| How the advisor framework registers advisors and their scope | CN-4-022 | CN-4-007 defines the advisor identity shape; CN-4-022 defines the registration and scope enforcement |
| Which system components are registered and their complete component_ids | CN-4-019 / Architect | The list in §4 is illustrative; the Architect defines the complete registry |
| **Delegation** | Future decision | One human acting on behalf of another within the same scope (agent staff for owner, family help, accountant for tenant). Could be represented by an optional `on_behalf_of` reference on the human principal, or treated as a higher-Term attribution concern. Not resolved here; **not a fifth principal type** — delegation is a relationship between two human principals, not a new kind of actor. |

---

*— End of CN-4-007 —*
