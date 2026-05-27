# CN-4-015 — Compliance DSL Framework

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-007 #8 (sandboxed evaluator + reviewed extensions), D-009 (freeze doctrine — pack version at emission), Law 5 (compliance is configured, not coded).
> **Glossary:** See `MASTER-GLOSSARY.md`.
> **Depends on:** CN-4-002 (Event Store Contract — pack_version_ref), CN-4-004 (Command Bus — compliance policy evaluation), CN-4-006 (Isolation — tenant↔jurisdiction binding), CN-4-012 (Document Engine — pack version at issuance), CN-4-014 (Time Authority — business date, clock protocol), CN-4-023 (Kernel Boundary — doctrine gate for extensions).
> **Boundaries:** Command bus policy evaluation → CN-4-004; document numbering format → CN-4-012; specific pack content → Term 1 (approval) / Term 2 (input) / Term 7 (validation); reporting consumption → Terms 1/5; tax API submission → Term 7.

---

## 1. What This Doc Defines

CN-4-015 defines the language compliance packs are written in, the sandboxed evaluator that runs them, the reviewed extension mechanism, how pack versioning supports the freeze doctrine, the pack approval gate, and how a tenant is bound to a jurisdiction's pack.

This doc enables Law 5: "compliance is configured, not coded." Adding a country or changing a tax rate is a pack change, not a code change.

---

## 2. What the Compliance DSL Is

The DSL is a **domain-specific language** for expressing compliance rules — tax rates, document format requirements, reporting rules, legal constraints per jurisdiction.

| Property | What it means |
|----------|---------------|
| **Declarative** | A compliance pack declares rules ("VAT standard rate = 18%", "invoice must include TIN"), not procedures. |
| **Sandboxed** | Rules execute in a sandboxed evaluator with no access to system internals, no side effects, no network calls, no ambient time, no randomness (§4). |
| **Versioned** | Every pack has a version. Events and documents reference the pack version at emission/issuance (D-009). Old versions remain available for replay and verification. |
| **Neutral** | The DSL does not name any specific country. The same grammar expresses rules for any jurisdiction. The Kernel does not know which jurisdictions exist. |

---

## 3. Pack Structure

A compliance pack contains:

| Component | What it declares |
|-----------|-----------------|
| **Tax treatments** | Named tax categories with rates and rules (e.g., `vat-standard: 18%`, `vat-exempt: 0%`). Referenced by `tax_treatment_ref` on Saleable Lines (CN-4-021). |
| **Document rules** | Requirements per document type (e.g., "invoice must include: seller TIN, buyer TIN, sequential number, tax breakdown"). |
| **Number format** | Format string for document numbering per document type (e.g., `INV-{YEAR}-{SEQ:5}`). Referenced by the Numbering Engine (CN-4-012). |
| **Reporting rules** | What must be reported, when, in what format. (Terms 1/5 consume these.) |
| **Constraint rules** | Business constraints imposed by law (e.g., "credit sale above threshold requires buyer identification"). |

### What a Pack Does NOT Contain

- Engine logic (how an accounting journal is posted) — Terms 5/6
- UI/display rules (how a dashboard looks) — Term 3
- Platform policy (agent rules, pricing) — Term 1
- Integration details (how to submit to a tax API) — Term 7

---

## 4. The Sandboxed Evaluator (D-007 #8)

The evaluator runs compliance rules against commands and data. It is sandboxed — rules cannot escape.

| Constraint | What it prevents |
|-----------|-----------------|
| **No side effects** | A rule cannot write events, modify state, or call external services. It reads input, produces a result (accept/value/reject). |
| **No system access** | A rule cannot read the event store, query projections, or access Kernel internals. It receives only the data passed to it by the command bus (CN-4-004). |
| **No network** | A rule cannot make HTTP calls, DNS lookups, or any external communication. |
| **No ambient time** | A rule cannot call `datetime.now()` or equivalent. Time is received as input from the clock protocol (CN-4-014). Rules evaluate against the business date provided to them, not the wall clock. |
| **No randomness** | A rule cannot generate random values. All outputs are deterministic from inputs. |
| **Deterministic** | Same input + same rule → same result, always. Required for replay (CN-4-009) and command bus policy evaluation (CN-4-004 §3). |
| **Resource-bounded** | Execution time and memory are capped. A malformed rule cannot hang the system. (Limits are Architect-phase.) |

---

## 5. Tenant↔Jurisdiction Binding

### How a Command Selects Its Pack

A tenant is bound to a jurisdiction's compliance pack at registration — set by the regional agent during onboarding (Law 6: distribution is regional; the agent is compliance-accountable).

| Step | What happens |
|------|-------------|
| **1. Registration** | The regional agent registers the tenant with a jurisdiction (e.g., "this tenant operates under Tanzania's rules"). Stored as a tenant property event (CN-4-006). |
| **2. Pack resolution** | When the command bus processes a command for this tenant, it resolves: tenant → jurisdiction → active pack version for that jurisdiction at this moment. |
| **3. Evaluation** | The resolved pack version's rules are evaluated against the command. |
| **4. Recording** | The resulting event carries `pack_version_ref` = the resolved pack version (CN-4-002, D-009). |

### Jurisdiction Change

If a tenant changes jurisdiction (rare — e.g., business relocates), a new tenant property event records the change. Events before the change use the old jurisdiction's pack; events after use the new one. Freeze doctrine applies.

---

## 6. Pack Versioning & Freeze (D-009)

### Versioning

| Rule | What it means |
|------|---------------|
| **Every pack has a version** | Identified by jurisdiction + version (e.g., `tz-compliance-2026.03`). |
| **Events reference pack version at emission** | `pack_version_ref` (CN-4-002) records which pack was active. |
| **Documents reference pack version at issuance** | (CN-4-012 §2). |
| **Old versions are retained** | Replay and document verification use historical pack versions. Never deleted. |

### Pack Version Is Authoritative for Freeze

The **pack version active at emission** is the authoritative reference for the freeze doctrine (D-009). A rule's `effective-from` date within a pack is scheduling of a known-in-advance change within an already-deployed pack — it is resolved against the transaction's **business date** (CN-4-014 §3), not record time.

**Precedence is clear:**
1. The pack version active at emission time determines which pack is used.
2. Within that pack, rules with `effective-from` dates are evaluated against the business date of the transaction.
3. Both pack version and business date are stored in the event — replay is deterministic.

### Buggy Pack Correction (Section 11, N5)

A pack is later found to contain a bug. D-009 says events are frozen under historical rules. Resolution:

1. A corrective pack version is issued (e.g., `tz-compliance-2026.03-fix1`).
2. Affected transactions get **compensating events** (CN-4-001) under the corrected rules.
3. Historical events still stand under the original (buggy) pack version for audit purposes — they are not reinterpreted.
4. New events going forward use the corrected pack version.

The audit trail is complete: original events + pack version + compensating events + corrected pack version. No history is rewritten.

---

## 7. Extension Mechanism (D-007 #8)

### The Problem

Some jurisdictions have rules the core DSL grammar cannot express. Pure DSL extension (new grammar) is slow and careful. No extension mechanism at all means "sorry, this country can't use BOS."

### The Solution: Reviewed, Sandboxed Extensions

| Property | Detail |
|----------|--------|
| **What it is** | A plugin function that runs inside the same sandbox as DSL rules. It can express logic the grammar cannot. |
| **Extensions are CODE** | Unlike packs (which are data/configuration), extensions are executable code. They pass through the **doctrine gate** (CN-4-023) because they are code. Packs that use extensions pass through the **pack approval gate** (§8). |
| **Same sandbox constraints** | No side effects, no system access, no network, no ambient time, no randomness, deterministic, resource-bounded. Extensions are not a backdoor. |
| **Version-frozen (D-009)** | Extension versions are frozen like pack versions. A pack references a specific extension version. Replay of historical events uses the historical extension version — not the current one. |
| **Guarded against overuse** | Doctrine checks (CN-4-019) can flag packs that use extensions excessively. Extensions should be the exception, not the pattern. If many packs need the same extension, the core grammar should be extended instead. |

### Extension vs Grammar Update

| Situation | Resolution |
|-----------|-----------|
| One jurisdiction needs a unique rule | Extension — specific to that pack, version-frozen |
| Multiple jurisdictions need the same capability | Grammar update — extend the core DSL; review + test + doctrine gate (CN-4-023) |

---

## 8. The Pack Approval Gate

CN-4-023 §3 established two distinct gates. The pack approval gate is the second:

| Property | Detail |
|----------|--------|
| **What it gates** | Changes to compliance packs (new packs, updated rules, new effective dates). |
| **Who approves** | Term 1 Compliance Officer — human review required. |
| **What is checked** | DSL syntax validity, sandbox compliance (no side effects, no ambient time, no randomness), determinism, regression tests against known scenarios. |
| **D-009 sensitivity** | Pack changes are freeze-sensitive — they affect how future events/documents are interpreted. The reviewer must understand the impact. |
| **Separate from doctrine gate** | This gate does not run the full doctrine-check suite (CN-4-019). It runs pack-specific validation. Code changes (including extensions) use the doctrine gate; pack data changes use this gate. |

---

## 9. Three Whys

### Why does this matter?

Law 5 says "compliance is configured, not coded." Without a DSL, every tax rule, every document requirement, every reporting format is an `if/else` block in engine code. Adding a new jurisdiction means code changes. Changing a tax rate means a deployment. The DSL makes compliance a data problem — change a pack, not a codebase.

### Why does it belong in the Kernel?

If compliance evaluation were per-engine, each engine would interpret rules differently. Tax computed by one engine would differ from tax computed by another for the same rate. The Kernel provides one evaluator so compliance results are consistent across every engine, every command, every document.

### Why this design?

A declarative DSL with a sandboxed evaluator + reviewed/version-frozen extensions balances expressiveness with safety. The DSL handles the common cases (rates, formats, thresholds). Extensions handle the rare exceptions — as code through the doctrine gate, version-frozen for replay. The sandbox prevents rules and extensions from becoming backdoors. Pack versioning + freeze ensures historical compliance is preserved. Tenant↔jurisdiction binding ensures the right pack is always selected.

---

## 10. Example — A New Tax Is Introduced

A jurisdiction introduces a new digital services levy. The compliance team writes a new rule in that jurisdiction's pack:

```
tax-treatment "digital-services-levy":
  rate: 5%
  applies-to: items tagged "digital-service"
  effective-from: 2027-01-01
```

1. The rule is declarative — rate, applicability, effective date.
2. The pack is submitted through the **pack approval gate**. The Compliance Officer (Term 1) reviews it.
3. Syntax validation passes. Sandbox compliance passes (no side effects, no ambient time, deterministic). Regression tests pass.
4. The pack is deployed as a new version.
5. From January 1, commands involving digital-service items (matched by business date ≥ 2027-01-01) are evaluated against this rule. The `pack_version_ref` on resulting events records the new pack version.
6. Events before January 1 are unaffected — they were emitted under the previous pack version. Freeze doctrine holds.

**No Kernel code changed. No engine code changed. No deployment of BOS itself.** A compliance pack was configured, reviewed, and activated.

---

## 11. Boundaries

| Topic | Lives in |
|-------|----------|
| Command bus (compliance policies evaluated here) | CN-4-004 |
| Event envelope (pack_version_ref) | CN-4-002 |
| Document engine (pack version at issuance, number format) | CN-4-012 |
| Time authority (business date, no ambient time) | CN-4-014 |
| Freeze doctrine | D-009 |
| Doctrine gate (code changes, including extensions) | CN-4-023 |
| Tenant registration / jurisdiction binding | CN-4-006 |
| Specific compliance packs (tax rules per country) | Term 1 (approval), Term 2 (input), Term 7 (validation) |
| Reporting consumption (what is reported, when) | Terms 1/5 |
| Tax API submission (external integration) | Term 7 |

---

## 12. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| DSL grammar specification (BNF/PEG) | Architect phase | Concept defines what the DSL expresses; Architect specifies the grammar |
| Evaluator implementation (interpreter, compiler, hybrid) | Architect phase | Concept says "sandboxed, deterministic, resource-bounded"; Architect decides mechanism |
| Extension review process (who reviews, what criteria beyond doctrine gate) | Term 1 + Architect | Conceptual rules are here; process detail is governance (Term 1) + implementation (Architect) |
| Resource limits for evaluation (time, memory caps) | Architect phase | Concept says "resource-bounded"; specific limits are Architect's |
| Pack regression test framework (how known scenarios are encoded) | Architect phase | Concept says "regression tests against known scenarios"; Architect designs the framework |
| Pack hot-reload (update without system restart) | Architect phase | CN-4-004 references this; mechanism is Architect's |
| Extension version storage and retrieval during replay | Architect phase | Concept says "version-frozen like packs"; Architect designs storage |

---

*— End of CN-4-015 —*
