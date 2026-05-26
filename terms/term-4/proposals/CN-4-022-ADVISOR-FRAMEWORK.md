# CN-4-022 — Advisor Framework & Role-Scope Binding

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-002A (Runtime Advisor Framework), D-005 (AI Mode — per-role dashboard advisor), D-007 #6 (runtime scope enforcement, Decision Journal fields).
> **CTRs:** CTR-007 (Terms 5/2/3 → us, Advisor Framework contract — negotiating). CTR-014 (Term 1 → us, AI cost governance + model approval — open).
> **Glossary:** See `MASTER-GLOSSARY.md` — Advisor, Decision Journal.
> **Depends on:** CN-4-001 (Doctrine), CN-4-006 (Isolation), CN-4-007 (Identity — advisor principal), CN-4-011 (Primitives — Consent), CN-4-013 (AI Guardrails & Decision Journal — schema owner), CN-4-017 (Resilience Modes).
> **Boundaries:** AI Guardrails + Decision Journal schema → CN-4-013; advisor identity → CN-4-007; specific advisors → Terms 5/2/1; AI Mode UX → Term 3; model governance → Term 1 (CTR-014); dashboard pattern → Term 7 (CTR-015); prompt-injection → Term 7 (CN-7-115); consent → CN-4-011.

---

## 1. What the Advisor Framework Is

The Advisor Framework is the single plug-in contract that every runtime AI advisor in BOS builds on (D-002A, D-005). It defines how an advisor is registered, what data it may see, how its scope is enforced, and what is recorded when it makes a suggestion.

Every advisor — tenant-facing, agent-facing, platform-facing, engine-specific — operates through this framework. There is one framework, not one per engine or per role.

---

## 2. The Advisor Contract

An advisor is defined by three things: **audience**, **scope**, and **model**.

### 2.1 Audience

Who this advisor serves. Audience is expressed as one of three **audience categories** — these correspond to the three groups in Charter §3 (Groups A, B, C) and are structural, not role-specific:

| Audience category | Charter group | Examples (illustrative) | Specific roles defined by |
|-------------------|--------------|------------------------|--------------------------|
| **Platform** | Group A — Platform Stewards | Admin, support, compliance officer | Term 1 |
| **Agent** | Group B — Regional Agents | Regional agent, agent staff | Term 2 |
| **Tenant** | Group C — Tenants | Owner, manager, cashier | Terms 3/5 |

Foundation defines these three audience categories. Higher Terms define the specific roles within each category. Foundation does not know "cashier" or "agent" — it sees "a role tag within the tenant audience category."

### 2.2 Scope

What data this advisor may read. Scope is bound to the advisor at registration and **enforced at runtime by the Kernel** (D-007 #6).

| Scope type | What the advisor sees | Constraint |
|-----------|----------------------|-----------|
| **Tenant-scoped** | One tenant's projections only | Default. Most advisors. Cannot see other tenants' data. |
| **Aggregate-opt-in** | Anonymised cross-tenant benchmarks | Only when the tenant has opted in via the Consent primitive (CN-4-011). Never exposes identifiable per-tenant data. |
| **Platform-scoped** | Cross-tenant aggregate or specific-tenant data (via platform scope, CN-4-006) | Platform advisors only. Audited via platform audit trail. |

Consent-dependent scopes (aggregate-opt-in) are **re-evaluated every invocation** — if the tenant has revoked consent since the last invocation, the scope is narrowed immediately (CN-4-011 revoke-wins).

### 2.3 Model

Which AI model backs this advisor. Swappable without changing the framework.

| Field | Description |
|-------|-------------|
| **model_id** | Provider + model name |
| **model_version** | Specific version |
| **model_config_ref** | Optional reference to model configuration (temperature, system prompt, etc.) — opaque to the Kernel |

Model approval and the model registry are Term 1's responsibility (CTR-014). Foundation provides the plug-in point; Term 1 governs which models are approved.

---

## 3. Runtime Scope Enforcement (D-007 #6)

The Advisor Framework enforces scope boundaries **at runtime in the Kernel** — not in the engine, not in the advisor's own code, not as a policy developers must remember.

### How It Works

| Step | What happens |
|------|-------------|
| 1 | Advisor is invoked (by a dashboard, a prompt, a proactive trigger) |
| 2 | The Kernel resolves the advisor's granted scope from its registration |
| 3 | Consent-dependent scopes are verified against current consent state (re-evaluated every invocation) |
| 4 | **All advisor context — prompt assembly and data retrieval — passes through the scope guard middleware** |
| 5 | The scope guard checks every data read: does this fall within the granted scope? |
| 6 | **If yes:** data is returned to the advisor |
| 7 | **If no:** read is **rejected**. The advisor receives no data. An audit entry is logged. |

### Full-Context Scope Coverage

The scope guard covers **all inputs to the advisor** — not just explicit data queries but the entire context the advisor operates on. No data enters the advisor's prompt or retrieval context without passing through the scope check. This prevents scope leakage through prompt assembly: even if an engine assembles a prompt for the advisor, every piece of data in that prompt has been scope-checked.

This is distinct from prompt-injection defence (Term 7, CN-7-115). Scope enforcement prevents the advisor from *seeing* out-of-scope data. Prompt-injection defence prevents *external input* from manipulating the advisor's behaviour. Both are necessary; they address different threats.

### Guarantees

- An advisor **cannot read data outside its granted scope** — provable at runtime (Section 8 #11).
- Scope enforcement is the Kernel's responsibility, not the engine's.
- If the scope guard rejects a read, the rejection is audited (actor = advisor principal, CN-4-007).

---

## 4. Role-Scope Binding (D-005)

D-005 says every dashboard, for every role, gets AI Mode. Foundation provides the **binding mechanism**; higher Terms define the specific bindings.

### The Binding

A role-scope binding is a **triple: role → scope → advisor**.

| Component | What it is | Who defines it |
|-----------|-----------|---------------|
| **Role** | A role tag within an audience category | Terms 1/2/3/5/6 |
| **Scope** | The data the advisor may read for this role | Foundation mechanism; Terms define the scope per role |
| **Advisor** | Which advisor instance serves this role | Terms 1/2/3/5 |

Foundation provides the data model for this triple and enforces the scope at runtime. Foundation does not define which roles exist, what scope each role gets, or which advisor serves each role — it provides the mechanism and the enforcement.

---

## 5. Advisory-Only Invariant (Law 3)

Every advisor, regardless of audience, scope, or model:

| Rule | What it means |
|------|---------------|
| **Never commits state** | An advisor never writes events, never creates commands autonomously, never modifies business data. |
| **Never approves transactions** | A human must approve any action resulting from advice. The action goes through the command bus with human confirmation. |
| **Reads projections only** | Advisors read derived state (projections), never the event stream directly. This makes advisors replay-safe — they do not affect event ordering or state derivation. |
| **May produce draft commands** | An advisor may generate a proposed command payload as part of its suggestion (e.g., a draft purchase order). The draft has **no effect** until a human submits it through the command bus. This is the AI Mode pattern (D-005): suggest an action, present it ready to execute, but the human decides. |
| **Every suggestion is journaled** | The Decision Journal (CN-4-013) records every suggestion. |

### Dual-Actor Audit

Every advisor invocation is a **dual-actor event** (CN-4-007): the advisor principal (model id/version, scope) and the invoking human principal are recorded together. The audit trail always shows both who asked and which AI answered.

---

## 6. Decision Journal Integration

Every advisor suggestion is recorded in the Decision Journal. CN-4-013 owns the full journal schema; the following are the **advisor-specific fields that CN-4-013 must include**:

| Field | Description |
|-------|-------------|
| **advisor_principal** | Full advisor identity (CN-4-007: advisor_id, model_id, model_version, granted_scope, framework_ref) |
| **invoking_principal** | The human who triggered the advisor (dual-actor) |
| **recommendation** | What the advisor suggested (human-readable) |
| **rationale** | Short reasoning summary — "show your work" |
| **data_ref** | What data the advisor saw (reference or summary — not the full dataset) |
| **prompt_ref** | Reference to the prompt/instruction that produced this suggestion (for reproducibility and legal defensibility) |
| **model_config_ref** | Reference to the model configuration active at the time (if applicable) |
| **human_decision** | Accepted, rejected, or ignored |
| **timestamp** | When the suggestion was made (clock protocol, CN-4-014) |
| **tenant_scope** | Which tenant this suggestion was made for |

**Full chain-of-thought is NOT stored** — it is model-internal, potentially enormous, and not required for explainability or legal defensibility (D-007 #6, CTR-009 resolved).

The journal is append-only (CN-4-001). A suggestion, once recorded, cannot be modified or deleted.

---

## 7. Open Questions Resolved by This Doc

| Question | Resolution |
|----------|-----------|
| **Section 11, N1 — Advisor cold-start** | The advisor contract includes a **data-sufficiency indicator**: each advisor declares whether it has enough data to advise. With insufficient data, the advisor may provide generic guidance (category-level, not tenant-specific) or indicate it cannot advise yet. The behaviour is per-advisor, not Foundation-wide — Foundation provides the sufficiency mechanism; each advisor defines its threshold. |
| **Section 11, N3 — Model-swap re-consent** | A model swap does not require tenant re-consent. Advisors are advisory-only (Law 3) — no state is committed. The Decision Journal records model_id/version per entry, so prior suggestions remain attributable to the prior model. Term 1 governs model approval (CTR-014); tenants are informed of model changes through Term 3 UX, not through a consent gate. |
| **Section 11, N4 — AI Mode in DEGRADED** | In DEGRADED mode (CN-4-017), projections may be stale. The advisor framework exposes a **freshness indicator** on the data it reads. Advisors in DEGRADED mode must: (a) disclose that data may be stale, (b) degrade their confidence level, or (c) decline to advise. In READ_ONLY mode, advisors may still read (projections are available) but any resulting action is blocked (no commands accepted). |
| **Section 11, q10 — AI knowledge boundary / benchmarking** | Cross-tenant benchmarking is opt-in only via the Consent primitive (CN-4-011). The advisor's scope must include `aggregate-opt-in`, the tenant must have a valid consent record, and the data provided is **anonymised aggregate** — statistical summaries across opted-in tenants, with no individual tenant's data recoverable. Consent is re-evaluated every invocation. |

---

## 8. Three Whys

### Why does this matter?

BOS wants AI support at every level — tenants, agents, platform staff. Without a framework, each engine builds its own AI integration with its own scope rules, its own journal, its own model binding. Scope enforcement is inconsistent. One engine's advisor reads cross-tenant data by accident. The audit trail is incomplete. Law 3 is honoured by convention, not by structure.

### Why does it belong in the Kernel?

The Advisor Framework is where Law 3 becomes mechanical. If advisory-only enforcement were per-engine, one engine's bug could allow an advisor to write events. The Kernel enforces the invariant: no advisor writes, period. The scope guard, the journal requirement, and the advisory-only constraint are Kernel-level because they must hold uniformly across every advisor on every dashboard.

### Why this design?

A plug-in contract (audience + scope + model) with Kernel-enforced scope and a mandatory journal gives maximum flexibility (any model, any role, any engine can have an advisor) with maximum safety (scope is enforced, suggestions are journaled, advisory-only holds). The alternative — per-engine AI integration — fragments safety guarantees and makes Law 3 unenforceable at the system level.

---

## 9. Example — Mama Amina Asks Her Dashboard

*Role and engine names appear only as illustrations.*

Mama Amina opens her dashboard. She types: "Should I reorder sugar? I think it's running low."

1. The dashboard invokes the **inventory advisor** (registered by Term 5; model=claude-sonnet/v4.6; scope=mama-amina-duka; audience=tenant).
2. The Kernel resolves the advisor's scope. Consent for benchmarking? Mama Amina has not opted in — scope remains tenant-only.
3. The scope guard verifies every data read: inventory projections for `mama-amina-duka` — **allowed**. No other tenant's data enters the prompt context.
4. The advisor reads: sugar stock = 5 bags, average weekly usage = 12 bags, last reorder = 3 weeks ago.
5. The advisor responds: "Yes — you have 5 bags of sugar, but you use about 12 per week. At this rate, you'll run out in 3 days. I recommend reordering 20 bags."
6. The advisor produces a **draft command**: a proposed purchase order for 20 bags of sugar from the last supplier. The draft is displayed but has no effect — it is a suggestion, not an action.
7. The **Decision Journal** records:
   - advisor: `inventory-advisor`, model: `claude-sonnet/v4.6`, scope: `mama-amina-duka`
   - invoking principal: `human:mama-amina`
   - recommendation: "Reorder 20 bags of sugar"
   - rationale: "5 bags remaining, 12/week average usage, 3-day runway"
   - data_ref: inventory projection snapshot ref
   - prompt_ref: ref to the prompt template + user query
   - human_decision: *(pending)*
8. Mama Amina clicks "Create Purchase Order." The draft command is submitted through the **command bus** — not through the advisor. The advisor suggested; the human decided; the command bus executes with Mama Amina as the acting principal.

If Mama Amina ignores the suggestion, the journal records `human_decision: ignored`. Tomorrow, the advisor may suggest again — it can check the journal for prior suggestions. The re-suggestion policy is per-advisor, not Foundation-wide.

---

## 10. Boundaries

| Topic | Lives in |
|-------|----------|
| AI Guardrails (forbidden operations list) + Decision Journal full schema | CN-4-013 (schema owner) |
| Advisor principal type (identity fields) | CN-4-007 |
| Consent primitive (opt-in for benchmarking, revoke-wins) | CN-4-011 |
| Specific advisor implementations (inventory, cash, BI/KPI) | Term 5 |
| Agent advisor | Term 2 |
| Tenant AI Mode UX / explainability presentation | Term 3 |
| AI cost governance / model registry / model approval | Term 1 (CTR-014) |
| AI Mode dashboard pattern (consistency across dashboards) | Term 7 (CN-7-004, CTR-015) |
| Prompt-injection defence (external input sanitisation) | Term 7 (CN-7-115) |
| Resilience modes (DEGRADED/READ_ONLY behaviour) | CN-4-017 |

---

## 11. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Model approval policy and model registry | Term 1 (CTR-014) | Foundation provides the model plug-in point; Term 1 governs approval |
| AI Mode dashboard pattern (consistent UX across all dashboards) | Term 7 (CTR-015) | Foundation provides the framework; Term 7 ensures pattern consistency |
| Explainability UX (how "show your work" is presented to tenants) | Term 3 | Foundation provides the journal fields; Term 3 presents them |
| Prompt-injection defence at the integration boundary | Term 7 (CN-7-115) | Foundation's scope guard prevents data leakage; Term 7 handles input sanitisation |
| Per-advisor data-sufficiency thresholds | Terms 5/2/1 | Foundation provides the sufficiency mechanism; thresholds are per-advisor |
| "Anonymised aggregate" — exact anonymisation mechanism (k-anonymity, differential privacy, etc.) | Architect phase | Concept says "statistical summaries, no individual tenant recoverable"; Architect specifies |

---

*— End of CN-4-022 —*
