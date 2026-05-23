# Term 1 — Platform Stewards

> **Parent:** [BOS-CONCEPT-CHARTER.md](../BOS-CONCEPT-CHARTER.md) — read first
> **Type:** People Term (Part A)
> **Status:** Concept Phase v1.0

---

## 1. Mission

Platform Stewards are the people inside BOS itself — not customers, not agents, but the **owners and operators** of the BOS platform as a product. They set the rules everyone else plays by: pricing, engine catalog, compliance pack approval, agent vetting, payout policy, system-wide policy.

This Term thinks about how those people do their work safely, transparently, and at scale across 10+ countries — without becoming bottlenecks, without losing audit trail, and without accidentally pushing changes that break thousands of tenants.

---

## 2. Mission Question

> **"How do we govern a system used in 100+ countries, by hundreds of agents, by millions of tenants — without breaking it, without playing favourites, and without losing the ability to prove what we changed and why?"**

---

## 3. Scope (In)

This Term owns concepts for:

- **Platform Admin Portal** — the dashboard, navigation, and pages that platform staff use
- **Pricing Authority** — how subscription plans, engine combos, and rates are defined and changed
- **Compliance Pack Approval** — the process by which a country's compliance rules become live in BOS
- **Agent Vetting and Governance** — agent registration approval, suspension, termination, and agreement management
- **Payout Authority** — approving commission payouts to agents
- **Platform Policies** — system-wide rules (trial periods, onboarding policy, refund policy, etc.)
- **Platform Audit Log** — the immutable record of every platform-level administrative action
- **Platform User Management** — who works inside BOS itself, with what permissions
- **Expansion Gates** — the readiness checks before BOS opens billing in a new country
- **Cross-Tenant Concerns** — when a tenant transfer is requested, when fraud is suspected, when an agent dispute arises

---

## 4. Scope (Out)

This Term does NOT own:

| What | Owned By |
|------|----------|
| Agent's daily workflow (onboarding tenants, supporting them) | Term 2 — Regional Distribution |
| Tenant's daily workflow (POS, kitchen, inventory) | Term 3 — Tenant Experience |
| The event store, primitives, security kernel | Term 4 — Foundation |
| Engine internals (how Accounting computes journals) | Term 5 — Universal Engines |
| How Retail/Restaurant/Hotel/Workshop work internally | Term 6 — Vertical Engines |
| Cross-term coherence checks | Term 7 — Integration & Coherence |
| External integration adapters (M-Pesa, TRA APIs) | Term 7 — Integration & Coherence |

---

## 5. Audiences Served

From Charter Section 3, this Term serves **Group A — Platform Stewards** entirely:

| Role | What Term 1 designs for them |
|------|------------------------------|
| Super Admin | The "god mode" controls, with safety brakes |
| Platform Admin | Day-to-day operational dashboards |
| Finance Admin | Pricing config, commission rules, payout queues |
| Agent Manager | Agent registration approval, performance dashboards |
| Compliance Officer | Compliance pack review and approval workflows |
| Support Staff | Read-only visibility into tenant and agent issues |

Term 1 also has a **secondary interest** in Group B (agents) and Group C (tenants) — but only to the extent that platform staff need to *see* them, not to *be* them.

---

## 6. Key Concepts This Term Will Produce

Concept documents under this Term:

| ID | Concept | What it answers |
|----|---------|-----------------|
| CN-1-001 | Platform Identity & Role Model | Who works at BOS, with what powers |
| CN-1-002 | Engine Catalog & Combo Management | How subscription plans are composed from engines |
| CN-1-003 | Pricing & Rate Governance | How prices are set, changed, and audited per region |
| CN-1-004 | Compliance Pack Lifecycle | How a country goes from "draft" to "live" |
| CN-1-005 | Agent Vetting & Governance | How agents are approved, monitored, suspended |
| CN-1-006 | Commission & Payout Authority | The platform-side rules of how agents get paid |
| CN-1-007 | Trial & Promotion Policy | How free trials and platform-wide promos are governed |
| CN-1-008 | Tenant Subscription Oversight | What platform sees about tenants without violating privacy |
| CN-1-009 | Audit & Accountability | Every platform action recorded; nothing deletable |
| CN-1-010 | Expansion Gates (New Country Readiness) | The checklist that gates a new country going live |
| CN-1-011 | Platform Dashboard Composition | What KPIs platform staff see, by role |
| CN-1-012 | Disaster & Recovery Authority | Who can do what in a crisis (data restore, mass revert) |

---

## 7. Edge Cases to Consider

The hard questions this Term must answer:

**Authority and Conflict**
- What happens when a Super Admin makes a destructive change at 2am with no second approver? Should certain actions require dual approval?
- What if a Finance Admin wants to refund a tenant, but the Compliance Officer says the refund would violate VAT regulations in that country?
- What if an Agent Manager wants to suspend an agent, but that agent has 50 active tenants — what happens to those tenants?

**Pricing and Fairness**
- If platform changes engine pricing globally, what happens to existing active subscriptions? Grandfathered? Or do they migrate at next renewal?
- Can prices differ per country (affordability weighting) without being seen as discrimination?
- If an agent negotiates a custom rate for a tenant, who approves it?

**Compliance Reality**
- What happens when a country's tax rule changes overnight (e.g., Tanzania raises VAT from 18% to 20%)? How fast can a compliance pack be updated and rolled out?
- Who is liable if a compliance pack has a bug that causes tenants to under-pay tax?
- Can a country be "partially live"? (e.g., retail compliance is ready, restaurant isn't)

**Audit and Trust**
- Can Super Admin actions be reversed by another Super Admin? Or are they final?
- What is logged about *failed* admin attempts (e.g., a Support Staff trying to access financial data)?
- How do we prove to regulators that platform did/didn't do something — in a court of law?

**Onboarding New Countries**
- What are the minimum viable requirements before BOS opens billing in a new country?
- Who has the authority to declare a country "live"?
- What's the rollback procedure if a country goes live and things break?

**Edge Roles**
- Does BOS have "Read-only auditor" accounts for external regulators or partners?
- Can a Compliance Officer also be a Finance Admin? (separation of duties)
- What about a Super Admin who is *also* registered as an Agent? (conflict of interest)

---

## 8. Success Criteria

This Term is doing well when:

1. **Every platform action can be explained, justified, and traced.** Nothing happens silently.
2. **Adding a new country takes one compliance pack + one expansion-gate sign-off.** No code changes.
3. **Pricing can be changed without breaking active subscriptions.** Rate snapshots are immutable.
4. **An agent dispute can be resolved in a documented, fair process.** Not by whim.
5. **No single human can destroy BOS.** All destructive actions require either dual approval or are technically impossible.
6. **A regulator can audit the platform without us preparing anything special.** Audit log is always ready.
7. **A new Platform Admin can be onboarded in one day** using the docs from this Term.

---

## 9. Architect's Role for This Term

After Concept Team writes documents, the Architect translates them into:

| Concept Output | Architect Output |
|----------------|------------------|
| Role permission matrix (CN-1-001) | RBAC schema, permission tables, middleware design |
| Engine catalog (CN-1-002) | Engine registry data model, combo definition schema |
| Pricing rules (CN-1-003) | Rate event schemas, price snapshot model, regional config schema |
| Compliance pack lifecycle (CN-1-004) | State machine spec, approval workflow event design |
| Agent governance (CN-1-005) | Agent record model, status transitions, audit hooks |
| Commission rules (CN-1-006) | Commission accrual engine spec, payout queue model |
| Trial policy (CN-1-007) | Trial event types, promotion event types, applicability rules |
| Audit log (CN-1-009) | Append-only audit table design, query API, retention policy |
| Expansion gates (CN-1-010) | Gate state model, readiness check API |
| Platform dashboards (CN-1-011) | Aggregation projections, KPI calculation rules |

**Architect's directive to Developer for Term 1:**
- Build the Platform Admin portal with RBAC enforced at every endpoint
- All admin actions emit immutable audit events
- Pricing changes never overwrite — they create new effective-date snapshots
- Compliance packs are versioned data, not code
- Every dashboard widget is rebuildable from events

---

## 10. Relationships with Other Terms

### Term 1 depends on:

| Term | Why |
|------|-----|
| Term 4 (Foundation) | Audit log primitive, event store, security primitives |
| Term 7 (Integration & Coherence) | Validates Term 1's policies don't conflict with Tenant or Agent expectations |

### Term 1 is depended on by:

| Term | Why |
|------|-----|
| Term 2 (Regional Distribution) | Agent governance rules; trial policy; commission framework |
| Term 3 (Tenant Experience) | Pricing visible to tenants; compliance packs that affect their docs |
| Term 5 (Universal Engines) | Engine catalog defines which engines exist |
| Term 6 (Vertical Engines) | Same as above for vertical engines |

### Cross-Term Hand-Offs

- → Term 2: "Here are the rules agents must follow." Term 2 designs the agent experience around them.
- → Term 3: "Here is the pricing and compliance that tenants see." Term 3 designs the tenant experience around them.
- ← Term 7: "Your trial policy contradicts what tenants expect on day 31." Term 1 revises.

---

## 11. Open Questions

Questions Term 1 has not yet decided. These need discussion with the Concept Lead before docs CN-1-001 through CN-1-012 are finalised.

1. **Dual approval threshold.** Should any platform action above a certain financial impact (e.g., refunding $10K+, suspending an agent with 100+ tenants) require two Super Admins?
2. **Tenant transfer arbitration.** When a tenant wants to switch from Agent A to Agent B, who decides — and on what timeline?
3. **Compliance pack publishers.** Can Regional Agents draft compliance pack updates that Platform Admins approve, or is drafting purely a Platform Compliance Officer job?
4. **Pricing transparency.** Should tenants see how their price is computed (engine + region + tier), or only the final number?
5. **Audit retention.** Forever, or with a published retention policy (e.g., 7 years)?
6. **Multi-tenancy of platform users.** Can a single human be a Platform Admin in two BOS instances (e.g., a partnered company)?
7. **AI in platform decisions.** Should AI suggest agent suspensions, pricing changes, compliance issues? (Charter says AI is advisory only — but how is this surfaced to platform staff?)
8. **Cross-currency commission.** Agent works in Tanzania (TZS), gets paid in… what? Whose currency? Whose FX rate?

---

## 12. Notes on Existing Repository

The existing repository has partial implementations of:
- Platform admin portal pages (`/platform/*`)
- Agent management features
- Subscription and billing scaffolding
- Audit logging
- Region packs (Phase 12)

**Concept Phase guidance:** These are reference for *what was tried*, not blueprints for *what we will build*. Concept docs CN-1-001 through CN-1-012 may agree with or diverge from existing code. The Charter wins; the existing code does not.

---

## 13. First Tasks for This Term

When this Term starts concept work, begin in this order:

1. **CN-1-001 first** — define roles before anything else, because every other doc references roles.
2. **CN-1-009 second** — define audit log shape early, because every other doc says "this is logged."
3. **CN-1-002 third** — engine catalog, because pricing and combos depend on it.
4. **CN-1-003 fourth** — pricing, since compliance, trials, and commissions depend on it.
5. Then in parallel: CN-1-004 (compliance), CN-1-005 (agents), CN-1-006 (commissions).
6. CN-1-007, CN-1-008, CN-1-010, CN-1-011, CN-1-012 follow as foundations stabilise.

---

*— End of Term 1 Brief —*
