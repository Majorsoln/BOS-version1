# Term 4 — Foundation
## Section 11 — Open Questions

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Status:** For Overseer review. Final brief-level section.
> **Governing decisions:** D-004 (neutral), D-005 (AI Mode), D-007 (scope resolutions), D-008 (consent), D-009 (freeze doctrine).
> **Glossary:** *Foundation* = this Term; *Kernel* = the artifact it designs (see `MASTER-GLOSSARY.md`).

---

## Open Questions

Brief §11 posed 10 open questions. Seven are now resolved by decisions or ratified sections. Three remain open and are assigned to CN docs. Five new questions have surfaced from our decisions.

---

### Resolved

These are closed. Recorded here so they are not re-raised.

| # | Original Question (Brief §11) | Resolution | Citation |
|---|-------------------------------|------------|----------|
| 1 | **Snapshot trust boundary.** How do we mark cached state as "non-truth" so no developer trusts it as source? | Non-truth marker type + CI doctrine check; runtime guard where feasible. | D-007 #10 |
| 2 | **Compensating events.** How is the corrected reading derived? Is "the truth" the latest, or the sum, or something else? | Truth = deterministic fold of all events including compensations; fold defined per primitive. | D-007 #3 |
| 3 | **Cross-tenant queries (platform).** How is the exception modelled — by a special "platform" actor with documented privileges? | Platform scope is a first-class parallel scope with its own audit trail — not an exception to isolation. Platform roles defined by Term 1. | D-007 #4, CTR-016 |
| 5 | **AI provider identity.** Is the provider/version recorded in the audit log? | Yes. The "advisor" principal type carries model identity/version. Decision Journal records model id/version on every suggestion. | D-007 #4, D-007 #6 |
| 6 | **Time zones across replay.** Is the business's local time anchored at registration, or recomputed each replay? | Anchored at registration, stored as event. Fiscal zone is a tenant property, not a clock property. Deterministic, never ambient. | Section 3 (Cluster E, ratified) |
| 7 | **Compliance DSL escape hatch.** Extend DSL (slow, careful) or allow custom function (dangerous, expressive)? | Reviewed, sandboxed extension mechanism — a middle path. Extensions are tested and reviewed like grammar changes; guarded against overuse. | D-007 #8 |
| 8 | **Document immutability vs corrections.** A document was issued with wrong VAT. The buyer disputes. We issue a credit note. Is the credit note a new document, an event that modifies the old, or both? | A credit note is a **new document** — with its own hash, number, and compliance-pack version — that references the original. The original remains immutable under its issuance-era rules. Not a modification. | D-009 |

---

### Still Open → CN Doc (from Brief §11)

| # | Original Question (Brief §11) | Assigned CN | Notes |
|---|-------------------------------|-------------|-------|
| 4 | **Identity as system.** When BOS itself acts (scheduled month-end close), what actor is recorded? A "system" principal? With what audit visibility? | CN-4-007 | D-007 #4 established fine-grained `system:<component>` principals. CN-4-007 must define: which system components get identities, what audit visibility they have, how scheduled actions are traced. |
| 9 | **Replay during incidents.** A bug emitted bad events for 4 hours. We want to ignore those events on replay. How is "selective replay" expressed without violating immutability? | CN-4-009 | Events are never deleted (Law 1). Direction: compensating events are appended for each bad event; the per-primitive fold then produces correct state. CN-4-009 must define the mechanism. |
| 10 | **AI knowledge boundary.** AI has read access to a tenant's events. Does AI also see other tenants' aggregated data (for benchmarking)? If yes, is this opt-in? | CN-4-022 | D-002A says cross-tenant benchmarking is opt-in only and never exposes identifiable data. CN-4-022 must define: how opt-in scope is expressed in the Advisor Framework, what "anonymised aggregate" means technically, and how the scope guard enforces it. |

---

### New Open → CN Doc (from Decisions)

| # | New Question | Source | Assigned CN | Notes |
|---|--------------|--------|-------------|-------|
| N1 | **Advisor cold-start.** A new tenant has no history. AI Mode is enabled from day one (D-005). What does the advisor do with zero data — generic advice, silence, or a minimum-data threshold? | D-005 | CN-4-022 | Framework-level: is there a "data sufficiency" concept in the advisor contract, or does each advisor define its own cold-start behaviour? |
| N2 | **Consent granularity.** Does consent apply per-channel (SMS yes, email no), per-purpose (marketing yes, transactional yes), or both? How granular is the consent record? | D-008 | CN-4-011 | Primitive shape must balance usability against compliance need. Too coarse = legally insufficient; too fine = unusable. |
| N3 | **Model-swap re-consent.** D-002A says the model is "swappable." If the model changes, do prior journal entries remain attributable? (Yes — journal records model id/version per entry.) But does a model swap require re-consent or notification to tenants? | D-002A, D-005 | CN-4-022 | Likely no re-consent (advisory-only, no state committed); Term 1 governs model approval. CN-4-022 should state this explicitly. |
| N4 | **AI Mode in DEGRADED.** AI Mode reads projections. In DEGRADED mode, projections may be stale. Does AI Mode degrade gracefully (warn about stale data), or shut off entirely? | D-005, D-007 #9 | CN-4-017 + CN-4-022 | Resilience Modes must define which capabilities degrade. Advisor Framework must define a "degraded" state for advisors. |
| N5 | **Freeze under a buggy compliance pack.** A compliance pack is later found to contain a bug. D-009 says events are interpreted under the rules active at emission. But those rules were wrong. How is the correction handled without violating freeze doctrine? | D-009 | CN-4-015 + CN-4-001 | Likely: a corrective pack version is issued; affected transactions get compensating events under the corrected rules; historical events still stand under the original (buggy) rules for audit purposes. CN-4-015 must define the correction workflow. |

---

### Needs Ruling

None. All questions are either resolved or clearly assignable to a CN doc.

---

## Summary

| Bucket | Count |
|--------|-------|
| Resolved (Brief §11) | 7 |
| Still Open → CN doc (Brief §11) | 3 |
| New Open → CN doc (from decisions) | 5 |
| Needs ruling | 0 |
| **Total open questions for CN docs** | **8** |

No new CTRs are implied — all questions fall within Foundation's own scope.

---

*— End of Section 11 —*
