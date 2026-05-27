# CN-4-008 — Audit Log Primitive

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-007 #4 (platform scope, dual-actor).
> **Glossary:** See `MASTER-GLOSSARY.md`.
> **Depends on:** CN-4-001 (Doctrine — append-only), CN-4-002 (Event Store Contract), CN-4-004 (Command Bus — produces events), CN-4-006 (Isolation — tenant/platform scope), CN-4-007 (Identity — actor/dual-actor), CN-4-010 (Projection Framework — audit log is a projection).
> **Boundaries:** Decision Journal (specialised audit) → CN-4-013; command bus → CN-4-004; tenant isolation → CN-4-006; identity → CN-4-007; projection framework → CN-4-010; doctrine check (audit completeness) → CN-4-019; retention policy → Term 1; read-level logging → Architect phase.

---

## 1. What This Doc Defines

CN-4-008 defines the **general audit log** — "the append-only journal beneath every other audit log" (Section 3). It specifies what every audit entry must contain, how the log is structured, and how it is queried. Specialised audit logs (like the Decision Journal, CN-4-013) extend this schema with domain-specific fields.

---

## 2. What Gets Audited

Every **state-changing action** in BOS produces an audit entry.

| Action type | Examples |
|-------------|---------|
| **Commands accepted** | Sale completed, PO approved, document issued |
| **Commands rejected** | Consent missing, scope mismatch, mode-gate rejection (CN-4-004 §4) |
| **System actions** | Scheduled job triggered, projection rebuilt, resilience mode transitioned |
| **Platform-scope operations** | Cross-tenant aggregate query, admin entering tenant scope (CN-4-006 §2) |
| **Advisor suggestions** | AI suggestion recorded (CN-4-013 — specialised entries extending this schema) |
| **Registration events** | Engine registered/updated/deactivated (CN-4-020 §4) |

### What Does NOT Get Audited (at this layer)

- **In-scope reads by the data owner** (a tenant user reading their own dashboard) — not state-changing, not cross-scope. Read-level logging for these is an Architect/infrastructure concern. **Important distinction:** cross-scope and platform-scope reads ARE audited (Section 8 #10, CN-4-006 §2, DC-011/DC-020) — a platform admin querying a tenant's data is always an audited operation, even though it is a read, because it crosses scope boundaries.
- **Engine-internal computation** (calculating a total, formatting a template) — only the resulting event/command matters.

---

## 3. Audit Entry Schema

Every audit entry contains these fields. This is the **general schema** — specialised audit logs extend it with domain-specific fields.

**Granularity:** One audit entry per event. Each event in the store has its own audit entry with its own `event_ref`. A command that produces multiple events atomically (CN-4-004 §2, step 4a) produces multiple audit entries, all sharing the same `correlation_id`. The full "lifecycle of a command" is reconstructed by querying all entries with that `correlation_id`.

| Field | Description |
|-------|-------------|
| **audit_entry_id** | Unique identifier |
| **event_ref** | Reference to the event in the event store that this entry corresponds to |
| **action_type** | What kind of action (command_accepted, command_rejected, system_action, platform_operation, advisor_suggestion, registration) |
| **actor** | Full principal record (CN-4-007 — principal type, identity, scope) |
| **tenant_scope** | Which tenant this action relates to (null for platform-only operations) |
| **target** | What was acted upon (aggregate ref, document ref, engine ref) |
| **outcome** | What happened (accepted, rejected + policy_name + reason, completed) |
| **timestamp** | Clock protocol (CN-4-014) |
| **correlation_id** | Links to the originating command (CN-4-002, CN-4-004) |

### Dual-Actor Entries

When a platform admin operates in a tenant's scope (CN-4-007 §3), the audit entry carries the **full dual-actor record** (acting_principal + operating_scope + own_scope). The entry appears in **both** the tenant's audit trail and the platform's audit trail.

---

## 4. The Audit Log Is a Projection

The audit log is **not a separate store** alongside the event store. It is a **projection** (CN-4-010) — a read-model derived from events.

This means:
- **Replayable** — the audit log can be rebuilt from events at any time.
- **Deterministic** — replay produces the same audit entries.
- **Integrity inherited** — the audit log inherits the event store's guarantees (hash chain, append-only, CN-4-003).
- **Never out of sync** — if the audit log is stale or corrupted, it is rebuilt from events. No reconciliation needed.

The events ARE the source of truth (CN-4-001). The audit log is a structured, queryable view built from them.

---

## 5. Two Audit Trails

BOS maintains two parallel audit trails, both derived from the same event store:

| Trail | What it contains | Who queries it |
|-------|-----------------|---------------|
| **Tenant audit trail** | All actions within a tenant's scope — commands, rejections, advisor suggestions, system actions affecting that tenant | Tenant owner ("who accessed my data?"), tenant auditors, regulators |
| **Platform audit trail** | All platform-scope operations — cross-tenant queries, admin actions, engine registrations, resilience transitions | Platform governance (Term 1), platform auditors |

A dual-actor action (admin in tenant scope) appears in **both** trails. The trails are separate projections over the same events, scoped by tenant_scope (tenant trail) or platform_scope (platform trail).

---

## 6. Querying the Audit Log

The audit log must support queries across these dimensions:

| Dimension | Example |
|-----------|---------|
| **By tenant** | "All actions for Mama Amina's duka this month" |
| **By actor** | "All actions by cashier-c today" |
| **By action type** | "All rejections this week" |
| **By date range** | "Everything between March 1–31" |
| **By target** | "All actions on sale-0042" |
| **By correlation** | "The full lifecycle of command X" |
| **Cross-dimensional** | "All platform admin actions on tenant Y in Q1" |

Large queries (regulator requests spanning months) must support **pagination**. The pagination mechanism is Architect-phase; the concept guarantees the query dimensions exist.

---

## 7. Audit Completeness Invariant

**Every state-changing action produces an audit entry.** There are no "dark" actions — actions that change state but leave no trace.

This is **structurally guaranteed**: the audit log is a projection over events, and every state change goes through the command bus (CN-4-004) which produces events. No event = no state change. Every event = an audit entry in the projection.

This invariant will be added to CN-4-019's doctrine-check catalog through the living-catalog process as a formal doctrine check.

---

## 8. Three Whys

### Why does this matter?

When a tenant asks "who touched my data?" — BOS must answer completely and truthfully. When a regulator asks "show me every action this business took in March" — the answer must be comprehensive, not partial. Without a structured audit log, these questions require forensic investigation of raw events. With the audit log, they are structured queries against a purpose-built view.

### Why does it belong in the Kernel?

If audit logging were per-engine, each engine would decide what to log and how. One engine might log rejections; another might not. One might record the actor; another might forget. The Kernel defines one audit schema and ensures every event produces a corresponding entry — the schema is uniform and completeness is guaranteed.

### Why this design?

The audit log as a projection (not a separate store) is the key design choice. A separate audit store creates a second source of truth that can diverge from events. A projection derived from events cannot diverge — it IS the events, structured differently. Rebuild the projection, and the audit log is restored. No reconciliation, no sync problems, no "the audit log says X but the events say Y."

---

## 9. Example — Mama Amina Asks "Who Accessed My Data?"

Mama Amina, concerned about privacy after a support interaction, queries her tenant audit trail for May 2026.

The audit log returns:

| Timestamp | Actor | Action | Outcome |
|-----------|-------|--------|---------|
| May 2 14:30 | human:cashier-b | retail.sale.complete | accepted |
| May 2 14:35 | advisor:inventory-advisor (claude-sonnet/v4.6) | suggestion recorded | (journal entry — CN-4-013) |
| May 5 09:00 | system:scheduler | daily aggregation | completed |
| May 10 15:10 | human:admin-042 (platform → mama-amina-duka) | viewed recent sales (support) | accepted (dual-actor) |
| May 12 11:00 | human:cashier-c | retail.sale.complete (receipt_delivery=sms) | **rejected** (consent.outreach_requires_consent) |

Mama Amina can see: her cashiers' actions, the AI advisor's suggestion, a system job, the platform admin's support visit (dual-actor — she knows exactly who, when, and why), and a rejected attempt. Every entry is traceable, every actor is identified by principal type, and the rejected attempt is visible.

She can answer her own question: "Yes, platform admin-042 accessed my data on May 10 for support. The AI inventory advisor made a suggestion on May 2. Both are recorded."

---

## 10. Boundaries

| Topic | Lives in |
|-------|----------|
| Decision Journal (specialised audit projection for AI suggestions) | CN-4-013 |
| Actor model / principal types | CN-4-007 |
| Tenant isolation / platform scope | CN-4-006 |
| Command bus (produces the events the audit log derives from) | CN-4-004 |
| Event store (source events) | CN-4-002 |
| Projection framework (audit log is a projection) | CN-4-010 |
| Doctrine enforcement (audit-completeness check) | CN-4-019 (pending living-catalog addition) |
| Read-level access logging | Architect phase / infrastructure |
| Audit retention policy | Term 1 (governance) |
| Regulator query interface design | Architect phase |

---

## 11. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Audit query API design (pagination, filtering, export format) | Architect phase | Concept defines query dimensions; Architect designs the API |
| Retention policy (how long audit entries are kept) | Term 1 | Retention operates on the projection/archival layer only — source events are never deleted (CN-4-001). Right-to-erasure is a separate future decision (crypto-shredding, flagged in CN-4-006 §9) — it does not mean deleting audit entries. Legal requirements may mandate long retention. |
| Read-level access logging (should non-state-changing reads be logged?) | Architect phase / Future decision | Concept audits state changes; read logging is infrastructure |
| Audit-completeness doctrine check (DC-NNN) | CN-4-019 (living-catalog) | Will be added as a formal check once this doc is merged |
| Performance of large-range queries (regulator requests) | Architect phase | Concept guarantees pagination; optimisation is Architect's |
| Audit log export format (for regulators, GDPR requests) | Architect phase + Term 1 | May align with GDPR export (CN-4-006 §3) |

---

*— End of CN-4-008 —*
