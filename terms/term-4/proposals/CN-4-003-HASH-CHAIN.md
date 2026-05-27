# CN-4-003 — Hash Chain & Tamper Detection

> **Parent:** [BOS-CONCEPT-CHARTER.md](../../../BOS-CONCEPT-CHARTER.md) — read first
> **Term:** 4 — Foundation
> **Status:** For Overseer review.
> **Governing decisions:** D-009 (freeze doctrine — events interpreted under historical rules; hash chain proves they haven't changed).
> **Glossary:** See `MASTER-GLOSSARY.md`.
> **Depends on:** CN-4-001 (Doctrine — immutability, no deletion), CN-4-002 (Event Store Contract — envelope, global_position), CN-4-017 (Resilience Modes — breach may trigger DEGRADED).
> **Boundaries:** Event shape → CN-4-002; replay → CN-4-009; document verification → CN-4-012; doctrine check DC-031 → CN-4-019; breach governance → Term 1; hash algorithm → Architect phase.

---

## 1. What This Doc Defines

CN-4-003 defines how the event store proves its integrity: how events are linked into a tamper-proof chain, how integrity is verified, and what happens when the chain is breached. The hash chain is the mathematical foundation of BOS's legal defensibility — it answers "prove nothing was changed" with a verifiable, deterministic proof.

---

## 2. The Hash Chain

### What Is Hashed

The hash computation covers:
- The **entire event envelope** (all CN-4-002 fields **except** `event_hash` itself — the hash is the output, not an input)
- The **payload**
- The **previous_hash** (linking this event to the chain)

`event_hash` is computed from these inputs. It is not included in its own computation (that would be circular). `previous_hash` is included exactly once — as an input to the current event's hash, not recomputed from the previous event's content during the current computation.

This ensures recomputation is **deterministic**: given the same envelope + payload + previous_hash, the same event_hash is always produced. DC-031 (CN-4-019) must be able to independently recompute and verify every hash.

### The Hash Formula

For any event at position N:

```
event_hash(N) = hash( envelope(N) + payload(N) + previous_hash(N) )
```

Where:
- `envelope(N)` = all CN-4-002 envelope fields for event N, excluding event_hash
- `payload(N)` = the engine-specific payload of event N
- `previous_hash(N)` = event_hash(N-1) — the hash of the immediately preceding event by global_position

### Genesis Anchor

The first event in the store (global_position = 1) has no preceding event. Its `previous_hash` is a **well-known, permanent genesis constant** — a fixed value defined once and never changed. This anchors the chain: verification of the first event is well-defined, and the genesis constant is part of the system's public specification.

The genesis constant is not secret — it is a known starting point. Its value is an Architect decision; the concept requires it to be permanent, documented, and never changed after the first event is written.

### Chain Property

- **Forward linkage:** Each event's `previous_hash` links it to the preceding event. The chain runs from genesis to the latest event.
- **Tamper detection:** Changing any field of event N changes `event_hash(N)`. This breaks the chain at N+1 (whose `previous_hash` no longer matches). The break propagates forward — every subsequent event's chain link is invalid.
- **Verification:** Walk the chain from any point. Recompute each event_hash from stored content + stored previous_hash. If every recomputation matches the stored event_hash, the chain is intact.

---

## 3. Chain Topology — The Serialisation Trade-Off

A single global chain linked by `global_position` provides the strongest integrity guarantee: one chain proves the entire store is untampered. But it imposes a **serialisation point** — every write must know the previous event's hash, which means writes are ordered sequentially at the hash-computation layer.

### The Trade-Off

| Approach | Integrity guarantee | Write throughput | Isolation alignment |
|----------|-------------------|-----------------|-------------------|
| **(a) Single global chain** | Strongest — one chain proves everything | Sequential writes at the hash layer (bottleneck for high-volume multi-tenant systems) | Cross-tenant ordering proven by hash |
| **(b) Per-tenant chains anchored to global_position** | Per-tenant integrity proven by hash; cross-tenant ordering proven by global_position (but not by hash linkage) | Parallel writes per tenant (no cross-tenant hash dependency) | Aligns with CN-4-006 (tenant isolation — each tenant's chain is independent) |

### Decision

**Option (b): per-tenant chains, each anchored to global_position.** Each tenant has its own hash chain. Within a tenant's chain, every event links to the previous event for that tenant. Global ordering is provided by `global_position` (CN-4-002) but is not hash-linked across tenants.

Reasoning:
- **Isolation alignment:** CN-4-006 guarantees tenant A cannot see tenant B. Per-tenant chains reinforce this — each tenant's integrity is independently provable without involving other tenants' data.
- **Throughput:** Multi-tenant systems with hundreds of tenants writing simultaneously cannot serialise all writes through one hash computation. Per-tenant chains allow parallel writes.
- **Verification alignment:** Per-tenant verification (§5, GDPR export, audit) is natural — verify one tenant's chain without scanning the global store.
- **Cross-tenant integrity:** Global ordering is preserved by `global_position` (database-assigned, monotonic). Cross-tenant tampering (inserting/reordering events across tenants) is detectable via global_position gaps or anomalies, even without cross-tenant hash linkage.

The genesis constant applies per tenant — the first event for each tenant anchors that tenant's chain.

---

## 4. Verification

### When Verification Runs

| Trigger | What is verified |
|---------|-----------------|
| **On demand (audit)** | A tenant's full chain or a date range. An auditor, regulator, or tenant requests proof. |
| **Periodic (scheduled)** | System principal (`system:hash-verifier`) runs incremental or full chain verification per tenant on a schedule. Audited. |
| **On replay** | Before or during replay, the chain is verified for the scope being replayed. |
| **Doctrine check DC-031** | End-to-end hash chain verification runs in CI as part of the gate (CN-4-019). |

### Verification Result

| Result | What it means |
|--------|---------------|
| **Pass** | Every event's hash matches recomputation for the verified scope. The chain is intact. |
| **Fail at position N** | The recomputed hash at position N does not match the stored hash. |

### Tampered vs Unverifiable

When verification fails at position N, two things are true:
- **Event at position N is suspected-tampered.** Its content or its previous_hash has been altered, or there is storage corruption.
- **Events after position N are unverifiable** — their `previous_hash` depends on the integrity of N. They are not necessarily tampered; their integrity simply cannot be confirmed until the break at N is investigated.

This distinction matters for governance: "tampered at N" is specific and actionable; "unverifiable after N" is a broader scope that requires investigation before conclusions.

---

## 5. Breach Response Protocol

When verification fails, the Kernel responds predictably and traceably.

| Step | What happens |
|------|-------------|
| **1. Alert** | The breach is recorded as a high-severity event by `system:hash-verifier`. The alert includes: position of the break, expected vs actual hash, tenant scope, timestamp of detection. |
| **2. Quarantine** | The affected range is flagged: event at N is **suspected-tampered**; events after N are **unverifiable**. Projections derived from unverifiable events are marked stale. |
| **3. Scope assessment** | Is the break isolated (one event, likely corruption) or systemic (range of events, likely attack)? Automated analysis based on chain continuity before and after the break. |
| **4. Human escalation** | The breach is escalated to platform governance (Term 1). Foundation detects and reports; Term 1 decides the response (investigate, restore from backup, notify affected tenants). |
| **5. No silent recovery** | The Kernel does NOT automatically "fix" the chain. Any remediation is a governance decision, audited, and produces its own events. The breach alert itself is immutable — evidence of the breach is preserved. |

### What the Kernel Does NOT Do

- Does not delete the suspected-tampered events (CN-4-001 — no deletion).
- Does not silently recompute hashes to "heal" the chain (that would hide evidence).
- Does not halt the system automatically (a breach may trigger DEGRADED mode through CN-4-017's defined transition rules, but the hash verifier does not unilaterally change the system's operational state).

---

## 6. Three Whys

### Why does this matter?

Legal defensibility requires proof that the record has not been tampered with. A court asks: "How do you know this event was not edited after the fact?" The hash chain is the answer — change one event, and the chain breaks. The break is detectable, the evidence is preserved, and the proof is mathematical, not testimonial.

### Why does it belong in the Kernel?

If hash chaining were per-engine, each engine would chain its own events — but the integrity guarantee would be fragmented. The Kernel chains per tenant because the promise is per-tenant: "your business records have not been tampered with." The Kernel owns the chain because it owns the event store and the integrity guarantee.

### Why this design?

Per-tenant hash chains with global_position anchoring balance integrity, throughput, and isolation. A single global chain would serialise all writes (unacceptable for a multi-tenant system at scale). No chain at all would leave integrity unproven. Per-tenant chains give each tenant an independently verifiable proof of their records' integrity, while global_position preserves cross-tenant ordering for system-wide operations.

---

## 7. Example — TRA Auditor Requests Proof

TRA arrives at Mzee Hassan's store (CN-4-001 example) and says: "Prove your March records were not edited."

1. BOS runs a chain verification for tenant `mzee-hassan-duka`, March 2026.
2. For every event in the tenant's chain within the date range, BOS recomputes `event_hash` from stored envelope + payload + previous_hash.
3. Every hash matches. The chain is intact.
4. BOS produces a verification report: "Tenant: mzee-hassan-duka. Events verified: positions 401–892. Chain intact. No breaks. Verification timestamp: 2026-04-15T09:30."
5. The TRA auditor has mathematical proof: no event in Mzee Hassan's record was changed after recording.

**What if someone had altered a sale?** Say event 505 was modified (sale amount changed). The recomputed hash of event 505 no longer matches the stored hash. The chain breaks at 505. Events 506 onward are flagged as **unverifiable** (not necessarily tampered — but their integrity depends on 505, which failed). The breach protocol triggers: alert, quarantine, scope assessment, escalation to Term 1. The alteration is not just detected — it is **provable and preserved as evidence**.

---

## 8. Boundaries

| Topic | Lives in |
|-------|----------|
| Event shape / envelope (the fields that are hashed) | CN-4-002 |
| Event doctrine (why events are immutable) | CN-4-001 |
| Replay engine (uses verified events) | CN-4-009 |
| Document verification (hash of issued documents — separate from event-chain hash) | CN-4-012 |
| Doctrine check DC-031 (end-to-end chain verification in CI) | CN-4-019 |
| Breach governance (investigation, tenant notification, remediation) | Term 1 |
| Resilience mode transition on breach | CN-4-017 |
| Hash algorithm choice, chain storage optimisation, genesis constant value | Architect phase |

---

## 9. Open Items

| Item | Assigned to | Notes |
|------|-------------|-------|
| Hash algorithm selection (SHA-256, BLAKE3, etc.) | Architect phase | Concept requires: cryptographically strong, collision-resistant, deterministic |
| Genesis constant value | Architect phase | Must be permanent, documented, never changed after first event is written |
| Chain anchoring to external notarisation (publishing chain-tip hashes to an external timestamping service) | Architect phase / Future decision | Strengthens legal defensibility (independent proof) but adds external dependency |
| Breach → DEGRADED: does a hash breach automatically trigger DEGRADED? | CN-4-017 | Likely yes; the transition rules and health-check thresholds are CN-4-017's concern |
| Verification scheduling (frequency, scope, incremental vs full) | Term 1 + Architect | Governance decides frequency; Architect implements |
| Cross-tenant ordering anomaly detection (global_position gaps or sequence breaks) | Architect phase | Per-tenant chains don't hash-link across tenants; anomaly detection on global_position is a supplementary check |

---

*— End of CN-4-003 —*
