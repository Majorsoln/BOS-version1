# Term 4 — Foundation
## Section 8 — Success Criteria

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Status:** For Overseer review.
> **Governing decisions:** D-004 (no hard numbers), D-005 (AI Mode), D-007 (scope resolutions), D-008 (consent), D-009 (freeze doctrine).
> **Glossary:** *Foundation* = this Term; *Kernel* = the artifact it designs (see `MASTER-GLOSSARY.md`).

---

## Success Criteria

Foundation is doing well when the following are demonstrably true. Each criterion is testable — "we know it's met when…"

| # | Criterion | How Verified |
|---|-----------|--------------|
| 1 | A new engine can be added without modifying the Kernel. Extension points work. | Registration API (CN-4-020) accepts a new engine manifest and passes compatibility check with no Kernel code change. Automated invariant test. |
| 2 | A new country can be added without Foundation knowing about it. The Compliance DSL is expressive enough. | A compliance pack written in the DSL deploys without any Kernel code change. CI doctrine check. |
| 3 | An auditor can prove the event store has not been tampered with. The hash chain holds. | Hash chain verification runs end-to-end and produces a pass/fail result. Automated invariant test. |
| 4 | A projection can be rebuilt in production without taking the system offline. Replay is non-blocking. | Rebuild runs while stale-but-available projections serve reads; no downtime. Replay test. |
| 5 | Two tenants cannot accidentally share data. Isolation is provable, not just claimed. | Automated cross-tenant leak-detection tests (CN-4-019) prove no data crosses scope boundaries. CI invariant. |
| 6 | An AI suggestion can be traced back to the exact data and prompt that produced it. The Decision Journal is complete. | Audit query on any journal entry returns: model id/version, prompt ref, data ref, recommendation, human decision. Automated check. |
| 7 | A document issued today can be verified decades later. Hash, numbering, template version, and compliance-pack version are all preserved. Verification uses the issuance-era pack and template — not current versions. | Verification logic returns pass on any historical document using the pack/template version recorded at issuance (D-009). Automated test. |
| 8 | A developer cannot accidentally violate the six Laws. Doctrine enforcement catches it before production. | Doctrine enforcement checks (CN-4-019) run in CI. A deliberately-violating change is blocked. Automated invariant test suite. |
| 9 | A bug in one engine cannot crash the Kernel. Engine isolation is real. | A malformed event from one engine is rejected by the Command Bus; the Kernel continues operating. Isolation test. |
| 10 | Platform-scope access is fully audited. Every cross-tenant query by a platform principal produces an audit entry with actor, scope, and timestamp. | Audit query returns complete entries for all platform-scope operations. Automated test. |
| 11 | An advisor cannot read data outside its granted scope — provable at runtime. | Scope guard rejects an out-of-scope read attempt. Automated invariant test. |
| 12 | A compensating event produces correct truth. The per-primitive fold, after compensation, matches expected state. | Replay after compensation produces deterministically correct state. Per-primitive fold test. |
| 13 | Replay reproduces historical state under historical rules — never retroactively reinterpreted. | Replay with a later compliance-pack version still produces state using the pack active at emission time (D-009). Automated replay test. |
| 14 | The consent primitive records consent and opt-out immutably. The command bus rejects an outreach command when no valid consent exists or when opt-out is recorded. | Consent-gated command rejected without valid consent event; accepted with one. Automated invariant test. |
| 15 | Advisory-only holds for AI Mode at the framework level. No advisor commits state or writes events. | An advisor attempting a write is rejected by the Kernel. CI doctrine check. |

---

## Verification Pattern

Foundation's success is provable by machine, not by human inspection. Nearly all criteria lean on **automated invariant tests** (CN-4-019) running in CI:

- **Doctrine enforcement checks** — block violations before production.
- **Replay tests** — prove determinism and freeze doctrine (D-009).
- **Isolation tests** — prove cross-tenant and cross-engine separation.
- **Audit queries** — prove completeness of the journal and platform-scope trail.
- **Scope guard tests** — prove AI and platform scope boundaries hold at runtime.

If a criterion cannot be expressed as an automated test, it is either aspirational (and must be reworded) or it belongs to another Term.

---

## Belongs Elsewhere

The following are explicitly NOT Foundation success criteria:

| Excluded criterion | Why | Where it belongs |
|--------------------|-----|-----------------|
| AI Mode is available on every dashboard | UX availability, not Kernel integrity | Terms 3, 1, 2 |
| AI Mode gives useful, role-relevant advice | Quality of advice, not framework soundness | Terms 5, 2, 3 (specific advisors) |
| Compliance packs cover all current tax rules | Pack content, not DSL framework | Term 1 (approval), Term 2 (input) |
| Performance: projection rebuild completes within N minutes | Hard number, infrastructure concern | Architect phase / CN-4-009 (non-functional) |
| End-to-end "no outreach sent without consent" across channels | Cross-term (Term 5 outreach + Term 7 channel gateway) | D-008: Terms 5, 7; Foundation owns only the consent primitive and command-bus rejection |

---

*— End of Section 8 —*
