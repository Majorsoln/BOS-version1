# Term 4 — Foundation
## Section 6 — Key Concepts This Term Will Produce

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Status:** For Overseer review.
> **Governing decisions:** D-004 (neutral, no hard numbers), D-007 (scope resolutions).
> **Glossary:** *Foundation* = this Term; *Kernel* = the artifact it designs (see `MASTER-GLOSSARY.md`).
> **Supersedes:** Brief §13 (First Tasks) — the catalogue and writing order below replace the Brief's original sequence.

---

## Concept Document Catalogue

Foundation produces **23 concept documents**, grouped by the five clusters defined in Section 3.

### Cluster A — Storage & Truth

| ID | Concept | What it answers | Decisions | CTRs |
|----|---------|-----------------|-----------|------|
| CN-4-001 | Event Sourcing Doctrine | Why events are the only source of truth, and what this implies for every other Foundation decision | — | — |
| CN-4-002 | Event Store Contract | What an event is, what it must contain, how it is stored, how schemas version | — | — |
| CN-4-003 | Hash Chain & Tamper Detection | How the store proves its integrity; what happens when integrity is breached | — | — |
| CN-4-004 | Command Bus & Policy Engine | How a request becomes an event (or doesn't); policy evaluation; rejection events | — | — |
| CN-4-009 | Replay Engine | Rebuilding state deterministically — full, scoped, partial; checkpoint handling | — | — |
| CN-4-010 | Projection Framework | How read-models are defined, registered, and rebuilt from events | — | — |
| CN-4-018 | Snapshot Storage (non-truth) | Performance caches; non-truth marker type; CI/runtime guards against misuse | D-007 #10 | — |

### Cluster B — Primitives

| ID | Concept | What it answers | Decisions | CTRs |
|----|---------|-----------------|-----------|------|
| CN-4-011 | Primitive Catalog | What the nine primitives are, why each exists, their shapes and legal operations | D-007 #2, #13 | — |
| CN-4-021 | Saleable Line & Tender Value Shapes | The abstract payable line and tender shapes for universal checkout | D-001, D-007 #1 | **CTR-001, CTR-003** (pending Term 5/6) |

### Cluster C — Security & Isolation

| ID | Concept | What it answers | Decisions | CTRs |
|----|---------|-----------------|-----------|------|
| CN-4-005 | Engine Contract Model | How an engine declares itself; manifest, capabilities, boundaries; isolation guarantees | — | — |
| CN-4-006 | Multi-Tenant Isolation | How tenant A cannot see tenant B — at every layer; platform scope mechanism (parallel scope) | D-007 #4 | **CTR-016** (pending Term 1) |
| CN-4-007 | Identity Primitive | Actor model; four principal types (human, advisor, system, machine); authority chain; platform principal type | D-007 #4 | **CTR-016** (pending Term 1) |
| CN-4-008 | Audit Log Primitive | The append-only journal beneath every other audit log; what every entry must contain | — | — |
| CN-4-016 | Security Primitives | Rate limits, anomaly detection, scope guards | — | — |
| CN-4-019 | Doctrine Enforcement Checks | The invariant tests that prove the Kernel is sound; CI enforcement | — | — |
| CN-4-020 | Extension Points (How to Add a New Engine) | Registration API concept — manifest, capability declaration, compatibility check | D-007 #11 | **CTR-018** (pending Term 6) |

### Cluster D — AI & Documents

| ID | Concept | What it answers | Decisions | CTRs |
|----|---------|-----------------|-----------|------|
| CN-4-013 | AI Guardrails & Decision Journal | Forbidden operations; scope guards; the journal schema (recommendation, rationale, data ref, model id, human decision) | — | — |
| CN-4-022 | Advisor Framework & Role-Scope Binding | The plug-in contract (audience, scope, model); runtime scope enforcement; role-scope binding mechanism | D-002A, D-005, D-007 #6 | **CTR-007** (pending Term 5/2/3), **CTR-014** (pending Term 1) |
| CN-4-023 | Kernel Boundary & Doctrine Gate | Developer-AI is out-of-kernel; the doctrine-enforcement gate every change must pass | D-002B | **CTR-008** (us → Term 1) |
| CN-4-012 | Document Engine & Numbering | Hash-verified documents; template versioning; fiscal-compliant numbering; block-reserved for DEGRADED; verification logic | D-007 #7, #12 | **CTR-017** (pending Term 7) |

### Cluster E — Time & Resilience

| ID | Concept | What it answers | Decisions | CTRs |
|----|---------|-----------------|-----------|------|
| CN-4-014 | Time Authority | Deterministic clock protocol; replay-safe time; fiscal day boundaries anchored at registration | — | — |
| CN-4-015 | Compliance DSL Framework | The grammar compliance packs are written in; sandboxed evaluator; reviewed extension mechanism | D-007 #8 | — |
| CN-4-017 | Resilience Modes | NORMAL / DEGRADED / READ_ONLY state machine; auto vs human-gated transitions; recovery protocol | D-007 #9 | — |

---

## Overlap Note

No CN docs need merging or splitting. The boundaries between related docs are clear:

| Pair | Distinction |
|------|-------------|
| CN-4-013 / CN-4-022 | CN-4-013 = constraints (what AI cannot do + journal schema); CN-4-022 = contract (how an advisor is registered + scope enforcement) |
| CN-4-019 / CN-4-023 | CN-4-019 = the invariant test suite (what is tested, CI hooks); CN-4-023 = the boundary declaration (why and when the tests run) |
| CN-4-005 / CN-4-020 | CN-4-005 = what an engine IS (contract, manifest, isolation); CN-4-020 = how a new one is ADDED (registration process, compatibility check) |
| CN-4-006 / CN-4-007 | Platform scope mechanism lives in CN-4-006 (isolation — where scoping is enforced); platform principal type lives in CN-4-007 (identity — where actor types are defined). Both reference CTR-016. |

---

## Writing Order

### Phase 1 — Doctrine Foundation (sequential, each builds on the last)

| Order | CN | Why this position |
|-------|-----|-------------------|
| 1st | CN-4-001 | Event Sourcing Doctrine — justifies everything else |
| 2nd | CN-4-002 | Event Store Contract — the concrete shape of an event |
| 3rd | CN-4-006 | Multi-Tenant Isolation — every other doc must respect it |
| 4th | CN-4-007 | Identity Primitive — the actor on every event |
| 5th | CN-4-005 | Engine Contract Model — enables Terms 5/6 |
| 6th | CN-4-011 | Primitive Catalog — the materials Terms 5/6 use |
| 7th | CN-4-020 | Extension Points — the contract for adding engines |

### Phase 2 — Decision-Driven Docs (after Phase 1; carry pending CTRs)

| Order | CN | Why this position |
|-------|-----|-------------------|
| 8th | CN-4-021 | Saleable Line & Tender — unblocks CTR-001/003 (Terms 5/6 waiting) |
| 9th | CN-4-022 | Advisor Framework — unblocks CTR-007/014 (Terms 5/2/3/1 waiting) |
| 10th | CN-4-023 | Kernel Boundary — unblocks CTR-008 (Term 1 waiting) |

### Phase 3 — Remaining Docs (parallel, no dependencies between them)

| CN | Depends on |
|-----|-----------|
| CN-4-003 | CN-4-002 |
| CN-4-004 | CN-4-001, CN-4-002 |
| CN-4-008 | CN-4-007 |
| CN-4-009 | CN-4-001, CN-4-002 |
| CN-4-010 | CN-4-001 |
| CN-4-012 | CN-4-011; carries CTR-017 |
| CN-4-013 | CN-4-022 |
| CN-4-014 | Standalone |
| CN-4-015 | Standalone |
| CN-4-016 | CN-4-006, CN-4-007 |
| CN-4-017 | Standalone |
| CN-4-018 | CN-4-010 |
| CN-4-019 | CN-4-023 |

**Flexibility:** CN-4-021 may be pulled earlier (once CN-4-011 is done) if Terms 5/6 are blocked on CTR-001/003. The Phase 2 ordering is a default, not a constraint — unblocking other Terms takes priority over sequential neatness.

---

## Cross-Term Summary

| CN doc(s) | Pending CTR(s) | Waiting Term(s) |
|-----------|----------------|-----------------|
| CN-4-021 | CTR-001, CTR-003 | Terms 5, 6 |
| CN-4-022 | CTR-007, CTR-014 | Terms 5, 2, 3, 1 |
| CN-4-023 | CTR-008 | Term 1 |
| CN-4-020 | CTR-018 | Term 6 |
| CN-4-012 | CTR-017 | Term 7 |
| CN-4-006, CN-4-007 | CTR-016 | Term 1 |

Six pending CTRs across nine concept documents. These parts of the catalogue are not final until the receiving Terms accept.

---

*— End of Section 6 —*
