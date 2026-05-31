# Deferred Discussions

> **Owner:** Overseer (Term 7 — Integration & Coherence)
> **Purpose:** Append-only log of topics raised by the Concept Lead during Term/CN discussions that are **out of immediate scope** but flagged for later detailed discussion. Each entry names the topic, the context in which it arose, the Terms/CN docs it will eventually touch, and a brief note on why it's deferred.
> **Discipline:** Entries are never removed. When a deferred topic is taken up as a formal CN doc, CTR, or decision, the entry is annotated with the link, but the entry itself stays — the audit trail of "this was raised on date X, picked up on date Y" is preserved.

---

## 2026-05-30 — Promotion Engine (CN-5-007) discussion

### D-DISC-001 — Tenant-customer promotion application and dashboard visibility
- **Raised during:** CN-5-007 (Promotion Engine) scoping discussion
- **Topic:** How tenant **customers** apply promotions at point of purchase (enter a voucher code? auto-detected by phone number? loyalty card scan?) and how they see their loyalty balance / available promotions / redemption history on **customer-facing surfaces**.
- **Why deferred:** CN-5-007 defines the engine mechanics (promotion definitions, application logic, cost-share, event contracts). The **customer-facing experience** is a **Term 3 (Tenant Experience)** concern that builds on the engine's read surface. The two layers must agree but are designed separately to honour D-006 (one Term per scope) and Brief §5 audience separation.
- **Terms/docs to involve when picked up:**
  - Term 3 (CN-3-* tenant-facing promotion UX)
  - Term 5 (CN-5-007 publishes the read surface; potentially CN-5-006 Reporting projections for customer-history queries)
  - Possibly Term 7 (channels for customer notification of available promotions, per D-008 + CTR-021)
- **Status:** Deferred — to be picked up when Term 3 starts work on tenant-customer-facing surfaces.

### D-DISC-002 — POS self-service expansion (per-business opt-in)
- **Raised during:** CN-5-007 (Promotion Engine) scoping discussion (broader Checkout/UX topic)
- **Topic:** A **self-service POS** capability where the end customer operates the POS terminal themselves (currently common in cafés, restaurants, hotels for kiosk ordering / self-checkout). Concept Lead notes this **started in cafés/restaurants/hotels** but the framework should **expand to any business that needs it** (pharmacy self-pickup, retail self-checkout, etc.).
- **Why deferred:** This is a substantial cross-term feature touching:
  - **Term 5** (CN-5-009 Checkout — self-service tender flow; possibly new commands like `checkout.self_service.start.request`; identity / cashier-vs-customer principal handling per CN-4-007)
  - **Term 6** (vertical-specific self-service flows — café kiosk vs hotel check-in kiosk differ operationally; same engine, different vertical wrappers)
  - **Term 3** (self-service UX patterns; on-screen customer guidance; consent capture)
  - **Term 4 / CN-4-007** (principal handling — customer as actor vs cashier as actor; audit trail; UI-04 origin chain extension)
  - **Term 7** (potentially — peripheral device integration: card readers, receipt printers, barcode scanners attached to kiosks)
- **Architectural framing (Overseer note):** Self-service is **NOT a new engine** — it is a **mode of operation** on existing engines (Checkout, Inventory deduction, Cash receipt). The doctrine that holds: every state change still passes through the command bus (CN-4-004); the actor field is the customer principal (CN-4-007); UI-04 origin chain still terminates in `checkout.settled.v1`; cost-share/promotion still applies. The work is mostly in (a) the customer-actor identity handling and (b) the vertical-specific self-service workflows, not new doctrine.
- **Status:** Deferred — to be picked up after Term 5 universal engines are complete and Term 6 begins (verticals are the natural home for "self-service per vertical").

---

## 2026-05-30 — General mechanism

Future deferred topics raised during any Term's CN discussion will be appended here under the date they were raised. The Overseer (this file's maintainer) is responsible for ensuring deferred topics are referenced in the **Open Items** section of any CN doc whose scope they intersect, so the topic is not lost when the relevant CN is written.

*— End of Deferred Discussions —*
