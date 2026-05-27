# CN-4-013 — AI Guardrails & Decision Journal

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-002A (Runtime Advisor Framework), D-007 #6 (Decision Journal fields, no full CoT), CTR-009 (resolved — explainability fields delivered).
> **Glossary:** See `MASTER-GLOSSARY.md` — Advisor, Decision Journal, Fold, Projection.
> **Depends on:** CN-4-001 (Doctrine — append-only, event-sourced), CN-4-002 (Event Store Contract), CN-4-006 (Isolation), CN-4-007 (Identity — advisor + system principals), CN-4-008 (Audit Log Primitive — the append-only journal beneath all audit logs), CN-4-010 (Projection Framework), CN-4-014 (Time Authority), CN-4-022 (Advisor Framework — contributed fields).
> **Boundaries:** Advisor Framework contract → CN-4-022; advisor identity → CN-4-007; audit log primitive → CN-4-008; doctrine enforcement tests → CN-4-019; specific advisors → Terms 5/2/1; explainability UX → Term 3; prompt-injection → Term 7.

---

## 1. What This Doc Defines

CN-4-013 owns two things:

- **AI Guardrails** — the forbidden operations and constraint framework that keeps AI advisory-only (Law 3). Three-deep enforcement: structural, runtime, CI.
- **Decision Journal** — the full, authoritative schema for recording every AI suggestion. CN-4-022 contributed the advisor-specific fields; this doc defines the complete schema and its event-sourced nature.

---

## 2. AI Guardrails

### Forbidden Operations

Every advisor, regardless of audience, scope, or model, is **structurally prevented** from:

| Forbidden operation | How enforced |
|--------------------|-------------|
| **Writing events** | The command bus rejects any event-emission attempt from an advisor principal (CN-4-007). The advisor principal type has no write capability. |
| **Creating commands autonomously** | An advisor may produce a draft command (CN-4-022 §5), but submitting it requires a human principal. The command bus rejects commands with an advisor as the sole acting principal. |
| **Approving transactions** | Approval requires a human principal (CN-4-011 Approval primitive). Advisor principals cannot satisfy the approval gate. |
| **Modifying business state** | Corollary of the above three — no write, no command, no approval = no state change. |
| **Reading outside granted scope** | Runtime scope guard (CN-4-022 §3) rejects out-of-scope reads per invocation. |
| **Reading the event stream directly** | Advisors read projections only (CN-4-022 §5). Never the raw event stream — this keeps advisors replay-safe. |

### Three-Deep Enforcement

| Layer | What it catches |
|-------|----------------|
| **Kernel structural** | Advisor principal type has no write capability; command bus rejects advisor-only commands. This is not policy — it is the absence of a mechanism. An advisor cannot write because no write path exists for its principal type. |
| **Runtime scope guard** | Out-of-scope reads blocked per invocation (CN-4-022 §3). Consent re-evaluated every invocation. |
| **CI doctrine checks** | CN-4-019 verifies no advisor code path writes events or creates commands (Section 8 #15). Catches design mistakes before production. |

Any single layer failing is caught by the others. The guardrails are defence-in-depth, not redundancy.

---

## 3. Decision Journal

### Relationship to the Audit Log Primitive (CN-4-008)

The Audit Log Primitive (CN-4-008) is "the append-only journal beneath every other audit log." The Decision Journal is a **specialised audit log** — it shares the Audit Log's foundational properties (append-only, immutable, queryable, event-sourced) but adds AI-specific fields (model identity, recommendation, rationale, data reference, human decision).

The relationship:
- **CN-4-008** defines the general audit log contract: append-only, what every audit entry must contain (actor, timestamp, tenant scope, action).
- **CN-4-013** extends this for AI suggestions: the general audit fields plus the AI-specific fields below.
- The Decision Journal is **not a separate store** — it is a specialised projection (CN-4-010) over the same event-sourced foundation. The Architect decides whether it is physically co-located with the general audit log or a separate projection; the concept guarantees it shares the same append-only, replayable, tamper-evident properties.

### The Journal Is Event-Sourced

Decision Journal entries are **events**, not mutable records. This follows CN-4-001 (Law 1: state is derived from events only).

| Event type | When emitted | Emitted by |
|-----------|-------------|-----------|
| `kernel.advisor.suggestion.recorded.v1` | When an advisor produces a suggestion | The **Kernel / Advisor Framework** acting as a system principal (`system:advisor-framework`, CN-4-007) |
| `kernel.advisor.decision.recorded.v1` | When a human accepts or rejects a suggestion | The **Kernel** recording the human's decision, with the human as the acting principal |

**The advisor does not write these events.** The Advisor Framework (a Kernel component) observes the advisor's output and records the suggestion as a system-principal event. This resolves the paradox: "advisors cannot write events" is true — the advisor produces output; the Kernel records it. The advisor's output passes through the framework; the framework writes the journal event under its own system principal.

The Decision Journal view is a **projection** (CN-4-010) rebuilt from these events — replayable, deterministic, auditable.

### Full Schema

| Field | Description | Source |
|-------|-------------|--------|
| **journal_entry_id** | Unique identifier for this entry | Generated at recording |
| **advisor_principal** | Full advisor identity: advisor_id, model_id, model_version, granted_scope, framework_ref | CN-4-007 |
| **invoking_principal** | The human who triggered the advisor (dual-actor) | CN-4-007, CN-4-022 §5 |
| **tenant_scope** | Which tenant this suggestion was made for | CN-4-006 |
| **recommendation** | What the advisor suggested (human-readable) | CN-4-022 §6 |
| **rationale** | Short reasoning summary — "show your work" | CN-4-022 §6, CTR-009 |
| **data_ref** | Point-in-time reference to what data the advisor saw — a projection snapshot at a specific global_position or snapshot id (CN-4-002, CN-4-009). Never a live query that could return different data later. This protects "show your work" for replay and legal defensibility. | CN-4-022 §6 |
| **prompt_ref** | Reference to the prompt/instruction that produced this suggestion (for reproducibility) | CN-4-022 §6 |
| **model_config_ref** | Reference to the model configuration active at the time (if applicable) | CN-4-022 §6 |
| **draft_command_ref** | If the advisor produced a draft command: a reference to the immutable draft. The referenced draft must not change after recording. Snapshot storage is opt-in for engines that require it; reference is the default. | CN-4-022 §5 |
| **human_decision** | Accepted or rejected. Recorded as a separate event (`kernel.advisor.decision.recorded.v1`) — not a mutation of the suggestion event. | CN-4-022 §6 |
| **decision_timestamp** | When the human decided (null if still pending) | Clock protocol (CN-4-014) |
| **suggestion_timestamp** | When the suggestion was made | Clock protocol (CN-4-014) |
| **freshness_indicator** | Whether the data was fresh or stale at time of suggestion (DEGRADED mode, CN-4-017). Retained in the journal for legal defensibility — "the advisor disclosed that data may be stale" is part of the suggestion's context. | CN-4-022 §7 |

**Full chain-of-thought is NOT stored** — it is model-internal, potentially enormous, and not required for explainability or legal defensibility (D-007 #6, CTR-009 resolved).

### human_decision Lifecycle

| State | How it occurs |
|-------|---------------|
| **pending** | The suggestion event has been recorded; no decision event yet. This is the stored state. |
| **accepted** | A `kernel.advisor.decision.recorded.v1` event is appended with decision=accepted and the human principal who accepted. |
| **rejected** | Same, with decision=rejected. |
| **ignored** | **Derived, not stored.** No decision event within a defined window, or the suggestion was superseded by a newer one. "Ignored" is computed by the projection — no phantom actor records a non-action. |

This follows CN-4-001: no mutation. The suggestion event stands; the decision event appends. "Ignored" is absence of a decision event, computed at read time.

---

## 4. "Previously Suggested" Check

The journal enables a **previously-suggested check**: before making a suggestion, an advisor can query the journal projection for recent suggestions to the same tenant on the same topic. This enables:

- **Don't nag**: if the same suggestion was made yesterday and ignored, the advisor may choose to stay quiet (per-advisor policy, not Foundation-wide — CN-4-022 §7 N1).
- **Pattern detection**: if a suggestion has been rejected multiple times, the advisor may escalate or change its approach.
- **Audit of repeated advice**: a regulator can see how many times an advisor suggested something before it was acted on.

The check is a **read against the journal projection**, not a separate mechanism.

---

## 5. Three Whys

### Why does this matter?

Law 3 says "AI is advisory only." Without guardrails and a journal, Law 3 is a statement of intent, not a structural guarantee. One engine's bug could let an advisor write an event. One advisor's suggestion could be untraceable — if it was wrong and harmful, who is accountable? The guardrails make Law 3 enforceable; the journal makes every AI action traceable.

### Why does it belong in the Kernel?

If guardrails were per-engine, one engine could accidentally give its advisor write access. If journals were per-engine, the schema would drift — one engine records model_id, another doesn't, a third forgets the human decision. The Kernel owns one guardrail framework and one journal schema so that Law 3 and traceability are uniform across every advisor on every dashboard.

### Why this design?

Three-deep enforcement (structural + runtime + CI) because any single layer can have bugs. The journal is event-sourced and projection-based so it inherits the Kernel's own guarantees: append-only, replayable, deterministic, tamper-evident. The schema is deliberately flat and queryable (not nested, not engine-specific) so that cross-engine, cross-advisor analysis is possible — a regulator can ask "show me all AI suggestions for this tenant this year" and get a coherent answer.

---

## 6. Example — An Advisor Suggestion That Was Wrong

**Mama Amina's BI advisor suggests extending credit:**

Mama Amina's BI advisor (Term 5, model=claude-sonnet/v4.6) analyses her cash flow and suggests: "You should extend credit to customer Mzee Juma — he has paid on time for the last 6 months."

The **Advisor Framework** (system:advisor-framework) records the suggestion as `kernel.advisor.suggestion.recorded.v1`:
- advisor: BI advisor, claude-sonnet/v4.6, scope=mama-amina-duka
- invoking_principal: human:mama-amina
- recommendation: "Extend credit to Mzee Juma"
- rationale: "6 months on-time payment history; average order TZS 150,000"
- data_ref: cash-flow projection at global_position 142,307
- freshness_indicator: fresh (NORMAL mode)

Mama Amina accepts. The Kernel records `kernel.advisor.decision.recorded.v1`:
- human_decision: accepted
- acting_principal: human:mama-amina
- decision_timestamp: 2026-03-15T10:42

She extends credit. Mzee Juma defaults.

Six months later, her accountant asks: "Why did we extend credit to Mzee Juma?"

The Decision Journal projection answers with every field. The trail is complete:
- The AI suggested (with what data, what model, what reasoning).
- The human decided (who, when).
- The data the advisor saw is frozen at the point-in-time reference — replayable, not a live query that returns different numbers today.
- Liability is clear: Mama Amina made the business decision. The advisor's role was advisory. The journal proves exactly what information she had.

If the AI had seen data outside its scope, the scope guard rejection logs would show it. If the AI had committed the credit autonomously, the guardrails would have prevented it — no write path exists for the advisor principal.

---

## 7. Boundaries

| Topic | Lives in |
|-------|----------|
| Advisor Framework contract (audience, scope, model, registration) | CN-4-022 |
| Advisor principal type (identity fields) | CN-4-007 |
| Audit Log Primitive (the general append-only journal) | CN-4-008 |
| Projection Framework (journal as projection) | CN-4-010 |
| Runtime scope enforcement | CN-4-022 §3 |
| Specific advisor implementations | Terms 5/2/1 |
| Explainability UX (how journal fields are shown to tenants) | Term 3 |
| Prompt-injection defence | Term 7 (CN-7-115) |
| Doctrine enforcement tests (CI layer of guardrails) | CN-4-019 |
| Consent for benchmarking | CN-4-011 |

---

## 8. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Journal query API design (pagination, filtering, aggregation) | Architect phase | Concept says "every field is queryable via the projection"; Architect designs the API |
| Physical co-location with general audit log vs separate projection | Architect phase | Concept says "specialised projection over the same event-sourced foundation"; Architect decides topology |
| Retention policy for journal entries | Term 1 | Governance: how long are AI suggestions kept? Legal requirements may mandate long retention. |
| Whether the journal supports annotations (human adds a note after the fact) | Future decision | Annotations would be separate append-only events referencing the original — never modifications |
| Per-advisor "previously suggested" policy defaults | Terms 5/2/1 | Foundation provides the mechanism; each advisor defines its nag-suppression rules |
| "Ignored" window definition (how long before pending becomes ignored) | Terms 5/2/1 + Architect | Per-advisor or system-wide default; the projection computes it |

---

*— End of CN-4-013 —*
