# Term 3 — Tenant Experience

> **Parent:** [BOS-CONCEPT-CHARTER.md](../BOS-CONCEPT-CHARTER.md) — read first
> **Type:** People Term (Part A)
> **Status:** Concept Phase v1.0

---

## 1. Mission

Tenant Experience is the heart of BOS. Every other part — engines, portals, primitives, agents, platform — exists to serve a single goal: **a business owner gets truth about their business, in their language, on their device, in a way they trust.**

This Term thinks about the people who actually use BOS to run real businesses: the shop owner closing for the day, the chef getting orders in the kitchen, the receptionist checking in a guest, the fundi cutting a window frame, the manager reviewing yesterday's sales. Their lives are not "user flows" — they are early mornings, late nights, low-light shops, slow internet, broken phones, customers waiting, and stress.

If this Term fails, BOS fails. The whole platform exists to make these moments work.

---

## 2. Mission Question

> **"How does Mama Asha — who sells nguo in Kariakoo, started business in 2008, can read but not write fluently, owns a 5-year-old Android phone, has 30-minute power outages, and trusts her notebook more than any computer — get the truth about her business and feel BOS is *hers*?"**

---

## 3. Scope (In)

This Term owns concepts for:

- **Tenant Dashboards** — what the owner, manager, and each role sees when they log in
- **Role-Based Experience** — how cashier, storekeeper, manager, owner each see different things
- **Mobile-First Design Philosophy** — BOS is used on phones more than laptops; this is a design law for this Term
- **Daily Workflows** — opening the shop, taking sales, closing the day, end-of-month
- **Reports the Owner Can Trust** — making numbers feel real, not abstract
- **First Day Experience** — what the tenant sees right after the agent finishes onboarding
- **Language and Tone** — Kiswahili, English, French (West Africa), and how BOS speaks to users
- **Offline Behaviour** — what the tenant can do when internet drops
- **Receipt and Customer Document Experience** — what the *end customer* sees (the buyer of a tenant's product)
- **Tenant Self-Service** — adding staff, adding items, adjusting prices, setting up a new branch
- **Notifications, Alerts, Reminders** — how the tenant learns about things needing attention
- **Trust and Confidence-Building Moments** — when BOS earns the tenant's trust, or loses it
- **Training and Help** — how a tenant learns BOS without the agent always being there

---

## 4. Scope (Out)

This Term does NOT own:

| What | Owned By |
|------|----------|
| The agent's experience and tools | Term 2 — Regional Distribution |
| Platform-side governance | Term 1 — Platform Stewards |
| Event store, primitives, security | Term 4 — Foundation |
| How accounting/inventory/cash actually compute things | Term 5 — Universal Engines |
| The internal mechanics of retail/restaurant/hotel/workshop | Term 6 — Vertical Engines |
| Integration with M-Pesa, banks, tax APIs | Term 7 — Integration & Coherence |
| Engine pricing visible to tenants | Term 1 owns rules; Term 3 designs how it's *shown* |

**Important nuance:** Term 3 owns the *surface* that tenants see. Underneath, every engine is owned by Term 5 or 6. Term 3 designs what's shown; Term 5/6 design what's computed.

---

## 5. Audiences Served

From Charter Section 3, this Term serves **Group C — Tenants** entirely:

| Role | What Term 3 designs for them |
|------|------------------------------|
| Tenant Owner | Full visibility, financial truth, decision support |
| Tenant Manager | Day-to-day operations, performance views, staff oversight |
| Cashier | Speed, simplicity, no mistakes possible |
| Storekeeper | Inventory accuracy, low friction stocking |
| Waiter / Chef | Restaurant-specific flows (orders, kitchen) |
| Fundi (Workshop) | Quote, cut list, job tracking |
| Receptionist (Hotel) | Check-in/out, room status, guest records |
| Accountant (Tenant-side) | Reports, period close, exports |
| Department Head | Their own KPIs, their staff |

And **Group D — Customers** (end customers) get a small but vital role:
- Receipt scanning (QR code verification)
- Self-service ordering (in Restaurant)
- Loyalty visibility

---

## 6. Key Concepts This Term Will Produce

Concept documents under this Term:

| ID | Concept | What it answers |
|----|---------|-----------------|
| CN-3-001 | Tenant Identity, Roles, and Onboarding Continuity | How a tenant comes alive on day 1, who logs in, and how roles emerge |
| CN-3-002 | The Owner's Dashboard | What an owner sees as their "home screen" |
| CN-3-003 | The Cashier Experience | The fastest possible POS-like flow with zero room for error |
| CN-3-004 | The Manager Experience | Mid-tier visibility — operations and staff oversight |
| CN-3-005 | The Storekeeper Experience | Inventory in and out, simple and clear |
| CN-3-006 | Vertical-Specific Frontline Roles | Waiter, chef, fundi, receptionist — each gets a focused doc |
| CN-3-007 | Mobile-First Design Principles | Touch targets, offline, low-light, low-bandwidth |
| CN-3-008 | Reports the Owner Trusts | How numbers are presented so the owner believes them |
| CN-3-009 | Language, Tone, and Cultural Voice | Kiswahili-first, plain language, no jargon |
| CN-3-010 | Notifications and Alerts | What gets notified, how, when, and to whom |
| CN-3-011 | Offline and Low-Connectivity Behaviour | What works without internet, what queues, what blocks |
| CN-3-012 | End-Customer Touchpoints | Receipts (paper, SMS, QR), self-service ordering, verification portal |
| CN-3-013 | Tenant Self-Service Setup | Adding items, staff, prices, branches without calling the agent |
| CN-3-014 | Training, Help, and In-App Learning | How tenants learn BOS through the product itself |
| CN-3-015 | Trust-Building Moments | Specific moments BOS must get right or lose the tenant |

---

## 7. Edge Cases to Consider

The hard questions this Term must answer:

**The Reality of Low Tech**
- A phone running Android 7 with 1GB RAM. Can it run BOS?
- The shop has Wi-Fi but it dies for 20 minutes every afternoon. What does BOS do during those 20 minutes?
- The owner has reading glasses and a 5-inch screen. What's the minimum font size we honour?
- The cashier is wearing oily gloves. Touch targets need to be how big?

**Trust Moments**
- The owner runs a sales report. It says "TZS 1,234,500 this week." The owner thinks "but I felt poorer this week." How does BOS prove the number?
- A receipt was issued and the customer disputes it 2 weeks later. How does the cashier or owner find it, verify it, and respond?
- The day's cash count doesn't match the system. Does BOS blame the cashier, or guide them through reconciliation?
- The owner sees an AI suggestion: "Reorder maize flour." How does the owner know whether to trust it?

**Speed Demands**
- At 6pm on a Friday, the queue is 8 customers deep. A sale takes 8 seconds. If we add an extra screen, queue becomes 12. What can never be added?
- A waiter taking 4 orders simultaneously, holding the phone in one hand. How is this UI shaped?

**Multi-Role Realities**
- The owner is also the cashier in the morning, the storekeeper at noon, and the accountant at night. How does role-switching work?
- The owner's daughter helps after school. She has no formal role. How is informal help handled?

**The "I Don't Know What I Don't Know" Problem**
- The owner has never seen a profit-and-loss statement. We show one. How do we make it actionable instead of intimidating?
- A new tenant doesn't know they can set tax rates per item. They put all items at default 18% VAT. Tax officer audits — half their items shouldn't have VAT. How does BOS prevent this earlier?

**Customer-Facing Moments**
- A customer asks "is this a real receipt?" The receipt has a QR code. How does verification work — fast, offline if needed, trustworthy?
- A restaurant has 20 tables with QR codes for self-ordering. A drunk customer orders 50 sodas. What stops the ridiculous order?

**Growth Moments**
- A tenant opens a second branch. Does the dashboard now show 2 columns? A drop-down? A toggle?
- A tenant hires their first employee. The owner now has to think about roles for the first time. How is this introduced gently?

**Cultural Realities**
- Many businesses run on credit ("kopa lipa baadaye"). Does BOS support this? How?
- "Customer is family" — discounts get given informally. How does BOS handle that without making the owner feel BOS is rigid?
- Religious holidays affect business patterns. Reports must be sensitive to this.

**Departure Moments**
- A tenant decides to leave BOS. They want their data. What do they get? How fast? In what format?
- A tenant's agent gets suspended. The tenant doesn't know yet. How are they told, and what changes?

---

## 8. Success Criteria

This Term is doing well when:

1. **A new cashier with no training can complete their first sale in under 60 seconds.** First-time use must work.
2. **An owner can answer "how much did we make yesterday?" in 3 taps from their phone.**
3. **A power cut for 2 hours does not lose a single sale.** Offline-first works.
4. **An owner who cannot read English fluently can use BOS confidently in Kiswahili.** Translation is not bolt-on.
5. **A customer disputing a receipt has the issue resolved in under 5 minutes** by any staff member, using just the system.
6. **A tenant uses BOS for 6 months without ever calling support.** Self-service works.
7. **An owner can switch from "I want to expand" intention to "second branch is live and selling" in one week** — including any agent assistance.
8. **The owner says "BOS feels like *mine*" — not "BOS is something I have to use."**

---

## 9. Architect's Role for This Term

After Concept Team writes documents, the Architect translates them into:

| Concept Output | Architect Output |
|----------------|------------------|
| Tenant identity (CN-3-001) | User account model with multi-role support, business context switching design |
| Owner dashboard (CN-3-002) | Aggregation projection design, KPI calculation pipeline |
| Cashier flow (CN-3-003) | POS UI state machine, offline queue design, sync logic |
| Mobile-first (CN-3-007) | PWA architecture, responsive breakpoints, accessibility checklist |
| Reports (CN-3-008) | Projection design for report types, drill-down query API |
| Language (CN-3-009) | i18n architecture, translation key catalog, locale-aware formatting |
| Notifications (CN-3-010) | Notification event types, delivery channel abstractions (in-app, SMS, push) |
| Offline (CN-3-011) | Service worker design, local storage strategy, conflict resolution |
| End-customer (CN-3-012) | Public receipt verification URL design, QR payload spec |
| Self-service (CN-3-013) | Settings UI design, validation rules, role-gated configuration |
| Training (CN-3-014) | In-app tour framework, contextual help anchor points |
| Trust moments (CN-3-015) | Specific UI patterns (e.g., "show your work" expandable explanations on numbers) |

**Architect's directive to Developer for Term 3:**
- The tenant app is a Progressive Web App, mobile-first
- All numbers shown to tenants must be traceable to their underlying events (drill-down)
- All UI strings go through i18n; no hardcoded English
- All touch targets ≥ 44px (accessibility)
- Offline queue uses event-sourcing principles: commands queue locally and replay when online
- Receipt QR codes are signed; verification works offline (cached public keys)

---

## 10. Relationships with Other Terms

### Term 3 depends on:

| Term | Why |
|------|-----|
| Term 1 (Platform Stewards) | Pricing visible to tenants; compliance rules that shape their documents |
| Term 2 (Regional Distribution) | Onboarding hand-off; the agent's role in tenant's first day |
| Term 4 (Foundation) | Identity, security, primitives, audit |
| Term 5 (Universal Engines) | The data Term 3 displays comes from engines |
| Term 6 (Vertical Engines) | Vertical-specific UI (POS, kitchen, room status, cut list) |
| Term 7 (Integration & Coherence) | Customer-facing integrations (SMS receipts, payment gateway feedback) |

### Term 3 is depended on by:

This Term is mostly a *consumer* of other Terms' output. But two important reverse dependencies:

| Term | Why |
|------|-----|
| Term 5 & 6 | If a tenant flow needs a new event or field, engines must add it. Term 3's UX needs *drive* engine evolution. |
| Term 7 | Tenant-side requirements often expose missing integrations. |

### Cross-Term Hand-Offs

- ← Term 2: "We just registered a new tenant. Here is the seed data and the agent's notes." Term 3 designs the first login.
- → Term 5: "The dashboard needs a 'cash on hand right now' number. Cash Engine, what event do we need?" Term 5 designs it.
- → Term 6: "Restaurant managers want to see 'covers per server'. Restaurant engine, can you emit this?" Term 6 designs it.
- ← Term 7: "We added SMS gateway integration. Here's the receipt-by-SMS contract." Term 3 wires it into the cashier flow.

---

## 11. Open Questions

1. **Multi-language reality.** Do we ship with full Kiswahili at launch, or English-only with Kiswahili coming later? (Strong recommendation: Kiswahili at launch for East Africa.)
2. **Identity portability.** If an owner has 3 businesses (some on BOS, some not), do they get one account or three? How is context switched?
3. **Customer accounts.** Does the *end customer* of a tenant have a relationship with BOS, or only with the tenant? (e.g., loyalty across multiple BOS-using tenants — yes/no?)
4. **AI visibility.** AI is advisory. But how much of AI's reasoning is shown to the tenant? Full chain-of-thought? Just the recommendation?
5. **Reports complexity.** Some tenants want simple reports, some want spreadsheets. How do we serve both without bloat?
6. **Credit (kopa lipa).** Do we model informal credit explicitly, or punt to Workshop's quote+invoice flow?
7. **Receipt format.** Paper printers in many shops. Do we support thermal printer protocols natively, or via agent install?
8. **Branch reporting independence.** Branch manager wants to see only their branch. Owner wants to see all. Can branch staff *see* owner-level data, ever?
9. **The "non-cash" handoff.** When a payment moves to M-Pesa, the cashier needs to know it arrived. How is the loop closed without depending on an integration that might lag?
10. **Day-end ritual.** Many shops have a "closing ritual" — count cash, settle the till, close the books. Is this ritual a first-class concept in BOS UX, or hidden in plumbing?

---

## 12. Notes on Existing Repository

The existing repository has:
- A Next.js frontend with `/dashboard` routes
- Some role-based access scaffolding
- Engine-specific pages (POS, kitchen workflow, workshop)

**Concept Phase guidance:** The existing UI was built bottom-up — features added as engines were built. Term 3 will redesign top-down — what the tenant needs, then how engines deliver it. Expect existing UI to be partially or fully reconceived.

**Critical:** Term 3 must not let the existing UI's structure constrain the new design. We are inventing the tenant experience from the user's lived day, not from the codebase's organisation.

---

## 13. First Tasks for This Term

When this Term starts concept work, begin in this order:

1. **CN-3-009 first** — language, tone, and cultural voice, because every doc must respect this from the start.
2. **CN-3-007 second** — mobile-first principles, because every interface follows these.
3. **CN-3-001 third** — tenant identity, roles, and first day, because the entry point shapes everything.
4. **CN-3-002 fourth** — owner dashboard, because this is the most-viewed screen.
5. **CN-3-003 fifth** — cashier experience, because this is the most-used screen.
6. **CN-3-015 sixth** — trust-building moments, because this is where BOS wins or loses tenants.
7. Then CN-3-004 to CN-3-014 in parallel as detailed UX work begins.

---

## 14. A Note on the User We Serve

Every concept doc in this Term must include at least one section titled "**For Mama Asha**" — describing how this specific concept feels, looks, and works for the archetypal user.

Mama Asha is not a persona invented for a slide deck. She is the reference human for every design decision. If a concept makes Mama Asha's life harder, it is rejected — no matter how elegant the technology.

She is the test.

---

## Standing Decisions & Cross-Term Work

> Read `DECISION-LOG.md`, `CROSS-TERM-COLLABORATION.md`, and `CROSS-TERM-REQUEST-REGISTER.md` before discussing these. File a CTR for every dependency. Bring each section to the **Concept Lead** and **Overseer (Term 7)**, discuss and agree first — the proposal is written only **after** agreement, and nothing is final without it.

| New Concept | From | This Term must decide |
|-------------|------|------------------------|
| CN-3-016 | D-001 | **One Universal Checkout UX** shared across all verticals (cashier flow, split payment, receipt) |
| CN-3-017 | D-002 | How AI **advice is presented** per role, and how much reasoning is shown (explainability / trust) |
| CN-3-018 | D-003 | Tenant-side **agent discovery** and **referral touchpoints** (without bypassing agent-led onboarding) |

**CTRs to expect:** files CTR-004 (checkout UI ← 5,6), CTR-009 (AI explainability ← 4,5), CTR-012 (agent discovery → Term 2); confirms CTR-010 (advisory-only).

---

*— End of Term 3 Brief —*
