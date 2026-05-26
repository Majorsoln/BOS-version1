# CN-4-021 — Saleable Line & Tender Value Shapes

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-001 (Universal Checkout), D-007 #1 (value shapes, not primitives; Checkout = Term 5 engine), D-009 (freeze doctrine — tax computed under pack version at emission).
> **CTRs:** CTR-001 (Term 6 → us, Saleable Line/Tender shapes — negotiating). CTR-003 (Term 5 → us, Checkout = engine not primitive — negotiating).
> **Glossary:** See `MASTER-GLOSSARY.md` — Saleable Line / Charge, Tender, Universal Checkout.
> **Depends on:** CN-4-001 (Doctrine), CN-4-002 (Event Store Contract), CN-4-005 (Engine Contract Model), CN-4-011 (Primitive Catalog), CN-4-015 (Compliance DSL).
> **Boundaries:** Checkout engine → Term 5; vertical bill production → Term 6; payment adapters → Term 7; checkout UX → Term 3; pricing → Term 1; promotions → Term 5.

---

## 1. What Value Shapes Are

Saleable Line and Tender are **value shapes** — immutable data structures with no lifecycle, no state machine, and no fold logic. They are carried inside events, not stored as entities with their own streams.

They are the handshake point between verticals (Term 6) and Universal Checkout (Term 5). Any vertical produces Saleable Lines; Checkout settles them using Tenders. Foundation owns the shapes; Term 5 builds the engine.

### Value Shapes vs Primitives

| | Primitives (CN-4-011) | Value Shapes (CN-4-021) |
|--|----------------------|------------------------|
| Have lifecycle (create, update, settle, etc.) | ✓ | ✗ |
| Have fold logic (compensating events) | ✓ | ✗ |
| Are building blocks engines compose | ✓ | ✓ (but simpler) |
| Are carried inside events | Sometimes (as references) | Always (as embedded data) |
| Owned by Foundation | ✓ | ✓ |

A Saleable Line is data inside an event, not an entity with its own stream. It is created once (when the vertical emits its bill) and consumed once (when Checkout settles it).

---

## 2. Saleable Line / Charge

A Saleable Line is the abstract payable unit any vertical produces. Checkout processes it without knowing which vertical created it.

### Shape

| Field | Type | Authority | Description |
|-------|------|-----------|-------------|
| **line_id** | Identifier | — | Unique within the bill/session |
| **item_ref** | Reference | Authoritative | Points to an Item/Service primitive instance (CN-4-011) |
| **description** | String | Authoritative | Human-readable label for the line (displayed on receipt) |
| **quantity** | Decimal | Authoritative | How many/how much |
| **unit_price** | Money (amount + currency) | Authoritative | Price per unit (pre-tax) |
| **tax_treatment_ref** | Reference | Authoritative | Points to the applicable tax treatment in the compliance pack. Tax is an input reference, not a computed value — the vertical declares which treatment applies; Checkout computes the tax amount at settlement using the pack version active at emission (D-009, CN-4-015). |
| **discount_refs** | List of references | Authoritative | Optional — discounts/promotions applied to this line (Term 5 Promotion Engine defines these) |
| **source_tag** | Opaque string | — | Identifies which vertical/context produced this line. **Opaque to Checkout** — Checkout carries it for traceability but never interprets it and never branches logic on it. A doctrine check (CN-4-019) may verify that no Checkout code path inspects source_tag content (Law 2). |
| **line_total** | Money | **Non-authoritative cache** | Pre-tax total: quantity × unit_price, adjusted by discounts. Carried for display convenience only. Checkout **must** recompute and verify — it never trusts this value blindly. If recomputation disagrees, the authoritative inputs (quantity, unit_price, discount_refs) win. |

### Authoritative vs Derived

The authoritative inputs are: `quantity`, `unit_price`, `discount_refs`, `tax_treatment_ref`. Everything computable from these is derived. `line_total` is a non-authoritative cache (pre-tax). The bill total and tax amounts are computed by Checkout at settlement — they are never carried on the line.

### Tax Computation (Law 5)

The vertical does **not** compute tax. It declares the `tax_treatment_ref` — a pointer into the compliance pack. Tax is computed at settlement by Checkout, using the compliance-pack version active at the time of emission (D-009, CN-4-015). This honours Law 5 (compliance is configured, not coded): tax rules live in packs, not in vertical logic or line shapes.

### Source Tag Guarantee (Law 2)

`source_tag` is opaque. Checkout may carry it in events and log it for traceability, but it is **forbidden** from branching logic on its content. If Checkout behaves differently based on source_tag, engine isolation is broken — the Checkout engine would "know" which vertical produced the line. Doctrine checks (CN-4-019) can enforce this.

---

## 3. Tender

A Tender is a payment method applied at settlement. Checkout accepts one or more tenders to settle the total of all Saleable Lines.

### Shape

| Field | Type | Description |
|-------|------|-------------|
| **tender_id** | Identifier | Unique within the settlement session |
| **method** | Registered category tag | Payment method category: cash, mobile_money, card, bank_transfer, credit, voucher. A controlled vocabulary validated against a method registry — extensible without Kernel change, but validated (not free-form, not a hard-coded enum). |
| **amount** | Money (amount + currency) | How much is applied via this method |
| **external_ref** | Reference | Optional — reference to external payment system (e.g., mobile money transaction ID). Opaque to the Kernel; meaningful to Term 7 adapters. |

### What the Tender Shape Does NOT Carry

- **No status field.** Tender is a value shape — lifecycle-free. The outcome of a tender (pending, confirmed, failed) is event-sourced by Checkout and Term 7 through tender-confirmation events, not through a mutable field on the shape.
- **No provider detail.** `method=mobile_money` is a category. Which provider handles it is Term 7's concern (payment adapters attach at the tender boundary, D-001).

### Multi-Tender Settlement

A single settlement can have multiple tenders (e.g., part cash, part mobile money). Checkout orchestrates the split. Foundation defines the shape of each tender; Checkout defines the settlement logic.

### Change Computation

If the sum of tender amounts exceeds the bill total (including tax), the difference is change. Change computation is Checkout's responsibility (Term 5), not Foundation's.

### Credit Tender → Obligation

When a tender has `method=credit` (or when the total is not fully covered by tenders), Checkout creates an **Obligation primitive** (CN-4-011) representing the amount owed. The Tender value shape records `method=credit` + `amount`; Checkout creates and manages the Obligation. The value shape does not contain the Obligation — it triggers its creation.

### Currency Invariant

A single settlement operates in **one currency**. All lines and all tenders in one settlement share the same currency. Cross-border / multi-currency settlement is the only exception (Charter §8.2 — currency in examples is local). The exact representation of currency (precision, rounding) is an Architect-phase detail.

---

## 4. The Vertical → Checkout Flow (D-001)

| Step | Who | What happens |
|------|-----|-------------|
| 1 | Vertical engine | Produces Saleable Lines from its business logic (basket, table bill, folio, quote). Declares tax_treatment_ref per line. Does NOT compute tax. |
| 2 | Vertical engine | Emits `<vertical>.bill.ready.v1` carrying the lines |
| 3 | Checkout (Term 5) | Receives the lines. Recomputes and verifies line_total from authoritative inputs. Computes tax per line using the compliance pack at the active version (D-009, CN-4-015). Displays total (pre-tax + tax). |
| 4 | Checkout (Term 5) | Accepts tenders from the cashier/user. Settles: validates totals, applies tenders, computes change, creates Obligation if credit, emits `checkout.settled.v1` |
| 5 | Downstream engines | Accounting, Cash, Inventory subscribe to `checkout.settled.v1` and process accordingly |

**Foundation provides step 0 (the shapes) only.** Steps 1–5 are engine logic owned by Terms 5/6.

---

## 5. Three Whys

### Why does this matter?

Without a shared line/tender shape, every vertical invents its own payment structure. Checkout would need to "know" each vertical — parsing retail baskets differently from restaurant bills differently from workshop quotes. Adding a new vertical would mean modifying Checkout. This is exactly the coupling Law 2 prohibits.

### Why does it belong in the Kernel?

The Saleable Line and Tender are the contract between verticals (Term 6) and Checkout (Term 5). If either Term defined the shape alone, the other would be forced to conform to a peer's design. Foundation defines the shape neutrally — neither vertical nor checkout "owns" it; both build to it.

### Why this design?

Value shapes (not primitives) because lines and tenders have no lifecycle — they are born inside an event and consumed once. Making them full primitives (with streams, fold logic, compensation) would over-engineer something that is conceptually a data transfer structure. The shapes are small, immutable, and self-contained — exactly what a handshake point needs.

---

## 6. Example — Karakana Settles a Quote

*Vertical names appear only as illustrations of the flow.*

A workshop completes a custom aluminium window. The workshop engine produces three Saleable Lines:

| line_id | description | qty | unit_price (TZS) | tax_treatment_ref | source_tag | line_total (TZS, pre-tax cache) |
|---------|-------------|-----|-------------------|-------------------|-----------|-------------------------------|
| L1 | Aluminium frame 1200×900mm | 1 | 85,000 | tz-vat-standard | workshop-quote-0087 | 85,000 |
| L2 | Glass panel 5mm clear | 2 | 22,000 | tz-vat-standard | workshop-quote-0087 | 44,000 |
| L3 | Installation labour | 1 | 30,000 | tz-vat-exempt | workshop-quote-0087 | 30,000 |

The workshop emits `workshop.bill.ready.v1` carrying these lines. **The workshop does not compute VAT.** It declares each line's `tax_treatment_ref` — that is all.

**Checkout receives the lines and computes tax:**
- Lines L1 and L2: `tz-vat-standard` → Checkout looks up the rate in the compliance pack (version active at emission, D-009). Say VAT = 18%. Tax on L1 = TZS 15,300. Tax on L2 = TZS 7,920.
- Line L3: `tz-vat-exempt` → no tax.
- Pre-tax total: TZS 159,000. VAT total: TZS 23,220. Grand total: TZS 182,220.

Checkout does not know what a "workshop" is — it sees three lines with amounts, tax treatment references, and an opaque source tag.

The customer pays with two tenders:

| tender_id | method | amount (TZS) | external_ref |
|-----------|--------|-------------|-------------|
| T1 | cash | 100,000 | — |
| T2 | mobile_money | 82,220 | MPESA-TXN-44821 |

Checkout settles: TZS 182,220 = TZS 100,000 cash + TZS 82,220 mobile money. No change. Receipt issued (Document primitive, CN-4-012). `checkout.settled.v1` emitted. Accounting, Cash subscribe and process. The mobile money tender's confirmation arrives as a separate event from Term 7's payment adapter — not as a status field on the tender shape.

---

## 7. Boundaries

| Topic | Lives in |
|-------|----------|
| Checkout engine (settlement logic, change, receipt issuance, tax computation at settlement) | Term 5 — Universal Engines |
| Vertical-specific bill production (how a basket, table bill, or quote becomes lines) | Term 6 — Vertical Engines |
| Payment adapters (M-Pesa, card processing, tender confirmation events) | Term 7 — attach at tender boundary (D-001) |
| Checkout UX (one screen for all verticals) | Term 3 (CTR-004) |
| Checkout pricing (always-on vs catalog item) | Term 1 (CTR-005) |
| Promotion/discount definitions | Term 5 (Promotion Engine) |
| Tax treatment rules (compliance pack content) | Compliance packs (Term 1/2/7) via CN-4-015 |
| Obligation primitive (created by Checkout on credit tender) | CN-4-011 |

---

## 8. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Exact money representation (amount + currency as a value object; precision, rounding) | Architect phase | Concept says "Money (amount + currency)"; Architect defines the implementation |
| Method registry: where are registered tender-method categories maintained? | Architect phase | Controlled vocabulary; extensible without Kernel change; Architect decides storage |
| Partial settlement / layaway / installment patterns | Term 5 | Checkout may emit partial settlement events; Foundation's shapes support it (multiple tenders, credit → Obligation) |
| Multi-currency settlement rules | Term 5 + Term 7 | Single-currency is the default; cross-border is an exception requiring adapter support |

---

*— End of CN-4-021 —*
