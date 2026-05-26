# CN-4-011 — Primitive Catalog

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-007 #2 (nine primitives, Consent≠Approval, Workflow≠Approval), D-007 #3 (compensating fold), D-007 #13 (Ledger has no close-period), D-008 (consent-first, opt-out), D-009 (freeze doctrine — documents immutable under issuance-era rules).
> **Glossary:** See `MASTER-GLOSSARY.md` — Party (≠ actor identity), Fold, Compensating Event.
> **Depends on:** CN-4-001 (Doctrine), CN-4-002 (Event Store Contract), CN-4-005 (Engine Contract Model), CN-4-006 (Isolation), CN-4-007 (Identity Primitive).
> **Boundaries:** Value shapes (Saleable Line, Tender) → CN-4-021; document engine mechanics → CN-4-012; compliance rules applied to primitives → CN-4-015; per-engine use of primitives → Terms 5/6.

---

## 1. What Primitives Are

Primitives are the Kernel's reusable building blocks — the materials every engine builds with. Each primitive defines a **shape** (data structure), a set of **legal operations**, and a **compensating-fold function** (D-007 #3). No primitive names a vertical, a country, or an integration (D-004).

Engines combine primitives to build business logic. The Kernel provides the materials; engines build the structures. A primitive does not know what structure it will be part of.

---

## 2. The Nine Primitives

### Ledger

| Aspect | Detail |
|--------|--------|
| **Purpose** | Double-entry building block. Records debits and credits. |
| **Knows** | Entries (debit/credit pairs, must balance), balances, account references |
| **Does NOT know** | Accounting rules, chart of accounts, period close (D-007 #13 — Term 5), tax computations |
| **Legal operations** | Post entry (debit + credit, must balance), query balance |
| **Fold** | Running balance = sum of all debit/credit entries including compensations. A compensating entry reverses the original entry's effect (debit→credit, credit→debit). |

### Item / Service

| Aspect | Detail |
|--------|--------|
| **Purpose** | The abstract thing being tracked, sold, or moved. |
| **Knows** | Identifier, description, unit of measure, status (active/inactive), categorisation tags |
| **Does NOT know** | Which vertical uses it, pricing (engine logic), supplier details (procurement) |
| **Legal operations** | Create, update attributes, deactivate, reactivate |
| **Fold** | Current state = latest attribute values after applying all change events including compensations. A compensating event rolls back to the prior attribute value. |

### Party

| Aspect | Detail |
|--------|--------|
| **Purpose** | A party that interacts with the business — customer, supplier, employee, or any external entity. |
| **Knows** | Identifier, name, contact references, type tags, relationship to tenant |
| **Does NOT know** | System identity or permissions (that is CN-4-007's actor model — a different concept), specific vertical context |
| **Legal operations** | Create, update attributes, link/unlink relationships |
| **Fold** | Current profile = latest attributes after all change events including compensations. |

**Naming note:** Charter §4.1 names this primitive "party." The Brief §3 used "actor," which created ambiguity with CN-4-007's actor identity model. "Party" is the Charter-aligned name. The MASTER-GLOSSARY records the distinction: **Party** = business-domain entity (customer, supplier); **Actor** = system identity on an event (human, advisor, system, machine).

### Document

| Aspect | Detail |
|--------|--------|
| **Purpose** | A hash-verified, immutable issued structure. |
| **Knows** | Content, template version, compliance-pack version at issuance (D-009), hash, document number, issuing actor, issuance timestamp |
| **Does NOT know** | What the document is for (invoice, receipt, PO — engine context), regional legal requirements (compliance packs) |
| **Legal operations** | Issue (creates immutable record), verify (checks hash) |
| **Fold** | Terminal fold — a document has a single event (`issued`) that is terminal. No subsequent event modifies the issued document; the fold is the issuance state itself. Cancellation or correction is a **new document** (with its own hash, number, and pack version) that references the original (D-009). The original document's stream contains `issued` only, forever. |

### Inventory Movement

| Aspect | Detail |
|--------|--------|
| **Purpose** | A record of something physical moving — in, out, transfer, adjustment. |
| **Knows** | Item reference, quantity, direction (in/out/transfer/adjustment), location references, movement timestamp |
| **Does NOT know** | Warehouse layout, FIFO/LIFO costing rules (engine logic), reorder points |
| **Legal operations** | Record movement (in, out, transfer, adjust) |
| **Fold** | Current quantity at location = sum of all movements (in positive, out negative) including compensations. A compensating movement reverses the quantity effect. |

### Obligation

| Aspect | Detail |
|--------|--------|
| **Purpose** | Something owed — payable, receivable, credit, deposit. |
| **Knows** | Amount, currency, debtor/creditor party references, status (open/partial/settled/voided), due reference |
| **Does NOT know** | Payment methods (checkout/tender), settlement logic, collection workflows |
| **Legal operations** | Create, partially settle, fully settle, void |
| **Fold** | Current status and outstanding amount = initial amount minus all settlement events plus compensations. A compensating event reverses a settlement or re-opens a voided obligation. |

### Approval

| Aspect | Detail |
|--------|--------|
| **Purpose** | A simple, auditable gate requiring human authorisation. |
| **Knows** | What is being approved (reference), who requested, who approved/rejected, decision, timestamp |
| **Does NOT know** | Multi-step workflows (that is Workflow), escalation rules, business-specific approval policies |
| **Legal operations** | Request, approve, reject |
| **Fold** | Current decision = the latest approval/rejection event. A compensating event re-opens the gate for re-decision. |

### Workflow

| Aspect | Detail |
|--------|--------|
| **Purpose** | A multi-step state machine moving through defined stages. |
| **Knows** | Current stage, defined stages and transitions, transition history, actors at each transition |
| **Does NOT know** | Which business process it models, what triggers transitions (engine logic) |
| **Legal operations** | Define stages, transition to next stage, revert to previous stage (if allowed by definition) |
| **Fold** | Current stage = result of applying all transition events in order. A compensating transition reverses a stage change. |

### Consent

| Aspect | Detail |
|--------|--------|
| **Purpose** | Explicit permission recorded immutably — data processing rights, terms acceptance, marketing opt-in. |
| **Knows** | Who consented (party reference), what they consented to (purpose + scope), when, consent status (granted/revoked) |
| **Does NOT know** | Privacy law specifics per jurisdiction (compliance packs), marketing campaign details |
| **Legal operations** | Grant consent, revoke consent (opt-out) |
| **Fold** | Current consent status = latest grant or revoke event per purpose+scope combination. Revoke always wins over prior grant — opt-out is honoured immediately (D-008). |

**Consent granularity:** Consent is keyed by **purpose + scope**. Channel is part of scope (e.g., "marketing via SMS to party X" is a distinct consent record from "marketing via email to party X"). This gives per-purpose and per-channel granularity without adding a separate dimension. Revocation at any granularity level is honoured immediately.

---

## 3. Primitive Distinctiveness

Two pairs require explicit distinction (D-007 #2):

**Consent ≠ Approval.** Both gate a decision. They differ in legal weight and lifecycle. Approval gates an operation (one-time gate, re-openable). Consent gates data-processing rights and carries data-law implications (ongoing, revocable at any time, revoke-wins). Merging them would either weaken consent's legal guarantees or over-complicate approval's simple gate.

**Workflow ≠ Approval.** Approval is a one-step gate (request → approve/reject). Workflow is a multi-step state machine (stage → stage → stage). Merging them adds state-machine complexity to the common one-step case. Approval can be a single stage within a Workflow, but Workflow is not needed for a simple approval.

---

## 4. Compensating-Fold Summary (D-007 #3)

Every primitive defines how compensation works. The universal rule (CN-4-001): truth = deterministic fold of all events including compensations.

| Primitive | Compensation pattern |
|-----------|---------------------|
| Ledger | Reversing entry (debit↔credit) |
| Item / Service | Attribute rollback to prior value |
| Party | Attribute rollback |
| Document | Terminal fold — no compensation of the document itself; corrections are new linked documents (D-009) |
| Inventory Movement | Reverse movement (in↔out) |
| Obligation | Re-open or reverse settlement |
| Approval | Re-open gate for re-decision |
| Workflow | Reverse stage transition |
| Consent | Re-grant or re-revoke (latest event wins per purpose+scope) |

---

## 5. Neutrality (D-004)

Every primitive is neutral by design:

- No primitive names a vertical. "Ledger" serves accounting, cash management, and any future engine that needs double-entry.
- No primitive names a country. "Consent" serves GDPR, Tanzania's data laws, and any future regulation.
- No primitive names an integration. "Obligation" serves any settlement method — the primitive does not know what settles it.

The rule from Section 1: whenever a primitive concept is tempted to mention a vertical, country, or integration — the abstraction is wrong.

---

## 6. Event Namespacing

Primitives are building blocks, not engines. They do **not** emit events under their own namespace. The engine that uses a primitive emits events under the engine's namespace (Charter §8.1):

- `accounting.journal.posted.v1` — not `ledger.entry.posted.v1`
- `procurement.order.approved.v1` — not `approval.decided.v1`

The Kernel (hash chain, replay, projections) processes all events regardless of namespace. Primitive integrity is preserved through the fold logic, not through event naming.

---

## 7. Three Whys

### Why does this matter?

Without shared primitives, every engine reinvents basic concepts — its own ledger, its own document structure, its own approval gate. The concepts drift apart. A "ledger entry" in one engine and a "ledger entry" in another become incompatible. Cross-engine operations (reporting, audit, compliance) become impossible because there is no common language for the building blocks of business.

### Why does it belong in the Kernel?

Primitives are the Kernel's materials. They enforce consistency across every engine that uses them. If primitives were per-engine, the Kernel could not guarantee that a "document" in one engine has the same integrity guarantees (hash-verified, immutable, freeze-doctrine-compliant) as a "document" in another. The guarantees must be uniform.

### Why this design?

Nine primitives cover the building blocks every business domain needs without over-abstracting. Each is small, focused, and composable. Engines combine primitives to build business logic. The alternative — one giant "business object" or a per-vertical primitive set — would be rigid, vertical-specific, and impossible to extend.

---

## 8. Example — How a Purchase Order Is Composed

*Engine names appear here only as illustrations of composition — the Kernel does not know which engines exist.*

A procurement engine needs to model a purchase order. It composes four Foundation primitives:

| Primitive used | Role in the purchase order |
|---------------|---------------------------|
| **Workflow** | The PO moves through stages: draft → submitted → approved → fulfilled → closed |
| **Obligation** | When approved, the PO creates an obligation — the business owes the supplier the agreed amount |
| **Document** | When approved, a hash-verified PO document is issued (immutable, numbered, pack-versioned) |
| **Item / Service** | Each line on the PO references an item/service from the catalog |

The procurement engine defines the business logic: what triggers each workflow transition, when the obligation is created, what the document template looks like. Foundation provides the building blocks; the engine provides the composition.

If the supplier disputes the PO:
- The **workflow** history shows who approved it and when.
- The **obligation** shows the exact amount owed and whether any payments were made.
- The **document** is hash-verified — proving the PO content has not changed since issuance.
- Each **item** on the PO is traceable to the catalog.

No engine other than procurement was involved. No Foundation primitive was modified. The composition used only the legal operations each primitive provides.

---

## 9. Boundaries

| Topic | Lives in |
|-------|----------|
| Saleable Line & Tender value shapes | CN-4-021 — value shapes, not full primitives |
| Document engine mechanics (template rendering, numbering allocator) | CN-4-012 |
| System identity (the actor field on events) | CN-4-007 — different concept from the Party primitive |
| Compliance rules applied to primitives (tax, document law) | CN-4-015 |
| Per-engine use of primitives (journal entries, POs, kitchen tickets) | Terms 5/6 |
| Full payload schemas (data contracts) | Architect phase |

---

## 10. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Full payload schema for each primitive | Architect phase | This doc defines concepts and shapes; the Architect specifies exact data contracts (frozen dataclasses, Section 9) |
| Whether primitives need a shared base interface or are fully independent types | Architect phase | The concept says "each primitive is an immutable dataclass"; the Architect decides whether a shared base adds value |
| Composition patterns (how engines combine primitives into business flows) | Terms 5/6 + Architect | Foundation provides materials; engines build structures; the PO example (§8) is illustrative, not prescriptive |
| Event type registry (how primitive-related events are catalogued) | CN-4-005 + CN-4-020 | Events are engine-namespaced; the global registry aggregates all engine manifests |

---

*— End of CN-4-011 —*
