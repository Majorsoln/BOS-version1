# CN-4-012 — Document Engine & Numbering

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-009 (freeze doctrine — pack + template version at issuance), D-007 #7 (verification split), D-007 #12 (DEGRADED numbering — block-reserved).
> **CTRs:** CTR-017 (us → Term 7, verification surface split — pending; includes integrity-vs-authenticity question).
> **Glossary:** See `MASTER-GLOSSARY.md` — Document (primitive, CN-4-011).
> **Depends on:** CN-4-001 (Doctrine — immutability), CN-4-002 (Event Store Contract), CN-4-003 (Hash Chain — event-level integrity), CN-4-004 (Command Bus — atomic issuance), CN-4-006 (Isolation — tenant-scoped numbering), CN-4-011 (Primitive Catalog — Document terminal fold).
> **Forward references:** CN-4-015 (Compliance DSL — number format, document rules).
> **Boundaries:** Document primitive → CN-4-011; event hash chain → CN-4-003; compliance pack rules → CN-4-015; verification public surface → Term 7 (CTR-017); specific templates → Terms 5/6; regional requirements → Term 1/2.

---

## 1. What This Doc Defines

CN-4-012 defines how BOS issues, numbers, and verifies documents. It covers three components:

- **Document Engine** — the builder that assembles, hashes, and issues documents
- **Numbering Engine** — fiscal-compliant, deterministic, branch/terminal-scoped numbering
- **Verification Logic** — the hash-check capability that proves a document's integrity

Together, these make every BOS document — invoice, receipt, purchase order, credit note — hash-verified, immutably numbered, and verifiable under the rules that existed at issuance.

---

## 2. Document Lifecycle

A document follows the Document primitive's terminal fold (CN-4-011): issued once, never modified.

| Phase | What happens |
|-------|-------------|
| **1. Template selection** | The engine selects a versioned template. The template defines the document's structure and layout. |
| **2. Content assembly** | The engine provides the content — line items, totals, party references, dates. Content comes from events/projections. |
| **3. Compliance binding** | The compliance-pack version active at issuance time is bound to the document (D-009 freeze). Tax computations, legal requirements, and format rules come from this pack version. |
| **4. Numbering** | The Numbering Engine assigns a fiscal-compliant, deterministic number (§3). |
| **5. Hash computation** | A document hash is computed over the **content + template-version reference + pack-version reference + document number** (§5). The hash covers the logical content and its version references, not rendered output. |
| **6. Issuance** | The document is emitted as an event (`<engine>.document.issued.v1`). The event carries: document content, template version, pack version, number, and the document hash. The document is now immutable. |

---

## 3. Numbering Engine

### Requirements

| Requirement | Why |
|-------------|-----|
| **Fiscal compliance** | Many jurisdictions require sequential, gap-free document numbers for tax documents. Numbers must be auditable. |
| **Deterministic** | Given the same sequence of issuance events, the same numbers are assigned. Replay-safe. |
| **Branch/terminal-scoped** | A business with multiple branches or POS terminals needs per-scope number sequences to avoid contention. |
| **Tenant-isolated** | Numbering sequences are per-tenant (CN-4-006). One tenant's numbers never interfere with another's. |
| **No pre-allocation of random ranges** | Numbers must be predictable and sequential within their scope (Brief §7). |

### How Numbering Works

| Property | Detail |
|----------|--------|
| **Scope** | Numbers are scoped to: tenant + document type + branch/terminal (if applicable). Each scope has its own independent sequence. |
| **Format** | Configurable per compliance pack (e.g., `INV-2026-0001`, `RCT-BR01-00042`). Foundation defines the numbering mechanism; the format is compliance-configured (Law 5, CN-4-015). |
| **Sequence** | Monotonically increasing within scope. Gap-free under normal operation. |
| **Assignment** | Number is assigned at issuance time, atomically with the document event emission (CN-4-004 atomic multi-event). |

### Numbering in DEGRADED Mode (D-007 #12)

When the system is in DEGRADED mode (CN-4-017), the central number allocator may be unreachable. Numbering must not halt.

| Mechanism | How it works |
|-----------|-------------|
| **Block reservation** | Each branch/terminal pre-reserves a block of numbers (e.g., numbers 1001–1100 for terminal T1). The block is assigned while in NORMAL mode, recorded as an event. |
| **DEGRADED issuance** | The terminal issues documents using numbers from its reserved block. Numbers are sequential within the block. |
| **Reconciliation** | When NORMAL mode resumes, used numbers are reconciled. Reconciliation is compliance-pack-driven: **gap-free jurisdictions** → the sequence continues from the last used number (no gaps); **gap-tolerant jurisdictions** → unused numbers in the block are marked as reserved-but-unused with an auditable reason (DEGRADED recovery). |

Block reservation is deterministic: the block assignment is an event; the terminal's use of numbers within the block is sequential; reconciliation is an event. Replay produces the same numbering.

---

## 4. Template Versioning

| Rule | What it means |
|------|---------------|
| **Templates are versioned** | A template has a version identifier. Changes create a new version. |
| **Documents reference their template version** | An issued document records which template version was used. |
| **Old documents are never re-rendered with new templates** | A document issued under template v1 stays as-is forever. If the template changes to v2, old documents are not retroactively reformatted. (D-009 freeze.) |
| **Template versions are retained** | Old template versions remain available — needed for verification and re-display of historical documents. Re-display in 2036 uses the template version referenced by the document to produce the original appearance. |

---

## 5. Document Hash & Verification Logic

### What Is Hashed

The document hash covers the **logical content and version references**: template-version reference, assembled content (all fields), compliance-pack-version reference, document number. The hash covers content + references, **not rendered output** (pixels, PDF bytes) — re-display is a rendering concern; integrity is a content concern.

### Two Layers of Integrity

| Layer | What it protects | How |
|-------|-----------------|-----|
| **Document hash** | The document's own content | Recompute hash from stored content + references; compare with stored hash. Enables **standalone/offline verification** — given just the document (e.g., a printed invoice with a QR code), verify it hasn't been altered. |
| **Event hash chain** (CN-4-003) | The event that carries the document | The document hash is stored **inside** the `document.issued.v1` event. The event is linked into the hash chain. Tampering with the document hash means tampering with the event, which breaks the chain. |

The two layers reinforce each other: the document hash enables offline verification (without access to the event store); the event chain prevents the document hash itself from being altered in the store.

### Integrity vs Authenticity

The document hash proves **integrity** — the content has not changed since issuance. It does not prove **authenticity** — that BOS (and not a forger) issued the document. A forger could construct a fake document with a matching hash.

**Authenticity** may require a **cryptographic signature**: BOS signs the document hash with a private key; a verifier checks the signature against BOS's public key. This proves the document was issued by BOS, not just that it is internally consistent.

Whether offline verification requires integrity-only (hash check) or integrity + authenticity (hash + signature) is a question for **CTR-017 (Term 7)**: Term 7 owns the external verification surface and must decide what level of proof the public portal / QR-scan / offline cache provides. Foundation provides the hash; the signature capability is flagged for Term 7.

---

## 6. Corrections and Cancellations (D-009)

A document cannot be modified after issuance (CN-4-011 terminal fold). Corrections follow D-009:

| Action | How it works |
|--------|-------------|
| **Correction** | A **new document** (e.g., credit note) is issued. It references the original document by number and hash. It has its own hash, number, template version, and pack version. The original is unchanged. |
| **Cancellation** | A **cancellation document** is issued, referencing the original. The original remains immutable. The cancellation records why and by whom. |
| **Voiding** | Same pattern — a new document referencing the original. No original is ever modified or deleted. |

The original + the correction/cancellation together tell the complete story. Both are independently verifiable by hash. Both carry their own numbering (corrections get their own numbers in their own document-type sequence).

---

## 7. Three Whys

### Why does this matter?

A document issued by BOS — an invoice, a receipt, a purchase order — may be presented to a tax authority, a bank, a court, or a supplier years after issuance. If the document can be silently altered, it has no legal weight. The document hash makes alteration detectable; the numbering makes gaps detectable; the template + pack versioning makes the document self-contained — verifiable under the rules that existed when it was issued, not today's rules.

### Why does it belong in the Kernel?

If document integrity were per-engine, each engine would implement its own hashing and numbering — inconsistently. One engine's documents would be verifiable; another's wouldn't. The Kernel provides one document engine so that every document in BOS — regardless of which engine issued it — carries the same integrity guarantees.

### Why this design?

Terminal fold (issue once, never modify) + hash (content + version references) + deterministic numbering + template/pack versioning gives the strongest guarantee: a document is frozen at issuance, verifiable forever, and self-contained. The two-layer integrity model (document hash + event chain) provides both standalone verification (offline, QR) and store-level tamper protection. The correction-as-new-document pattern preserves the complete audit trail.

---

## 8. Example — Mama Amina's Supplier Disputes an Invoice

Mama Amina issued purchase order PO-2026-0034 to a supplier three months ago. The supplier claims the PO stated 50 bags of cement at TZS 18,000 each. Mama Amina's records show 40 bags at TZS 20,000.

1. BOS retrieves document PO-2026-0034.
2. **Hash verification:** BOS recomputes the hash from the stored content + template-version reference + pack-version reference + number. It matches the stored hash. The document has not been altered since issuance.
3. The document shows: 40 bags at TZS 20,000 (total TZS 800,000), template version v3, compliance-pack tz-procurement-2026.01.
4. The document number PO-2026-0034 is sequential in Mama Amina's PO sequence — no gaps before or after.
5. The event carrying this document is part of the hash chain (CN-4-003) — the document hash inside the event is also chain-protected.

The dispute is resolved: the document is provably unaltered at both layers. If a correction had been made after issuance, it would exist as a separate document (e.g., PO-AMEND-2026-0034-001) referencing the original — both independently verifiable.

---

## 9. Boundaries

| Topic | Lives in |
|-------|----------|
| Document primitive (terminal fold, "issued" event) | CN-4-011 |
| Event store (document issuance events) | CN-4-002 |
| Event hash chain (event-level integrity) | CN-4-003 |
| Freeze doctrine (pack + template version at issuance) | D-009 |
| Command bus (atomic issuance with numbering) | CN-4-004 |
| Tenant isolation (numbering is per-tenant) | CN-4-006 |
| Compliance pack format rules / number format grammar | CN-4-015 + compliance packs |
| Verification public surface (QR, portal, offline, signature question) | Term 7 (CTR-017) |
| Specific document templates (invoice, receipt, PO designs) | Terms 5/6 |
| Regional document legal requirements | Term 1 (approval) / Term 2 (input) |

---

## 10. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Document hash algorithm (same as event chain or different?) | Architect phase | May differ; Architect decides |
| Template storage and versioning mechanism | Architect phase | Concept says "versioned, retained, referenced"; Architect designs storage |
| Number format grammar (how format strings are expressed in compliance packs) | CN-4-015 + Architect | Format is compliance-configured; grammar is DSL's concern |
| Block reservation size policy (how many numbers per block in DEGRADED) | Architect phase + Term 1 | Too small = runs out; too large = many reserved-but-unused |
| QR code content / offline verification payload | Term 7 (CTR-017) | Foundation provides the hash; Term 7 designs the QR payload |
| Integrity vs authenticity for offline verification (hash-only or hash + cryptographic signature?) | Term 7 (CTR-017) | Flagged: hash = integrity; authenticity may need signature. Term 7 decides for the external surface. |
| Long-term template rendering compatibility (re-display in 2036) | Architect phase | Concept says "template versions retained"; Architect ensures rendering engine can process old templates |

---

*— End of CN-4-012 —*
