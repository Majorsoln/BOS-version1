# BOS Concept Charter

> **Version:** 1.0 (Concept Phase)
> **Status:** Living Document — the constitution of all concept work
> **Audience:** Concept Terms, Architects, Developers, AI Agents
> **Reading Order:** Read this FIRST before any Term Brief

---

## 0. How to Read This Document

This Charter is the **single source of truth** for what BOS is, who it serves, and how concept work is organised. Every Concept Term reads this first, then their own Term Brief.

**Three reading levels:**

| Level | Sections | Time | For Whom |
|-------|----------|------|----------|
| **Quick** | 1, 2, 5 | 10 min | Anyone joining a Term |
| **Working** | All sections | 30 min | Term Leads, Architects |
| **Deep** | All + linked Term Briefs | 2 hours | Integration & Coherence Team |

**Rule:** If any part of this Charter conflicts with a Term Brief, **the Charter wins**. Term Briefs may add detail but never override.

---

## 1. What BOS Is (Identity)

### 1.1 The Problem

African businesses — from the mama selling tomatoes in Kariakoo to the workshop building windows in Arusha to the hotel in Zanzibar — share a common pain:

> **They cannot trust their own numbers.**

Most are running on three broken tools at once:
- A notebook (truth lives in someone's head)
- A spreadsheet (truth changes silently)
- A "system" that breaks every time tax law changes

When the owner asks *"How much did we really make this month?"*, no one can answer with confidence. When the tax officer arrives, documents do not match each other. When the owner expands to a second branch, the second branch becomes a second universe.

This is not a small inconvenience. It is the **single biggest reason** small and medium African businesses cannot grow, raise capital, or survive transitions of ownership.

### 1.2 What BOS Is

> **BOS (Business Operating System) is a deterministic, event-sourced, legally defensible business kernel.**

It is the **foundation** on which any kind of business can run — retail, restaurant, workshop, hotel, services, and (in future) insurance, healthcare, marketing, logistics — without forking the system.

**The three things BOS guarantees:**

1. **Truth.** Every business event is recorded once and never silently changed. The owner can always answer *"what really happened?"* with proof.
2. **Replay.** The system's state can be rebuilt from events at any point in history. Nothing is hidden in databases that can be tampered with.
3. **Locality.** BOS works the same way in Tanzania, Kenya, Uganda, Rwanda, Nigeria, Ghana — but follows each country's tax law, document law, and language without code changes.

### 1.3 What BOS Is NOT

To prevent scope drift, BOS is explicitly **not** these things:

| BOS is NOT | Why |
|------------|-----|
| An ERP system | ERPs assume static processes; BOS adapts via events |
| A statutory accounting system | BOS gives management truth; a chartered accountant files taxes |
| A point-of-sale app | POS is one feature; BOS is the kernel below all features |
| A country-specific compliance tool | BOS is global; compliance is plugged in per region |
| A replacement for human judgement | AI advises; humans decide |
| A free product | BOS is sold through regional agents on subscription |

### 1.4 The BOS Promise

> **"Global by architecture, local by law, neutral by design."**

- **Global by architecture:** One codebase, one kernel, works anywhere
- **Local by law:** Every region has its own compliance pack — no `if country == "TZ"` in code
- **Neutral by design:** BOS does not prefer one supplier, bank, payment provider, or vertical over another

---

## 2. Founding Principles (Doctrine)

These principles are **non-negotiable**. Every Concept Term must honour them. Any proposal that breaks one is rejected automatically.

### 2.1 The Three Laws

**Law 1 — State Is Derived from Events Only.**
There is no hidden mutation anywhere in BOS. The event store is the only place where truth lives. Projections, dashboards, and reports are all rebuildable from events.

**Law 2 — Engines Are Isolated.**
No engine calls another engine directly. They communicate only by publishing and subscribing to events. This means Retail can grow without breaking Restaurant, and new engines (insurance, healthcare) can be added without touching old ones.

**Law 3 — AI Is Advisory Only.**
AI never commits state, never writes events autonomously, never approves a transaction without a human. AI suggests; humans decide. This is a legal requirement, not a preference.

### 2.2 Three Additional Laws (Added by Concept Team)

**Law 4 — Flexibility Is a First-Class Citizen.**
The system must accommodate verticals not yet imagined. Today: retail, restaurant, hotel, workshop. Tomorrow: insurance, hospital, school, logistics, marketing agency. Foundation must not assume which verticals exist.

**Law 5 — Compliance Is Configured, Not Coded.**
Every country's tax rules, document rules, and reporting rules live in **data files** (compliance packs), not in `if/else` blocks. Adding Ethiopia must not require a single code change.

**Law 6 — Distribution Is Regional.**
There is no "Global Agent." Every agent operates within one country/region because compliance is regional. An agent who cannot file proper tax documents in their region cannot sell BOS.

### 2.3 Behavioural Doctrine

| Doctrine | Meaning |
|----------|---------|
| **Build from truth to operations to insight to intelligence to scale** | No skipping phases. No premature optimisation. |
| **If an action cannot be explained as an event, corrected safely, and audited clearly — it does not belong in BOS** | Test for every new feature |
| **Read the Charter, then act** | No assumptions. No "I think it works like..." |
| **Document the reason, not just the action** | Future readers need the "why" |

### 2.4 Standing Decisions

The Laws above are non-negotiable doctrine. Concrete choices made *within* the doctrine are recorded in **`DECISION-LOG.md`** (owned by Term 7). A standing decision may never contradict a Law; if it appears to, Term 7 escalates to the Concept Lead. Every Term must read the Decision Log after this Charter and honour its decisions in their proposals.

---

## 3. The People (Audiences)

BOS serves four distinct groups of people. Each Concept Term focuses on a specific subset.

### 3.1 Group A — Platform Stewards (BOS itself)

| Role | What they do |
|------|--------------|
| Super Admin | Owns the platform, sets the highest policies |
| Platform Admin | Day-to-day platform operations |
| Finance Admin | Pricing, agent commissions, payouts |
| Agent Manager | Registers, trains, monitors regional agents |
| Compliance Officer | Approves regional compliance packs |
| Support Staff | Level-2 support for agents and tenants |

**Note:** Platform Stewards never touch tenant business data. They configure the platform; tenants run their businesses.

### 3.2 Group B — Regional Agents (Distribution)

| Role | What they do |
|------|--------------|
| Regional Agent (Owner) | Registered in one country; sells BOS to local businesses; provides L1 support; handles regional compliance documents |
| Agent Staff | Helps the owner with onboarding and support |
| Agent Viewer | Read-only access (for back-office helpers) |

**Critical rule:** Tenants cannot self-register. Every tenant must come through a Regional Agent in their country. This guarantees compliance accountability.

### 3.3 Group C — Tenants (Businesses Using BOS)

By type of business:

| Tenant Type | Examples |
|-------------|----------|
| Retail | Supermarket, electronics shop, pharmacy, boutique |
| Restaurant | Sit-down, fast food, café, bar |
| Hotel/Hospitality | Hotel, guest house, lodge |
| Workshop | Aluminium fabrication, carpentry, glass, panel-making |
| Service Provider | Consulting, repair, salon, professional services |
| Mixed | A hotel that also runs a restaurant and a shop |
| **Future verticals** | Insurance broker, clinic, school, marketing agency |

By role inside a tenant:

| Role | Permissions |
|------|------------|
| Tenant Owner | Full control of their business |
| Tenant Manager | Day-to-day operations, no financial config |
| Cashier | Sales, no refunds without approval |
| Storekeeper | Inventory only |
| Department-specific roles | Waiter, chef, fundi, receptionist, etc. |

### 3.4 Group D — External Stakeholders

| Stakeholder | Relationship |
|-------------|--------------|
| End customers | Pay tenants; receive receipts; scan QR codes |
| Suppliers | Sell to tenants; receive POs and payments |
| Tax authorities | TRA (TZ), KRA (KE), URA (UG), etc. Receive compliance documents |
| Payment providers | M-Pesa, Tigo Pesa, Airtel Money, banks, card networks |
| Banks | Hold tenant accounts; receive payment references |

External stakeholders never log into BOS. They interact through documents, integrations, or QR codes.

---

## 4. The System (What We're Building)

BOS has **five layers**. Each layer has a Concept Term responsible for it.

### 4.1 Layer 1 — Foundation (The Kernel)

The unbreakable bottom. Everything else sits on top.

- Event store (append-only, hash-chained)
- Command bus and dispatcher
- Engine registry and isolation
- Primitives (ledger, item, party, document, workflow, approval, obligation, inventory movement)
- Security and tenant isolation
- Replay engine
- Document engine
- Audit log

**Owned by:** Term 4 — Foundation

### 4.2 Layer 2 — Universal Engines (For Every Business)

Engines that every tenant needs, regardless of vertical.

- Accounting (management-truth double entry)
- Cash Management (drawers, sessions, reconciliation)
- Inventory (movements, FIFO/LIFO, lots, offcuts)
- Procurement (requisition → PO → GRN → invoice → payment)
- HR & Payroll (employees, payroll, ledger integration)
- Reporting & BI (KPI projections, snapshots)

**Owned by:** Term 5 — Universal Engines

### 4.3 Layer 3 — Vertical Engines (Per Business Type)

Engines specific to a vertical. **New verticals are added here without touching Layers 1, 2, or 4.**

Current:
- Retail (POS, baskets, multi-price)
- Restaurant (tables, orders, kitchen workflow, split billing)
- Hotel/Hospitality (rooms, reservations, stays)
- Workshop (parametric geometry, cut lists, quotes, offcuts)

**Future (designed to plug in cleanly):**
- Insurance
- Healthcare
- Marketing services
- Logistics
- Education

**Owned by:** Term 6 — Vertical Engines

### 4.4 Layer 4 — Intelligence Layer (Advisory & Safety)

Cross-cutting concerns that touch every engine.

- AI Advisory (cost anomalies, profit simulation, risk alerts)
- Decision Journal (every AI suggestion is logged)
- Promotion Engine (discounts, vouchers, loyalty)
- Compliance Engine (regional packs)
- Security & Anomaly Detection

**Owned by:** Term 4 (Foundation) for guardrails; Term 5 (Universal) for engines; Term 1 (Platform) for compliance packs.

### 4.5 Layer 5 — Platform Layer (BOS the Product)

The systems that make BOS itself work as a business.

- Subscription plans and billing
- Agent management and commission
- Tenant onboarding
- White-label branding
- Integration gateway (M-Pesa, banks, tax APIs)
- Platform dashboards

**Owned by:** Term 1 — Platform Stewards (admin); Term 2 — Regional Distribution (agent side); Term 7 — Integration & Coherence (gateway).

---

## 5. The Concept Terms (7)

Concept work is divided into **seven Terms**. Each Term has a focused mission, a defined audience, and an architect role.

### 5.1 The Two Halves

**Part A — People Terms (3)** focus on **who uses BOS and why**.
**Part B — System Terms (3)** focus on **what we are building**.
**Part C — Coherence Term (1)** ensures the other six are saying the same thing.

```
PART A — PEOPLE TERMS              PART B — SYSTEM TERMS         PART C — COHERENCE
────────────────────────          ────────────────────────      ──────────────────────
1. Platform Stewards    ◄──┐      4. Foundation                  7. Integration &
2. Regional Distribution    ├─────► 5. Universal Engines    ◄────── Coherence
3. Tenant Experience    ◄──┘      6. Vertical Engines              (validates all)
```

### 5.2 Term Summary Table

| # | Term | Mission Question | Owns |
|---|------|------------------|------|
| 1 | Platform Stewards | *How do we govern a system used in 10+ countries without breaking?* | Admin portal, policy, compliance approval |
| 2 | Regional Distribution | *How do we put BOS in the hands of a Kariakoo seller without scaring her?* | Agent portal, onboarding, regional compliance |
| 3 | Tenant Experience | *How does Mama Asha get truth about her business from her phone?* | Tenant dashboards, mobile UX, roles, training |
| 4 | Foundation | *How do we build a kernel that holds under decades of change — and still proves to a court that nothing was silently changed?* | Event store, primitives, security, doctrine enforcement |
| 5 | Universal Engines | *What does EVERY business — retail, hotel, hospital, marketing — share?* | Accounting, cash, inventory, procurement, HR, reporting |
| 6 | Vertical Engines | *What is unique to each business type, and how do new ones plug in?* | Retail, Restaurant, Hotel, Workshop, + future verticals |
| 7 | Integration & Coherence | *Are all six Terms saying the same thing? Do their ideas fit together?* | Cross-term validation, charter enforcement, external gateway logic |

### 5.3 Term Briefs Location

Each Term has a separate Brief document with full detail:

```
bos-concept/
├── BOS-CONCEPT-CHARTER.md            ← You are here
└── terms/
    ├── TERM-1-PLATFORM-STEWARDS.md
    ├── TERM-2-REGIONAL-DISTRIBUTION.md
    ├── TERM-3-TENANT-EXPERIENCE.md
    ├── TERM-4-FOUNDATION.md
    ├── TERM-5-UNIVERSAL-ENGINES.md
    ├── TERM-6-VERTICAL-ENGINES.md
    └── TERM-7-INTEGRATION-COHERENCE.md
```

Each Brief follows the same structure (see Section 7).

---

## 6. How Concept Work Flows

### 6.1 The Three Roles in Each Term

Every Term has three roles working in sequence:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  CONCEPT TEAM   │───►│   ARCHITECT     │───►│  IMPLEMENTER    │
│  (Logic)        │    │   (Design)      │    │  (Code)         │
└─────────────────┘    └─────────────────┘    └─────────────────┘
   Asks WHY              Decides HOW            Writes WHAT
   Defines purpose       Designs structure      Builds it
   Maps user pain        Maps to architecture   Tests it
   Writes requirements   Writes specifications  Writes code
```

**Concept Team** (this layer) does not write code. It writes:
- The problem statement
- The user stories with edge cases
- The logic flow (in plain language)
- The success criteria
- The questions the Architect must answer

**Architect** translates concepts into:
- Module/component design
- Data contracts (event schemas, command schemas)
- Integration points with other engines
- Sequence diagrams
- Implementation directives for the developer

**Implementer (Developer)** receives the Architect's directives and writes:
- Code
- Tests
- Documentation references back to the Concept and Architect docs

### 6.2 The Charter Hierarchy

```
                        BOS-CONCEPT-CHARTER.md
                        (Constitution — wins all conflicts)
                                  │
              ┌──────────┬────────┴────────┬──────────────┐
              ▼          ▼                 ▼              ▼
         TERM BRIEFS  TERM BRIEFS    TERM BRIEFS    TERM BRIEFS
         (purpose,    (purpose,      (purpose,      (purpose,
         scope, role) scope, role)   scope, role)   scope, role)
              │           │              │              │
              ▼           ▼              ▼              ▼
         CONCEPT     CONCEPT         CONCEPT         CONCEPT
         DOCS        DOCS            DOCS            DOCS
         (per topic) (per topic)     (per topic)     (per topic)
              │           │              │              │
              └───────────┴──────┬───────┴──────────────┘
                                 ▼
                        TERM 7 VALIDATION
                        (does it all fit?)
                                 ▼
                        ARCHITECT SPECS
                                 ▼
                        DEVELOPER CODE
```

### 6.3 Workflow Rules

1. **No code without a concept doc.** If a feature has no concept doc, it cannot be built.
2. **No concept doc without Term ownership.** Every concept doc belongs to exactly one Term.
3. **Cross-Term concepts go through Term 7.** If a concept touches two Terms (e.g., a tenant dashboard widget that needs a foundation primitive), Term 7 mediates.
4. **Charter conflicts halt work.** If a concept conflicts with the Charter, work stops until the Charter is amended OR the concept is revised.
5. **Concept docs are living.** They are versioned, not frozen. But changes are reviewed.
6. **Cross-Term needs become Cross-Term Requests.** When a proposal depends on another Term (a primitive, field, event, policy, or UI contract), the dependency is filed as a **CTR** in `CROSS-TERM-REQUEST-REGISTER.md`. A proposal cannot advance to the Architect phase while it has any open CTR. See `CROSS-TERM-COLLABORATION.md`.

### 6.4 Deliverables Per Term

Every Term produces, at minimum:

| Deliverable | Format | Audience |
|-------------|--------|----------|
| Term Brief | `.md` | Everyone (definition of the Term) |
| Concept documents (multiple) | `.md` per topic | Architects |
| User stories with edge cases | `.md` or table | Architects + Developers |
| Success criteria | `.md` | Everyone |
| Open questions log | `.md` | Term 7 + other Terms |

---

## 7. Term Brief Structure (Template)

Every Term Brief follows this exact structure. This is enforced for AI-agent readability.

```markdown
# Term N — [Name]

## 1. Mission
One paragraph. Why this Term exists.

## 2. Mission Question
The single question this Term answers.

## 3. Scope (In)
Bulleted list. What this Term owns.

## 4. Scope (Out)
Bulleted list. What this Term explicitly does NOT own (with pointer to who does).

## 5. Audiences Served
Which Group(s) from Charter Section 3 this Term serves.

## 6. Key Concepts This Term Will Produce
Bulleted list. What concepts this Term writes documents for.

## 7. Edge Cases to Consider
The hard questions this Term must answer.

## 8. Success Criteria
How we know this Term is doing well.

## 9. Architect's Role for This Term
What the Architect does after Concept Team finishes.

## 10. Relationships with Other Terms
Who this Term depends on; who depends on this Term.

## 11. Open Questions
The things this Term has not yet decided.
```

---

## 8. Cross-Cutting Rules

These rules apply to ALL Terms, all concepts, all docs.

### 8.1 Naming Conventions

| Item | Convention | Example |
|------|------------|---------|
| Event types | `<engine>.<noun>.<verb>.v<n>` | `retail.sale.completed.v1` |
| Commands | `<engine>.<noun>.<action>.request` | `retail.sale.complete.request` |
| Files (concept) | `UPPER-KEBAB-CASE.md` | `TENANT-DASHBOARD-CONCEPT.md` |
| Files (term briefs) | `TERM-N-NAME.md` | `TERM-3-TENANT-EXPERIENCE.md` |
| Concept doc IDs | `CN-<term>-<seq>` | `CN-3-001`, `CN-3-002` |

### 8.2 Language

- **Document body:** English (technical, AI-friendly)
- **User stories:** English, but include real African business names (Mama Asha, Karakana ya Mzee John, Hoteli ya Bahari)
- **Currency in examples:** TZS, KES, UGX, NGN — never USD unless explicitly modelling cross-border
- **No country-specific code logic in concept docs.** Compliance lives in packs.

### 8.3 The "Three Whys" Rule

Every concept doc must answer:
1. **Why does this matter?** (user pain)
2. **Why does it belong in BOS and not elsewhere?** (kernel justification)
3. **Why this design and not alternatives?** (reasoning, with alternatives discussed)

### 8.4 The "Show Don't Tell" Rule

Every concept must include at least one concrete example using a real African scenario:
- A specific business (Duka la Mama Asha, Karakana ya Arusha)
- A specific moment (Friday evening rush, end-of-month closing)
- A specific decision (whether to give credit, whether to expand)

Abstract concepts without examples are rejected.

---

## 9. Success Criteria for the Concept Phase

We will know the Concept Phase has succeeded when:

1. **Every Term has a complete Brief.** All seven Term Briefs are written, reviewed, and approved.
2. **Every layer has at least 5 concept documents.** Foundation, Universal Engines, Vertical Engines each have working drafts.
3. **An Architect can read a concept doc and design from it without asking questions.** This is the bar.
4. **Term 7 has validated cross-term coherence.** No two Terms contradict each other.
5. **The Charter has been amended at least once.** Because if it hasn't, we haven't really stress-tested it.
6. **A new vertical can be added on paper without changing Foundation or Universal Engines.** This is the flexibility test.
7. **A regional compliance change (e.g., a new tax rule in Rwanda) can be made by editing one config file in a concept.** This is the locality test.

---

## 10. What Happens After the Concept Phase

The Concept Phase ends when Section 9's criteria are met. After that:

```
Concept Phase ──► Architecture Phase ──► Implementation Phase
   (Logic)            (Design)              (Code)
```

The existing repository (reviewed during concept work) serves as **reference only**. The Concept Phase does not assume the existing code is correct; it assumes only that the existing code teaches us about edge cases we might miss.

**No code is written during Concept Phase.** All output is `.md` documents.

---

## 11. Living Document Notice

This Charter is version 1.0. It will be amended. Amendment rules:

- Any Term may propose an amendment via a written proposal to Term 7.
- Term 7 evaluates whether the amendment maintains coherence.
- Amendments approved by Term 7 + the Concept Lead (you, the human) become effective.
- All Terms are notified of amendments via `CHANGELOG.md`.
- Standing decisions are recorded in `DECISION-LOG.md`; cross-Term agreements are tracked in `CROSS-TERM-REQUEST-REGISTER.md` per the process in `CROSS-TERM-COLLABORATION.md`.

---

## 12. Glossary

| Term | Meaning |
|------|---------|
| **BOS** | Business Operating System — the kernel we are designing |
| **Tenant** | A business that uses BOS (the customer) |
| **Agent** | A regional reseller who onboards and supports tenants |
| **Engine** | A bounded module that owns one business domain (Accounting, Retail, etc.) |
| **Event** | An immutable record of something that happened |
| **Command** | A request to do something (may be accepted or rejected) |
| **Primitive** | A reusable building block shared by many engines |
| **Compliance Pack** | A configuration bundle that adapts BOS to one country's laws |
| **Vertical** | A type of business (Retail, Restaurant, Hotel — and future: Insurance, Healthcare) |
| **Projection** | A read-model rebuilt from events (used for dashboards and queries) |
| **Concept Term** | A focused team working on one slice of BOS thinking |
| **Term Brief** | The document defining a Term's mission and scope |
| **Concept Document** | A focused document on one topic owned by one Term |

---

## 13. Final Word

> *"BOS is not a system. It is a promise — that any African business, from any town, of any size, can have truth about itself. The Concept Phase is how we honour that promise before writing a single line of code."*

Read your Term Brief next.

— *End of Charter*
