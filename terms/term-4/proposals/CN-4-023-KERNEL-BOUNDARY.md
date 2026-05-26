# CN-4-023 — Kernel Boundary & Doctrine Gate

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review. Completes Phase 2.
> **Governing decisions:** D-002B (Developer-AI outside the kernel).
> **CTRs:** CTR-008 (us → Term 1, Developer-AI governance — open).
> **Glossary:** See `MASTER-GLOSSARY.md` — Developer-AI, Kernel.
> **Depends on:** CN-4-001 (Doctrine), CN-4-006 (Isolation), CN-4-008 (Audit Log), CN-4-009 (Replay), CN-4-015 (Compliance DSL), CN-4-019 (Doctrine Enforcement Checks).
> **Boundaries:** Doctrine tests themselves → CN-4-019; Developer-AI governance → Term 1 (CTR-008); runtime AI → CN-4-022; engine registration gate → CN-4-020; compliance-pack gate → CN-4-015 + Term 1; CI/CD pipeline → Architect phase.

---

## 1. What This Doc Defines

This doc has two halves:

- **Part A — Kernel Boundary:** what is inside the Kernel (runtime, production, doctrine-enforced) and what is outside (Developer-AI, build-time, code-proposals-only). The boundary is structural — not a policy.
- **Part B — Doctrine Gate:** the automated checkpoint every proposed change must pass before release. The gate is the wall the doctrine tests (CN-4-019) form.

---

## 2. Part A — Kernel Boundary Declaration

### What Is Inside the Kernel

Everything Foundation designs: event store, command bus, primitives, isolation, identity, audit, replay, projections, AI guardrails, Advisor Framework, document engine, time authority, compliance DSL, resilience modes, extension points. These are **runtime components** — they run in production, process business events, and enforce doctrine.

### What Is Outside the Kernel

**Developer-AI** (D-002B) — the build-time environment that helps evolve BOS itself. It:

- Maintains understanding of the codebase
- Receives instructions from the Concept Lead (human)
- Briefs the implementer (Claude Code) to make changes
- Produces code proposals (pull requests), never runtime state

### The Hard Boundary (Non-Negotiable)

| Rule | What it means |
|------|---------------|
| **Developer-AI never runs inside the production kernel** | It does not process business events, does not read tenant data at runtime, does not participate in the command bus. |
| **Developer-AI never writes events or business state** | It produces code proposals — source files, docs, test files. Never event-store entries. |
| **Every change passes human review + CI** | A code proposal becomes part of the Kernel only after a human approves the PR and the doctrine gate passes. No autonomous merge. |
| **Developer-AI is not an advisor** | Advisors (CN-4-022) are runtime, tenant-scoped, journaled. Developer-AI is build-time, code-scoped, PR-tracked. They share no infrastructure, no scope, no journal. |
| **Developer-AI proposals receive no special treatment** | A PR from Developer-AI goes through the same gate, the same review, the same checks as a PR from a human developer. There is no fast track. |
| **Test data is synthetic or anonymised** | Developer-AI and its test pipeline use synthetic or anonymised fixtures, never live tenant data. This protects tenant isolation (CN-4-006) at the build layer. |

### Why This Boundary Matters

If Developer-AI ran inside the kernel, it could — by accident or exploit — modify runtime state, bypass doctrine, or leak tenant data through code changes. The boundary makes this structurally impossible: Developer-AI's output is code (reviewed by humans, tested by CI), never runtime actions.

This is how Law 3 survives at the build layer: the runtime is never modified by an autonomous agent without a human-gated, test-gated pipeline.

---

## 3. Part B — The Doctrine-Enforcement Gate

### What the Gate Is

The doctrine gate is the automated checkpoint every proposed **code change** to the Kernel must pass before release. It is the mechanical enforcement of Section 8 (Success Criteria) — every criterion expressible as an automated test runs here.

### Two Distinct Gates

BOS has **two separate gates** for two types of change. They are parallel, not nested. This doc defines the first; the second belongs to CN-4-015 and Term 1.

| Gate | What it gates | Defined in |
|------|--------------|-----------|
| **Doctrine gate** (this doc) | Code changes to the Kernel and engines | CN-4-023 + CN-4-019 |
| **Pack approval gate** | Compliance-pack changes (tax rules, document rules, regional law) | CN-4-015 + Term 1 (Compliance Officer). D-009-sensitive — pack changes affect the freeze doctrine. |

The two gates exist because the concerns differ: code changes risk doctrine violations; pack changes risk compliance errors. Mixing them would either over-constrain packs (forcing full CI on a tax-rate update) or under-constrain code (applying pack-level review to Kernel logic).

### When the Doctrine Gate Runs

| Trigger | Context |
|---------|---------|
| **Every pull request** | Before merge, the gate runs against the proposed change |
| **Every CI build** | On every commit to a protected branch |
| **Engine registration** (CN-4-020) | Build-time gate runs before runtime registration |
| **Developer-AI proposals** (D-002B) | The gate is the test-suite-as-gate; same gate as human PRs |

### What the Gate Checks

The gate runs the **doctrine enforcement checks** (CN-4-019). At concept level:

| Category | What is checked |
|----------|----------------|
| **Event doctrine** | Append-only guarantee holds; no UPDATE/DELETE paths; compensating fold produces correct state |
| **Isolation** | Cross-tenant leak-detection passes; cross-engine import detection passes; snapshot isolation holds |
| **Identity** | Every event carries a valid actor; dual-actor records present when scopes cross |
| **Advisory-only** | No advisor code path writes events or creates commands autonomously |
| **Time** | No `datetime.now()` in engine or primitive logic; all time from clock protocol |
| **Scope** | Advisor scope guard covers all inputs; platform scope is read-only; consent re-evaluated |
| **Freeze** | Events reference pack version at emission; documents reference pack + template version at issuance |
| **Snapshots** | Non-truth marker present; no code treats snapshots as source of state |
| **Engine contracts** | Manifests valid; event schemas conform; no undeclared event emissions |

### Gate Behaviour

| Result | What happens |
|--------|-------------|
| **All checks pass** | Change may proceed to merge (after human approval) |
| **Any check fails** | Change is **blocked**. The failing check is reported. The change cannot merge until the failure is resolved. |
| **Gate cannot run** (CI unavailable, infrastructure failure) | Change is **blocked**. The gate is **fail-closed**: absence of a passing gate = failure, never a pass. No change merges without a completed, passing gate run. |

### No Override

The gate has **no bypass, no override, no emergency skip**. If a change fails a doctrine check, the change must be fixed — not the gate. This is deliberate: an override mechanism would become the default under pressure. The gate's value comes from its absoluteness.

### Doctrine-Check Updates

If a doctrine check is found to be wrong (false positive, overly strict, or missing a case), the check itself is updated through a **heightened process**:

| Aspect | Requirement |
|--------|------------|
| **Approval authority** | Explicit approval from the Concept Lead or designated governance authority — not any reviewer. |
| **Change type** | Recorded as a **doctrine-affecting change** — a distinct category in the audit trail. |
| **Gate applies** | The check-update PR itself passes all other doctrine checks. The gate gates its own evolution. |
| **Audit** | The change is auditable: who proposed, who approved, what was changed, why. Legal defensibility of the wall itself. |

This prevents the wall from eroding silently. Every weakening of a doctrine check is visible, authorised, and traceable.

---

## 4. Build-Provenance Attestation

Every release that passes the doctrine gate produces an **attestation record** — proof of what was checked and by whom.

### Attestation Fields

| Field | Description |
|-------|-------------|
| **commit_hash** | The exact code version released |
| **checks_run** | List of every doctrine check that was executed |
| **check_results** | Pass/fail for each check |
| **approver** | The human who approved the PR |
| **timestamp** | When the gate completed (clock protocol) |
| **gate_version** | Version of the doctrine-check suite used |

### Storage and Retention

The attestation record is stored immutably — it may mirror the hash-chain / audit log pattern (CN-4-008) so that the Kernel's own build lineage is auditable and tamper-evident. The Kernel's provenance is as defensible as its business data.

Storage mechanism is an Architect-phase decision. Retention policy is Term 1's responsibility (governance). The concept guarantees: an attestation exists for every release, it is immutable, and it is queryable.

---

## 5. Relationship Between CN-4-019 and CN-4-023

| Doc | What it defines |
|-----|----------------|
| **CN-4-019** (Doctrine Enforcement Checks) | **The tests themselves** — what invariants exist, what each test checks, how they map to Section 8 criteria |
| **CN-4-023** (this doc) | **When and why the tests run** — the gate concept, the boundary declaration, the no-override rule, the fail-closed rule, the pipeline integration, the attestation |

CN-4-019 is "what we test." CN-4-023 is "the wall the tests form."

---

## 6. Three Whys

### Why does this matter?

Without a boundary, Developer-AI could propose changes that bypass doctrine — intentionally or through subtle bugs. Without a gate, a developer under deadline pressure could merge a change that breaks isolation, skips audit fields, or introduces `datetime.now()`. The boundary and gate together ensure that BOS's guarantees hold regardless of who proposes a change or how urgently it's needed.

### Why does it belong in the Kernel?

The boundary declaration is a Kernel concept because it defines what the Kernel IS (runtime, production, doctrine-enforced) versus what it is NOT (build-time tooling, code proposals). The gate is a Kernel concept because it enforces the Kernel's own invariants — no external authority can override them.

### Why this design?

A no-override, fail-closed gate is extreme by industry standards, where emergency bypasses are common. BOS chooses this because its core promise is legal defensibility — "prove to a court that nothing was tampered with." A gate with an override is a gate that can be opened. A gate without an override is a wall. BOS needs a wall. The build-provenance attestation extends that defensibility to the Kernel's own evolution — not just "the data is sound" but "the code that processes the data is sound, and here is the proof."

---

## 7. Example — Developer-AI Proposes an Optimisation

**The Concept Lead instructs Developer-AI:** "The projection rebuild for large tenants is slow. Propose an optimisation."

1. Developer-AI analyses the codebase, understands the replay engine (CN-4-009), and proposes a change: add a checkpoint mechanism that allows rebuilds to resume from a saved point instead of replaying from the beginning.

2. Developer-AI (via Claude Code) creates a pull request. The PR contains code changes and new test fixtures — all synthetic, no tenant data.

3. **The doctrine gate runs** (same gate as any human PR — no special treatment):
   - Append-only: ✓ (checkpoints are additive, not mutations)
   - Isolation: ✓ (checkpoints are per-tenant)
   - Time: ✓ (no `datetime.now()`)
   - Replay determinism: ✓ (rebuild from checkpoint produces the same state as full rebuild)
   - All other invariants: ✓

4. **Human reviews the PR.** The Concept Lead (or a designated reviewer) reads the change, confirms it makes sense, approves.

5. The change merges. **Attestation record** created: commit hash, checks run (all pass), approver (Concept Lead), timestamp, gate version.

6. The Kernel version advances.

**What if the optimisation accidentally broke isolation?** Say the checkpoint code shared a cache across tenants. The gate's cross-tenant leak-detection test would **fail**. The PR cannot merge. Developer-AI is notified. It proposes a fix. The gate runs again. Only when all checks pass and a human approves does the change enter the Kernel.

**What if CI was down?** The gate is fail-closed. The PR sits unmerged until CI recovers and the gate runs successfully. No emergency bypass.

At no point did Developer-AI touch runtime state, read tenant data, or bypass the gate.

---

## 8. CTR-008 — Developer-AI Governance

This doc declares the boundary and the gate. **Term 1 governs the environment around it:**

### CTR-008 — Developer-AI governance + model registry
- **From Term:** 4
- **To Term(s):** 1
- **Decision / Topic:** D-002B / Developer-AI (build-time)
- **Boundary Object:** BO-3
- **What is needed:** Platform governance for: (a) who may trigger Developer-AI, (b) who approves changes/releases, (c) the model registry for build-time AI, (d) audit of build actions.
- **Why:** Build-time AI must never modify the runtime without human + test gates; governance lives with Platform Stewards.
- **Proposed contract:** Developer-AI runs outside the kernel (this doc declares it); Claude Code implements via PR; release only after human review + CI doctrine gate (this doc defines it); Term 1 owns trigger authority, approval authority, model registry, and build-action audit. Attestation retention policy owned by Term 1.
- **Status:** OPEN
- **Resolution:** —

---

## 9. Boundaries

| Topic | Lives in |
|-------|----------|
| The doctrine enforcement tests themselves (what invariants, what each checks) | CN-4-019 (Doctrine Enforcement Checks) |
| Developer-AI governance (who triggers, who approves, model registry, build audit, attestation retention) | Term 1 (CTR-008) |
| The Advisor Framework (runtime AI — different from Developer-AI) | CN-4-022 |
| Engine registration build-time gate | CN-4-020 (Extension Points) |
| Compliance-pack approval gate (parallel, separate from this gate) | CN-4-015 + Term 1 (Compliance Officer) |
| CI/CD pipeline design (how the gate integrates with tooling) | Architect phase |
| Attestation storage mechanism | Architect phase |
| Tenant data isolation at the build layer | CN-4-006 (synthetic/anonymised fixtures enforce this) |

---

## 10. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Developer-AI governance (trigger authority, approval authority, model registry, build audit) | Term 1 (CTR-008) | Foundation declares the boundary and gate; Term 1 governs the environment |
| CI/CD pipeline integration (how the gate is wired into tooling) | Architect phase | Foundation defines the gate concept; Architect implements the pipeline |
| Attestation storage mechanism (hash-chained, database, etc.) | Architect phase | Foundation says "immutable, queryable, tamper-evident"; Architect decides how |
| Attestation retention policy (how long, archival) | Term 1 | Governance concern — how long build provenance is kept |
| Whether the doctrine gate applies to infrastructure changes (deploy configs, scaling rules) | Architect phase | Concept gates code and engine registrations; infrastructure may need its own gate |

---

*— End of CN-4-023 —*
