# Term 7 — Integration & Coherence

> **Parent:** [BOS-CONCEPT-CHARTER.md](../BOS-CONCEPT-CHARTER.md) — read first
> **Type:** Coherence Term (Part C — Cross-Cutting)
> **Status:** Concept Phase v1.0

---

## 1. Mission

Six Concept Terms are doing focused work in their own domains. Each is doing it well — and that is the danger. Six Terms working in parallel can produce six brilliant, internally-consistent ideas that **don't fit together**.

Term 7 exists so that does not happen.

This Term owns two responsibilities:

**Responsibility A — Coherence.** Continuously check that the six Terms are speaking the same language. When Term 3 says "the cashier sees this," is what they see actually what Term 5 can produce? When Term 2 promises agents a tenant transfer flow, has Term 1 actually defined how transfers are arbitrated? When Term 6 adds a new vertical event, do the relevant Universal Engines in Term 5 know about it?

**Responsibility B — External Integration.** BOS does not live alone. It must talk to M-Pesa, Tigo Pesa, Airtel Money, banks, the Tanzania Revenue Authority, the Kenya Revenue Authority, payment gateways, SMS providers, courier services, and more. Every one of these connections is a place where the doctrine of "events are truth, AI is advisory, engines are isolated" must be honoured even though we don't control the other side. This Term designs the gateway through which BOS speaks to the outside world.

Term 7 is also the Term that takes on the role most aligned with this project's **logic-thinker role**: standing back from any one part to ask *"Does this whole thing make sense?"*

---

## 2. Mission Question

> **"Are all six Terms saying the same thing? And when BOS must speak to the world — M-Pesa, tax authorities, banks, SMS providers — how does it do so without compromising any of the laws this Charter rests on?"**

---

## 3. Scope (In)

This Term has **two distinct halves**.

### 3.1 Half A — Coherence Work

- **Charter Enforcement** — every concept doc from every Term is reviewed for Charter compliance
- **Cross-Term Conflict Detection** — when two Terms describe the same thing differently, this Term flags it
- **Open Question Triage** — the Open Questions sections of every Term are aggregated and routed
- **Doctrine Drift Watch** — when implementation/concept practice starts to drift from the Charter, this Term sounds the alarm
- **Glossary Authority** — when a term-of-art is used differently across Terms, this Term resolves it
- **Pattern Consistency** — when one Term invents a pattern (e.g., event naming, role inheritance), this Term ensures other Terms either adopt it or have a documented reason not to
- **The CHANGELOG** — every change to the Charter is recorded here
- **Decision Log** — when a cross-Term decision is made (e.g., "we will not support Global Agents"), it is recorded here

### 3.2 Half B — Integration Gateway

- **Integration Doctrine** — the philosophy of how BOS speaks to the outside (event-driven, never direct DB access, adapters as stateless translators)
- **Inbound Integration Framework** — external system → adapter → validate → translate → BOS command
- **Outbound Integration Framework** — BOS event → publisher → translate → external system
- **Adapter Catalog** — which external systems we connect to, by region and category
- **Idempotency & Reference Tracking** — never apply the same external event twice (e.g., M-Pesa callbacks can arrive multiple times)
- **Failure & Retry Policy** — what happens when M-Pesa is down, when TRA's API returns 500, when an SMS doesn't deliver
- **Authentication & Verification** — HMAC signatures, OAuth, API keys, certificate management
- **Integration Audit Log** — every external interaction recorded, append-only
- **Permissions Framework** — which integrations a tenant has access to, granted by Platform
- **Webhook Framework** — receiving notifications from external systems safely

### 3.3 Specific Integration Categories (sketches, not full designs)

The actual adapters are built when needed. But Term 7 defines categories and their patterns:

- **Mobile Money** — M-Pesa, Tigo Pesa, Airtel Money, MTN MoMo
- **Bank Payment** — direct bank transfers, statement reconciliation
- **Card Payment** — Visa/Mastercard via payment gateways
- **Tax Authority** — TRA (TZ), KRA (KE), URA (UG), and others; EFD/e-invoicing
- **SMS Gateway** — receipt SMS, notification SMS, OTP
- **Email** — transactional and notification email
- **Courier/Logistics** — for delivery-enabled tenants
- **Identity Verification** — KYC providers for agent or tenant onboarding
- **Document Repository** — external storage (S3-compatible) for issued documents (long-term retention)

---

## 4. Scope (Out)

This Term does NOT own:

| What | Owned By |
|------|----------|
| Decisions about *what* features exist | The other six Terms |
| Engine internals (Accounting, Inventory, Retail, etc.) | Term 5 / Term 6 |
| The Charter itself (Term 7 enforces it; it doesn't write it) | The Concept Lead with all Terms |
| UI for integration configuration | Term 1 (Platform Stewards) sets policy; Term 3 (Tenant Experience) handles tenant-facing config |
| Specific compliance pack content (e.g., TZ VAT rules) | Term 1 (approval); regional agents (input) |

**Important distinction:** Term 7 doesn't *decide what BOS does*. It checks that what other Terms decide *fits together* and *can talk to the outside world*. Term 7 is referee, not striker.

---

## 5. Audiences Served

Term 7's audience is **the other six Terms** and the **Architects who synthesise across them**.

| Audience | What they get from Term 7 |
|----------|---------------------------|
| All other Terms | Coherence reviews, conflict resolution, pattern guidance |
| Architects | A unified picture across Terms; integration contracts |
| Term 5 / Term 6 | Integration event contracts (inbound triggers, outbound emissions) |
| Term 1 | Integration permissions framework |
| Term 3 | What integration touchpoints look like to tenants (e.g., "M-Pesa payment failed" notification) |

End-users — tenants, agents, platform staff — are served only indirectly: they experience the *result* of Term 7's work (everything fits, integrations work).

---

## 6. Key Concepts This Term Will Produce

### 6.1 Coherence Concepts

| ID | Concept | What it answers |
|----|---------|-----------------|
| CN-7-001 | Coherence Review Process | How concept docs are reviewed for Charter and cross-Term consistency |
| CN-7-002 | Conflict Resolution Protocol | When two Terms disagree, how is it settled |
| CN-7-003 | Master Glossary | Single authoritative list of terms-of-art across all docs |
| CN-7-004 | Pattern Library | Patterns used across Terms (e.g., state machines, audit hooks, scope guards) |
| CN-7-005 | Charter Amendment Process | How Charter changes are proposed, evaluated, recorded |
| CN-7-006 | Open Questions Registry | Aggregated cross-Term open questions, status-tracked |
| CN-7-007 | Decision Log | Cross-Term decisions, with date, parties, and reasoning |
| CN-7-008 | Doctrine Drift Indicators | Signals that practice is drifting from doctrine |

### 6.2 Integration Concepts

| ID | Concept | What it answers |
|----|---------|-----------------|
| CN-7-100 | Integration Doctrine | The philosophy of external integration |
| CN-7-101 | Inbound Integration Framework | External → BOS, via adapters |
| CN-7-102 | Outbound Integration Framework | BOS → External, via publishers |
| CN-7-103 | Adapter Pattern & Lifecycle | What an adapter is, how it is built and retired |
| CN-7-104 | Idempotency & External Reference | Preventing duplicate processing |
| CN-7-105 | Failure, Retry, and Backoff Policy | What to do when external systems fail |
| CN-7-106 | Authentication & Verification | HMAC, OAuth, API keys, cert management |
| CN-7-107 | Integration Audit Log | Append-only log of every external interaction |
| CN-7-108 | Permissions for Integrations | Tenant access control to integrations |
| CN-7-109 | Webhook Framework | Receiving external notifications safely |
| CN-7-110 | Integration Category: Mobile Money | M-Pesa, Tigo Pesa, etc. — common patterns |
| CN-7-111 | Integration Category: Tax Authority | TRA, KRA, URA — common patterns |
| CN-7-112 | Integration Category: SMS & Email | Notification channels |
| CN-7-113 | Integration Category: Banks & Cards | Bank statement, card payment |
| CN-7-114 | Document Long-Term Retention | External storage for issued documents |

---

## 7. Edge Cases to Consider

### 7.1 Coherence Edge Cases

- Term 3 wants a field that doesn't exist in Term 5's event. Who adds it — Term 3 specs the request, Term 5 implements, Term 7 mediates the contract.
- Term 6 introduces a vertical concept (e.g., "kitchen station") that Term 1 hadn't anticipated in pricing. Does this require a new combo, new engine, both?
- Term 2 wants agents to see a "tenant health score." Term 3 says this could harm tenant trust if surfaced. How is this debated and resolved?
- Term 4's primitive (e.g., obligation) is being used differently by Term 5 (Accounting) and Term 6 (Restaurant). Is this fine, or a leak?
- The Charter says "AI is advisory only." Term 5 proposes an "auto-reorder" feature where AI triggers a purchase order if approved by a tenant policy. Is this still "advisory"? (Edge case — Term 7 referees.)

### 7.2 Integration Edge Cases

**Mobile Money**
- An M-Pesa callback says "TZS 50,000 received from 0712345678 ref ABC123." How is this matched to a specific sale, customer, or tenant? What if multiple matches are possible?
- A tenant's M-Pesa account is closed by Vodacom mid-day. What does the cashier see when trying to take an M-Pesa payment?
- M-Pesa is down for 4 hours. Sales continue with cash. When M-Pesa is back, the customer wants to pay via M-Pesa for a sale from 2 hours ago. Allowed? How represented?
- A customer claims they paid via M-Pesa but BOS has no record (callback was lost). Reconciliation flow?
- A refund via M-Pesa requires re-authorising the original payment reference. The original ref is from 6 months ago and may be archived. Recovery?

**Tax Authority (TRA-style)**
- TRA's e-invoice API requires every B2B invoice to be authorised before being given to the buyer. The API takes 30 seconds. The cashier needs to give the invoice now. Cache? Queue? Block?
- TRA changes its API spec without notice. Production breaks. Rollback?
- TRA's authorisation includes a unique fiscal number. If we cached this number and the cache expires, can we re-request?
- A B2C sale doesn't require pre-authorisation but must be reported in monthly batches. Batch event design.

**SMS**
- Customer's phone number is wrong. SMS bounces. Should the system retry, surface to the cashier, or both?
- SMS gateway charges per message. Cost-control: do we limit how many SMS per tenant per day? Where is this configured?
- A customer doesn't want SMS receipts. Where is consent captured? (Foundation's consent primitive.)

**Banks**
- Bank statement reconciliation: matching an event-store transaction to a bank line. Mostly automated, but mismatches are common. UI for the accountant to reconcile manually.
- Direct debit / standing order — recurring. Is this an integration concern or a domain (subscriptions, again)?

**Document Storage**
- We issue a document (hash, content, PDF). Foundation stores hash + content. Long-term, the PDF goes to external storage. The hash is the link. What if external storage loses the PDF? (We can regenerate from the event snapshot.)
- A regulator demands documents from 5 years ago. We have hash and event snapshot. Regenerate the PDF — is it byte-identical? (Should be — that's the immutability promise.)

**Cross-Cutting**
- An adapter has a bug that's emitting spurious events into BOS. How is this contained?
- An adapter must be retired (M-Pesa rolls out a new API and deprecates the old). Migration with no downtime?
- A tenant moves from one payment provider to another. Old data preserved; new data attached. Provider-agnostic representation?

---

## 8. Success Criteria

This Term is doing well when:

1. **No two Terms describe the same thing inconsistently.** Glossary holds.
2. **Every cross-Term open question has a status** — open, in discussion, resolved, escalated.
3. **The Charter has been amended cleanly** at least once during the concept phase, with audit trail.
4. **A new integration (e.g., a new mobile money provider) can be added without modifying engine code** — only an adapter.
5. **An external system being down does not block BOS internal operations** — events queue, state stays consistent.
6. **External events that arrive twice are processed once** — idempotency holds.
7. **Every external interaction is audited** — the regulator can ask "did BOS ever call TRA at 10:34 on Monday?" and we can answer.
8. **An auditor reviewing the concept phase concludes the system holds together** — the test of coherence.
9. **A junior engineer assigned to "wire up M-Pesa" can do it from the docs** without redesigning patterns.

---

## 9. Architect's Role for This Term

After Concept Team writes documents, the Architect translates them into:

| Concept Output | Architect Output |
|----------------|------------------|
| Inbound framework (CN-7-101) | Adapter ABC, adapter registry, dispatcher pipeline (validate → translate → command bus) |
| Outbound framework (CN-7-102) | Publisher ABC, publisher registry, event dispatcher with retry |
| Adapter pattern (CN-7-103) | Adapter base class, lifecycle hooks, registration API |
| Idempotency (CN-7-104) | ExternalEventReference model, deduplication strategy |
| Retry policy (CN-7-105) | Backoff algorithm, retry queue, dead-letter handling |
| Authentication (CN-7-106) | HMAC verifier, OAuth flow, key rotation procedures |
| Audit log (CN-7-107) | Integration audit table, query API |
| Permissions (CN-7-108) | Permission check API, business-scoped grants |
| Webhooks (CN-7-109) | Webhook signature verification, replay protection |
| M-Pesa category (CN-7-110) | M-Pesa adapter skeleton, common error mapping |
| Tax authority (CN-7-111) | E-invoice queueing pattern, fiscal number cache |

**Architect's directive to Developer for Term 7:**
- All external communication goes through the integration layer — never direct API calls from engines
- Every adapter is stateless: input → translate → output, no held state
- Every inbound event includes an `external_reference` for idempotency
- Failed external calls retry with exponential backoff up to a configurable limit, then dead-letter
- All integration audit log entries are append-only
- HMAC verification is mandatory for webhook endpoints
- Permission checks are enforced at the dispatcher level, not in adapters

---

## 10. Relationships with Other Terms

### Term 7 depends on:

| Term | Why |
|------|-----|
| Term 4 (Foundation) | Audit log, security, event-driven architecture |
| All other Terms (for Half A) | Coherence requires reading all Term outputs |

### Term 7 is depended on by:

| Term | Why |
|------|-----|
| All other Terms | They submit work for coherence review |
| Term 3 (Tenant Experience) | Customer-facing integrations (SMS receipts, payment status) |
| Term 5 (Universal Engines) | Cash, Procurement, Accounting touch external payment/banking systems |
| Term 6 (Vertical Engines) | Vertical integrations (kitchen printers, hotel PMS bridges) |

### Cross-Term Hand-Offs

- ← Every Term: "Here is our concept doc draft. Review for coherence."
- → Every Term: "Your doc conflicts with X / Y / Z. Here's the conflict and suggested resolution."
- ↔ Term 5 (Cash): "M-Pesa needs to feed into Cash events. Here's the contract."
- ↔ Term 1 (Platform): "Tax authority integration requires platform-level credentials per region. Here's the model."
- ↔ Term 3 (Tenant Experience): "When M-Pesa fails, the cashier needs to see X. Here's the UI contract."

---

## 11. Open Questions

1. **Term 7 membership.** Is Term 7 a few permanent thinkers (the natural fit for the "logic-thinker" role of this project) or rotating reviewers from other Terms? Hybrid?
2. **Coherence review timing.** Continuous (review every doc as written), gated (batch reviews before phase transitions), or both?
3. **Conflict escalation.** When two Terms genuinely disagree and Term 7 can't mediate, who decides? (Likely the Concept Lead — the human.)
4. **Integration test environments.** Do we maintain sandbox accounts at M-Pesa, TRA, etc. for testing? Per agent? Per platform? Cost implications.
5. **Provider neutrality.** Tax authority APIs differ by country. Is there a country-agnostic abstraction, or one adapter per country with no abstraction? (Probably abstraction *within reason*.)
6. **Webhook ordering.** External webhooks may arrive out of order. Does BOS resequence or process in arrival order?
7. **AI in integrations.** Can AI advise on integration choices (e.g., "this M-Pesa transaction is likely fraudulent")? Where in the pipeline does AI sit?
8. **Cost-sharing for shared integrations.** If BOS pays for the SMS gateway, do tenants get charged per message? Or is it bundled in subscription?
9. **The "long-lived integration" problem.** Some integrations (banking) require persistent connections. Does Foundation support this, or is it strictly request/response?
10. **Provider failure compensation.** If M-Pesa is down and a tenant loses sales, does BOS owe anything? Is this an SLA matter?
11. **Coherence as ongoing function.** After the concept phase, does Term 7 continue into architecture/implementation phases? (Strongly: yes.)
12. **Multi-region integrations.** A tenant in Tanzania paying a supplier in Kenya. Cross-border integration. Whose adapter? Which currency? Which compliance?

---

## 12. Notes on Existing Repository

The existing repository has:
- `core/integration/` — inbound, outbound, audit, adapters, permissions (Phase 9, 48 tests passing)
- Some adapter scaffolding but no production-grade adapters yet for M-Pesa, TRA, etc.
- Cross-engine subscription wiring (well-developed, with `_event_to_dict` translation pattern)

**Concept Phase guidance:** The integration framework in code is a solid skeleton. Term 7 will largely flesh it out with specific patterns and external system concepts.

**For the coherence half:** There is no existing equivalent in the code (because the code was built without a Charter). Term 7's coherence work is genuinely new ground in this project.

---

## 13. First Tasks for This Term

When this Term starts concept work, begin in this order:

**For Coherence (Half A):**
1. **CN-7-001 first** — coherence review process, so the Term can begin reviewing other Terms' work.
2. **CN-7-003 second** — master glossary, so every Term agrees on words.
3. **CN-7-005 third** — Charter amendment process.
4. **CN-7-002 fourth** — conflict resolution protocol.
5. **CN-7-006, CN-7-007, CN-7-004, CN-7-008** — supporting docs, written as needs surface.

**For Integration (Half B):**
6. **CN-7-100 first** — integration doctrine.
7. **CN-7-101, CN-7-102 in parallel** — inbound and outbound frameworks (they mirror each other).
8. **CN-7-103, CN-7-104** — adapter pattern, idempotency.
9. **CN-7-105, CN-7-106, CN-7-107, CN-7-108, CN-7-109** — operational concerns.
10. **CN-7-110 to CN-7-114** — specific integration categories.

The two halves can run in parallel within the Term, but Half A's work (CN-7-001, CN-7-003) must complete first because the other Terms depend on Term 7's review process to validate their drafts.

---

## 14. The Logic-Thinker Role

Among the seven Terms, Term 7 most closely matches this project's central role: a logic thinker standing outside the architecture, refusing to write code, asking only "does this make sense?"

This is not by accident. The role demands distance from any one engine, any one vertical, any one user group. The role demands willingness to question every decision, including its own previous ones.

Term 7's chair is held by the Concept Lead (the human) and the AI Logic Thinker working in partnership. The chair never writes a final word alone — every coherence decision is discussed, every conflict resolved with input from the affected Terms, every integration pattern validated against use cases.

---

## 15. A Note on the Discipline of Saying "No"

Term 7's most important word is "no." Saying "no" to scope creep. Saying "no" to a Term that wants to invent a new pattern when an existing one would do. Saying "no" to an integration shortcut that violates doctrine. Saying "no" to letting the Charter be ignored "just this once."

But saying "no" with reasons. Always with reasons.

> *"A Charter without an enforcer is a wish. Term 7 is how the wish becomes a discipline."*

---

## Standing Decisions & Cross-Term Work

> This Term **owns** `DECISION-LOG.md`, `CROSS-TERM-COLLABORATION.md`, and `CROSS-TERM-REQUEST-REGISTER.md`. It triages every CTR and mediates conflicts. As **Overseer**, it coordinates section-by-section discussion and confirms coherence — but it **never overrides the Concept Lead**, who is the final authority on every proposal.

| New Concept | From | This Term must decide |
|-------------|------|------------------------|
| CN-7-115 | D-002 | **AI in the integration pipeline** (e.g., fraud hints) and **prompt-injection defence** for external data; verify advisory-only holds across runtime, integrations, and the build pipeline |

**Coherence duties from the decisions:**
- **D-001:** confirm payment adapters attach at the **tender** boundary (CTR-006) and that every vertical truly shares one checkout.
- **D-002:** drive CTR-010 — collect each Term's AI touchpoints and confirm every one is advisory/human-gated.
- **D-003:** drive CTR-013 — confirm referral never creates a de-facto Global Agent or escapes regional compliance (Law 6).

---

*— End of Term 7 Brief —*
