# Term 4 — Foundation
## Section 9 — Architect's Role for This Term

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Status:** For Overseer review.
> **Governing decisions:** D-004 (neutral, no hard numbers), D-007 (scope resolutions), D-008 (consent), D-009 (freeze doctrine).
> **Glossary:** *Foundation* = this Term; *Kernel* = the artifact it designs (see `MASTER-GLOSSARY.md`).

---

## Framing

Per Section 5 (ratified), Architects are a future audience. This section describes what the Architect produces **after** the Concept Phase — translating each ratified CN doc into implementable design. It is not work happening now.

The Architect translates docs in the same dependency order as Section 6 (Phase 1 → 2 → 3).

---

## Concept Output → Architect Output

### Cluster A — Storage & Truth

| CN doc | Concept delivers | Architect produces |
|--------|-----------------|-------------------|
| CN-4-001 (Event Sourcing Doctrine) | Why events are sole truth; freeze doctrine (D-009); compensating event fold rules | Doctrine-as-code spec; event immutability guarantees at storage layer |
| CN-4-002 (Event Store Contract) | Event shape, required fields, schema versioning contract, ordering guarantees | Storage schema; append-only triggers; sequence/ordering mechanism; event envelope spec |
| CN-4-003 (Hash Chain & Tamper Detection) | Integrity proof requirements; breach response protocol | Hash algorithm choice; chain verification API; tamper alert and quarantine procedure |
| CN-4-004 (Command Bus & Policy Engine) | Request→event flow; policy evaluation; rejection design | Dispatcher architecture; policy evaluation pipeline; rejection event schema |
| CN-4-009 (Replay Engine) | Determinism requirements; scoped/partial replay; rebuild-during-operation rules | Replay engine architecture; checkpoint storage; scope-filtering; background rebuild orchestration |
| CN-4-010 (Projection Framework) | Read-model lifecycle; registration; rebuild protocol | Projection protocol; registry data model; idempotent handler pattern; failure recovery |
| CN-4-018 (Snapshot Storage) | Non-truth marker; CI/runtime guard requirements | Snapshot table with non-truth marker column; expiry logic; guard implementation (type-level + CI) |

### Cluster B — Primitives

| CN doc | Concept delivers | Architect produces |
|--------|-----------------|-------------------|
| CN-4-011 (Primitive Catalog) | 9 primitives: shapes, legal operations, fold logic (D-007 #2/#3/#13) | Each primitive's data shape (frozen dataclass); operation signatures; compensating-fold implementation spec |
| CN-4-021 (Saleable Line & Tender) | Value shapes for checkout; neutrality constraints (D-001/D-007 #1) | Data contracts (immutable value objects); validation rules; interface between Foundation shapes and Term 5 Checkout engine |

### Cluster C — Security & Isolation

| CN doc | Concept delivers | Architect produces |
|--------|-----------------|-------------------|
| CN-4-005 (Engine Contract Model) | Manifest, capabilities, isolation guarantees; subscription framework | Engine registry data model; manifest schema; compatibility check API; subscription registration mechanism |
| CN-4-006 (Multi-Tenant Isolation) | Scope enforcement at every layer; platform scope (D-007 #4); GDPR export scoping | Tenant scope object; middleware enforcement; leak-detection test suite; platform-scope implementation |
| CN-4-007 (Identity Primitive) | 4 principal types; dual-actor record; authority chain (D-007 #4) | Actor model schema; principal type enum; authentication chain spec; dual-scope audit record design |
| CN-4-008 (Audit Log Primitive) | Append-only journal; required fields; platform-scope auditability | Audit table schema; query API (pagination, scoping); retention policy; regulator-query interface |
| CN-4-016 (Security Primitives) | Rate limits, anomaly detection, scope guards | Rate limiter design; anomaly rule engine; scope guard middleware |
| CN-4-019 (Doctrine Enforcement Checks) | What invariants exist; how they map to CI (Section 8 criteria) | Invariant test suite design; CI hooks; linter rules; pre-commit and pipeline integration |
| CN-4-020 (Extension Points) | Registration API; manifest + compatibility check (D-007 #11) | Registration endpoint spec; manifest schema; compatibility check algorithm; new-engine checklist automation |

### Cluster D — AI & Documents

| CN doc | Concept delivers | Architect produces |
|--------|-----------------|-------------------|
| CN-4-013 (AI Guardrails & Decision Journal) | Forbidden operations; journal schema; previously-suggested check | Forbidden-ops list (code-level); journal table schema; query API for explainability |
| CN-4-022 (Advisor Framework & Role-Scope Binding) | Plug-in contract; runtime scope enforcement; role→scope→advisor binding (D-002A/D-005/D-007 #6) | Advisor registration interface; scope-enforcement middleware; role-scope binding data model; model plug-in protocol |
| CN-4-023 (Kernel Boundary & Doctrine Gate) | Developer-AI out-of-kernel declaration; release gate requirements (D-002B) | CI/CD doctrine-gate design; test-suite-as-gate configuration; release pipeline spec |
| CN-4-012 (Document Engine & Numbering) | Hash-verified documents; template versioning; deterministic numbering; block-reservation; freeze doctrine (D-007 #7/#12, D-009) | Template schema + version control; render pipeline; numbering allocator (branch/terminal blocks); verification API |

### Cluster E — Time & Resilience

| CN doc | Concept delivers | Architect produces |
|--------|-----------------|-------------------|
| CN-4-014 (Time Authority) | Clock protocol; fiscal zone anchoring; replay-safe time | Clock interface (FixedClock for tests, SystemClock for runtime); fiscal-zone storage; time-injection pattern |
| CN-4-015 (Compliance DSL Framework) | Grammar; sandboxed evaluator; extension mechanism (D-007 #8); pack versioning (D-009) | Grammar spec (BNF/PEG); validation engine; sandboxed runtime evaluator; extension-point security model; pack version storage |
| CN-4-017 (Resilience Modes) | State machine; transition triggers (auto/human-gated); recovery protocol (D-007 #9) | Mode state machine implementation; health-check triggers; transition event schema; recovery confirmation flow |

---

## Architect's Directives to Developer

Constraints the Architect must honour and pass to the Developer. These are concept-level requirements, not implementation choices.

### From Brief §9

| Directive | Status |
|-----------|--------|
| All Kernel code is pure Python (no framework dependency in core) | Keep |
| Every primitive is an immutable dataclass (`frozen=True`) | Keep |
| No `datetime.now()` inside engine logic — must go through the Clock protocol (CN-4-014) | Keep |
| Every audit entry, event, and decision-journal entry is append-only | Keep |
| Every command rejection is policy-traced (the rejection names the policy that caused it) | Keep (spirit); Architect decides the exact signature |
| All Section 8 criteria are enforced as automated invariant tests in CI (CN-4-019) | Refined (replaces "11+ architectural invariants") |

### New Directives (from Decisions)

| Directive | Source |
|-----------|--------|
| Runtime scope guard for advisors — every advisor read goes through scope-enforcement middleware | D-007 #6, CN-4-022 |
| Consent-gated command rejection — the command bus checks the consent primitive before allowing outreach commands | D-008, CN-4-011 |
| Freeze/versioning — every event references the compliance-pack version at emission; every document references pack + template version at issuance | D-009 |
| Non-truth marker on snapshots — snapshots carry a type-level marker that prevents compile-time/runtime use as source of state | D-007 #10, CN-4-018 |
| Engine isolation enforced at runtime AND CI — cross-engine imports/calls rejected at both layers | D-007 #5, CN-4-005 |
| Platform-scope operations carry dual audit (actor identity + operating scope) | D-007 #4, CN-4-007/008 |

---

## Too Detailed for Concept

The following are decisions the Architect makes — not specified by the Concept Phase:

| Item | Why it belongs to the Architect |
|------|--------------------------------|
| Specific database choice (Postgres, etc.) | Infrastructure; concept says "append-only, hash-chained" |
| Hash algorithm choice (SHA-256, BLAKE3, etc.) | Implementation; concept says "prove integrity" |
| Specific middleware framework | Implementation; concept says "scope enforcement at runtime" |
| Sequence number implementation (database sequence vs logical clock) | Implementation; concept says "ordering guarantees" |
| Pack version storage format (semver, date-based, etc.) | Implementation; concept says "version referenced at emission" |

Section 9 states **what** the Architect must produce and **what constraints** must be honoured. **How** they implement it is their professional judgement.

---

*— End of Section 9 —*
