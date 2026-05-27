# CN-4-017 — Resilience Modes

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-007 #9 (auto NORMAL→DEGRADED, human-gated deeper/recovery), D-004 (no hard numbers — thresholds are config).
> **Glossary:** See `MASTER-GLOSSARY.md`.
> **Depends on:** CN-4-003 (Hash Chain — breach as health dimension), CN-4-004 (Command Bus — write-mode gate), CN-4-007 (Identity — system:resilience-monitor), CN-4-008 (Audit Log — transitions recorded), CN-4-009 (Replay — availability per mode), CN-4-010 (Projection Framework — freshness, rebuild), CN-4-012 (Document Engine — block-reserved numbering), CN-4-016 (Security — rate-limit tightening), CN-4-020 (Extension Points — registration gating), CN-4-022 (Advisor Framework — freshness disclosure).
> **Boundaries:** Command bus write gate → CN-4-004; projection freshness → CN-4-010; numbering blocks → CN-4-012; rate-limit tightening → CN-4-016; registration gating → CN-4-020; advisor freshness → CN-4-022; tenant notification → Term 3; integration degradation → Term 7; transition governance → Term 1.

---

## 1. What This Doc Defines

CN-4-017 defines the NORMAL / DEGRADED / READ_ONLY state machine: the three operational modes, transition rules, what each mode means for every Kernel component, and how the system behaves during degradation. Multiple Foundation docs reference this; this doc formalises the complete model.

---

## 2. The Three Modes

| Mode | System state | What works |
|------|-------------|-----------|
| **NORMAL** | Full operation. All components available. | Everything — commands, events, projections, advisors, registration, replay, numbering. |
| **DEGRADED** | Partial operation. Some components impaired. System is functional but constrained. | Commands accepted (some may be gated). Projections may be stale. Advisors disclose freshness. Numbering uses reserved blocks. Registration requires human approval. Rate limits may tighten. |
| **READ_ONLY** | Minimum operation. No state changes accepted. | Reads from projections. Point-in-time replay. Verification. No commands accepted. No events emitted. No registration. |

There are exactly three modes. Planned maintenance is handled as a human-gated NORMAL→READ_ONLY transition, distinguished from emergency by the trigger field on the transition event (see §4).

---

## 3. State Machine

```
         auto (health-check)
NORMAL ──────────────────────► DEGRADED
  ▲  ▲                          │  ▲
  │  │    human-gated            │  │  human-gated
  │  │    (recovery)             │  │  (partial recovery)
  │  │                           ▼  │
  │  └───────────────────── READ_ONLY
  │                              ▲
  │   human-gated (emergency     │
  │    or planned maintenance)   │
  └──────────────────────────────┘
```

### Transition Rules (D-007 #9)

| Transition | Trigger | Authority |
|-----------|---------|----------|
| **NORMAL → DEGRADED** | Health-check failure — automated, deterministic. Cannot wait for human in a safety-critical moment. | **Automatic** — audited by `system:resilience-monitor`. |
| **DEGRADED → READ_ONLY** | Severe degradation (multiple components failing, data integrity at risk). | **Human-gated** — requires platform admin confirmation. |
| **DEGRADED → NORMAL** | Health checks pass, impaired components recovered. | **Human-gated** — always requires explicit human confirmation. No auto-recovery. |
| **READ_ONLY → DEGRADED** | Partial recovery, some components back. | **Human-gated** — requires platform admin. |
| **READ_ONLY → NORMAL** | Full recovery. | **Human-gated** — requires platform admin. |
| **NORMAL → READ_ONLY** | Extreme event (emergency) or planned maintenance. | **Human-gated** — platform admin action. |

### Why Auto for NORMAL→DEGRADED Only

A health-check failure is time-sensitive — waiting for a human could mean data loss or inconsistency. The auto-transition protects the system. All other transitions require human judgement: entering READ_ONLY is severe; leaving DEGRADED too early risks relapse.

---

## 4. Transitions Are Events (Runtime Layer)

Every mode transition is recorded as an event:

| Event type | Fields |
|-----------|--------|
| `kernel.resilience.mode_changed.v1` | old_mode, new_mode, trigger (health-check ID, hash-breach ref, or human actor), trigger_type (automated / human / planned_maintenance / emergency), impaired_dimensions (which health dimensions triggered — see §5), timestamp |

Emitted by `system:resilience-monitor` (automatic transitions) or the human platform admin (gated transitions).

**Runtime layer, not replay-deterministic (per CN-4-016 pattern):** Health checks run at runtime (live measurements, ephemeral). The mode-transition event is a **recorded fact** — replay sees that the transition occurred but does not re-run the health checks that caused it. The health-check logic is ephemeral; the transition event is permanent.

### In-Flight Command Semantics

A command is evaluated against the mode **at the time of its processing** (atomic). A mode change does not retroactively affect events already committed. Commands submitted after the transition see the new mode.

---

## 5. Health Checks

Health checks are the triggers for automatic NORMAL→DEGRADED transitions. They are deterministic, measurable, and auditable.

| Health dimension | What is measured |
|-----------------|-----------------|
| **Event store latency** | Write latency exceeds acceptable range |
| **Projection lag** | Projection processed_position (CN-4-010 §4) is too far behind the store's latest position |
| **Component availability** | A Kernel component (numbering allocator, hash verifier, etc.) is unreachable |
| **Error rate** | Command rejection rate due to system errors (not policy rejections) exceeds threshold |
| **Hash-chain breach** | Hash verification (CN-4-003) detects a break — a health dimension that triggers auto NORMAL→DEGRADED. Going to READ_ONLY remains human-gated even for breach (D-007 #9). This resolves CN-4-003 §9 open item. |

### Thresholds Are Configuration

Health-check thresholds are **not hard-coded** (D-004). They are configuration, adjustable by platform governance (Term 1). Foundation defines **what is measured**; thresholds are set per deployment.

### Impaired-Dimension Context

The transition event records **which health dimension(s) triggered** the transition. Components can respond proportionately — e.g., if projection lag triggered DEGRADED but the event store is healthy, commands still flow normally while projections catch up. Full granularity of per-component response is an Architect decision; the concept provides the dimension context so proportionate response is possible.

---

## 6. Per-Component Behaviour by Mode

| Component | NORMAL | DEGRADED | READ_ONLY |
|-----------|--------|----------|-----------|
| **Command Bus** (CN-4-004) | Full flow | Gated — some commands may be deferred or restricted per degraded-mode rules | All state-changing commands rejected |
| **Event Store** | Full writes | Writes accepted (may be slower) | No writes |
| **Projections** (CN-4-010) | Live updates | May be stale (freshness exposed via processed_position) | Available for reads; rebuilds that persist derived state may be deferred |
| **Replay** (CN-4-009) | Full capabilities | Available, may be deprioritised | Read-only replay available (point-in-time, verification) |
| **Advisors** (CN-4-022) | Full | Disclose data may be stale (freshness_indicator); may degrade confidence or decline | May read projections; resulting actions blocked (no commands) |
| **Numbering** (CN-4-012) | Central allocator | Block-reserved from pre-reserved blocks | No new documents (no writes) |
| **Registration** (CN-4-020) | Automated after checks | Human-gated (safety during instability) | Blocked |
| **Rate limits** (CN-4-016) | Normal thresholds | May tighten (lower ceilings to protect limited resources) | N/A (no commands) |
| **Hash verification** (CN-4-003) | Available | Available | Available |
| **Audit log** (CN-4-008) | Live | Live (records all actions including mode transitions) | Available for reads |

---

## 7. Three Whys

### Why does this matter?

Systems fail. Networks partition. Databases slow down. Storage fills up. Without defined resilience modes, the system either crashes entirely (unacceptable — tenants lose access) or continues in an undefined state (dangerous — partial writes, stale data served as fresh, inconsistent numbering). Resilience modes make degradation **predictable and auditable** — the system knows what it can and cannot do, and tells its users honestly.

### Why does it belong in the Kernel?

If resilience were per-engine, each engine would define its own degradation — one accepts writes while another rejects them, creating inconsistent state. The Kernel defines one set of modes that apply system-wide. When the system is DEGRADED, every component knows it and behaves accordingly.

### Why this design?

Three modes cover the operational spectrum without over-complicating. Auto for NORMAL→DEGRADED (safety-critical, time-sensitive) + human-gated for everything else (high-consequence decisions) balances responsiveness with human oversight. The impaired-dimension context allows proportionate response without requiring a separate mode per failure type. Per-component behaviour tables make the impact concrete — no engine has to guess what "DEGRADED" means.

---

## 8. Example — Projection Lag Triggers DEGRADED Mode

Mama Amina's duka is operating normally. A database issue causes the projection framework to fall behind — the audit log projection is 500 events behind the store.

1. **Health check** detects: projection lag exceeds threshold (dimension: `projection_lag`).
2. `system:resilience-monitor` emits `kernel.resilience.mode_changed.v1`: NORMAL → DEGRADED, trigger = `health:projection-lag-exceeded`, trigger_type = automated, impaired_dimensions = [`projection_lag`].
3. **Impact:**
   - Commands still accepted — cashiers can still make sales (event store is healthy).
   - Projections are stale — dashboards show data that may be minutes behind. The freshness indicator shows "as of 5 minutes ago."
   - Mama Amina's AI advisor discloses: "Data may be delayed — my suggestions are based on information from 5 minutes ago."
   - Numbering switches to block-reserved — the POS terminal uses pre-reserved numbers.
   - Rate limits tighten — fewer commands per second to reduce pressure.
   - Registration of new engines requires human approval.
4. The database issue is resolved. Projections catch up. Health checks pass.
5. A platform admin confirms recovery: `kernel.resilience.mode_changed.v1`: DEGRADED → NORMAL, trigger_type = human, actor = `human:admin-042`.
6. Normal operation resumes. The entire incident is auditable — every transition, every health dimension, every actor recorded.

---

## 9. Boundaries

| Topic | Lives in |
|-------|----------|
| Command bus write-mode gate | CN-4-004 |
| Projection freshness / rebuild | CN-4-010 |
| Replay availability per mode | CN-4-009 |
| Numbering block-reservation | CN-4-012 |
| Advisor freshness disclosure | CN-4-022 |
| Registration gating | CN-4-020 |
| Rate-limit tightening | CN-4-016 |
| Hash verification / breach detection | CN-4-003 |
| Audit log (transitions recorded) | CN-4-008 |
| System principal (resilience-monitor) | CN-4-007 |
| Mode transition governance (who approves human-gated) | Term 1 |
| Tenant notification of mode changes | Term 3 |
| Integration behaviour during degradation | Term 7 |

---

## 10. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Health-check thresholds (specific values per dimension) | Architect phase + Term 1 | Foundation defines what is measured; thresholds are config |
| Health-check implementation (polling, push, hybrid) | Architect phase | Mechanism is Architect's |
| DEGRADED-mode command restrictions (which commands deferred vs allowed) | Architect phase + Terms 5/6 | Concept says "gated"; specific rules per engine are Architect + engine |
| Tenant notification of mode changes | Term 3 | UX concern — how tenants are informed |
| Integration behaviour in DEGRADED/READ_ONLY | Term 7 | How integrations degrade |
| Rate-limit tightening specifics in DEGRADED | Architect phase + Term 1 | Config/governance |
| Per-dimension proportionate response (how components use impaired_dimensions to respond) | Architect phase | Concept provides the context; Architect designs the response logic |

---

*— End of CN-4-017 —*
