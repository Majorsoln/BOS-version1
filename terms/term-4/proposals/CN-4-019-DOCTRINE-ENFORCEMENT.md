# CN-4-019 — Doctrine Enforcement Checks

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-007 #5 (runtime + CI enforcement), D-007 #10 (snapshots), D-009 (freeze doctrine).
> **Glossary:** See `MASTER-GLOSSARY.md`.
> **Depends on:** CN-4-023 (Kernel Boundary & Doctrine Gate — the wall these tests form), and every CN doc that contributes an invariant: CN-4-001, CN-4-002, CN-4-003, CN-4-005, CN-4-006, CN-4-007, CN-4-009, CN-4-011, CN-4-012, CN-4-013, CN-4-014, CN-4-015, CN-4-017, CN-4-020, CN-4-021, CN-4-022.
> **Boundaries:** The gate (when/why tests run, no-override, fail-closed, attestation) → CN-4-023; CI/CD tooling → Architect phase; compliance-pack validation → CN-4-015 + Term 1; engine business-logic tests → Terms 5/6.

---

## 1. What This Doc Defines

CN-4-023 defined **the wall** — when the tests run, the no-override rule, the fail-closed behaviour, the attestation. This doc defines **the tests themselves**: the specific checks, how they map to Section 8 criteria and the six Laws, and how the catalog is organised and maintained.

---

## 2. The Invariant Catalog

Every Section 8 success criterion expressible as an automated test becomes a doctrine check. The catalog also includes structural invariants that underpin those criteria. Organised by category.

### Event Doctrine

| Check ID | What it tests | Section 8 | Type | Checks against |
|----------|--------------|-----------|------|---------------|
| DC-001 | No UPDATE/DELETE paths exist in event store access code | — | Static | CN-4-001 Assertion 3, CN-4-002 §4 |
| DC-002 | Compensating fold produces correct state per primitive | #12 | Integration | CN-4-001 §3.1, D-007 #3, CN-4-011 |
| DC-003 | Events reference pack_version_ref at emission | #13 | Schema/manifest | D-009, CN-4-002 |
| DC-004 | Replay with later pack version still uses historical rules | #13 | Integration | D-009, CN-4-001 §3.2 |
| DC-034 | Additive-only schema versioning — changes to an existing event type are additive; breaking change = new type/version | — | Schema/manifest | CN-4-002 §3 |
| DC-035 | Stream-ordering integrity — per-stream sequence has no gaps; optimistic-concurrency conflicts are rejected | — | Integration | CN-4-002 §2, O1 |

### Isolation

| Check ID | What it tests | Section 8 | Type | Checks against |
|----------|--------------|-----------|------|---------------|
| DC-005 | Cross-tenant query returns zero data from other tenants | #5 | Integration | CN-4-006 §1 |
| DC-006 | Cross-tenant command is rejected | #5 | Integration | CN-4-006 §1 |
| DC-007 | Projection rebuild for tenant A processes zero events from tenant B | #5 | Integration | CN-4-006 §4 |
| DC-008 | Snapshot for tenant A contains zero data from tenant B | #5 | Integration | CN-4-006 §4 |
| DC-009 | Cross-engine imports detected and blocked | #9 | Static | CN-4-005 §2, D-007 #5 |
| DC-010 | No undeclared event emissions (event type must be in engine manifest) | #9 | Schema/manifest | CN-4-005 §5 |
| DC-011 | Platform-scope operations produce platform audit entries | #10 | Integration | CN-4-006 §2 |

### Identity

| Check ID | What it tests | Section 8 | Type | Checks against |
|----------|--------------|-----------|------|---------------|
| DC-012 | Every event carries a valid actor with principal type | — | Schema/manifest | CN-4-002, CN-4-007 |
| DC-013 | Dual-actor records present when scope crosses (platform admin in tenant scope) | #10 | Integration | CN-4-007 §3 |

### Advisory-Only

| Check ID | What it tests | Section 8 | Type | Checks against |
|----------|--------------|-----------|------|---------------|
| DC-014 | No advisor code path writes events | #15 | Static | CN-4-013 §2, Law 3 |
| DC-015 | No advisor code path creates commands autonomously (without human principal) | #15 | Static | CN-4-013 §2 |
| DC-016 | Advisor scope guard rejects out-of-scope reads | #11 | Integration | CN-4-022 §3 |
| DC-033 | Decision Journal completeness — every journal entry contains: advisor_principal (incl. model id/version), prompt_ref, data_ref, recommendation, human_decision | #6 | Schema/manifest | CN-4-013 §3 |
| DC-036 | Journal events emitted by system principal only — no `kernel.advisor.*.recorded` event has an advisor as the emitting principal | — | Schema/manifest | CN-4-013 §3 (B — advisor never writes; Kernel records on its behalf) |

### Time

| Check ID | What it tests | Section 8 | Type | Checks against |
|----------|--------------|-----------|------|---------------|
| DC-017 | No `datetime.now()` in engine or primitive logic | #8 | Static | CN-4-014, Charter doctrine |
| DC-018 | All time assignments come from clock protocol | #8 | Static | CN-4-014 |

### Scope & Consent

| Check ID | What it tests | Section 8 | Type | Checks against |
|----------|--------------|-----------|------|---------------|
| DC-019 | Advisor scope guard covers all inputs including prompt assembly | #11 | Integration | CN-4-022 §3 |
| DC-020 | Platform scope is read-only against tenant data | #10 | Integration | CN-4-006 §2 |
| DC-021 | Consent-gated command rejection (outreach without consent blocked) | #14 | Integration | CN-4-011, D-008 |
| DC-022 | source_tag on Saleable Line is not branched on by Checkout code | — | Static | CN-4-021 §2, Law 2 |

### Freeze & Documents

| Check ID | What it tests | Section 8 | Type | Checks against |
|----------|--------------|-----------|------|---------------|
| DC-023 | Documents reference pack + template version at issuance | #7 | Schema/manifest | D-009, CN-4-012 |
| DC-024 | Historical document verifies under issuance-era pack/template | #7 | Integration | D-009 |
| DC-037 | Currency invariant — single settlement = single currency | — | Integration | CN-4-021 §3 |

### Snapshots

| Check ID | What it tests | Section 8 | Type | Checks against |
|----------|--------------|-----------|------|---------------|
| DC-025 | Non-truth marker present on all snapshots | — | Static | D-007 #10, CN-4-018 |
| DC-026 | No code path treats a snapshot as source of state | — | Static | D-007 #10 |

### Engine Contracts

| Check ID | What it tests | Section 8 | Type | Checks against |
|----------|--------------|-----------|------|---------------|
| DC-027 | Engine manifests valid (all required fields present) | #1 | Schema/manifest | CN-4-005 |
| DC-028 | Event schemas conform to CN-4-002 envelope rules | #1 | Schema/manifest | CN-4-002 |
| DC-029 | New engine registration passes compatibility checks without Kernel modification | #1 | Integration | CN-4-020 |

### Extension & Flexibility

| Check ID | What it tests | Section 8 | Type | Checks against |
|----------|--------------|-----------|------|---------------|
| DC-030 | A compliance pack deploys without Kernel code change | #2 | Integration | CN-4-015, Law 5 |
| DC-031 | Hash chain verification runs end-to-end and passes | #3 | Integration | CN-4-003 |
| DC-032 | Projection rebuild completes while stale projections serve reads | #4 | Integration | CN-4-009, CN-4-017 |

**Total: 37 doctrine checks** covering all 15 Section 8 criteria plus structural invariants.

---

## 3. Check Types

| Type | How it works | Count | Examples |
|------|-------------|-------|---------|
| **Static analysis** | Scans code without running it. Detects forbidden patterns (imports, `datetime.now()`, snapshot-as-source, advisor-writes). Fast, runs on every commit. | 11 | DC-001, DC-009, DC-014, DC-015, DC-017, DC-018, DC-022, DC-025, DC-026, DC-034 (partial), DC-036 (partial) |
| **Integration test** | Runs the system with synthetic test data and verifies invariants hold. Detects runtime violations (cross-tenant leaks, scope guard failures, fold errors, ordering violations). | 17 | DC-002, DC-004, DC-005, DC-006, DC-007, DC-008, DC-011, DC-013, DC-016, DC-019, DC-020, DC-021, DC-024, DC-029, DC-030, DC-031, DC-032, DC-035, DC-037 |
| **Schema/manifest validation** | Validates declared schemas and manifests against contracts. Detects structural mismatches. | 9 | DC-003, DC-010, DC-012, DC-023, DC-027, DC-028, DC-033, DC-034 (partial), DC-036 (partial) |

All three types run as part of the doctrine gate (CN-4-023). The Architect decides the exact tooling (linters, test frameworks, validators); the concept defines what each check must verify.

---

## 4. How Checks Map to the Six Laws

| Law | Enforcing checks |
|-----|-----------------|
| **Law 1 — State from events only** | DC-001, DC-002, DC-003, DC-004, DC-025, DC-026, DC-034, DC-035 |
| **Law 2 — Engines isolated** | DC-009, DC-010, DC-022, DC-027, DC-028, DC-029 |
| **Law 3 — AI advisory only** | DC-014, DC-015, DC-016, DC-019, DC-033, DC-036 |
| **Law 4 — Flexibility** | DC-029, DC-030 |
| **Law 5 — Compliance configured** | DC-003, DC-004, DC-023, DC-024, DC-030 |
| **Law 6 — Distribution regional** | No direct doctrine check — see below |

**Law 6 gap:** Regional distribution is a business policy enforced by Term 1/2 (agent accountability, regional compliance packs), not a code invariant the Kernel can test. This is correct and deliberate — the Kernel does not know which regions exist (D-004 neutrality). If a testable Kernel-level invariant for Law 6 is identified in the future, it will be added through the living-catalog process.

---

## 5. Living Catalog

The doctrine check catalog is a **living list** — new checks may be added as new CN docs are written, new decisions are made, or new edge cases are discovered.

| Rule | What it means |
|------|---------------|
| **Adding a check** | A new check is added through the normal gated process (CN-4-023: PR, human review, all existing checks pass). No special authority needed. |
| **Modifying or removing a check** | Requires **Concept Lead or governance authority approval** (CN-4-023 §3: doctrine-affecting change). Audited as a doctrine-affecting change. The wall does not weaken silently. |
| **Check IDs are stable** | Once assigned, a DC-NNN ID is not reused. A retired check is marked deprecated, not reassigned. |

---

## 6. Three Whys

### Why does this matter?

Section 8 promises 15 success criteria. Without automated enforcement, these are aspirations — words in a document. The doctrine checks make them mechanical: if a criterion is violated, the build fails. The system cannot be released in a state that breaks its own promises.

### Why does it belong in the Kernel?

The checks enforce Kernel-level guarantees (isolation, immutability, advisory-only, determinism). If checks were per-engine, each engine would test only itself — cross-engine invariants (isolation, event-only communication) would go untested. The Kernel runs the checks because only the Kernel has the authority and the cross-cutting view to verify system-wide invariants.

### Why this design?

A numbered, categorised catalog with explicit mappings to Section 8 criteria and CN docs makes every check traceable — a regulator can ask "how do you enforce isolation?" and the answer is "DC-005 through DC-011, here are the test results from the build-provenance attestation (CN-4-023 §4)." The three check types cover different failure modes. The living-catalog rule ensures the checks evolve with the system without weakening.

---

## 7. Example — A Developer Introduces a Cross-Engine Import

A developer working on a procurement engine adds `from engines.accounting.journal import post_entry` — a direct import across engine boundaries.

1. The developer commits and pushes.
2. The doctrine gate (CN-4-023) triggers.
3. **DC-009 (cross-engine import detection)** runs static analysis. It finds `engines.accounting` imported from within `engines.procurement`.
4. **DC-009 fails.** The gate reports: "Cross-engine import detected: engines.procurement → engines.accounting.journal. Law 2 violation."
5. The PR cannot merge. The developer must remove the import and use the event-subscription pattern (CN-4-005 §3) instead.

The developer did not intend to break isolation — they wanted a utility function. But the doctrine check caught it before the violation reached production. No human reviewer needed to spot it (though human review also runs). The check is automated, deterministic, and has no override.

---

## 8. Boundaries

| Topic | Lives in |
|-------|----------|
| The gate (when checks run, no-override, fail-closed, attestation) | CN-4-023 |
| Each CN doc defines what its invariants ARE | CN-4-001 through CN-4-022 |
| CI/CD tooling (linters, test frameworks, pipeline config) | Architect phase |
| Compliance-pack validation (separate gate) | CN-4-015 + Term 1 |
| Engine-specific business logic tests | Terms 5/6 — doctrine checks test Kernel invariants, not engine business logic |

---

## 9. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Exact tooling for each check type (which linter, which test framework) | Architect phase | Concept defines what is checked; Architect decides how |
| Performance budget for the full check suite | Architect phase | The gate must not block developer velocity; speed is Architect's concern |
| Incremental vs full run (only affected checks on partial change) | Architect phase | Concept says "all checks must pass"; optimisation is Architect's |
| Test fixtures for integration checks (synthetic data sets) | Architect phase | Must be synthetic/anonymised (CN-4-023 §2) |
| **Audit completeness check** — "every state-changing action produces an audit entry" | Pending CN-4-008 | Will be added to the catalog through the living-catalog process once CN-4-008 defines the audit contract. |
| Law 6 enforcement (if any Kernel-testable aspect emerges) | Future decision | Currently no direct doctrine check; may change |

---

*— End of CN-4-019 —*
