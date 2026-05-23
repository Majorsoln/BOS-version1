# Term 2 — Regional Distribution

> **Parent:** [BOS-CONCEPT-CHARTER.md](../BOS-CONCEPT-CHARTER.md) — read first
> **Type:** People Term (Part A)
> **Status:** Concept Phase v1.0

---

## 1. Mission

Regional Distribution is how BOS reaches actual businesses. Every tenant on BOS comes through a **Regional Agent** — a person or company registered in one country, accountable for that country's compliance, trained to onboard local businesses, and rewarded by commission.

There is no "Global Agent." This is a Charter law: distribution is regional because compliance is regional. An agent who cannot file the right tax document in Dar es Salaam cannot sell BOS to a Dar es Salaam business.

This Term designs how agents work: the portal they use, the onboarding flow they walk new tenants through, the commission they earn, the training they receive, the compliance documents they upload, and the safety boundaries (because agents have power that, abused, hurts tenants and BOS together).

---

## 2. Mission Question

> **"How do we put BOS into the hands of a tomato seller in Kariakoo without scaring her — through an agent who understands her country, speaks her language, knows her tax officer, and earns fairly for that work?"**

---

## 3. Scope (In)

This Term owns concepts for:

- **Agent Portal** — the dashboard and pages agents use daily (`/agent/*`)
- **Tenant Onboarding Workflow** — the multi-step process an agent walks a new tenant through
- **Regional Compliance Document Management** — the docs agents must upload to operate in their country (business registration, tax certificate, agreement)
- **Commission Visibility & Payouts (Agent side)** — how an agent sees what they're earning and when they get paid
- **L1 Support** — first-line support that agents provide to their tenants
- **Tenant Health on Agent Side** — agents seeing if their tenants are using BOS well, paying on time, hitting limits
- **Agent Training & Certification** — how agents become competent enough to sell BOS responsibly
- **Sub-Agent / Agent Staff Management** — how an agent business hires helpers
- **Transfer Requests** — when a tenant asks to move to a different agent
- **Market Intelligence (Agent View)** — what an agent sees about their region's market without crossing privacy lines

---

## 4. Scope (Out)

This Term does NOT own:

| What | Owned By |
|------|----------|
| Setting commission rates and trial policy | Term 1 — Platform Stewards |
| Approving agent registration, suspension, termination | Term 1 — Platform Stewards |
| Designing the tenant's own daily UX (POS, kitchen, inventory) | Term 3 — Tenant Experience |
| The kernel, audit log, security | Term 4 — Foundation |
| Engine internals (Accounting, Cash, Inventory, Procurement, etc.) | Term 5 — Universal Engines |
| Vertical engine logic (Retail POS internals, Restaurant orders, etc.) | Term 6 — Vertical Engines |
| External integrations (M-Pesa for tenant payments) | Term 7 — Integration & Coherence |

**Important distinction:** Term 1 sets the *rules* agents follow. Term 2 designs the *experience* agents have inside those rules.

---

## 5. Audiences Served

From Charter Section 3, this Term serves **Group B — Regional Agents** entirely:

| Role | What Term 2 designs for them |
|------|------------------------------|
| Regional Agent (Owner) | Full agent portal, commission view, onboarding wizard |
| Agent Staff | Tenant management and support, no financial visibility |
| Agent Viewer | Read-only dashboards |

Secondary audience: **Tenants during onboarding** — the agent is sitting next to them, but the tenant is the one whose business is being set up. This handoff moment (agent → tenant) is jointly designed with Term 3.

---

## 6. Key Concepts This Term Will Produce

Concept documents under this Term:

| ID | Concept | What it answers |
|----|---------|-----------------|
| CN-2-001 | Regional Agent Identity & Roles | What an agent is, sub-roles, what they can and cannot do |
| CN-2-002 | Agent Onboarding & Activation | How a person becomes a working agent, from application to first sale |
| CN-2-003 | Regional Compliance Document Pipeline | What docs each country requires, how they are reviewed and refreshed |
| CN-2-004 | Tenant Onboarding Wizard | The step-by-step flow an agent uses to register a new business on BOS |
| CN-2-005 | Agent Dashboard | What an agent sees daily, what KPIs matter |
| CN-2-006 | Commission Visibility & Payout Request | How agents track earnings and request payouts |
| CN-2-007 | L1 Support Tooling | The tools agents use to help tenants with first-line problems |
| CN-2-008 | Tenant Health View (Agent Side) | What agents see about their tenants (usage, billing, risk) |
| CN-2-009 | Agent Training & Certification | The materials, exams, and renewal cycle for agents |
| CN-2-010 | Agent Staff Management | How an agent business adds employees and assigns them |
| CN-2-011 | Tenant Transfer Workflow | What happens when a tenant asks to switch agents |
| CN-2-012 | Agent Market Intelligence | Data an agent sees about their region (anonymised, non-leaky) |

---

## 7. Edge Cases to Consider

The hard questions this Term must answer:

**Onboarding Realities**
- A tenant arrives without a tax registration number (very common with informal businesses). Does onboarding proceed, pause, or warn?
- A tenant has 3 branches across 2 cities. Does the agent set up all 3 at once or progressively?
- A tenant is moving from a competitor system with 5 years of data. Does BOS import? If not, how is this conversation framed?
- A tenant is a non-literate business owner. How does onboarding handle this — through the agent acting on the owner's verbal consent?

**Trust and Boundaries**
- Can an agent log in *as* a tenant to help? If yes, how is this logged? If no, how does support happen?
- An agent discovers a tenant is being investigated for tax evasion. What's the agent's duty — to BOS, to the tenant, to the authority?
- An agent's family member is also a tenant. Conflict of interest?
- An agent wants to "white-label" BOS as their own brand. Allowed or not? Under what conditions?

**Commission Edge Cases**
- A tenant churns after 2 months. Does the agent get clawback? How much?
- A tenant grows 10x in one year. Does the agent's commission compound or cap?
- An agent suspended midway through the month. What happens to that month's accruing commission?
- A tenant pays late or partially. Is commission earned on invoice or on collection?

**Regional Compliance**
- An agent's regional compliance docs expire (e.g., annual business licence). What happens to their portal access? To their tenants?
- A country changes its tax rules. How does the agent learn? How does this update reach their tenants?
- An agent operates in a country that doesn't issue digital tax registration. Workaround?

**Multi-Agent Friction**
- A tenant has branches in two countries (Tanzania HQ, Kenya branch). Two agents? One agent? Who supports?
- An agent in Tanzania wants to sell to a Kenyan business. Allowed? (Probably no — Law 6 — but the doc must say so explicitly with reasoning.)
- A regional agent dies / goes out of business. What happens to their tenants?

**Support and Knowledge**
- An agent doesn't know how to answer a tenant's question about a specific engine. Where does the question go? How quickly?
- A tenant repeatedly contacts L2 support directly, bypassing the agent. Is this allowed? How does the agent get notified?

**Sub-Agents and Staff**
- An agent hires 5 staff. Each staff member onboards tenants. Whose name is on the agreement? Who gets the commission?
- An agent's staff member leaves and joins a competing agent. Do the tenants they onboarded follow?

---

## 8. Success Criteria

This Term is doing well when:

1. **A new agent can onboard a tenant in under 30 minutes** without calling support, using just the agent portal and the training docs.
2. **An agent always knows what they will earn this month** — no surprises at payout time.
3. **No agent can accidentally violate their region's compliance** — the portal prevents it.
4. **A tenant transfer between agents is fair, transparent, and traceable** — neither agent feels cheated.
5. **An agent who is suspended cannot harm their tenants** — tenants keep running, support shifts, compliance docs roll over.
6. **An agent can grow their business** — from 1 staff to 20, from 5 tenants to 500, without the portal becoming unusable.
7. **A regulator visiting an agent's office can see exactly what BOS has authorised that agent to do** — and exactly what compliance docs are on file.

---

## 9. Architect's Role for This Term

After Concept Team writes documents, the Architect translates them into:

| Concept Output | Architect Output |
|----------------|------------------|
| Agent identity model (CN-2-001) | Agent record schema, sub-role permission table |
| Agent onboarding (CN-2-002) | Application state machine, event types for status transitions |
| Compliance docs pipeline (CN-2-003) | Document upload model, review workflow, expiry watchers |
| Tenant onboarding wizard (CN-2-004) | Multi-step form spec, event types for each step, validation rules per country |
| Agent dashboard (CN-2-005) | Read-model projections, KPI aggregation queries |
| Commission view (CN-2-006) | Commission accrual projection, payout request command flow |
| L1 support (CN-2-007) | Ticket model, escalation policy, support log |
| Tenant health (CN-2-008) | Tenant-side projections agent is permitted to see (privacy-bounded) |
| Training & certification (CN-2-009) | Course content schema, progress tracking, expiry rules |
| Agent staff (CN-2-010) | Sub-account model, permission inheritance, audit trail |
| Transfer workflow (CN-2-011) | Transfer request command, approval state machine, attribution change events |
| Market intelligence (CN-2-012) | Aggregated read-model with k-anonymity guarantees |

**Architect's directive to Developer for Term 2:**
- Agent portal is a separate UI from Platform Admin and Tenant Dashboard, with distinct routes (`/agent/*`)
- Every agent action that affects a tenant emits an event tagged with the agent's ID
- Regional compliance is enforced before any tenant-affecting action
- Commission accrual is event-driven and re-computable from history (no cached writes)
- All agent ↔ tenant interactions go through the audit log

---

## 10. Relationships with Other Terms

### Term 2 depends on:

| Term | Why |
|------|-----|
| Term 1 (Platform Stewards) | Agent rules, commission policy, compliance pack approval process |
| Term 3 (Tenant Experience) | The onboarding wizard hands off to a tenant's first dashboard view — must be smooth |
| Term 4 (Foundation) | Identity primitives, audit log, security |
| Term 7 (Integration & Coherence) | Validates that agent workflows don't break tenant assumptions or platform policy |

### Term 2 is depended on by:

| Term | Why |
|------|-----|
| Term 3 (Tenant Experience) | The "first day" of a tenant is created by an agent; that handoff matters |
| Term 1 (Platform Stewards) | Platform needs to see agent performance to govern them |

### Cross-Term Hand-Offs

- ← Term 1: "Commission rules just changed." Term 2 redesigns the agent dashboard's earnings view.
- → Term 3: "Here's what a new tenant sees on day 1 after onboarding." Term 3 takes it from there.
- ← Term 4: "We added a new primitive: 'consent record'." Term 2 uses it for tenant authorisation during onboarding.
- ↔ Term 7: "An agent action in Tanzania needs to trigger an M-Pesa pre-auth check. Is this in scope?" Term 7 mediates.

---

## 11. Open Questions

1. **Agent acting "as" tenant.** Can an agent impersonate a tenant for support? If yes, with what restrictions and what visibility to the tenant?
2. **Sub-agent legal model.** Are agent staff employees of the agent, or independent contractors of BOS? This affects liability.
3. **Cross-border tenants.** Tenant HQ in Tanzania, branch in Kenya. One Tanzanian agent? Or one agent per country, with a "lead" relationship? Or platform-managed?
4. **Tenant self-discovery.** The Charter says tenants can't self-register. But can a tenant visit a public page, see local agents, and request contact — bypassing being "approached"?
5. **Commission appeal process.** If an agent disputes a commission calculation, where does the appeal go? Is there a formal SLA?
6. **Agent reputation.** Should tenants rate their agent? Public or private? Does it affect commission?
7. **Training enforcement.** If an agent's certification lapses, do they lose portal access immediately, or does a warning period apply?
8. **Marketing materials.** Can agents create their own marketing? Or must they use BOS-approved templates?
9. **Agent currency.** Agent in Tanzania has a tenant paying in TZS. Commission accrued in TZS. But agent wants to be paid in USD. Who handles the FX?
10. **Knowledge transfer.** When an agent retires/sells the business, how is their book of tenants transferred (with consent)?

---

## 12. Notes on Existing Repository

The existing repository has partial implementations of:
- Agent portal pages (`/agent/dashboard`, `/agent/tenants`, `/agent/profile`)
- Agent registration and lifecycle (Phase 12 onboarding scaffolding)
- Commission structures (with what appears to be commission-range tiers)
- Region packs

**Note:** The existing repo refers to "Global Agents" and "Regional Agents." Per Charter Law 6, this Term will collapse to **Regional Agents only**. Existing Global Agent constructs are reference; Term 2 will *redefine* agency as regional-only.

---

## 13. First Tasks for This Term

When this Term starts concept work, begin in this order:

1. **CN-2-001 first** — agent identity and roles, since everything else describes what an agent can do.
2. **CN-2-003 second** — regional compliance docs pipeline, because agents cannot exist without their region's docs.
3. **CN-2-002 third** — agent onboarding flow, which uses the previous two.
4. **CN-2-004 fourth** — tenant onboarding wizard, the highest-volume activity an agent will do.
5. **CN-2-005, CN-2-006 in parallel** — dashboard and commission, because they share data models.
6. Then CN-2-007 to CN-2-012 as edge cases surface.

---

## Standing Decisions & Cross-Term Work

> Read `DECISION-LOG.md`, `CROSS-TERM-COLLABORATION.md`, and `CROSS-TERM-REQUEST-REGISTER.md` before discussing these. File a CTR for every dependency. Bring each section to the **Concept Lead** and **Overseer (Term 7)**, discuss and agree first — the proposal is written only **after** agreement, and nothing is final without it.

| New Concept | From | This Term must decide |
|-------------|------|------------------------|
| CN-2-013 | D-002 | The **Agent Advisor** — AI that helps agents with onboarding/support (advisory only) |
| CN-2-014 | D-003 | The **multi-agent experience** and **referral flow**: who may refer (agent/tenant/public), the **attribution model** (who owns a referred tenant), fair-competition guardrails; update CN-2-011 transfer for attribution changes |

**CTRs to expect:** files CTR-011 (referral/commission → Term 1); receives CTR-012 (tenant discovery/referral ← Term 3) and CTR-013 (referral vs Law 6 ← Term 7); confirms CTR-010 (advisory-only).

---

*— End of Term 2 Brief —*
